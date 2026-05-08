## amoxsql

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# AmoxSQL — Project Guide

## Overview
**AmoxSQL** is a desktop SQL IDE for local data analysis, built with Electron + React + Express + DuckDB.
- **Version**: 2.0.1
- **Author**: Flavio Sandoval (@dsandovalflavio)
- **License**: AmoxSQL Community License (Source Available)
- **Tagline**: "The Modern Codex for Local Data Analysis"

## Runtime Topology (big picture)
Three processes cooperate at runtime:
1. **Electron main** ([electron/main.js](electron/main.js)) — owns the BrowserWindow, holds the single-instance lock, handles native dialogs / window controls via `ipcMain`, and **spawns the Express server as a `utilityProcess`** on port 3001.
2. **Express server** ([server/index.js](server/index.js)) — all REST endpoints + DuckDB connection. The renderer talks to it over HTTP, not IPC.
3. **Renderer (React)** — in dev, loaded from Vite on `http://localhost:5173`; in prod, from `client/dist/`. The preload bridge ([electron/preload.js](electron/preload.js)) exposes only a narrow `window.electronAPI` for dialogs, shell-open, and window controls. Data calls go directly to `http://localhost:3001`.

When debugging "it works in dev but not in the built app," check (a) Vite dev server vs `client/dist` loading in `main.js`, and (b) whether the utility process spawned the server — server stdout is piped through main.

## Architecture

```
Electron Shell
├── electron/main.js          — Main process, IPC, window management
├── electron/preload.js       — Context bridge (electronAPI)
├── client/                   — React 19 SPA (Vite 7)
│   ├── src/App.jsx           — Root component, phase management
│   ├── src/components/       — 64+ components
│   └── src/utils/            — HTML report gen, notebook parser
└── server/                   — Express 5 backend (port 3001)
    ├── index.js              — 70+ REST endpoints (~2300 lines)
    ├── DatabaseManager.js    — DuckDB Neo API connection
    ├── AiManager.js          — AI provider abstraction
    └── ai/                   — AI subsystem (tools, prompts, memory, persistence)
```

## Tech Stack
| Layer     | Technology                                |
|-----------|-------------------------------------------|
| Desktop   | Electron 33                               |
| Frontend  | React 19.2, Vite 7.2, Monaco Editor 4.7  |
| Charts    | Recharts 3.7                              |
| Backend   | Express 5.2 (Node.js)                     |
| Database  | DuckDB (Neo API nativa @duckdb/node-api)  |
| AI SDK    | Vercel AI SDK 6.0 + Zod                   |
| AI        | Ollama (local), Google Gemini (cloud)     |
| Build     | electron-builder (NSIS/Windows)           |

## Key Commands
```bash
npm start              # Dev: concurrently runs Vite, waits on :5173, then launches Electron
npm run client:dev     # Frontend only (Vite on port 5173)
npm run client:build   # Production build → client/dist/
npm run dist           # client:build + electron-builder (NSIS installer, Windows)
```

**No test, lint, or typecheck scripts exist** in `package.json` — don't claim to have run them. If you change code, verify by running the app and exercising the affected UI path.

The `postinstall` hook runs `electron-builder install-app-deps` to rebuild native modules (notably `@duckdb/node-api`) against Electron's Node ABI. If DuckDB fails to load after `npm install`, re-run `npm run postinstall`.

## Project Structure — Key Files

### Frontend (client/src/components/)
- `App.jsx` (44KB) — Root, phases: WELCOME → SELECTING_DB → IDE
- `SqlEditor.jsx` (57KB) — Monaco editor, autocomplete, CTE debug
- `SqlNotebook.jsx` (20KB) — Notebook interface with cells
- `NotebookCell.jsx` (28KB) — Individual cell (SQL, Markdown, Input)
- `ResultsTable.jsx` (38KB) — Paginated results with sort/filter
- `DataVisualizer/` — 12+ chart types (Recharts)
- `DataProfiler.jsx` (30KB) — Statistical profiling
- `AiSidebar.jsx` (36KB) — AI chat assistant
- `SettingsModal.jsx` (97KB) — Full settings UI
- `LayoutManager.jsx` — Split-pane layout with tabs
- `EditorPane.jsx` — Editor container, detects .sqlnb for notebook mode
- `DatabaseExplorer.jsx` — Schema browser
- `FileExplorer.jsx` — Project file browser
- `DbtPanel.jsx` — DBT integration
- `ErDiagram.jsx` — ER diagram visualization
- `CommandPalette.jsx` — Ctrl+Shift+P quick actions

