# 📍 TRACKER DEBUG - TEMPLATES

## 🎯 Testing i Developer Tools

Gå til: **Developer Tools → Template** og lim inn følgende:

---

### Template 1: Enkel oversikt (alle trackere)

```jinja2
{% for entity in states.device_tracker %}
{{ entity.entity_id }}:
  Status: {{ entity.state }}
  Sist endret: {{ relative_time(entity.last_changed) }} siden
  Sist oppdatert: {{ relative_time(entity.last_updated) }} siden

{% endfor %}

{% for entity in states.person %}
{{ entity.entity_id }}:
  Status: {{ entity.state }}
  Sist endret: {{ relative_time(entity.last_changed) }} siden
  Sist oppdatert: {{ relative_time(entity.last_updated) }} siden

{% endfor %}
```

---

### Template 2: Hvilke trackere fungerer? (med vurdering)

```jinja2
🔍 TRACKER ANALYSE
==================

{% set ns = namespace(working=[], broken=[]) %}

{# Sjekk device_trackers #}
{% for entity in states.device_tracker %}
  {% set minutes_since_update = (now() - entity.last_updated).total_seconds() / 60 %}
  {% if minutes_since_update < 60 %}
    {% set ns.working = ns.working + [entity.entity_id] %}
  {% else %}
    {% set ns.broken = ns.broken + [entity.entity_id] %}
  {% endif %}
{% endfor %}

{# Sjekk person entities #}
{% for entity in states.person %}
  {% set minutes_since_update = (now() - entity.last_updated).total_seconds() / 60 %}
  {% if minutes_since_update < 60 %}
    {% set ns.working = ns.working + [entity.entity_id] %}
  {% else %}
    {% set ns.broken = ns.broken + [entity.entity_id] %}
  {% endif %}
{% endfor %}

✅ FUNGERENDE TRACKERE ({{ ns.working | length }}):
{% for tracker in ns.working %}
  - {{ tracker }} ({{ states(tracker) }})
    Oppdatert: {{ relative_time(states[tracker.split('.')[0]][tracker.split('.')[1]].last_updated) }} siden
{% endfor %}

❌ IKKE-FUNGERENDE TRACKERE ({{ ns.broken | length }}):
{% for tracker in ns.broken %}
  - {{ tracker }} ({{ states(tracker) }})
    Sist oppdatert: {{ relative_time(states[tracker.split('.')[0]][tracker.split('.')[1]].last_updated) }} siden
{% endfor %}

💡 ANBEFALING:
{% if ns.working | length == 0 %}
  ⚠️ INGEN trackere fungerer! Du må sette opp Companion App.
{% elif ns.working | length == 1 %}
  ✅ Bruk {{ ns.working[0] }} - denne er pålitelig!
{% else %}
  ✅ Du har {{ ns.working | length }} fungerende trackere
  🎯 Anbefalt: Bruk den som baserer seg på GPS (Companion App)
{% endif %}
```

---

### Template 3: Live monitoring (oppdateres hver sekund)

```jinja2
📊 LIVE TRACKER STATUS
{{ now().strftime('%H:%M:%S') }}
==================

{% for entity in states.device_tracker %}
{{ entity.name }}:
  🏷️  {{ entity.entity_id }}
  📍 {{ entity.state | upper }}
  ⏱️  Oppdatert for {{ ((now() - entity.last_updated).total_seconds() / 60) | round(0) }} min siden
  🔄 Endret for {{ ((now() - entity.last_changed).total_seconds() / 60) | round(0) }} min siden
  {% if (now() - entity.last_updated).total_seconds() / 60 < 60 %}✅ AKTIV{% else %}❌ INAKTIV{% endif %}

{% endfor %}

{% for entity in states.person %}
{{ entity.name }}:
  🏷️  {{ entity.entity_id }}
  📍 {{ entity.state | upper }}
  ⏱️  Oppdatert for {{ ((now() - entity.last_updated).total_seconds() / 60) | round(0) }} min siden
  🔄 Endret for {{ ((now() - entity.last_changed).total_seconds() / 60) | round(0) }} min siden
  {% if (now() - entity.last_updated).total_seconds() / 60 < 60 %}✅ AKTIV{% else %}❌ INAKTIV{% endif %}

{% endfor %}
```

