# Smart Kapasitetsledd-system for Norge

## 🎯 Hva dette systemet gjør

**Holder deg under 5 kW kapasitetsledd** (konfigurerbart) ved å bruke **riktig norsk formel**:

```
Kapasitetsledd = (Topp #1 + Topp #2 + Topp #3) / 3
```

### ✅ Riktig logikk:

**Dag 1-2 i måneden (<3 topper):**
- Kutt aggressivt hvis prognose > 5.0 kWh

**Dag 3-31 i måneden (har 3 topper):**
- Beregn om prognosen faktisk vil øke kapasitetsleddet
- Kutt KUN hvis det påvirker topp-3 og overstiger målgrense
- Ikke sløs med komfort!

**Eksempel:**
```
Topp #1: 5.5 kWh
Topp #2: 5.2 kWh
Topp #3: 4.8 kWh
Kapasitetsledd: (5.5+5.2+4.8)/3 = 5.17 kW ❌ Over 5 kW!

Prognose nå: 4.5 kWh
→ Påvirker ikke topp-3 → IKKE KUTT (sløser med komfort)

Prognose nå: 5.0 kWh
→ Ny topp #3 → Kapasitetsledd = 5.23 kW → KUTT!
```

---

## 📦 Hva du får

### 1. Template sensorer:
- `sensor.kapasitetsledd_aktuelt` - Nåværende gjennomsnitt
- `sensor.kapasitetsledd_trygg_grense` - Hva kan jeg bruke nå?
- `sensor.kapasitetsledd_hvis_prognose` - Hva blir det hvis prognose realiseres?
- `sensor.kapasitetsledd_margin` - Margin til målgrense
- `sensor.kapasitetsledd_kutt_anbefaling` - Skal vi kutte? (KUTT NÅ / VARSLING / TRYGT)

### 2. Smart automasjoner:
- **Kapasitetsledd smart peak-kontroll** - Kutter basert på ekte logikk
- **Kapasitetsledd normalisering** - Gjenoppretter når trygt
- **Effektledd oppdater topp-3** - Logger timer riktig
- **Effektledd nullstill** - Nullstiller ved månedsskifte
- **+ Alle dine eksisterende automasjoner** (overoppheting, robotstøvsuger, kino, etc.)

### 3. Konfigurerbart:
- `input_number.kapasitetsledd_maal` - Målgrense (standard: 5.0 kW)
- `input_number.kapasitetsledd_kutt_hysterese_min` - Hvor lenge over før kutt (2 min)
- `input_number.kapasitetsledd_gjenopprett_hysterese_min` - Hvor lenge under før gjenopprett (5 min)

---

## 🚀 Installasjon (15 minutter)

### STEG 1: Backup (1 min)

```bash
# SSH til Home Assistant eller bruk Terminal add-on
cd /config
cp configuration.yaml configuration.yaml.backup
cp automations.yaml automations.yaml.backup
```

### STEG 2: Legg til template sensorer (5 min)

**Åpne `configuration.yaml`**

**Finn `template:` seksjonen.** Den ser sannsynligvis slik ut:

```yaml
template:
  - sensor:
      # Dine eksisterende sensorer:
      - name: "Leilighet (restforbruk)"
        # ...
      - name: "Prognose for inneværende time"
        # ...
```

**LEGG TIL** de nye sensorene **UNDER** dine eksisterende.

**Kopier ALT fra `configuration_kapasitetsledd_sensorer.yaml`** under `- sensor:` seksjonen.

Det skal se omtrent slik ut:

