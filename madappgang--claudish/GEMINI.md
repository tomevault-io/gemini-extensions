## claudish

> Handles auth, endpoints, headers, rate limiting. Optionally overrides stream format for aggregators.

# Claudish - Development Notes

## Release Process

**Releases are handled by CI/CD** - do NOT manually run `npm publish`.

1. Bump version in `package.json`
2. Commit with conventional commit message (e.g., `feat!: v3.0.0 - description`)
3. Create annotated tag: `git tag -a v3.0.0 -m "message"`
4. Push with tags: `git push origin main --tags`
5. CI/CD will automatically publish to npm

## Build Commands

- `bun run build` - Full build (extracts models + bundles)
- `bun run build:ci` - CI build (bundles only, no model extraction)
- `bun run dev` - Development mode

## Model Routing (v4.0+)

### New Syntax: `provider@model[:concurrency]`

```bash
# Explicit provider routing
claudish --model google@gemini-2.0-flash "task"
claudish --model openrouter@deepseek/deepseek-r1 "task"

# Native auto-detection (no prefix needed)
claudish --model gpt-4o "task"          # → OpenAI
claudish --model gemini-2.0-flash "task" # → Google
claudish --model llama-3.1-70b "task"   # → OllamaCloud

# Local models with concurrency
claudish --model ollama@llama3.2:3 "task"  # 3 concurrent requests
```

### Provider Shortcuts
- `g@`, `google@` → Google Gemini
- `oai@` → OpenAI Direct
- `cx@`, `codex@` → OpenAI Codex (Responses API)
- `or@`, `openrouter@` → OpenRouter
- `mm@`, `mmax@` → MiniMax
- `mmc@` → MiniMax Coding Plan
- `kimi@`, `moon@` → Kimi
- `glm@`, `zhipu@` → GLM
- `gc@` → GLM Coding Plan
- `llama@`, `oc@` → OllamaCloud
- `litellm@`, `ll@` → LiteLLM (requires LITELLM_BASE_URL)
- `ollama@` → Ollama (local)
- `lmstudio@` → LM Studio (local)
- Custom endpoint names also work as provider prefixes (e.g., `my-vllm@model-name`) — see "Custom Endpoints" below

### Default Provider Configuration (v7.0.0+)

The default provider for auto-routing is configurable. Set it via:

- **Config file**: `"defaultProvider": "openrouter"` in `~/.claudish/config.json`
- **Env var**: `CLAUDISH_DEFAULT_PROVIDER=litellm`
- **CLI flag**: `claudish --default-provider google "task"`

**Precedence** (highest to lowest):
1. CLI flag `--default-provider`
2. `CLAUDISH_DEFAULT_PROVIDER` env var
3. `defaultProvider` in config file
4. Legacy LITELLM auto-promotion (if `LITELLM_BASE_URL` + `LITELLM_API_KEY` set without explicit `defaultProvider`)
5. `OPENROUTER_API_KEY` present → OpenRouter
6. Hardcoded `"openrouter"`

**Example config**:
```json
{
  "defaultProvider": "litellm",
  "customEndpoints": { ... }
}
```

Valid values: any built-in provider name (`"openrouter"`, `"litellm"`, `"openai"`, `"anthropic"`, `"google"`) or a custom endpoint name defined in `customEndpoints`.

**Interaction with routing rules**: When `defaultProvider` is set and no explicit `routing["*"]` catch-all exists, Claudish synthesizes `routing["*"] = [defaultProvider]` at config load time. An explicit `routing["*"]` always wins.

**Legacy behavior**: If `LITELLM_BASE_URL` and `LITELLM_API_KEY` are set but `defaultProvider` is absent, LiteLLM is still promoted to first in the fallback chain. Claudish emits a one-shot stderr hint suggesting you set `defaultProvider` explicitly.

### Vendor Prefix Auto-Resolution (ModelCatalogResolver)

API aggregators (OpenRouter, LiteLLM) require vendor-prefixed model names that users shouldn't need to know. The `ModelCatalogResolver` interface searches each aggregator's dynamic model catalog to find the correct prefix automatically.

**How it works**: User types bare model name → resolver searches the provider's already-fetched model list → finds the exact match with vendor prefix → sends the prefixed name to the API.

