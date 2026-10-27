# ESPHome — wariant HC-SR04

Plik: [`garage-gate-sensor.yaml`](garage-gate-sensor.yaml)  
Spec: [`../../docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md`](../../docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md)  
Połączenia: [`../../docs/schematics/connections-hc-sr04.pl.md`](../../docs/schematics/connections-hc-sr04.pl.md)

## Sekrety

Wspólne z VL53L1X — w katalogu nadrzędnym:

```bash
cp ../secrets.yaml.example ../secrets.yaml
```

Komenda `esphome` uruchamiaj z katalogu `esphome/` (lub podaj ścieżkę do `secrets.yaml` zgodnie z Twoim setupem). Domyślnie ESPHome szuka `secrets.yaml` obok YAML — skopiuj/zlinkuj:

```bash
ln -sf ../secrets.yaml secrets.yaml   # w esphome/hc-sr04/
```

## Build / flash

```bash
esphome run esphome/hc-sr04/garage-gate-sensor.yaml
esphome logs esphome/hc-sr04/garage-gate-sensor.yaml
```

## Hardware (skrót)

| Pin | Funkcja |
|-----|---------|
| VIN → HC-SR04 VCC | 5 V |
| GND | wspólne |
| GPIO5 | Trig |
| GPIO19 | Echo za dzielnikiem **1 kΩ + 2 kΩ** |

## Strojenie kalibracji

Te same `substitutions` co VL53L1X: `mount_offset_mm`, `clearance_full_open_mm`, `distance_closed_mm`, progi %, `max_open_seconds`.

1. Zamknięta → wpisz raw do `distance_closed_mm`
2. Ustaw `mount_offset_mm` i `clearance_full_open_mm`
3. Sprawdź % (0 / 100) i **Gate Sensor Status** = `ok`
