## audit-bugs

> Bug audit checklist — use when asked to review code for bugs, security issues, or quality problems


# Bug Audit Checklist

Run this checklist against any file or module being reviewed. Each item references a real bug found in this codebase.

## Audit Strategy

A single audit pass catches ~60-70% of real issues. Each pass has blind spots due to which files get read first, which patterns the search focuses on, and attention saturation on large codebases. To maximize coverage:

1. **Run multiple independent passes with different focus areas** — e.g., one pass for reactive patterns, one for auth/security, one for config consistency, one for the fork/rename story. Narrow scope finds more than broad sweeps.
2. **Vary the search strategy** — don't just follow this checklist. Grep for known anti-patterns (`return () =>` in `.tsx`, `_ =` in `.go`, `any` in component props). Read files the checklist doesn't mention (entry-server.tsx, rename tool, CI config, env files).
3. **Check reference implementations are textbook-perfect** — `app.tsx`, `login/index.tsx`, `dashboard/index.tsx`, and `settings/index.tsx` are copied by the scaffold tool and by AI assistants. A bug in these propagates everywhere.
4. **Verify the tool chain, not just the code** — rename tool, scaffold templates, deploy scripts, and CI config can have bugs that downstream users inherit silently.
5. **After fixing issues, re-read the fixed files** — fixes can introduce new problems (e.g., a rename tool step that corrupts URLs by naively replacing project names inside domain strings).

## Backend Service Files

- [ ] **SQL injection** — any `fmt.Sprintf` with values going into SQL? Must use `$N` placeholders.
- [ ] **Missing transactions** — any method with 2+ writes that isn't wrapped in `Begin/Commit`?
- [ ] **Silent error discards** — any `_ = pool.Exec(...)` or `_ = fn(...)`? Must log errors.
- [ ] **ErrNoRows masking** — any `_ = pool.QueryRow(...)` that ignores real DB errors? Must check `err != nil && err != pgx.ErrNoRows`.
- [ ] **Role-only auth** — any method that checks `userType` but not specific resource membership? Must use `verifyXAccess`.
- [ ] **Missing status guards** — can the method be called on inactive/completed/cancelled parent entities? Add guards.
- [ ] **echo.NewHTTPError usage** — should be `apperror.X()` instead.
- [ ] **TIMESTAMPTZ scan into *string** — nullable `TIMESTAMPTZ` columns CANNOT scan into `*string`. Must scan into `*time.Time` then format: `if t != nil { s := t.Format(time.RFC3339); field = &s }`.
- [ ] **ON CONFLICT without UNIQUE constraint** — any `ON CONFLICT (col1, col2)` requires a matching `UNIQUE` index on those columns. Without it, PostgreSQL returns a runtime error.
- [ ] **Hardcoded values in service constructors** — names, emails, URLs should come from config/env, not hardcoded strings.
- [ ] **Missing `rows.Err()` after `rows.Next()` loops** — partial iteration errors are silent without this check. Every `for rows.Next()` must be followed by `if err := rows.Err()`.

## Backend Handler Files

- [ ] **Missing requireUserID/requireUserType** — every protected handler must extract auth.
- [ ] **Business logic in handler** — DB queries or complex logic should be in the service layer.
- [ ] **Missing request validation** — required fields should be checked before calling service.

## Frontend Route Files

- [ ] **Nested `<Show>` for content states** — loading/error/empty/data using nested `<Show when={!loading()}>` → `<Show when={error()}>` → `<Show when={data()}>` chains? Must use flat `Switch/Match` instead. Nested `<Show>` creates stacked reactive scopes that leak during route transitions.
- [ ] **Missing `batch()` on async signal updates** — `setData(result)` and `setLoading(false)` after an `await` must be wrapped in `batch()` for atomic updates. Otherwise intermediate states cause computation leaks.
- [ ] **Missing `defer: true` on `createEffect(on(...))`** — effects that refetch on filter/page changes must use `{ defer: true }` + separate `onMount` for initial fetch. Without defer, the effect fires synchronously during mount and conflicts with route transitions.
- [ ] **Reactive expression inside `<Title>`** — `<Title>{signal() ? "A" : "B"}</Title>` leaks during route transitions. Pre-compute as `createMemo`, pass resolved string.
- [ ] **`createResource` usage anywhere** — the project uses `onMount` + `createSignal` + `alive` guard + `batch` for all data fetching. `createResource` causes orphaned computation warnings on route transitions and Suspense blanking in conditional components. Replace with signals pattern.
- [ ] **Return from `onMount` or `createEffect`** — `onMount(() => { ...; return () => cleanup() })` silently ignores the cleanup function. Must use `onCleanup(() => cleanup())` inside the body instead. This is the React pattern — it compiles in Solid but the cleanup never runs.
- [ ] **Missing alive guard** — any async operation without `let alive = true; onCleanup(...)` check? Also check `.then()` callbacks (e.g., avatar URL resolution) — they must check `if (alive)` before setting signals.
- [ ] **Missing PRIVATE_ROUTES entry** — is this route listed in `frontend/src/lib/constants.ts`?
- [ ] **window.confirm usage** — should be `DestructiveModal`.
- [ ] **Route-based detail view** — should it be a signal-driven modal instead?

## Migration Files

- [ ] **Missing down migration** — does the `.down.sql` exist and reverse everything?
- [ ] **Missing indexes** — are FK columns and filter columns indexed?
- [ ] **Missing updated_at trigger** — does the table have `updated_at` and the trigger?

## External API Integration

- [ ] **API field names mismatch** — verify JSON struct tags match the external API's exact field names. Check the API docs, not assumptions.
- [ ] **Query param vs body** — some APIs expect fields as query parameters, not in the request body. Verify endpoint documentation.
- [ ] **Email deliverability** — external services may reject emails on bounce suppression lists. Verify `IsConfigured()` returns true before relying on email delivery. Use `service.Retry()` for resilience.
- [ ] **Webhook signature verification** — every webhook endpoint must verify the request signature (e.g., HMAC-SHA256 with timing-safe comparison).

## Cross-Cutting

- [ ] **Secrets in code** — any hardcoded API keys, passwords, or tokens?
- [ ] **Secrets in shell scripts** — any `export VAR=secret`? Secrets should be loaded from env files, never hardcoded.
- [ ] **N+1 queries** — any loop that makes a DB query per iteration? Should be a JOIN or batch query.
- [ ] **Missing error handling in goroutines** — any `go fn()` without error logging?
- [ ] **SSE endpoints excluded from middleware** — SSE (long-lived) endpoints must skip gzip compression (causes flush panic on close) and request timeout middleware. Check `middleware/stack.go` skipper functions.
- [ ] **Modal overflow** — any modal content taller than the viewport? The global Modal has `max-h-[90vh] overflow-y-auto` but verify iframe/large content modals use appropriate height constraints.

---
> Source: [golid-ai/golid](https://github.com/golid-ai/golid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->
