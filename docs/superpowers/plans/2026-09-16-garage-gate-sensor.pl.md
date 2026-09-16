# Garage Gate Sensor — plan implementacji

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Czujnik otwarcia bramy garażowej (ESP32 + VL53L1X) z % otwarcia, stanem open/closed, czasem otwarcia i alertem „za długo”, raportowany przez ESPHome do Home Assistant; schemat KiCad + dokumentacja połączeń.

**Architecture:** Stałe zasilanie 5 V USB → ESP32 DevKit V1 → I²C (GPIO21/22) → moduł VL53L1X. ESPHome liczy `d_eff`, % z `mount_offset_mm` i `clearance_full_open_mm`, histerezę i czas otwarcia na urządzeniu. HA: dashboard + automat powiadomień. KiCad: schemat ideowy z footprintami modułów (bez routingu PCB v1).

**Tech Stack:** ESPHome 2026.8 (jak tank-level), ESP32 Arduino (`framework: arduino`, `board: esp32dev`), external component `github://jasonwbarnett/esphome-vl53l1x-pololu` (`vl53l1x_pololu`), KiCad 8/9, Home Assistant.

**Spec:** [`docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md`](../specs/2026-09-16-garage-gate-sensor-design.md)

## Global Constraints

- Brak deep sleep, baterii, MOSFET-a zasilania czujnika (v1).
- I²C: SDA GPIO21, SCL GPIO22; VL53L1X VIN z 3V3 DevKit; SHUT → 3V3.
- Kalibracja wyłącznie przez `substitutions` w YAML (bez encji `number` w HA w v1).
- `span = (distance_closed_mm - mount_offset_mm) - clearance_full_open_mm` musi być > 0; inaczej status `invalid_calibration`.
- Progi histerezy domyślnie: open ≥ 10 %, close ≤ 5 %.
- `max_open_seconds` domyślnie 900 (15 min).
- Nie reimplementować % w HA.

---

## Mapa plików

| Plik | Odpowiedzialność |
|------|------------------|
| `.gitignore` | ESPHome secrets, KiCad cache, `.history` |
| `README.pl.md` / `README.en.md` | Opis projektu, GPIO, build, linki do docs |
| `docs/README.md` | Indeks dokumentacji |
| `esphome/garage-gate-sensor.yaml` | Firmware |
| `esphome/secrets.yaml.example` | Szablon sekretów |
| `esphome/README.pl.md` | Build, logi, strojenie kalibracji |
| `docs/schematics/connections.pl.md` | Tabela połączeń |
| `docs/schematics/connections.en.md` | EN mirror |
| `docs/home-assistant/example-automation.yaml` | Przykład notify przy `open_too_long` |
| `docs/kicad/garage-gate-sens-devkit/*` | Projekt KiCad |
| `symbols/garage-gate.kicad_sym` (w katalogu KiCad) | Uproszczony ESP32 DevKit + opcjonalnie moduł ToF |

---

### Task 1: Szkielet repozytorium

**Files:**
- Create: `.gitignore`, `README.pl.md`, `README.en.md`, `docs/README.md`, `esphome/secrets.yaml.example`

**Interfaces:**
- Produces: struktura katalogów zgodna ze spec §9.

- [ ] **Step 1: `.gitignore`**

```gitignore
.esphome/
.secrets.yaml
esphome/secrets.yaml
**/.history/
**/*-backups/
**/*.kicad_prl
```

- [ ] **Step 2: `esphome/secrets.yaml.example`**

```yaml
wifi_ssid: "YOUR_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
garage_gate_api_encryption_key: "GENERATE_32_CHAR_KEY"
```

- [ ] **Step 3: Krótkie README** — skopiuj styl z `tank-level/README.pl.md`: hardware table, link do spec i planu, komenda `esphome run esphome/garage-gate-sensor.yaml`.

- [ ] **Step 4: Weryfikacja**

```bash
cd /Users/mkalkowski/Projects/SiteProjects/ESP32/GageGateSens
ls -la esphome docs
```

Expected: katalogi istnieją.

- [ ] **Step 5: Commit**

```bash
git add .gitignore README.pl.md README.en.md docs/README.md esphome/secrets.yaml.example
git commit -m "chore: add repo skeleton and ESPHome secrets template"
```

---

### Task 2: ESPHome — I²C, VL53L1X, surowy dystans

**Files:**
- Create: `esphome/garage-gate-sensor.yaml`, `esphome/README.pl.md`

**Interfaces:**
- Produces: sensor `gate_distance_raw` (mm), id `gate_distance_raw` na komponencie distance.

