## cairn

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Cairn — Agent Architecture Guide

## Stack summary

- **Electron + Next.js 16** (App Router, static export) — desktop app
- **Tailwind CSS v4** — all colours via CSS custom properties (`var(--token)`), never raw Tailwind colour names
- **Zustand** — domain slices in `src/store/slices/`; composed in `src/store/index.ts`
- **better-sqlite3** — dual ABI (Electron + pkg/Node 22); all DB access goes through IPC from the renderer
- **D3 v7.9.0** — all SVG analytics rendering

## Styling rules

- **All colours** must use CSS variables: `var(--background)`, `var(--accent)`, `var(--text-primary)`, etc.
- **Alpha variants**: `color-mix(in srgb, var(--token) X%, transparent)` — never hardcode `rgba()`
- **Font sizes**: use `rem`-based Tailwind classes (`text-xs`, `text-sm`, `text-[0.714rem]`, etc.). Never use `text-[Npx]` — pixel classes don't scale with the font size setting.
- **Font scaling**: `--font-scale` CSS variable is set on `<html>` inline by `applyFontScale()`. Root `font-size: calc(14px * var(--font-scale))`. SVG `fontSize` attributes must be multiplied by `useFontScale()` from `analyticsHooks.ts`.
- `mcp-server.ts` uses inlined SQL only (no `queries.ts` import) due to Node ABI boundary

## Build

```bash
npm run compile   # rebuilds dist-electron/ + dist-mcp/mcp-server.bundle.js
npx tsc --noEmit  # type-check (always run after changes)
```

esbuild is stricter than tsc — backticks inside template literals must be unescaped at the template level. Use `import * as z from "zod"` (not `import { z }`) in all Electron files.

## Views and navigation

| View | Key | Component |
|------|-----|-----------|
| Overview | `⌘1` | `ProjectOverview` |
| Notes | `⌘2` | `NotesView` |
| Board | `⌘3` | `KanbanBoard` |
| Idea Flow | `⌘4` | `IdeaFlowView` |
| Knowledge Graph | `⌘5` | `KnowledgeGraphView` — Force + Radial only |
| Insights | `⌘6` | `InsightsView` — all analytics canvases |
| Settings | — | `SettingsView` |

`activeView` union: `"overview" | "notes" | "board" | "flow" | "graph" | "insights" | "chat" | "search" | "settings"`

## Knowledge Graph vs Insights split

**KnowledgeGraphView** (`src/components/graph/KnowledgeGraphView.tsx`) — Force-directed and Radial tree layouts only. Reads from `graphData` store slice (populated by `loadGraph()`). `GraphLayoutMode = "force" | "radial"`.

**InsightsView** (`src/components/insights/InsightsView.tsx`) — hosts all seven analytics canvases. Also calls `loadGraph()` on mount (same as KGV) because canvases scope data via `useScopedData(nodes)` which needs `graphData.nodes` populated. Local `InsightsLayout` type — not stored in the global store.

## Analytics canvas architecture

All analytics canvases follow the same pattern:

```
InsightsView
  └── <XxxCanvas nodes={allNodes} ... />
        ├── useContainerDims(ref)     — ResizeObserver → { width, height }
        ├── useScopedData(nodes)      — derives activeProjects, scopedCards etc. from store
        ├── useFontScale()            — returns fontScale number for SVG fontSize scaling
        └── D3 / SVG rendering
```

Shared modules:
- `analyticsUtils.ts` — `PRIORITY_COLOR`, `CANVAS_PAD`, `truncateName`, `HOUR_MS`, `DAY_MS`, etc.
- `analyticsHooks.ts` — `useContainerDims`, `useScopedData`, `useFontScale`
- `AnalyticsShared.tsx` — `<CanvasEmptyState>`, `<CanvasTooltip>`, `<SvgTimeAxis>`
- `graphUtils.ts` — `resolveCssVar()` for canvas 2D context colour lookups

## Store slices

| Slice | File | Key exports |
|-------|------|-------------|
| UI | `slices/ui.ts` | `theme`, `setTheme`, `fontScale`, `setFontScale`, `activeView`, `setView`, `applyFontScale`, `applyTheme` |
| Workspace | `slices/workspace.ts` | `workspaces`, `projects`, `createProject`, `updateProject` |
| Board | `slices/board.ts` | `columns`, `cards`, `createCard`, `updateCard`, `moveCard` |
| Notes | `slices/notes.ts` | `notes`, `createNote`, `updateNote`, `deleteNote` |
| Tags | `slices/tags.ts` | `tags`, `createTag`, `updateTag` |
| Chat | `slices/chat.ts` | `threads`, `messages`, `sendMessage` |
| Graph | `slices/graph.ts` | `graphData`, `graphLoading`, `graphFilters`, `graphLayout`, `loadGraph`, `setGraphLayout`, `setGraphFilters` |
| Selectors | `slices/selectors.ts` | `getWorkspaceProjects`, `getProjectNotes`, `search` |

Hydration: `hydrate()` (web/localStorage) and `hydrateFromElectron()` both restore `theme` and `fontScale` from localStorage on startup.

## Font scale

```
FontScale = 1 | 1.1 | 1.2 | 1.3 | 1.4
DEFAULT_FONT_SCALE = 1.2  (M)
FONT_SCALE_KEY = "fontScale"  (localStorage key)
```

`applyFontScale(scale)` sets `document.documentElement.style.setProperty("--font-scale", scale)`. This overrides the CSS cascade `:root { --font-scale: 1 }` via inline style specificity.

## Data model (abbreviated)

```
Workspace
  └── Project
        ├── Note[]           (.md file + SQLite row)
        ├── Dashboard[]      (SQLite only, type="dashboard", content=HTML)
        ├── BoardColumn[]
        │     └── TaskCard[]
        ├── IdeaFlow
        │     ├── IdeaFlowNode[]
        │     └── IdeaFlowEdge[]
        └── ChatThread
              └── ChatMessage[]
```

Notes write to both `.md` files and SQLite simultaneously. Dashboards write to SQLite only (no `.md` file).

## Key constraints

- Never import from `queries.ts` in `mcp-server.ts` — it crosses the Node ABI boundary
- All DB writes from renderer go through `ipc()` / `ipcAwait()` to the Electron main process
- `graphData` is lazy — only populated when `loadGraph(activeWorkspaceId)` is called. Both `KnowledgeGraphView` and `InsightsView` call it on mount.
- D3 `fontSize` in SVG must always be multiplied by `useFontScale()` — never hardcode px values

## Project Management

Use cairn mcp server to manage notes, tasks and idea flow nodes.  
When you find anything interesting create a note in the right project and add a card to the board (if needed).

---
> Source: [ddutchie/cairn](https://github.com/ddutchie/cairn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-05 -->
