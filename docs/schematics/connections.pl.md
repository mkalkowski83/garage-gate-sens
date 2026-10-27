# Połączenia — ESP32 DevKit V1 + VL53L1X (TOF400C)

Moduł: **ESP32-DevKitC / DevKit V1** (38 pin), moduł **VL53L1X** na breakout (I²C).

**Zasilanie:** USB (lub zasilacz 5 V) **wpinany w ESP32**. DevKit reguluje **3V3**, które zasila cały czujnik. Bez baterii i bez osobnego bulk C2 na +5 V (v1).

**Schemat ideowy (KiCad):** [`../kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.pdf`](../kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.pdf) — projekt: [`../kicad/garage-gate-sens-vl53l1x/`](../kicad/garage-gate-sens-vl53l1x/).

## 1. Zasilanie

| Źródło | ESP32 DevKit | Uwagi |
|--------|--------------|--------|
| USB / +5 V | 5V / VIN | Zasilanie całego układu przez DevKit |
| GND | GND | Wspólna masa z modułem ToF |
| (wewnętrznie) | 3V3 | Źródło dla VL53L1X VIN + SHUT |

## 2. VL53L1X (moduł 6-pin)

Kolejność pinów na module (pin 1): **VIN → GND → SDA → SCL → INT → SHUT**.

| Pin modułu | Sygnał | ESP32 DevKit | Uwagi |
|------------|--------|--------------|--------|
| 1 | VIN | 3V3 | Zasilanie z DevKit |
| 2 | GND | GND | |
| 3 | SDA | GPIO21 (D21) | I²C |
| 4 | SCL | GPIO22 (D22) | I²C |
| 5 | INT | — | NC (`no_connect`; polling ESPHome) |
| 6 | SHUT | 3V3 | Stałe włączenie |

**Kondensator:** C1 **100 nF** między 3V3 a GND przy module ToF.

## 3. Kalibracja w `esphome/vl53l1x/garage-gate-sensor.yaml`

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
| GPIO21 (D21) | I²C SDA |
| GPIO22 (D22) | I²C SCL |
