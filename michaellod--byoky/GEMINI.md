## byoky

> Monorepo using pnpm workspaces:

# byoky

## Project structure

Monorepo using pnpm workspaces:

- `packages/core` — shared types, crypto (Web Crypto API), message protocol, provider registry
- `packages/sdk` — `@byoky/sdk` npm package for developers (includes `@byoky/sdk/server` for backend relay)
- `packages/extension` — browser extension built with WXT (Chrome, Firefox, Safari)
- `packages/bridge` — `@byoky/bridge` local HTTP proxy + native messaging host for CLI/desktop apps
- `packages/relay` — WebSocket relay server for mobile pairing, remote agents, and gift relay
- `packages/vault` — server-side vault (Hono on Railway). Owns the authoritative Token Pool at `GET /pool`, `POST /pool/list`, `POST /pool/unlist`; also hosts encrypted credential sync and gift links
- `packages/marketplace` — **legacy compatibility shim only**. Absorbs old-client POSTs to `marketplace.byoky.com/gifts` and forwards to the vault. Delete once older extensions/mobile clients have upgraded
- `packages/web` — `byoky.com` Next.js site: landing page, `/docs`, `/apps` app registry + admin, `/token-pool`, `/demo`, `/chat`, `/gift`, `/pair`. App submissions land in Postgres via `lib/apps-db.ts`; `middleware.ts` rewrites `api.byoky.com/v1/apps/*` onto `/api/apps/*`
- `packages/ios` — iOS app (SwiftUI wallet + Safari extension)
- `packages/android` — Android app (Kotlin/Compose standalone wallet)
- `packages/openclaw-plugin` — OpenClaw provider plugin that routes through `@byoky/bridge`
- `packages/create-byoky-app` — `npx create-byoky-app` scaffolder; `submit` command POSTs to `api.byoky.com/v1/apps/submit`

## Development

- `pnpm install` — install all dependencies
- `pnpm dev` — start extension in Chrome dev mode
- `pnpm build` — build all packages
- `pnpm typecheck` — type check all packages

## Architecture

- **Proxy model**: API keys never leave the extension. Background script proxies all LLM API calls.
- **Custom fetch**: SDK provides `createFetch()` that routes requests through the extension via `window.postMessage` → content script → `chrome.runtime.Port` → background script.
- **Encryption**: AES-256-GCM with PBKDF2 key derivation (600K iterations), using Web Crypto API.
- **State**: Zustand in the popup, `browser.storage.local` for persistence.
- **Streaming**: Uses `chrome.runtime.Port` for long-lived connections; chunks forwarded via `TransformStream`.

## Conventions

- TypeScript strict mode
- WXT convention for entrypoints (`entrypoints/` directory)
- React functional components in popup
- No default exports except where WXT requires them

---
> Source: [MichaelLod/byoky](https://github.com/MichaelLod/byoky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-18 -->
