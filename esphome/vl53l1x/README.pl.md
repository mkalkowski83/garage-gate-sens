# ESPHome — wariant VL53L1X

Plik: [`garage-gate-sensor.yaml`](garage-gate-sensor.yaml)  
Spec: [`../../docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md`](../../docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md)  
Połączenia: [`../../docs/schematics/connections.pl.md`](../../docs/schematics/connections.pl.md)

## Sekrety

```bash
ln -sf ../secrets.yaml secrets.yaml   # w esphome/vl53l1x/
# lub: cp ../secrets.yaml.example ../secrets.yaml  (raz w esphome/)
```

## Build / flash

```bash
esphome run esphome/vl53l1x/garage-gate-sensor.yaml
esphome logs esphome/vl53l1x/garage-gate-sensor.yaml
```

Wymaga GitHub (komponent `esphome-vl53l1x-pololu`).

## Strojenie kalibracji

| Parametr | Znaczenie |
|----------|-----------|
| `distance_closed_mm` | raw przy zamkniętej |
| `clearance_full_open_mm` | dystans przy 100 % |
| `mount_offset_mm` | offset montażu |
| `distance_open_mm` | kontrola ≈ clearance + mount |

**Gate Sensor Status** = `ok` przed uznaniem kalibracji.
