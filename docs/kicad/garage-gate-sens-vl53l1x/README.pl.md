# KiCad — Garage Gate Sensor (DevKit + VL53L1X)

Schemat ideowy prototypu: **USB → ESP32 → 3V3 → VL53L1X**.

**Krok 1:** [`STEP1.pl.md`](STEP1.pl.md)

## Otwarcie

```
docs/kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.kicad_pro
```

## Symbole własne

Biblioteka `garage-gate` w `symbols/garage-gate.kicad_sym`:

| Symbol | Opis |
|--------|------|
| `ESP32_DEVKIT` | Tylko **5V/VIN, GND, 3V3, GPIO21/D21/SDA, GPIO22/D22/SCL** |
| `VL53L1X` | Piny **VIN, GND, SDA, SCL, INT, SHUT** (1–6) |

## Schemat (rev 0.2)

| Ref | Element |
|-----|---------|
| U2 | ESP32 DevKit |
| U1 | VL53L1X |
| C1 | 100 nF (3V3–GND) |

Bez **C2**. INT = **no_connect**. Bez `PWR_FLAG` na 3V3. Footprinty odroczone.

## PDF

Eksport: `docs/schematics/garage-gate-sens-devkit.pdf`.
