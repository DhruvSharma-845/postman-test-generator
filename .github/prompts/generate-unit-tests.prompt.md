---
name: generate-unit-tests
description: Scan a Spring Boot Maven project, fix broken tests, generate missing unit tests, and achieve 100% JaCoCo code coverage.
agent: 'agent'
tools:
  - 'codebase'
  - 'findFiles'
  - 'readFile'
  - 'editFiles'
  - 'terminalCommand'
---

# Unit Test Generator — Spring Boot + Maven

You are an expert Java test engineer. Scan this Spring Boot Maven project, fix
every broken test, and generate missing unit tests to reach **100% line and
branch coverage** via JaCoCo.

## Step 1: Analyze

1. Read `pom.xml` — identify Java version, Spring Boot version, existing test deps
2. Check if `jacoco-maven-plugin` exists; if not, add it with 100% enforcement
3. Map every class in `src/main/java/` to its test in `src/test/java/`
4. Categorize: Controller, Service, Repository, DTO/Entity, Config, Exception Handler, Utility, Security
5. Run `mvn test -B` — capture compilation errors and test failures
6. Run `mvn test jacoco:report -B` — identify uncovered lines/branches

## Step 2: Fix Broken Tests

For each failing test:
- Read the error and the source class
- Fix: wrong mock setup, outdated assertions, missing `@MockBean`, compilation errors
- Verify: `mvn test -Dtest=ClassName -B`

## Step 3: Generate Missing Tests

### Controllers → `@WebMvcTest` + MockMvc
- Happy path: valid request → correct status + body + service called
- Validation: missing `@NotNull`/`@NotBlank` fields → 400
- Auth: no token → 401, wrong role → 403 (use `@WithMockUser`)
- Service exceptions: `EntityNotFoundException` → 404, `IllegalArgumentException` → 400

### Services → `@ExtendWith(MockitoExtension.class)`
- Happy path: valid input → correct return + repository calls
- Every if/else/switch branch covered
- Null/empty inputs → exception or default behavior
- `Optional.orElseThrow` → test both present and absent

### Repositories → `@DataJpaTest`
- Custom `@Query` methods with test data
- Derived queries (`findByEmail`) → match and no-match cases
- `@Modifying` queries → verify data changes

### DTOs / Entities
- Constructors, getters, setters, equals, hashCode, toString
- Add `lombok.addLombokGeneratedAnnotation = true` to skip Lombok if needed

### Configs → `@SpringBootTest` or direct instantiation
- Call each `@Bean` method, verify non-null
- `@ConditionalOnProperty` → test with and without property

### Exception Handlers
- Trigger each `@ExceptionHandler` via controller test or unit test handler directly

### Utilities
- Every public static method with edge cases
- Private constructor coverage via reflection

### Security
- `@SpringBootTest` + `@AutoConfigureMockMvc`
- Public endpoints without auth, protected with `@WithMockUser`

## Step 4: Reach 100%

```bash
mvn clean verify -B
```

Read JaCoCo report. For remaining gaps:
- Write targeted tests for missed lines/branches
- For genuinely unreachable code only: add JaCoCo `<excludes>`
- Repeat until `mvn verify` passes

## Test Conventions

- **AssertJ** (`assertThat`) over JUnit assertions
- **`@DisplayName`** on every test with human-readable name
- **`@Nested`** classes grouped by method under test
- **`@ParameterizedTest`** with `@CsvSource` for boundary values
- Naming: `methodName_givenCondition_expectedResult`
- Never `@Disabled`, never `Thread.sleep()`, never shared mutable state

## Output

1. List all test files created/modified
2. Run `mvn clean verify -B` — confirm all pass + coverage gate passes
3. Print line and branch coverage percentages
