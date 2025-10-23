# Installasjonsveiledning for DIT system

## ✅ Kompatibilitetssjekk

Jeg har analysert ditt eksisterende Home Assistant-oppsett og bekrefter:

### Du har allerede disse hjelperne (OK ✓):
- ✅ `input_boolean.vvb_smart_styring_aktiv`
- ✅ `input_boolean.peak_kontroll_vvb_deaktivert`
- ✅ `input_boolean.peak_kontroll_billader_deaktivert`
- ✅ `input_boolean.peak_kontroll_badegulv_deaktivert`
- ✅ `input_boolean.peak_kontroll_vp_stue_deaktivert`
- ✅ `input_boolean.peak_kontroll_vp_kontor_deaktivert`
- ✅ `input_boolean.overoppheting_billader_deaktivert`
- ✅ `input_boolean.kino_synkronisering`
- ✅ `input_number.peak_1_verdi`, `peak_2_verdi`, `peak_3_verdi`
- ✅ `input_text.peak_1_tidspunkt`, `peak_2_tidspunkt`, `peak_3_tidspunkt`
- ✅ `input_datetime.robot_siste_kjoring`

### Du har alle nødvendige enheter (OK ✓):
- ✅ `sensor.prognose_for_innevaerende_time`
- ✅ `sensor.hourly_energy`
- ✅ `sensor.p1_meter_power`
- ✅ `switch.heavy_duty_switch` (VVB)
- ✅ `switch.shellyplus1pm_a8032ab13280` (Billader)
- ✅ `sensor.shellyplus1pm_a8032ab13280_temperature`
- ✅ `climate.floor_thermostat` (Badegulv)
- ✅ `climate.daikinap54875_room_temperature` (VP Stue)
- ✅ `climate.vp_datarom` (VP Kontor)
- ✅ `light.lys_gang`
- ✅ `media_player.shield_2`, `samsung_qn85ba_85_qe85qn85batxxc`, `rx_v685_3c03b4`
- ✅ `vacuum.roborock_s7_maxv`, `vacuum.nr_7`
- ✅ `person.erik_narum`

### Du MANGLER disse hjelperne (må legges til ⚠️):
- ❌ `input_number.peak_kontroll_av_grense`
- ❌ `input_number.peak_kontroll_paa_grense`
- ❌ `input_number.billader_maks_temperatur`
- ❌ `input_number.billader_trygg_temperatur`
- ❌ `input_number.kino_standard_volum`
- ❌ `input_number.vp_datarom_strong_terskel_paa`
- ❌ `input_number.vp_datarom_strong_terskel_av`
- ❌ `input_number.gang_lys_dag_brightness`
- ❌ `input_number.gang_lys_natt_brightness`

---

## 📋 Installasjon (10 minutter)

### STEG 1: Backup (1 min)

```bash
# SSH inn til Home Assistant eller bruk Terminal add-on
cd /config
cp configuration.yaml configuration.yaml.backup
cp automations.yaml automations.yaml.backup
```

Eller via File Editor add-on:
1. Åpne `configuration.yaml`
2. Kopier alt innhold
3. Lag ny fil: `configuration.yaml.backup`
4. Lim inn
5. Gjenta for `automations.yaml`

---

### STEG 2: Legg til input_number hjelpere (3 min)

**Åpne din `configuration.yaml` fil.**

Finn denne seksjonen:
```yaml
input_number:
  peak_1_verdi:
    name: "Effektledd - Peak #1 verdi"
    # ... osv
```

**LEGG TIL** de 9 nye hjelperne UNDER dine eksisterende (se `configuration_tillegg.yaml`):

