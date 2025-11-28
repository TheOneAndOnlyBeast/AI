# KAPASITETSLEDD-SYSTEM: OPPDATERT TIL 2025 NETTLEIEMODELL (GLITRE NETT)

## 🎯 Oppsummering

Systemet er nå oppdatert for å følge den **offisielle norske nettleiemodellen 2025** basert på analysen av Glitre Nett, RME (Reguleringsmyndigheten for energi) og ny regulering.

### KRITISK ENDRING: Topp-3 DAGER (ikke timer)

**TIDLIGERE FEIL:**
- Systemet tracket topp-3 TIMER (når som helst i måneden)
- Kunne gi feil kapasitetsledd-beregning

**NY KORREKT IMPLEMENTERING:**
- Systemet tracker nå topp-3 DAGER (høyeste time per dag, fra 3 distinkte dager)
- Følger formelen: `K_grunnlag = (P_max,dag1 + P_max,dag2 + P_max,dag3) / 3`
- Sikrer at de tre dagene er DISTINKTE (ikke samme dag flere ganger)

---

## 📊 Nye Funksjoner

### 1. **Glitre Nett 2025 Trappetrinnsfunksjon**

Kapasitetsleddet prises nå i trappetrinn (ikke lineært):

| Trinn | Intervall | Pris/mnd |
|-------|-----------|----------|
| 1     | 0-2 kW    | 160 kr   |
| 2     | 2-5 kW    | 205 kr   |
| 3     | 5-10 kW   | 350 kr   |
| 4     | 10-15 kW  | 725 kr   |
| 5     | 15-20 kW  | 940 kr   |
| 6     | 20+ kW    | 1200 kr  |

**Viktige grenser:**
- **5.0 kW:** Fra 205 → 350 kr (+145 kr/mnd = +1740 kr/år)
- **10.0 kW:** Fra 350 → 725 kr (+375 kr/mnd = +4500 kr/år) 🔴 KRITISK!

### 2. **Nye Sensorer**

#### `sensor.kapasitetsledd_trinn`
Viser hvilket trinn du er i (f.eks. "Trinn 3 (5-10 kW)")

**Attributter:**
- `trinn_nummer`: Trinnnummer (1-6)
- `pris_naa`: Gjeldende pris i kr/mnd
- `neste_grense`: Grense til neste trinn
- `neste_pris`: Pris ved neste trinn

#### `sensor.kapasitetsledd_kostnad`
Viser kostnad per måned basert på gjeldende trinn (kr/mnd)

**Attributter:**
- `formel`: "Glitre Nett 2025 - Trappetrinn"
- `trinn`: Hvilket trinn du er i
- `potensiale_hvis_prognose`: Hva kostnaden blir hvis prognosen slår til

#### `sensor.kapasitetsledd_margin_til_trinn`
Viser hvor mange kW du har igjen til neste trinn (f.eks. 0.5 kW)

#### `sensor.dagens_maks_hittil`
Viser dagens høyeste timeforbruk hittil i dag

### 3. **Forbedret Kutt-Logikk**

`sensor.kapasitetsledd_kutt_anbefaling` er nå **trinngrense-bevisst**:

- **KUTT NÅ!** - Prognosen vil flytte deg til neste (dyrere) trinn
- **VARSLING** - Du nærmer deg en trinngrense (innen 0.5 kW)
- **TRYGT** - Du er trygt under grensen

**Nye attributter:**
- `grunn`: Forklaring av hvorfor (inkl. kostnadskonsekvens)
- `potensial_kostnadsøkning`: Hvor mye dyrere det blir (kr/mnd)

---

## 🔧 Tekniske Endringer

### Configuration.yaml

**Nye input_number:**
- `dagsmaks_idag`: Tracker dagens maks time (nullstilles kl 00:00)
- `peak_1_dag`, `peak_2_dag`, `peak_3_dag`: Dag i måneden for hver peak (sikrer distinkte dager)

**Nye sensorer:**
- `sensor.dagens_maks_hittil`: Dagens høyeste timeforbruk
- `sensor.kapasitetsledd_trinn`: Trappetrinn-indikator
- `sensor.kapasitetsledd_kostnad`: Månedlig kostnad (kr/mnd)
- `sensor.kapasitetsledd_margin_til_trinn`: Margin til neste trinn

**Oppdaterte sensorer:**
- `sensor.kapasitetsledd_aktuelt`: Lagt til `peak_X_dag` attributter
- `sensor.kapasitetsledd_trygg_grense`: Tar nå hensyn til trappetrinn-grenser
- `sensor.kapasitetsledd_kutt_anbefaling`: Aggressiv beskyttelse mot trinnhopp

