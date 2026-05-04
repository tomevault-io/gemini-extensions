## scarab-hub

> This file helps humans and coding agents work effectively in **scarab-hub** (branded in the UI as **PoE Flipup**): a single-page **Path of Exile** tool for vendor-recipe profitability (“flipping”) and related views, including a **Scarab 3-to-1 simulation**.

# Agent / contributor guide

This file helps humans and coding agents work effectively in **scarab-hub** (branded in the UI as **PoE Flipup**): a single-page **Path of Exile** tool for vendor-recipe profitability (“flipping”) and related views, including a **Scarab 3-to-1 simulation**.

For product overview, scripts, and folder tree, see [README.md](./README.md).

## Tech stack

- **Build**: Vite 5 — `root: src`, static assets from `public/`, output `dist/`
- **App code**: Vanilla **ES modules** (no React/Vue)
- **Tests**: Vitest + jsdom (`vite.config.js` sets `test.environment`)

## Commands

| Command | Use |
|--------|-----|
| `npm install` | Install dependencies |
| `npm run dev` | Dev server (default port **5173**, see `vite.config.js`) |
| `npm run build` / `npm run preview` | Production bundle and local preview |
| `npm test` | Vitest |
| `npm run test:ui` / `npm run test:coverage` | Vitest UI / coverage |

## Where things live

| Area | Location |
|------|----------|
| Entry & orchestration | `src/main.js` (large; coordinates loading, routing, views) |
| HTML shell | `src/index.html` |
| Global styles | `src/styles/main.css` |
| UI pieces | `src/js/components/` |
| Category list/grid UIs | `src/js/views/` |
| Domain types & helpers | `src/js/models/` |
| Fetching, prices, league, routing, calculations | `src/js/services/` |
| Per-category grid behavior | `src/js/adapters/`, `src/js/config/gridConfig.js` |
| Shared utilities | `src/js/utils/` |
| Bundled JSON (items, price fallbacks) | `public/data/` |
| Images | `public/assets/` |
| Tests | `tests/unit/`, `tests/integration/`, `tests/performance/` |

## Architecture notes

- **Routing**: Hash-based (`#/…`); logic in `src/js/services/router.js`. Welcome is `#`; categories use paths like `#/category` and optional `#/category/simulation`.
- **Data**: Item definitions load from `public/data/items/`. League and prices use `https://data.poeatlas.app/` with fallbacks under `public/data/prices/` and browser caching (see README). Some categories use external weight/MLE data (e.g. poedata.dev) as documented in README.
- **New category work**: Usually touches `models/`, a `*CalculationService.js`, views (list + grid), optional `adapters/`, `gridConfig.js`, `navigation.js`, and wiring in `main.js`.

## Conventions for changes

- Prefer **small, focused diffs** that match existing patterns (imports, naming, DOM style).
- **Extend** existing services/components rather than duplicating logic.
- **`main.js`** is the central wiring layer; new features often add a thin hook there—avoid unrelated refactors when fixing something else.
- **Tests**: Add or update tests under `tests/` when behavior is non-trivial or regression-prone; mirror paths where possible (`tests/unit/services/…` for `src/js/services/…`).

## Legal

*Path of Exile* is a trademark of Grinding Gear Games. This project is an independent fan tool and is not affiliated with or endorsed by GGG.

---
> Source: [weo-soft/scarab-hub](https://github.com/weo-soft/scarab-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-01 -->
