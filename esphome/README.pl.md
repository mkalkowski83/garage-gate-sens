# ESPHome — czujnik bramy garażowej

Dwa równoległe warianty (osobne drzewa):

| Wariant | YAML | README |
|---------|------|--------|
| **VL53L1X** (ToF) | [`vl53l1x/garage-gate-sensor.yaml`](vl53l1x/garage-gate-sensor.yaml) | [vl53l1x/README.pl.md](vl53l1x/README.pl.md) |
| **HC-SR04** (ultradźwięk) | [`hc-sr04/garage-gate-sensor.yaml`](hc-sr04/garage-gate-sensor.yaml) | [hc-sr04/README.pl.md](hc-sr04/README.pl.md) |

## Sekrety (wspólne)

```bash
cp secrets.yaml.example secrets.yaml
# Uzupełnij Wi‑Fi i gdos_sensor__api_key
# W podkatalogu wariantu: ln -sf ../secrets.yaml secrets.yaml
```
