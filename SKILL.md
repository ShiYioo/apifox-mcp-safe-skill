---
name: apifox-mcp-safe-redacted
description: 通用 Apifox 接口安全维护规则（脱敏版）
---

# Apifox 接口维护规则

1. 修改前先读取项目、分支、目录和接口详情，禁止猜测 ID。
2. 先核对代码中的 Controller 类级与方法级映射，得到完整实际路径。
3. 严格区分消费端与工业端路径，消费端前缀不能丢失，也不能用消费端路径覆盖工业端接口。
4. 同一 HTTP 方法和完整路径已存在时，更新原接口；只有不存在时才新增，避免重复接口。
5. 保留旧接口路径。仅修改参数、响应或说明时，不得擅自改路径。
6. 上传或更新接口时必须填写完整参数：请求体字段、路径参数、查询参数、请求头、required、请求示例、媒体类型、响应结构和响应示例。
7. 新增业务字段时，要补齐完整请求模型，不能只上传新增字段。SEO 场景必须包含可选字段：seoTitle、seoDescription、seoKeywords。
8. 完整 schema 不等于完成。请求示例必须是可直接发送的完整业务请求，覆盖所有可写业务字段及必要的嵌套对象/数组；保存接口禁止保留只含 ID、少数字段或只含 SEO 字段的简化示例。
9. 修改请求体前，必须核对 Controller 方法签名、实体/DTO 继承字段和嵌套 DTO；纳入所有后端可接收的业务字段。仅排除后端明确只读、忽略或由服务端维护的审计字段。
9.1 不能把 JSON 请求体当作完整请求。必须从 Controller 方法及其调用链确认鉴权、分页、路径参数、查询参数、请求头、Cookie 和服务端派生字段后，才可编写接口参数与示例。
9.2 `@RequestBody` 仅说明请求体绑定方式和媒体类型，不代表接口没有其他参数。Controller 调用 `startPage()` 时，必须将 `pageNum`、`pageSize`、`orderByColumn`、`isAsc`、`reasonable` 作为可选 URL 查询参数同步到 Apifox。
9.3 Controller 通过 `SecurityUtils.getUserId()` 或其派生方法获取当前用户/商家时，必须配置必填请求头 `Authorization: Bearer <access_token>`。不得把服务端从登录态解析的 `userId`、`merchantId` 等伪造成客户端请求字段；只有 Controller 明确接收时才可填写。
9.4 可直接发送的请求示例必须覆盖完整 HTTP 契约：方法、路径、必填请求头、查询参数和后端实际格式的请求体。JSON 接口除 Schema 字段示例外，还必须在 `requestBody.examples` 中保存完整 raw JSON 示例，`mediaType` 为 `application/json`。
10. 新增和修改接口的必填字段不同时，分别提供示例：修改示例必须含标识符，新增示例不得让人误以为服务端生成的标识符必填。
11. 响应也必须按后端真实返回类型维护：状态码、包装结构、泛型 data、分页 total/rows、嵌套字段、响应媒体类型和响应示例都要与 Controller 实际返回一致，禁止保留空的响应 schema 或猜测通用响应；XML、文本、二进制和文件响应必须使用对应媒体类型。
12. HTTP 接口的 parameters、requestBody、responses、responseExamples 等嵌套字段按工具要求使用 JSON 字符串；jsonSchema 内部保持对象，禁止二次序列化。
13. 不限定 JSON。Apifox 的传参格式和 Content-Type 必须与后端真实契约一致，可为 JSON、form-data、x-www-form-urlencoded、XML、Text、Binary、GraphQL、文件上传或无请求体。
14. 每次变更后重新读取接口，核对名称、方法、完整路径、参数、请求体和响应 schema、媒体类型及示例。
15. 出现 Parameter is missing 或 Invalid Parameter 时，先检查 envelope、参数类型、字段名和 JSON 字符串格式，只修正明确问题后重试。
16. 项目、域名、接口 ID、账号、Token、业务数据和环境地址不得写入脱敏规则。
17. 创建接口时先提交最小载荷（名称、方法、路径、目录、状态、说明、请求参数），再通过更新接口分层补充请求体与响应；创建请求直接携带嵌套的请求体或响应结构会被拒绝（Invalid Parameter）。
18. 字段较多或结构复杂的请求体、响应，优先先创建数据模型（schema），再在请求体或响应中以 $ref 引用；内联的大而复杂的 JSON Schema 可能间歇性触发校验失败，此时不要逐字符排查，直接切换为 $ref 引用。
19. 响应示例不能通过更新接口写入（提交会返回成功但不落库）；正确通道是 OpenAPI 导入：在导入规格对应响应的 content 中携带 example 字段，导入后自动生成成功示例。merge 模式可保留响应 ID 并挂到同状态码的响应上；个别路径 merge 稳定报错时，改用整覆盖模式且规格中必须携带完整定义。
23. 带响应列表的更新会清空该接口已有响应示例；仅更新参数、请求体等字段的更新不影响示例。收尾顺序必须固定：先完成全部结构更新，最后导入响应示例；示例导入之后不得再提交任何含响应列表的更新。
24. 导入按同方法同路径匹配既有接口：merge 会清空请求头参数、重置请求体类型，并覆盖响应结构（规格未带 schema 时置空）；整覆盖模式整体替换定义并把接口状态改为已发布。导入后如需修复参数与请求体，只能使用不含响应列表的更新。导入规格中“完整 schema 加示例”的组合超过约 2KB 容易被拒，纯示例规格可承受更大体积。
20. 更新响应列表时，每一项都要携带从读回结果取得的响应 ID；省略 ID 会导致每次更新都重新生成响应 ID，使已保存的引用失效。
21. 读回验证以单接口详情读取为准；聚合读取（如按实体读取 OAS 定义）可能命中缓存返回旧数据，不能作为最终验证依据。
22. 需要新目录时可通过开放接口创建（名称、父目录 ID、类型 http），父目录 ID 与类型均为字符串。

