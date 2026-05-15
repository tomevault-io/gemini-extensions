## feishu-claude-bridge

> When a user clones this project and asks you to help set it up, follow these steps.

# CLAUDE.md — Setup Guide for AI Assistants

When a user clones this project and asks you to help set it up, follow these steps.

## What This Is

Feishu Claude Bridge — a Node.js daemon that connects Feishu/Lark to Claude Code CLI. Users chat with a bot in Feishu, the daemon calls Claude Code, and streams responses back as real-time cards.

## Prerequisites

- **Node.js >= 20** (`node --version`)
- **Claude Code CLI** installed and logged in (`claude --version`)
- **Feishu enterprise self-built app** (see Feishu App Setup below)

## Install & Build

```bash
cd /path/to/feishu-claude-bridge
npm install
npm run build
```

`npm run build` uses esbuild to bundle `src/` → `dist/daemon.mjs`.

## Configure

```bash
cp config.env.example config.env
```

Edit `config.env`:

```bash
# ── Required ──
CTI_FEISHU_APP_ID=cli_xxxxxxxxxx
CTI_FEISHU_APP_SECRET=xxxxxxxxxxxxxxxx
CTI_DEFAULT_WORKDIR=/path/to/your/project

# ── Optional ──
CTI_FEISHU_DOMAIN=feishu            # "feishu" or "lark"
CTI_DEFAULT_MODE=code               # code / plan / ask
CTI_FEISHU_REQUIRE_MENTION=true     # Require @bot in group chats
# CTI_FEISHU_ALLOWED_USERS=ou_xxx   # Access control (comma-separated)
# CTI_AUTO_APPROVE=true             # Auto-approve all tool permissions
# CTI_CLAUDE_CODE_EXECUTABLE=/path/to/claude  # Override CLI path

# ── Third-party API provider (optional) ──
# ANTHROPIC_API_KEY=your-key
# ANTHROPIC_BASE_URL=https://your-provider.com/v1
# ANTHROPIC_AUTH_TOKEN=your-auth-token
```

## Feishu App Setup

In the [Feishu Open Platform](https://open.feishu.cn/app):

1. Create enterprise self-built app
2. Enable **Bot** capability
3. Go to **Events & Callbacks** → select **Use persistent connection** (WebSocket)
4. Subscribe to event: `im.message.receive_v1`
5. Add scopes:
   - `im:message` — Send messages
   - `im:message.receive_v1` — Receive messages
   - `im:message:readonly` — Read messages
   - `im:resource` — Upload/download resources
   - `im:chat:readonly` — Read chat list
   - `im:message.reactions:write_only` — Typing indicator
   - `cardkit:card` — CardKit v2 streaming cards
6. Publish app version

## Start

```bash
# Daemon mode (macOS launchd, auto-restarts on crash)
bash scripts/daemon.sh start

# Check status
bash scripts/daemon.sh status

# View logs
bash scripts/daemon.sh logs

# Stop
bash scripts/daemon.sh stop
```

Or foreground for debugging: `npm run dev` or `npm start` (requires build first)

> `daemon.sh` is macOS-only (uses launchd). On other platforms, use `npm run dev` or a process manager like pm2 / systemd.

## Verify It Works

```bash
bash scripts/daemon.sh logs
```

Look for these lines in order:
1. `[ws] client ready` — REST client initialized
2. `[feishu] Started (botOpenId: ou_xxx)` — Bot identity resolved
3. `[ws] ws client ready` — WebSocket connected

Then send any message to the bot in Feishu. It should reply.

## Development

```bash
npm run typecheck              # TypeScript check (tsc --noEmit)
npm run dev                    # Foreground mode
npm run build                  # Production build

npx tsx --test src/__tests__/unit.test.ts         # 55 unit tests (no network)
npx tsx --test src/__tests__/feishu-api.test.ts    # 5 API connectivity tests
npx tsx --test src/__tests__/integration.test.ts   # 6 full integration tests
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `Cannot start: missing appId or appSecret` | Check `config.env` exists in the project root and has credentials |
| WebSocket doesn't connect | Enable "使用长连接接收事件" in Feishu dev console |
| Bot doesn't respond in group | @mention the bot, or set `CTI_FEISHU_REQUIRE_MENTION=false` |
| Permission denied / 403 | Add missing scopes in Feishu dev console and republish |
| `claude` CLI not found | Install Claude Code CLI, or set `CTI_CLAUDE_CODE_EXECUTABLE` |
| Card rendering fails | Add `cardkit:card` scope and republish |

## For Architecture & Implementation Details

See [ARCHITECTURE.md](./ARCHITECTURE.md) — read that file before making code changes.

---
> Source: [PI-33/feishu-claude-bridge](https://github.com/PI-33/feishu-claude-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-09 -->
