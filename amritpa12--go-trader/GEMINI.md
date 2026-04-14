## go-trader

> - Requires Go 1.26.0 — install via Homebrew: `brew install go@1.26`

# go-trader Project Context

## Environment
- Requires Go 1.26.0 — install via Homebrew: `brew install go@1.26`
- Go is not in PATH via shell; use `/opt/homebrew/bin/go` directly (e.g. `cd scheduler && /opt/homebrew/bin/go build .`)
- Python venv at `.venv/bin/python3` (used by executor.go at runtime)
- Python deps managed with `uv` (see `pyproject.toml` / `uv.lock`)

## Setup
- `uv sync` — install Python deps into `.venv`
- Copy `scheduler/config.example.json` → `scheduler/config.json` and fill in API keys

## Repo Structure
- `scheduler/` — Go scheduler (single `package main`); all .go files compile together
  - `executor.go` — Python subprocess runner; max 4 concurrent, 30s timeout per script
  - `server.go` — HTTP status server (`/status`, `/health` endpoints)
  - `discord.go` — Discord alert notifications; `FormatCategorySummary` outputs a monospace code-block table (Strategy/Value/PnL/PnL%) via `writeCatTable`; `fmtComma` handles comma formatting — always pass absolute values (never signed floats)
  - `init.go` — `go-trader init` interactive wizard; `generateConfig(InitOptions) *Config` is pure/testable; `runInit` orchestrates I/O
  - `prompt.go` — `Prompter` struct (String/YesNo/Choice/MultiSelect/Float); inject `NewPrompterFromReader(r,w)` for tests
- `shared_scripts/` — Python entry-point scripts called by the scheduler
  - `check_strategy.py` — spot strategy signal checker
  - `check_options.py` — unified options checker (`--platform=deribit|ibkr`)
  - `check_price.py` — price check script
  - `check_hyperliquid.py` — Hyperliquid perps checker (`<strategy> <symbol> <timeframe> [--mode=paper|live]`; `--execute` for live orders)
- `platforms/` — platform-specific adapters (deribit, ibkr, binanceus, hyperliquid)
  - `deribit/adapter.py` — DeribitExchangeAdapter (live quotes, real expiries/strikes)
  - `ibkr/adapter.py` — IBKRExchangeAdapter (CME strikes, Black-Scholes pricing)
  - `binanceus/adapter.py` — BinanceUSExchangeAdapter (spot only)
  - `hyperliquid/adapter.py` — HyperliquidExchangeAdapter (live perps prices, paper/live trading via `HYPERLIQUID_SECRET_KEY`)
- `shared_tools/` — shared Python utilities (pricing.py, exchange_base.py, data_fetcher, storage)
- `shared_strategies/` — shared strategy logic (spot/, options/)
- `core/` — legacy data utilities used by backtest (data_fetcher, storage)
- `strategies/` — legacy spot strategy logic used by backtest
- `backtest/` — backtesting and paper trading scripts; `run_backtest.py` needs `PYTHONPATH=core:strategies`
- `archive/` — retired/unused modules
- `SKILL.md` — agent operations guide (setup, deploy, backtest commands)

## Key Patterns
- Git commands: always run from repo root, not from `scheduler/` (git add/commit fail with path errors otherwise)
- Platform adapters loaded via `importlib` in `check_options.py`; class discovered by `endswith("ExchangeAdapter")` — all adapter classes must use that suffix
- Scheduler communicates with Python scripts via subprocess stdout JSON; scripts must always output valid JSON even on error
- Python scripts exit 1 on error (Go parses JSON from stdout regardless of exit code)
- Option positions stored in `StrategyState.OptionPositions map[string]*OptionPosition`
- Mutex `mu sync.RWMutex` guards `state`; RLock for reads, Lock for all mutations
- Per-strategy loop uses 6 fine-grained lock phases: RLock(read inputs) → Lock(CheckRisk) → no lock(subprocess) → Lock(execute signal) → RLock/no lock/Lock(mark prices) → RLock(status log)
- Audit lock balance: `grep -n "mu\.\(R\)\?Lock\(\)\|mu\.\(R\)\?Unlock\(\)" scheduler/main.go`
- Platform dispatch: `StrategyConfig.Platform` field (inferred from ID prefix in LoadConfig); use `s.Platform == "ibkr"` not ID prefix checks
- ID prefix → platform: `hl-` → hyperliquid, `ibkr-` → ibkr, `deribit-` → deribit, else → binanceus
- Strategy types: "spot", "options", "perps" — perps paper mode reuses `ExecuteSpotSignal`; live mode calls `RunHyperliquidExecute` before state update
- Hyperliquid sys.path conflict: SDK installs as `hyperliquid` package — clashes with `platforms/hyperliquid/`; fix: add `platforms/hyperliquid/` directly to sys.path (not `platforms/`), then `from adapter import HyperliquidExchangeAdapter`
- Fee dispatch: `CalculatePlatformSpotFee(platform, value)` — 0.035% hyperliquid, 0.1% binanceus (replaces bare `CalculateSpotFee` for platform-aware spot/perps trades)
- State persisted to `scheduler/state.json` (path set in config); per-platform files at `platforms/<name>/state.json`

## Build & Deploy
- Build: `cd scheduler && /opt/homebrew/bin/go build -o ../go-trader .`
- Restart: `systemctl restart go-trader`
- Only needed when `scheduler/*.go` files change
- Python script changes take effect on next scheduler cycle (no rebuild needed)
- Config changes: `systemctl restart go-trader` (no rebuild)
- Service file changes: `systemctl daemon-reload && systemctl restart go-trader`

## Backtest
- `run_backtest.py`: `PYTHONPATH=core:strategies .venv/bin/python3 backtest/run_backtest.py --strategy <n> --symbol BTC/USDT --timeframe 1h --mode single`
- `backtest_options.py`: `.venv/bin/python3 backtest/backtest_options.py --underlying BTC --since YYYY-MM-DD --capital 10000`
- `backtest_theta.py`: `.venv/bin/python3 backtest/backtest_theta.py --underlying BTC --since YYYY-MM-DD --capital 10000`

## ISSUES.md
- When marking an issue fixed: update the row (`NO` → `YES`) **and** the Summary table at the bottom (`Fixed` count +1, `Unfixed` count -1 for that category and Total)

## Testing
- `python3 -m py_compile <file>` — syntax check Python files
- `cd scheduler && /opt/homebrew/bin/go build .` — compile check
- `cd scheduler && /opt/homebrew/bin/go test ./...` — run all unit tests
- `cd scheduler && /opt/homebrew/bin/gofmt -w <file>.go` — format after editing Go files (`-l *.go` lists all files needing formatting)
- Multi-line Go edits with tabs: Edit tool may fail on tab-indented blocks; fallback: `python3 -c "content=open(f).read(); open(f,'w').write(content.replace(old,new,1))"`
- Smoke test: `./go-trader --once`
- Run with config: `./go-trader --config scheduler/config.json`
- Smoke test interactive CLI: `printf "answer1\nanswer2\n" | ./go-trader init`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amritpa12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/amritpa12)
<!-- tomevault:4.0:gemini_md:2026-04-09 -->
