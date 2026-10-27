# Czujnik otwarcia bramy garażowej

Monitor otwarcia bramy na **ESP32 DevKit V1** + **ESPHome** / **Home Assistant**: **% otwarcia**, **otwarta/zamknięta**, **czas otwarcia**, **otwarta za długo**.

Dwa równoległe warianty czujnika odległości:

| Wariant | Czujnik | Zasilanie czujnika | Firmware |
|---------|---------|-------------------|----------|
| **VL53L1X** | ToF (TOF400C), I²C | **3V3** z DevKit | [`esphome/vl53l1x/`](esphome/vl53l1x/) |
| **HC-SR04** | Ultradźwięk | **VIN (5 V)** + dzielnik Echo 1k+2k | [`esphome/hc-sr04/`](esphome/hc-sr04/) |

Montaż (oba): przy torze łańcucha; **mniejsza odległość → większe otwarcie**. Kalibracja przez `substitutions` w YAML.

## Dokumentacja

| Temat | VL53L1X | HC-SR04 |
|-------|---------|---------|
| Połączenia | [connections.pl.md](docs/schematics/connections.pl.md) | [connections-hc-sr04.pl.md](docs/schematics/connections-hc-sr04.pl.md) |
| Spec | [2026-09-16-…design.md](docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md) | [2026-10-26-…hc-sr04-design.md](docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md) |
| Plan | [2026-09-16-….pl.md](docs/superpowers/plans/2026-09-16-garage-gate-sensor.pl.md) | [2026-10-26-…hc-sr04.md](docs/superpowers/plans/2026-10-26-garage-gate-sensor-hc-sr04.md) |
| KiCad | [garage-gate-sens-vl53l1x/](docs/kicad/garage-gate-sens-vl53l1x/) | [garage-gate-sens-hc-sr04/](docs/kicad/garage-gate-sens-hc-sr04/) |
| ESPHome | [esphome/README.pl.md](esphome/README.pl.md) | j.w. |

English: [README.en.md](README.en.md)

## Firmware

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
# VL53L1X:
esphome run esphome/vl53l1x/garage-gate-sensor.yaml
# HC-SR04:
esphome run esphome/hc-sr04/garage-gate-sensor.yaml
```
