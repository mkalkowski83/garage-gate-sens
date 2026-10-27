# Garage Gate Sensor — plan implementacji

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Czujnik otwarcia bramy garażowej (ESP32 + VL53L1X) z % otwarcia, stanem open/closed, czasem otwarcia i alertem „za długo”, raportowany przez ESPHome do Home Assistant; schemat KiCad + dokumentacja połączeń.

**Architecture:** Zasilanie USB → ESP32 DevKit V1 (`5V/VIN`) → regulator DevKit → `3V3` zasila VL53L1X (`VIN` + `SHUT`); I²C GPIO21/22. ESPHome liczy `d_eff`, % z `mount_offset_mm` i `clearance_full_open_mm`, histerezę i czas otwarcia. HA: dashboard + automat powiadomień. KiCad: schemat ideowy (bez C2, bez footprintów/routingu PCB w v1).

**Tech Stack:** ESPHome 2026.8 (jak tank-level), ESP32 Arduino (`framework: arduino`, `board: esp32dev`), external component `github://jasonwbarnett/esphome-vl53l1x-pololu` (`vl53l1x_pololu`), KiCad 8/9, Home Assistant.

**Spec:** [`docs/superpowers/specs/2026-09-16-garage-gate-sensor-design.md`](../specs/2026-09-16-garage-gate-sensor-design.md)

## Global Constraints

- Brak deep sleep, baterii, MOSFET-a zasilania czujnika, bulk C2 na +5 V (v1).
- Zasilanie całego układu przez USB DevKit; czujnik tylko z ESP **3V3**.
- I²C: SDA GPIO21 (D21), SCL GPIO22 (D22); VL53L1X VIN i SHUT z 3V3; INT = NC.
- KiCad: bez `PWR_FLAG` na 3V3 (pin ESP `3V3` jest już `power_out`).
- Kalibracja wyłącznie przez `substitutions` w YAML (bez encji `number` w HA w v1).
- `span = (distance_closed_mm - mount_offset_mm) - clearance_full_open_mm` musi być > 0; inaczej status `invalid_calibration`.
- Progi histerezy domyślnie: open ≥ 10 %, close ≤ 5 %.
- `max_open_seconds` domyślnie 900 (15 min).
- Nie reimplementować % w HA.
- Footprinty odroczone (v1 = schemat ideowy).

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
| `symbols/garage-gate.kicad_sym` (w katalogu KiCad) | `ESP32_DEVKIT` (5 pinów) + `VL53L1X` (VIN…SHUT) |

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
gdos_sensor__api_key: "GENERATE_32_CHAR_KEY"
gdos_sensor__ota_password: "YOUR_OTA_PASSWORD"
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
    key: !secret gdos_sensor__api_key

ota:
  - platform: esphome
    password: !secret gdos_sensor__ota_password

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
| USB → 5V/VIN | — | Zasilanie całego układu przez DevKit |
| 3V3 | VIN, SHUT | Bez C2 na +5 V |
| GND | GND | |
| GPIO21 (D21) | SDA | I²C |
| GPIO22 (D22) | SCL | I²C |
| — | INT | NC |

Sekcja **Kalibracja YAML**: opis `mount_offset_mm`, `clearance_full_open_mm`, `distance_closed_mm`, walidacja `distance_open_mm ≈ clearance + mount`.

- [ ] **Step 2: Kopia EN** — ten sam układ.

- [ ] **Step 3: Commit**

```bash
git add docs/schematics/
git commit -m "docs: add connection tables PL/EN"
```

---

### Task 6: KiCad — schemat ideowy (bez footprintów)

**Files:**
- Modify: `docs/kicad/garage-gate-sens-devkit/garage-gate-sens-devkit.kicad_sch`, `STEP1.pl.md`, `README.pl.md`, `symbols/garage-gate.kicad_sym`
- Create: `docs/schematics/garage-gate-sens-devkit.pdf` (eksport ręczny z KiCad)

**Interfaces:**
- Produces: ERC clean; refs **U2**=ESP32, **U1**=VL53L1X, **C1**=100 nF; brak C2; INT=`no_connect`.

- [x] **Step 1: Symbole** — `ESP32_DEVKIT` (tylko 5V/VIN, GND, 3V3, GPIO21/SDA, GPIO22/SCL) + `VL53L1X` (VIN…SHUT).

- [x] **Step 2: Schemat** — USB/+5V → U2 `5V/VIN`; 3V3 → U1 VIN+SHUT + C1 100 nF do GND; I²C SDA/SCL; INT NC. **Bez C2.**

- [x] **Step 3: ERC** — `PWR_FLAG` tylko na zewnętrznym +5V jeśli potrzebny; **nie** na 3V3 (kolizja z `power_out` pinu ESP).

- [ ] **Step 4: Usuń C2** z arkusza jeśli jeszcze obecny; zapisz `.kicad_sch`.

- [ ] **Step 5: Plot PDF** → `docs/schematics/garage-gate-sens-devkit.pdf`; link w `connections*.md`.

- [ ] **Step 6: Commit** (po akceptacji użytkownika)

```bash
git add docs/kicad/ docs/schematics/
git commit -m "docs(kicad): finalize garage gate schematic without C2"
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
| KiCad ERC: `PWR_FLAG` on 3V3 | Do not place it — ESP `3V3` is already `power_out` |
