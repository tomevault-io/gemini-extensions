## openapi-generic-sdk

> This is the **OpenAPI Generic SDK** - a language-agnostic reference implementation (pseudocode + Rust reference) for building OpenAPI client libraries. It is NOT a functional SDK itself but a blueprint for implementing SDKs in specific languages.

# CLAUDE.md

## Project Overview

This is the **OpenAPI Generic SDK** - a language-agnostic reference implementation (pseudocode + Rust reference) for building OpenAPI client libraries. It is NOT a functional SDK itself but a blueprint for implementing SDKs in specific languages.

- **Author**: Michael Cuffaro (@maiku1008)
- **Maintainer**: Francesco Bianco
- **License**: MIT
- **Version**: 1.0.0 (SDK_VERSION in Makefile)

## Project Structure

```
openapi-generic-sdk/
├── src/                          # Pseudocode specifications
│   ├── oauth_client.pseudo       # OauthClient class spec
│   ├── api_client.pseudo         # ApiClient class spec
│   └── types.pseudo              # Type definitions, structs, enums, constants
├── tests/                        # Pseudocode test specifications
│   ├── oauth_client_tests.pseudo
│   └── api_client_tests.pseudo
├── examples/                     # Pseudocode usage examples
│   ├── token_generation.pseudo
│   └── api_calls.pseudo
├── docs/
│   └── implementation-guide.md   # Detailed implementation guide with language examples
├── reference/                    # Rust reference implementation (crate: openapi-sdk)
│   ├── Cargo.toml
│   ├── src/lib.rs                # Full Rust implementation
│   ├── examples/                 # Rust examples
│   └── docs/                     # Rust-specific docs (contributing, code-of-conduct, crates-io)
├── Makefile                      # Build/release commands
└── README.md
```

## Architecture

Three core components:

### 1. OauthClient
- Basic Auth (base64-encoded `username:api_key`)
- Methods: `get_scopes`, `create_token`, `get_tokens`, `delete_token`, `get_counters`
- Endpoints: `/scopes`, `/token`, `/token/{id}`, `/counters/{period}/{date}`

### 2. ApiClient (Client in Rust)
- Bearer token auth
- Generic `request(method, url, payload?, params?)` method
- Convenience methods: `get`, `post`, `put`, `delete`
- Supports all HTTP methods: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS

### 3. Type System
- Structs: `TokenResponse`, `ScopeInfo`, `CounterInfo`, `APIErrorResponse`, `SDKConfig`, `Credentials`, `HTTPResponse`, `RequestConfig`
- Enums: `Environment` (PRODUCTION, TEST), `HTTPMethod`
- Error classes: `HTTPError`, `ValueError`

## API Endpoints

### OAuth Endpoints
| Method | Endpoint                    | Description            |
|--------|-----------------------------|------------------------|
| GET    | `/scopes`                   | List available scopes  |
| POST   | `/token`                    | Create access token    |
| GET    | `/token`                    | List existing tokens   |
| DELETE | `/token/{id}`               | Delete a token         |
| GET    | `/counters/{period}/{date}` | Get usage statistics   |

### Environment URLs
- **Production**: `https://oauth.openapi.it`
- **Test**: `https://test.oauth.openapi.it`

## Makefile Commands

- `make dev-push` - Add, commit (prompts for message), push
- `make dev-version` - Show version from Cargo.toml
- `make publish` - Commit and push release
- `make release` - Commit, tag with SDK_VERSION, force-push tags

## Rust Reference Implementation

- Crate name: `openapi-sdk`
- Async runtime: Tokio
- HTTP client: reqwest (with blocking + json features)
- Serialization: serde + serde_json
- Auth encoding: base64
- Tests: `cargo test` (unit tests in lib.rs)
- Examples: `cargo run --example token_generation`, `cargo run --example api_calls`

## Key Implementation Requirements

- **Auth**: Basic auth for OAuth, Bearer for API calls
- **HTTP**: JSON serialization/deserialization, query parameter handling
- **Errors**: Distinguish 4xx (client) vs 5xx (server), include status code + body
- **Config**: Support test/production environments, configurable timeouts/retries
- **Env vars**: `OPENAPI_USERNAME`, `OPENAPI_API_KEY`, `OPENAPI_ACCESS_TOKEN`, `OPENAPI_TEST_MODE`, `OPENAPI_BASE_URL`
- **Defaults**: Timeout 30s, connection timeout 10s, max retries 3, retry delay 1s

## Testing Strategy

- **Unit tests**: Mock HTTP responses, test all methods, validate error handling
- **Integration tests**: Real API calls against test environment
- **Test data**: Standardized mock responses defined in test pseudocode files
- **Coverage target**: >90%

## Conventions

- Pseudocode files use `.pseudo` extension
- The `.gitignore` includes `reference/` (Rust reference is a separate subproject)
- The root `.gitignore` is Laravel-flavored (historical)
- Each language SDK should follow its own naming conventions while preserving the API surface

---
> Source: [openapi/openapi-generic-sdk](https://github.com/openapi/openapi-generic-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->
