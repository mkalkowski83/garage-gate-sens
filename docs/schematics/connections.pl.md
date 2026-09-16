# Połączenia — ESP32 DevKit V1 + VL53L1X (TOF400C)

Moduł: **ESP32-DevKitC / DevKit V1** (38 pin), moduł **VL53L1X** na breakout (I²C).

Zasilanie: **stałe 5 V USB** (zasilacz ścienny) — bez baterii w wersji v1.

**Schemat ideowy (KiCad):** [`garage-gate-sens-devkit.pdf`](garage-gate-sens-devkit.pdf) — projekt źródłowy: [`../kicad/garage-gate-sens-devkit/`](../kicad/garage-gate-sens-devkit/).

## 1. Zasilanie

| Źródło | ESP32 DevKit | Uwagi |
|--------|--------------|--------|
| +5 V (USB) | 5V / VIN | Wspólna masa z modułem ToF |
| GND | GND | |

## 2. VL53L1X (moduł 6-pin)

| ESP32 DevKit | VL53L1X | Uwagi |
|--------------|---------|--------|
| 3V3 | VIN | Zasilanie modułu (3,3 V z DevKit) |
| GND | GND | |
| GPIO21 | SDA | I²C |
| GPIO22 | SCL | I²C |
| 3V3 | SHUT | Stałe włączenie czujnika |
| — | INT | Niepodłączone (polling w ESPHome) |

## 3. Kalibracja w `esphome/garage-gate-sensor.yaml`

| Parametr | Znaczenie |
|----------|-----------|
| `mount_offset_mm` | Odejmowany od każdego odczytu (czujnik cofnięty względem osi ruchu / „po zamknięciu”) |
| `clearance_full_open_mm` | Odległość czujnik → koniec bramy przy **pełnym otwarciu** (martwa strefa, 100 %) |
| `distance_closed_mm` | Surowy odczyt (mm) przy **zamkniętej** bramie |
| `distance_open_mm` | Opcjonalna kontrola: powinno być ≈ `clearance_full_open_mm + mount_offset_mm` |

**Logika:** mniejszy dystans → większe % otwarcia. Encja **Gate Sensor Status** musi pokazywać `ok` przed uznaniem kalibracji za gotową.

## 4. Mapa GPIO

| GPIO | Funkcja |
|------|---------|
| GPIO21 | I²C SDA |
| GPIO22 | I²C SCL |
