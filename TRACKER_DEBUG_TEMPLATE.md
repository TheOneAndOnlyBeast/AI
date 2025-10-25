# 🔍 TRACKER DEBUG - Finn aktive/inaktive trackere

Bruk denne template i **Developer Tools → Template** for å se hvilke trackere som faktisk fungerer akkurat nå.

---

## 🎯 Enkel oversikt (kopier denne!)

```jinja2
🔍 TRACKER ANALYSE - {{ now().strftime('%H:%M:%S') }}
==================

{% set ns = namespace(working=[], broken=[]) %}

{# Sjekk alle device_trackers #}
{% for entity in states.device_tracker %}
  {% set minutes = (now() - entity.last_updated).total_seconds() / 60 %}
  {% if minutes < 60 %}
    {% set ns.working = ns.working + [entity] %}
  {% else %}
    {% set ns.broken = ns.broken + [entity] %}
  {% endif %}
{% endfor %}

{# Sjekk alle person entities #}
{% for entity in states.person %}
  {% set minutes = (now() - entity.last_updated).total_seconds() / 60 %}
  {% if minutes < 60 %}
    {% set ns.working = ns.working + [entity] %}
  {% else %}
    {% set ns.broken = ns.broken + [entity] %}
  {% endif %}
{% endfor %}

✅ AKTIVE TRACKERE ({{ ns.working | length }}):
{% if ns.working | length == 0 %}
  (ingen)
{% else %}
{% for entity in ns.working %}
  📍 {{ entity.entity_id }}
     Navn: {{ entity.name }}
     Status: {{ entity.state | upper }}
     Oppdatert: {{ relative_time(entity.last_updated) }} siden
     Endret: {{ relative_time(entity.last_changed) }} siden

{% endfor %}
{% endif %}

❌ INAKTIVE TRACKERE ({{ ns.broken | length }}):
{% if ns.broken | length == 0 %}
  (ingen)
{% else %}
{% for entity in ns.broken %}
  📍 {{ entity.entity_id }}
     Navn: {{ entity.name }}
     Status: {{ entity.state | upper }}
     Oppdatert: {{ relative_time(entity.last_updated) }} siden
     Endret: {{ relative_time(entity.last_changed) }} siden

{% endfor %}
{% endif %}

💡 ANBEFALING:
{% if ns.working | length == 0 %}
  ⚠️ INGEN trackere er aktive! Sett opp Companion App med GPS.
{% elif ns.working | length == 1 %}
  ✅ Bruk {{ ns.working[0].entity_id }} - denne er aktiv!
{% else %}
  ✅ Du har {{ ns.working | length }} aktive trackere.
  🎯 Med VPN: Bruk GPS-basert tracker (Companion App)!
{% endif %}

---

FORKLARING:
• AKTIV = Oppdatert siste timen
• INAKTIV = Ikke oppdatert på > 1 time
• "Oppdatert" = Sist Home Assistant mottok data
• "Endret" = Sist status endret seg (home ↔ not_home)

⚠️ MED VPN:
Ping/router-baserte trackere kan vise "aktiv" men være stuck på "home"!
Test ved å gå ut av huset og sjekk om status endrer seg til "not_home".
```

---

## 🚀 Hvordan bruke:

1. **Åpne Home Assistant**
2. Gå til: **Innstillinger → Developer Tools → Template**
3. **Kopier hele template** over (fra ``` til ```)
4. **Lim inn** i template editoren
5. Se resultatet med én gang!

---

## 🧪 Testing med VPN:

**For å finne ut hvilke trackere som FAKTISK fungerer med VPN:**

1. **Kjør template nå** (mens hjemme) - noter alle "AKTIVE"
2. **Gå ut av huset** med mobilen
3. **Vent 5 minutter**
4. **Kjør template igjen** (via mobil/annen enhet)
5. **Sjekk hvilke som endret status** til "not_home"

**Forventet med VPN:**
- ❌ Ping-trackere: Status "home" (selv når borte) = VIRKER IKKE
- ❌ Router-trackere: Status "home"/"unavailable" = VIRKER IKKE
- ✅ GPS-trackere (Companion App): Status "not_home" = VIRKER! 🎯

---

## 📊 Eksempel output:

```
🔍 TRACKER ANALYSE - 14:32:15
==================

✅ AKTIVE TRACKERE (2):
  📍 person.erik_narum
     Navn: Erik Narum
     Status: HOME
     Oppdatert: 2 minutter siden
     Endret: 3 timer siden

  📍 device_tracker.iphone_erik
     Navn: iPhone (Erik)
     Status: HOME
     Oppdatert: 1 minutt siden
     Endret: 3 timer siden

❌ INAKTIVE TRACKERE (1):
  📍 device_tracker.ping_iphone
     Navn: iPhone Ping
     Status: HOME
     Oppdatert: 2 dager siden
     Endret: 5 dager siden

💡 ANBEFALING:
  ✅ Du har 2 aktive trackere.
  🎯 Med VPN: Bruk GPS-basert tracker (Companion App)!
```

---

## 🎯 Hva gjør jeg med resultatet?

### Hvis du har GPS-basert tracker (Companion App):
✅ **Bruk `person.erik_narum`** i automations
✅ Person entity vil automatisk bruke beste tracker (GPS)

### Hvis ingen aktive trackere:
❌ **Installer Companion App** på mobilen
❌ **Aktiver GPS** med "Alltid"-tillatelse
❌ **Legg til tracker** i Person entity

### Hvis trackere viser "aktiv" men er stuck på "home":
⚠️ **Det er VPN-problem!**
⚠️ **Test ved å gå ut** og se om status endrer seg
⚠️ **Erstatt med GPS-tracker** (eneste som fungerer med VPN)

---

**Dette er kun for debugging - ingen permanent sensor/dashboard! 🔍**
