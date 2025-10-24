# Home Assistant - Smart Kapasitetsledd-system

## 📦 Hva du får

**KOMPLETTE filer klare til copy-paste:**

1. **configuration.yaml** - Konfigurasjon med NYE kapasitetsledd-sensorer
2. **automations/manual.yaml** - Alle manuelle automasjoner (35 KB)
3. **automations/ui.yaml** - Tom fil for UI-automatiseringer (362 B)
4. **scripts.yaml** - Tom fil for scripts
5. **scenes.yaml** - Tom fil for scenes

**VIKTIG:** configuration.yaml inneholder KUN nye helpers og sensorer. Eksisterende helpers (peak_1/2/3, input_boolean, etc.) som allerede finnes i ditt system er IKKE duplikert - dette forhindrer konfigurasjonsfeil!

## ✅ Hva er nytt

### Smart kapasitetsledd-kontroll (RIKTIG FORMEL!)

**Før:**
- Kuttet alltid ved fast grense (4.7 kWh)
- Tok IKKE hensyn til eksisterende topp-3 timer
- Kuttet unødvendig selv når det ikke påvirket kapasitetsleddet

**Nå:**
```
Kapasitetsledd = (Peak #1 + Peak #2 + Peak #3) / 3

Eksempel:
Peak #1: 5.5 kWh, Peak #2: 5.2 kWh, Peak #3: 4.8 kWh
Kapasitetsledd: (5.5+5.2+4.8)/3 = 5.17 kW

Prognose nå: 4.5 kWh
→ Påvirker ikke topp-3 → IKKE KUTT (sløser ikke med komfort)

Prognose nå: 5.3 kWh
→ Ny topp #3 → Kapasitetsledd = 5.33 kW → KUTT NÅ!
```

### Nye sensorer du får:

1. `sensor.kapasitetsledd_aktuelt` - Nåværende gjennomsnitt (X.XX kW)
2. `sensor.kapasitetsledd_trygg_grense` - Hva kan jeg bruke nå? (X.X kWh)
3. `sensor.kapasitetsledd_hvis_prognose` - Hva blir det hvis prognose realiseres?
4. `sensor.kapasitetsledd_margin` - Margin til målgrense
5. `sensor.kapasitetsledd_kutt_anbefaling` - KUTT NÅ! / VARSLING / TRYGT
6. `binary_sensor.kapasitetsledd_faresone` - Er vi i faresonen?

### Nye automasjoner:

- ⚡ **Kapasitetsledd - Smart peak-kontroll** (erstatter gammel peak-kontroll)
- ✅ **Kapasitetsledd - Normalisering** (intelligent gjenopprettelse)
- 📊 **Effektledd - Oppdater topp-3** (med notifikasjoner)
- 🔄 **Effektledd - Nullstill** (ved månedsskifte)

### Alle dine eksisterende automasjoner er med:

- 🔥 Overopphetingsvern billader
- 🤖 Robotstøvsugere
- 📺 Kino-synkronisering
- 🌡️ Varmepumpe datarom (dynamisk fan + tørkesyklus)
- 💡 Gang lys (dag/natt)
- 🏠 System oppstart normalisering

---

## 🚀 Installasjon (5 minutter!)

### STEG 1: Backup (1 min)

```bash
cd /config
cp configuration.yaml configuration.yaml.backup
```

### STEG 2: Erstatt/oppdater filene (3 min)

**Via File Editor (anbefalt):**

**A) configuration.yaml:**
1. Åpne `configuration.yaml` i File Editor (DIN eksisterende fil)
2. Finn linjen `automation:` eller `automation: !include automations.yaml`
3. Endre til: `automation: !include_dir_merge_list automations/`
4. Legg til de NYE kapasitetsledd-sensorene fra `configuration.yaml` i dette repoet
   - Kun seksjonen under "NYE: KAPASITETSLEDD-SENSORER"
   - Ikke dupliker eksisterende helpers!
5. LAGRE

**B) Opprett automations-mappe:**
```bash
cd /config
mkdir -p automations
```

**C) Flytt eksisterende automatiseringer:**
1. Åpne `automations.yaml` (din eksisterende fil med UI-automatiseringer)
2. Kopier innholdet
3. Opprett ny fil: `automations/ui.yaml`
4. Lim inn innholdet
5. LAGRE

**D) Legg til manuelle automatiseringer:**
1. Opprett ny fil: `automations/manual.yaml`
2. KOPIER alt innhold fra `automations/manual.yaml` i dette repoet
3. LIM INN
4. LAGRE

**E) Slett gamle filer (valgfritt):**
```bash
# Kun hvis du har tatt backup!
rm /config/automations.yaml
```

### STEG 3: Sjekk og restart (2 min)

1. **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Vent på ✓ grønn melding
4. Klikk **RASK-START**
5. Vent 2 minutter

### STEG 4: Opprett NYE helpers (1 min)

De 3 nye helperne for kapasitetsledd må opprettes manuelt via UI:

