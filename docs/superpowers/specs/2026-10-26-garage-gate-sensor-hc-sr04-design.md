# Garage gate sensor — HC-SR04 variant — design

**Date:** 2026-10-26  
**Status:** Approved  
**Prior art:** VL53L1X v1 — `docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md`

## Goal

Parallel garage-gate monitor on **ESP32 + HC-SR04**: same Home Assistant entities as VL53L1X (open %, open/closed, open duration, open-too-long), with field calibration via YAML `substitutions`.

## Decisions

- **Two variants, separate trees** — VL53L1X and HC-SR04 both stay; no shared ESPHome package in this scope.
- **Git:** merge VL53L1X branch to `main`, then branch `feature/garage-gate-sensor-hc-sr04` for this work. On that branch, relocate `esphome/garage-gate-sensor.yaml` → `esphome/vl53l1x/` and `docs/kicad/garage-gate-sens-devkit/` → `docs/kicad/garage-gate-sens-vl53l1x/` (update links), then add HC-SR04 tree.
- **Feature parity** with VL53L1X firmware logic (copy + swap sensor).
- **Mounting:** sensor on chain-track geometry; **smaller distance → more open** (same % model as v1).
- **Power:** USB → ESP32 DevKit; HC-SR04 **VCC from VIN (5 V)**; common GND. (User board may already have VCC/GND wired.)
- **Pins:** Trig **GPIO5**, Echo **GPIO19** after resistor divider.
- **Echo level shift:** divider **1 kΩ (series) + 2 kΩ (to GND)** ≈ 3.3 V into ESP.
- **C1:** optional; not required when power is short USB→VIN→module; add only if readings are noisy.
- **Calibration knobs (substitutions):** `mount_offset_mm`, `clearance_full_open_mm`, `distance_closed_mm`, `distance_open_mm` (check), open/close % thresholds, `max_open_seconds`.

## Scope

- `esphome/hc-sr04/` — ESPHome YAML (+ short README)
- `docs/kicad/garage-gate-sens-hc-sr04/` — schematic (DevKit + HC-SR04 + divider; C1 optional)
- `docs/schematics/connections-hc-sr04.*.md` + PDF export
- Root README links to **both** variants
- Reorganize VL53L1X paths into a dedicated subtree when introducing the split

### Out of scope

- Shared YAML packages, HA live `number` calibration, NVS open-duration across reboot
- MOSFET / level-shifter IC, custom PCB, outdoor enclosure, gate actuation
- Removing or abandoning the VL53L1X variant

## Acceptance criteria

1. Bench: approaching target raises open %; entities match VL53L1X set.
2. After tuning substitutions: closed ≈ 0 %, full open ≈ 100 %; bad span → `invalid_calibration`.
3. Binary open/closed uses % hysteresis (no flutter); too-long alert works.
4. Docs/schematic match wiring (VIN, GND, GPIO5/18, 1k+2k); C1 marked optional; KiCad ERC clean.
5. `esphome compile` succeeds for the HC-SR04 project.

## Layout (target)

```
esphome/vl53l1x/          # relocated from garage-gate-sensor.yaml
esphome/hc-sr04/
docs/kicad/garage-gate-sens-vl53l1x/   # relocated from garage-gate-sens-devkit
docs/kicad/garage-gate-sens-hc-sr04/
docs/schematics/connections*.md       # VL53L1X
docs/schematics/connections-hc-sr04.*
docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md
docs/superpowers/specs/2026-10-26-garage-gate-sensor-hc-sr04-design.md
```
