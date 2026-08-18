# Home Assistant integration

Entities exposed by the device, dashboard cards, derived sensors and
automations.

---

## 1. Entities

Once the device is adopted, ESPHome exposes:

| Entity | Unit | Purpose |
|---|---|---|
| `sensor.zisterne_pegel_zisterne_fuellstand` | L | **Primary value** — usable stock |
| `sensor.zisterne_pegel_zisterne_fuellstand_prozent` | % | For gauges |
| `sensor.zisterne_pegel_zisterne_wassersaeule` | cm | Water column above probe |
| `sensor.zisterne_pegel_zisterne_nutzbare_hoehe` | cm | Column above pump intake |
| `sensor.zisterne_pegel_sonde_rohspannung` | V | Diagnostic — for calibration |
| `binary_sensor.zisterne_pegel_zisterne_sonde_stoerung` | — | Cable break or overrange |

Entity IDs depend on the device name in your ESPHome config. Adjust the
examples below accordingly.

---

## 2. Dashboard cards

### Gauge

```yaml
type: gauge
entity: sensor.zisterne_pegel_zisterne_fuellstand_prozent
name: Zisterne
min: 0
max: 100
severity:
  green: 50
  yellow: 25
  red: 0
needle: true
```

> Severity thresholds are worth setting from your own consumption, not from
> percentages. If a full irrigation cycle uses 670 L and the tank holds 3000 L,
> then 25 % is one cycle's worth — which is the number that actually matters.

### Level with history

```yaml
type: vertical-stack
cards:
  - type: entities
    title: Zisterne
    entities:
      - entity: sensor.zisterne_pegel_zisterne_fuellstand
        name: Vorrat
      - entity: sensor.zisterne_pegel_zisterne_wassersaeule
        name: Wassersäule
      - entity: binary_sensor.zisterne_pegel_zisterne_sonde_stoerung
        name: Sonde

  - type: history-graph
    hours_to_show: 72
    entities:
      - sensor.zisterne_pegel_zisterne_fuellstand
```

72 hours is a useful default: long enough to show a refill curve after an
irrigation cycle, short enough to read.

### Diagnostic card

Keep this somewhere out of the way — it is for commissioning and fault-finding,
not daily use.

```yaml
type: entities
title: Zisterne Sonde — Diagnose
entities:
  - entity: sensor.zisterne_pegel_sonde_rohspannung
    name: Rohspannung
  - entity: sensor.zisterne_pegel_zisterne_nutzbare_hoehe
    name: Höhe über Ansaugung
  - entity: sensor.zisterne_pegel_wifi_signal_db
    name: WLAN
  - entity: sensor.zisterne_pegel_uptime
    name: Laufzeit
```

The raw voltage is what you read during calibration. Having it on a card saves
climbing to the shed with a multimeter every time.

---

## 3. Derived sensors

### Refill rate

How fast the tank recovers. For an open-bottomed cistern this is the
groundwater inflow; for a sealed one it is roof runoff.

Create a **Derivative** helper:

| Field | Value |
|---|---|
| Source | `sensor.zisterne_pegel_zisterne_fuellstand` |
| Precision | 0 |
| Time window | 1 hour |
| Unit | L/h |

> Use a long time window. With a short one the derivative is dominated by
> sensor noise; the level itself changes over hours.

A **negative** value during an irrigation cycle is consumption. A **positive**
one afterwards is recovery — that is the number that tells you whether
tomorrow's cycle can run.

### Days of stock remaining

Requires a known daily consumption. If you have a flow meter on the outlet,
use its daily total; otherwise substitute your calculated irrigation demand.

```yaml
template:
  - sensor:
      - name: "Zisterne Reichweite"
        unit_of_measurement: "d"
        state: >
          {% set vorrat = states('sensor.zisterne_pegel_zisterne_fuellstand') | float(0) %}
          {% set bedarf = states('sensor.bewaesserung_bedarf_heute') | float(0) %}
          {% if bedarf > 0 %}
            {{ (vorrat / bedarf) | round(1) }}
          {% else %}
            unknown
          {% endif %}
```

This ignores refill, so it is a worst case. For an open-bottomed cistern that
is conservative by a wide margin.

---

## 4. Automations

### Low level warning

```yaml
alias: Zisterne — Vorrat niedrig
triggers:
  - trigger: numeric_state
    entity_id: sensor.zisterne_pegel_zisterne_fuellstand
    below: 500
    for: "00:15:00"
actions:
  - action: notify.mobile_app_your_phone
    data:
      title: Zisterne
      message: >
        Vorrat bei {{ states('sensor.zisterne_pegel_zisterne_fuellstand') }} L.
        Bewässerung reicht noch etwa
        {{ states('sensor.zisterne_reichweite') }} Tage.
mode: single
```

The `for: 15 minutes` matters — it prevents an alert while a pump is briefly
drawing the level down past the threshold.

### Probe fault

```yaml
alias: Zisterne — Sonde gestört
triggers:
  - trigger: state
    entity_id: binary_sensor.zisterne_pegel_zisterne_sonde_stoerung
    to: "on"
    for: "00:05:00"
actions:
  - action: notify.mobile_app_your_phone
    data:
      title: Zisterne Sonde
      message: >
        Signal außerhalb des gültigen Bereichs
        ({{ states('sensor.zisterne_pegel_sonde_rohspannung') }} V).
        Kabelbruch oder Sondendefekt.
mode: single
```

Under 0.3 V means less than 4 mA — a cable break. Over 2.3 V means more than
20 mA — overrange or a short. Either way the level reading is not trustworthy
until resolved.

### Dry-run protection

**This is the one that pays for the whole project.**

Without it, a pump that runs dry keeps running. In this installation it
cycled at its pressure switch for 65 minutes — 500 to 630 W with drops to 1 W
every few seconds — because no pressure could build. That is considerably worse
for a submersible pump than steady dry running.

```yaml
alias: Zisterne leer — Pumpe schützen
triggers:
  - trigger: numeric_state
    entity_id: sensor.zisterne_pegel_zisterne_fuellstand
    below: 50
actions:
  - action: switch.turn_off
    target:
      entity_id: switch.your_pump_socket
  - action: input_boolean.turn_on
    target:
      entity_id: input_boolean.zisterne_leer
  - action: notify.mobile_app_your_phone
    data:
      title: Zisterne leer
      message: Pumpe abgeschaltet. Auf Hauswasser umstellen.
mode: single
```

Two details that are easy to get wrong:

**Set the flag before switching off.** If an "always on" automation watches the
pump socket, it will switch it straight back on. The flag has to block that
automation, and it has to be set first.

**Do not clear the flag automatically** when the level recovers. An
open-bottomed cistern refills within hours, so the pump would restart into a
tank that has 200 litres in it. Clear it manually, or require the level to
exceed a much higher threshold.

If you have no level sensor yet, the same protection can be built from a flow
meter: pump drawing power but no flow for three minutes means either an empty
tank or a jammed meter — both require stopping.

---

## 5. Cross-check against a float ladder

If you are replacing float switches, keep them for a few weeks. The contacts
sit at known heights, so every transition is a free calibration point.

```yaml
type: history-graph
hours_to_show: 168
entities:
  - sensor.zisterne_pegel_zisterne_fuellstand
  - sensor.old_float_ladder_level
```

Plotted together, the probe should step through the ladder's levels at the same
moments. A consistent offset means the calibration zero is out; a consistent
slope difference means the span is.