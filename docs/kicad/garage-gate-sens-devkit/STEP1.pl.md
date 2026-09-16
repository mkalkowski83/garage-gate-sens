# KiCad — krok 1: ESP32 + VL53L1X (5 V USB)

Schemat ideowy v1: zasilanie stałe, DevKit, moduł ToF na I²C. Odpowiednik tabeli: [`../../schematics/connections.pl.md`](../../schematics/connections.pl.md).

> **Uwaga:** Plik `garage-gate-sens-devkit.kicad_sch` w repo jest kopią startową z projektu tank-level — **wyczyść schemat** (zostaw tylko elementy poniżej) lub zbuduj nowy arkusz według tej listy.

## Otwarcie projektu

1. KiCad **8** lub **9** → **File → Open Project** → `docs/kicad/garage-gate-sens-devkit/garage-gate-sens-devkit.kicad_pro`
2. Schematic Editor → dodaj bibliotekę `garage-gate` (już w `sym-lib-table`).

## Elementy (rev 0.1)

| Ref | Element |
|-----|---------|
| J1 | Zasilanie USB 5 V (symbol `power:+5V`, `power:GND`, opcjonalnie `Connector:USB_B_Micro`) |
| U1 | `garage-gate:ESP32_DEVKIT` |
| J2 | Moduł VL53L1X — `Connector_Generic:Conn_01x06` (etykiety: VIN, GND, SDA, SCL, INT, SHUT) |
| C1 | 100 nF — między +3V3 i GND przy module ToF |
| C2 | 470 µF (opcjonalnie) — +5V / GND przy wejściu |

## Połączenia

| Net | Połączenie |
|-----|------------|
| +5V | J1 → U1 `5V/VIN` |
| GND | J1, U1, J2 pin 2 |
| +3V3 | U1 `3V3` → J2 VIN, J2 SHUT |
| I2C_SDA | U1 `GPIO21/SDA` → J2 SDA |
| I2C_SCL | U1 `GPIO22/SCL` → J2 SCL |

## Footprinty (moduły)

| Ref | Footprint (przykład) |
|-----|----------------------|
| U1 | `Pin_Header_Straight_1x19` ×2 (DevKit na płytce — tylko dokumentacja) |
| J2 | `Pin_Header_Straight_1x06` 2,54 mm (breakout VL53L1X) |

## ERC i PDF

1. **Inspect → Electrical Rules Check** — popraw PWR_FLAG na liniach zasilania jeśli wymagane.
2. **File → Plot** → PDF do `docs/schematics/garage-gate-sens-devkit.pdf`.