---

### Template 4: Spesifikk tracker (erstatt ENTITY_ID)

```jinja2
{% set tracker = 'device_tracker.iphone_17pro' %}

📱 {{ states[tracker.split('.')[0]][tracker.split('.')[1]].name }}
==================

Entity ID: {{ tracker }}
Status: {{ states(tracker) | upper }}

⏱️  Tidslinje:
  - Last Updated: {{ states[tracker.split('.')[0]][tracker.split('.')[1]].last_updated.strftime('%Y-%m-%d %H:%M:%S') }}
    ({{ relative_time(states[tracker.split('.')[0]][tracker.split('.')[1]].last_updated) }} siden)

  - Last Changed: {{ states[tracker.split('.')[0]][tracker.split('.')[1]].last_changed.strftime('%Y-%m-%d %H:%M:%S') }}
    ({{ relative_time(states[tracker.split('.')[0]][tracker.split('.')[1]].last_changed) }} siden)

📊 Attributter:
{% for attr, value in state_attr(tracker, '') | dictsort %}
  - {{ attr }}: {{ value }}
{% endfor %}

💡 Vurdering:
{% set minutes_since = (now() - states[tracker.split('.')[0]][tracker.split('.')[1]].last_updated).total_seconds() / 60 %}
{% if minutes_since < 5 %}
  ✅ Veldig pålitelig (oppdatert for {{ minutes_since | round(0) }} min siden)
{% elif minutes_since < 60 %}
  ✅ Pålitelig (oppdatert for {{ minutes_since | round(0) }} min siden)
{% elif minutes_since < 1440 %}
  ⚠️  Kanskje upålitelig (oppdatert for {{ (minutes_since / 60) | round(1) }} timer siden)
{% else %}
  ❌ Ikke pålitelig (oppdatert for {{ (minutes_since / 1440) | round(1) }} dager siden)
{% endif %}
```

---

## 🔧 Hvordan bruke dette:

### Steg 1: Test i Developer Tools
1. Åpne Home Assistant
2. Gå til **Innstillinger → Developer Tools → Template**
3. Lim inn én av templates over
4. Se resultatene live!

### Steg 2: Legg til dashboard
1. Kopier innholdet fra `dashboard_tracker_debug.yaml`
2. Gå til ditt dashboard → **Edit Dashboard**
3. **Add Card → Manual** (nederst)
4. Lim inn innholdet
5. **Save**

### Steg 3: Bruk den nye sensoren
- **Entity ID**: `sensor.tracker_status_oversikt`
- **Attributter**: `trackere` (liste med alle trackere og status)
- Kan brukes i automations, conditions, etc.

---

## 📱 Test med VPN:

1. Gå ut av huset (med mobilen)
2. Vent 5 minutter
3. Åpne Developer Tools → Template
4. Kjør Template 2 ("Hvilke trackere fungerer?")
5. Se hvilke trackere som faktisk viser "not_home" vs "home"

**Forventet resultat med VPN:**
- ❌ Ping-baserte trackere: Vil vise "home" (fordi VPN-tunnel)
- ❌ Router-baserte trackere: Vil vise "home" eller "unavailable"
- ✅ GPS-baserte trackere (Companion App): Vil vise "not_home" ✅

---

## 🎯 Hva skal du se etter:

| Tracker Type | Status når borte | Fungerer med VPN? |
|--------------|------------------|-------------------|
| Ping (nmap, etc) | home ❌ | Nei |
| Router (UniFi, etc) | home/unavailable ❌ | Nei |
| IP-basert | home ❌ | Nei |
| Wi-Fi SSID | upålitelig ⚠️ | Nei |
| GPS (Companion App) | not_home ✅ | **JA!** |
| Bluetooth beacon | varierer | Ja (hvis hjemme) |

**Konklusjon**: Med always-on VPN MÅ du bruke GPS (Companion App)! 🎯
