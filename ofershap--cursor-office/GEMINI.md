## state-machine

> Working: `typing | editing | running | searching | reading` → character walks to desk (fast: 5.0), sits, types.

# Agent State Machine & Idle Behavior

## Activity States
Working: `typing | editing | running | searching | reading` → character walks to desk (fast: 5.0), sits, types.
`phoning` → character walks to PHONE_COL/PHONE_ROW, stands facing back, left arm holds handset. Triggers when agent spawns subagent (`Task` tool).
`idle` → autonomous waypoint patrol.
`celebrating` → jumps in center, auto-expires after ~4 seconds, then idle.
`error` → shows ⁉️ bubble (4s), walks to idle area, then auto-transitions to idle.

## Celebration Rules
- Only triggers on `stop` hook with `status: "completed"` AND the session had "work" tools (editing/running/writing). Read-only or text-only sessions go straight to idle.
- `cursorWatcher.ts` tracks `didWork` flag — set true on editing/running/writing tools, reset on idle/stop.
- Auto-expires: after `animFrame > 16` (~4 seconds), transitions to idle.
- Interrupted by work: any new work activity cuts celebration immediately.

## State Transitions (setActivity)
- Any working state: sets target to DESK pos. Character rushes there.
- `phoning`: sets target to PHONE pos. Phone object starts vibrating.
- `idle`: resets `idleActionTimer` to 0 (immediately picks next waypoint).
- `celebrating`: sets target to center (CELEBRATE_COL/ROW), resets animFrame.
- `error`: shows ⁉️ bubble, sets target to idle area. After bubble expires → auto-idle.

## Detection Modes (cursorWatcher.ts)
Two modes, auto-selected on startup:

### Mode 1: Transcript Parsing (default, zero setup)
- Watches `~/.cursor/projects/*/agent-transcripts/*.jsonl`.
- Reads last lines → infers activity from content heuristics.
- Idle timeout: **8 seconds** of no file changes.
- User lines (`"role":"user"`) → immediate idle (turn is over).

### Mode 2: Cursor Hooks (opt-in, precise)
- Activated via command palette: "Cursor Office: Enable Hooks".
- Installs hooks into `~/.cursor/hooks.json` (see extension-architecture.mdc).
- Hook script writes to `/tmp/cursor-office-state.json`, extension watches that file.
- Events: `preToolUse` → activity, `stop` → celebrate/idle, `beforeSubmitPrompt` → idle, `subagentStart` → phoning, `subagentStop` → back to work.
- Hooks are global — fires for ALL chats across ALL workspaces.

## Working State Persistence
`isWorking()` returns true for all 5 working activities.
While working + at desk, speech bubble auto-cycles "Working." dots (0.5s timer).
While phoning + at phone, speech bubble auto-cycles "Delegating." dots (0.5s timer).
`isAtDesk()` and `isAtPhone()` tolerance is 0.5 tiles.

## Idle Behavior
`IDLE_WAYPOINTS` array - sequential list of (position, action, duration).
Character visits them in order, wrapping around. Each has a speech bubble.
Between actions, returns to center (3, 1.8) with `'stand'` action.
Walk speed when idle: 1.5 (leisurely). When heading to work: 5.0 (urgent).

## Click-to-Attract
`attractToObject(char, objectId)` - only works when idle.
Delays 0.8-2.0 seconds (setTimeout) before moving character to object.
Uses `OBJECT_POSITIONS` map which must match `IDLE_WAYPOINTS` coords.

## Hitboxes
Pixel-space, UNSCALED. Each object has `{ w, h }`.
Hit test in `hitTest.ts`: `objX = col * TILE_SIZE * scale`, then checks if mouse is within `w * scale` and `h * scale`.
Common pitfall: making hitboxes too small. Cat/arcade/plant all needed 1.5-2x enlargement.

---
> Source: [ofershap/cursor-office](https://github.com/ofershap/cursor-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-03 -->
