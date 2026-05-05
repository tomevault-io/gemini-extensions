## dynatrace-managed-mcp

> You are a Developer working on the Dynatrace Managed Model-Context-Protocol (MCP) Server project.

# Dynatrace Managed MCP Server

You are a Developer working on the Dynatrace Managed Model-Context-Protocol (MCP) Server project.

This is a TypeScript/Node.js MCP server that connects AI assistants to self-hosted Dynatrace Managed deployments. It uses the MCP SDK (https://www.npmjs.com/package/@modelcontextprotocol/sdk) to expose Dynatrace observability data as MCP `tools` and `resources`.

## Build, Test, and Lint Commands

```bash
# Build
npm run build              # Compile TypeScript to dist/

# Test
npm test                   # Run all tests
npm run test:unit          # Unit tests only (src/**/__tests__/*.test.ts)
npm run test:integration   # Integration tests (tests/**/*.integration.test.ts)
npm run coverage           # Generate coverage report (coverage/lcov-report/index.html)

# Run specific test file
npm test -- tests/integration/capabilities.integration.test.ts

# Formatting
npm run prettier           # Check formatting
npm run prettier:fix       # Auto-fix formatting

# Run server
npm run serve              # Run HTTP server on port 8080
node --env-file=.env ./dist/index.js        # Run with stdio transport
node --env-file=.env ./dist/index.js --http # Run with HTTP transport

# Development tools
npx @modelcontextprotocol/inspector node --env-file=.env ./dist/index.js  # MCP Inspector
```

**Integration tests** require a `.env` file with real Dynatrace Managed credentials (see `.env.template`). Tests expect two environments: one with alias `testAlias` (valid) and one with alias `invalidApiToken` (wrong token for error testing).

## Architecture

### High-Level Structure

```
MCP Client (AI Assistant)
    ↓
MCP Server (src/index.ts)
    ↓
Capability Clients (src/capabilities/*.ts)
    ↓
Auth Client Manager (src/authentication/managed-auth-client.ts)
    ↓
Dynatrace Managed API (multiple environments)
```

The MCP server supports **multi-environment** setups via `DT_ENVIRONMENT_CONFIGS`, allowing queries across multiple Dynatrace Managed instances.

### Key Components

- **`src/index.ts`**: Main entrypoint. Registers MCP tools, handles tool routing, defines MCP metadata/instructions
- **`src/capabilities/*.ts`**: Each file implements one API domain (problems, entities, logs, metrics, etc.)
  - Contains both API calls and response formatting logic
  - Exports an API client class (e.g., `ProblemsApiClient`)
  - Client methods like `listProblems()` make API calls
  - Formatting methods like `formatList()` convert responses to LLM-friendly strings
- **`src/authentication/managed-auth-client.ts`**: `ManagedAuthClientManager` manages API clients for multiple environments, handles authentication
- **`src/utils/`**: Shared utilities (logging, date formatting, environment parsing, telemetry)

### MCP Tooling Principles

1. **Tool naming**: Prefix all tools with `dynatrace_managed_` to avoid conflicts with Dynatrace SaaS MCP (e.g., `dynatrace_managed_list_problems`)
2. **Response formatting**:
   - **List tools** (e.g., `list_problems`): Extract key fields, format as readable strings for the LLM
   - **Detail tools** (e.g., `get_problem_details`): Return raw JSON with contextual wrapper and "Next Steps" recommendations
3. **Next Steps**: Include recommendations in tool responses to guide the LLM (e.g., suggest viewing Dynatrace UI, using related tools)
4. **Error handling**: Let errors bubble to `index.ts` tool wrapper; trust Dynatrace API errors to be meaningful

## Repository Structure

```
src/
├── index.ts                          # MCP server entrypoint, tool registration
├── authentication/
│   ├── managed-auth-client.ts        # Multi-environment auth manager
│   └── __tests__/*.test.ts           # Auth unit tests
├── capabilities/                     # API domain implementations
│   ├── problems-api.ts               # Problems API client
│   ├── entities-api.ts               # Entities API client
│   ├── logs-api.ts                   # Logs API client
│   ├── metrics-api.ts                # Metrics API client
│   ├── events-api.ts                 # Events API client
│   ├── security-api.ts               # Security problems API client
│   ├── slo-api.ts                    # SLO API client
│   └── __tests__/*.test.ts           # Capability unit tests
├── utils/
│   ├── logger.ts                     # Winston-based logging
│   ├── environment.ts                # Parse DT_ENVIRONMENT_CONFIGS
│   ├── telemetry-openkit.ts          # OpenKit telemetry
│   ├── date-formatter.ts             # Timestamp formatting
│   ├── user-agent.ts                 # User-Agent header generation
│   ├── version.ts                    # Package version utils
│   └── __tests__/*.test.ts           # Util unit tests
└── interfaces/                       # TypeScript type definitions

tests/
├── integration/*.integration.test.ts # Integration tests (real API calls)
└── api-contract/*.test.ts            # API contract validation tests

dist/                                 # Build output (compiled JS)
```

## Key Conventions

### Testing Strategy

- **Unit tests** (`src/**/__tests__/*.test.ts`): Mock API responses, test formatting logic defensively
- **Integration tests** (`tests/**/*.integration.test.ts`): Real API calls to Dynatrace Managed with `.env` credentials
- **API contract tests** (`tests/api-contract/*.test.ts`): Validate code matches real API responses
- All response object fields are declared as **optional** (defensive coding—don't assume API response structure)

### Logging

- Use `winston` logger from `src/utils/logger.ts` (not `console.log`)
- Debug logs should include: tool call parameters, API responses (full JSON), tool return values
- Logs written to `dynatrace-managed-mcp.log` in CWD
- Set `LOG_LEVEL=debug` for development
- Minimal `console.error()` use (only startup message)—prefer logging

### Response Processing

**Lists** (e.g., `list_problems`):

- Define strongly-typed interfaces with all fields optional
- Extract pertinent info and format as strings (LLMs struggle with complex JSON)

**Details** (e.g., `get_problem_details`):

- Return raw JSON (preserves all data, handles varying Davis configurations)
- Wrap with context and Next Steps recommendations

### Dependencies

**Allowed only:**

- `@modelcontextprotocol/sdk` (MCP framework)
- `zod-to-json-schema` (schema validation)
- `axios` (HTTP client)
- `winston` (logging)
- `commander` (CLI parsing)
- `@dynatrace/openkit-js` (telemetry)

**Do not install other dependencies** without discussion.

### Authentication

- Uses API tokens (not OAuth) passed in HTTP headers
- Multi-environment support via `DT_ENVIRONMENT_CONFIGS` JSON array
- Required API scopes documented in `README.md#api-scopes-for-managed-deployment`

### Changelog Maintenance

- Add entries to `CHANGELOG.md` under `## Unreleased Changes` for new features
- Follow `.github/instructions/changelog.instructions.md` for formatting
- Use past tense, user-centric language, semantic versioning

## Development Guidelines

- **Complete implementations**: No TODOs, placeholders, or missing pieces
- **Readability over performance**: Code should be clear and maintainable
- **Add tests for bugs**: When fixing bugs, add test cases
- **Defensive typing**: All API response fields should be optional
- **Verify builds**: Always run `npm run build` and `npm run serve` after changes
- **Check logs**: Review `dynatrace-managed-mcp.log` during development

---
> Source: [dynatrace-oss/dynatrace-managed-mcp](https://github.com/dynatrace-oss/dynatrace-managed-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-05 -->
