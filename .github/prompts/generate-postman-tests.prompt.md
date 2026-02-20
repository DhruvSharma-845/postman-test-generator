---
name: generate-postman-tests
description: Analyze repo structure and generate a comprehensive Postman Collection v2.1 JSON with test cases for all REST API endpoints, covering every edge case.
agent: 'agent'
tools:
  - 'codebase'
  - 'findFiles'
  - 'readFile'
  - 'editFiles'
  - 'terminalCommand'
---

# Postman Test Cases Generator

You are an expert API test engineer. Your job is to analyze the current repository's codebase, discover all REST API endpoints, and generate a **complete Postman Collection v2.1 JSON file** with exhaustive test cases covering every edge case.

## Phase 1: Discover API Endpoints

Scan the repo for route/controller definitions. Search for these patterns:

- **Express**: `router.get/post/put/patch/delete`, `app.get/post/...`
- **NestJS**: `@Get()`, `@Post()`, `@Put()`, `@Patch()`, `@Delete()`, `@Controller()`
- **FastAPI**: `@app.get`, `@router.post`, etc.
- **Django**: `urlpatterns`, `path()`, `re_path()`, ViewSets
- **Spring Boot**: `@GetMapping`, `@PostMapping`, `@RequestMapping`
- **Go (Gin/Echo/Chi)**: `r.GET`, `e.POST`, `r.Route`
- **Rails**: `resources`, `get`, `post` in `routes.rb`
- **ASP.NET**: `[HttpGet]`, `[HttpPost]`, `MapGet`, `MapPost`
- **OpenAPI/Swagger**: `swagger.json`, `openapi.yaml`, `openapi.json`

For each endpoint, extract:
- HTTP method and full path (including path params like `:id` or `{id}`)
- Request body schema (from models, DTOs, validation schemas, type definitions)
- Query parameters and headers
- Authentication requirements
- Response schemas and status codes
- Validation rules (min/max, required, regex, enum values from Joi, Zod, class-validator, Pydantic, etc.)

Also identify:
- Auth mechanism (JWT, API key, OAuth2, Basic)
- Base URL pattern and environment variables
- Common headers (Content-Type, Accept)
- Database models and relationships

## Phase 2: Generate Test Cases

For **every** discovered endpoint, generate test cases from ALL applicable categories:

### Category 1: Happy Path
- Valid request with all required fields returns expected status (200/201/204)
- Verify response body structure matches expected schema
- Verify Content-Type response header
- Verify response time is under 2000ms

### Category 2: Authentication & Authorization
- No auth token → 401
- Expired/invalid token → 401
- Malformed token → 401
- Valid token but insufficient permissions → 403
- Test each role's access level if role-based auth exists

### Category 3: Required Fields Validation
- Omit each required field individually → 400/422
- Empty string for each required field
- Null for each required field
- Completely empty body

### Category 4: Type Mismatch Validation
- String where number expected
- Number where string expected
- Array where object expected
- Boolean where string expected

### Category 5: Boundary Values
- Strings: empty, 1 char, max length, max+1 length
- Numbers: 0, negative, min-1, min, max, max+1, decimal where integer expected
- Arrays: empty, single item, max items, max+1
- Dates: past, future, invalid format, epoch boundary

