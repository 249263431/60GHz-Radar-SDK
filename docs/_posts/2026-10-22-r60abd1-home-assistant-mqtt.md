---
layout: post
title: "R60ABD1 Sleep & Breathing Radar in Home Assistant via MQTT (ESP32 Gateway)"
date: 2026-10-22 09:00:00 +0800
categories: [integration]
tags: [home-assistant, mqtt, r60abd1, sleep radar, breathing, esp32]
---

The [gateway guide]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %}) wires the R60ABD1 60 GHz sleep/breathing radar to an ESP32-S3 and publishes its frames to MQTT. This article is the **consume** half: pull that MQTT feed into Home Assistant, expose breathing rate and sleep state as entities, put them on a dashboard, and fire two sensible automations. Nothing here is tied to a vendor cloud — your broker, your HA.

> The R60ABD1 is **not a medical device**. Treat the values as indicators, not a diagnosis.

## What you need

- The ESP32 gateway from the gateway guide, already publishing MQTT (default topic `radar/R60ABD1/state`).
- A broker Home Assistant can reach (Mosquitto, EMQX, or any MQTT 3.1/5 broker). The gateway guide points the ESP32 at your broker.
- Home Assistant with the **MQTT** integration installed.

## Enable the MQTT integration

If Home Assistant is not connected to your broker yet, follow the [MQTT setup walkthrough]({{ site.baseurl }}{% post_url 2026-10-18-home-assistant-radar-integration %}) first (add the MQTT integration, point it at Mosquitto/EMQX, set credentials). This article assumes the MQTT integration is already connected — below we only declare the R60ABD1 sensors.

## The payload

**Baseline** — exactly what the gateway guide publishes:

```json
{"breath":22,"sleep":1}
```

- `breath` — breathing rate from control `0x81` / command `0x02`; 1 byte, **0–35 breaths/min**, refreshed about every 3 s (R60ABD1 manual §7).
- `sleep` — sleep state from control `0x84` / command `0x02`: **0 = deep, 1 = light, 2 = awake, 3 = off-bed**, updated about every 10 min.

**Extended (optional)** — extend the gateway's publish to emit these extra keys (every key maps to a R60ABD1 §7 frame; all are safe to add):

```json
{"breath":22,"breath_status":1,"occupancy":1,"sleep":1,"sleep_quality":86,"heart":68}
```

- `breath_status` — `0x81`/`0x01`: `1` normal, `2` high (>25/min), `3` low (<10/min), `4` none.
- `occupancy` — presence (`0x80`/`0x01`, or the comprehensive frame byte 0; `1`/`0`).
- `sleep_quality` — `0x84`/`0x06`, 1 byte **0–100**.
- `heart` — average heart rate from the comprehensive frame (`0x84`/`0x0C`, byte 3).

## Entities in `configuration.yaml`

Baseline entities (breathing rate + raw sleep state):

{% raw %}
```yaml
mqtt:
  sensor:
    - name: "R60ABD1 Breathing Rate"
      unique_id: r60abd1_breathing_rate
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.breath }}"
      unit_of_measurement: "breaths/min"
      expire_after: 30
    - name: "R60ABD1 Sleep State"
      unique_id: r60abd1_sleep_state
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.sleep }}"
      expire_after: 1200
```
{% endraw %}

Map the numeric sleep state to text with a template sensor:

{% raw %}
```yaml
template:
  - sensor:
      - name: "R60ABD1 Sleep State Text"
        unique_id: r60abd1_sleep_state_text
        state_topic: "radar/R60ABD1/state"
        value_template: >-
          {% set m = {0:'Deep',1:'Light',2:'Awake',3:'Off-bed'} %}
          {{ m[value_json.sleep | int] if value_json.sleep is defined else 'unknown' }}
```
{% endraw %}

If you publish the **extended** payload, add:

{% raw %}
```yaml
mqtt:
  binary_sensor:
    - name: "R60ABD1 Occupancy"
      unique_id: r60abd1_occupancy
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.occupancy }}"
      payload_on: "1"
      payload_off: "0"
      device_class: occupancy
      expire_after: 1200
  sensor:
    - name: "R60ABD1 Breathing Status"
      unique_id: r60abd1_breathing_status
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.breath_status }}"
      expire_after: 30
    - name: "R60ABD1 Sleep Quality"
      unique_id: r60abd1_sleep_quality
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.sleep_quality }}"
      unit_of_measurement: "%"
      expire_after: 1200
    - name: "R60ABD1 Heart Rate"
      unique_id: r60abd1_heart_rate
      state_topic: "radar/R60ABD1/state"
      value_template: "{{ value_json.heart }}"
      unit_of_measurement: "bpm"
      expire_after: 1200
```
{% endraw %}

`expire_after` marks the entity **unavailable** if the broker stops publishing, so a dropped gateway doesn't leave a stale number on the dashboard.

## Dashboard (Lovelace)

A small **Entities** card is enough to start:

```yaml
type: entities
title: R60ABD1 Sleep & Breathing
entities:
  - entity: sensor.r60abd1_breathing_rate
  - entity: sensor.r60abd1_sleep_state_text
  - entity: binary_sensor.r60abd1_occupancy   # extended payload
  - entity: sensor.r60abd1_sleep_quality       # extended payload
  - entity: sensor.r60abd1_heart_rate          # extended payload
```

Or a **Glance** card for at-a-glance tiles. The `entity_id`s above assume the `unique_id` values set in the YAML; if you let HA auto-generate IDs, check **Developer → States** for the actual `entity_id`.

## Automations

Two practical triggers. Both use the baseline payload (no extended keys required).

**Breathing rate too high** (matches the manual's "high" threshold of >25/min):

```yaml
automation:
  - alias: "R60ABD1 breathing rate high"
    trigger:
      - platform: numeric_state
        entity_id: sensor.r60abd1_breathing_rate
        above: 25
        for: "00:00:30"
    action:
      - service: notify.notify
        data:
          title: "R60ABD1 breathing alert"
          message: "Breathing rate above 25/min."
```

**Off-bed / away** (sleep state 3):

```yaml
automation:
  - alias: "R60ABD1 off-bed"
    trigger:
      - platform: state
        entity_id: sensor.r60abd1_sleep_state
        to: "3"
    action:
      - service: notify.notify
        data:
          title: "R60ABD1 status"
          message: "Person is off-bed / away."
```

## Notes and safety

- **Not a medical device.** Do not use for life-safety or diagnosis without redundant design.
- Refresh cadence follows the module: breathing ~every 3 s, sleep state ~every 10 min. `expire_after` keeps the UI honest if the link drops.
- Keep `GND` common between the R60ABD1 and the ESP32 (see the gateway guide); a floating ground is the usual cause of garbage frames.
- This is the **consume** half of the two-architecture story in [MQTT vs UART]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %}): the module speaks only UART, the ESP32 speaks MQTT, HA listens.

## Where to go next

- Build the gateway: [R60ABD1 + ESP32-S3 SIP-S/MQTT gateway]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %})
- First-good-frame checklist: [Bring-Up checklist]({{ site.baseurl }}{% post_url 2026-10-22-radar-module-bring-up-checklist %})
- Platform-agnostic parsing: [Minimal UART parser (SIP-S V1.0)]({{ site.baseurl }}{% post_url 2026-10-22-minimal-uart-parser-sip-s %})
- Module specs: [60 GHz / 24 GHz module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- GPIO caution: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %})

Both the R60ABD1 module and the ESP32-S3 board are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
