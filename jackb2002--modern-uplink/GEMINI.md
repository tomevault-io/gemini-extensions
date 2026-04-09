## modern-uplink

> - Do NOT add `Co-Authored-By: Claude` or any AI attribution to commits

# Uplink Source Code — Project Notes

## Commit Style
- Do NOT add `Co-Authored-By: Claude` or any AI attribution to commits
- Keep commit messages concise and professional

---

## Project Goal
Modernise the Uplink (Introversion Software, 2001) source code to build and run on modern macOS and Windows, while adding quality-of-life improvements and re-enabling removed features.

**Repo:** https://github.com/Jackb2002/modern-uplink (private)
**Local clone:** /Users/Shared/uplink-src

---

## Branch Strategy
- `master` — stable baseline, do not break
- `feature/macos-build` — getting it compiling on macOS
- `feature/toomanysecrets` — re-enabling the cheat/debug menu
- `feature/modernisation` — compiler fixes, updated deps, C++ modernisation

---

## Completed Work (feature/modernisation)

### TooManySecrets cheat menu — DONE
Re-enabled in `uplink/src/app/opengl.cpp`. F1 opens the cheat screen when a debugger is attached (macOS: `sysctl` P_TRACED check). No username requirement.

### WorldConfig — DONE
`uplink/src/game/data/worldconfig.h` — ~110 tunable gameplay constants + mission probability arrays centralised in `WorldConfig` struct. Single global `g_WorldConfig` defined in `worldgenerator.cpp`. All `data.h` `#define` constants are still there as defaults; `WorldConfig` members initialise from them. Future: load from a config/mod file.

### Mission Registry — DONE
`missiongenerator.cpp` `Initialise()` no longer has switch statements. Each type registers a `MissionTypeEntry` with generate/isComplete function pointers + probability array pointer + completionField documentation. See memory/missions.md.

**Critical Probability rule:** `Probability::Validate()` requires every DArray index from 0 to `size-1` to be valid AND sum to 100. Every mission type from 0 to the highest registered type must have a probabilities entry — even placeholder types (all-zeros). MISSION_SPECIAL (0), MISSION_REMOVECOMPANY (9) both need slots.

### SDL2 Input — DONE
`lib/gucci/gucci_sdl.cpp`: SDL2 does not fire `SDL_TEXTINPUT` for control keys. Added `SDL_KEYDOWN` handler routing backspace (`\b`), delete (127), enter (`\r`), escape (27), tab (`\t`) through `gciKeyboardHandlerP`.

### run.sh — DONE
Repo root `run.sh` launches `uplink/src/lib/uplink.full` from the correct working directory.

---

## Directory Structure

```
contrib/        Third-party libs: SDL, SDL_mixer, freetype, FTGL, libjpeg, libmikmod, zlib, unrar, tcp4u, irclib
docs/           Developer HTML documentation (start with quick source tour.html)
Installer/      Windows NSIS installer scripts and data
lib/            8 custom Introversion libraries (see below)
loki/           Linux installer config (Loki installer framework — ignore for macOS)
other/          Encrypted GameBibles / design docs
tools/          Encryption, language, memory leak, debug utilities
uplink/src/     Main game source (206 .cpp, 207 .h files)
```

---

## Internal Libraries (lib/)

| Library | Purpose |
|---|---|
| **eclipse** | UI widgets — buttons, icons, animations. Data + callbacks only, no rendering. |
| **gucci** | Graphics — OpenGL/SDL init, TrueType fonts (FTGL), image loading (TIFF/JPEG), screen modes. Has platform split: `resize_win32.cpp` / `resize_unix.cpp` |
| **soundgarden** | Audio — MOD music, SFX, playlists. Platform split: `soundgarden_win32.cpp`, `soundgarden_unix.cpp`, `soundgarden_sdlmixer.cpp` |
| **redshirt** | Encryption — save game and data file encryption/decryption, hash verification |
| **bungle** | ZIP access — game data files are renamed ZIPs |
| **tosser** | Template data structures — `LList`, `DArray`, `BTree`. Header-only, no separate compilation needed |
| **vanbakel** | Task manager — in-game running processes, priority, visual interface (uses eclipse) |
| **mmgr** | Memory debugger — tracks all allocations with file/line info, leak detection via macro interception of new/delete/malloc/free |

---

## Main Game Source (uplink/src/)

