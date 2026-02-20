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

## Java Unit Testing Conventions

When generating, fixing, or discussing Java unit tests in this Spring Boot Maven project:

- Use **JUnit 5** (Jupiter) with **AssertJ** (`assertThat`) for all assertions
- Use **Mockito** via `@ExtendWith(MockitoExtension.class)` for service/utility tests
- Use **`@WebMvcTest`** + `MockMvc` for controller tests (not `@SpringBootTest` unless integration)
- Use **`@DataJpaTest`** for repository tests with embedded H2
- Use **`@WithMockUser`** / `SecurityMockMvcRequestPostProcessors` for security tests
- Target **100% line and branch coverage** via JaCoCo
- Add `@DisplayName` with human-readable descriptions on every test method
- Group tests using `@Nested` classes by method under test
- Use `@ParameterizedTest` with `@CsvSource` or `@MethodSource` for boundary values
- Naming convention: `methodName_givenCondition_expectedResult`
- Never use `@Disabled` to skip tests — fix them instead
- Never use `Thread.sleep()` — use Awaitility for async tests
- Never share mutable state between tests
- Test file location: `src/test/java/` in matching package, named `{ClassName}Test.java`
- For Lombok classes, add `lombok.addLombokGeneratedAnnotation = true` to `lombok.config` to exclude generated code from coverage