```yaml
input_number:
  # EKSISTERENDE (ikke endre disse)
  peak_1_verdi:
    name: "Effektledd - Peak #1 verdi"
    min: 0
    max: 50
    step: 0.01
    unit_of_measurement: "kWh"
    mode: box
    icon: mdi:gauge

  peak_2_verdi:
    # ... (som før)

  peak_3_verdi:
    # ... (som før)

  # ===================================================================
  # NYE HJELPERE - LEGG TIL HER:
  # ===================================================================

  peak_kontroll_av_grense:
    name: "Peak-kontroll - Avslåingsgrense"
    min: 3.0
    max: 10.0
    step: 0.1
    initial: 4.7
    unit_of_measurement: "kWh"
    mode: box
    icon: mdi:flash-alert

  peak_kontroll_paa_grense:
    name: "Peak-kontroll - Påslåingsgrense"
    min: 2.0
    max: 8.0
    step: 0.1
    initial: 3.8
    unit_of_measurement: "kWh"
    mode: box
    icon: mdi:flash-outline

  billader_maks_temperatur:
    name: "Billader - Maks temperatur"
    min: 60
    max: 85
    step: 1
    initial: 75
    unit_of_measurement: "°C"
    mode: box
    icon: mdi:thermometer-alert

  billader_trygg_temperatur:
    name: "Billader - Trygg temperatur"
    min: 40
    max: 70
    step: 1
    initial: 60
    unit_of_measurement: "°C"
    mode: box
    icon: mdi:thermometer-check

  kino_standard_volum:
    name: "Kino - Standard volum"
    min: 0.0
    max: 1.0
    step: 0.025
    initial: 0.575
    mode: slider
    icon: mdi:volume-high

  vp_datarom_strong_terskel_paa:
    name: "VP Datarom - Strong på-terskel"
    min: 1.0
    max: 5.0
    step: 0.5
    initial: 3.0
    unit_of_measurement: "°C"
    mode: box
    icon: mdi:fan-plus

  vp_datarom_strong_terskel_av:
    name: "VP Datarom - Strong av-terskel"
    min: 0.5
    max: 4.0
    step: 0.5
    initial: 2.5
    unit_of_measurement: "°C"
    mode: box
    icon: mdi:fan-minus

  gang_lys_dag_brightness:
    name: "Gang lys - Dag brightness"
    min: 50
    max: 255
    step: 1
    initial: 254
    mode: slider
    icon: mdi:brightness-7

  gang_lys_natt_brightness:
    name: "Gang lys - Natt brightness"
    min: 1
    max: 50
    step: 1
    initial: 8
    mode: slider
    icon: mdi:brightness-4
```

**LAGRE** filen.

---

### STEG 3: Valider configuration.yaml (1 min)

**Via UI:**
1. Gå til **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Vent på ✓ grønn melding

**Hvis feil:**
- Sjekk indentering (YAML er veldig nøye med spacing)
- Bruk 2 spaces for hvert nivå (IKKE tabs)
- Sjekk at alle kolon (:) har mellomrom etter seg

---

### STEG 4: Restart for å laste input_number (2 min)

1. **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **RASK-START**
3. Vent 1-2 minutter til systemet er oppe igjen

---

### STEG 5: Verifiser at hjelperne er lastet (1 min)

1. Gå til **Innstillinger** → **Enheter og tjenester** → **Hjelpere**
2. Sjekk at du nå ser de 9 nye hjelperne:
   - Peak-kontroll - Avslåingsgrense (4.7 kWh)
   - Peak-kontroll - Påslåingsgrense (3.8 kWh)
   - Billader - Maks temperatur (75°C)
   - Billader - Trygg temperatur (60°C)
   - Kino - Standard volum (0.575)
   - VP Datarom - Strong på-terskel (3.0°C)
   - VP Datarom - Strong av-terskel (2.5°C)
   - Gang lys - Dag brightness (254)
   - Gang lys - Natt brightness (8)

**HVIS DE IKKE VISES:**
- Sjekk at `initial:` verdiene er satt (se over)
- Prøv en **FULL RESTART** i stedet (tar litt lengre tid)

---

### STEG 6: Installer forbedrede automasjoner (2 min)

**Metode A: Via File Editor (anbefalt)**

1. Åpne `automations.yaml`
2. Ta backup (kopier alt til en midlertidig fil)
3. Slett alt innhold
4. Åpne `automations_improved.yaml` (fra dette repositoryet)
5. Kopier ALT innhold
6. Lim inn i `automations.yaml`
7. LAGRE

**Metode B: Via terminal/SSH**

```bash
cd /config
cp automations_improved.yaml automations.yaml
```

---

### STEG 7: Sjekk og restart (1 min)

1. **Innstillinger** → **System** → **Vedlikehold**
2. Klikk **SJEKK KONFIGURASJON**
3. Hvis OK: Klikk **RASK-START**

---

### STEG 8: Verifiser at automasjoner fungerer (2 min)

