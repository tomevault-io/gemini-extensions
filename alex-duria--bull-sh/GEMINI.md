## bull-sh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**bullsh** is an agentic CLI for investment research. Users prompt naturally, and an AI agent autonomously gathers SEC filings, social sentiment, analyst data, and news to build investment theses.

## Build & Development Commands

```bash
# Install dependencies
pip install -e .                    # Core
pip install -e ".[dev,rag,export]"  # All optional dependencies

# Run the CLI
bullsh                              # Interactive REPL (freestyle)
bullsh research NVDA                # Freestyle research
bullsh research NVDA --framework piotroski  # Framework-guided
bullsh compare AMD NVDA INTC        # Compare up to 3 companies
bullsh --debug                      # Enable debug logging
bullsh --debug --debug-filter "tools,api"  # Filter debug logs
bullsh --no-intro                   # Skip animated intro

# Run tests
pytest                              # All tests
pytest tests/test_config.py         # Single test file
pytest -k "test_name"               # Run specific test
pytest --cov=bullsh                 # With coverage

# Linting and type checking
ruff check bullsh/                  # Lint
ruff format bullsh/                 # Format
mypy bullsh/                        # Type check (strict mode)
```

## Architecture

Uses **Stampede**, an autonomous Plan→Execute→Reflect loop for research orchestration.

```
bullsh/
├── cli.py              # Typer app, subcommands, REPL entry
├── config.py           # Load .env (secrets) + config.toml (prefs)
├── logging.py          # Debug logging with filtering
├── agent/
│   ├── orchestrator.py # Main dispatcher, integrates Stampede loop
│   ├── base.py         # SubAgent ABC with bounded iterations
│   ├── research.py     # Single-company deep dive subagent
│   ├── compare.py      # Multi-company parallel research subagent
│   └── stampede/       # Autonomous Plan→Execute→Reflect architecture
│       ├── schemas.py      # Pydantic models (Understanding, Task, TaskPlan, etc.)
│       ├── understanding.py # Query comprehension with confidence scoring
│       ├── planner.py      # Task decomposition with framework awareness
│       ├── executor.py     # Dependency-aware parallel task execution
│       ├── reflector.py    # Self-validation with "default to complete" philosophy
│       ├── synthesizer.py  # Streaming response generation
│       └── loop.py         # Main orchestration loop
├── factors/            # Multi-factor analysis module (pure Python math)
│   ├── calculator.py   # Z-score calculations across factors
│   ├── regression.py   # Fama-French factor regression
│   ├── scenarios.py    # What-if scenario modeling
│   ├── session.py      # Interactive factor session state machine
│   ├── fetcher.py      # Price/factor data fetching
│   └── excel_factors.py # Factor analysis Excel export
├── frameworks/
│   ├── base.py         # Framework loader, base class
│   ├── piotroski.py    # F-Score computation (9-point quantitative)
│   ├── porter.py       # Five Forces extraction (qualitative)
│   └── valuation.py    # Multi-method price targets
├── tools/
│   ├── base.py         # ToolResult dataclass, ToolDefinition, get_tools_for_claude()
│   ├── sec.py          # SEC EDGAR via edgartools (auto-indexes for RAG)
│   ├── social.py       # StockTwits (primary) + Reddit (fallback) sentiment
│   ├── yahoo.py        # Analyst ratings, price data via yfinance
│   ├── news.py         # DuckDuckGo news + web search
│   ├── rag.py          # Vector search over indexed SEC filings
│   ├── excel.py        # Excel spreadsheet generation
│   ├── export.py       # PDF/DOCX export
│   ├── factors.py      # Factor calculation tool wrapper
│   ├── thesis.py       # Thesis export with YAML frontmatter provenance
│   ├── financials.py   # Unified financial statements (Financial Datasets API → Yahoo fallback)
│   └── insiders.py     # Insider transactions from Financial Datasets API
├── ui/
│   ├── repl.py         # Interactive loop with Rich + prompt_toolkit
│   ├── intro.py        # Animated candlestick intro
│   ├── theme.py        # Color theme
│   ├── formatter.py    # Response formatting
│   └── status.py       # Tool status indicators
└── storage/
    ├── cache.py        # HTTP response caching
    ├── sessions.py     # Session persistence and search
    ├── vectordb.py     # DuckDB vector storage for RAG
    └── artifacts.py    # Export artifact tracking
```

## Analysis Frameworks

Frameworks provide **structure, not constraints**. They guide research while preserving user creativity.

