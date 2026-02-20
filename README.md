# Copilot Agents for Spring Boot — API & Unit Testing

Two GitHub Copilot agents + GitHub Actions workflow for a Spring Boot Maven project:

1. **Postman Test Generator** — generates Postman Collection v2.1 JSON for all REST endpoints
2. **Unit Test Generator** — fixes broken tests, creates missing ones, targets 100% JaCoCo coverage
3. **GitHub Actions** — runs tests + enforces 100% coverage on every push/PR

## Setup

Copy the `.github` folder into your Spring Boot project root:

```
your-spring-boot-project/
├── .github/
│   ├── agents/
│   │   ├── postman-test-generator.agent.md    # Postman test agent
│   │   └── unit-test-generator.agent.md       # Unit test agent
│   ├── prompts/
│   │   ├── generate-postman-tests.prompt.md   # Postman prompt file
│   │   └── generate-unit-tests.prompt.md      # Unit test prompt file
│   ├── workflows/
│   │   └── test-and-coverage.yml              # CI pipeline
│   └── copilot-instructions.md                # Always-on conventions
├── src/
├── pom.xml
└── ...
```

### Prerequisites

1. VS Code with GitHub Copilot Chat extension
2. Enable prompt files in VS Code settings:
   ```json
   { "github.copilot.chat.promptFiles": true }
   ```
3. Agent mode available (VS Code 1.99+)

## Usage

### Postman Test Generator

```
# Option A: Select the agent from the agent picker
@postman_test_generator generate Postman tests for all my APIs

# Option B: Use the prompt file
/generate-postman-tests

# Option C: Reference in chat
#prompt:generate-postman-tests
```

**Output**: `postman/<name>.postman_collection.json` — import directly into Postman.

### Unit Test Generator

```
# Option A: Select the agent
@unit_test_generator fix all broken tests and reach 100% coverage

# Option B: Use the prompt file
/generate-unit-tests

# Option C: Reference in chat
#prompt:generate-unit-tests
```

**What it does**:
1. Scans `pom.xml` — adds JaCoCo plugin if missing
2. Maps every class → its test file, finds gaps
3. Runs `mvn test` — fixes compilation errors and failures
4. Generates missing tests for controllers, services, repositories, DTOs, configs, exception handlers, utilities, security
5. Runs `mvn clean verify` — repeats until 100% coverage

### GitHub Actions (automatic)

The `test-and-coverage.yml` workflow runs on every push/PR to `main`, `master`, or `develop`:

1. Checks out code, sets up JDK 17 + Maven cache
2. Runs `mvn clean verify` (tests + JaCoCo enforcement)
3. Posts a **coverage report comment** on PRs with line/branch percentages
4. **Fails the build** if overall coverage drops below 100%
5. Uploads JaCoCo HTML report + Surefire reports as downloadable artifacts

To adjust the JDK version, edit `java-version` in the workflow. To adjust the coverage threshold, edit `min-coverage-overall` and the JaCoCo plugin `<minimum>` in `pom.xml`.

## What the Unit Test Agent Generates

| Source Layer | Test Annotation | What Gets Tested |
|---|---|---|
| `@RestController` | `@WebMvcTest` + `MockMvc` | Happy path, validation (400), auth (401/403), service exceptions (404/500) |
| `@Service` | `@MockitoExtension` | Every public method, every if/else branch, null/empty inputs, exceptions |
| `@Repository` | `@DataJpaTest` | Custom `@Query`, derived queries, `@Modifying` queries |
| DTOs / Entities | Plain JUnit 5 | Constructors, getters/setters, equals/hashCode, toString, builders |
| `@Configuration` | `@SpringBootTest` | Every `@Bean` method, conditional beans |
| `@ControllerAdvice` | `@WebMvcTest` or unit | Every `@ExceptionHandler` with correct status + body |
| Utility classes | Plain JUnit 5 | All static methods, edge cases, private constructor |
| Security config | `@SpringBootTest` + `MockMvc` | Public vs protected endpoints, role-based access |

## Files Reference

| File | When It Activates | Purpose |
|------|-------------------|---------|
| `copilot-instructions.md` | Every Copilot Chat request | Adds API + unit testing conventions to all responses |
| `postman-test-generator.agent.md` | When selected or auto-inferred | Full Postman collection generation agent |
| `unit-test-generator.agent.md` | When selected or auto-inferred | Fix + generate unit tests for 100% coverage |
| `generate-postman-tests.prompt.md` | `/generate-postman-tests` | On-demand Postman prompt |
| `generate-unit-tests.prompt.md` | `/generate-unit-tests` | On-demand unit test prompt |
| `test-and-coverage.yml` | Push/PR to main/master/develop | CI pipeline: tests + coverage gate |
