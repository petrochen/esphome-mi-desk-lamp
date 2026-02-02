# ESPHome Mi Desk Lamp

Custom ESPHome firmware for **Xiaomi Mi Desk Lamp (MJTD01YL)** with advanced features.

## Features

- **Circadian Lighting** — automatic color temperature based on sunrise/sunset
- **Rotary Encoder** — brightness and color temperature control
- **Hourly Reminders** — subtle blinks at :00 (2x) and :30 (1x)
- **Web Interface** — control via browser
- **Home Assistant** — native API integration
- **OTA Updates** — wireless firmware updates

## Hardware

| Component | GPIO |
|-----------|------|
| Cold White LED | GPIO4 (PWM) |
| Warm White LED | GPIO5 (PWM) |
| Rotary Encoder A | GPIO13 |
| Rotary Encoder B | GPIO12 |
| Button | GPIO2 |

## Installation

1. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your WiFi credentials
2. First flash via serial (FTDI): connect TX→RX, RX→TX, GND→GND, hold GPIO0 to GND while powering on
3. Run: `esphome run mi-desk-lamp.yaml --device /dev/cu.usbserial-0001`
4. Subsequent updates via OTA: `esphome upload mi-desk-lamp.yaml --device <IP>`

## Controls

| Action | Result |
|--------|--------|
| Rotate encoder | Adjust brightness |
| Hold button + rotate | Adjust color temperature |
| Click button | Toggle on/off |
| Double-click button | Reset to circadian mode |

## Circadian Schedule

Color temperature adjusts automatically based on sun position:

| Period | Temperature |
|--------|-------------|
| Night → Sunrise | 2700K (warm) |
| Sunrise → +2h | 2700K → 5500K |
| Day | 5500K → 6500K (cool) |
| Sunset -2h → Sunset | 6500K → 3500K |
| Sunset → +2h | 3500K → 2700K |

## Configuration

Edit coordinates in `mi-desk-lamp.yaml` for your location:

```yaml
sun:
  latitude: 38.7223°
  longitude: -9.1393°
```

## References

- [Blakadder Templates](https://templates.blakadder.com/mi_desk_lamp.html)
- [ESPHome Devices](https://devices.esphome.io/devices/mi-desklamp/)
- [syssi/esphome-mi-desk-lamp](https://github.com/syssi/esphome-mi-desk-lamp)

## License

MIT
