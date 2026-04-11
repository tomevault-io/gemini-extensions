## git-rest

> Auto-generated from all feature plans. Last updated: 2025-10-11

# git-rest Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-11

## Active Technologies
- Python 3.12+ + Flask (with Blueprints, application factory pattern), gitpython, pydantic, Gunicorn, Nginx (reverse proxy), Docker, pytest, Behave, uv, ruff (001-develop-git-rest)
- Python 3.12+ + Flask, gitpython, pydantic (002-i-want-to)
- Filesystem (user data in separate directories/namespaces) (002-i-want-to)
- Python 3.12+ + Flask, gitpython, pydantic, pytest, Behave, uv, ruff (003-improve-testing-go)
- Filesystem (per-user repo directories) (004-multi-repo-testing)
- Python 3.12+ + Flask, flasgger (005-interactive-demo-use)
- N/A (no persistent storage required for demo) (005-interactive-demo-use)
- Python 3.12+ + Flask (Blueprints, application factory), gitpython, pydantic (008-explicit-clone-url)

## Guidance for AI Command Execution
- When creating a new terminal session remember to activate the Python environment:
  ```sh
  source .venv/bin/activate
  ```
- When running environment-specific commands such as `pytest`, `ruff`, or similar tools, always use `uv run` as a prefix (e.g., `uv run pytest`, `uv run ruff check .`).
- This ensures the correct environment and dependencies are used for all test and lint operations.

## Project Structure
```
src/
tests/
```

## Commands
cd src [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] pytest [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] ruff check .

## Code Style
Python 3.12+: Follow standard conventions

## Recent Changes
- 009-init-new-repo: Added Python 3.12+ + Flask, gitpython, pydantic
- 008-explicit-clone-url: Added Python 3.12+ + Flask (Blueprints, application factory), gitpython, pydantic
- 007-exception-exposure-there: Added [if applicable, e.g., PostgreSQL, CoreData, files or N/A]

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jondavid-black)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jondavid-black)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
