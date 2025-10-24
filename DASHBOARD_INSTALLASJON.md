# 🎨 Dashboard Installasjon - Smart Kapasitetsledd

## 📋 Hva du får

Et komplett, moderne dashboard som kombinerer:

✅ **NYE Kapasitetsledd-sensorer** (gauge, status, topp-3)
✅ **Power flow** (visuell energi-flyt)
✅ **Enhetskontroll** (VVB, billader, gulvvarme, varmepumper)
✅ **Person tracking** (batteriindikator, tilkobling)
✅ **Lysstyring** (gang + soverom)
✅ **Detaljert statistikk** (forbruk, faser, spenning)

**Design:** Moderne, responsivt, fungerer perfekt på både PC og mobil!

---

## 🚀 Installasjon (2 minutter!)

### METODE 1: Via File Editor (enklest!)

1. **Åpne `dashboard_komplett.yaml` fra dette repoet**
2. **MARKER ALT** (Ctrl+A / Cmd+A)
3. **KOPIER** (Ctrl+C / Cmd+C)

4. **Gå til Home Assistant:**
   - Klikk på de 3 prikkene (øverst til høyre)
   - Velg **"Rediger dashbord"**
   - Klikk på de 3 prikkene igjen
   - Velg **"Raw configuration editor"**

5. **LAGE NYTT VIEW:**
   ```yaml
   # Finn "views:" seksjon i din dashboard-config
   # Legg til et nytt view:

   views:
     - title: Kapasitetsledd  # <-- NYTT VIEW
       path: kapasitetsledd
       icon: mdi:transmission-tower
       badges: []
       cards:
         # LIM INN ALT INNHOLD FRA dashboard_komplett.yaml HER
   ```

6. **LAGRE** → **FERDIG!**

---

### METODE 2: Helt nytt dashboard

Hvis du vil ha dette som et helt eget dashboard (anbefalt for testing):

1. **Gå til Innstillinger → Dashboards**
2. **Klikk "+ LEGG TIL DASHBOARD"**
3. **Navn:** `Smart Kapasitetsledd`
4. **Ikon:** `mdi:transmission-tower`
5. **Klikk "OPPRETT"**

6. **Åpne det nye dashboardet**
7. **Klikk de 3 prikkene → "Rediger dashbord"**
8. **Klikk de 3 prikkene → "Raw configuration editor"**

9. **SLETT alt innhold**
10. **LIM INN dette:**

```yaml
views:
  - title: Hjem
    path: home
    icon: mdi:home
    badges: []
    cards:
      # LIM INN ALT INNHOLD FRA dashboard_komplett.yaml HER
```

11. **LAGRE** → **FERDIG!**

---

## 📦 Avhengigheter (Custom Cards)

Dette dashboardet bruker noen custom cards som må installeres via HACS:

### 1. **custom:button-card**
```
HACS → Frontend → SØK "Button Card" → INSTALLER
```

### 2. **custom:bubble-card** (for lysstyring)
```
HACS → Frontend → SØK "Bubble Card" → INSTALLER
```

### 3. **custom:power-flow-card-plus** (for energy flow)
```
HACS → Frontend → SØK "Power Flow Card Plus" → INSTALLER
```

### 4. **custom:multiple-entity-row** (for kapasitetsledd-detaljer)
```
HACS → Frontend → SØK "Multiple Entity Row" → INSTALLER
```

**VIKTIG:** Etter installasjon av custom cards, må du:
1. **Tøm nettleser-cache** (Ctrl+Shift+R / Cmd+Shift+R)
2. **Restart Home Assistant** (Innstillinger → System → Restart)

---

## 🎨 Tilpassing

### Endre målgrense:

I dashboardet, klikk på **"Målgrense"** for å endre kapasitetsledd-målet (standard: 5 kW).

### Endre farger på gauge:

Finn denne seksjonen i YAML:
```yaml
- type: gauge
  entity: sensor.kapasitetsledd_aktuelt
  severity:
    green: 0      # ← Grønn fra 0 kW
    yellow: 4.5   # ← Gul fra 4.5 kW
    red: 5.0      # ← Rød fra 5.0 kW
```

### Endre enhetsrekkefølge:

I **"ENHETSKONTROLL"**-seksjonen kan du flytte horizontal-stack kortene for å endre hvilke enheter som vises først.

### Fjerne person tracking:

Fjern hele seksjonen med:
```yaml
# PERSON TRACKING OG LYS
```

---

## 🖼️ Dashboard-forhåndsvisning

### PC (Desktop):
- Gauges vises side-ved-side (2 kolonner)
- Enheter vises 2-3 per rad
- Power flow får full bredde
- Optimal lesbarhet på store skjermer

### Mobil:
- Gauges staples vertikalt
- Enheter vises 1-2 per rad
- Automatisk skalering
- Swipe for detaljer

---

## ❓ Feilsøking

### "Custom element doesn't exist: custom:button-card"

**Løsning:** Installer custom cards via HACS (se Avhengigheter over)

### "Entity not found: sensor.kapasitetsledd_aktuelt"

**Løsning:**
1. Sjekk at configuration.yaml har de nye template-sensorene
2. Kjør **Full Restart** i Home Assistant
3. Vent 2 minutter for sensorer å oppdatere

### Gauges viser "Unknown"

**Løsning:**
1. Sjekk at `sensor.hourly_energy` fungerer (Utviklerverktøy → Tilstander)
2. Sjekk at `input_number.peak_1/2/3_verdi` har verdier (ikke 0)
3. Vent til neste timeskifte for effektledd-oppdatering

### Person tracking viser feil bilde

**Løsning:**
1. Endre `entity_picture: /local/avatars/erik.png` til din faktiske bilde-path
2. Eller fjern linjen for å bruke default person-ikon

### Power flow viser ikke data

**Løsning:**
1. Sjekk at `sensor.p1_meter_power` har verdier
2. Sjekk at alle individual sensorer (billader, VVB, etc.) eksisterer
3. Fjern enheter fra power flow som ikke finnes i ditt system

---

## ✅ Sjekkliste

- [ ] Custom cards installert via HACS
- [ ] Nettleser-cache tømt
- [ ] Home Assistant restartet
- [ ] dashboard_komplett.yaml kopiert
- [ ] Nytt view eller dashboard opprettet
- [ ] YAML limt inn
- [ ] Dashboard lagret
- [ ] Alle sensorer vises riktig
- [ ] Enheter har riktig status
- [ ] Gauges viser data

---

## 🎯 Tips & Triks

### Legg til på mobil hjemskjerm:

1. **iOS:** Safari → Del → "Legg til på hjem-skjerm"
2. **Android:** Chrome → Meny → "Legg til på startskjerm"

### Bruk som standard dashboard:

Innstillinger → Dashboards → Velg dashboard → "SETT SOM STANDARD"

### Legg til flere personer:

Dupliser person tracking-kortet og endre:
```yaml
entity: device_tracker.DIN_ENHET
name: DITT_NAVN
entity_picture: /local/avatars/DITT_BILDE.png
```

### Legg til flere lys:

Dupliser bubble-card lys-kortet og endre:
```yaml
entity: light.DITT_LYS
name: DITT_LYSNAVN
icon: mdi:DITT_IKON
```

---

## 🔄 Oppdateringer

Hvis du senere vil oppdatere dashboardet:
1. Pull siste versjon fra git
2. Kopier ny `dashboard_komplett.yaml`
3. Lim inn i Raw configuration editor
4. Lagre

---

**Versjon:** 1.0
**Dato:** 2025-10-24
**Kompatibilitet:** ✅ Home Assistant 2025.6.3+
**Custom Cards:** button-card, bubble-card, power-flow-card-plus, multiple-entity-row
**Status:** ✅ Produksjonsklar!