| Directory | Contents |
|---|---|
| `app/` | Core app, OpenGL renderer (`opengl.cpp` 19.5KB, `opengl_interface.cpp` 1479 lines), serialisation, globals |
| `game/` | Game state, logic, `scriptlibrary.cpp` (68.7KB — mission scripting engine) |
| `world/` | World state, player, NPCs, rating, dates, connections, companies |
| `world/computer/` | All computer types: bank, gateway, LAN, databank, security systems |
| `world/generator/` | Procedural content: missions, companies, BBSes, LANs, news, plots |
| `world/scheduler/` | Event system: arrests, missions, bank robberies, player wipes, news updates |
| `interface/` | Interface coordinator and base classes |
| `interface/localinterface/` | Local system screens: email, finance, IRC, hardware, software, map, missions |
| `interface/remoteinterface/` | Remote computer screens: BBS, accounts, logs, console, cyphers, etc. |
| `interface/taskmanager/` | 40+ hacking tool implementations: tracers, crackers, probes, nukes, etc. |
| `mainmenu/` | Main menu, login, options, game-over screens |
| `network/` | Network protocol, client/server connections |
| `view/` | Rendering, FPS display |

---

## Key Config File
`uplink/src/app/globals_defines.h` — build-time feature flags:
- `VERSION_NUMBER_INT` = "10.0"
- `SAVEFILE_VERSION` = "SAV62"
- Build mode: `FULLGAME` / `DEMOGAME` / `TESTGAME` (one must be defined)
- `CHEATMODES_ENABLED` — **uncomment to re-enable TooManySecrets**
- `DEBUGLOG_ENABLED` — enables debug logging
- `CODECARD_ENABLED` — enables code card DRM check
- `VERIFY_UPLINK_LEGIT` — enables legitimacy check

---

## Platform / Build Notes

### Original build targets
- **Windows:** Visual Studio 2005 (`uplink/src/Uplink.sln`)
- **Linux:** Makefiles + autoconf (`BUILDING.linux`)

### Platform split pattern used throughout
```cpp
#ifdef WIN32
    // Windows-specific code
#else
    // Unix/Linux code
#endif
```

Files with platform splits: `gucci/`, `soundgarden/`, `uplink.cpp` (exception handling), `options.cpp`, various interface files.

### macOS Build Strategy (use feature/macos-build branch)
Target: use the Linux Makefile path, substituting Homebrew deps.

Required Homebrew packages:
```bash
brew install sdl2 sdl2_mixer freetype jpeg libxml2
```
FTGL and GLUT may need sourcing — check Homebrew or build from contrib/.

Key issues to resolve:
1. Update `standard.mk` and `uplink/src/Makefile` to use Homebrew include/lib paths
2. Fix modern compiler errors (clang is stricter than gcc 2001-era)
3. OpenGL is deprecated on macOS but still works — suppress warnings for now
4. `soundgarden_sdlmixer.cpp` is the right audio backend for macOS
5. GTK1 (loki installer) — **ignore entirely**, not needed for the game itself

### Common Modern Compiler Issues to Expect
- Non-const string literals assigned to `char*`
- Implicit `int` return types
- Old-style casts
- `register` keyword (removed in C++17)
- VLAs (variable-length arrays) — not standard in C++
- `#include` ordering issues

---

## Audio
`soundgarden` has three backends — use `soundgarden_sdlmixer.cpp` for macOS/Linux. Win32 backend uses DirectSound.

---

## Save Files
Encrypted with `redshirt` library. Format version `SAV62`, backwards compatible to `SAV56`. Do not change serialisation without bumping `SAVEFILE_VERSION`.

---

## Scripting / Missions
`game/scriptlibrary.cpp` (68.7KB) contains the mission scripting engine. New mission types go here. This is the main file for adding content/variety.

---

## Notes on Modernisation Priorities
1. ~~Get it compiling on macOS~~ — done
2. ~~Re-enable TooManySecrets~~ — done (debugger-attached F1)
3. Fix compiler warnings/errors for clean build
4. SDL2 is already in use (contrib has old SDL1, but build uses Homebrew SDL2)
5. New mission variety — extend `missiongenerator.cpp` using registry pattern
6. WorldConfig mod/config file loading — infrastructure is ready
7. Longer term: replace deprecated OpenGL fixed-function with modern pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jackb2002)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jackb2002)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
