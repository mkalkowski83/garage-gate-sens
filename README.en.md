# Garage gate opening sensor

ESP32 DevKit V1 + **ESPHome** / **Home Assistant**: **open %**, **open/closed**, **open duration**, **open too long**.

Two parallel distance-sensor variants:

| Variant | Sensor | Sensor power | Firmware |
|---------|--------|--------------|----------|
| **VL53L1X** | ToF (TOF400C), I²C | DevKit **3V3** | [`esphome/vl53l1x/`](esphome/vl53l1x/) |
| **HC-SR04** | Ultrasonic | **VIN (5 V)** + Echo divider 1k+2k | [`esphome/hc-sr04/`](esphome/hc-sr04/) |

Mounting (both): chain-track geometry; **smaller distance → more open**. Calibrate via YAML `substitutions`.

## Docs

| Topic | VL53L1X | HC-SR04 |
|-------|---------|---------|
| Connections | [connections.en.md](docs/schematics/connections.en.md) | [connections-hc-sr04.en.md](docs/schematics/connections-hc-sr04.en.md) |
| Spec | [2026-09-16 design](docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md) | [2026-10-26 HC-SR04](docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md) |
| Plan | [2026-09-16 plan](docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md) | [2026-10-26 plan](docs/superpowers/plans/2026-10-26-garage-gate-sensor-hc-sr04.md) |
| KiCad | [garage-gate-sens-vl53l1x/](docs/kicad/garage-gate-sens-vl53l1x/) | [garage-gate-sens-hc-sr04/](docs/kicad/garage-gate-sens-hc-sr04/) |
| ESPHome | [esphome/README.pl.md](esphome/README.pl.md) | same |

Polski: [README.pl.md](README.pl.md)

## Firmware

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
esphome run esphome/vl53l1x/garage-gate-sensor.yaml
esphome run esphome/hc-sr04/garage-gate-sensor.yaml
```
