# Home Assistant Automasjoner - Versjon 2.0

## Oversikt

Dette er en komplett forbedret versjon av dine Home Assistant-automasjoner med fokus på stabilitet, konfigurerbarhet og bedre feilhåndtering.

## 📦 Genererte filer

1. **automations_improved.yaml** - Alle forbedrede automasjoner
2. **configuration_helpers.yaml** - Input_number hjelpere for konfigurasjon
3. **configuration_input_boolean.yaml** - Input_boolean hjelpere for tilstandssporing

---

## ✨ Hva er forbedret?

### 🔴 KRITISKE FORBEDRINGER

#### 1. Peak-kontroll: Stabilere retrigger-logikk
**Problem:** Time_pattern trigger hver 2. minutt kunne kutte neste enhet før effekten av forrige kutt var synlig.

**Løsning:**
- Økt retrigger-intervall fra 2 til **5 minutter**
- Lagt til **3 minutters delay** etter hvert kutt
- Dette gir systemet tid til å stabilisere seg

```yaml
# FØR:
trigger:
  - platform: time_pattern
    minutes: "/2"

# ETTER:
trigger:
  - platform: time_pattern
    minutes: "/5"
action:
  - delay: "00:03:00"  # Ny delay etter hvert kutt
```

#### 2. VVB smart styring aktiveres kun når nødvendig
**Problem:** `input_boolean.vvb_smart_styring_aktiv` ble reaktivert ALLTID ved normalisering, selv om VVB aldri ble kuttet.

**Løsning:** Smart styring aktiveres nå KUN når VVB faktisk gjenopprettes:

```yaml
# Flyttet fra toppen av normaliseringen til kun VVB-seksjonen
- if:
    - condition: state
      entity_id: input_boolean.peak_kontroll_vvb_deaktivert
      state: "on"
  then:
    - service: input_boolean.turn_on
      target:
        entity_id: input_boolean.vvb_smart_styring_aktiv
```

#### 3. Varmepumpe tørkesyklus: Lagrer og gjenoppretter modus
**Problem:** Tørkesyklusen endret alltid til `cool` etter endt syklus, uavhengig av tidligere modus.

**Løsning:** Lagrer nåværende modus før tørking og gjenoppretter den etterpå:

```yaml
action:
  - variables:
      previous_mode: "{{ states('climate.vp_datarom') }}"
  # ... kjør tørkesyklus ...
  - service: climate.set_hvac_mode
    data:
      hvac_mode: "{{ previous_mode }}"
```

---

### 🟡 VIKTIGE FORBEDRINGER

#### 4. Tilgjengelighetssjekker overalt
**Problem:** Automasjoner forsøkte å kontrollere enheter uten å sjekke om de var tilgjengelige.

**Løsning:** Lagt til sjekker i alle conditions:

```yaml
- condition: template
  value_template: "{{ states('sensor.navn') not in ['unavailable', 'unknown'] }}"
```

#### 5. Gang lys: Fjernet dobbel-kommando
**Problem:** To identiske light.turn_on kommandoer med 5 sek delay kunne forårsake flimring.

**Løsning:** Redusert til én kommando med lengre transition:

```yaml
# FØR: To kommandoer med delay
- service: light.turn_on
  data:
    transition: 4
- delay: "00:00:05"
- service: light.turn_on
  data:
    transition: 2

# ETTER: Én kommando
- service: light.turn_on
  data:
    transition: 5
```

#### 6. Kino: Økt timeout for receiver
**Problem:** 20 sekunders timeout var for kort ved kald oppstart.

**Løsning:** Økt til **45 sekunder** med bedre feilhåndtering:

```yaml
- wait_template: >
    {{ states('media_player.rx_v685_3c03b4') not in ['off', 'unavailable', 'unknown'] }}
  timeout:
    seconds: 45
  continue_on_timeout: true
- if:
    - condition: template
      value_template: "{{ receiver er klar }}"
  then:
    # Sett volum
  else:
    # Logg advarsel
```

#### 7. Notifikasjoner ved normalisering
**Problem:** Ingen tilbakemelding når enheter ble skrudd på igjen.

**Løsning:** Lagt til persistent_notification for alle gjenopprettinger:

```yaml
- service: persistent_notification.create
  data:
    title: "Peak-kontroll - VP Kontor gjenopprettet"
    message: >
      Prognose: {{ states('sensor.prognose_for_innevaerende_time') }} kWh.
      Varmepumpe kontor aktivert igjen.
```

---

### 🟢 MINDRE FORBEDRINGER

#### 8. Konfigurerbare verdier via input_number
**Problem:** Hardkodede verdier krevde kode-endringer for justering.

**Løsning:** Alle viktige terskler kan nå justeres via UI:

