# ESP3D + Marlin — Wireless Control via ESP32

Guide for connecting an ESP32-C3 module running [ESP3D](https://github.com/luc-lebosse/ESP3D) firmware to the BTT SKR Mini E3 V3.0 board for WiFi-based printer control.

## Marlin configuration (already applied)

The following changes are committed in `../Marlin/Configuration.h`:

| Setting | Value | Purpose |
|---|---|---|
| `SERIAL_PORT` | `-1` | USB — primary serial (PC, OctoPrint, etc.) |
| `SERIAL_PORT_2` | `2` | USART2 — dedicated channel for ESP3D |
| `BAUDRATE_2` | `250000` | Must match the baud rate configured in ESP3D |

Marlin accepts G-code from both ports simultaneously, so USB and WiFi work in parallel.

> **Do not enable** `ESP3D_WIFISUPPORT` or `WIFISUPPORT` in `../Marlin/Configuration_adv.h` — those are meant for boards where the ESP32 is the main MCU. Here the STM32G0B1 runs Marlin and the ESP32-C3 is just a serial-to-WiFi bridge.

## ESP3D configuration (already applied)

The ESP3D config lives in `esp3d/configuration.h` (relative to this directory). Key settings:

- **MCU:** ESP32-C3, 4 MB flash
- **Protocol:** `RAW_SERIAL` (plain serial passthrough)
- **Target firmware:** `MARLIN`
- **Serial output:** `USE_SERIAL_1` (connected to Marlin USART2)
- **WiFi TX power:** `WIFI_POWER_5dBm` (low power — sufficient for short-range use inside the enclosure)
- **Features enabled:** HTTP web UI, Telnet, WebSocket, OTA updates, mDNS/SSDP discovery, notifications
- **Reset pin:** GPIO 3 (hold low for >1 s at boot to factory-reset ESP3D settings)

WiFi credentials are stored in `myconfig.h` (git-ignored) or set via the ESP3D web UI after first boot in AP mode.

## Wiring (ESP32-C3 to SKR Mini E3 V3.0)

All signals are 3.3 V logic — no level shifter needed.

| ESP32-C3 | SKR Mini E3 V3.0 | Notes |
|---|---|---|
| TX | RX (USART2) | PA3 on STM32G0B1 |
| RX | TX (USART2) | PA2 on STM32G0B1 |
| GND | GND | Common ground |
| 3.3 V | 3.3 V | Or use a separate regulator |

Refer to `../docs/btt_pinout.pdf` and `../docs/btt_schematic.pdf` for the exact header location. On the SKR Mini E3 V3.0, USART2 pins (PA2/PA3) are typically exposed on the TFT/WiFi header.

## Flashing

1. **Marlin** — build and flash `../Marlin/Configuration.h` + `../Marlin/Configuration_adv.h` onto the SKR Mini E3 V3.0 via SD card or DFU.
2. **ESP3D** — build this project using PlatformIO (`platformio.ini`) and flash onto the ESP32-C3 via USB or OTA.

## Verification

After powering on:

1. **USB** still works as before (Pronterface, OctoPrint, etc.).
2. **WiFi** — connect to the ESP3D access point (first boot) or your local network, then open the ESP3D web UI to send G-code.
3. Both channels can send commands simultaneously; Marlin handles arbitration internally.

## Troubleshooting

- **No response on USART2** — verify TX/RX are not swapped; double-check with `../docs/btt_pinout.pdf`.
- **Baud mismatch** — `BAUDRATE_2` in Marlin and the serial baud in ESP3D must be identical (currently both `250000`). You can change the baud at runtime with `M575` if `BAUD_RATE_GCODE` is enabled in Marlin.
- **Fallback port** — if USART2 is unavailable on your board revision, try `SERIAL_PORT_2 1` (USART1). Note: USART1 may conflict with the EXP1 display connector.
