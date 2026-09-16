# Czujnik otwarcia bramy garażowej

Monitor otwarcia bramy na **ESP32 DevKit V1**, czujnik **VL53L1X** (moduł TOF400C, I²C) i **ESPHome** z integracją **Home Assistant**. Raportuje **% otwarcia**, stan **otwarta/zamknięta**, **czas otwarcia** i sygnał **otwarta za długo**. Zasilanie **5 V USB** (garaż, sucho).

## Hardware (v1)

| Element | Rola |
|---------|------|
| ESP32 DevKit V1 | MCU, Wi‑Fi |
| Moduł VL53L1X (TOF400C) | Pomiar odległości do końca bramy |
| Zasilacz USB 5 V | Stałe zasilanie |

Montaż: laser na elemencie toru łańcucha; **mniejsza odległość → większe otwarcie**.

## Dokumentacja

| Temat | Link |
|-------|------|
| Połączenia | [docs/schematics/connections.pl.md](docs/schematics/connections.pl.md) |
| Specyfikacja | [docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md](docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md) |
| Plan | [docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md](docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md) |
| KiCad | [docs/kicad/garage-gate-sens-devkit/](docs/kicad/garage-gate-sens-devkit/) |
| ESPHome | [esphome/README.pl.md](esphome/README.pl.md) |

English: [README.en.md](README.en.md)

## Firmware

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
esphome run esphome/garage-gate-sensor.yaml
```

Kalibracja: `mount_offset_mm`, `clearance_full_open_mm`, `distance_closed_mm` w `esphome/garage-gate-sensor.yaml`.
