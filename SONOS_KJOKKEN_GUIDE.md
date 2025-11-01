# 🎵 Sonos Kjøkken - Automatisk Radio

## ✅ Hva jeg har laget:

### 3 Automations:
1. **Start radio** - Når du er på kjøkkenet (30 sek forsinkelse)
2. **Stopp radio** - Når du forlater (3 min grace period)
3. **Stopp ved TV** - Pauser automatisk når TV skrus på

### 1 Input Boolean:
- `input_boolean.sonos_kjokken_auto` - Skru av/på automatiseringen

---

## ⚠️ DU MÅ ENDRE DISSE:

I `automations/manual.yaml` må du bytte ut følgende placeholders:

### 1. Sonos Entity ID
```yaml
media_player.living_room  # BYTT TIL DIN SONOS!
```
**Finn din Sonos:**
1. Gå til **Innstillinger → Enheter & Tjenester → Integrasjoner**
2. Finn **Sonos**
3. Klikk på "X enheter"
4. Finn "Living Room" (eller hva den heter)
5. Kopier entity_id (f.eks. `media_player.living_room`)

### 2. Kjøkkensensor Entity ID
```yaml
binary_sensor.KJOKKEN_BEVEGELSE  # BYTT TIL DIN SENSOR!
```
**Finn din sensor:**
1. Gå til **Utviklerverktøy → Tilstander**
2. Søk etter "kjøkken" eller "kitchen" eller "motion"
3. Kopier entity_id (f.eks. `binary_sensor.motion_sensor_kitchen`)

**TIPS:** Hvis du IKKE har kjøkkensensor, kan du bruke:
- `person.erik_narum` + en zone "Kjøkken"
- En lysbryter som proxy (hvis du alltid skrur på lys på kjøkkenet)

### 3. Radio URL (valgfritt)
```yaml
media_content_id: "x-sonosapi-stream:s6712?sid=254&flags=8224&sn=0"  # NRK P1
```

**Populære radiokanaler for Sonos:**
- NRK P1: `x-sonosapi-stream:s6712?sid=254&flags=8224&sn=0`
- NRK P2: `x-sonosapi-stream:s24939?sid=254&flags=8224&sn=0`  
- NRK P3: `x-sonosapi-stream:s80247?sid=254&flags=8224&sn=0`
- Radio Norge: `x-rincon-mp3radio://http://stream.radionorge.no/rn_mp3_m`

**Eller finn din egen:**
1. Start radio manuelt i Sonos-appen
2. Gå til **Utviklerverktøy → Tilstander**
3. Finn `media_player.living_room`
4. Se på `media_content_id` attributtet
5. Kopier verdien

---

## 🎛️ Innstillinger du kan justere:

### Forsinkelser:
```yaml
for:
  seconds: 30  # Hvor lenge du må være på kjøkkenet før start
```
```yaml
for:
  minutes: 3  # Hvor lenge før den stopper når du går
```

### Volum:
```yaml
volume_level: 0.15  # Start-volum (15%)
volume_level: 0.30  # Mål-volum (30%)
volume_level: 0.10  # Fade-out volum
```

### Tider:
```yaml
after: "07:00:00"   # Start tid (morgen)
before: "22:00:00"  # Slutt tid (kveld)
```

---

## 💡 Forbedringer jeg har lagt til:

### ✅ Intelligent oppstart:
- **30 sek forsinkelse** - Unngår at den starter hvis du bare går forbi
- **Fade-in** - Starter på 15%, går opp til 30%
- **Sjekker om Sonos allerede spiller** - Avbryter ikke eksisterende musikk

### ✅ Smart av-logikk:
- **3 min grace period** - Stopper ikke med én gang hvis du går ut kort
- **Fade-out** - Reduserer til 10% før pause
- **Pause (ikke stopp)** - Kan fortsette der du slapp hvis du kommer tilbake

### ✅ Kontekst-bevissthet:
- **Kun dagtid** (07:00-22:00)
- **Ikke når TV er på**
- **Sjekker at du er hjemme**
- **Automatisk pause når TV skrus på**

### ✅ Manuell kontroll:
- **input_boolean.sonos_kjokken_auto** - Skru av/på i UI

---

## 🚀 Slik aktiverer du:

### Steg 1: Finn dine entity_id-er
(se over)

### Steg 2: Oppdater automations/manual.yaml
Søk etter `# BYTT UT` og erstatt med riktige entity_id-er

### Steg 3: Restart Home Assistant
**Innstillinger → System → Restart**

### Steg 4: Test!
1. Gå inn på kjøkkenet
2. Vent 30 sekunder
3. Radio skal starte på 30% volum
4. Gå ut
5. Vent 3 minutter
6. Radio skal pause

---

## 🎯 Fremtidige forbedringer:

Vil du ha noen av disse?
- **Forskjellige radiokanaler** basert på tid (P1 om morgenen, P3 om kvelden)
- **Høyere volum i helgene**
- **Sjekk om du har gjester** (ikke start hvis andre er hjemme)
- **Voice announcements** når du kommer inn ("God morgen!")
- **Snooze-funksjon** (deaktiver i 1 time)
- **Volum basert på tid** (lavere om morgenen)

Si fra hvis du vil ha noen av disse! 🎵
