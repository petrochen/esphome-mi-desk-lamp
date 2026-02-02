# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub:** https://github.com/petrochen/esphome-mi-desk-lamp

## Project Overview

ESPHome firmware for **Xiaomi Mi Desk Lamp (MJTD01YL)** — smart desk lamp with rotary encoder, button, dual-color (warm/cold white) LED control, **circadian lighting** based on sunrise/sunset, and **hourly time reminders**.

## Hardware

- **MCU:** ESP8266 (esp01_1m, 1MB flash, ~40KB free RAM)
- **Button:** GPIO2 (inverted, pull-up)
- **Rotary Encoder:** GPIO13 (A), GPIO12 (B)
- **Cold White LED:** GPIO4 (PWM 1000Hz)
- **Warm White LED:** GPIO5 (PWM 1000Hz)

Note: ESP8266 has internal temperature sensor but it measures chip temperature (~60°C), not ambient. External sensor (DHT22, DS18B20) needed for room temperature.

Reference: https://templates.blakadder.com/mi_desk_lamp.html

## Commands

```bash
# Compile firmware
esphome compile mi-desk-lamp.yaml

# Upload via serial (FTDI) - first time only
esphome run mi-desk-lamp.yaml --device /dev/cu.usbserial-0001

# Upload via OTA (WiFi) - preferred
esphome upload mi-desk-lamp.yaml --device 192.168.2.53

# View logs (requires enabling logger first)
esphome logs mi-desk-lamp.yaml --device OTA
```

## Serial Flashing (First Time)

1. Connect FTDI adapter: TX→U0RX, RX→U0TX, GND→GND
2. Power lamp from its original PSU (not FTDI 3.3V)
3. Hold GPIO0 to GND while powering on to enter bootloader
4. Run `esphome run mi-desk-lamp.yaml --device /dev/cu.usbserial-0001`

## Network

- **IP:** 192.168.2.53
- **mDNS:** mi-desk-lamp.local
- **Web UI:** http://192.168.2.53 (port 80)
- **API Port:** 6053 (Home Assistant)
- **OTA Port:** 8266
- **Fallback AP:** Mi-Desk-Lamp-AP / 12345678

## Lamp Controls

| Action | Result |
|--------|--------|
| Rotate encoder | Adjust brightness (configurable step, default 10%) |
| Hold button + rotate | Adjust color temperature (disables circadian) |
| Click button | Toggle on/off |
| Double-click button | Reset to circadian mode |

## Circadian Lighting

Automatic color temperature based on sun position (Lisbon: 38.7223°N, -9.1393°W).

| Period | Temperature | Effect |
|--------|-------------|--------|
| Night → Sunrise | 2700K | Warm, sleep-friendly |
| Sunrise → +2h | 2700K → 5500K | Gradual wake-up |
| Day | 5500K → 6500K | Cool, alertness |
| Sunset -2h → Sunset | 6500K → 3500K | Evening wind-down |
| Sunset → +2h | 3500K → 2700K | Melatonin-friendly |

**How it works:**
- SNTP syncs time (Europe/Lisbon timezone)
- Sun component calculates sunrise/sunset astronomically from coordinates
- On light turn-on: applies temperature based on current time phase
- Manual override disables circadian until double-click or web toggle

## Hourly Time Reminders

When lamp is on:
- **:00** (every hour) — 2 blinks
- **:30** (every half hour) — 1 blink

Helps track time while working without checking the clock.

## Web UI Controls

- **Light** — on/off toggle + brightness slider
- **Color Temperature** — slider 2700K to 6500K
- **Brightness Step** — encoder step size (1-20%, default 10%)
- **Circadian Mode** — enable/disable automatic temperature
- **Test Single Blink** — test button for single blink
- **Test Double Blink** — test button for double blink
- **Sunrise/Sunset** — today's calculated times

## Memory Optimization

ESP8266 has limited RAM (~40KB). To prevent `Could not allocate memory for JSON document` errors:

- **Logger disabled** (`baud_rate: 0`, level: WARN) — saves ~2KB RAM
- **web_server: local: true** — reduced functionality, less memory
- **Removed wifi_info sensors** (IP, SSID, MAC) — fewer entities to serialize
- **internal: true** on encoder, button sensors — not exposed to web UI

To enable debugging temporarily:
```yaml
logger:
  level: DEBUG
  baud_rate: 115200
```

## Key Configuration Details

- `resolution: 1` — encoder sensitivity
- `on_clockwise/on_anticlockwise` — separate handlers prevent value accumulation
- `gamma_correct: 0` — linear brightness response
- `default_transition_length: 0s` — instant response
- `constant_brightness: true` — consistent perceived brightness
- `web_server: port 80, local: true` — built-in web interface, memory optimized
- `timezone: Europe/Lisbon` — for correct sunrise/sunset calculation
- `api: reboot_timeout: 0s` — don't reboot on API disconnect

## Resources

- [ESPHome Devices - Mi Desklamp](https://devices.esphome.io/devices/mi-desklamp/)
- [syssi/esphome-mi-desk-lamp](https://github.com/syssi/esphome-mi-desk-lamp)
- [Blakadder Templates](https://templates.blakadder.com/mi_desk_lamp.html)
- [ESPHome Sun Component](https://esphome.io/components/sun.html)