- [ ] **Step 1: Minimalny YAML z external component**

```yaml
substitutions:
  device_name: garage-gate
  friendly_name: Garage Gate Sensor
  log_level: INFO
  pin_i2c_sda: GPIO21
  pin_i2c_scl: GPIO22
  # Placeholdery montażu — zmień po instalacji
  clearance_full_open_mm: "80"
  mount_offset_mm: "0"
  distance_closed_mm: "1200"
  distance_open_mm: "80"
  open_threshold_percent: "10"
  close_threshold_percent: "5"
  max_open_seconds: "900"

external_components:
  - source: github://jasonwbarnett/esphome-vl53l1x-pololu
    components: [vl53l1x_pololu]

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:
  level: ${log_level}

api:
  encryption:
    key: !secret garage_gate_api_encryption_key

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "${friendly_name} Fallback"
    password: "configure-me"

i2c:
  sda: ${pin_i2c_sda}
  scl: ${pin_i2c_scl}
  scan: true
  frequency: 100kHz

vl53l1x_pololu:
  distance_mode: long
  timing_budget: 100ms
  inter_measurement: 150ms

sensor:
  - platform: vl53l1x_pololu
    distance:
      id: gate_distance_raw
      name: "Gate Distance Raw"
      unit_of_measurement: "mm"
      accuracy_decimals: 0
      update_interval: 500ms
      filters:
        - sliding_window_moving_average:
            window_size: 5
            send_every: 1
```

- [ ] **Step 2: Kompilacja (lokalnie, gdy masz secrets)**

```bash
esphome compile esphome/garage-gate-sensor.yaml
```

Expected: `SUCCESS` (bez flash).

- [ ] **Step 3: Flash + logi biurkowe**

```bash
esphome run esphome/garage-gate-sensor.yaml
esphome logs esphome/garage-gate-sensor.yaml
```

Expected: wartości mm zmieniają się przy ruchu celu; I²C scan widzi `0x29`.

- [ ] **Step 4: Commit**

```bash
git add esphome/garage-gate-sensor.yaml esphome/README.pl.md
git commit -m "feat(esphome): add VL53L1X raw distance via vl53l1x_pololu"
```

---

### Task 3: ESPHome — d_eff, %, status kalibracji

**Files:**
- Modify: `esphome/garage-gate-sensor.yaml`

**Interfaces:**
- Consumes: `id(gate_distance_raw).state` (float, mm).
- Produces: `gate_distance_effective`, `gate_open_percent`, `gate_sensor_status` (text).

- [ ] **Step 1: Globals na ostatni dobry odczyt**

```yaml
globals:
  - id: last_good_raw_mm
    type: float
    restore_value: no
    initial_value: "0"
  - id: last_good_ms
    type: unsigned long
    restore_value: no
    initial_value: "0"
```

- [ ] **Step 2: Template sensory (lambda zgodna ze spec §3)**

```yaml
sensor:
  - platform: template
    name: "Gate Distance Effective"
    id: gate_distance_effective
    unit_of_measurement: "mm"
    accuracy_decimals: 0
    update_interval: 1s
    lambda: |-
      const float raw = id(gate_distance_raw).state;
      if (isnan(raw)) return NAN;
      id(last_good_raw_mm) = raw;
      id(last_good_ms) = millis();
      const float mount = atof("${mount_offset_mm}");
      return raw - mount;

  - platform: template
    name: "Gate Open Percent"
    id: gate_open_percent
    unit_of_measurement: "%"
    accuracy_decimals: 1
    update_interval: 1s
    lambda: |-
      float raw = id(gate_distance_raw).state;
      if (isnan(raw)) {
        if (millis() - id(last_good_ms) > 30000) return NAN;
        raw = id(last_good_raw_mm);
      }
      const float mount = atof("${mount_offset_mm}");
      const float d_eff = raw - mount;
      const float d_closed_eff = atof("${distance_closed_mm}") - mount;
      const float d_open_eff = atof("${clearance_full_open_mm}");
      const float span = d_closed_eff - d_open_eff;
      if (span <= 0.0f) return NAN;
      float pct = 100.0f * (d_closed_eff - d_eff) / span;
      if (pct < 0.0f) pct = 0.0f;
      if (pct > 100.0f) pct = 100.0f;
      return pct;

  - platform: template
    name: "Gate Sensor Status"
    id: gate_sensor_status
    update_interval: 5s
    lambda: |-
      const float mount = atof("${mount_offset_mm}");
      const float d_closed_eff = atof("${distance_closed_mm}") - mount;
      const float d_open_eff = atof("${clearance_full_open_mm}");
      if ((d_closed_eff - d_open_eff) <= 0.0f) return std::string("invalid_calibration");
      const float raw = id(gate_distance_raw).state;
      if (isnan(raw) && (millis() - id(last_good_ms) > 30000)) return std::string("no_reading");
      return std::string("ok");
```

