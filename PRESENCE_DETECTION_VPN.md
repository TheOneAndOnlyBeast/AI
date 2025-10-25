# ===================================================================
# PRESENCE DETECTION MED ALWAYS-ON VPN
# ===================================================================

## ⚠️ KRITISK: VPN endrer alt!

Med always-on VPN (som deg) er **kun GPS-basert tracking pålitelig!**

---

## ❌ Hva som IKKE fungerer med VPN:

### 1. **Ping-tracking**
```yaml
device_tracker:
  - platform: ping
    hosts:
      erik: 192.168.1.100
```
**Problem:** VPN gjør at telefonen alltid ser ut som den er på hjemmenettverket, selv når du er ute.

### 2. **Router-basert tracking** (Unifi, ASUS, etc.)
**Problem:** VPN tunnelerer all trafikk gjennom hjemmerouteren, så router ser telefonen som "tilkoblet" hele tiden.

### 3. **IP-basert tracking**
**Problem:** VPN maskerer din faktiske IP og viser hjemme-IP.

### 4. **Wi-Fi SSID detection**
**Problem:** Upålitelig - fungerer bare hvis VPN slås av når ikke på hjemme-Wi-Fi (som du ikke gjør).

---

## ✅ Hva som FUNGERER med VPN:

### 1. **GPS/Geofencing** (Companion App) ⭐ BEST!

**Hvorfor det fungerer:**
- GPS er uavhengig av nettverkstilkobling
- Geofencing bruker telefonens lokasjon, ikke IP/nettverk
- Fungerer perfekt selv med VPN alltid på

**Setup:**
1. Installer **Home Assistant Companion App**
2. Gi tillatelse til **Lokasjon: "Alltid"**
3. Gi tillatelse til **Bakgrunnsoppdatering**
4. App vil automatisk oppdatere når du krysser geofence (standard 100m fra hjemmet)

### 2. **Person Entity** (kombinerer GPS-trackere)

**Hvorfor det fungerer:**
- Bruker GPS fra Companion App
- Kan kombinere flere GPS-sources
- Velger automatisk beste tilgjengelige tracker

**Setup:**
```yaml
# I Home Assistant:
# Innstillinger → Personer → Din person
# Legg til: device_tracker.iphone_erik (fra Companion App)
```

### 3. **Bluetooth Beacon** (avansert)

**Hvorfor det fungerer:**
- Bluetooth er uavhengig av nettverk
- Detekterer om telefonen er innenfor BLE-rekkevidde (~10m)
- Fungerer perfekt med VPN

**Eksempel setup:**
```yaml
device_tracker:
  - platform: bluetooth_le_tracker
    track_new_devices: false
    interval_seconds: 30
```

Krever ESP32 eller Raspberry Pi med Bluetooth.

---

## 🎯 ANBEFALT LØSNING FOR DEG:

### **Person Entity + Companion App GPS**

Dette er **ENESTE pålitelige løsning** med always-on VPN!

**STEG 1: Installer Companion App**

**iOS:**
1. App Store → "Home Assistant"
2. Logg inn på Home Assistant
3. **VIKTIG:** Innstillinger → Home Assistant → Lokasjon → **"Alltid"**
4. **VIKTIG:** Innstillinger → Generelt → Bakgrunnsoppdatering → **På**
5. **VIKTIG:** Innstillinger → Personvern → Stedstjenester → Home Assistant → **"Alltid" + "Presis lokasjon"**

**Android:**
1. Google Play → "Home Assistant"
2. Logg inn
3. **VIKTIG:** Tillatelser → Lokasjon → **"Tillat hele tiden"**
4. **VIKTIG:** Batteri → **Ikke optimaliser** (kritisk!)
5. **VIKTIG:** Autostart → **På**

**STEG 2: Konfigurer Person Entity**

1. **Innstillinger → Personer → Din person**
2. **Legg til tracker:** `device_tracker.erik_iphone` (fra Companion App)
3. **Fjern gamle upålitelige trackere** (de fungerer ikke med VPN uansett)
4. **LAGRE**

**STEG 3: Test**

1. Gå utenfor hjemmet (100m+)
2. Vent 1-2 minutter
3. Sjekk **Utviklerverktøy → Tilstander → person.erik_narum**
4. State skal endre til **"not_home"**

---

