# KiCad — garage-gate-sens-hc-sr04

Schemat ideowy: ESP32 DevKit V1 + HC-SR04 + dzielnik Echo 1 kΩ / 2 kΩ.

**Źródło prawdy połączeń:** [`../../schematics/connections-hc-sr04.pl.md`](../../schematics/connections-hc-sr04.pl.md)

## Otwarcie

```
docs/kicad/garage-gate-sens-hc-sr04/garage-gate-sens-hc-sr04.kicad_pro
```

Biblioteka: `symbols/garage-gate.kicad_sym` — `ESP32_DEVKIT`, `HC_SR04`. Rezystory: `Device:R` z biblioteki KiCad.

## Elementy

| Ref | Część | Uwagi |
|-----|--------|--------|
| U1 | ESP32_DEVKIT | **USB** (5 V in), 5V/VIN out, GND, GPIO5, GPIO19 |
| U2 | HC_SR04 | VCC←VIN, GND, Trig, Echo |
| R1 | 1 kΩ | Echo → GPIO19 (series) |
| R2 | 2 kΩ | węzeł → GND |
| C1 | 100 nF | **opcjonalny** VCC–GND przy module |
| +5V / PWR_FLAG | na pinie USB | zasilanie z sieci (USB) — ERC |

## Kroki

Patrz [STEP1.pl.md](STEP1.pl.md). Po ERC: Plot PDF → `docs/schematics/garage-gate-sens-hc-sr04.pdf` (gdy narysujesz schemę).
