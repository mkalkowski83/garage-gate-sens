# Garage gate opening sensor — design spec

**Date:** 2026-09-16  
**Repo:** `garage-gate-sens` (GageGateSens)  
**Status:** Approved (hardware schematic finalized)

## 1. Goal

Wireless garage gate monitor: **opening percentage**, **open/closed** state with hysteresis, **current open duration**, and **open too long** alert. Firmware on **ESP32** with **ESPHome**; **Home Assistant** for dashboards and notifications. Power via **USB into ESP32 DevKit**; electronics inside a dry garage.

**Measurement geometry:** VL53L1X on a chain-track-related mount; laser points at the **end of the gate leaf**. **Smaller distance → more open**; **larger distance → more closed**.

**Approach:** ESPHome-centric (“tank-level style”) — ESP is source of truth for %, binary state, duration, and too-long flag.

## 2. Hardware

| Item | Role |
|------|------|
| ESP32 DevKit V1 | MCU, Wi‑Fi, onboard 5 V→3.3 V regulation |
| VL53L1X module (TOF400C breakout) | Distance via I²C |
| USB 5 V supply into DevKit | Mains power for whole board |
| C1 100 nF | Decoupling on 3V3 at ToF module |

**Power topology:** USB (or 5 V wall adapter) → ESP32 `5V/VIN` → DevKit regulator → `3V3` → VL53L1X `VIN` and `SHUT`. The sensor is powered only from ESP **3V3**, not from a separate 5 V rail.

**Not in v1:** battery, charger, boost, bulk capacitor on +5 V (C2), sensor power MOSFET, custom PCB, outdoor enclosure, footprints / PCB placement focus.

### 2.1 Connections

| ESP32 / power | Connection |
|---------------|------------|
| +5 V (USB into DevKit) | ESP32 `5V/VIN` |
| GND | Common with module pin 2 |

VL53L1X module pin order: **VIN, GND, SDA, SCL, INT, SHUT** (pins 1–6).

| Pin | Signal | Connection |
|-----|--------|------------|
| 1 | VIN | ESP32 3V3 |
| 2 | GND | GND |
| 3 | SDA | GPIO21 (D21 on DevKit silkscreen) |
| 4 | SCL | GPIO22 (D22 on DevKit silkscreen) |
| 5 | INT | NC (`no_connect`; ESPHome polls over I²C) |
| 6 | SHUT | 3V3 (always on) |

**KiCad ERC:** `PWR_FLAG` only on external +5 V if needed. Do **not** place `PWR_FLAG` on 3V3 — ESP `3V3` is already `power_out` and is the source for that net.

### 2.2 KiCad

- Path: `docs/kicad/garage-gate-sens-vl53l1x/` (formerly `garage-gate-sens-devkit`)
- Ideological schematic: USB/+5V → DevKit → VL53L1X on I²C + C1
- Custom symbols (`garage-gate.kicad_sym`):
  - `ESP32_DEVKIT` — only pins used: `5V/VIN`, `GND`, `3V3`, `GPIO21/D21/SDA`, `GPIO22/D22/SCL`
  - `VL53L1X` — pins 1–6: VIN, GND, SDA, SCL, INT, SHUT
- Schematic refs (as drawn): **U2** = ESP32 DevKit, **U1** = VL53L1X, **C1** = 100 nF
- Export PDF → `docs/schematics/garage-gate-sens-devkit.pdf`
- Connection tables: `docs/schematics/connections*.pl.md`
- v1: schematic + BOM notes; no routed PCB; footprints deferred

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
5. KiCad ERC clean (no `PWR_FLAG` on 3V3); PDF matches connection doc.
6. ESPHome compile for project ESPHome version (align with tank-level, e.g. 2026.8).

## 8. Out of scope v1

Custom PCB, gate control, battery/solar, outdoor sensor housing, multiple sensors, Modbus/RS485 variants, bulk +5 V capacitor, footprint assignment.

## 9. Repository layout (target)

```
esphome/vl53l1x/garage-gate-sensor.yaml
docs/schematics/connections*.md
docs/kicad/garage-gate-sens-vl53l1x/
docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md
README.pl.md / README.en.md
```

Parallel HC-SR04 variant: `docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md`.