### Category 6: Security / Injection
- SQL injection: `'; DROP TABLE--`, `1 OR 1=1`
- XSS: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`
- NoSQL injection: `{"$gt": ""}`
- Path traversal: `../../../etc/passwd`
- Unicode and emoji in text fields

### Category 7: Path Parameters
- Valid ID → 200
- Non-existent ID → 404
- Invalid format (string where UUID expected, negative number) → 400/404
- Zero as ID, very large number as ID

### Category 8: Query Parameters
- Valid pagination returns correct subset
- Invalid page/limit (negative, zero, string)
- Sort by non-existent field
- Exceeding max page size

### Category 9: HTTP Method & Content-Type
- Wrong HTTP method → 405
- Wrong Content-Type → 415 or 400

### Category 10: Rate Limiting & Idempotency
- Rapid repeated requests → 429 (if rate limiting exists)
- PUT/DELETE idempotency: repeated calls produce same result

### Category 11: Data Integrity (CRUD flow)
- Create → Read: resource is retrievable
- Create → Update: changes persist
- Create → Delete: resource removed, GET returns 404
- PATCH only modifies specified fields

### Category 12: List/Search Endpoints
- Empty collection returns 200 with empty array
- Pagination: first page, last page, out-of-range page
- Sorting: ascending, descending
- Filtering: exact match, partial match, no match
- Combining filters + sort + pagination

## Phase 3: Build the Postman Collection JSON

Output a valid Postman Collection v2.1 JSON using this structure:

```json
{
  "info": {
    "name": "<API_NAME> - Test Suite",
    "_postman_id": "<generate-uuid>",
    "description": "Auto-generated test suite. Required env vars:\n- baseUrl\n- authToken\n- adminToken\n\nRun folders top-to-bottom.",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "variable": [
    { "key": "baseUrl", "value": "http://localhost:3000", "type": "string" },
    { "key": "authToken", "value": "", "type": "string" }
  ],
  "item": [
    {
      "name": "<Resource Name>",
      "item": [
        { "name": "Happy Path", "item": [ /* request items */ ] },
        { "name": "Authentication", "item": [ /* request items */ ] },
        { "name": "Validation", "item": [ /* request items */ ] },
        { "name": "Edge Cases", "item": [ /* request items */ ] },
        { "name": "Data Integrity", "item": [ /* request items */ ] }
      ]
    }
  ]
}
```

Each request item must follow this format:

```json
{
  "name": "Descriptive test name (expected status)",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Content-Type", "value": "application/json" },
      { "key": "Authorization", "value": "Bearer {{authToken}}" }
    ],
    "body": {
      "mode": "raw",
      "raw": "{ ... }",
      "options": { "raw": { "language": "json" } }
    },
    "url": {
      "raw": "{{baseUrl}}/api/resource",
      "host": ["{{baseUrl}}"],
      "path": ["api", "resource"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "type": "text/javascript",
        "exec": [
          "pm.test('Status is 201', function () {",
          "    pm.response.to.have.status(201);",
          "});",
          "pm.test('Has expected fields', function () {",
          "    const json = pm.response.json();",
          "    pm.expect(json).to.have.property('id');",
          "    pm.collectionVariables.set('resourceId', json.id);",
          "});"
        ]
      }
    }
  ]
}
```

### Test Script Conventions

- Use `pm.test()` with `pm.expect()` (Chai assertions) for all checks
- Store dynamic values via `pm.collectionVariables.set()` (created IDs, tokens)
- Use pre-request scripts for auth token setup
- Use Postman dynamic variables: `{{$randomFirstName}}`, `{{$randomEmail}}`, `{{$randomUUID}}`, `{{$timestamp}}`
- Never hardcode secrets — always use `{{variable}}` syntax
- Add a Cleanup folder at the end with DELETE requests to remove test data
- Adapt error assertions to the framework's error format (NestJS `{ statusCode, message, error }` vs Express `{ error }` vs DRF `{ detail }` vs Spring Boot `{ timestamp, status, error, path }`)

### Auth Pre-Request Script Pattern (JWT)

For APIs using JWT, add this as a collection-level pre-request script:

```javascript
const tokenExpiry = pm.collectionVariables.get('tokenExpiry');
if (tokenExpiry && Date.now() < parseInt(tokenExpiry)) return;

pm.sendRequest({
    url: pm.collectionVariables.get('baseUrl') + '/api/auth/login',
    method: 'POST',
    header: { 'Content-Type': 'application/json' },
    body: {
        mode: 'raw',
        raw: JSON.stringify({
            email: pm.collectionVariables.get('testUserEmail'),
            password: pm.collectionVariables.get('testUserPassword')
        })
    }
}, function (err, res) {
    if (!err && res.code === 200) {
        const json = res.json();
        pm.collectionVariables.set('authToken', json.token || json.access_token);
        pm.collectionVariables.set('tokenExpiry', String(Date.now() + 3500000));
    }
});
```

## Phase 4: Output

1. Write the collection to `postman/<collection-name>.postman_collection.json` in the project root
2. Provide a summary:
   - Total endpoints discovered
   - Total test cases generated
   - Breakdown by category
   - Any endpoints that couldn't be fully analyzed (with reasons)
3. List required environment variables the user must configure

## Important Rules

- Respect validation libraries (Joi, Zod, class-validator, Pydantic) — derive rules from their schemas
- If an OpenAPI/Swagger spec exists, use it as primary source of truth, cross-referenced with code
- Order requests logically: Create before Read/Update/Delete
- Handle pagination defaults — test with and without explicit params
- For security tests, assert the server does NOT return 500 (injection should be handled gracefully)
