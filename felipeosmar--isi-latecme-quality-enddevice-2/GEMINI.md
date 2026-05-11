## isi-latecme-quality-enddevice-2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Flash

Requires ESP-IDF v5.5.3. Always source the environment first:

```bash
. /home/felipe/.espressif/v5.5.3/esp-idf/export.sh
idf.py build
```

**Two firmware variants** are built from this single codebase via `CONFIG_THERMOCOUPLE_ENABLED`:

| Variant | Build command | Enables |
|---|---|---|
| Standard | `idf.py build` | I2C temp/humidity only |
| Salt spray | `idf.py -DSDKCONFIG_DEFAULTS="sdkconfig.defaults;sdkconfig.defaults.salt_spray" build` | + MAX6675 thermocouple |

**Local variant switching:** The committed `sdkconfig` reflects the standard variant (`CONFIG_THERMOCOUPLE_ENABLED=n`). To build the salt spray variant locally, delete `sdkconfig` first and use the `-DSDKCONFIG_DEFAULTS` flag above. CI always uses fresh sdkconfig from defaults.

**Existing salt_spray devices in the field:** Devices with `auto_update_branch = "salt_spray"` will stop receiving updates after the Kconfig migration (no `salt_spray-rN` releases exist). Change `auto_update_branch` to `"main"` via the web UI (Config → Auto-Update → Branch). The auto-updater will then download `lorawan-enddevice-main-rN-salt_spray.bin` automatically based on the compiled variant.

Flash modes (via `./flash.sh`):
- `./flash.sh update` — firmware + web UI, preserves user config (most common)
- `./flash.sh app` — firmware only
- `./flash.sh www` — web interface only
- `./flash.sh all` — factory reset (erases userdata partition)

Monitor: `idf.py -p /dev/ttyUSB0 monitor` (exit with Ctrl+])

Set `PORT` and `BAUD` env vars for non-default serial port/baud rate.

**No test suite exists.** There is no unit testing infrastructure.

## Architecture

ESP32 LoRaWAN sensor node: reads I2C sensors + MAX6675 thermocouple, encodes as CayenneLPP, sends via LoRaWAN Class A OTAA to ChirpStack. Web interface for config/monitoring.

### FreeRTOS Tasks (created in main.c)

| Task | Core | Pri | Stack | Role |
|------|------|-----|-------|------|
| sensor | 0 | 5 | 6KB | Periodic I2C + MAX6675 reads |
| lorawan | 1 | 6 | 8KB | OTAA join, radio session (C++ RadioLib) |
| uplink | 0 | 4 | 4KB | Build CayenneLPP payload, send via LoRaWAN |
| display | 0 | 3 | 4KB | OLED page cycling (sensors/system) |

### Init Sequence (main.c → app_main)

log_buffer → buzzer → config (LittleFS) → WiFi (AP/STA) → web server → health monitor → FreeRTOS tasks

### Data Flow

Sensors → `sensor_data_t` (mutex-protected) → uplink task reads it → CayenneLPP encode → `lorawan_send()`

CayenneLPP channels: 1=I2C temperature, 2=I2C humidity, 4=thermocouple temperature.

### Key Boundaries

- **C/C++ boundary**: `lorawan_handler.cpp` wraps RadioLib (C++) with `extern "C"` API. `EspHal.h` is the ESP32 HAL for RadioLib's SPI/GPIO.
- **Config persistence**: JSON in LittleFS on `userdata` partition (64KB). This partition is NOT erased by `idf.py flash` — survives firmware updates. Changing code defaults does NOT update saved config on device; use config migration in `config_manager.c`.
- **Two LittleFS partitions**: `www` (192KB, web UI files, reflashed with firmware) and `userdata` (64KB, user config, preserved across updates).

## Code Organization

```
main/
├── main.c                    # Entry point, task creation, init sequence
├── lorawan/                  # RadioLib C++ wrapper + ESP32 HAL
├── sensors/                  # sensor_manager (auto-detect) + 4 drivers
│   ├── sensor_manager.c      # I2C bus init, probe 0x40→0x44→0x38, periodic reads
│   ├── sht20_driver.c        # I2C 0x40
│   ├── sht3x_driver.c        # I2C 0x44 (SHT30/SHT31/SHT40)
│   ├── am2315c_driver.c      # I2C 0x38 (AM2315C/AHT20)
│   └── max6675_driver.c      # SPI bit-bang thermocouple (GPIO 33/27/32)
├── config/config_manager.c   # JSON config load/save/getters, thread-safe
├── webserver/                # HTTP server + handlers/ (8 API handler files)
├── payload/cayenne_lpp.c     # CayenneLPP encoder (max 51 bytes)
├── display/oled_display.c    # SSD1306 128x64 OLED
├── wifi/wifi_manager.c       # AP/STA mode management
├── health/health_monitor.c   # Watchdog, heap monitoring
├── logs/log_buffer.c         # Ring buffer capturing ESP_LOG output
├── interface/buzzer.c        # PWM buzzer (GPIO4)
└── www/                      # Static web files (HTML/CSS/JS) → www partition
```

## Hardware Constraints (JVTECH v1.2 Board)

**Critical pin restrictions:**
- GPIO 16/17: **crash when configured as output** — never use
- GPIO 36/39: suffer WiFi interference — avoid for sensors

**Pin assignments (hardcoded in various drivers):**
- LoRa SX1276 SPI: SCK=18, MISO=19, MOSI=23, NSS=5, RST=14, DIO0=26, DIO1=13
- I2C bus (sensors + OLED): SDA=21, SCL=22
- MAX6675 (software SPI, avoids HW SPI conflict): SCK=32, SO=35, CS=33
- Buzzer: GPIO4 (via BC817 transistor, active HIGH)

## ESP-IDF Gotchas

- **sdkconfig.defaults** only applies when `sdkconfig` does not exist. `idf.py fullclean` removes `build/` but NOT `sdkconfig`. To apply sdkconfig.defaults changes: delete `sdkconfig` manually, then rebuild.
- **Interrupt WDT** (TG1WDT): timeout increased to 800ms in sdkconfig.defaults (default 300ms is too short for panic backtrace). This is the IWDT, not the task WDT.
- **Coredump to flash** is enabled — use `idf.py coredump-info` for post-mortem debugging.

## Web API Pattern

All endpoints under `/api/`. Auth via Basic Auth (config-controlled). Handlers are in `main/webserver/handlers/`. To add a new endpoint: create handler in `api_*.c`, declare in `handlers.h`, register in `web_server_init()`.

## Adding New Components

**New sensor**: Create `main/sensors/newsensor_driver.c/h` → add to `sensor_manager.c` auto-detect → add to `CMakeLists.txt` SRCS → update uplink payload channels.

**New config field**: Add to `config_t` struct + getter in `config_manager.c/h` → add default in `config_reset_defaults()` → add JSON read/write in load/save internals → update `config.json`.

---
> Source: [felipeosmar/isi-latecme-quality-enddevice-2](https://github.com/felipeosmar/isi-latecme-quality-enddevice-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-30 -->
