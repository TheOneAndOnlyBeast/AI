# Hurtigreferanse - Home Assistant Automasjoner v2.0

## 🎚️ Juster innstillinger (via UI)

**Innstillinger → Enheter og tjenester → Hjelpere**

### Peak-kontroll blir for aggressiv?
```
peak_kontroll_av_grense: Øk fra 4.7 til 5.0 kWh
peak_kontroll_paa_grense: Øk fra 3.8 til 4.2 kWh
```

### Peak-kontroll kutter for sent?
```
peak_kontroll_av_grense: Senk fra 4.7 til 4.5 kWh
peak_kontroll_paa_grense: Senk fra 3.8 til 3.5 kWh
```

### Billader blir for varm?
```
billader_maks_temperatur: Senk fra 75 til 70°C
```

### Kino-volum er for høyt/lavt?
```
kino_standard_volum: Juster fra 0.575
(0.0 = 0%, 0.5 = 50%, 1.0 = 100%)
```

### Gang lys for sterkt på dagen?
```
gang_lys_dag_brightness: Senk fra 254 til 200
```

### Gang lys for svakt på natten?
```
gang_lys_natt_brightness: Øk fra 8 til 15
```

### Varmepumpe datarom kjøler for aggressivt?
```
vp_datarom_strong_terskel_paa: Øk fra 3.0 til 4.0°C
```

---

## 🔍 Sjekk status

### Er peak-kontroll aktiv?
**Utviklerverktøy → Tilstander**

Søk etter: `input_boolean.peak_kontroll_`

- `vvb_deaktivert: on` → VVB er kuttet
- `billader_deaktivert: on` → Billader er kuttet
- osv.

### Hva er nåværende prognose?
**Utviklerverktøy → Tilstander**

Søk etter: `sensor.prognose_for_innevaerende_time`

### Hva er billaderen sin temperatur?
**Utviklerverktøy → Tilstander**

Søk etter: `sensor.shellyplus1pm_a8032ab13280_temperature`

---

## 📋 Vanlige oppgaver

### Deaktivere peak-kontroll midlertidig
1. Gå til **Innstillinger → Automatiseringer**
2. Finn "Peak-kontroll - Skru av enheter (PROGNOSE v5)"
3. Klikk **⋮** → **Deaktiver**
4. (Husk å aktivere igjen senere!)

### Tvinge normalisering (skru på alt igjen)
1. Gå til **Utviklerverktøy → Tjenester**
2. Velg tjeneste: `automation.trigger`
3. Målrett: `automation.peak_kontroll_normalisering_prognose_v5`
4. Klikk **UTFØR TJENESTE**

### Se logger for en automasjon
1. Gå til **Innstillinger → Automatiseringer**
2. Klikk på automatiseringen
3. Klikk på **SPOR** (øverst til høyre)

### Se alle system-logger
**Innstillinger → System → Logger**

Filtrer på:
- "Peak-kontroll" → Se alle peak-relaterte hendelser
- "Kino synk" → Se kino-systemet
- "VP Datarom" → Se varmepumpe datarom
- "Gang lys" → Se lys-styring

---

## 🚨 Feilsøking

### Automasjon trigges ikke
1. Sjekk at den er aktivert: **Innstillinger → Automatiseringer**
2. Sjekk conditions: Gå inn på automatiseringen → **⋮** → **Informasjon**
3. Sjekk logger: **Innstillinger → System → Logger**

### "Entity not available"
- Sjekk at sensoren/enheten eksisterer: **Utviklerverktøy → Tilstander**
- Sjekk at den ikke er "unavailable" eller "unknown"

### Peak-kontroll kutter aldri
1. Sjekk at `sensor.prognose_for_innevaerende_time` eksisterer
2. Sjekk at verdien er over `input_number.peak_kontroll_av_grense`
3. Sjekk at automatiseringen er aktivert

### Peak-kontroll gjenoppretter aldri
1. Sjekk at prognosen er under `input_number.peak_kontroll_paa_grense`
2. Sjekk at input_boolean `peak_kontroll_*_deaktivert` er "on"
3. Vent 5 minutter (automatisk sjekk)

### Gang lys fungerer ikke
1. Sjekk at `light.lys_gang` eksisterer
2. Sjekk at automatiseringen er aktivert
3. Slå lyset av og på igjen

### Kino-synkronisering fungerer ikke
1. Sjekk at `input_boolean.kino_synkronisering` er "on"
2. Sjekk at alle media_player enheter eksisterer
3. Vent opptil 50 sekunder for volumjustering

---

## 📊 Anbefalte verdier per scenario

### Høy strømpris (aggressiv kutt-strategi)
```
peak_kontroll_av_grense: 4.0 kWh
peak_kontroll_paa_grense: 3.0 kWh
```

### Normal strømpris (balansert)
```
peak_kontroll_av_grense: 4.7 kWh  ← STANDARD
peak_kontroll_paa_grense: 3.8 kWh  ← STANDARD
```

### Lav strømpris (avslappet)
```
peak_kontroll_av_grense: 5.5 kWh
peak_kontroll_paa_grense: 4.5 kWh
```

### Sommertemperatur (mindre kjøling)
```
vp_datarom_strong_terskel_paa: 4.0°C
vp_datarom_strong_terskel_av: 3.0°C
```

### Vintertemperatur (mer kjøling)
```
vp_datarom_strong_terskel_paa: 2.5°C
vp_datarom_strong_terskel_av: 2.0°C
```

---

## 🔗 Nyttige lenker

### Home Assistant dokumentasjon
- [Automasjoner](https://www.home-assistant.io/docs/automation/)
- [Templates](https://www.home-assistant.io/docs/configuration/templating/)
- [Conditions](https://www.home-assistant.io/docs/scripts/conditions/)

### Debugging
- **Utviklerverktøy → Template**: Test templates
- **Utviklerverktøy → Tilstander**: Se alle entity tilstander
- **Utviklerverktøy → Tjenester**: Utfør tjenester manuelt

---

## 💡 Tips

### Eksperimenter trygt
1. Kopier eksisterende automasjon
2. Endre kopien
3. Test med midlertidige verdier
4. Slett kopien når ferdig

### Bruk varslinger
Alle viktige hendelser sender `persistent_notification` - du får varsel på dashboard.

### Logg viktige verdier
Bruk **Historikk** til å se hvordan verdier endrer seg over tid:
- `sensor.prognose_for_innevaerende_time`
- `sensor.hourly_energy`
- `sensor.shellyplus1pm_a8032ab13280_temperature`

### Lag dashbord for peak-kontroll
Lag et nytt dashbord med:
- `sensor.prognose_for_innevaerende_time` (gauge)
- `input_number.peak_kontroll_av_grense` (slider)
- `input_number.peak_kontroll_paa_grense` (slider)
- Alle `input_boolean.peak_kontroll_*` (toggle)

---

**Versjon:** 2.0
**Oppdatert:** 2025-10-23