### Backend (server/)
- `index.js` (88KB) — All REST endpoints
- `DatabaseManager.js` (10KB) — DuckDB connection lifecycle
- `AiManager.js` (14KB) — Ollama/Gemini provider management
- `ai/agenticLoop.js` — Main tool-calling loop (drives multi-step AI turns)
- `ai/tools.js` — AI tool definitions (execute_sql, list_tables, describe_table, display_chart, suggest_followups)
- `ai/systemPrompt.js` — Dynamic prompt builder with schema context
- `ai/skills.js` — Skill definitions invoked by the agent
- `ai/modelProfiles.js` — Per-model capability / parameter profiles (Ollama + Gemini)
- `ai/promptOnlyMode.js` — Fallback path when the active model can't do tool-calling
- `ai/profiling.js` — Table/column profiling used as AI context
- `ai/compaction.js` — Context window token management
- `ai/persistence.js` — Conversation storage in DuckDB (`amoxsql_ai` schema)
- `ai/memory.js` — Cross-conversation memory extraction
- `ai/userRules.js` — `RULES.md` loader for custom per-project AI behavior
- `ai/testRunner.js` — Local test harness for AI flows
- `ai/_sqlHelpers.js` — Shared SQL utilities for tools

### Utilities
- `client/src/utils/notebookParser.js` — Parse/serialize .sqlnb files (JSON v2.0 + legacy marker format)
- `client/src/utils/generateHtmlReport.js` — Self-contained HTML report export with charts as PNG

## File Formats
- `.sql` — Plain SQL files
- `.sqlnb` — SQL Notebook (JSON v2.0 with cells array + environment)
- `.sqlnb.state.json` — Sidecar file for notebook visual state (results cache, chart configs)
- `.amoxvis` — Chart configuration files
- `RULES.md` — Per-project AI behavior rules

## State Management
- **No Redux/Zustand** — React Context (ToastProvider) + local useState + localStorage/sessionStorage
- **localStorage keys**: `amoxsql-theme`, `amoxsql-accent`, `amoxsql-editor-layout`, `amoxsql-editor-settings`, `amoxsql-sidebar-width`, `amoxsql-ui-zoom`
- **sessionStorage**: `amoxsql-open-tabs`

## API Server (port 3001)
- `/api/project/*` — Project management
- `/api/files/*` — File CRUD
- `/api/db/*` — Database operations (connect, query, schema, import)
- `/api/query` — Execute SQL
- `/api/profile` — Data profiling
- `/api/ai/*` — AI chat, conversations, config
- `/api/dbt/*` — DBT integration
- `/api/export-data` — Data export (CSV, Parquet, Excel)
- `/api/notebook-state` — Notebook sidecar state persistence
- `/api/snippets`, `/api/bookmarks` — User snippets and bookmarks
- `/api/settings/*` — Config and Ollama model management

## Conventions
- CSS Variables for theming (30+ tokens), dark/light themes
- Lazy loading for heavy modals (React.lazy)
- All AI tools use Zod schemas for validation
- DuckDB query history auto-tracked in `amox_query_history` table
- AI conversations persisted in `amoxsql_ai` schema within DuckDB
- Config stored at `~/.amoxsql/config.json`
- **Desktop-native mindset**: DuckDB is local and fast — do not reason about network latency, loading spinners, or caching like a web app. Call queries directly.
- **Do NOT introduce list/table virtualization** (e.g. `@tanstack/react-virtual`). Prior attempts hurt performance in this app; `ResultsTable` paginates instead.

---
> Source: [DSandovalFlavio/AmoxSQL](https://github.com/DSandovalFlavio/AmoxSQL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->
