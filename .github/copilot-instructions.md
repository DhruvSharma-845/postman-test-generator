## API Testing Conventions

When generating or discussing Postman tests or API test cases:

- Output Postman Collection v2.1 JSON format, importable directly into Postman
- Use `pm.test()` with `pm.expect()` (Chai BDD assertions) for all test scripts
- Never hardcode secrets, tokens, or URLs — use `{{collectionVariable}}` syntax
- Use Postman dynamic variables (`{{$randomEmail}}`, `{{$randomUUID}}`, `{{$timestamp}}`) for unique test data
- Store created resource IDs via `pm.collectionVariables.set()` for request chaining
- Organize test folders by resource, then by category: Happy Path, Authentication, Validation, Edge Cases, Data Integrity
- Always include a Cleanup folder at the end to delete test data
- Cover these edge case categories for every endpoint: happy path, auth/authz (401/403), required field validation, type mismatches, boundary values, SQL/XSS/NoSQL injection, path params (404, invalid format), query params (pagination, sorting, filtering), wrong HTTP method (405), wrong Content-Type (415), and CRUD data integrity flows
- Adapt error response assertions to the framework's error format
