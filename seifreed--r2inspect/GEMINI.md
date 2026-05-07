## r2inspect

> r2inspect is a malware analysis framework that automates static inspection of PE, ELF, and Mach-O binaries using radare2/r2pipe.

# r2inspect – Copilot Instructions

r2inspect is a malware analysis framework that automates static inspection of PE, ELF, and Mach-O binaries using radare2/r2pipe.

## Prerequisites

- Python 3.13+ and radare2 in PATH
- `libmagic` (`brew install libmagic` on macOS, `apt install libmagic-dev` on Linux)
- Active virtualenv (`.venv` or `venv`)

## Commands

```bash
# Install (development)
pip install -e ".[dev]"

# Tests
pytest tests/unit/ -x -q                                      # all unit tests
pytest tests/unit/test_smoke.py -x -v                         # single file
pytest tests/unit/test_smoke.py -k "test_function_name" -v    # single test
pytest tests/integration/ -x -q                               # integration (requires radare2 + binaries)
pytest tests/unit/ --cov=r2inspect --cov-report=term-missing  # with coverage

# Lint / format
ruff check r2inspect/
ruff check --fix r2inspect/
black r2inspect/
black --check r2inspect/
mypy r2inspect/

# CLI
r2inspect samples/fixtures/hello_pe.exe
r2inspect -j samples/fixtures/hello_pe.exe   # JSON output
r2inspect -c samples/fixtures/hello_pe.exe   # CSV output
```

Integration tests resolve binaries from `../r2inspect-test-binaries/` or `R2INSPECT_TEST_BINARIES_DIR`.

## Architecture

```
CLI (click)  →  create_inspector() [factory.py]
                  ├─ R2Session.open() → r2pipe → radare2
                  ├─ R2PipeAdapter  (adapters/r2pipe_adapter.py)
                  ├─ AnalyzerRegistry  (registry/)
                  └─ PipelineBuilder
             →  R2Inspector  [core/inspector.py]  (context-managed facade)
             →  AnalysisPipeline  [pipeline/]
                  ├─ Stages: FileInfo, FormatDetection, FormatAnalysis,
                  │          Hashing, Detection, Security, Metadata
                  ├─ Parallel runtime (ThreadPoolExecutor) or sequential
                  └─ Each stage pulls analyzers from the registry by category/format
             →  Individual Analyzers  [modules/]  (BaseAnalyzer subclasses)
             →  Output  (console/rich, JSON, CSV via cli/)
```

**Batch mode:** `--batch` flag → `application/batch_service.py` → `batch_discovery_runtime.py` finds executables via magic/extension, processes with ThreadPoolExecutor, writes per-file JSON or unified CSV.

## Adding an Analyzer

1. Create a `BaseAnalyzer` subclass in `r2inspect/modules/`:
   ```python
   class MyAnalyzer(BaseAnalyzer):
       def analyze(self) -> dict[str, Any]: ...
       def get_supported_formats(self) -> set[str]: return {"PE", "PE32", "PE32+"}
   ```
2. Register it in `r2inspect/registry/default_registry_data.py`:
   ```python
   {
       "name": "my_analyzer",
       "module_path": "r2inspect.modules.my_module",
       "class_name": "MyAnalyzer",
       "category": AnalyzerCategory.DETECTION,
       "file_formats": PE_FORMATS,   # use the pre-defined sets
       "required": False,
       "description": "...",
   }
   ```
3. Add a Pydantic result model in `r2inspect/schemas/` if needed.

Format set constants (`PE_FORMATS`, `ELF_FORMATS`, `MACHO_FORMATS`) are defined at the top of `default_registry_data.py`.

## Testing Conventions

- **No mocks / monkeypatching.** Use `FakeR2Adapter` from `tests/helpers/r2_fakes.py`:
  ```python
  fake = FakeR2Adapter(cmdj_responses={"iij": [...]}, cmd_responses={"iz": "..."})
  adapter = R2PipeAdapter(fake)
  result = MyAnalyzer(adapter=adapter).analyze()
  ```
- `FakeR2Adapter` records all calls in `.calls` dict for assertion.
- Unit tests live in `tests/unit/`; they must not spawn radare2.
- Use pytest markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.requires_r2`, `@pytest.mark.slow`.
- `conftest.py` enforces: single worker, 1 GB memory limit, 5-minute CPU limit per test.

## Key Conventions

**Style:** black + ruff, 100-char line length, target Python 3.13. isort with `profile = "black"`. Google-style docstrings.

**Interfaces over concrete types:** depend on protocols in `r2inspect/interfaces/core.py` (`ConfigLike`, `FileValidatorLike`, `AnalyzerBackend`, etc.) rather than concrete classes.

**Error handling:** use `r2inspect/error_handling/` unified handler with classifier and severity levels—don't raise raw exceptions from analyzers.

**Config:** `Config` (thread-safe wrapper) → `R2InspectConfig` (Pydantic). Config sources in priority order: CLI flags → `~/.r2inspect/config.json` → env vars (e.g., `R2INSPECT_MAX_WORKERS`) → `Config.DEFAULT_CONFIG`. Access typed config via `config.typed_config`.

**Schemas:** all structured results are Pydantic models in `r2inspect/schemas/`. Use `ResultConverter` in `schemas/converters.py` for dict ↔ model conversion; don't build raw dicts at call sites.

**Pipeline parallelism:** controlled by `pipeline.parallel_execution` config key and `pipeline.max_workers`. Stages that are independent run in parallel; stages with data dependencies are sequential.

---
> Source: [seifreed/r2inspect](https://github.com/seifreed/r2inspect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-06 -->
