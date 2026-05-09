## agentmask

> agentmask is an open-source secrets firewall for AI coding agents. It prevents Claude Code, Cursor, and other AI assistants from reading, leaking, or committing secrets like API keys, tokens, passwords, and connection strings.

# agentmask — Contributor Context

## What This Is

agentmask is an open-source secrets firewall for AI coding agents. It prevents Claude Code, Cursor, and other AI assistants from reading, leaking, or committing secrets like API keys, tokens, passwords, and connection strings.

**agentmask's value is the IDE integration layer** — hooks, blocklist, MCP server, behavioral rules. Currently supports Claude Code and Cursor with auto-detection. Primary detection is delegated to [gitleaks](https://github.com/gitleaks/gitleaks) (150+ battle-tested rules). An **agentmask scanner** (`src/scanner/tier2.ts`) runs as a second pass to catch patterns gitleaks's `generic-api-key` rule deliberately excludes: `password`/`passwd`/`pwd` field assignments, connection strings with embedded credentials, and provider prefixes without dedicated gitleaks rules (`whsec_`, `GOCSPX-`).

## Architecture

```
agentmask = IDE integration layer (Claude Code + Cursor)
  ├── Scanner backend: gitleaks (150+ rules, auto-downloaded if not installed)
  ├── agentmask scanner: complementary TS regex pass (password fields, conn strings, whsec_, GOCSPX-)
  ├── Blocklist manager (.agentmask/blocklist.json, shared between IDEs)
  ├── IDE adapters (src/ide/targets.ts — hook format, rules format, config paths per IDE)
  ├── Hooks (pre-read, pre-write, pre-bash, post-scan) with --format adapter
  ├── MCP server (safe_read, env_names, scan_file, scan_staged)
  └── Behavioral rules (per-IDE: .claude/rules/*.md, .cursor/rules/*.mdc)
```

### Three Reinforcing Layers

```
Layer 1: BLOCK (PreToolUse hooks)
  → Pre-read: static path patterns + dynamic blocklist lookup (no subprocess, <5ms)
  → Pre-write: gitleaks + agentmask scan on content via temp file (~200ms)
  → Pre-bash: pattern match on commands + gitleaks scan on staged files for git commit
  → Post-scan: gitleaks + agentmask scan on tool output, warns + auto-adds to blocklist

Layer 2: REDIRECT (MCP server)
  → safe_read: reads file, uses merged gitleaks + agentmask findings to redact secrets
  → env_names: lists .env variable names without values
  → scan_file / scan_staged: explicit scans (scan_file also runs agentmask scanner)

Layer 3: INSTRUCT (.claude/rules/agentmask.md)
  → Behavioral rules telling the agent to prefer safe_read
  → Installed automatically by `agentmask init`
```

## The Blocklist System

The key innovation is the **dynamic blocklist** (`.agentmask/blocklist.json`):

1. `agentmask init` runs `gitleaks dir .` on the entire repo, then runs the agentmask scanner on the same tree, and merges the findings
2. Every file containing a detected secret (from either scanner) is added to the blocklist
3. Pre-read hook checks the blocklist on every Read call — blocked files never enter context
4. Post-scan hook catches secrets in files NOT in the blocklist (new/modified files) and auto-adds them
5. First read of a new secret file still leaks (unavoidable), but every subsequent read is blocked
6. `allow-path` removes entries from the blocklist (after secrets are fixed)
7. Re-running `agentmask init` rescans and rebuilds the blocklist

## Project Structure

```
src/
├── cli.ts                  # CLI entrypoint — Commander.js, 8 commands
├── cli/
│   ├── scan.ts             # `agentmask scan` — gitleaks + agentmask scanner, merged output
│   ├── init.ts             # `agentmask init` — gitleaks + agentmask scan + blocklist + hooks + MCP + rules
│   ├── remove.ts           # `agentmask remove` — clean uninstall including blocklist
│   └── allowlist.ts        # `allow-path` (also removes from blocklist), `allow-value`
├── gitleaks/
│   ├── binary.ts           # Find system gitleaks or auto-download from GitHub releases
│   ├── runner.ts           # Subprocess wrapper: scanDir, scanFile, scanContent, scanStaged
│   └── index.ts            # Barrel export
├── scanner/
│   ├── file-patterns.ts    # Static blocked path globs (.env, *.pem, etc.), binary detection
│   └── tier2.ts            # agentmask scanner: password fields, conn strings, whsec_, GOCSPX-
├── ide/
│   └── targets.ts          # IDE adapters: Claude Code + Cursor (hook format, rules, config paths, detection)
├── hooks/
│   ├── common.ts           # Hook I/O: readStdin, block(), allow(), --format adapter, safety timer
│   ├── blocklist.ts        # Dynamic blocklist (.agentmask/blocklist.json): load, save, query, migrate
│   ├── pre-read.ts         # Static patterns + blocklist lookup → block or allow (no subprocess)
│   ├── pre-bash.ts         # Command pattern match + gitleaks scanStaged on git commit
│   ├── pre-write.ts        # gitleaks + agentmask scanContent on content being written
│   └── post-scan.ts        # gitleaks + agentmask scanContent on tool output, warns + auto-blocklists
├── mcp/
│   └── server.ts           # MCP server with 4 tools (uses @modelcontextprotocol/sdk)
└── config/
    ├── loader.ts           # .agentmask.toml config loader + merging
    └── index.ts
```

