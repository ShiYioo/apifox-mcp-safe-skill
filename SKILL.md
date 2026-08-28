---
name: apifox-mcp-safe
description: Safely create, inspect, update, and validate Apifox HTTP APIs and test cases through MCP, especially when requests fail with "Invalid Parameter" or "Parameter is missing".
---

# Apifox MCP Safe

Use this skill whenever an Apifox API or test case must be created or changed through MCP. The goal is a verifiable Apifox artifact, not merely a successful MCP response.

## Non-negotiable workflow

1. Resolve IDs before mutating anything.
   - Call `listAccessibleProjects` and identify the project by name; never invent a project ID.
   - Call `getProjectSummary` when structure IDs may be stale or unknown. Use its main branch, module, and folder IDs.
   - Call `getStructureInfo` to locate the endpoint, then `readEntityDetails` or `getHttpEndpoint` to obtain the complete current definition.
   - For test cases, use `listTestCases` and `getTestCase` before updating; preserve the existing ID and unrelated fields.

2. Treat tool schemas literally.
   - Path parameters belong under `pathParams`; required request headers belong under `headers`; URL locale belongs under `queryParams`.
   - `updateHttpEndpoint` expects most nested fields as JSON strings: `parameters`, `requestBody`, `responses`, `responseExamples`, `auth`, `advancedSettings`, `commonParameters`, and similar fields. Do not pass nested objects to those fields.
   - `updateTestCase` expects nested objects, not JSON-encoded strings. Do not copy the endpoint update convention into test-case updates.
    - Supply required IDs (`projectId`, `apiDetailId`, `categoryId` when creating a test case) from Apifox responses only.
    - For every mutating HTTP call, include the client identity headers when available: `X-Client-Version: 2.7.17` and a non-empty `X-Device-Id` (for example `codex-client`). Include `queryParams: { locale: "zh-CN" }` as required by the MCP envelope.

3. Preserve the request media type.
   - For JSON endpoints, set the endpoint request body to `type: "json"` and add an enabled `Content-Type: application/json` header.
   - For test cases, set the header in `parameters.header` and set request body `type`, `contentType`, and `mediaType` to `application/json`.
   - Include the raw JSON body in all three fields when supported by the test-case schema: `data`, `raw`, and `json`. Keep the parameter list too, because Apifox uses it for its editor.
   - Never use `multipart/form-data` unless the server contract explicitly requires it.

4. Update from the current object, not a partial guess.
   - Start with the object returned by `getHttpEndpoint`/`getTestCase`.
   - Change only the requested fields, while retaining response IDs, examples, auth, advanced settings, visibility, and unrelated parameters.
    - For endpoint updates, `responses` itself is a JSON-encoded string, but each `responses[*].jsonSchema` value inside that string must be a JSON Schema object. Do not JSON-encode `jsonSchema` a second time; doing so persists the schema as a literal string and produces the wrong Apifox model.
    - A minimal `updateHttpEndpoint` is valid and was verified in practice: preserve the current endpoint, then send only `name`, `method`, `path`, `folderId`, `status`, `description`, plus JSON-string fields that are intentionally changed. Do not invent a complete replacement payload unless the read-back object has been converted exactly to the tool's JSON-string format.
    - Prefer updating a matching existing endpoint over creating a duplicate. In this project, the old endpoint ID was retained and its method/path/name were changed successfully; this also preserves existing response/schema IDs automatically.

5. Verify after every mutation.
   - Re-read the object with `getHttpEndpoint` or `getTestCase`.
   - Check the persisted values, not only `{success:true}`: IDs, path, method, request body type, media type, Content-Type, response ID, and raw body.
   - For test cases, call `listTestCases` filtered by endpoint ID and confirm the expected count and names.
   - If a client UI appears stale, trust the MCP read-back first and allow for UI cache delay.

## Error recovery

When Apifox returns `Parameter is missing`:

- Check the exact required envelope for the tool. For HTTP tools this normally means `headers["X-Project-Id"]` and `pathParams.http_api_id`/`projectId` as specified by the tool declaration.
- Add `X-Client-Version: 2.7.17` and a non-empty `X-Device-Id` if the API rejects an old or missing client identity. A low-version error is distinct from a payload error.
- Also include `queryParams: { locale: "zh-CN" }`; the generated OpenAPI envelope marks `queryParams` as required even when the endpoint itself has no query parameters.
- Confirm numeric IDs are numbers where the schema says number, while project headers/path values are strings where declared as strings.
- Retry only after correcting one concrete mismatch; do not blindly resend the same payload.

When Apifox returns `Invalid Parameter`:

- Compare the payload against the tool declaration and the last read-back object.
- Look for object/string mismatches, omitted required IDs, malformed JSON strings, unsupported enum values, and wrong path-parameter names (`http_api_id` versus `httpApiId`).
- Reduce to a minimal update containing the current object plus one intended change, then re-read and layer additional changes.
- Do not send guessed `auth`, `responses`, `responseExamples`, or `advancedSettings` objects during a first update. These fields are JSON strings with Apifox-specific shapes; malformed nested schemas, double-serialized `jsonSchema`, unsupported auth shapes, or response examples with object data can trigger `Invalid Parameter`. First update the scalar fields, verify, then add nested fields one at a time.

## Proven HTTP update envelope

```json
{
  "headers": {
    "X-Project-Id": "<PROJECT_ID>",
    "X-Client-Version": "2.7.17",
    "X-Device-Id": "codex-client"
  },
  "pathParams": { "http_api_id": <HTTP_API_ID> },
  "queryParams": { "locale": "zh-CN" },
  "body": {
    "name": "<ENDPOINT_NAME>",
    "method": "post",
    "path": "/api/robots/search/history/hot/refresh",
    "folderId": <FOLDER_ID>,
    "status": "released",
    "description": "<DESCRIPTION>",
    "parameters": "{\"path\":[],\"query\":[],\"cookie\":[],\"header\":[{\"name\":\"Authorization\",\"type\":\"string\",\"required\":true},{\"name\":\"Content-Type\",\"value\":\"application/json\",\"enable\":true}]}",
    "requestBody": "{\"type\":\"json\",\"parameters\":[]}"
  }
}
```

When a test run reports `Content type 'multipart/form-data' not supported`:

- Fix the test case, not just the endpoint documentation. Set `parameters.header` Content-Type and request-body `type/contentType/mediaType/data/raw/json` as described above.
- Re-read the test case and ensure the raw JSON is persisted before claiming the fix.

## HTTP validation

If the service is available locally, send one real HTTP request with `Content-Type: application/json` and a serialized JSON body. Use this only as complementary validation; a local 200 does not replace Apifox read-back.

## References

Read [references/payload-shapes.md](references/payload-shapes.md) when constructing or debugging a payload. It contains compact known-good shapes for endpoint and test-case updates and a failure matrix.