**Built-in:**
- **Piotroski F-Score**: 9-point quantitative financial health (computed from 10-K/10-Q)
- **Porter's Five Forces**: Qualitative competitive analysis (extracted from 10-K text)
- **Hedge Fund Pitch**: Output format for thesis (Thesis → Catalysts → Valuation → Risks)

**Custom frameworks**: Users can define their own criteria in TOML files (`~/.bullsh/frameworks/custom/`)

**Philosophy**: The thesis is the user's - shaped by their questions, interpretations, and follow-ups. Two users researching the same stock produce different theses.

## Factor Analysis Module

The `factors/` module provides multi-factor stock analysis with pure Python calculations (no LLM calls for math). Key concepts:

- **Z-scores**: Cross-sectional comparison against peer group (value, momentum, quality, growth, size, volatility)
- **Fama-French regression**: Decomposes returns into market, size (SMB), and value (HML) factor exposures
- **Scenario modeling**: What-if analysis for factor shocks
- **Token-efficient**: All math is local; LLM only used for interpretation

The module uses a state machine (`FactorSession`) with stages: TICKER_INPUT → PEER_SELECTION → FACTOR_ANALYSIS → REGRESSION → SCENARIOS

## Stampede Architecture

Stampede is the autonomous research loop that powers bullsh. It implements a Plan→Execute→Reflect cycle inspired by how expert analysts work.

```
User Query
    │
    ▼
┌──────────────┐
│  UNDERSTAND  │  ← Extract intent, tickers, depth, confidence
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────────┐
│    ITERATION LOOP (max 5)            │
│  ┌────────┐  ┌─────────┐  ┌────────┐ │
│  │  PLAN  │→ │ EXECUTE │→ │REFLECT │ │
│  └────────┘  └─────────┘  └────────┘ │
│       ▲                       │      │
│       └── guidance ◄──────────┘      │
│                                      │
│  is_complete? → YES → EXIT LOOP      │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────┐
│  SYNTHESIZE  │  ← Stream final response
└──────────────┘
```

**Key Features:**
- **Confidence-based clarification**: If understanding confidence < 0.8, asks user to clarify
- **Simple query shortcuts**: Quick lookups bypass planning entirely
- **Framework-aware planning**: Piotroski gets 9-signal tasks, Porter gets competitive analysis tasks
- **Dependency-aware parallelism**: Independent tasks run in parallel via `asyncio.gather`
- **Self-reflection**: "Default to complete" philosophy - only iterates if critical data missing
- **Company name resolution**: "Tesla" → "TSLA", "Apple" → "AAPL" automatically

## Key Design Decisions

- **Stampede by default**: Orchestrator uses Stampede loop for autonomous research; legacy tool loop still available
- **Frameworks as compass**: Guide conversation without limiting exploration; users can override assessments
- **Bounded iterations**: Max 5 Stampede iterations (Plan→Execute→Reflect); max 15 fallback tool iterations
- **Parallel in compare**: Compare agent spawns research subagents in parallel (one per company)
- **Graceful degradation**: Failed tools noted in response with confidence scores, never blocks flow
- **Single API key**: Only Anthropic API required; all data sources are free/scraped
- **Confidence scores**: All tools return `ToolResult` with confidence 0-1 and status
- **RAG-first for filings**: SEC filings auto-index on fetch; use `rag_search` before `web_search` for filing content
- **Prompt caching**: System prompts cached with `ephemeral` cache_control for 90% cost reduction
- **Token limits**: Session and per-turn limits with warnings at 80% threshold

## Testing

Unit tests with mocked responses only - no external network calls in CI. Uses `pytest-asyncio` with `asyncio_mode = "auto"`.

```bash
pytest                          # All tests
pytest tests/test_config.py     # Single file
pytest -k "test_tool"           # Pattern match
pytest -v --tb=short            # Verbose with short tracebacks (default)
```

Mock HTTP responses with `respx` for httpx-based tools.

## Changelog Maintenance

**IMPORTANT**: After completing any significant work, you MUST update `CHANGELOG.md` with a plain-English summary of changes.

### What to log:
- New features or commands added
- Bug fixes
- Architecture changes
- Breaking changes
- Dependencies added/removed
- Files created or significantly modified

### Format:
```markdown
## [Date] - Brief Title

**Developer**: [Name or "Claude (AI)"]

### Added
- Description of new feature

### Changed
- Description of modification

### Fixed
- Description of bug fix

### Files Modified
- `path/to/file.py` - what changed
```

### Guidelines:
- Write for human maintainers, not machines
- Be specific: "Added SEC 10-K parsing" not "Updated tools"
- Include the WHY when non-obvious
- Group related changes together
- Most recent entries at the top

---
> Source: [alex-duria/bull.sh](https://github.com/alex-duria/bull.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->
