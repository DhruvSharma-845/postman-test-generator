# Postman Test Generator for GitHub Copilot Chat

Generates comprehensive Postman Collection v2.1 JSON files with test cases for all REST API endpoints, covering every edge case.

## Setup

Copy the `.github` folder into any repository's root:

```
your-repo/
├── .github/
│   ├── copilot-instructions.md          # Always-on API testing conventions
│   └── prompts/
│       └── generate-postman-tests.prompt.md  # On-demand test generator
└── ... your code ...
```

### Prerequisites

1. VS Code with GitHub Copilot Chat extension installed
2. Enable prompt files in VS Code settings:
   ```json
   {
     "github.copilot.chat.promptFiles": true
   }
   ```
3. Ensure Copilot agent mode is available (VS Code 1.99+)

## Usage

### Option 1: Invoke the prompt file (recommended)

Open Copilot Chat and type:

```
/generate-postman-tests
```

Copilot will scan your codebase and generate a full Postman collection.

### Option 2: Reference it in chat

In any Copilot Chat conversation, reference the prompt:

```
#prompt:generate-postman-tests — generate tests for the users and orders APIs
```

### Option 3: General chat (uses copilot-instructions.md automatically)

Just ask Copilot in chat:

```
Generate Postman test cases for all my API endpoints
```

The `copilot-instructions.md` ensures Copilot follows the right conventions.

## What Gets Generated

- A `postman/<name>.postman_collection.json` file importable into Postman
- Tests organized by resource, then by category:
  - Happy Path
  - Authentication (401/403)
  - Validation (required fields, type mismatches)
  - Edge Cases (boundary values, injection, special characters)
  - Data Integrity (CRUD flows)
- Pre-request scripts for auth token management
- Cleanup folder to remove test data

## Supported Frameworks

Express, NestJS, FastAPI, Django, Spring Boot, Go (Gin/Echo/Chi), Rails, ASP.NET, and any API with an OpenAPI/Swagger spec.

## Files Explained

| File | When It Activates | What It Does |
|------|-------------------|--------------|
| `copilot-instructions.md` | Every Copilot Chat request in this repo | Adds API testing conventions to all responses |
| `generate-postman-tests.prompt.md` | When you invoke `/generate-postman-tests` | Full agent workflow: scan → analyze → generate collection |
