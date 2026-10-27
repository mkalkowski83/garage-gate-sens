# Connections — ESP32 DevKit V1 + VL53L1X (TOF400C)

**ESP32-DevKitC / DevKit V1** (38-pin) and **VL53L1X** breakout (I²C).

**Power:** USB (or 5 V adapter) **into the ESP32**. The DevKit regulates **3V3**, which powers the sensor. No battery and no bulk C2 on +5 V in v1.

**Schematic (KiCad):** [`../kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.pdf`](../kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.pdf) — source: [`../kicad/garage-gate-sens-vl53l1x/`](../kicad/garage-gate-sens-vl53l1x/).

## 1. Power

| Source | ESP32 DevKit | Notes |
|--------|--------------|--------|
| USB / +5 V | 5V / VIN | Whole device powered through the DevKit |
| GND | GND | Common with ToF module |
| (on-board) | 3V3 | Supplies VL53L1X VIN + SHUT |

## 2. VL53L1X (6-pin module)

Pin order on the breakout (pin 1): **VIN → GND → SDA → SCL → INT → SHUT**.

| Module pin | Signal | ESP32 DevKit | Notes |
|------------|--------|--------------|--------|
| 1 | VIN | 3V3 | From DevKit |
| 2 | GND | GND | |
| 3 | SDA | GPIO21 (D21) | I²C |
| 4 | SCL | GPIO22 (D22) | I²C |
| 5 | INT | — | NC (`no_connect`; ESPHome polling) |
| 6 | SHUT | 3V3 | Tie high (enabled) |

**Capacitor:** C1 **100 nF** between 3V3 and GND at the ToF module.

## 3. Calibration (`esphome/vl53l1x/garage-gate-sensor.yaml`)

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
| GPIO21 (D21) | I²C SDA |
| GPIO22 (D22) | I²C SCL |
