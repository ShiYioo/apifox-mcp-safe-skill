# Apifox MCP payload shapes

## HTTP endpoint update

Required envelope:

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
    "parameters": "{\"path\":[],\"query\":[],\"cookie\":[],\"header\":[{\"name\":\"Content-Type\",\"value\":\"application/json\"}]}",
    "requestBody": "{\"type\":\"json\",\"parameters\":[{\"name\":\"keyword\",\"type\":\"string\"}]}"
  }
}
```

### Verified recovery pattern

When creating with `createHttpEndpoint` repeatedly returns `Invalid Parameter`, first locate a semantically matching endpoint and use `updateHttpEndpoint` with the smallest scalar payload. Use the generic shape below:

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

The successful call deliberately omitted `responses`, `auth`, `advancedSettings`, examples, and custom fields. Those fields should be copied from a fresh read-back object or added one at a time. When adding responses, keep the outer `responses` field as a JSON string, but keep each inner `jsonSchema` as an object; do not call `JSON.stringify` on the schema itself.

Nested endpoint fields are JSON strings. `pathParams` is not the same as the endpoint's `parameters.path` array.

## Test-case update

Required envelope:

```json
{
  "headers": {
    "X-Project-Id": "<PROJECT_ID>",
    "X-Client-Version": "2.7.17",
    "X-Device-Id": "codex-client"
  },
  "pathParams": { "projectId": <PROJECT_ID_NUMBER>, "id": <TEST_CASE_ID> },
  "queryParams": { "locale": "zh-CN" },
  "body": {
    "id": <TEST_CASE_ID>,
    "apiDetailId": <HTTP_API_ID>,
    "projectId": <PROJECT_ID_NUMBER>,
    "categoryId": <CATEGORY_ID>,
    "parameters": {
      "path": [], "query": [], "cookie": [],
      "header": [{ "name": "Content-Type", "value": "application/json", "enable": true }]
    },
    "requestBody": {
      "type": "json",
      "contentType": "application/json",
      "mediaType": "application/json",
      "data": "{\"keyword\":\"学习\",\"limit\":20}",
      "raw": "{\"keyword\":\"学习\",\"limit\":20}",
      "json": "{\"keyword\":\"学习\",\"limit\":20}",
      "parameters": [
       { "name": "<PARAMETER_NAME>", "type": "string", "value": "<VALUE>", "enable": true }
      ]
    },
    "responseId": 200
  }
}
```

Test-case nested fields are objects. `requestBody.data/raw/json` are strings containing JSON, not objects.

## Failure matrix

| Symptom | Likely cause | Corrective check |
|---|---|---|
| `Parameter is missing` | Missing envelope field or wrong path key | Compare tool declaration; verify `headers`, `pathParams`, and required IDs |
| `Invalid Parameter` | Wrong type, enum, or object/string encoding | Compare with read-back; validate every nested JSON string |
| Client version too low | Missing/old client identity | Use `X-Client-Version: 2.7.17` and non-empty `X-Device-Id` |
| Multipart not supported | Test case persisted form-data mode | Read back `type/contentType/mediaType` and `data/raw/json` |
| MCP success but UI unchanged | Apifox client cache | Re-read through MCP, then wait for client refresh |
