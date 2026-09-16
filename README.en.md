# Garage gate opening sensor

Garage gate monitor using **ESP32 DevKit V1**, **VL53L1X** (TOF400C I²C module), **ESPHome**, and **Home Assistant**. Reports **open %**, **open/closed** state, **open duration**, and **open too long**. Powered from a **5 V USB** adapter (indoor garage).

## Hardware (v1)

| Part | Role |
|------|------|
| ESP32 DevKit V1 | MCU, Wi‑Fi |
| VL53L1X breakout (TOF400C) | Distance to gate end |
| 5 V USB adapter | Mains power |

Mounting: sensor on chain-track hardware; **shorter distance → more open**.

## Documentation

| Topic | Link |
|-------|------|
| Wiring | [docs/schematics/connections.en.md](docs/schematics/connections.en.md) |
| Design spec | [docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md](docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md) |
| Implementation plan | [docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md](docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md) |
| KiCad | [docs/kicad/garage-gate-sens-devkit/](docs/kicad/garage-gate-sens-devkit/) |
| ESPHome | [esphome/README.pl.md](esphome/README.pl.md) |

Polski: [README.pl.md](README.pl.md)

## Firmware

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
esphome run esphome/garage-gate-sensor.yaml
```

Tune `mount_offset_mm`, `clearance_full_open_mm`, and `distance_closed_mm` in `esphome/garage-gate-sensor.yaml`.
