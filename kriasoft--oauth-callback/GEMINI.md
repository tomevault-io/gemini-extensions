## oauth-callback

> **ADRs** (`docs/adr/NNN-slug.md`): Architectural decisions (reference as ADR-NNN)

# OAuth Callback Project Guide

## Documentation

**ADRs** (`docs/adr/NNN-slug.md`): Architectural decisions (reference as ADR-NNN)  
**SPECs** (`docs/specs/slug.md`): Design specifications (reference as SPEC-slug)

## Project Structure

```bash
oauth-callback/
├── src/                     # Source code
│   ├── index.ts             # Main entry - exports getAuthCode(), OAuthError, mcp namespace
│   ├── mcp.ts               # MCP SDK exports - browserAuth(), storage, types
│   ├── server.ts            # HTTP server for OAuth callbacks
│   ├── errors.ts            # OAuthError class and error handling
│   ├── mcp-types.ts         # TypeScript interfaces for MCP integration
│   ├── auth/                # Authentication providers
│   │   ├── browser-auth.ts  # MCP SDK-compatible OAuth provider
│   │   └── browser-auth.test.ts
│   ├── storage/             # Token storage implementations
│   │   ├── memory.ts        # In-memory token store
│   │   └── file.ts          # Persistent file-based token store
│   └── utils/               # Utility functions
│       └── token.ts         # Token expiry calculations
│
├── templates/               # HTML templates for callback pages
│   ├── success.html         # Success page with animated checkmark
│   ├── error.html           # Error page for OAuth failures
│   └── build.ts             # Template compiler (bundles HTML into TypeScript)
│
├── examples/                # Usage examples
│   ├── demo.ts              # Interactive demo with mock OAuth server
│   ├── github.ts            # GitHub OAuth integration example
│   └── notion.ts            # Notion MCP with Dynamic Client Registration
│
├── dist/                    # Build output (generated)
│   ├── index.js             # Main bundle
│   ├── index.d.ts           # Main TypeScript declarations
│   ├── mcp.js               # MCP-specific bundle
│   ├── mcp.d.ts            # MCP TypeScript declarations
│   └── ...
│
├── package.json             # Project metadata and dependencies
├── tsconfig.json            # TypeScript configuration
├── README.md                # User documentation
└── CLAUDE.md                # This file - AI assistant context
```

## Module Organization

### Main Export (`oauth-callback`)

- `getAuthCode()` - Core OAuth authorization code capture
- `OAuthError` - OAuth-specific error class
- `mcp` namespace - Access to all MCP-specific functionality
- Storage implementations for backward compatibility

### MCP Export (`oauth-callback/mcp`)

- `browserAuth()` - MCP SDK-compatible OAuth provider
- `inMemoryStore()` - Ephemeral token storage
- `fileStore()` - Persistent file-based token storage
- Type exports: `BrowserAuthOptions`, `Tokens`, `TokenStore`, `ClientInfo`, `OAuthStore`

## Key Constraints

- Design Philosophy: Prioritize ideal design over backward compatibility
- Runtime: Always use Bun (not Node.js/NPM). Bun auto-loads .env files
- MCP SDK: OAuth/auth implementation in `node_modules/@modelcontextprotocol/sdk/dist/esm/client/auth.js`, `node_modules/@modelcontextprotocol/sdk/dist/esm/client/auth.d.ts`

---
> Source: [kriasoft/oauth-callback](https://github.com/kriasoft/oauth-callback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-06 -->