| Parameter | Standard | Min | Max | Bruksområde |
|-----------|----------|-----|-----|-------------|
| peak_kontroll_av_grense | 4.7 kWh | 3.0 | 10.0 | Når enheter kuttes |
| peak_kontroll_paa_grense | 3.8 kWh | 2.0 | 8.0 | Når enheter gjenopprettes |
| billader_maks_temperatur | 75°C | 60 | 85 | Overoppheting nødstopp |
| billader_trygg_temperatur | 60°C | 40 | 70 | Gjenoppstart lading |
| kino_standard_volum | 0.575 | 0.0 | 1.0 | Receiver volum |
| vp_datarom_strong_terskel_paa | 3.0°C | 1.0 | 5.0 | Aktivere strong fan |
| vp_datarom_strong_terskel_av | 2.5°C | 0.5 | 4.0 | Deaktivere strong fan |
| gang_lys_dag_brightness | 254 | 50 | 255 | Lysstyrke dag |
| gang_lys_natt_brightness | 8 | 1 | 50 | Lysstyrke natt |

#### 9. Effektledd: Forbedret timing
**Problem:** Kjørte 10 sek FØR timeskiftet, sensoren kunne være utdatert.

**Løsning:** Kjører nå 30 sekunder ETTER timeskiftet:

```yaml
# FØR:
trigger:
  - platform: time_pattern
    minutes: 59
    seconds: 50

# ETTER:
trigger:
  - platform: time_pattern
    hours: "*"
    minutes: 0
    seconds: 30
```

#### 10. Robotstøvsuger: Batterisjekk
**Problem:** Startet støvsugere uten å sjekke batterinivå.

**Løsning:** Sjekker at batteri er minst 20% før start:

```yaml
condition:
  - condition: template
    value_template: >
      {{ state_attr('vacuum.roborock_s7_maxv', 'battery_level') | int(0) >= 20 }}
```

#### 11. Forbedret logging
Alle automasjoner logger nå viktige hendelser til system_log:

```yaml
- service: system_log.write
  data:
    message: "Peak-kontroll: VVB kuttet. Prognose=4.9 kWh"
    level: warning
```

Nivåer:
- **error**: Kritiske feil (overoppheting)
- **warning**: Viktige hendelser (peak-kutt)
- **info**: Normale hendelser (gjenopprettinger)
- **debug**: Detaljert info (lys-justeringer)

---

## 📋 Installasjonsveiledning

### Steg 1: Backup
```bash
# Ta backup av eksisterende konfigurasjon
cd /config
cp automations.yaml automations.yaml.backup
```

### Steg 2: Opprett hjelpere

#### Via Home Assistant UI (ANBEFALT):

1. Gå til **Innstillinger** → **Enheter og tjenester** → **Hjelpere**

2. Opprett **Input numbers** (klikk "+ OPPRETT HJELPER" → "Nummer"):

```
Navn: Peak-kontroll - Avslåingsgrense
Minimum: 3.0
Maksimum: 10.0
Trinn: 0.1
Måleenhet: kWh
Ikon: mdi:flash-alert
Standardverdi: 4.7
```

Gjenta for alle i `configuration_helpers.yaml`.

3. Opprett **Input booleans** (klikk "+ OPPRETT HJELPER" → "Bryter"):

```
Navn: VVB - Smart styring aktiv
Ikon: mdi:water-boiler
Initial tilstand: PÅ
```

Gjenta for alle i `configuration_input_boolean.yaml`.

#### Via YAML (Alternativ):

Legg til i `configuration.yaml`:

```yaml
input_number: !include configuration_helpers.yaml
input_boolean: !include configuration_input_boolean.yaml
```

Kopier filene til `/config/`:
```bash
cp configuration_helpers.yaml /config/
cp configuration_input_boolean.yaml /config/
```

### Steg 3: Installer automasjoner

**Metode A: Overskrive eksisterende (hvis du bruker automations.yaml)**
```bash
cp automations_improved.yaml /config/automations.yaml
```

**Metode B: Side ved side (testing)**
```bash
cp automations_improved.yaml /config/automations_v2.yaml
```

Legg til i `configuration.yaml`:
```yaml
automation: !include automations_v2.yaml
```

**Metode C: Via UI**
Kopier automasjoner én for én:
1. Gå til **Innstillinger** → **Automatiseringer og scener**
2. Klikk **+ OPPRETT AUTOMASJON** → **Start med tom**
3. Klikk **⋮** → **Rediger i YAML**
4. Lim inn automatiseringen

### Steg 4: Valider konfigurasjon

1. Gå til **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Hvis OK: Klikk **RASK-START**

Eller via CLI:
```bash
ha core check
ha core restart
```

### Steg 5: Sett standardverdier

Gå til **Innstillinger** → **Enheter og tjenester** → **Hjelpere** og juster verdiene:

- peak_kontroll_av_grense: **4.7**
- peak_kontroll_paa_grense: **3.8**
- billader_maks_temperatur: **75**
- billader_trygg_temperatur: **60**
- kino_standard_volum: **0.575**
- vp_datarom_strong_terskel_paa: **3.0**
- vp_datarom_strong_terskel_av: **2.5**
- gang_lys_dag_brightness: **254**
- gang_lys_natt_brightness: **8**