```yaml
template:
  - sensor:
      # EKSISTERENDE SENSORER (behold disse!)
      - name: "Leilighet (restforbruk)"
        unique_id: leilighet_restforbruk_avansert
        state: >
          # ... (din kode)

      - name: "Prognose for inneværende time"
        unique_id: prognose_for_timen_p1
        # ... (din kode)

      # NYE KAPASITETSLEDD-SENSORER (legg til disse!)
      - name: "Kapasitetsledd aktuelt"
        unique_id: kapasitetsledd_aktuelt
        unit_of_measurement: "kW"
        # ... (resten fra configuration_kapasitetsledd_sensorer.yaml)

      - name: "Kapasitetsledd trygg grense"
        # ...

      # osv... (alle 5 sensorer)

  # LEGG OGSÅ TIL binary_sensor SEKSJONEN (nederst i filen):
  - binary_sensor:
      - name: "Kapasitetsledd faresone"
        unique_id: kapasitetsledd_faresone
        device_class: problem
        state: >
          {% set margin = states('sensor.kapasitetsledd_margin') | float(0) %}
          {{ margin < 0.5 }}
```

### STEG 3: Legg til input_number (3 min)

**I samme `configuration.yaml` fil:**

**Finn `input_number:` seksjonen:**

```yaml
input_number:
  # Eksisterende:
  peak_1_verdi:
    name: "Effektledd - Peak #1 verdi"
    # ...
```

**LEGG TIL disse 3 nye:**

```yaml
input_number:
  # EKSISTERENDE (behold!)
  peak_1_verdi:
    # ...
  peak_2_verdi:
    # ...
  peak_3_verdi:
    # ...

  # NYE (legg til):
  kapasitetsledd_maal:
    name: "Kapasitetsledd - Målgrense"
    min: 2.0
    max: 25.0
    step: 0.5
    initial: 5.0
    unit_of_measurement: "kW"
    mode: box
    icon: mdi:target

  kapasitetsledd_kutt_hysterese_min:
    name: "Kapasitetsledd - Kutt hysterese (minutter)"
    min: 1
    max: 10
    step: 1
    initial: 2
    unit_of_measurement: "min"
    mode: box
    icon: mdi:timer-sand

  kapasitetsledd_gjenopprett_hysterese_min:
    name: "Kapasitetsledd - Gjenopprett hysterese (minutter)"
    min: 2
    max: 10
    step: 1
    initial: 5
    unit_of_measurement: "min"
    mode: box
    icon: mdi:timer-check
```

**LAGRE** filen.

### STEG 4: Sjekk og restart (2 min)

1. **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Vent på ✓ grønn melding
4. Klikk **RASK-START**
5. Vent 2 minutter

### STEG 5: Verifiser sensorer (1 min)

1. Gå til **Utviklerverktøy** → **Tilstander**
2. Søk etter `kapasitetsledd`
3. Du skal se:
   - `sensor.kapasitetsledd_aktuelt`
   - `sensor.kapasitetsledd_trygg_grense`
   - `sensor.kapasitetsledd_hvis_prognose`
   - `sensor.kapasitetsledd_margin`
   - `sensor.kapasitetsledd_kutt_anbefaling`
   - `binary_sensor.kapasitetsledd_faresone`
   - `input_number.kapasitetsledd_maal`
   - `input_number.kapasitetsledd_kutt_hysterese_min`
   - `input_number.kapasitetsledd_gjenopprett_hysterese_min`

### STEG 6: Installer automasjoner (2 min)

**Via File Editor (anbefalt):**
1. Åpne `automations.yaml`
2. SLETT alt innhold (du har backup!)
3. Åpne `automations_kapasitetsledd.yaml`
4. KOPIER alt
5. LIM INN i `automations.yaml`
6. LAGRE

**Via SSH/Terminal:**
```bash
cd /config
cp automations_kapasitetsledd.yaml automations.yaml
```

### STEG 7: Sjekk og restart (2 min)

1. **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Hvis OK: Klikk **RASK-START**

### STEG 8: Verifiser automasjoner (1 min)

1. Gå til **Innstillinger** → **Automatiseringer**
2. Sjekk at du ser:
   - ⚡ **Kapasitetsledd - Smart peak-kontroll** (ny!)
   - ✅ **Kapasitetsledd - Normalisering** (ny!)
   - 📊 **Effektledd - Oppdater topp-3**
   - 🔄 **Effektledd - Nullstill ved månedsskifte**
   - 🔥 **Overopphetingsvern - Skru AV Billader**
   - 🤖 **Start robotstøvsugere når jeg er borte**
   - 📺 **Kino: En på = alle på**
   - 🌡️ **Varmepumpe Datarom - Dynamisk fan speed**
   - 💡 **Gang lys - Smart justering**
   - (og alle dine andre)

