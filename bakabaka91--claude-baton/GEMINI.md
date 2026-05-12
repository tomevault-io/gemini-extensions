## claude-baton

> Session lifecycle management for Claude Code — checkpoint, resume, and EOD summaries via local SQLite. See PLAN.md for full spec.

# claude-baton — Development Guide

## What is this?
Session lifecycle management for Claude Code — checkpoint, resume, and EOD summaries via local SQLite. See PLAN.md for full spec.

## Quick start
```bash
npm install
npm run build        # compile TypeScript
npm test             # run tests
npm run dev          # start MCP server (stdio)
```

## Project structure
- `src/index.ts` — MCP server entry, tool definitions
- `src/types.ts` — TypeScript interfaces for all data models
- `src/store.ts` — SQLite operations via sql.js (CRUD)
- `src/llm.ts` — claude -p wrapper
- `src/cli.ts` — CLI commands (setup, status, export, import, reset) + auto-checkpoint handler
- `src/utils.ts` — Utility helpers
- `prompts/` — Auto-checkpoint, daily summary prompt templates
- `commands/` — Slash command files (memo-checkpoint, memo-resume, memo-eod, memo-retro)
- `tests/` — Test suite
- `bin/claude-baton.js` — CLI entry point

## Non-negotiable constraints
1. **Never touch git branches** — no snapshots, no switching, no creating branches
2. **Single SQLite DB** — all projects in `~/.claude-baton/store.db`, never per-project files
3. **stdio transport** — standard MCP, not SSE
4. **Zero API keys** — all LLM calls via `claude -p`, never import anthropic SDK
5. **3 dependencies max** — `@modelcontextprotocol/sdk`, `sql.js`, `commander`
6. **sql.js pure WASM** — no native SQLite bindings, no better-sqlite3, no node-sqlite3

## Agent routing (mandatory)

When working on code, use the scoped agent for that domain. Never let one agent touch another's files.

| Work being done | Agent | Files owned |
|---|---|---|
| Data types, SQLite operations, utilities | `store` | `src/types.ts`, `src/store.ts`, `src/utils.ts` |
| MCP server, tool definitions, claude -p wrapper | `server` | `src/index.ts`, `src/llm.ts` |
| Prompts | `pipeline` | `prompts/` |
| CLI commands, bin entry point, slash commands | `cli` | `src/cli.ts`, `bin/`, `commands/` |
| Writing or updating tests | `test-gen` | `tests/` |

**Rules:**
- Every agent must load its required skills before writing code (listed in each agent's .md)
- After any agent writes code: run `/build` and `/test` in the main conversation
- Never trust agent output without verification — always run tests yourself

## Available commands

| Command | Purpose |
|---|---|
| `/build` | Compile TypeScript + format with Prettier |
| `/test` | Run test suite |
| `/lint` | Run ESLint + Prettier check |
| `/dev` | Start MCP server in dev mode |
| `/plan [description]` | Create implementation plan (no coding) |
| `/pr` | Pre-flight, test, lint, create PR |
| `/verify` | Run acceptance checklist from PLAN.md |

## Slash commands (shipped with product)

Installed to `~/.claude/commands/` during `claude-baton setup`.

| Command | Purpose |
|---|---|
| `/memo-checkpoint` | Save session state with git context and learnings before /compact or /clear |
| `/memo-resume` | Restore context from last checkpoint at session start |
| `/memo-eod` | End-of-day summary combining git activity with stored data |
| `/memo-retro` | Analyze checkpoint history, surface recurring learnings, propose memory updates |

## Releasing & versioning

- **Version source of truth**: `package.json` only. The CLI reads it at runtime — never hardcode versions elsewhere.
- **npm published**: package is `claude-baton` on npmjs.com, owned by `santoshus`.

**To release a new version:**
```bash
npm version patch     # or minor/major — bumps package.json, commits, tags
npm publish           # publishes to npm registry
git push && git push --tags   # push commit + tag to GitHub
```

**Rules:**
- Always run `/build` and `/test` before publishing
- Never publish with failing tests
- The `prepare` script runs `tsc` automatically before `npm publish`
- Use `npm version` to bump — never edit version in package.json manually

## Current state
v2.2.0 session lifecycle manager. 4 MCP tools, 4 slash commands, PreCompact auto-checkpoint hook, learnings capture, retro analysis, cost transparency. Published on npm.

---
> Source: [bakabaka91/claude-baton](https://github.com/bakabaka91/claude-baton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->
