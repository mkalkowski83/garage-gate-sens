# Connections — ESP32 DevKit V1 + HC-SR04

Board: **ESP32-DevKitC / DevKit V1** (38-pin), **HC-SR04** ultrasonic module.

**Power:** USB (or 5 V adapter) **into the ESP32**. HC-SR04 **VCC from VIN (5 V)**; common **GND**. C1 100 nF at the module is **optional** (not required with short USB→VIN wiring).

**KiCad:** [`../kicad/garage-gate-sens-hc-sr04/`](../kicad/garage-gate-sens-hc-sr04/) — see README / STEP1.

## 1. Power

| Source | ESP32 DevKit (U1) | HC-SR04 (U2) | Notes |
|--------|-------------------|--------------|--------|
| Mains USB | **USB** pin | — | DevKit power input |
| (5 V rail) | **5V/VIN** | VCC | VIN from USB; feeds sensor |
| GND | GND | GND | Common ground |

## 2. Signals + Echo divider

| HC-SR04 | ESP32 / passives | Notes |
|---------|------------------|--------|
| Trig | GPIO5 | 3.3 V from ESP is OK |
| Echo | → **R1 1 kΩ** → node → GPIO19; node → **R2 2 kΩ** → GND | ~3.3 V at GPIO |

## 3. Calibration (`esphome/hc-sr04/garage-gate-sensor.yaml`)

| Parameter | Meaning |
|-----------|---------|
| `mount_offset_mm` | Subtracted from every reading |
| `clearance_full_open_mm` | Distance at **full open** (100 %) |
| `distance_closed_mm` | Raw when gate **closed** |
| `distance_open_mm` | Optional check ≈ clearance + mount |

**Logic:** smaller distance → higher open %. **Gate Sensor Status** must be `ok` before calibration is considered done.

## 4. GPIO map

| GPIO | Function |
|------|----------|
| GPIO5 | Trig |
| GPIO19 | Echo (after 1k+2k divider) |