- [ ] **Step 3: Kompilacja**

```bash
esphome compile esphome/garage-gate-sensor.yaml
```

Expected: SUCCESS.

- [ ] **Step 4: Test biurkowy** — przybliż karton: % rośnie; ustaw `distance_closed_mm` ~ aktualny raw przy „daleko”, `clearance_full_open_mm` ~ `d_eff` przy „blisko”.

- [ ] **Step 5: Commit**

```bash
git commit -am "feat(esphome): effective distance, open percent, sensor status"
```

---

### Task 4: ESPHome — open/closed, czas otwarcia, too long

**Files:**
- Modify: `esphome/garage-gate-sensor.yaml`

**Interfaces:**
- Consumes: `id(gate_open_percent).state`.
- Produces: `gate_is_open` (global bool), binary `gate_open`, sensor `gate_open_duration_s`, binary `gate_open_too_long`.

- [ ] **Step 1: Globals histerezy i czasu**

```yaml
globals:
  - id: gate_is_open
    type: bool
    restore_value: no
    initial_value: "false"
  - id: open_since_ms
    type: unsigned long
    restore_value: no
    initial_value: "0"
```

- [ ] **Step 2: Interval 1s — aktualizacja latch + open_since**

```yaml
interval:
  - interval: 1s
    then:
      - lambda: |-
          const float pct = id(gate_open_percent).state;
          if (isnan(pct)) return;
          const float open_th = atof("${open_threshold_percent}");
          const float close_th = atof("${close_threshold_percent}");
          if (!id(gate_is_open) && pct >= open_th) {
            id(gate_is_open) = true;
            id(open_since_ms) = millis();
          } else if (id(gate_is_open) && pct <= close_th) {
            id(gate_is_open) = false;
          }
```

- [ ] **Step 3: Binary i duration**

```yaml
binary_sensor:
  - platform: template
    name: "Gate Open"
    id: gate_open
    device_class: garage_door
    lambda: |-
      return id(gate_is_open);

  - platform: template
    name: "Gate Open Too Long"
    id: gate_open_too_long
    lambda: |-
      if (!id(gate_is_open)) return false;
      const float max_s = atof("${max_open_seconds}");
      const float elapsed = (millis() - id(open_since_ms)) / 1000.0f;
      return elapsed > max_s;

sensor:
  - platform: template
    name: "Gate Open Duration"
    id: gate_open_duration_s
    unit_of_measurement: "s"
    accuracy_decimals: 0
    update_interval: 1s
    lambda: |-
      if (!id(gate_is_open)) return 0.0f;
      return (millis() - id(open_since_ms)) / 1000.0f;
```

- [ ] **Step 4: Test** — symuluj open (pct > 10): duration rośnie; ustaw `max_open_seconds: "10"` tymczasowo → `Gate Open Too Long` ON po 10 s.

- [ ] **Step 5: Commit**

```bash
git commit -am "feat(esphome): hysteresis, open duration, too-long binary"
```

---

### Task 5: Dokumentacja połączeń

**Files:**
- Create: `docs/schematics/connections.pl.md`, `docs/schematics/connections.en.md`

**Interfaces:**
- Produces: tabele pinów zgodne z KiCad (Task 6).

- [ ] **Step 1: Tabela PL** (nagłówek jak tank-level `connections-ao3401a.pl.md`):

| ESP32 DevKit | VL53L1X | Uwagi |
|--------------|---------|--------|
| 3V3 | VIN | |
| GND | GND | |
| GPIO21 | SDA | I²C |
| GPIO22 | SCL | I²C |
| 3V3 | SHUT | stałe Wł. |
| 5V/VIN | — | z USB 5 V |

Sekcja **Kalibracja YAML**: opis `mount_offset_mm`, `clearance_full_open_mm`, `distance_closed_mm`, walidacja `distance_open_mm ≈ clearance + mount`.

- [ ] **Step 2: Kopia EN** — ten sam układ.

- [ ] **Step 3: Commit**

```bash
git add docs/schematics/
git commit -m "docs: add connection tables PL/EN"
```

---

### Task 6: KiCad — schemat ideowy + footprinty