---

## 🧪 Testing

### Test 1: Peak-kontroll
1. Senk `peak_kontroll_av_grense` midlertidig til 0.5 kWh
2. Vent 2 minutter
3. Sjekk at enheter kuttes i riktig rekkefølge
4. Hev `peak_kontroll_paa_grense` til 10 kWh
5. Sjekk at enheter skrus på igjen med notifikasjoner

### Test 2: Billader overoppheting
1. Se nåværende temperatur på `sensor.shellyplus1pm_a8032ab13280_temperature`
2. Senk `billader_maks_temperatur` til litt under nåværende
3. Sjekk at billader kuttes med kritisk varsel
4. Hev `billader_trygg_temperatur` til over nåværende
5. Sjekk at billader starter igjen

### Test 3: Kino-synkronisering
1. Slå på én kino-enhet
2. Sjekk at alle enheter slås på
3. Vent på volumjustering (kan ta opptil 50 sekunder)
4. Slå av én enhet
5. Sjekk at alle slås av

### Test 4: Gang lys
1. Slå på gang lyset på dagtid
2. Sjekk at brightness er korrekt (ingen flimring)
3. Juster `gang_lys_dag_brightness`
4. Vent 15 minutter og sjekk at brightness oppdateres

---

## 🔧 Feilsøking

### Automasjoner trigges ikke

**Sjekk logger:**
```yaml
# Legg til i configuration.yaml for debugging
logger:
  default: info
  logs:
    homeassistant.components.automation: debug
```

**Sjekk tilstander:**
```bash
# I Developer Tools → Template
{{ states('sensor.prognose_for_innevaerende_time') }}
{{ states('input_number.peak_kontroll_av_grense') }}
```

### "Entity not available"

Sjekk at alle sensorer og enheter eksisterer:
- `sensor.prognose_for_innevaerende_time`
- `sensor.hourly_energy`
- `sensor.shellyplus1pm_a8032ab13280_temperature`
- Alle climate, switch, light, media_player enheter

### Input_number ikke funnet

Sørg for at hjelperne er opprettet. Sjekk:
```bash
# I Developer Tools → States
Filtrer på: input_number.peak
```

---

## 📊 Sammenligning: Før vs. Etter

| Funksjon | Før | Etter |
|----------|-----|-------|
| **Peak retrigger-intervall** | 2 min | 5 min |
| **Stabiliseringstid etter kutt** | 0 sek | 3 min |
| **Kino receiver timeout** | 20 sek | 45 sek |
| **Gang lys kommandoer** | 2x (flimring) | 1x (smooth) |
| **VVB smart styring logikk** | Alltid reaktivert | Kun ved behov |
| **Tørkesyklus VP** | Fast til cool | Gjenoppretter modus |
| **Notifikasjoner** | Kun ved kutt | Ved kutt OG gjenopprettelse |
| **Konfigurerbarhet** | Hardkodet | 9 justerbare parametre |
| **Tilgjengelighetssjekker** | Mangelfulle | Komplett |
| **Logging** | Minimal | Detaljert (4 nivåer) |
| **Effektledd timing** | 10 sek før | 30 sek etter |
| **Robotstøvsuger start** | Ingen sjekk | Batterisjekk 20% |

---

## 🎯 Anbefalte justeringer

Etter 1 uke med kjøring, vurder disse justeringene:

### Hvis peak-kontroll kutter for ofte:
- Øk `peak_kontroll_av_grense` fra 4.7 til 5.0 kWh

### Hvis peak-kontroll ikke kutter raskt nok:
- Senk `peak_kontroll_av_grense` fra 4.7 til 4.5 kWh
- Senk `peak_kontroll_paa_grense` tilsvarende (hold 0.9 kWh forskjell)

### Hvis gang lys er for sterkt/svakt:
- Juster `gang_lys_dag_brightness` og `gang_lys_natt_brightness`

### Hvis varmepumpe datarom ikke regulerer bra:
- Øk `vp_datarom_strong_terskel_paa` for mer aggressiv kjøling
- Mink forskjellen mellom på/av-terskel for mindre hysterese

---

## 📞 Support

Hvis du oppdager feil eller har spørsmål:
1. Sjekk Home Assistant loggen: **Innstillinger** → **System** → **Logger**
2. Aktiver debug logging (se Feilsøking)
3. Sjekk at alle enheter og sensorer faktisk eksisterer

---

## 📝 Lisens og ansvarsfraskrivelse

Disse automatiseringene er levert "som de er" uten noen garanti. Test grundig før produksjonsbruk.
Vær spesielt forsiktig med sikkerhetskritiske automasjoner som billader-overoppheting.

**Versjon:** 2.0
**Dato:** 2025-10-23
**Kompatibilitet:** Home Assistant 2024.1+
