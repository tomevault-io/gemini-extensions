## slack-chat-migrator

> Tool for migrating Slack workspace exports to Google Chat spaces.

# Slack Chat Migrator

Tool for migrating Slack workspace exports to Google Chat spaces.

## Architecture

The codebase uses **dependency injection** — `SlackToChatMigrator` is the composition root that wires all services together. Service functions receive only the explicit dependencies they need via `MigrationContext` (immutable config) and `MigrationState` (mutable tracking).

In **dry-run mode**, `DryRunChatService`/`DryRunDriveService` are injected in place of real API services, eliminating scattered `if dry_run` checks.

```
src/slack_chat_migrator/
├── cli/            # CLI entry points and report generation
│   ├── commands.py    # CLI facade — re-exports from sub-modules
│   ├── common.py      # Shared CLI infrastructure (DefaultGroup, options, InterruptHandler)
│   ├── migrate_cmd.py # migrate command and MigrationOrchestrator
│   ├── validate_cmd.py
│   ├── cleanup_cmd.py   # (deprecated — use migrate --complete)
│   ├── init_cmd.py      # Interactive config.yaml generator
│   ├── setup_cmd.py     # GCP setup wizard (requires [setup] extras)
│   ├── permissions_cmd.py # (deprecated — use validate --creds_path)
│   ├── report.py      # Migration report formatting
│   └── renderers/     # Progress output
│       ├── __init__.py        # Renderer factory (auto-detects TTY)
│       ├── rich_renderer.py   # Rich live progress display
│       └── plain_renderer.py  # Plain text fallback
├── core/           # Core logic
│   ├── channel_processor.py # Per-channel migration orchestration
│   ├── cleanup.py           # Post-migration cleanup (import mode completion, members)
│   ├── config.py            # YAML config loading and validation
│   ├── context.py           # MigrationContext frozen dataclass (immutable config)
│   ├── migration_logging.py # Migration success/failure logging
│   ├── migrator.py          # Composition root — wires all deps, owns lifecycle
│   ├── progress.py          # ProgressTracker event emitter (pub/sub for renderers)
│   └── state.py             # MigrationState with typed sub-states (Spaces/Messages/Users/etc.)
├── services/       # External API integrations
│   ├── chat/       # Google Chat API
│   │   ├── chat_uploader.py    # Chat-based media upload
│   │   └── dry_run_service.py  # No-op Chat API for dry-run mode
│   ├── chat_adapter.py      # Typed wrapper over raw Chat API service
│   ├── drive/      # Google Drive API
│   │   ├── drive_uploader.py      # Drive file upload logic
│   │   ├── dry_run_service.py     # No-op Drive API for dry-run mode
│   │   ├── folder_manager.py      # Drive folder creation and management
│   │   └── shared_drive_manager.py
│   ├── drive_adapter.py     # Typed wrapper over raw Drive API service
│   ├── export_inspector.py  # Slack export analysis (pure I/O, no API calls)
│   ├── files/      # Slack file handling
│   │   ├── file.py              # FileHandler class (delegates to download/permissions)
│   │   ├── file_download.py     # Slack file download logic
│   │   └── file_permissions.py  # Drive file ownership/sharing
│   ├── messages/   # Message migration pipeline
│   │   ├── message_attachments.py
│   │   ├── message_builder.py   # Message payload construction (Slack → Chat format)
│   │   ├── message_sender.py    # Message send logic, error handling, stats
│   │   └── reaction_processor.py # Batch reaction processing
│   ├── setup/      # GCP setup wizard services (optional deps)
│   │   ├── setup_service.py    # Orchestrator and persistent state
│   │   ├── gcp_project.py      # Project creation/selection
│   │   ├── api_enablement.py   # API enablement
│   │   ├── service_account.py  # SA creation, key download, role grants
│   │   └── delegation.py       # Domain-wide delegation test
│   ├── spaces/     # Space lifecycle management
│   │   ├── discovery.py           # Space discovery and mapping for resumption
│   │   ├── historical_membership.py # Historical member import (createTime/deleteTime)
│   │   ├── regular_membership.py  # Regular member addition (post-import)
│   │   └── space_creator.py      # Space creation, listing, and import mode cleanup
│   ├── user.py              # User mapping (Slack → Google)
│   └── user_resolver.py     # User identity resolution and impersonation
└── utils/          # Shared utilities
    ├── api.py      # API retry logic, credential handling
    ├── formatting.py
    ├── logging.py
    ├── permissions.py
    └── user_validation.py
tests/
├── unit/           # Fast, isolated tests
└── integration/    # Tests requiring external services
```

## Development Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pre-commit install
```

## Common Commands

```bash
# Lint and format
ruff check src/slack_chat_migrator/ tests/          # lint
ruff check --fix src/slack_chat_migrator/ tests/    # lint + autofix
ruff format src/slack_chat_migrator/ tests/         # format

# Type check
mypy src/slack_chat_migrator/

# Tests
pytest tests/ -v                           # all tests
pytest tests/unit/ -v                      # unit only
pytest tests/ --cov=slack_chat_migrator    # with coverage

# Commit message validation
cz check --message "fix: description"
```

## Conventions

- **Python 3.9+** — no walrus operator in hot paths, use `from __future__ import annotations` sparingly
- **Conventional Commits** — enforced by commitizen pre-commit hook and CI
  - `feat:`, `fix:`, `refactor:`, `docs:`, `ci:`, `test:`, `chore:`
- **Trunk-based development** — work on feature branches, merge to `main`, release via tags
- **Line length** — 88 characters (ruff)
- **Import order** — managed by ruff (isort-compatible, black profile)

## Release Process

1. `cz bump` — bumps version in `__init__.py`, updates `CHANGELOG.md`, creates git tag
2. `git push && git push --tags`
3. Tag push triggers `.github/workflows/release.yml` → creates GitHub Release
4. GitHub Release triggers `.github/workflows/python-publish.yml` → publishes to PyPI

---
> Source: [nicklamont/slack-chat-migrator](https://github.com/nicklamont/slack-chat-migrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-13 -->
