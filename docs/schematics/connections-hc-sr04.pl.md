# Połączenia — ESP32 DevKit V1 + HC-SR04

Moduł: **ESP32-DevKitC / DevKit V1** (38 pin), **HC-SR04** (ultradźwiękowy).

**Zasilanie:** USB (lub zasilacz 5 V) **wpinany w ESP32**. HC-SR04 **VCC z VIN (5 V)**; wspólne **GND**. C1 100 nF przy module — **opcjonalny** (nie wymagany przy krótkich połączeniach USB→VIN).

**KiCad:** [`../kicad/garage-gate-sens-hc-sr04/`](../kicad/garage-gate-sens-hc-sr04/) — patrz README / STEP1.

## 1. Zasilanie

| Źródło | ESP32 DevKit (U1) | HC-SR04 (U2) | Uwagi |
|--------|-------------------|--------------|--------|
| USB sieciowy | pin **USB** | — | Wejście zasilania DevKit |
| (szyna 5 V) | **5V/VIN** | VCC | VIN zasilany z USB; stąd czujnik |
| GND | GND | GND | Wspólna masa |

## 2. Sygnały + dzielnik Echo

| HC-SR04 | ESP32 / pasywne | Uwagi |
|---------|-----------------|--------|
| Trig | GPIO5 | 3,3 V z ESP OK |
| Echo | → **R1 1 kΩ** → węzeł → GPIO19; węzeł → **R2 2 kΩ** → GND | ~3,3 V na GPIO |

## 3. Kalibracja w `esphome/hc-sr04/garage-gate-sensor.yaml`

| Parametr | Znaczenie |
|----------|-----------|
| `mount_offset_mm` | Odejmowany od każdego odczytu |
| `clearance_full_open_mm` | Dystans przy **pełnym otwarciu** (100 %) |
| `distance_closed_mm` | Raw przy **zamkniętej** bramie |
| `distance_open_mm` | Opcjonalna kontrola ≈ clearance + mount |

**Logika:** mniejszy dystans → większe % otwarcia. **Gate Sensor Status** = `ok` przed uznaniem kalibracji.

## 4. Mapa GPIO

| GPIO | Funkcja |
|------|---------|
| GPIO5 | Trig |
| GPIO19 | Echo (za dzielnikiem 1k+2k) |