**Files:**
- Create: `docs/kicad/garage-gate-sens-devkit/garage-gate-sens-devkit.kicad_pro`, `.kicad_sch`, `README.pl.md`, opcjonalnie `symbols/garage-gate.kicad_sym`
- Create: `docs/schematics/garage-gate-sens-devkit.pdf` (eksport ręczny z KiCad)

**Interfaces:**
- Produces: PDF i ERC bez błędów krytycznych; net labels `+5V`, `+3V3`, `GND`, `I2C_SDA`, `I2C_SCL`.

- [ ] **Step 1: Skopiuj wzorzec z tank-level** — uproszczony symbol ESP32 z `tank-level/docs/kicad/tank-level-devkit/symbols/` (tylko piny: 5V, GND, 3V3, GPIO21, GPIO22) lub narysuj blok tekstowy + pin headers 2×19 footprint.

- [ ] **Step 2: Schemat** — `PWR` USB 5V → DevKit; moduł VL53L1X jako 6-pin connector; C1 100nF na 3V3; opcjonalnie C2 bulk 470µF na 5V (DNP dozwolone).

- [ ] **Step 3: ERC** w KiCad — popraw PWR_FLAG na liniach zasilania jeśli wymagane (patrz tank-level README o PWR_FLAG).

- [ ] **Step 4: Plot PDF** → `docs/schematics/garage-gate-sens-devkit.pdf`; w `connections.pl.md` dodaj link.

- [ ] **Step 5: Commit**

```bash
git add docs/kicad/ docs/schematics/garage-gate-sens-devkit.pdf
git commit -m "docs(kicad): garage gate devkit schematic and PDF export"
```

---

### Task 7: Home Assistant — przykład automatu

**Files:**
- Create: `docs/home-assistant/example-automation.yaml`

- [ ] **Step 1: Przykład**

```yaml
# Wklej do automations.yaml lub UI; encje po dodaniu urządzenia ESPHome mogą mieć prefiks esphome.
automation:
  - alias: "Garage gate open too long notify"
    trigger:
      - platform: state
        entity_id: binary_sensor.garage_gate_sensor_gate_open_too_long
        to: "on"
    action:
      - service: notify.mobile_app
        data:
          title: "Brama garażowa"
          message: "Brama otwarta dłużej niż limit — sprawdź garaż."
```

- [ ] **Step 2: W `esphome/README.pl.md`** — sekcja „Dodanie do HA”, strojenie `substitutions`, sprawdzenie `Gate Sensor Status == ok`.

- [ ] **Step 3: Commit**

```bash
git add docs/home-assistant/ esphome/README.pl.md
git commit -m "docs: HA automation example and ESPHome tuning notes"
```

---

### Task 8: Weryfikacja końcowa (DoD ze spec §7)

**Files:** brak nowych — checklist.

- [ ] **Step 1: Compile**

```bash
esphome compile esphome/garage-gate-sensor.yaml
```

- [ ] **Step 2: Montaż** — brama zamknięta ~0 %, pełne otwarcie ~100 % po strojeniu offset/clearance.

- [ ] **Step 3: Histereza** — brak fluttera `Gate Open` na granicy.

- [ ] **Step 4: Czas + too long** — zgodnie z Task 4.

- [ ] **Step 5: KiCad PDF vs connections.pl.md** — te same piny.

- [ ] **Step 6: Push (opcjonalnie)**

```bash
git push -u origin main
```

---

## Self-review (plan vs spec)

| Wymaganie spec | Task |
|----------------|------|
| VL53L1X I²C, 5 V, GPIO21/22 | 2, 5, 6 |
| mount_offset + clearance_full_open | 3 |
| Encje HA (wszystkie) | 2–4 |
| Histereza, filtry, 500 ms / 1 s | 2–4 |
| KiCad + PDF + connections | 5–6 |
| HA notify example | 7 |
| DoD testy | 8 |
| Poza zakresem v1 | nie ma tasków |

Brak TBD w planie. Nazwy encji ESPHome mogą wymagać dopasowania `entity_id` w automacie po pierwszym parowaniu z HA.

## Ryzyka

| Ryzyko | Mitygacja |
|--------|-----------|
| Brak natywnego `vl53l1x` w core ESPHome | Pin `jasonwbarnett/esphome-vl53l1x-pololu` w YAML |
| Wibracje łańcucha | `window_size: 5`, histereza 10/5 % |
| Złe `span` po montażu | `invalid_calibration` + log w README jak stroić |
| Nazwa encji HA inna niż w przykładzie | Sprawdź w HA → Developer Tools → States |
