## javabox

> This file describes the codebase structure, key constraints, and conventions for AI agents working in this repository.

# JavaBox — Agent Guidance

This file describes the codebase structure, key constraints, and conventions for AI agents working in this repository.

## What This Project Is

JavaBox runs JDK 21 in the browser via WebAssembly. It supports two execution modes:

1. **Direct WASM** (fast): OpenJDK Zero interpreter compiled directly to WASM via Emscripten. Architecture: Browser → Emscripten WASM (OpenJDK Zero C++ interpreter) → JVM → CompileServer. Target: ~3-5s boot.

2. **QEMU mode** (legacy): Container2wasm pipeline. Architecture: Browser → Emscripten WASM (QEMU) → TCG JIT → Linux kernel → JVM. Current: ~55s boot.

Both modes expose the same CompileServer protocol (JBOX_PING/JBOX_COMPILE/JBOX_DOOM) and the same TypeScript SDK API.

## Repository Layout

```
openjdk/            Forked OpenJDK 21u with Emscripten OS layer (branch: wasm-emscripten)
  src/hotspot/os/emscripten/     Emscripten OS abstraction (~15 files)
  src/hotspot/os_cpu/emscripten_zero/  Zero CPU port for Emscripten (~11 files)
container/          Alpine Linux 3.21 + JDK 21 (Dockerfile) — QEMU mode
build/
  build-direct.sh   Orchestrate full direct WASM build
  build-jvm-wasm.sh Compile OpenJDK Zero → libjvm.a via Emscripten
  build-jdk-modules.sh  jlink JDK class library on host
  build-doom-jar.sh     Compile Doom Java sources on host
  link-direct-wasm.sh   Final emcc link → javabox-direct.js + .wasm + .data
  jvm-stdin-bridge.c    SharedArrayBuffer stdin ring buffer for direct mode
  convert.sh        c2w conversion — QEMU mode
  serve-config.json, serve.py  Local serving with COOP/COEP headers
web/
  index.html        Main UI — dual mode (?mode=direct or ?mode=qemu)
  doom.html         Doom frontend — dual mode
packages/
  sdk/              TypeScript SDK (@javabox/sdk) — compile/run/fs API
examples/           Usage examples
tests/              Playwright e2e tests (support both modes via JAVABOX_MODE env)
PLAN.md             Full project specification and phase breakdown
```

## Container Build

- Base image: `oraclelinux:9-slim` from Docker Hub (no auth required)
- JDK: Eclipse Temurin JDK 21 installed via tarball, then shrunk with `jlink`
- Must always build with `--platform linux/amd64`
- Container runtime: **Rancher Desktop** (`~/.rd/bin/docker`) — NOT Docker Desktop, NOT plain `docker`
  - `nerdctl` does not work here because Rancher Desktop is running the `dockerd` (moby) engine, not containerd
  - Use `~/.rd/bin/docker` or add `~/.rd/bin` to `$PATH`

Build command:
```sh
~/.rd/bin/docker build --platform linux/amd64 -t javabox:latest ./container/
```

Smoke test:
```sh
~/.rd/bin/docker run --rm --platform linux/amd64 javabox:latest bash -c 'java -version && javac -version'
```

## WASM Conversion (c2w) — Two Modes

`build/convert.sh` auto-detects the platform, or accepts `--mode arm-mac|linux`.

The output (`out.wasm`, `out.js`, `out.data`) is a **static artifact** — only re-run conversion when `container/Dockerfile` changes.

### `--mode arm-mac` (Apple Silicon macOS)

Uses a Colima x86_64 VM (`brew install colima`). Slow on first run (~20–40 min); VM is reused on subsequent runs.

```sh
./build/convert.sh --mode arm-mac ~/javabox-wasm
```

**Why Colima, not QEMU user-mode:**
- c2w's BuildKit compiles `runc` using Go 1.24 for amd64
- Go 1.24's GC (`gcBgMarkWorker`) crashes under QEMU **user-mode** emulation on ARM
- `tonistiigi/binfmt --install all` does NOT fix this
- Colima with `--arch x86_64` runs a full x86_64 VM (QEMU full system emulation) — Go runs natively inside it, no crash

Colima VM profile name: `javabox-x86` (created automatically by the script).

### `--mode linux` (Linux x86_64 — fast, for CI pipelines)

Uses the native c2w binary. Fast (~5–10 min). Use in CI pipelines and on Linux build machines.

```sh
# Install c2w once
wget -qO- https://github.com/container2wasm/container2wasm/releases/download/v0.8.3/container2wasm-v0.8.3-linux-amd64.tar.gz \
  | tar xz && sudo mv c2w c2w-net /usr/local/bin/

./build/convert.sh --mode linux /tmp/javabox-wasm
```

## TypeScript SDK (`packages/sdk/`)

- Entry: `packages/sdk/src/index.ts`
- Types: `packages/sdk/src/types.ts` — all public interfaces
- Key class: `packages/sdk/src/container.ts` — `JavaContainer.boot()`, `compile()`, `run()`, `exec()`
- Build: `cd packages/sdk && npm run build` (uses tsc)
- Type-check only: `cd packages/sdk && npx tsc --noEmit`

**Open questions that must be resolved empirically against a live WASM build:**
- `EMSCRIPTEN_WORKSPACE_PATH` in `container.ts` — the exact path for the virtio-9p shared mount
- Boot sentinel detection — confirm `/etc/profile.d/javabox-ready.sh` runs on QEMU Wasm startup
- PTY strategy — `process.ts` has two stub strategies (A: script+sentinel, B: xterm-pty)

## Serving the WASM Output

The WASM output requires these HTTP headers (SharedArrayBuffer dependency):
```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

For local dev: `npx serve <output-dir> --config build/serve-config.json`

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Base OS | `oraclelinux:9-slim` | Matches project spec; small; no auth needed |
| JDK | Eclipse Temurin 21 via tarball | `java-21-openjdk-devel` pulls 200+ GUI deps on OL9 |
| jlink compression | `--compress=zip-6` | `--compress=2` deprecated in JDK 21; `--strip-debug` requires objcopy |
| c2w mode | `--to-js` (QEMU Wasm/TCG) | Full x86_64 Linux emulation in browser |
| Networking default | `c2w-net-proxy.wasm` | HTTP/S only via browser Fetch, no relay needed |
| Container runtime | Rancher Desktop `dockerd` | User preference; nerdctl not available (containerd not running) |

## What Not To Do

- Do not run `docker build` without `--platform linux/amd64` — results in aarch64 packages on Apple Silicon
- Do not use `--strip-debug` in jlink — requires `objcopy` (binutils) which is not installed
- Do not attempt c2w conversion on macOS/Apple Silicon — it will fail at the runc build step
- Do not change the `CMD` in the Dockerfile away from `["/bin/bash", "--login"]` — the `--login` flag is required for `/etc/profile.d/javabox-ready.sh` to execute at container start

---
> Source: [bmarti44/javabox](https://github.com/bmarti44/javabox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->