## AI 执行清单

1. 先从 Controller 类级和方法级注解组合出实际运行路径，再操作 Apifox。
2. 严格区分消费端、工业端、管理端和内部接口，禁止用一个作用域覆盖另一个作用域。
3. 相同 HTTP 方法加相同完整路径已存在时，只更新原接口；不存在才新增。
4. 除非用户明确要求，不得修改旧路径。
5. JSON 请求必须同时配置：requestBody.type=json、requestBody.mediaType=application/json、requestBody.parameters 参数行、匹配的 jsonSchema、有效 JSON 示例，以及启用的 Content-Type: application/json 请求头。
6. 不能只提交 jsonSchema，否则 Apifox 可能显示空参数表或错误的表单模式。
7. 查询接口也必须完整填写参数和示例；新增字段时要保留完整请求模型。SEO 请求必须包含可选 seoTitle、seoDescription、seoKeywords。
7.1 请求体为 JSON 不等于完整请求为 JSON：必须从 Controller 和调用链识别认证请求头、路径/查询参数、Cookie、服务端登录态派生字段和分页规则。
7.2 `@RequestBody` 仅定义 Body 绑定。出现 `startPage()` 时，在 Apifox 参数表配置可选 query 参数 `pageNum`、`pageSize`、`orderByColumn`、`isAsc`、`reasonable`；不得错误写入 JSON Body。
7.3 Controller 使用 `SecurityUtils.getUserId()` 或其派生逻辑识别用户/商家时，配置必填 `Authorization: Bearer <access_token>`，且不得将服务端解析的 `userId`、`merchantId` 误写成前端必传字段。
7.4 JSON 接口必须在 `requestBody.examples` 保存可直接发送的完整 raw JSON 示例；仅有 JSON Schema 或单字段 `example` 不算请求示例。
8. 保存接口的 schema 和示例必须一致：示例覆盖全部可写业务字段与必要的嵌套字段，不得只展示 ID、少数字段或新加字段；创建、修改要求不同时分别给出示例。
9. 响应 schema 和响应示例也必须与后端真实返回一致；分页、统一响应包装和 XML 等特殊响应必须按实际结构配置，不能只填请求参数。
10. 接口嵌套字段按 MCP 要求使用 JSON 字符串；其中 jsonSchema 必须保持对象，禁止二次序列化。
11. 每次变更后必须重新读取接口，核对完整路径、作用域、方法、请求/响应媒体类型、请求头、参数行、请求与响应 schema、required、请求与响应示例；仅返回 success 不算完成。

最后更新：2026-09-17
