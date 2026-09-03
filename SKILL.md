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
10. 新增和修改接口的必填字段不同时，分别提供示例：修改示例必须含标识符，新增示例不得让人误以为服务端生成的标识符必填。
11. HTTP 接口的 parameters、requestBody、responses、responseExamples 等嵌套字段按工具要求使用 JSON 字符串；jsonSchema 内部保持对象，禁止二次序列化。
12. 不限定 JSON。Apifox 的传参格式和 Content-Type 必须与后端真实契约一致，可为 JSON、form-data、x-www-form-urlencoded、XML、Text、Binary、GraphQL、文件上传或无请求体。
13. 每次变更后重新读取接口，核对名称、方法、完整路径、参数、请求体 schema、媒体类型和示例。
14. 出现 Parameter is missing 或 Invalid Parameter 时，先检查 envelope、参数类型、字段名和 JSON 字符串格式，只修正明确问题后重试。
15. 项目、域名、接口 ID、账号、Token、业务数据和环境地址不得写入脱敏规则。

## AI 执行清单

1. 先从 Controller 类级和方法级注解组合出实际运行路径，再操作 Apifox。
2. 严格区分消费端、工业端、管理端和内部接口，禁止用一个作用域覆盖另一个作用域。
3. 相同 HTTP 方法加相同完整路径已存在时，只更新原接口；不存在才新增。
4. 除非用户明确要求，不得修改旧路径。
5. JSON 请求必须同时配置：requestBody.type=json、requestBody.mediaType=application/json、requestBody.parameters 参数行、匹配的 jsonSchema、有效 JSON 示例，以及启用的 Content-Type: application/json 请求头。
6. 不能只提交 jsonSchema，否则 Apifox 可能显示空参数表或错误的表单模式。
7. 查询接口也必须完整填写参数和示例；新增字段时要保留完整请求模型。SEO 请求必须包含可选 seoTitle、seoDescription、seoKeywords。
8. 保存接口的 schema 和示例必须一致：示例覆盖全部可写业务字段与必要的嵌套字段，不得只展示 ID、少数字段或新加字段；创建、修改要求不同时分别给出示例。
9. 接口嵌套字段按 MCP 要求使用 JSON 字符串；其中 jsonSchema 必须保持对象，禁止二次序列化。
10. 每次变更后必须重新读取接口，核对完整路径、作用域、方法、媒体类型、请求头、参数行、schema、required、示例和响应；仅返回 success 不算完成。

最后更新：2026-09-03