3. Sjekk at ALLE er **aktivert** (blå toggle)

---

## 🧪 Testing (5 minutter)

### Test 1: Sjekk sensorer (2 min)

1. Gå til **Utviklerverktøy** → **Tilstander**
2. Finn `sensor.kapasitetsledd_aktuelt`
3. Klikk på den → Se **Attributter**

Skal vise:
```yaml
kapasitetsledd_aktuelt: X.XX kW
peak_1: X.XX
peak_2: X.XX
peak_3: X.XX
peak_1_tid: "DD.MM.YYYY (kl. HH-HH)"
# osv...
```

4. Sjekk `sensor.kapasitetsledd_kutt_anbefaling`
   - Skal vise: "TRYGT", "VARSLING", eller "KUTT NÅ!"

### Test 2: Simuler høy prognose (3 min)

**ADVARSEL:** Dette vil faktisk kutte enheter!

1. Gå til **Utviklerverktøy** → **Tilstander**
2. Finn `sensor.prognose_for_innevaerende_time`
3. Noter nåværende verdi (f.eks. 3.2 kWh)
4. Finn `input_number.kapasitetsledd_maal`
5. Sett den til **litt UNDER prognosen** (f.eks. 3.0 kW)
6. Vent 2-3 minutter
7. Sjekk at `sensor.kapasitetsledd_kutt_anbefaling` endrer seg til "KUTT NÅ!"
8. Sjekk at du får notifikasjon om at enheter kuttes
9. **VIKTIG:** Sett `kapasitetsledd_maal` tilbake til **5.0** kW!

---

## 📊 Dashboard (anbefalt)

Lag et nytt dashboard for å overvåke kapasitetsleddet:

### Kort 1: Kapasitetsledd oversikt

```yaml
type: entities
title: Kapasitetsledd Oversikt
entities:
  - entity: sensor.kapasitetsledd_aktuelt
    name: "Aktuelt kapasitetsledd"
  - entity: input_number.kapasitetsledd_maal
    name: "Målgrense"
  - entity: sensor.kapasitetsledd_margin
    name: "Margin til mål"
  - type: divider
  - entity: sensor.prognose_for_innevaerende_time
    name: "Prognose nå"
  - entity: sensor.kapasitetsledd_trygg_grense
    name: "Trygg grense"
  - entity: sensor.kapasitetsledd_hvis_prognose
    name: "Kapasitetsledd hvis prognose"
  - type: divider
  - entity: sensor.kapasitetsledd_kutt_anbefaling
    name: "Vurdering"
  - entity: binary_sensor.kapasitetsledd_faresone
    name: "Faresone"
```

### Kort 2: Topp-3 timer

```yaml
type: entities
title: Topp-3 Timer
entities:
  - entity: input_number.peak_1_verdi
    name: "Topp #1"
  - entity: input_text.peak_1_tidspunkt
    name: "Tidspunkt"
  - type: divider
  - entity: input_number.peak_2_verdi
    name: "Topp #2"
  - entity: input_text.peak_2_tidspunkt
    name: "Tidspunkt"
  - type: divider
  - entity: input_number.peak_3_verdi
    name: "Topp #3"
  - entity: input_text.peak_3_tidspunkt
    name: "Tidspunkt"
```

### Kort 3: Gauge (visuelt)

```yaml
type: gauge
entity: sensor.kapasitetsledd_aktuelt
name: Kapasitetsledd
min: 0
max: 10
needle: true
segments:
  - from: 0
    color: green
    to: 4.5
  - from: 4.5
    color: yellow
    to: 5.0
  - from: 5.0
    color: red
```

---

## 🎯 Hvordan det fungerer

### Scenario 1: Tidlig i måneden (dag 3)

