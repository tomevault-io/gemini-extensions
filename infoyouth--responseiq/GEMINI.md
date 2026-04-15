## responseiq

> ResponseIQ is an **AI-native self-healing infrastructure copilot**. It reads application

# ResponseIQ — GitHub Copilot Instructions

## Project Overview

ResponseIQ is an **AI-native self-healing infrastructure copilot**. It reads application
logs, loads the actual source code referenced in stack traces into LLM context using
Tree-sitter AST parsing, and generates surgical, context-aware remediation patches.

---

## Architecture

```
Log Input → Noise Filter → Concurrent Scan (asyncio) → AI Classifier
    → Context Extractor (Tree-sitter) → LLM Reasoning → Trust Gate (7 rules)
    → ProofBundle (SHA-256 chain) → GitHub PR (githubkit) → PR Bot (/responseiq)
```

Key source files:
- `src/responseiq/cli.py` — CLI entry point (`--mode scan | fix | shadow`)
- `src/responseiq/app.py` — FastAPI server (webhooks: Datadog, PagerDuty, Sentry)
- `src/responseiq/services/remediation_service.py` — Core LLM reasoning brain
- `src/responseiq/services/github_pr_service.py` — GitHub PR bot (githubkit)
- `src/responseiq/integrations/github_integration.py` — GitHub client wrapper (githubkit)
- `src/responseiq/utils/context_extractor.py` — Tree-sitter AST source loading
- `src/responseiq/db.py` — SQLAlchemy + psycopg3 (URL: `postgresql+psycopg://`)

---

## Tech Stack Rules

| What | Use | Never Use |
|---|---|---|
| GitHub API | `githubkit` + `TokenAuthStrategy` | ~~PyGithub~~ |
| Tree-sitter langs | `tree-sitter-language-pack` | ~~tree-sitter-languages~~ |
| Postgres driver | `psycopg[binary]>=3.0` | ~~psycopg2-binary~~ |
| Security scan | `pip-audit` | ~~safety~~ |
| Linting/format | `ruff` | ~~black~~, ~~flake8~~ |
| Package manager | `uv` | ~~pip~~ directly |

---

## Coding Conventions

- Python 3.12, strict mypy, ruff format + lint
- All GitHub REST calls use `_split_repo(repo_name)` → `(owner, repo)` tuple
- All GitHub responses access data via `.parsed_data` (not direct attribute access)
- `RequestFailed` is the githubkit exception (not `GithubException`)
- Dry-run mode activates automatically when `RESPONSEIQ_GITHUB_TOKEN` is unset
- ProofBundle fields use UTC-aware datetimes (`datetime.now(timezone.utc)`)
- Tree-sitter language calls require `# type: ignore[arg-type]` due to strict `Literal` typing

---

## Testing

```bash
make test           # uv run pytest -n auto --dist=loadscope
make lint           # ruff format --check + ruff check + mypy
make all            # lint + type-check + test + build + security
```

- Tests use `pytest-xdist` with `loadscope` — do NOT share mutable state across test modules
- Integration tests in `tests/integration/` require `RUN_INTEGRATION=1` and a live Postgres
- Fixtures are in `fixtures/` — use `fixture_high.json`, `fixture_medium.json`, `fixture_none.json`
- GitHub PR service tests are **dry-run only** — no real API calls in tests

---

## Commit Convention

Conventional Commits are enforced by `python-semantic-release`:
- `feat:` → minor version bump
- `fix:` → patch version bump
- `feat!:` or `BREAKING CHANGE:` → major version bump
- `chore:`, `docs:`, `ci:`, `refactor:` → no release (unless configured)

Add `[skip ci]` to skip the CI pipeline (e.g., for changelog-only commits).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/infoyouth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-10 -->