**Current resolvers**:
- **OpenRouter**: `or@qwen3-coder-next` → searches catalog → sends `qwen/qwen3-coder-next`
- **LiteLLM**: `ll@gpt-4o` → searches model groups → finds `openai/gpt-4o` (prefix-strip match)
- **Static fallback**: `OPENROUTER_VENDOR_MAP` for cold starts when catalog isn't loaded yet

**Key design rules**:
- Exact match only — no fuzzy/normalized matching. Find the right prefix, don't guess the model.
- Dynamic catalogs (from provider APIs) are PRIMARY. Static map is cold-start fallback only.
- Resolution happens BEFORE handler construction (in `proxy-server.ts`), not inside adapters.
- Sync entry point (`resolveModelNameSync()`) — uses in-memory caches + `readFileSync`, no async propagation.

**Firebase slim catalog** (v7.0.0+): The `aggregators[]` field on model documents provides a typed multi-provider routing index. Each entry is `{ provider, externalId, confidence }`. CLI consumers can look up `provider → externalId` directly instead of walking the `sources` array. The catalog backend lives in the [models-index](https://github.com/MadAppGang/models-index) repo.

**Adding a new aggregator resolver**: Implement `ModelCatalogResolver` interface in `providers/catalog-resolvers/`, register in `model-catalog-resolver.ts`. No changes to proxy-server or provider-resolver needed.

**Architecture doc**: `ai-docs/sessions/dev-arch-20260305-104836-a48a463d/architecture.md`

## Local Model Support

Claudish supports local models via:
- **Ollama**: `claudish --model ollama@llama3.2` (or `ollama@llama3.2:3` for concurrency)
- **LM Studio**: `claudish --model lmstudio@model-name`
- **Custom URLs**: `claudish --model http://localhost:11434/model`

### Context Tracking for Local Models

Local model APIs (LM Studio, Ollama) report `prompt_tokens` as the **full conversation context** each request, not incremental tokens. The `writeTokenFile` function uses assignment (`=`) not accumulation (`+=`) for input tokens to handle this correctly.

## Custom Endpoints (v7.0.0+)

Define named custom endpoints in `~/.claudish/config.json` under the `customEndpoints` key. Each endpoint registers as a provider prefix usable with `@` syntax.

### Config schema

**Simple endpoint** (most common):
```json
{
  "customEndpoints": {
    "my-vllm": {
      "kind": "simple",
      "url": "http://gpu-box:8000",
      "format": "openai",
      "apiKey": "${VLLM_API_KEY}",
      "modelPrefix": "my-org/",
      "models": ["llama3.1-70b", "qwen2.5-72b"]
    }
  }
}
```

**Complex endpoint** (full control):
```json
{
  "customEndpoints": {
    "corp-proxy": {
      "kind": "complex",
      "displayName": "Corporate LLM Proxy",
      "transport": "openai",
      "baseUrl": "https://llm.corp.internal",
      "apiPath": "/api/v2/chat/completions",
      "apiKey": "${CORP_LLM_KEY}",
      "authScheme": "X-Api-Key",
      "headers": { "X-Team": "platform" },
      "streamFormat": "openai-sse",
      "modelPrefix": "",
      "models": ["gpt-4o", "claude-sonnet"]
    }
  }
}
```

Use as: `claudish --model my-vllm@llama3.1-70b "task"` or `claudish --model corp-proxy@gpt-4o "task"`.

### Key details

- **`${VAR_NAME}` expansion**: The `apiKey` field expands environment variables at startup. Use this instead of hardcoding secrets in config.
- **Zod validation**: Claudish validates all custom endpoints at proxy startup. Invalid entries emit a stderr warning and are skipped — they don't crash the proxy.
- **Runtime registration**: Endpoints call `registerRuntimeProvider()` and `registerRuntimeProfile()` to inject themselves into the provider resolver and transport layers.
- **`models` field** (optional): When present, limits the endpoint to listed models. Omit to allow any model name.
- **`modelPrefix` field** (optional): Prepended to the user-specified model name before sending to the API.

## Three-Layer Adapter Architecture (v5.14.0+)

The translation pipeline has three decoupled layers:

### Layer 1: FormatConverter — wire format translation
Translates between Claude API format and target model's wire format (messages, tools, payload).
Each converter declares its stream format via `getStreamFormat()`.
- **Interface**: `adapters/format-converter.ts`
- **Implementations**: OpenAIAdapter, AnthropicPassthroughAdapter, GeminiAdapter, CodexAdapter, OllamaCloudAdapter, LiteLLMAdapter
- **Message/tool conversion**: `handlers/shared/format/openai-messages.ts`, `openai-tools.ts`

### Layer 2: ModelTranslator — model dialect translation
Translates model-specific dialect differences (context windows, thinking→reasoning_effort, vision rules).
- **Interface**: `adapters/model-translator.ts`
- **Implementations**: GLMAdapter, GrokAdapter, MiniMaxAdapter, DeepSeekAdapter, QwenAdapter, CodexAdapter
- **Selection**: `AdapterManager` auto-selects based on model ID

### Layer 3: ProviderTransport — HTTP transport
Handles auth, endpoints, headers, rate limiting. Optionally overrides stream format for aggregators.
- **Interface**: `providers/transport/types.ts`
- **Stream format override**: LiteLLM and OpenRouter implement `overrideStreamFormat()` → `"openai-sse"`

### Composition in ComposedHandler
```
ComposedHandler = FormatConverter (explicit adapter) + ModelTranslator (auto-selected) + ProviderTransport
```

**Stream parser selection** (3-tier priority):
```typescript
transport.overrideStreamFormat() ?? modelAdapter.getStreamFormat() ?? providerAdapter.getStreamFormat()
```

**Adding a new provider**: Add one entry to `PROVIDER_PROFILES` table in `providers/provider-profiles.ts`.
**Adding a new model**: Create a ModelTranslator adapter, register in `adapters/adapter-manager.ts`.
**Verifying wiring**: `claudish --probe <model>` shows the full adapter composition.

### Stream Parsers
Located in `handlers/shared/stream-parsers/`:
- `openai-sse.ts` — OpenAI SSE → Claude SSE (used by most providers)
- `anthropic-sse.ts` — Anthropic SSE passthrough (MiniMax, Kimi direct)
- `gemini-sse.ts` — Gemini SSE → Claude SSE
- `ollama-jsonl.ts` — Ollama JSONL → Claude SSE
- `openai-responses-sse.ts` — OpenAI Responses API → Claude SSE (Codex)

## Debug Logging

Debug logging is behind the `--debug` flag and outputs to `logs/` directory. It's disabled by default.
Keep full debug logging (including empty chunks, raw deltas) in log files — needed to understand real model streaming behavior. Suppress noise at the registration/initialization level (e.g., conditional middleware), not at the streaming data level.

### Raw SSE Capture (v5.14.0+)

When `--debug` is active, both stream parsers log raw SSE events:
- `[SSE:openai] {...}` — every OpenAI SSE data line
- `[SSE:anthropic] {...}` — every Anthropic SSE data line

These are greppable and extractable into test fixtures for regression testing.

## Debugging Failed Model Translations

When a model produces wrong output (0 bytes, garbled, wrong format), use this workflow:

### 1. Reproduce with --debug
```bash
claudish --model minimax-m2.5 --debug "say hello"
# Debug log written to logs/claudish_YYYY-MM-DD_HH-MM-SS.log
```

### 2. Verify wiring with --probe
```bash
claudish --probe minimax-m2.5
# Shows: transport, format adapter, model translator, stream format, overrides
```

### 3. Analyze the debug log
Use the `/debug-logs` slash command in Claude Code:
```
/debug-logs logs/claudish_2026-03-17_09-41-32.log
```

This command:
1. Reads the log and counts text chunks, tool calls, HTTP errors, fallback chains
2. Diagnoses the failure mode (no SSE content, text but 0 stdout, wrong parser, etc.)
3. Extracts SSE fixtures from `[SSE:*]` lines using `test-fixtures/extract-sse-from-log.ts`
4. Adds a regression test to `format-translation.test.ts`
5. Runs tests to confirm the regression is captured

### 4. Extract fixtures manually (alternative)
```bash
bun run packages/cli/src/test-fixtures/extract-sse-from-log.ts logs/claudish_*.log
# Creates: test-fixtures/sse-responses/<model>-<format>-turn<N>.sse
```

### 5. Run format translation tests
```bash
bun test packages/cli/src/format-translation.test.ts
```

## Channel Mode (v6.4.0+)

The MCP server supports a channel mode that enables async model sessions with push notifications.

### Architecture

Uses the low-level `Server` class (not `McpServer`) from `@modelcontextprotocol/sdk/server/index.js` to declare `experimental: { 'claude/channel': {} }` capability. The SDK's `assertNotificationCapability()` has no default case — custom notification methods like `notifications/claude/channel` pass through.

### Components (`packages/cli/src/channel/`)

- **SessionManager** — spawns `claudish --model X --stdin --quiet` child processes, tracks lifecycle, enforces timeouts
- **SignalWatcher** — per-session state machine (starting→running→tool_executing→waiting_for_input→completed/failed/cancelled)
- **ScrollbackBuffer** — in-memory ring buffer (2000 lines) for session output

### MCP Tools (11 total)

- **Low-level** (4): `run_prompt`, `list_models`, `search_models`, `compare_models`
- **Agentic** (2): `team`, `report_error`
- **Channel** (5): `create_session`, `send_input`, `get_output`, `cancel_session`, `list_sessions`

Tool gating via `CLAUDISH_MCP_TOOLS` env var: `all` (default), `low-level`, `agentic`, `channel`.

### Tool Registration Pattern

Uses a `ToolDefinition[]` registry with raw JSON Schema (not Zod). Two `setRequestHandler` calls replace McpServer's ergonomic API:
- `ListToolsRequestSchema` → returns filtered tool list
- `CallToolRequestSchema` → dispatches to handler by name

### Channel Notifications

`server.notification({ method: "notifications/claude/channel", params: { content, meta } })` — pushed by SessionManager's `onStateChange` callback on state transitions.

### Testing

```bash
bun test --cwd . ./packages/cli/src/channel/*.test.ts
```

59 tests across 4 files: scrollback-buffer (11), signal-watcher (12), session-manager (21), e2e-channel (15).

E2E tests use `--strict-mcp-config --bare --dangerously-skip-permissions` for isolation. SessionManager tests use a fake-claudish PATH shim (`channel/test-helpers/fake-claudish.ts`).

## Test Infrastructure

### Format Translation Test Harness
`packages/cli/src/format-translation.test.ts` — SSE replay tests for the full translation pipeline.

**Fixture-based**: Each `.sse` file in `test-fixtures/sse-responses/` is a captured SSE stream from a real provider response. Tests replay fixtures through the stream parser and assert correct Claude SSE output.

**Helpers**: `parseClaudeSseStream()`, `extractText()`, `extractToolNames()`, `extractStopReason()`, `fixtureToResponse()`

**Adding regression tests**: After extracting fixtures from a debug log, add a `describe("Regression: <model>")` block. Template is at the bottom of the test file.

## Version Bumping Checklist

When releasing a new version, update ALL of these locations:
1. `package.json` (root monorepo version)
2. `packages/cli/package.json` (npm-published package - **CI/CD publishes from here**)
3. `packages/cli/src/version.ts` (fallback VERSION constant — moved from cli.ts in v7.0.0)

The fallback VERSION in version.ts ensures compiled binaries (Homebrew, standalone) display the correct version when package.json isn't available. The `packages/cli/package.json` version is what npm publishes - if it's not updated, npm publish will fail.

## Learned Preferences

### Tools & Commands
<!-- learned: 2026-03-28 session: 03cd7cc5 source: repeated_pattern -->
- Use `bun` for all package management and scripts (`bun run build`, `bun test`, etc.) — not npm or yarn
<!-- learned: 2026-04-06 session: df311293 source: repeated_pattern -->
- Use Grep/grep tool for code investigation instead of mnemex — prefer built-in search tools during investigation phases

### Workflow
<!-- learned: 2026-04-06 session: df311293 source: explicit_rule -->
- Don't run claudish directly in main bash — use dedicated channel sessions or `/delegate`

---
> Source: [MadAppGang/claudish](https://github.com/MadAppGang/claudish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-20 -->
