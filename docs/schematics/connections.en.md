# Connections — ESP32 DevKit V1 + VL53L1X (TOF400C)

**ESP32-DevKitC / DevKit V1** (38-pin) and **VL53L1X** breakout (I²C).

Power: **fixed 5 V USB** wall adapter (no battery in v1).

**Schematic (KiCad):** [`garage-gate-sens-devkit.pdf`](garage-gate-sens-devkit.pdf) — source: [`../kicad/garage-gate-sens-devkit/`](../kicad/garage-gate-sens-devkit/).

## 1. Power

| Source | ESP32 DevKit | Notes |
|--------|--------------|--------|
| +5 V (USB) | 5V / VIN | Common ground with ToF module |
| GND | GND | |

## 2. VL53L1X (6-pin module)

| ESP32 DevKit | VL53L1X | Notes |
|--------------|---------|--------|
| 3V3 | VIN | Module supply from DevKit 3.3 V |
| GND | GND | |
| GPIO21 | SDA | I²C |
| GPIO22 | SCL | I²C |
| 3V3 | SHUT | Tie high (sensor enabled) |
| — | INT | NC (ESPHome polling) |

## 3. Calibration (`esphome/garage-gate-sensor.yaml`)

| Parameter | Meaning |
|-----------|---------|
| `mount_offset_mm` | Subtracted from every raw reading (sensor mounted back from travel reference) |
| `clearance_full_open_mm` | Sensor-to-gate-end distance at **fully open** (dead zone, 100 %) |
| `distance_closed_mm` | Raw reading (mm) when gate is **closed** |
| `distance_open_mm` | Optional check: should be ≈ `clearance_full_open_mm + mount_offset_mm` |

**Logic:** shorter distance → higher open %. **Gate Sensor Status** should read `ok` before trusting calibration.

## 4. GPIO map

| GPIO | Function |
|------|----------|
| GPIO21 | I²C SDA |
| GPIO22 | I²C SCL |
