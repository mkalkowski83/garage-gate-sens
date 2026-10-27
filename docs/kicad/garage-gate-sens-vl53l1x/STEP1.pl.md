# KiCad — krok 1: ESP32 + VL53L1X (zasilanie przez USB DevKit)

Schemat ideowy v1: USB zasila ESP32; **3V3 z DevKit** zasila VL53L1X. Odpowiednik tabeli: [`../../schematics/connections.pl.md`](../../schematics/connections.pl.md).

## Otwarcie projektu

1. KiCad **8** lub **9** → **File → Open Project** → `docs/kicad/garage-gate-sens-vl53l1x/garage-gate-sens-devkit.kicad_pro`
2. Schematic Editor → biblioteka `garage-gate` (w `sym-lib-table`).

## Elementy (rev 0.2)

| Ref | Element |
|-----|---------|
| — | `power:+5V` + `power:GND` (USB / zasilanie DevKit) |
| U2 | `garage-gate:ESP32_DEVKIT` (piny: 5V/VIN, GND, 3V3, GPIO21/D21/SDA, GPIO22/D22/SCL) |
| U1 | `garage-gate:VL53L1X` (piny 1–6: VIN, GND, SDA, SCL, INT, SHUT) |
| C1 | 100 nF — między **3V3** i **GND** przy module ToF |

**Nie używać:** C2 (bulk na +5 V) — zbędny przy zasilaniu przez USB ESP.

## Połączenia

| Net | Połączenie |
|-----|------------|
| +5V | → U2 `5V/VIN` (USB w DevKit) |
| GND | U2, U1 pin 2, C1 |
| 3V3 | U2 `3V3` → U1 `VIN` (1), U1 `SHUT` (6), C1 |
| I2C_SDA | U2 `GPIO21/D21/SDA` → U1 `SDA` (3) |
| I2C_SCL | U2 `GPIO22/D22/SCL` → U1 `SCL` (4) |
| — | U1 `INT` (5) — **no_connect** |

## ERC

1. `PWR_FLAG` tylko na zewnętrznym **+5V**, jeśli ERC tego wymaga.
2. **Nie** stawiać `PWR_FLAG` na **3V3** — pin U2 `3V3` jest `power_out` (kolizja ERC).
3. INT musi mieć flagę **no connect**.

## PDF

**File → Plot** → `docs/schematics/garage-gate-sens-devkit.pdf`.

Footprinty: odroczone w v1 (schemat ideowy).