### Automations_manual.yaml

**NYE AUTOMATIONS:**

1. **`dagsmaks_oppdater_hver_time`**
   - Kjører 30 sek etter hver time
   - Oppdaterer `dagsmaks_idag` hvis denne timen var høyere
   - Logger til system_log

2. **`effektledd_oppdater_topp3_dager`**
   - Kjører kl 23:59:00 (rett før midnatt)
   - Sammenligner dagens maks med topp-3 DAGER
   - Sikrer at de tre dagene er DISTINKTE
   - Håndterer kompleks sorteringslogikk:
     - Hvis dagen allerede finnes i topp-3: Oppdater kun den dagen
     - Hvis ny dag: Sammenlign og sorter inn i topp-3
   - Nullstiller `dagsmaks_idag` kl 00:00:30
   - Sender notifikasjon ved nye topper (inkl. kostnad)

**OPPDATERT:**
- `effektledd_nullstill_topper`: Nullstiller nå også `peak_X_dag` og `dagsmaks_idag`

**FJERNET:**
- `effektledd_oppdater_topper`: Erstattet med ny dagsmaks-logikk

### Dashboard.yaml

**Oppdatert:**
- Tittel: "Kapasitetsledd (Glitre Nett 2025)"
- Lagt til: Trinn, Kostnad, Margin til trinn, Dagens maks
- Topp-3 tabell: Tydeliggjort "Topp-3 DAGER (2025-modell)"
- Fargekoding basert på trinn (grønn/blå/oransje/rød)
- Viser både dagens maks og denne timens forbruk

---

## 📖 Hvordan Systemet Fungerer Nå

### Daglig Syklus

**Hver time (kl XX:00:30):**
1. Les P1-måler timeforbruk (sensor.p1_timeforbruk_korrekt)
2. Sammenlign med dagens maks hittil
3. Oppdater `dagsmaks_idag` hvis høyere

**Hver midnatt (kl 23:59:00):**
1. Les `dagsmaks_idag` (dagens høyeste time)
2. Sammenlign med topp-3 DAGER
3. Sjekk om dagen allerede finnes i topp-3:
   - Hvis JA: Oppdater kun den dagen hvis høyere
   - Hvis NEI: Sorter inn som ny dag i topp-3
4. Oppdater kapasitetsledd-beregning
5. Send notifikasjon hvis ny topp
6. Nullstill `dagsmaks_idag` kl 00:00:30

**Hver måned (1. dag kl 00:01:00):**
1. Nullstill alle peak-verdier
2. Nullstill dag-trackere
3. Start ny måned

### Smart Peak-Kontroll

**Hver 5. minutt + ved trigger:**
1. Sjekk prognose for inneværende time
2. Beregn kapasitetsledd hvis prognosen realiseres
3. Sjekk om dette vil:
   - Hoppe til neste trappetrinn → **KUTT NÅ!**
   - Overskride målgrense → **KUTT NÅ!**
   - Nærme seg trinngrense (< 0.5 kW) → **VARSLING**
4. Kutt laster i prioritert rekkefølge hvis nødvendig
5. Vis kostnadskonsekvens i notifikasjon

---

## 🎓 Eksempler

### Eksempel 1: Normal Måned

**Situasjon:**
- Dag 3: Høyeste time = 4.2 kWh
- Dag 7: Høyeste time = 4.5 kWh
- Dag 15: Høyeste time = 3.8 kWh

**Beregning:**
```
Kapasitetsledd = (4.5 + 4.2 + 3.8) / 3 = 4.17 kW
Trinn: 2 (2-5 kW)
Kostnad: 205 kr/mnd
```

### Eksempel 2: Kritisk Scenario

**Situasjon:**
- Topp-3: 4.8 kW, 4.6 kW, 4.5 kW
- Gjeldende kapasitetsledd: 4.63 kW (Trinn 2 = 205 kr/mnd)
- Prognose i dag: 5.2 kWh

**Analyse:**
```
Hvis prognose: (5.2 + 4.8 + 4.6) / 3 = 4.87 kW
Fortsatt Trinn 2 (under 5.0 kW)
Handling: VARSLING (nær grensen)
```

**MEN hvis prognose blir 5.5 kWh:**
```
Hvis prognose: (5.5 + 4.8 + 4.6) / 3 = 5.0 kW
NYE Trinn 3 (5-10 kW)
NYE Kostnad: 350 kr/mnd (+145 kr/mnd = +1740 kr/år!)
Handling: KUTT NÅ! 🚨
```

