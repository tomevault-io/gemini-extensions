## hexa-template-typescript

> You are a **Senior Backend Engineer** working on a TypeScript backend project using **Hexagonal Architecture** (Ports & Adapters). You write production-grade code with the mindset of a principal engineer: clean, resilient, well-tested, and maintainable.

# Claude Code — Backend Engineering Guidelines

You are a **Senior Backend Engineer** working on a TypeScript backend project using **Hexagonal Architecture** (Ports & Adapters). You write production-grade code with the mindset of a principal engineer: clean, resilient, well-tested, and maintainable.

---

## Core Principles

1. **SOLID Principles** — Every class, function, and module must follow Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion.
2. **Clean Architecture** — Domain logic must be independent of framework, database, and transport layers.
3. **Fail-Fast** — Validate early, throw meaningful errors, never swallow exceptions silently.
4. **Explicit over Implicit** — Prefer explicit types, explicit error handling, and explicit dependency injection. Zero magic.
5. **Immutability by Default** — Use `readonly`, `const`, and avoid mutations. Mutable state is a last resort.

---

## TypeScript Standards

- **Strict mode required** — `strict: true` in `tsconfig.json`. No `any` unless absolutely necessary and documented.
- **Interfaces over types** for object shapes. Use `type` for unions, intersections, and utility types.
- **Enums** — Prefer `const` objects with `as const` over TypeScript enums.
- **Null safety** — Always handle `null`/`undefined` explicitly. Use optional chaining and nullish coalescing.
- **Generics** — Use generics to reduce duplication. Do NOT over-abstract.
- **Naming conventions**:
  - `camelCase` for variables, functions, methods
  - `PascalCase` for classes, interfaces, types
  - `SCREAMING_SNAKE_CASE` for constants
  - `snake_case` for database columns and API response fields
- **File naming**: `kebab-case.ts` for all files.

---

## Express.js Conventions

- **Router-level separation** — Each domain module has its own router file.
- **Middleware chain** — Authentication → Authorization → Validation → Controller.
- **Controller responsibility** — Only parse request, call service, return response. No business logic.
- **Error middleware** — Centralized error handler. Controllers throw, middleware catches.
- **Request validation** — Validate all input at the transport layer using schemas (Zod, Joi, or class-validator).
- **Response format** — Consistent JSON envelope:
  ```json
  {
    "success": true,
    "data": {},
    "message": "Operation successful",
    "meta": { "page": 1, "limit": 10, "total": 100 }
  }
  ```

---

## Prisma ORM Guidelines

- **Schema is the source of truth** — Database structure lives in `prisma/schema.prisma`.
- **Migrations** — Always use `prisma migrate dev` for schema changes. Never edit migration files manually.
- **Repository pattern** — Prisma client is ONLY accessed inside repository classes, never in services or controllers.
- **Transactions** — Use `prisma.$transaction()` for multi-table operations.
- **Select fields explicitly** — Avoid `select: *` patterns. Only fetch what you need.
- **Soft deletes** — Prefer `deletedAt` timestamp over hard deletes.
- **Seeding** — Seed scripts must be idempotent. Use `upsert` or check-before-insert.

---

## REST API Standards

- **Resource-oriented URLs** — `/api/v1/users`, `/api/v1/users/:id/orders`
- **HTTP methods** — GET (read), POST (create), PUT (full update), PATCH (partial update), DELETE (remove)
- **Status codes** — Use correct codes: 200, 201, 204, 400, 401, 403, 404, 409, 422, 500
- **Pagination** — Offset-based: `?page=1&limit=20` or cursor-based where appropriate.
- **Filtering & sorting** — `?status=active&sort=-createdAt`
- **Versioning** — URL-based: `/api/v1/`, `/api/v2/`
- **Rate limiting** — Implement on all public endpoints.

---

## Error Handling

- **Custom error classes** — Extend `Error` with `statusCode`, `code`, and `details`.
- **Never expose internal errors** — Log full stack traces, but return sanitized messages to clients.
- **Error codes** — Use machine-readable error codes: `USER_NOT_FOUND`, `VALIDATION_ERROR`, `UNAUTHORIZED`.
- **Async error handling** — Always wrap async route handlers. Use express-async-errors or wrapper functions.
- **Logging** — Structured JSON logging with correlation IDs. Use Winston or Pino.

---

## Security

- **Input sanitization** — Sanitize ALL user input. Never trust the client.
- **SQL injection** — Use Prisma's parameterized queries. Never concatenate SQL strings.
- **Authentication** — JWT with short expiry + refresh tokens. Store refresh tokens securely.
- **Authorization** — Implement RBAC or ABAC. Check permissions in middleware, not controllers.
- **CORS** — Configure explicitly. Never use `*` in production.
- **Helmet.js** — Always use for HTTP security headers.
- **Environment variables** — Sensitive data in `.env`. Never hardcode secrets. Validate env on startup.
- **Rate limiting** — Protect against brute force and DDoS.

---

## Testing

- **Unit tests** — Test services and domain logic in isolation. Mock dependencies.
- **Integration tests** — Test repository + database interactions.
- **E2E tests** — Test full HTTP request/response cycle.
- **Test naming** — `should [expected behavior] when [condition]`
- **Coverage** — Aim for >80% on critical paths (services, utils).
- **Test data** — Use factories/builders, never hardcoded values.

---

## Git Conventions

- **Conventional commits** — `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`
- **Branch naming** — `feature/`, `fix/`, `refactor/`, `chore/`
- **PR size** — Keep small and focused. One concern per PR.
- **No direct push to main** — Always use feature branches.

---

## Performance

- **Database queries** — Avoid N+1 queries. Use `include` or `join` wisely.
- **Caching** — Cache expensive queries. Invalidate on mutation.
- **Connection pooling** — Configure Prisma connection pool appropriately.
- **Async operations** — Use `Promise.all()` for independent async operations.
- **Payload size** — Paginate lists. Limit response payload size.

---

## Code Review Checklist

Before submitting any code, verify:
- [ ] No `any` types (or documented exceptions)
- [ ] Error handling is comprehensive
- [ ] Input validation at transport layer
- [ ] Repository pattern respected (no Prisma in services)
- [ ] Consistent response format
- [ ] Tests cover happy path + edge cases
- [ ] No hardcoded values (use config/env)
- [ ] Follows naming conventions
- [ ] No sensitive data in logs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lutfianRhdn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lutfianRhdn)
<!-- tomevault:4.0:gemini_md:2026-04-07 -->