## Key Design Decisions

- **gitleaks as the primary scanner** — we don't reinvent detection. gitleaks has 150+ battle-tested rules. Auto-downloaded if not installed.
- **agentmask scanner for gitleaks gaps** — pure TypeScript regex pass (`src/scanner/tier2.ts`) that runs alongside gitleaks wherever gitleaks runs (init, scan, safe_read, scan_file, pre-write, post-scan). Catches `password`/`passwd`/`pwd` fields, connection strings with embedded creds, `whsec_`, and `GOCSPX-` — all patterns the gitleaks `generic-api-key` rule deliberately excludes. Runs in-process (<50ms), unconditional, no config. Merged into the same `GitleaksFinding[]` shape via `mergeFindings()` so downstream code is unchanged.
- **TypeScript ESM** — same ecosystem as Claude Code and the MCP SDK
- **Graceful degradation** — hook crashes → exit 1 (allow), NEVER exit 2 (block). gitleaks subprocess failure → allow. A bug must never block the user's work.
- **Pre-read is pure blocklist lookup** — no subprocess, no gitleaks call. <5ms. The blocklist was built at init time.
- **Pre-write/post-scan shell out to gitleaks** — ~200ms per call, writes content to temp file, scans, cleans up
- **Post-scan warns AND auto-blocklists** — first read of a new secret file leaks (unavoidable), every subsequent read is blocked
- **Pre-write hook skips .env files** — they're expected to contain secrets
- **4-second safety timeout** on every hook (Claude Code's limit is 5s)

## Gitleaks Integration

### Binary Management (`src/gitleaks/binary.ts`)
1. Checks system PATH for `gitleaks`
2. Checks cache at `~/.agentmask/bin/gitleaks`
3. If neither found, downloads pinned version from GitHub releases
4. Supports macOS (arm64/x64) and Linux (arm64/x64)

### Runner (`src/gitleaks/runner.ts`)
All functions return `GitleaksFinding[]` (parsed from gitleaks JSON output):
- `scanDir(path)` — scans a directory
- `scanFile(path)` — scans a single file
- `scanContent(content, filename?)` — writes to temp file, scans, cleans up
- `scanStaged(cwd)` — runs `gitleaks git --staged`

### Gitleaks JSON Output Format
```json
{
  "RuleID": "stripe-access-token",
  "Description": "Found a Stripe Access Token...",
  "StartLine": 4,
  "EndLine": 4,
  "StartColumn": 13,
  "EndColumn": 44,
  "Match": "sk_live_...",
  "Secret": "sk_live_...",
  "File": "/path/to/file.ts",
  "Entropy": 4.601410,
  "Fingerprint": "..."
}
```

## Build & Test

```bash
npm install                         # install dependencies
npm run build                       # or: node node_modules/.bin/tsup
npm test                            # or: node node_modules/.bin/vitest run
```

Requires gitleaks installed (`brew install gitleaks`) for tests that involve scanning.

Build output goes to `dist/`. The CLI entrypoint is `dist/cli.js`.

## Hook Protocol (Claude Code)

Hooks receive JSON on stdin:
```json
{
  "tool_name": "Read",
  "tool_input": { "file_path": ".env" },
  "cwd": "/path/to/project",
  "tool_response": "..." // only for PostToolUse
}
```

Exit codes:
- **0** — allow the operation. Stdout parsed as JSON for `hookSpecificOutput`.
- **2** — block the operation. Stderr shown to Claude as error message.
- **1** (or any other) — non-blocking warning. Operation proceeds.

## Config Format

`.agentmask.toml` at project root:
```toml
[scan]
blocked_paths = [".env.custom"]

[[allowlists]]
paths = ["tests/**"]
description = "Test files"

[[allowlists]]
stopwords = ["EXAMPLE_KEY"]
```

## Test Fixtures

`tests/fixtures/` contains intentional fake secrets for testing detection. These are allowlisted in `.gitleaks.toml`. When adding new test fixtures with secret-format strings, add the path to the gitleaks allowlist AND unblock on GitHub push protection.

## What's NOT Built Yet

- Copilot / Windsurf / other IDE support
- CI/CD GitHub Actions workflow
- npm publish automation
- SessionStart hook to auto-rescan on new sessions
- HTTP hook mode (hooks call running MCP server instead of spawning subprocess)

---
> Source: [adithyan-ak/agentmask](https://github.com/adithyan-ak/agentmask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-25 -->
