# Apifox MCP Safe

针对 Apifox MCP 在创建或更新 API、测试用例时返回信息过于笼统的问题，帮助 AI 从反复出现的 `Parameter is missing` 和 `Invalid Parameter` 中恢复，并最终完成可验证的导入或更新。

Apifox 的这两类错误通常不会指出缺失的是哪个字段、路径参数名称是否错误、嵌套对象是否应序列化为 JSON 字符串，或 ID 与类型是否不匹配。AI 若仅依据错误文本重试，往往会原样发送错误请求，陷入循环。本技能将排查流程和已验证的 payload 约束固化为可执行规则。

## 解决的问题

- `Parameter is missing`：定位缺失的请求信封字段、错误的路径参数名，或未从 Apifox 响应中取得的必需 ID。
- `Invalid Parameter`：识别对象与 JSON 字符串混用、枚举值错误、双重序列化和不兼容的嵌套结构。
- API 导入或更新失败：先读取已有对象，再以最小 payload 更新，避免猜测完整的 Apifox 内部模型。
- 测试用例提交后变成 `multipart/form-data`：强制将请求体与请求头校正为 JSON，并通过读回确认持久化结果。
- MCP 返回成功但界面没有更新：以 MCP 读回结果为准，区分 UI 缓存与实际写入失败。

## 核心方法

1. **先解析 ID，再执行写操作**

   从 `listAccessibleProjects`、`getProjectSummary`、`getStructureInfo` 和详情读取接口取得项目、目录、接口、测试用例 ID；不要猜测 ID。

2. **严格按工具声明构造信封**

   HTTP 接口请求通常需要 `headers`、`pathParams` 和 `queryParams`。特别注意 `http_api_id` 与 `httpApiId` 并不等价；即使接口没有业务查询参数，也应携带 `queryParams: { "locale": "zh-CN" }`。

3. **分清“对象”和“JSON 字符串”**

   `updateHttpEndpoint` 的 `parameters`、`requestBody`、`responses` 等嵌套字段通常需要 JSON 字符串；`updateTestCase` 的对应嵌套字段通常需要对象。两种约定不可互换。

4. **从当前对象增量修改**

   先用 `getHttpEndpoint` 或 `getTestCase` 读取当前定义，只改需求涉及的字段。首次更新使用最小 payload，验证成功后再逐项加入响应、鉴权、示例和高级设置。

5. **每次写入都读回验证**

   不把 `{ "success": true }` 当作完成。重新读取并检查路径、方法、请求体类型、`Content-Type`、响应 ID 和原始 JSON 是否真正保存。

## 处理两类模糊报错

### `Parameter is missing`

按以下顺序检查：

1. 对照 MCP 工具声明，确认 `headers`、`pathParams`、`queryParams` 和 `body` 是否齐全。
2. 检查路径参数名称是否精确匹配，例如 `http_api_id`。
3. 确认 `X-Project-Id`、`projectId`、`apiDetailId`、`categoryId` 等必填 ID 均来自实际 Apifox 响应。
4. 按声明校验 ID 类型：有些位置要求数字，有些位置要求字符串。
5. 补充客户端标识：`X-Client-Version: 2.7.17` 和非空 `X-Device-Id`。
6. 每次只修复一个已确认的不匹配项，再重试并读回。

### `Invalid Parameter`

按以下顺序缩小问题：

1. 将 payload 与工具声明及最近一次读回对象逐字段对照。
2. 检查 JSON 字符串字段是否传成对象，或对象字段是否被错误 `JSON.stringify`。
3. 检查嵌套 JSON 是否可解析，枚举值是否受支持。
4. 移除猜测生成的 `responses`、`auth`、`responseExamples`、`advancedSettings` 等复杂字段。
5. 用名称、方法、路径、目录等标量字段做最小更新；成功后再逐项恢复复杂字段。
6. `responses` 可以整体序列化，但其内部的 `jsonSchema` 必须保留为对象，不能再次序列化为字符串。

## 推荐的 MCP 工作流

```text
读取项目和结构 ID
  -> 读取接口或测试用例当前详情
  -> 按工具声明构造最小更新 payload
  -> 提交更新
  -> 读回确认持久化结果
  -> 逐步加入其余字段并重复验证
```

对于创建接口持续得到 `Invalid Parameter` 的场景，优先查找语义相同的已有接口，并使用 `updateHttpEndpoint` 做最小更新。这能保留既有响应和 schema ID，且比猜测完整创建请求更容易成功。

## 请求体约束

JSON 接口应满足：

- 接口定义的请求体类型为 `json`。
- 存在启用的 `Content-Type: application/json` 请求头。
- 测试用例的 `requestBody.type`、`contentType`、`mediaType` 均为 JSON。
- 测试用例的 `data`、`raw`、`json` 为包含 JSON 的字符串，同时保留参数列表。
- 除非服务端契约明确要求，否则不要使用 `multipart/form-data`。

## 安装与使用

将此目录放入 Codex 的 skills 目录，或在任务中加载 `apifox-mcp-safe` 技能。当需求涉及通过 Apifox MCP 创建、导入、更新 HTTP API 或测试用例，尤其出现上述两类错误时，技能会要求 AI 按读取、最小更新、读回验证的闭环执行，而不是盲目重发同一请求。

详细的已验证请求结构请查看 [SKILL.md](SKILL.md) 与 [references/payload-shapes.md](references/payload-shapes.md)。

## 适用边界

本技能解决的是 Apifox MCP 请求结构和导入过程中的可诊断性不足，不替代服务端接口本身的调试。即使本地真实 HTTP 请求返回 200，也仍应执行 Apifox MCP 读回，以确认定义或测试用例已正确写入。
