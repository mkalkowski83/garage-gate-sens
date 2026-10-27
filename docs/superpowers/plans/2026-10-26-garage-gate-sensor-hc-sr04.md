# Garage Gate Sensor HC-SR04 — Implementation Plan

> **For agentic workers:** Use superpowers:executing-plans or subagent-driven-development. Checkboxes track tasks. **Do not auto-commit** unless the user explicitly asks (repo rule).

**Goal:** Parallel HC-SR04 variant with HA feature parity to VL53L1X; separate trees; easy YAML calibration.

**Architecture:** Copy VL53L1X ESPHome logic; replace ToF/I²C with `ultrasonic` (Trig GPIO5, Echo GPIO19). Power: USB→DevKit; HC-SR04 VCC from VIN; Echo via 1k+2k divider. Relocate VL53L1X paths; add `esphome/hc-sr04/` + KiCad/docs.

**Tech Stack:** ESPHome (`ultrasonic`), ESP32 DevKit V1, HC-SR04, KiCad, Home Assistant.

**Spec:** [`docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md`](../specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md)

## Global Constraints

- Divider: Echo → 1 kΩ → GPIO19; node → 2 kΩ → GND.
- C1 optional (not required).
- Same % model / entity names as VL53L1X; calibration via `substitutions` only.
- No shared ESPHome package; no MOSFET / level-shifter IC / custom PCB.

## Files

| Path | Action |
|------|--------|
| `esphome/garage-gate-sensor.yaml` → `esphome/vl53l1x/` | Relocate + fix links |
| `esphome/hc-sr04/garage-gate-sensor.yaml` | Create (ultrasonic port) |
| `esphome/hc-sr04/README.pl.md` | Create |
| `esphome/README.pl.md` | Index both variants |
| `docs/kicad/garage-gate-sens-devkit/` → `…-vl53l1x/` | Relocate + fix links |
| `docs/kicad/garage-gate-sens-hc-sr04/` | New schematic + symbols |
| `docs/schematics/connections-hc-sr04.{pl,en}.md` | Create |
| `README.{pl,en}.md`, `docs/README.md` | Dual-variant links |
| Spec/plan already under `docs/superpowers/` | Keep |

## Tasks

### Task 1: Branch + relocate VL53L1X tree

- [x] Create/switch `feature/garage-gate-sensor-hc-sr04` (from current VL53 work; merge→`main` when user commits/approves separately).
- [x] Move ESPHome YAML + KiCad folder to `vl53l1x` / `garage-gate-sens-vl53l1x` names; update internal links in READMEs, connections, specs pointers.
- **Done when:** VL53L1X builds path `esphome/vl53l1x/garage-gate-sensor.yaml`; old paths gone or redirected in docs.

### Task 2: ESPHome HC-SR04 firmware

- [x] Create `esphome/hc-sr04/garage-gate-sensor.yaml`: same globals/templates/hysteresis/entities; `ultrasonic` on GPIO5/GPIO19; drop I²C/VL53/range_status; `no_reading` hold 30 s on NaN echo.
- [x] `esphome/hc-sr04/README.pl.md` + root `esphome/README.pl.md` index.
- **Done when:** YAML present; substitutions match spec; `esphome compile` if toolchain available (else document command for user).

### Task 3: Schematics + KiCad HC-SR04

- [x] `connections-hc-sr04.pl.md` / `.en.md` (VIN, GND, Trig/Echo, 1k+2k, C1 optional).
- [x] KiCad project: symbols + empty sch + STEP1 (place R1/R2 in KiCad per STEP1); C1 optional.
- **Done when:** Docs match pins; schematic openable / ERC notes documented.

### Task 4: Root docs

- [x] README.pl/en + `docs/README.md` list both variants with paths to specs/plans/connections.
- **Done when:** Newcomer finds both trees from root README.

## Done when (AC)

1. Approaching target raises %; entity set matches VL53L1X (minus ToF diagnostics).
2. Substitutions calibrate 0/100 %; bad span → `invalid_calibration`.
3. Hysteresis + too-long work.
4. Docs/schematic = VIN/GND/GPIO5/18 + 1k+2k; C1 optional.
5. Compile command documented; run when ESPHome available.

## Git note

Merge `feature/garage-gate-sensor-v1` → `main` and push: **only after user commits + explicit merge/push ask**. This plan lands on `feature/garage-gate-sensor-hc-sr04`.
