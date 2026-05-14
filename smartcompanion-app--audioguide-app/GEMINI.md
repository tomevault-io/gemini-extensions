## audioguide-app

> SmartCompanion Audioguide App — a free, open-source PWA for interactive audioguide experiences in museums and cultural institutions. It runs in any modern mobile browser without installation and is fully offline-capable.

# CLAUDE.md

## Project Overview

SmartCompanion Audioguide App — a free, open-source PWA for interactive audioguide experiences in museums and cultural institutions. It runs in any modern mobile browser without installation and is fully offline-capable.

This repo is the **shell/wrapper application**. The bulk of functionality lives in workspace packages (`@smartcompanion/ui`, `@smartcompanion/services`, `@smartcompanion/data`, `@smartcompanion/native-audio-player`).

## Tech Stack

- **Stencil.js** v4 — Web Components framework (JSX with `h` factory, Preact-style)
- **Ionic Framework** v8 — UI components and routing
- **TypeScript** — Target ES2022, strict unused locals/params enabled
- **SCSS/SASS** — Styling with CSS custom properties
- **Workbox** v7 — Service worker and offline caching (conditional)
- **npm** — Package manager

## Common Commands

```bash
npm start          # Dev server with hot reload (--dev --watch --serve)
npm run build      # Production build → www/
npm test           # Run spec & e2e tests once
npm run test.watch # Continuous test watching
npm run generate   # Generate new Stencil component boilerplate
```

## Project Structure

```
src/
├── components/app-root/    # Single Stencil component (app shell)
│   ├── app-root.tsx        # Main component: routing, menu, navigation
│   └── app-root.css
├── global/
│   ├── app.ts              # Global initialization — applies dark mode via prefers-color-scheme
│   └── app.scss            # Global styles & CSS custom property (color) variables (light + dark)
├── services/
│   └── index.ts            # ServiceFacade initialization & export
├── assets/icon/            # PWA icons (favicon.ico, icon.png 512x512)
├── index.ts                # Entry point — imports Ionic & @smartcompanion/ui
├── index.html              # HTML shell
├── sw.js                   # Service worker (Workbox)
└── manifest.json           # PWA manifest
stencil.config.ts           # Build config & runtime environment variables
www/                        # Build output (gitignored)
```

## Architecture

### Component Architecture
- **Single Stencil component** (`app-root`) defined in this repo; all other UI components come from `@smartcompanion/ui`
- Stencil decorators: `@Component`, `@State` (no `@Prop` on app-root)
- Lifecycle: `componentDidLoad()` for initialization; `render()` returns JSX

### Routing
- Ionic Router with hash-based routes
- Routes: `/` (loading), `/language`, `/selection`, `/stations/:stationId`, `/pin`, `/error`
- Route guards via `serviceFacade.canLoadRoute()` in `beforeEnter`
- Route change listener on `/stations/default` to update reactive state

### Service Pattern
- **ServiceFacade** — single facade coordinates all services
- Imported from `src/services/index.ts` as `serviceFacade`
- Key methods: `__()` (i18n), `getRoutingService()`, `getMenuService()`, `canLoadRoute()`, `registerDefaultServices()`, `registerCollectibleAudioPlayerService()`, `registerOfflineLoadService()`, `registerOnlineLoadService()`

### State Management
- Stencil `@State()` for reactive component state
- Translation strings held as `@State` properties, updated on route changes

### Data Loading
- **Online mode**: `registerOnlineLoadService()` — fetches from `Env.DATA_URL`
- **Offline mode**: `registerOfflineLoadService()` — fetch with service worker caching
- Toggle via `OFFLINE_SUPPORT` in `stencil.config.ts`

### Dark Mode
- Uses Ionic's class-based dark palette (`@ionic/core/css/palettes/dark.class.css`)
- `src/global/app.ts` listens to `prefers-color-scheme: dark` and toggles `.ion-palette-dark` on `<html>`
- Color tokens defined twice in `src/global/app.scss`: light values on `:root`, dark overrides on `:root.ion-palette-dark`
- Menu logo: two `<img>` elements (`#main-menu-image-light`, `#main-menu-image-dark`); CSS toggles visibility — do not use `content: url(...)` on `<img>` (unreliable cross-browser)
- Embedders can force a palette via the `UPDATE_DARK_MODE` postMessage and swap logos via `UPDATE_MENU_IMAGE` with `{ light?, dark? }`

## Configuration & Customization

All runtime configuration lives in **`stencil.config.ts`** (rebuild required to apply changes):

| Variable | Default | Description |
|---|---|---|
| `Env.TITLE` | `"Animals"` | App title |
| `Env.DATA_URL` | GitHub sample JSON | External data source URL |
| `Env.OFFLINE_SUPPORT` | `"disabled"` | Enable service worker (`"enabled"`) |

**Customization points:**
1. **Colors**: SCSS variables in `src/global/app.scss` (CSS custom properties with `--sc-` prefix); each color has a light and a `*-dark` counterpart
2. **Title**: `index.html`, `manifest.json`, and `stencil.config.ts`
3. **Data URL**: `stencil.config.ts` → `DATA_URL`
4. **Offline**: `stencil.config.ts` → `OFFLINE_SUPPORT: "enabled"`
5. **Icons**: Replace files in `src/assets/icon/`
6. **Logo**: Replace `src/assets/logo.png` (light) and `src/assets/logo-dark.png` (dark)

## Code Conventions

- **Component tags**: kebab-case (`app-root`, `sc-page-*`)
- **Classes**: PascalCase
- **Functions/variables**: camelCase
- **CSS variables**: `--sc-` prefix, kebab-case
- **Imports**: Async/await throughout; no callbacks
- **JSX**: Stencil's `h` factory (not React's `createElement`)
- **Formatting**: Prettier with 180-char print width, single quotes, 2-space indent, LF line endings
- Navigation: always call `serviceFacade.getMenuService().close()` before navigating
- Hash navigation: check current hash before navigating to prevent duplicate routes

## Workspace Dependencies

Internal packages from the SmartCompanion monorepo — do not modify these directly from this repo:
- `@smartcompanion/ui` — UI components
- `@smartcompanion/services` — core services (routing, menu, i18n, data loading)
- `@smartcompanion/data` — data types and structures
- `@smartcompanion/native-audio-player` — audio playback

## Build Output

- Output: `www/` (gitignored)
- Generates ESM bundles + legacy JS
- Service worker generated conditionally based on `OFFLINE_SUPPORT`
- Static assets only — no backend required
- Deployable to GitHub Pages, Netlify, or any static host

---
> Source: [smartcompanion-app/audioguide-app](https://github.com/smartcompanion-app/audioguide-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-06 -->
