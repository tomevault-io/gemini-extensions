## clankbrain

> > **Keep this file under 200 lines.** Commands only. Project conventions → `rules/project-context.md`. Everything the agent needs to remember → `.claude/memory/`. Long sessions → `/learn` then `/compact`.

# [Project Name] — Claude Code + Codex Project Context

> **Keep this file under 200 lines.** Commands only. Project conventions → `rules/project-context.md`. Everything the agent needs to remember → `.claude/memory/`. Long sessions → `/learn` then `/compact`.

---

## Session Commands

| Tier | Commands |
|------|----------|
| **Core** | `Start Session` · `End Session` |
| **On Demand** | `Plan` · `Debug Session` · `/learn` · `/evolve` · `Check Drift` · `Guard Check` · `Pre-Ship Check` · `Code Health` · `Mode` · `Estimate` · `Handoff` · `Search Memory` · `Generate Guards` · `Generate Skills` · `Update Kit` |
| **Opt-In** | `Team Pull` / `Team Push` · `Sync Memory` / `Pull Memory` |

---

### `Setup Memory`
If `setup.py` exists run it. Otherwise: `python -c "import urllib.request; exec(urllib.request.urlopen('https://raw.githubusercontent.com/YehudaFrankel/clankbrain/main/install.py').read().decode())"`

### `Start Session`
1. Run `python tools/memory.py --session-start`
2. Read `STATUS.md` → `memory/lessons.md` → `memory/decisions.md` → `memory/tasks/regret.md` → `memory/tasks/todo.md`
3. Scan `memory/plans/` — surface any file with `Status: Draft` or `On Hold`
4. Report: "Session N ready. Last change: [X]. What are we working on?"

### `End Session`
1. Run `/learn` — extract lessons + decisions from this session
2. Update memory files for everything changed (see Auto-Save Rule)
3. Update `STATUS.md` — increment session, one-line summary
4. Run `python tools/memory.py --check-drift`
5. If on a team: share what you learned → `Team Push`
6. Report: "Session N complete. Updated: [list]. Memory clean."

### `Plan [feature]`
Invoke the `plan` skill. Opens/creates `memory/plans/[slug].md`, walks problem → research → options → decision → spec. Always show the **full plan file** after every update — never a diff or summary.

### `Debug Session`
Invoke the `debug-session` skill: reproduce → isolate → hypothesize → fix only the confirmed root cause → verify → log to `tasks/errors.md`.

### On-Demand Commands
| Command | Action |
|---------|--------|
| `Check Drift` | `python tools/memory.py --check-drift` — fix any found |
| `Guard Check` | `python tools/memory.py --guard-check` — PASS/FAIL per guard |
| `Pre-Ship Check` | guard-check + drift-check + session edit count + `git diff --stat` |
| `Code Health` | Scan for console.log, hardcoded values, missing error handling, dead code, files >500 lines |
| `Mode [develop\|review\|safe\|deploy]` | Set + enforce tool access constraints for the session |
| `Estimate: [task]` | Read files → complexity + risk + file list → write to todo.md if confirmed |
| `Handoff` | Generate `HANDOFF.md` from STATUS + todo + decisions + errors + gotchas |
| `Search Memory: [topic]` | `python tools/memory.py --search "[topic]"` |
| `Progress Report` | `python tools/memory.py --progress-report` |
| `Kit Health` | `python tools/memory.py --kit-health` — fix FAILs immediately |
| `Generate Guards` | Invoke `generate-guards` skill |
| `Generate Skills` | Invoke `generate-guards` then scan stack for useful project skills |
| `Analyze Codebase` | Scan all JS/CSS/backend files, update memory files with findings |
| `Install Memory` | Scan codebase, fill memory files, copy to system memory path |
| `Update Kit` | Run `update.py` if present, otherwise fetch from clankbrain repo |
| `/learn` | Invoke `learn` skill — extract lessons, decisions, rejected approaches |
| `/evolve` | Invoke `evolve` skill — patch failing skills, cluster patterns (every 3–5 sessions) |

### Team (opt-in)
| Command | What it does |
|---------|-------------|
| `Setup Team [url]` | You're the manager. Run once. Sends teammates the URL to join. |
| `Join Team [url]` | You're a new member. Run once with the URL your manager sent. |
| `Team Push` | Share what you learned with the team. Run at End Session. |
| `Team Status` | Check last sync times and what's been shared. |

