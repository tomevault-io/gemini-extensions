## discord-messages-reader

> discord/__init__.py       — Main AppModule (stable Discord)

# Discord Messages Reader — Claude Code Notes

## Project Layout

```
appModules/
  discord/__init__.py       — Main AppModule (stable Discord)
  discordptb/__init__.py    — Re-exports AppModule for Discord PTB
  discordcanary/__init__.py — Re-exports AppModule for Discord Canary
tests/
  conftest.py               — NVDA stub installation + app_module fixture
  test_filter.py            — _filterAndAnnounce unit tests
  test_announce.py          — _scheduleAnnounce / _doAnnounce unit tests
  test_uia.py               — _getLatestMessageViaUIA mock tests
  test_smoke.py             — Lifecycle and event handler smoke tests
manifest.ini                — NVDA add-on manifest
build.py                    — Creates dist/*.nvda-addon (ZIP)
pytest.ini                  — Runs tests from tests/ directory
```

## Build and Test

```bash
python build.py             # produces dist/discord_messages_reader-X.X.X.nvda-addon
pytest                      # 131 tests, all should pass
```

Pytest is installed at:
`C:\Users\<username>\AppData\Roaming\Python\Python313\Scripts\pytest.exe`

## Installed Location (for rapid iteration)

```
C:\Users\<username>\AppData\Roaming\nvda\addons\discord_messages_reader\appModules\discord\__init__.py
```

After editing, deploy with:
```bash
cp appModules/discord/__init__.py \
   "$APPDATA/nvda/addons/discord_messages_reader/appModules/discord/__init__.py"
```
Then restart NVDA (Ctrl+Alt+N) to reload.

## Key Design Decisions

- **UIA polling (500ms)** is the primary message detection mechanism.
  IAccessible WinEvent hooks are unreliable in Discord's Chromium renderer
  when focus is in the edit field.
- **WinEvent hook** is kept only as a fast-path trigger and for `_lastHookTime`
  (used to suppress spurious `event_valueChange` when Discord clears the edit
  field after sending a message). It is debounced to avoid stacking UIA reads
  during navigation.
- **`core.callLater`** is used for all timer scheduling — it is thread-safe
  (internally posts to the main thread), so `_schedulePoll` can be called from
  any thread, including the Dummy-N worker thread NVDA uses when Discord
  launches while NVDA is already running.
- **Message list caching** (`_cachedMsgList`/`_cachedMsgListName`): the
  expensive `FindAll` UIA tree walk is skipped on subsequent polls when the
  element is still valid. The cache is invalidated on COM errors or when the
  element name changes (channel switch). Implemented in `_getMsgListViaUIA`.
- **`disableBrowseModeByDefault = True`** suppresses NVDA's virtual buffer for
  Discord, which is the recommended approach for Electron apps.
- **Message dedup** is content-based only (`_lastText`). No time window — the
  UIA text includes a timestamp so genuinely new messages always differ.
- **Foreground guard** in `_uiaRead` prevents announcements when Discord is
  not the active window.

## Log Level

The add-on uses `log.debug` for per-message lines (UIA read, announcing,
speaking) so they do not appear in users' INFO-level logs. Load/terminate
events and errors use `log.info` / `log.warning`.

## Discord PTB / Canary

`discordptb/__init__.py` and `discordcanary/__init__.py` each contain a
single line: `from discord import AppModule`. NVDA matches add-on AppModules
by executable name (without extension, lowercase).

## Supported Discord Formats

IAccessible name format: `"username , body , HH:MM AM"`
UIA plain-text format: the name IS the message body (less common path)

Both are handled by `_filterAndAnnounce` and `_doAnnounce`.

---
> Source: [blindndangerous/discord-messages-reader](https://github.com/blindndangerous/discord-messages-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-27 -->
