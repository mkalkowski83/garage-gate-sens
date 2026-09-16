# ESPHome — czujnik bramy garażowej

Plik: [`garage-gate-sensor.yaml`](garage-gate-sensor.yaml)

## Sekrety

```bash
cp secrets.yaml.example secrets.yaml
# Uzupełnij Wi‑Fi i garage_gate_api_encryption_key (esphome encryption-key)
```

## Build / flash

```bash
esphome run esphome/garage-gate-sensor.yaml
esphome logs esphome/garage-gate-sensor.yaml
```

Wymaga dostępu do GitHub (pobranie `esphome-vl53l1x-pololu` przy pierwszej kompilacji).

## Strojenie kalibracji

1. Brama **zamknięta** → odczyt **Gate Distance Raw** → wpisz `distance_closed_mm`.
2. Ustaw `mount_offset_mm` (cofnięcie czujnika względem osi ruchu).
3. Ustaw `clearance_full_open_mm` (odległość czujnik–koniec bramy przy pełnym otwarciu).
4. Sprawdź **Gate Open Percent** (~0 % zamknięta, ~100 % otwarta).
5. **Gate Sensor Status** = `ok`.

Opcjonalnie: `distance_open_mm` ≈ `clearance_full_open_mm + mount_offset_mm`.

## Home Assistant

Przykład automatu: [`../docs/home-assistant/example-automation.yaml`](../docs/home-assistant/example-automation.yaml)