Team Pull runs automatically at Start Session — no command needed.

### Sync (opt-in)
- `Sync Memory` / `Pull Memory` → `python sync.py push` / `python sync.py pull`
- Setup: `python sync.py setup [repo-url]`

---

## Skill Map

| Workflow | Skills in Order |
|----------|----------------|
| **Build to Learn (discovery)** | Declare `build-mode: learn` → `product-risk` → `prototype-hypothesis` → *(prototype)* → `parallel-prototypes` (if multiple options) → log to `prototype_log.md` |
| **Build to Earn (delivery)** | Declare `build-mode: earn` → `search-first` → `plan` → *(code)* → `code-reviewer` → `verification-loop` → `/learn` |
| **Bug Fix** | `debug-session` → *(fix)* → `verification-loop` → `/learn` |
| **End of Session** | `/learn` → `/evolve-check` → `/evolve` *(every 3–5 sessions)* |
| **Maintenance** | `check-drift` → `guard-check` → `code-health` |
| **Memory** | `/recall [topic]` · `search-memory` · `/forget [topic]` |

Full agent orchestrations (human-in-the-loop breakpoints): see `.claude/agents/`

---

## Auto-Save Rule

After any code change, immediately update the relevant memory file — don't wait for End Session:

| What changed | Update this |
|---|---|
| JS function added/changed | `js_functions.md` |
| HTML/CSS changed | `html_css_reference.md` |
| Endpoint or backend method | `backend_reference.md` |
| Architecture decision | `decisions.md` |
| Rejected approach | `tasks/regret.md` |
| Any change when code-map exists | `code-map.md` — flow, line number, DB entry |

---

## Memory File Conventions (MemPalace-inspired)

Every memory file uses this frontmatter + structure:

```markdown
---
name: short-id
description: one-line hook for MEMORY.md index
type: rule | correction | decision | state | reference | user
valid_from: YYYY-MM-DD        # optional — memory not applicable before this date
valid_until: YYYY-MM-DD       # required for type: state/project — when this may go stale
related: [other-memory.md]    # optional — tunnels to connected memories
---

[Summary — the rule, fact, or decision in plain English]

**Why:** [reason this matters]
**How to apply:** [when to use it]

## Source
> [Verbatim snippet from the conversation where this was established]
— Session N
```

**Types:**
- `rule` — permanent coding/workflow rule (no expiry needed)
- `correction` — one-time fix Claude made that shouldn't repeat
- `decision` — locked architectural choice
- `state` — current phase, active work, temporary facts — always needs `valid_until`
- `reference` — pointer to external system (Jira, Slack, URL)
- `user` — who the user is, their preferences and expertise

**Why `## Source`:** Storing verbatim context alongside summaries improves recall accuracy from ~84% to ~97%. When the summary is ambiguous, the Source block gives Claude the original exchange to reason from.

**`related:`** links are followed automatically by the pre-edit hook (Tunnels). If two memories are connected — e.g. a rule and its exception — link them.

Run `python tools/memory.py --mempalace-audit` to find files missing Source blocks or valid_until dates.

---

## Autonomous Behaviors

- **Skill chaining:** add `## Auto-Chain` to any SKILL.md — `On pass: → run X` / `On fail: → run Y`
- **Self-healing:** minimal fix → retry → escalate only if second attempt fails (add `## Recovery` to skills)
- **Unsaved memory reminder:** stop hook fires when memory has unsaved changes
- **Compound learning:** `/learn` every session → `skill_scores.md` → `/evolve` every 3–5 sessions → better skills

---

@rules/build-mode.md
@rules/plan-before-edit.md
@rules/work-rules.md
@rules/token-rules.md

---

> **Project conventions, file paths, tech stack, design system, and gotchas belong in `rules/project-context.md`.**
> Copy the template: `cp examples/project-context-template.md .claude/rules/project-context.md`

## Session Starter Prompt
> "Read CLAUDE.md or AGENTS.md, plus STATUS.md. We're continuing [Project Name]. Check which phases are complete and let's pick up where we left off."

---
> Source: [YehudaFrankel/clankbrain](https://github.com/YehudaFrankel/clankbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->
