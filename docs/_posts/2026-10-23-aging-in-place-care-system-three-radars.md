---
layout: post
title: "Building an Aging-in-Place Care System with Three 60GHz Radars: Fall, Sleep, Presence"
date: 2026-10-23 08:00:00 +0800
categories: [guide]
tags: [aging-in-place, elderly care, fall detection, sleep radar, presence radar, home-assistant, mqtt]
---

Each of our three product guides describes one radar on its own: the [fall radar]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}), the [sleep radar]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}), and the [presence radar]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}). What none of them covers is the view a system designer actually needs: **how three radars work together in one home**, on one broker, feeding one alerting path.

This article is that reference architecture for aging-in-place, home care, and assisted-living deployments.

> **This is a safety and wellness system, not a medical one.** The radars do not diagnose, treat, or prevent a condition, and they are not a substitute for human care, a call system, or emergency services. Nothing here is a medical claim.

## Why three sensors in one dwelling

A single radar answers one question well. A caring adult asks several questions in different rooms, at different hours:

| Space | Radar | Question it answers | Key signals |
|---|---|---|---|
| Bathroom / shower | Fall radar | Did someone go down, and did they get up? | `fallStatus`, `fallPosition`, `fallDuration`, dwell |
| Bedroom, bed | Sleep radar | Are they in bed, breathing, actually resting? | in/out of bed, breathing rate, heart rate, sleep stage, anomaly flags |
| Living room, hallway | Presence radar | Is anyone here, and are they moving or still? | occupancy, motion, body-movement level, approach/leave |

Two themes repeat across them. The **bathroom** is where cameras are unacceptable and a call button is often out of reach. The **bedroom and hallway** answer the slower questions — did they get up at 3 a.m., did they reach the kitchen, have they been unnaturally still for an hour?

## Reference architecture

```
 Fall radar ─┐
 Sleep radar ┼──MQTT──>  Broker  ──MQTT──>  Home Assistant / your backend ──> caregiver alert
 Presence ───┘          (1883)                    (automation + escalation)
```

Keep three properties fixed from day one:

1. **One broker, one set of credentials per device.** See the [Artemis setup guide]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %}) for a self-hosted option; Mosquitto and managed brokers work identically from the radar's point of view.
2. **A unique ClientID per radar.** Two devices sharing one ID will boot each other off indefinitely.
3. **Topic namespaces per model** — `/Radar60FL/{DeviceID}/…`, `/Radar60SP/{DeviceID}/…`, `/RadarHP/{DeviceID}/…`. Give each device an ID that encodes the room (`fl-bathroom-01`), because that ID is what you will group on later.

## Integration

Declare the entities per area in Home Assistant, following the patterns in the [integration overview]({{ site.baseurl }}{% post_url 2026-10-18-home-assistant-radar-integration %}) and the per-model guides. Then build automations around *combinations*, which is where the real value sits:

- **Fall → notify.** A bathroom `fallStatus` change is the highest-priority event. Set `fallDuration` long enough to ignore someone who stumbled and recovered — see the fall radar guide for the timing trade-off.
- **Still too long → check.** Combine the bathroom fall radar's dwell alarm with the presence radar's occupancy: someone present in a room and completely still beyond your threshold is worth a nudge.
- **Night-time absence.** If the sleep radar reports out-of-bed and the presence radar reports no occupancy anywhere for a set window overnight, escalate.
- **Poor night, repeated.** End-of-session sleep score and stage durations trend badly several nights running — that is a conversation, not an alarm.

Keep notifications in your own application: step 5 in the fall guide's alarm chain is yours. **The radar does not place a call and does not contact anyone by itself.**

## Escalation design

Radar signals are noisy by nature; escalate in stages rather than on one measurement.

1. **Soft signal** — the module's own states help here: the fall radar can be configured with a fall duration so a recovered stumble never becomes an alarm, and its local voice call asks the person whether they are all right. A spoken reply can stand the alarm down before anyone is contacted. See the [fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}) for the voice phrases and their limits.
2. **Confirm on the network** — your backend correlates a second signal (presence in another room, absence elsewhere, time of day).
3. **Contact** — push, SMS, or a call placed by your own service, with a documented fallback when nobody acknowledges.

An **acknowledge button** matters as much as the alarm. Without it, carers learn to ignore the alert.

## Privacy by design

Radar output is the reason this is acceptable in bedrooms and bathrooms: distance, presence, motion, and state — **no video, no audio, no image**. With a broker you run yourself, data stays on the LAN by default; retention becomes a policy choice you control rather than a vendor's. If you are deploying in the EU, treat that local-only default as the starting point for your own compliance work and confirm details with your own advisor — this article is engineering guidance, not legal advice.

## Per-room deployment checklist

- **Bathroom / shower.** Ceiling-mount above the fixture and the open floor, not above the door; coverage for a small room is on the order of 3 m × 2 m, height ≤ 2.8 m. Small mistakes here dominate false alarms — see [fall detection failure modes]({{ site.baseurl }}{% post_url 2026-10-17-fall-detection-failure-modes %}).
- **Bedroom.** Side-wall at 1.8–2.2 m, or above the bed head tilted 30°–45°; breathing degrades first if you exceed roughly 1.5 m. The [sleep radar mounting article]({{ site.baseurl }}{% post_url 2026-10-16-sleep-radar-installation-position %}) has the geometry.
- **Living room / hallway.** Ceiling-mounted, long side toward the door, within about 3° of level.
- **Everywhere.** Keep fans, air-conditioner internal motors, swaying metal curtains, and pets out of the main lobe; the [occupancy relay article]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}) lists the same interference sources from the module datasheet.
- Keep to **2.4 GHz Wi-Fi** for the finished units.

## Limitations you should design around

- **No person counts.** Treat multi-occupancy as ambiguous; nobody should assume these modules resolve several people reliably.
- **Derived values.** Sleep stage and sleep score are computed, not measured. The sleep-sensing article explains what is measured versus inferred.
- **False alarms are a tuning problem, not a defect.** Differences between a small bathroom and a large bedroom change the right sensitivity and dwell settings.
- **Test in situ.** Ceiling height, layout, and reflections differ per home; a few nights of local data beats any default.

## Where to go next

- The three radars in detail: [fall]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}), [sleep]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}), [presence]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %})
- Broker you own: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- Device configuration: [setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %})
- Certifying your own product: [certification support]({{ site.baseurl }}{% post_url 2026-10-23-certification-support-ce-fcc-own-brand %})

Evaluation units and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
