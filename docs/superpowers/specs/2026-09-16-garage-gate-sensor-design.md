# Garage gate opening sensor — design spec

**Date:** 2026-09-16  
**Repo:** `garage-gate-sens` (GageGateSens)  
**Status:** Approved in brainstorming (with mechanical offset parameters)

## 1. Goal

Wireless garage gate monitor: **opening percentage**, **open/closed** state with hysteresis, **current open duration**, and **open too long** alert. Firmware on **ESP32** with **ESPHome**; **Home Assistant** for dashboards and notifications. Fixed **5 V USB** supply; electronics inside a dry garage.

**Measurement geometry:** VL53L1X on a chain-track-related mount; laser points at the **end of the gate leaf**. **Smaller distance → more open**; **larger distance → more closed**.

**Approach:** ESPHome-centric (“tank-level style”) — ESP is source of truth for %, binary state, duration, and too-long flag.

## 2. Hardware

| Item | Role |
|------|------|
| ESP32 DevKit V1 | MCU, Wi‑Fi |
| VL53L1X module (e.g. TOF400C breakout) | Distance via I²C |
| 5 V USB adapter | Mains power |
| Optional 100 nF on 3V3 at module | Decoupling (footprint 0805) |
| Optional bulk cap on 5 V input | Stability (footprint in schematic) |

**Not in v1:** battery, charger, boost, sensor power MOSFET, custom PCB, outdoor enclosure.

### 2.1 Connections

| Signal | Connection |
|--------|------------|
| +5 V | USB input → ESP32 5V/VIN |
| GND | Common |
| +3V3 | ESP32 3V3 → module VIN (preferred) |
| I2C SDA | GPIO21 |
| I2C SCL | GPIO22 |
| SHUT | Tie to 3V3 (always on) |
| INT | NC (polling) |

### 2.2 KiCad

- Path: `docs/kicad/garage-gate-sens-devkit/`
- Ideological schematic: power, DevKit, VL53L1X module, net labels
- **Footprints:** DevKit header pattern; VL53L1X breakout (verify pin spacing against physical module)
- Export PDF → `docs/schematics/`
- Connection tables: `docs/schematics/connections*.pl.md` (mirror tank-level layout)
- v1: schematic + BOM; no routed PCB

## 3. Mechanical parameters and calibration

Two **mechanical** values (user-configured in YAML `substitutions`) describe mounting that raw VL53L1X readings alone do not encode:

| Parameter | YAML name | Meaning |
|-----------|-----------|---------|
| Clearance at full open | `clearance_full_open_mm` | Physical distance **sensor → gate end** when the gate is **fully open** (dead zone / stop position). This is the **effective distance at 100 % open**. |
| Mount offset (closed side) | `mount_offset_mm` | Constant distance subtracted from **every raw reading** because the sensor is mounted **back** from the closed reference along the travel axis (offset “after closed” / along track). |

Optional **field calibration** readings (still recommended on site):

| Parameter | YAML name | Meaning |
|-----------|-----------|---------|
| Raw at closed | `distance_closed_mm` | VL53L1X reading when gate is **fully closed** (largest raw value). Used to validate span and tune `mount_offset_mm`. |
| Raw at full open | `distance_open_mm` | VL53L1X reading when gate is **fully open** (smallest raw). Should be consistent with `clearance_full_open_mm + mount_offset_mm` within a few mm. |

### 3.1 Effective distance

```
d_eff = raw_mm - mount_offset_mm
```

### 3.2 Open percentage

Closed reference (effective):

```
d_closed_eff = distance_closed_mm - mount_offset_mm
```

Open reference (effective) — from mechanical dead zone, not only a one-time reading:

```
d_open_eff = clearance_full_open_mm
```

Linear map (closer gate → higher %):

```
span = d_closed_eff - d_open_eff
percent = clamp(100 * (d_closed_eff - d_eff) / span, 0, 100)
```

If `span <= 0`, firmware must report error state (`invalid_calibration`) and avoid bogus %.

**Validation after install:** with gate closed, `d_eff ≈ d_closed_eff`; with gate fully open, `d_eff ≈ d_open_eff`. Mismatch → adjust `mount_offset_mm` or `clearance_full_open_mm`, or re-measure `distance_closed_mm` / `distance_open_mm`.

### 3.3 Hysteresis (open vs closed binary)

On **Gate Open Percent** (not raw mm):

- Transition to **open:** `percent >= open_threshold_percent` (default 10)
- Transition to **closed:** `percent <= close_threshold_percent` (default 5)
- Implement with latched state in template/globals to avoid flutter from chain vibration.

## 4. ESPHome entities

| Entity | Type | Notes |
|--------|------|--------|
| Gate Distance Raw | sensor (mm) | After VL53L1X filters |
| Gate Distance Effective | sensor (mm) | `raw - mount_offset_mm` (optional but useful for tuning) |
| Gate Open Percent | sensor (%) | Formula in §3.2 |
| Gate Open | binary_sensor | Hysteresis on % |
| Gate Open Duration | sensor (s) | Seconds since open; 0 when closed |
| Gate Open Too Long | binary_sensor | Open AND duration > `max_open_seconds` |
| Gate Sensor Status | sensor (text) | `ok` / `no_reading` / `invalid_calibration` |

**Timing:** VL53L1X update ~500 ms; moving average/median (e.g. window 5); templates for %, binary, duration at 1 s.

**Substitutions (defaults are placeholders until install):**

- `clearance_full_open_mm`
- `mount_offset_mm`
- `distance_closed_mm`
- `distance_open_mm` (optional cross-check only)
- `open_threshold_percent`, `close_threshold_percent`
- `max_open_seconds` (e.g. 900)

**Out of scope v1:** HA `number` entities for live calibration; NVS restore of open-since across reboot (duration resets on reboot if gate still open — acceptable v1).

## 5. Home Assistant

- Use ESP entities directly on dashboard (gauge %, gate icon, duration).
- Example automation: notify when **Gate Open Too Long** turns on.
- Do not recompute % in HA templates (ESP is authoritative).

## 6. Errors and edge cases

- I²C / invalid reading: hold last good value up to 30 s, then `no_reading`.
- `invalid_calibration`: bad span or persistent NaN.
- Indoor garage: no IP rating required for v1; note vibration filtering in docs.

## 7. Testing (DoD)

1. Bench: move target → % increases as target approaches sensor.
2. Install: closed ≈ 0 %, full open ≈ 100 % after offset/clearance tuning.
3. Hysteresis: no binary flutter at partial open.
4. Duration and too-long threshold.
5. KiCad ERC clean; PDF matches connection doc.
6. ESPHome compile for project ESPHome version (align with tank-level, e.g. 2026.8).

## 8. Out of scope v1

Custom PCB, gate control, battery/solar, outdoor sensor housing, multiple sensors, Modbus/RS485 variants.

## 9. Repository layout (target)

```
esphome/garage-gate-sensor.yaml
docs/schematics/connections*.pl.md
docs/kicad/garage-gate-sens-devkit/
docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md
README.pl.md / README.en.md
```