### Eksempel 3: Distinkte Dager

**FEIL (gammel måte):**
- Time 1 på dag 5: 6.0 kWh
- Time 2 på dag 5: 5.8 kWh
- Time 3 på dag 7: 5.5 kWh
- Beregning: (6.0 + 5.8 + 5.5) / 3 = 5.77 kW ❌

**RIKTIG (ny måte - 2025):**
- Dag 5 (høyeste time): 6.0 kWh
- Dag 7 (høyeste time): 5.5 kWh
- Dag 12 (høyeste time): 5.2 kWh
- Beregning: (6.0 + 5.5 + 5.2) / 3 = 5.57 kW ✅

---

## 🔍 Viktige Grenseverdier å Unngå

| Grense | Fra → Til | Økning | Årlig Økning |
|--------|-----------|--------|--------------|
| 2.0 kW | 160 → 205 kr | +45 kr/mnd | +540 kr/år |
| **5.0 kW** | **205 → 350 kr** | **+145 kr/mnd** | **+1740 kr/år** ⚠️ |
| **10.0 kW** | **350 → 725 kr** | **+375 kr/mnd** | **+4500 kr/år** 🔴 |
| 15.0 kW | 725 → 940 kr | +215 kr/mnd | +2580 kr/år |
| 20.0 kW | 940 → 1200 kr | +260 kr/mnd | +3120 kr/år |

**Tips:** Sett målgrense til 4.8 kW for å ha buffer til 5.0 kW-grensen!

---

## 📱 Dashboard-Indikatorer

**Fargekoding:**
- 🟢 Grønn: Trinn 1 (0-2 kW) - Svært lavt forbruk
- 🔵 Blå: Trinn 2 (2-5 kW) - Normalt husholdning
- 🟠 Oransje: Trinn 3 (5-10 kW) - Høyt forbruk, vær forsiktig!
- 🔴 Rød: Trinn 4+ (10+ kW) - Ekstremt dyrt!

---

## 🚀 Fremtidige Forbedringer

Basert på analysen kan følgende legges til senere:

1. **Dynamisk kapasitetsprising** (når Elhub støtter det)
   - Pris basert på nettbelastning, ikke kun kW
   - Tillater høy effekt på natt uten straff

2. **Sesongprising**
   - Vinter (Okt-Mar): Høyere satser
   - Sommer (Apr-Sept): Lavere satser

3. **Fleksibilitetstariffer**
   - Rabatt for å akseptere utkobling
   - 0 min varsel: 80% rabatt
   - 15 min varsel: 65% rabatt
   - 2 timer varsel: 50% rabatt

4. **Elavgift-tracking**
   - Spore faktiske avgifter (endres 2025-2026)
   - Enova-avgift fjernes 1. jan 2026

---

## ✅ Testing

Før du går live, sjekk:

1. **Konfigurasjonsvalidering:**
   ```bash
   Home Assistant > Utviklerverktøy > YAML > Sjekk konfigurasjon
   ```

2. **Manuell test av dagsmaks:**
   - Gå til Utviklerverktøy > Tilstander
   - Sett `input_number.dagsmaks_idag` til en testverdi
   - Vent til midnatt (eller trigger automation manuelt)
   - Sjekk at topp-3 oppdateres korrekt

3. **Test trappetrinn:**
   - Sett `input_number.peak_1_verdi` til 6.0
   - Sett `input_number.peak_2_verdi` til 5.0
   - Sett `input_number.peak_3_verdi` til 4.5
   - Beregnet: (6.0 + 5.0 + 4.5) / 3 = 5.17 kW
   - Forventet: Trinn 3, Kostnad 350 kr/mnd ✅

---

## 📚 Kilder

- **Glitre Nett prisliste 2025:** Trappetrinnsfunksjon 0-2, 2-5, 5-10, 10-15, 15-20, 20+
- **RME inntektsramme 2026:** 43.5 milliarder (opp 6% fra 2025)
- **Norsk nettleiemodell 2025:** Kapasitetsledd = Gjennomsnitt av topp-3 DAGER (distinkte)
- **Flaskehalsinntekter:** Fjernes fra 2027 → forventet prisøkning 20-30%

---

**Oppdatert:** 28. november 2025
**Modell:** Glitre Nett 2025 Trappetrinn
**Status:** Produksjonsklar ✅
