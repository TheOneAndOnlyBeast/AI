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

⚠️ **VIKTIG: Velg riktig metode basert på hva du vil gjøre!**

---

### 📋 METODE 1: Helt nytt dashboard (ANBEFALT!)

**Bruk denne metoden hvis du vil ha et helt nytt, separat dashboard.**

**Fil: `dashboard_komplett.yaml`**

1. **Gå til Innstillinger → Dashboards**
2. **Klikk "+ LEGG TIL DASHBOARD"**
3. **Navn:** `Smart Kapasitetsledd`
4. **Ikon:** `mdi:transmission-tower`
5. **Klikk "OPPRETT"**

6. **Åpne det nye dashboardet**
7. **Klikk de 3 prikkene (øverst høyre) → "Raw configuration editor"**
8. **SLETT alt innhold**
9. **Åpne `dashboard_komplett.yaml` og KOPIER ALT (Ctrl+A → Ctrl+C)**
10. **LIM INN i Home Assistant (Ctrl+V)**
11. **Klikk "LAGRE"** → **FERDIG!** 🎉

---

### 📋 METODE 2: Legg til i eksisterende dashboard

**Bruk denne metoden hvis du vil legge til som et nytt VIEW i ditt eksisterende dashboard.**

**Fil: `dashboard_view_only.yaml`**

1. **Åpne ditt eksisterende dashboard**
2. **Klikk de 3 prikkene (øverst høyre) → "Rediger dashbord"**
3. **Klikk "+ LEGG TIL VIEW" (nederst på siden)**
4. **Tittel:** `Kapasitetsledd`
5. **Ikon:** `mdi:transmission-tower`
6. **Klikk "LAGRE"**

7. **Klikk de 3 prikkene igjen → "Raw configuration editor"**
8. **Scroll HELT NED til du finner det NYE viewet (ser slik ut):**
   ```yaml
   views:
     - title: Home
       cards:
         # ... eksisterende kort
     - title: Kapasitetsledd  # <-- DITT NYE VIEW
       icon: mdi:transmission-tower
       cards: []  # <-- TOM!
   ```

9. **Åpne `dashboard_view_only.yaml` og KOPIER ALT innhold**
10. **Erstatt `cards: []` med det du kopierte**
    - Fjern `[]` og lim inn kortene
    - Pass på at indenteringen er riktig (samme som i eksempelet under)

11. **Eksempel på riktig struktur:**
    ```yaml
    - title: Kapasitetsledd
      icon: mdi:transmission-tower
      cards:
        - type: markdown  # <-- FØRSTE KORT
          content: |
            # ⚡ Smart Kapasitetsledd-system
            ...
        - type: horizontal-stack  # <-- ANDRE KORT
          cards:
            ...
    ```

12. **Klikk "LAGRE"** → **FERDIG!** 🎉

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