## 🔧 Feilsøking med VPN:

### "GPS oppdaterer ikke"

**iOS:**
```
Innstillinger → Personvern og sikkerhet → Stedstjenester → Home Assistant
→ Sjekk at "Alltid" er valgt
→ Sjekk at "Presis lokasjon" er på
```

**Android:**
```
Innstillinger → Apper → Home Assistant → Tillatelser → Lokasjon
→ Sjekk at "Tillat hele tiden" er valgt
→ Sjekk at "Bruk presis lokasjon" er på

Innstillinger → Batteri → Home Assistant
→ Sjekk at "Ikke optimaliser" er valgt (KRITISK!)
```

### "Companion App sier 'unknown' eller 'unavailable'"

**Sjekk i Companion App:**
1. Åpne appen → Innstillinger → Companion App
2. Gå til "Sensorer"
3. Finn "Lokasjon" sensorer
4. Sjekk at de er aktivert
5. Manuell oppdatering: Dra ned for å refreshe

### "Person entity viser fortsatt 'home' når jeg er ute"

**Debug:**
1. Utviklerverktøy → Tilstander → `device_tracker.erik_iphone`
2. Sjekk "latitude" og "longitude" - oppdateres de?
3. Sjekk "gps_accuracy" - skal være < 100m
4. Hvis GPS ikke oppdateres → Sjekk lokasjonsti llatelser (se over)

---

## 📱 BONUS: Companion App Settings (iOS)

Åpne Companion App → Innstillinger → Companion App:

**Location:**
- ✅ Accuracy: "High"
- ✅ Zone Enter/Exit: **På**
- ✅ Background Fetch: **På**
- ✅ Significant Location Change: **På**

**Notifications:**
- ✅ Tillat varslinger (for debugging)

---

## 🚀 ROBOTSTØVSUGERE MED VPN

Med VPN må robotstøvsuger-automatiseringen bruke **person entity** (GPS):

```yaml
trigger:
  - platform: state
    entity_id: person.erik_narum  # GPS-basert!
    from: home
    to: not_home
    for: "02:00:00"  # Borte i 2 timer = jobb
```

**Smart logikk (i stedet for 48-timers throttle):**
- ✅ Kun på **hverdager** (mandag-fredag)
- ✅ Kun mellom **09:00-16:00** (arbeidstid)
- ✅ Må være borte i **minst 2 timer** (jobb, ikke handle)
- ✅ Maks **én gang per dag**

Dette starter støvsugere KUN når du sannsynligvis er på jobb! 🤖

---

## 💡 ALTERNATIVER (hvis GPS ikke fungerer):

### **Bluetooth Beacon** (avansert)

Kjøp en **Tile** eller **ESP32**-basert beacon:
1. Fest beacon på nøkkelring
2. Home Assistant detekterer beacon via Bluetooth
3. Hvis beacon er hjemme = du er hjemme
4. Hvis beacon er borte = du er borte

**Fordeler:**
- ✅ Fungerer med VPN
- ✅ Veldig pålitelig
- ✅ Lavt strømforbruk

**Ulemper:**
- ❌ Krever ekstra hardware
- ❌ Må huske å ha beacon med seg

---

## 📋 OPPSUMMERING:

**Med always-on VPN:**

1. ✅ **Installer Companion App** (OBLIGATORISK!)
2. ✅ **Gi "Alltid" lokasjonsti llatelser**
3. ✅ **Bruk Person Entity** i automatiseringer
4. ✅ **Test at GPS oppdaterer** når du går ut
5. ✅ **Smart robotstøvsuger-logikk** (2 timer + hverdag + 09-16)

**99% pålitelighet med GPS-tracking!** 📱🎯

---

## ⚙️ Alternativ: Deaktiver VPN hjemme

Hvis du vil bruke Wi-Fi-basert tracking:

**iOS Shortcuts:**
1. Opprett automation: "Når kobler til [Ditt Wi-Fi]" → "Skru av VPN"
2. Opprett automation: "Når kobler fra [Ditt Wi-Fi]" → "Skru på VPN"

**Android Tasker:**
1. Profil: "Wifi Connected" → [Ditt Wi-Fi]
2. Task: "VPN Off"
3. Exit Task: "VPN On"

Men **GPS er fortsatt mest pålitelig!** 🎯
