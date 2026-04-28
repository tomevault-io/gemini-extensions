## pomodoro-timer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Aesthetic Pomodoro timer app — desktop-only, anonymous-by-default, optionally synced to Supabase. Reference design: studywithme.io. See `PRD.md` for full spec and phased execution plan.

## Stack

- **Next.js 15** (App Router) + **TypeScript** + **Tailwind CSS**
- **FastAPI** (Python) for cloud API routes, proxied via Next.js rewrites at `/api/py/*`
- **Supabase** — auth, Postgres, Row-Level Security
- **pnpm** as the package manager
- **Space Grotesk** via `@fontsource/space-grotesk`

## Commands

```bash
pnpm dev          # start Next.js dev server (localhost:3000) + FastAPI dev server (port 8000)
pnpm build        # production build
pnpm typecheck    # tsc --noEmit
pnpm lint         # ESLint

# FastAPI (run separately or via a Procfile/concurrently setup)
uvicorn api.index:app --reload --port 8000
```

**Parallel-safe (run together):**
- `pnpm typecheck` + `pnpm lint` — independent static checks, no shared output
- `pnpm dev` + `uvicorn api.index:app --reload --port 8000` — Next.js and FastAPI are separate processes

**Must run sequentially:**
- `pnpm build` then deploy — build must finish before any deploy step
- Supabase migrations before `pnpm dev` — schema must exist before the app reads it

## Architecture

**Routing & API proxy.** One page: `app/page.tsx`. All API calls use `/api/py/*` — `next.config.ts` rewrites these to `http://127.0.0.1:8000/api/py/*` in dev. On Vercel, `api/*.py` files deploy as Python serverless functions.

**Timer engine.** `lib/timer/useTimer.ts` is drift-safe: on start it stores `endsAt = Date.now() + remainingMs` and recomputes `remaining = endsAt - Date.now()` every 250 ms via `requestAnimationFrame`. Do not use `setInterval` decrement.

**Sequence state machine.** Managed in `lib/timer/sequence.ts`. State is `{mode, pomosDoneInCycle}`. Transitions: `pom → short` (or `long` when `pomosDoneInCycle === 4`, then reset to 0); `short|long → pom`. Progress dots in `ModePills.tsx` read `pomosDoneInCycle`.

**Persistence.** Anonymous users: `localStorage` (shaped in `lib/storage/local.ts`). Signed-in users: Supabase tables `settings` and `todos` (see `supabase/schema.sql`). Sync logic with last-write-wins per field lives in `lib/storage/sync.ts`. On first sign-in, prompt user to keep cloud / overwrite / merge (default = merge).

**Themes.** Manifest in `lib/themes.ts` — four themes (seoul, tokyo, paris, fire), each with a `.webm` video and `.jpg` poster under `public/bg/`. `BackgroundLayer.tsx` handles crossfade. Videos must be `playsInline muted loop autoPlay`. Max 3 MB per compressed video.

**Side effects on session end.** `lib/audio.ts` — preloaded alert sounds from `public/sounds/`. `lib/notifications.ts` — Notification API; only request permission the first time a user explicitly enables notifications, not on page load.

**Tab title & favicon.** While running: `document.title = \`${mm}:${ss} · Pomodoro\``. Favicon swapped via `<canvas>` → dataURL → `<link rel="icon">` every second.

**Supabase clients.** Browser: `lib/supabase/client.ts`. Server (cookies): `lib/supabase/server.ts`. Session refresh: `middleware.ts` at repo root (wraps `lib/supabase/middleware.ts`).

## Quality gates

Before shipping: `pnpm typecheck`, `pnpm lint`, Lighthouse a11y ≥ 90, background video ≤ 3 MB compressed.

---
> Source: [aaf1007/pomodoro-timer](https://github.com/aaf1007/pomodoro-timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-19 -->
