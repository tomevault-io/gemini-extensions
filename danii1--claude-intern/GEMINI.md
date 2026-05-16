## claude-intern

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Claude Intern - AI tool for automatically implementing JIRA tasks using Claude Code. Supports single/batch task processing via JQL queries, fetches JIRA details, formats for Claude, and automates git workflow + PR creation.

## Development Commands

- `bun start [TASK-KEYS...]` - Run with Bun
- `bun run build` - Build to `dist/` for distribution
- `bun run typecheck` - Type check without compilation
- `bun test` - Run test suite
- `bun run install-global` - Build and install globally for testing

## Architecture

### Core Components

- **[src/index.ts](src/index.ts)** - Main entry, CLI parsing, orchestrates workflow: fetch → format → git → claude → commit → PR
- **[src/lib/jira-client.ts](src/lib/jira-client.ts)** - JIRA REST API v3 client, JQL queries, fetches issues/comments/attachments
- **[src/lib/claude-formatter.ts](src/lib/claude-formatter.ts)** - Formats JIRA data (ADF/HTML → Markdown) for Claude prompts
- **[src/lib/utils.ts](src/lib/utils.ts)** - Git operations, file handling utilities
- **[src/lib/github-reviews.ts](src/lib/github-reviews.ts)** - GitHub API client for PR reviews
- **[src/lib/review-formatter.ts](src/lib/review-formatter.ts)** - Formats PR review feedback for Claude
- **[src/lib/address-review.ts](src/lib/address-review.ts)** - Handles PR review responses
- **[src/lib/auto-review-loop.ts](src/lib/auto-review-loop.ts)** - Automatic PR self-review and improvement loop
- **[src/webhook-server.ts](src/webhook-server.ts)** - Webhook server for automated PR review handling
- **[src/types/](src/types/)** - TypeScript interfaces

### Key Workflows

**JIRA Task Processing:**
1. Fetch JIRA details → 2. Transition to "In Progress" → 3. Create `feature/{task-key}` branch → 4. Run clarity check → 5. Execute Claude → 6. Commit changes → 7. Create PR (optional) → 8. Auto-review loop (optional) → 9. Post summary to JIRA

**Auto-Review Loop** (with `--auto-review` flag):
1. Fetch PR diff → 2. Run Claude to review code (JSON feedback) → 3. Parse feedback by priority → 4. Address critical/high/medium issues → 5. Commit & push fixes → 6. Repeat up to N iterations (default: 5) or until approved

**PR Review Handling:**
1. Webhook receives review → 2. Check bot mention → 3. Queue review → 4. Switch worktree to PR branch → 5. Fetch comments → 6. Run Claude → 7. Commit fixes → 8. Push & reply

### Configuration

**Environment Variables (.claude-intern/.env):**
- `JIRA_BASE_URL`, `JIRA_EMAIL`, `JIRA_API_TOKEN` - JIRA credentials
- `GITHUB_TOKEN` or `GITHUB_APP_ID` + `GITHUB_APP_PRIVATE_KEY_PATH` - GitHub auth
- `BITBUCKET_TOKEN` - Bitbucket auth
- `WEBHOOK_SECRET` - GitHub webhook verification
- `CLAUDE_INTERN_OUTPUT_DIR` - Output directory (default: `/tmp/claude-intern-tasks`)

**Project Settings (.claude-intern/settings.json):**
```json
{
  "projects": {
    "PROJ": {
      "inProgressStatus": "In Progress",
      "todoStatus": "To Do",
      "prStatus": "In Review"
    }
  }
}
```

### Output Structure

```
{output-dir}/{task-key}/
├── task-details.md                      # Formatted task for Claude
├── feasibility-assessment.md            # Clarity check results
├── implementation-summary.md            # Success output
├── implementation-summary-incomplete.md # Failure output
├── auto-review-summary.json             # Auto-review loop results
├── iteration-{N}/                       # Auto-review iteration artifacts
│   ├── feedback.json                    # Structured review feedback
│   └── review-prompt.txt                # Prompt sent to Claude
└── attachments/                         # JIRA attachments
```

## Testing

- Uses Bun's native test runner (`bun:test` API)
- Tests in `tests/` directory use isolated temp directories for parallel execution
- Import from `bun:test`: `describe`, `test`, `expect`, `beforeEach`, `afterEach`
- Use `beforeEach`/`afterEach` for setup/cleanup to enable parallel test runs

## Key Implementation Details

- **Runtime**: Bun (required for bun:sqlite in webhook queue)
- **Git branches**: `feature/{task-key-lowercase}` naming convention
- **Claude execution**: Spawns subprocess with `-p --dangerously-skip-permissions`
- **JIRA integration**: Posts summaries in Atlassian Document Format
- **Webhook isolation**: Sequential queue + single reusable worktree at `/tmp/claude-intern-review-worktree/`
  - Automatically cleans up stale worktree registrations from old paths (e.g., `.claude-intern/review-worktree/`)
- **Dependency installation**: Auto-detects package managers (bun/pnpm/npm/poetry/etc.) when preparing worktrees

---
> Source: [danii1/claude-intern](https://github.com/danii1/claude-intern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-14 -->
