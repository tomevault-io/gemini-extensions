## simpleralph

> SimpleRalph is a standalone Python CLI project for running a file-driven autonomous coding loop inside any repository.

# SimpleRalph Agent Guide

SimpleRalph is a standalone Python CLI project for running a file-driven autonomous coding loop inside any repository.

## Start here
- Project overview: `README.md`
- Contribution workflow: `CONTRIBUTING.md`
- Package metadata: `pyproject.toml`
- CLI entrypoint: `src/simpleralph/cli.py`
- Session/runtime logic: `src/simpleralph/project.py`
- Status/log helpers: `src/simpleralph/state.py`
- Regression tests: `tests/`

## Project shape
- `src/simpleralph/cli.py` — user-facing CLI with `init`, `run`, `status`, `export`
- `src/simpleralph/project.py` — session creation, config handling, agent command wiring, loop execution
- `src/simpleralph/state.py` — stop marker detection, iteration updates, merged per-iteration logging
- `src/simpleralph/templates/` — seeded session artifacts (`PRD.md`, `Tasks.md`, `Status.md`, `Log.md`, `agent_prompt.txt`)
- `tests/test_project.py` — session/bootstrap/export and CLI behavior tests
- `tests/test_state.py` — log/status helper tests

## Core behavior to preserve
- SimpleRalph is **file-driven**, not chat-memory-driven.
- The main local session directory is `.simpleralph/` inside the target repository.
- A session contains:
  - `PRD.md`
  - `Tasks.md`
  - `Status.md`
  - `Log.md`
  - `agent_prompt.txt`
  - `config.env`
- `run` updates the iteration counter in `Status.md`, invokes the configured agent command, then runs compile/test gates.
- The loop stops only when `Status.md` contains one of these markers on its own line / prefix:
  - `ALL_TASKS_DONE`
  - `CRITICAL_FAILURE: <reason>`

## Agent model
- SimpleRalph is **agent-agnostic**.
- The current default session config still seeds an OpenCode-compatible command template, but the architecture is generic.
- The canonical hook for runtime agent execution is `AGENT_COMMAND` in `config.env`.
- Available environment variables for `AGENT_COMMAND` are assembled in `build_agent_environment(...)` in `src/simpleralph/project.py`.
- If the target repository contains `AGENTS.md`, its path is exposed as `AGENTS_FILE`, and the default generated command attaches it.

## Development rules
- Keep the project small and inspectable.
- Prefer explicit files and simple CLI behavior over hidden orchestration.
- Do not hardcode repository-specific test suites or domain-specific tasks into the generic package.
- Preserve alpha usability: one topic → one session → one verified step at a time.
- Keep shell commands and docs aligned. If CLI semantics change, update `README.md` and tests together.

## Validation requirements
- Run tests with:
  - `PYTHONPATH=src python3 -m pytest -q`
- For install smoke testing, prefer a fresh virtualenv inside this project directory.
- If you change session or command-template behavior, verify:
  - `simpleralph init`
  - `simpleralph status`
  - `simpleralph export`
  still work from an installed CLI.

## Open-source hygiene
- `README.md` is the public landing page. Keep it credible and consistent with implemented behavior.
- `CONTRIBUTING.md` should stay short and practical.
- Avoid committing local runtime artifacts like `.venv-smoke/`, `.pytest_cache/`, or target repo `.simpleralph/` state.
- Keep package metadata URLs in `pyproject.toml` aligned with the public GitHub repository.

## Commit message skill
- When drafting or creating a git commit for this repository, use the commit title format: `[Type] 변경 내용`
- Use Korean as the base language for the title, and keep only the tag in English.
- Allowed tags:
  - `[Feat]` 새로운 기능 추가
  - `[Fix]` 버그 수정
  - `[Refactor]` 동작 변화 없는 구조 개선
  - `[Docs]` 문서 수정
  - `[Test]` 테스트 추가/수정
  - `[Chore]` 설정, 정리, 기타 유지보수
  - `[Build]` 빌드/패키지 관련 변경
  - `[CI]` CI 설정 변경
  - `[Style]` 포맷, 주석, 비기능 스타일 수정
  - `[Perf]` 성능 개선
  - `[Revert]` 커밋 되돌리기
- Keep the title to one line and do not end it with a period.
- Prefer a message that makes the reason for the change clear, not just the surface-level edit.
- Good examples:
  - `[Feat] 세션 export 명령 추가`
  - `[Fix] crash 이후 누락된 history snapshot 복구`
  - `[Docs] crash recovery 동작 방식 문서화`

## Common pitfalls
- `--repo` is a global CLI flag. Document examples as `simpleralph --repo /path ...`, not after the subcommand.
- `agent_prompt.txt` is a text prompt file, not Markdown.
- `export` writes fixed filenames and can overwrite previous exported session files in the same directory.
- `AGENT_COMMAND`, `COMPILE_CMD`, and `TEST_CMD` are executed via `/bin/bash`; document that clearly for portability concerns.

---
> Source: [Seungwan98/SimpleRalph](https://github.com/Seungwan98/SimpleRalph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-26 -->