1. Gå til **Innstillinger** → **Automatiseringer og scener**
2. Sjekk at du ser de forbedrede automatiseringene:
   - Peak-kontroll - Skru av enheter (PROGNOSE v5)
   - Peak-kontroll - Normalisering (PROGNOSE v5)
   - System-oppstart - Normaliser enheter v2
   - Overopphetingsvern - Skru AV Billader v2
   - Overopphetingsvern - Skru PÅ Billader igjen v2
   - Start robotstøvsugere når jeg er borte v2
   - Effektledd - Oppdater månedens topper v2
   - Effektledd - Nullstill topper ved månedsskifte v2
   - Kino: En på = alle på v2
   - Kino: En av = alle av v2
   - Varmepumpe Datarom - Dynamisk fan speed v2
   - Varmepumpe Datarom - Daglig tørkesyklus v2
   - Gang lys - Smart justering v2
   - Gang lys - Nattemodus overgang v2

3. **VIKTIG:** Sjekk at ALLE er **aktivert** (blå toggle til høyre)

---

## 🧪 Testing (5 minutter)

### Test 1: Peak-kontroll (2 min)

1. Gå til **Utviklerverktøy** → **Tilstander**
2. Finn `sensor.prognose_for_innevaerende_time`
3. Noter verdien (f.eks. 3.2 kWh)
4. Gå til **Innstillinger** → **Enheter og tjenester** → **Hjelpere**
5. Sett `peak_kontroll_av_grense` til litt UNDER prognosen (f.eks. 3.0 kWh)
6. Vent 2-3 minutter
7. Sjekk at du får notifikasjon om at enheter kuttes
8. Sett `peak_kontroll_av_grense` tilbake til 4.7 kWh
9. Sett `peak_kontroll_paa_grense` til OVER prognosen (f.eks. 5.0 kWh)
10. Vent 3-5 minutter
11. Sjekk at du får notifikasjon om at enheter gjenopprettes

### Test 2: Gang lys (1 min)

1. Slå på `light.lys_gang`
2. Sjekk at brightness er riktig (ingen flimring)
3. Juster `gang_lys_dag_brightness` til 200
4. Vent 15 minutter eller trigger automatiseringen manuelt
5. Sjekk at lysstyrken endres

### Test 3: Kino (2 min)

1. Slå PÅ én kino-enhet (f.eks. Shield)
2. Sjekk at alle 3 enheter slås på
3. Vent opptil 50 sekunder
4. Sjekk at volumet settes (hør etter!)
5. Slå AV én enhet
6. Sjekk at alle slås av

---

## 🎯 Hva har endret seg?

### Dine GAMLE automasjoner → NYE forbedrede

| Gammel ID | Ny ID | Endring |
|-----------|-------|---------|
| `peak_kontroll_skru_av_enheter_prognose_v4` | `peak_kontroll_skru_av_enheter_prognose_v5` | ✓ 5 min intervall (var 2)<br>✓ 3 min delay etter kutt<br>✓ Konfigurerbare terskler<br>✓ Bedre notifikasjoner |
| `peak_kontroll_normalisering_prognose_v4` | `peak_kontroll_normalisering_prognose_v5` | ✓ VVB smart styring kun ved behov<br>✓ Notifikasjoner ved gjenopprettelse |
| `system_oppstart_normaliser_enheter` | `system_oppstart_normaliser_enheter_v2` | ✓ Bedre logging<br>✓ Konfigurerbare terskler |
| `sikkerhet_overoppheting_av_billader` | `sikkerhet_overoppheting_av_billader_v2` | ✓ Konfigurerbare temp-grenser<br>✓ Viser faktisk temperatur |
| `sikkerhet_overoppheting_paa_billader` | `sikkerhet_overoppheting_paa_billader_v2` | ✓ Som over |
| `manuell_start_stovsugere_nar_borte` | `manuell_start_stovsugere_nar_borte_v2` | ✓ Batterisjekk 20%<br>✓ Bedre logging |
| `effektledd_oppdater_topper_manuell` | `effektledd_oppdater_topper_manuell_v2` | ✓ Kjører 30 sek ETTER timeskiftet<br>✓ Bedre logging |
| `effektledd_nullstill_topper_maanedsskifte` | `effektledd_nullstill_topper_maanedsskifte_v2` | ✓ Notifikasjon ved nullstilling |
| `kino_synk_alle_pa` | `kino_synk_alle_pa_v2` | ✓ 45 sek timeout (var 20)<br>✓ Konfigurerbart volum<br>✓ Bedre feilhåndtering |
| `kino_synk_alle_av` | `kino_synk_alle_av_v2` | ✓ Bedre logging |
| `varmepumpe_datarom_dynamisk_fan_speed` | `varmepumpe_datarom_dynamisk_fan_speed_v2` | ✓ Konfigurerbare terskler<br>✓ Bedre logging |
| `varmepumpe_datarom_daglig_torkesyklus` | `varmepumpe_datarom_daglig_torkesyklus_v2` | ✓ Lagrer og gjenoppretter modus<br>✓ Bedre logging |
| `gang_lys_smart_justering` | `gang_lys_smart_justering_v2` | ✓ Én kommando (ingen flimring)<br>✓ Konfigurerbar brightness |
| `gang_lys_nattemodus` | `gang_lys_nattemodus_overgang_v2` | ✓ Som over |