**Innstillinger → Enheter og tjenester → Hjelpere → OPPRETT HJELPER**

1. **Kapasitetsledd - Målgrense**
   - Type: Number
   - Navn: `Kapasitetsledd - Målgrense`
   - ID: `kapasitetsledd_maal`
   - Min: 2.0, Max: 25.0, Step: 0.5
   - Initial: 5.0
   - Enhet: kW
   - Ikon: `mdi:target`

2. **Kapasitetsledd - Kutt hysterese (minutter)**
   - Type: Number
   - Navn: `Kapasitetsledd - Kutt hysterese (minutter)`
   - ID: `kapasitetsledd_kutt_hysterese_min`
   - Min: 1, Max: 10, Step: 1
   - Initial: 2
   - Enhet: min
   - Ikon: `mdi:timer-sand`

3. **Kapasitetsledd - Gjenopprett hysterese (minutter)**
   - Type: Number
   - Navn: `Kapasitetsledd - Gjenopprett hysterese (minutter)`
   - ID: `kapasitetsledd_gjenopprett_hysterese_min`
   - Min: 2, Max: 10, Step: 1
   - Initial: 5
   - Enhet: min
   - Ikon: `mdi:timer-check`

### STEG 5: Verifiser (1 min)

**Sjekk sensorer:**
- Gå til **Utviklerverktøy** → **Tilstander**
- Søk etter `kapasitetsledd`
- Du skal se 6 nye sensorer!

**Sjekk automasjoner:**
- Gå til **Innstillinger** → **Automatiseringer**
- Søk etter "Kapasitetsledd"
- Sjekk at de er aktivert (blå toggle)

---

## ⚙️ Konfigurering

### Juster målgrense:

**Innstillinger → Enheter og tjenester → Hjelpere**

- `Kapasitetsledd - Målgrense` → Sett til **5.0 kW** (standard)
- `Kapasitetsledd - Kutt hysterese` → **2 minutter** (hvor lenge over før kutt)
- `Kapasitetsledd - Gjenopprett hysterese` → **5 minutter** (hvor lenge under før gjenopprett)

### Endre målgrense:

- **4.0 kW** → Aggressiv sparing (leilighet)
- **5.0 kW** → Balansert (standard)
- **7.0 kW** → Mer komfort (stort hus)

---

## 📊 Dashboard (anbefalt)

Lag et nytt dashboard med disse kortene:

### Kort 1: Kapasitetsledd oversikt

```yaml
type: entities
title: Kapasitetsledd
entities:
  - entity: sensor.kapasitetsledd_aktuelt
    name: "Aktuelt"
  - entity: input_number.kapasitetsledd_maal
    name: "Målgrense"
  - entity: sensor.kapasitetsledd_margin
    name: "Margin"
  - entity: sensor.prognose_for_innevaerende_time
    name: "Prognose nå"
  - entity: sensor.kapasitetsledd_trygg_grense
    name: "Trygg grense"
  - entity: sensor.kapasitetsledd_kutt_anbefaling
    name: "Vurdering"
```

### Kort 2: Gauge (visuelt)

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

## ❓ Feilsøking

### "Entity not found: sensor.kapasitetsledd_aktuelt"

**Løsning:** Kjør **FULL RESTART** (ikke quick restart)

### Automasjoner trigges ikke

**Løsning:**
1. Sjekk at `sensor.kapasitetsledd_kutt_anbefaling` faktisk sier "KUTT NÅ!"
2. Sjekk **Innstillinger → System → Logger** for feilmeldinger

### Kutter for ofte

**Løsning:** Øk `Kapasitetsledd - Kutt hysterese` til 5 minutter

---

## ✅ Sjekkliste

- [ ] Backup av `configuration.yaml` tatt
- [ ] Backup av `automations.yaml` tatt
- [ ] `configuration.yaml` erstattet
- [ ] `automations.yaml` erstattet
- [ ] Configuration sjekket ✓
- [ ] System restartet
- [ ] Sensorer vises i Utviklerverktøy
- [ ] Automasjoner er aktivert
- [ ] Målgrense satt til 5.0 kW
- [ ] Dashboard opprettet (valgfritt)

---

## 🎯 Hva skjer nå?

Systemet vil:
1. Overvåke prognosen hver 5. minutt
2. Beregne om prognosen vil påvirke kapasitetsleddet
3. Kutte laster KUN hvis det faktisk reduserer kapasitetsleddet
4. Gjenopprette laster når det er trygt
5. Varsle deg om alle endringer

**Mål: Holde kapasitetsledd under 5 kW!**

---

**Versjon:** 3.0 - Kapasitetsledd-optimalisert
**Dato:** 2025-10-23
**Kompatibilitet:** ✅ Home Assistant 2025.6.3
**Formel:** ✅ Korrekt norsk kapasitetsledd (gjennomsnitt av topp-3)
**Status:** ✅ Produksjonsklar - klar til copy-paste!
