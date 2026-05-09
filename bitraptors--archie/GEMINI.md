## archie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Archie — AI-powered architecture analysis and enforcement for coding agents. Analyzes any codebase, generates a structured blueprint (JSON), and produces: CLAUDE.md, AGENTS.md, per-folder context files, and real-time enforcement hooks. Works with any language, zero dependencies beyond Python 3.9+.

## Repository Layout

- `archie/` — Python package (`archie-cli`): CLI commands, analysis engine, standalone scripts
- `archie/standalone/` — Zero-dependency Python scripts (scanner, renderer, validator, intent layer, health, drift, hooks)
- `npm-package/` — NPM distribution (`npx @bitraptors/archie`): copies scripts + Claude Code commands to target projects
- `tests/` — Test suite (pytest)
- `docs/` — Architecture documentation
- `landing/` — Landing page
- `v1/` — Archived V1 web app (FastAPI backend + Next.js frontend, obsolete)

## Commands

### Standalone Scripts
```bash
# Scanner — analyze repository structure
python3 archie/standalone/scanner.py /path/to/project

# Renderer — generate CLAUDE.md, AGENTS.md, rule files from blueprint
python3 archie/standalone/renderer.py /path/to/project

# Validator — check generated output against actual codebase
python3 archie/standalone/validate.py all /path/to/project

# Hooks — install enforcement hooks
python3 archie/standalone/install_hooks.py /path/to/project

# Intent layer — AI-generated per-folder CLAUDE.md via DAG scheduling
python3 archie/standalone/intent_layer.py prepare /path/to/project
python3 archie/standalone/intent_layer.py next-ready /path/to/project
```

### NPM Package
```bash
# Install Archie into a project (copies scripts + commands)
npx @bitraptors/archie /path/to/project

# Then in Claude Code on the target project:
/archie-scan      # Architecture health check (1-3 min, run often)
/archie-deep-scan # Comprehensive architecture baseline (15-20 min, run once)
/archie-viewer    # Blueprint inspector
```

### Tests
```bash
python -m pytest tests/ -v
```

## Two-Command Architecture

- **`/archie-scan`** — Architecture health check (1-3 min). Runs deterministic scanner for data gathering, then AI analyzes architecture like a senior architect: finds dependency violations, pattern drift, complexity hotspots, proposes enforceable rules.
- **`/archie-deep-scan`** — Comprehensive baseline (15-20 min). Full 2-wave AI analysis producing blueprint, per-folder CLAUDE.md, rules, and health metrics. Run once, then use `/archie-scan` for incremental checks.

## Deep Scan Pipeline (2-Wave)

1. **Scanner** — Counts files, detects frameworks, builds file tree
2. **Wave 1** (parallel) — 3-4 Sonnet agents gather facts:
   - Structure agent: Components, layers, file placement
   - Patterns agent: Communication, design patterns, integrations
   - Technology agent: Stack, deployment, dev rules
   - UI Layer agent: UI components, state, routing (only if frontend_ratio >= 0.20)
3. **Wave 2** — Reasoning agent (Opus) reads all Wave 1 output, produces architectural reasoning:
   - Decision chain (rooted constraint tree with violation keywords)
   - Key decisions with forced_by/enables links
   - Trade-offs with violation signals
   - Pitfalls with causal chains (stems_from)
   - Architecture diagram, implementation guidelines
4. **Normalize** — AI reshapes raw output to canonical schema
5. **Render** — Deterministic JSON→Markdown (CLAUDE.md, AGENTS.md, rule files)
6. **Validate** — Cross-reference output against actual codebase
7. **Intent Layer** — AI-generated per-folder CLAUDE.md via bottom-up DAG
8. **Scan report** — Phase 4 of Step 9 writes `.archie/scan_report.md` with ranked findings in the same format `/archie-scan` emits (so `/archie-share` and future trend runs pick it up immediately after a deep scan)

## Key Data Model

Blueprint JSON (`blueprint.json`) contains: `meta`, `architecture_rules`, `decisions` (with `decision_chain`), `components`, `communication`, `quick_reference`, `technology`, `deployment`, `frontend`, `pitfalls`, `implementation_guidelines`, `development_rules`, `architecture_diagram`.

## Rules System

Rules come from two sources:

1. **Blueprint extraction** (`archie/rules/extractor.py`) — Deterministically extracts `file_placement` and `naming` rules from the blueprint.
2. **AI-proposed rules** (`/archie-scan`) — The scan AI proposes rules with check types: `forbidden_import`, `required_pattern`, `forbidden_content`, `architectural_constraint`, `file_naming`.

Additionally, `platform_rules.json` provides predefined architectural checks installed with every project.

The `/archie-deep-scan` Wave 2 (Opus reasoning) produces deeper architectural context in the blueprint: decision chains, trade-offs with violation signals, pitfalls with causal chains. These inform the AI's analysis during `/archie-scan` but are not extracted as mechanical check rules.

## File Sync

Standalone scripts exist in two places (canonical → copy):
- `archie/standalone/*.py` → `npm-package/assets/*.py`
- `.claude/commands/archie-*.md` (including `archie-scan.md`, `archie-deep-scan.md`) → `npm-package/assets/archie-*.md`

Always edit `archie/standalone/` first, then copy to `npm-package/assets/`.
Always edit `.claude/commands/` first, then copy to `npm-package/assets/`.

**Before committing, run the sync checker:**
```bash
python3 scripts/verify_sync.py
```
This verifies all canonical files, asset copies, and `archie.mjs` references are consistent. Catches missing copies, orphan assets, and dead installer references.

## Skill routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.
The skill has specialized workflows that produce better results than ad-hoc answers.

Key routing rules:
- Product ideas, "is this worth building", brainstorming → invoke office-hours
- Bugs, errors, "why is this broken", 500 errors → invoke investigate
- Ship, deploy, push, create PR → invoke ship
- QA, test the site, find bugs → invoke qa
- Code review, check my diff → invoke review
- Update docs after shipping → invoke document-release
- Weekly retro → invoke retro
- Design system, brand → invoke design-consultation
- Visual audit, design polish → invoke design-review
- Architecture review → invoke plan-eng-review
- Save progress, checkpoint, resume → invoke checkpoint
- Code quality, health check → invoke health

---
> Source: [BitRaptors/Archie](https://github.com/BitRaptors/Archie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-25 -->
