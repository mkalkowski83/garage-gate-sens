# STEP1 — schemat HC-SR04

1. KiCad → **File → Open Project** → `docs/kicad/garage-gate-sens-hc-sr04/garage-gate-sens-hc-sr04.kicad_pro`
2. Schematic Editor → nowy arkusz A4 (jeśli pusty).
3. Umieść z `garage-gate`: **ESP32_DEVKIT**, **HC_SR04**; z Device: **R** ×2 (opcjonalnie **C**).
4. Połącz według [`connections-hc-sr04.pl.md`](../../schematics/connections-hc-sr04.pl.md):
   - **USB** (U1) ← `+5V` + `PWR_FLAG` (zasilanie z sieci przez USB DevKit)
   - USB / 5V/VIN ↔ HC-SR04 VCC
   - GND wspólne
   - GPIO5 ↔ Trig
   - Echo → R1 (1k) → GPIO19; węzeł → R2 (2k) → GND
   - C1 opcjonalnie między VCC a GND przy U2
5. **Inspect → Electrical Rules Checker** — `PWR_FLAG` tylko na zewnętrznym +5 V (USB), nie na wyjściach DevKit.
6. **File → Plot** → PDF do `docs/schematics/garage-gate-sens-hc-sr04.pdf`.
