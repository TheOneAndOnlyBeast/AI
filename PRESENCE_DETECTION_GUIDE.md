# ===================================================================
# PÅLITELIG PRESENCE DETECTION - GUIDE
# ===================================================================

## STEG 1: Installer Home Assistant Companion App

### iOS:
1. App Store → Søk "Home Assistant"
2. Last ned "Home Assistant"
3. Åpne appen → Logg inn
4. Gi tillatelser:
   - Lokasjon: "Alltid"
   - Bevegelse & Fitness: Tillat
   - Bakgrunnsoppdatering: På
   - Varsler: Tillat

### Android:
1. Google Play → Søk "Home Assistant"
2. Last ned "Home Assistant"
3. Åpne appen → Logg inn
4. Gi tillatelser:
   - Lokasjon: "Tillat hele tiden"
   - Batteri: Ikke optimaliser (viktig!)
   - Autostart: Tillat

## STEG 2: Konfigurer Person Entity

1. **Innstillinger → Personer**
2. **Klikk på din person (Erik Narum)**
3. **Legg til trackere:**
   - device_tracker.iphone_17pro (eksisterende)
   - device_tracker.erik_iphone (ny fra Companion app)
4. **LAGRE**

Home Assistant vil automatisk bruke den mest pålitelige trackeren!

## STEG 3: Legg til Ping-tracking (backup)

Legg til i configuration.yaml:

```yaml
device_tracker:
  - platform: ping
    hosts:
      erik_iphone_wifi: 192.168.1.XXX  # <-- Endre til din iPhone IP
    consider_home: 180
    scan_interval: 60
```

Slik finner du IP:
1. iPhone → Innstillinger → Wi-Fi
2. Klikk på ditt nettverk
3. Se IP-adresse

## STEG 4: Oppdater automatiseringer

Endre ALLE automatiseringer fra device_tracker til person:

```yaml
# Før:
entity_id: device_tracker.iphone_17pro

# Nå:
entity_id: person.erik_narum
```

## STEG 5: Test!

1. **Sjekk at person entity fungerer:**
   - Utviklerverktøy → Tilstander
   - Søk: person.erik_narum
   - State skal være "home" når hjemme

2. **Test ved å gå ut:**
   - Gå utenfor geofence (vanligvis 100m fra hjemmet)
   - Vent 1-2 minutter
   - Sjekk at person.erik_narum endrer til "not_home"

3. **Test automatisering:**
   - Robotstøvsuger-automatiseringen skal nå trigge når du drar

## FORDELER MED DENNE LØSNINGEN:

✅ **Companion App:**
- Oppdateres øyeblikkelig ved ankomst/avreise
- Geofencing fungerer selv uten Wi-Fi
- Sender masse nyttig data (batteri, aktivitet, etc.)

✅ **Person Entity:**
- Kombinerer ALLE trackere
- Velger automatisk den beste/nyeste
- Fallback hvis én tracker feiler

✅ **Ping (backup):**
- Fungerer selv om app ikke sender oppdateringer
- Pålitelig når på Wi-Fi
- Enkel og stabil

## FEILSØKING:

### "Companion app viser ikke home/away"

**iOS:**
1. Innstillinger → Home Assistant → Lokasjon → "Alltid"
2. Innstillinger → Personvern → Stedstjenester → Home Assistant → "Alltid"
3. Innstillinger → Generelt → Bakgrunnsoppdatering → På for Home Assistant

**Android:**
1. Innstillinger → Apper → Home Assistant → Tillatelser → Lokasjon → "Tillat hele tiden"
2. Innstillinger → Batteri → Ikke optimaliser Home Assistant
3. Innstillinger → Apper → Home Assistant → Autostart → På

### "Person entity viser feil status"

Sjekk hvilken tracker som brukes:
1. Utviklerverktøy → Tilstander → person.erik_narum
2. Se "source" attributt
3. Sjekk at den trackeren faktisk fungerer

### "Ping tracker fungerer ikke"

1. Sjekk at iPhone er på Wi-Fi (ikke mobildata)
2. Sjekk at IP-adresse stemmer
3. Ping fra terminal: `ping 192.168.1.XXX`
4. Noen routere blokkerer ping - sjekk router-innstillinger

## BONUS: Zone-basert automatisering

Opprett soner for spesifikke steder:

```yaml
zone:
  - name: Jobb
    latitude: 59.9139
    longitude: 10.7522
    radius: 100
    icon: mdi:briefcase

  - name: Butikk
    latitude: 59.9100
    longitude: 10.7400
    radius: 50
    icon: mdi:cart
```

Trigger når du er på jobb:
```yaml
trigger:
  - platform: state
    entity_id: person.erik_narum
    to: Jobb
```

## ANBEFALT ENDRING TIL AUTOMATISERINGER:

Alle automatiseringer som bruker presence bør bruke person entity:

1. **Robotstøvsugere** ✅ (allerede endret)
2. **Lysstyring** (slå av lys når borte)
3. **Termostater** (senk temp når borte)
4. **Alarmsystemer** (aktiver alarm når borte)
5. **Varslinger** (send notis når noen kommer hjem)

---

**KONKLUSJON:**

person.erik_narum + Companion App = 99% pålitelig! 🎯
