## xhs-autopilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**xhs-autopilot** — Xiaohongshu (小红书) content creation autopilot. Automates the full workflow from competitor research → content writing → cover image generation → publishing via MCP tools.

## Setup

```bash
uv sync                              # install dependencies (pillow, playwright)
uv run playwright install chromium   # first-time only: install browser for screenshots
```

## Common Commands

```bash
# Cover generation (Pillow — basic)
uv run python scripts/generate_cover.py --title "标题" --template gradient --color warm

# Cover generation (HTML+Playwright — high quality, preferred)
uv run python scripts/screenshot_cover.py --html scripts/cover_template.html --output workspace/<run_id>/cover.png
```

## Architecture

### Cover Image Pipeline (two approaches)

1. **Pillow** (`scripts/generate_cover.py`): Programmatic image generation. Four templates (`gradient`, `minimal`, `list`, `bold`) × four color schemes (`warm`, `cool`, `green`, `neutral`). Output: 1242×1660px (3:4 portrait). Good for batch generation with dynamic text.

2. **HTML+Playwright** (`scripts/cover_template.html` + `scripts/screenshot_cover.py`): Edit the HTML template, then screenshot to PNG. Produces much higher visual quality (glassmorphism, gradients, animations frozen at capture). Preferred for polished covers.

### MCP Integration

The project connects to Xiaohongshu via the [xiaohongshu-mcp](https://github.com/xpzouying/xiaohongshu-mcp) MCP Server (external binary, not included in this repo — download from [Releases](https://github.com/xpzouying/xiaohongshu-mcp/releases)). Available MCP tools:
- `search_feeds` — search notes with sort/filter (综合/最多点赞/最多收藏/最新) and time range
- `get_feed_detail` — note detail + comments (unreliable: often fails with "not found in noteDetailMap"; search results alone usually suffice)
- `publish_content` — publish image+text notes; `images` accepts local absolute paths, `tags` accepts string arrays without `#`
- `publish_with_video` — publish video notes
- `user_profile`, `like_feed`, `favorite_feed`, `post_comment_to_feed`, `reply_comment_in_feed`
- `check_login_status` / `get_login_qrcode` / `delete_cookies` — auth management

### Skill: `/xhs-creator`

Defined in `.claude/skills/xhs-creator/SKILL.md`. Five-phase workflow:
1. **Research** — 3-dimension search (most liked, most saved, latest)
2. **Analysis** — pattern extraction (titles, structure, tags, comments)
3. **Generate** — title (≤20 chars) + body (600-900 chars safe zone, platform max ~1000) + 5-8 tags + cover image
4. **Review** — human approval with AskUserQuestion
5. **Publish** — immediate or scheduled via `publish_content`

## Platform Constraints

- **Title**: max 20 Chinese characters
- **Body**: ~1000 char limit; emoji may count as multiple chars; target 600-900
- **Cover image**: 1242×1660px (3:4), max 5MB (auto-converts to JPEG if exceeded)
- **Tags**: passed via `tags` parameter as plain strings (no `#` prefix)

## Rules

- **Keep docs in sync**: After modifying code (new features, changed behavior, updated paths, etc.), always check whether `CLAUDE.md` and `README.md` need corresponding updates.

## Font Handling (macOS)

PingFang.ttc location varies across macOS versions. On macOS 15+ (Sequoia) it lives under `/System/Library/AssetsV2/...` — the code uses `fc-list` to discover it at runtime. Never hardcode the path.

---
> Source: [Jinsong-Zhou/xhs-autopilot](https://github.com/Jinsong-Zhou/xhs-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->