```
Peak #1: 4.5 kWh
Peak #2: 4.2 kWh
Peak #3: 3.8 kWh
Kapasitetsledd: (4.5+4.2+3.8)/3 = 4.17 kW ✅

Trygg grense: 3.7 kWh (peak #3 - 0.1)
Prognose nå: 5.2 kWh

→ sensor.kapasitetsledd_hvis_prognose = (5.2+4.5+4.2)/3 = 4.63 kW
→ sensor.kapasitetsledd_kutt_anbefaling = "KUTT NÅ!" (4.63 < 5.0)
→ Automation KUTTER laster!
```

### Scenario 2: Sent i måneden (dag 28)

```
Peak #1: 5.5 kWh
Peak #2: 5.2 kWh
Peak #3: 4.8 kWh
Kapasitetsledd: (5.5+5.2+4.8)/3 = 5.17 kW ❌ (allerede over mål)

Trygg grense: 4.7 kWh (peak #3 - 0.1)
Prognose nå: 4.5 kWh

→ sensor.kapasitetsledd_hvis_prognose = 5.17 kW (uendret)
→ sensor.kapasitetsledd_kutt_anbefaling = "TRYGT"
→ Automation gjør INGENTING (sløser ikke med komfort!)
```

### Scenario 3: Kritisk situasjon

```
Peak #1: 5.2 kWh
Peak #2: 5.0 kWh
Peak #3: 4.8 kWh
Kapasitetsledd: 5.0 kW (akkurat på grensen!)

Prognose nå: 5.1 kWh

→ Ny topp #3 → Kapasitetsledd = 5.1 kW
→ "KUTT NÅ!"
→ Kutter VVB (høyest effekt)
→ Ny prognose: 3.2 kWh
→ "TRYGT" → Gjenoppretter VVB etter 5 min
```

---

## 🔧 Justering

### Endre målgrense:

**Innstillinger** → **Enheter og tjenester** → **Hjelpere** → `Kapasitetsledd - Målgrense`

- Sett til **4.0 kW** for aggressiv sparing
- Sett til **7.0 kW** for mer komfort

### Endre hysterese:

- `Kutt hysterese`: Hvor lenge over trygg grense før kutt (standard: 2 min)
- `Gjenopprett hysterese`: Hvor lenge under før gjenopprett (standard: 5 min)

---

## ❓ Feilsøking

### "Entity not found: sensor.kapasitetsledd_aktuelt"

**Problem:** Template sensorer ikke lastet.

**Løsning:**
1. Sjekk at template-koden er riktig indenert i `configuration.yaml`
2. Sjekk **Innstillinger → System → Logger** for feilmeldinger
3. Kjør **FULL RESTART** (ikke quick restart)

### Automation trigges ikke

**Løsning:**
1. Sjekk at `sensor.kapasitetsledd_kutt_anbefaling` faktisk sier "KUTT NÅ!"
2. Sjekk at minst én enhet er på
3. Sjekk logger: **Innstillinger → System → Logger**

### Kutter for ofte / for sjelden

**Løsning:**
1. Juster `input_number.kapasitetsledd_kutt_hysterese_min`
2. Øk til 5 min hvis det kutter for ofte
3. Senk til 1 min hvis det reagerer for sent

---

## ✅ Sjekkliste

- [ ] Backup av configuration.yaml
- [ ] Backup av automations.yaml
- [ ] Template sensorer lagt til i configuration.yaml
- [ ] 3 input_number hjelpere lagt til
- [ ] Configuration validert ✓
- [ ] System restartet
- [ ] Sensorer vises i Utviklerverktøy
- [ ] automations_kapasitetsledd.yaml kopiert til automations.yaml
- [ ] Configuration validert ✓ igjen
- [ ] System restartet igjen
- [ ] Automasjoner er aktivert
- [ ] Test 1 (sensorer) OK
- [ ] Test 2 (simulering) OK
- [ ] Dashboard opprettet (valgfritt)
- [ ] Ingen feilmeldinger i logger

---

**Versjon:** 3.0 - Kapasitetsledd-optimalisert
**Dato:** 2025-10-23
**Kompatibilitet:** ✅ Home Assistant 2025.6.3
**Formel:** ✅ Korrekt norsk kapasitetsledd