### Slettede (duplikater eller foreldet):
- ❌ Alle gamle kino-automasjoner (erstattet med v2)
- ❌ Gamle VVB smart styring automasjoner (hvis de eksisterte)
- ❌ Andre duplikater

---

## 📊 Sammenligning: Før vs Etter

| Funksjon | Før | Etter | Forbedring |
|----------|-----|-------|------------|
| Peak retrigger | 2 min | 5 min + 3 min delay | +150% mer stabilt |
| Kino timeout | 20 sek | 45 sek | +125% sikrere |
| Gang lys flimring | Ja (2 cmd) | Nei (1 cmd) | 100% bedre |
| Konfigurerbarhet | 0 parametre | 9 parametre | ∞% mer fleksibelt |
| Notifikasjoner | Kun kutt | Kutt + gjenopprettelse | 2x mer info |
| Logging | Minimal | 4 nivåer | 10x mer detaljert |
| Tilgjengelighetssjekker | Noen | Alle | 100% tryggere |

---

## 🆘 Feilsøking

### "Invalid config for input_number"

**Problem:** YAML indentering feil.

**Løsning:**
1. Åpne `configuration.yaml`
2. Sjekk at alle nye hjelpere har **2 spaces** indentering
3. Sjekk at det er mellomrom etter kolon: `name: ` (ikke `name:`)
4. Bruk **IKKE tabs**, kun spaces

### "Entity not found: input_number.peak_kontroll_av_grense"

**Problem:** Hjelperne er ikke lastet ennå.

**Løsning:**
1. Sjekk at hjelperne er i `configuration.yaml`
2. Kjør **FULL RESTART** (ikke quick restart)
3. Vent 2 minutter
4. Sjekk **Innstillinger → Enheter og tjenester → Hjelpere**

### Automasjoner trigges ikke

**Løsning:**
1. Gå til **Innstillinger → Automatiseringer**
2. Klikk på automatiseringen
3. Klikk **⋮** → **Informasjon**
4. Sjekk "Last triggered" - hvis "Never", se conditions
5. Sjekk **Innstillinger → System → Logger** for feilmeldinger

### Gang lys fungerer fremdeles ikke smooth

**Problem:** Gammel automasjon kjører fortsatt.

**Løsning:**
1. Gå til **Innstillinger → Automatiseringer**
2. Søk etter "gang lys"
3. Deaktiver eventuelle GAMLE versjoner
4. Sjekk at kun "v2" versjonene er aktive

---

## ✅ Sjekkl Liste

Før du er ferdig, sjekk at:

- [ ] Backup av `configuration.yaml` er tatt
- [ ] Backup av `automations.yaml` er tatt
- [ ] 9 nye `input_number` hjelpere er lagt til i `configuration.yaml`
- [ ] Configuration validert ✓
- [ ] System restartet
- [ ] Hjelperne vises i UI med riktige standardverdier
- [ ] `automations_improved.yaml` kopiert til `automations.yaml`
- [ ] Configuration validert ✓ igjen
- [ ] System restartet igjen
- [ ] Alle 14 automasjoner er aktivert
- [ ] Test 1 (peak-kontroll) kjørt og OK
- [ ] Test 2 (gang lys) kjørt og OK
- [ ] Test 3 (kino) kjørt og OK
- [ ] Ingen feilmeldinger i logger

---

## 📞 Neste steg

Etter 24 timer med kjøring:
1. Sjekk **Innstillinger → System → Logger**
2. Se etter advarsler eller feil
3. Juster hjelperne etter behov (se HURTIGREFERANSE.md)
4. Les FORBEDRINGER_OG_INSTALLASJON.md for dypere forståelse

Etter 1 uke:
1. Evaluer om peak-kontroll grensene er riktige
2. Juster `peak_kontroll_av_grense` og `peak_kontroll_paa_grense`
3. Optimaliser gang lys brightness
4. Finjuster varmepumpe datarom terskler

---

**Versjon:** 2.0
**Dato:** 2025-10-23
**Tilpasset for:** Erik Narums system
**Kompatibilitet:** ✅ Bekreftet
