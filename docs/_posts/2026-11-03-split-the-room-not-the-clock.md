---
layout: post
title: "Multi-Function Whole-Home AI Monitor (HT-W)"
date: 2026-11-03 08:00:00 +0800
categories: [fundamentals]
tags: [radar, mmwave, multi-target, tracking, fall detection, sleep]
description: "Five types of sensing in one ceiling-mounted 60 GHz mmWave radar: headcount and position, falls, respiration, sleep, and room entry and exit, reported over Wi-Fi."
---

<!-- filename: 2026-11-03-split-the-room-not-the-clock.md -->

**Whole-home presence sensing and event reporting built on 60 GHz mmWave radar**

The Multi-Function Whole-Home AI Monitor (HT-W) is a 60 GHz mmWave radar product that senses human motion and posture. Across indoor scenarios, it combines algorithm processing and application logic to sense people within an area — including headcount, position, heart rate and respiration, falls, and sleep quality. It is stable, resistant to interference and privacy-preserving, suits elderly care and security applications, and transmits event information and device status over Wi-Fi.

## One Device, Five Types of Sensing

| Function | Output | Capacity |
|---|---|---|
| People and position tracking | Live positions, trajectories | 0–5 people |
| Fall and prolonged-stillness monitoring | Fall, stillness, low-posture alarms | 0–3 people |
| Respiration and heart rate | Breathing and heart rate values, high/low alerts | 1 person, one frame every 3 s |
| Sleep detection | Awake / light / deep, get-ups and turning over, sleep quality score | 1 person |
| Bed and room entry / exit | Entry and exit event notifications | 0–3 people |

All five run in parallel on a single device, without a separate sensor, power supply or data channel for each function. Each function has its own capacity limit, and system integration should be designed against the capacity of the function in question.

## Region Labels: Multi-Target and Multi-Task in Parallel

The field of view can be configured with region labels for a bed, a doorway, a sofa and similar areas. Labels are defined as polygons in the device's own coordinate system and can be adjusted to the actual room. Respiration, heart rate and sleep detection run inside the bed label; fall and prolonged-stillness monitoring run outside it; the doorway label handles entry and exit counting.

Dividing the space is what allows multi-target tracking and multi-task sensing to run at the same time. Within one room, the sleeper's breathing and sleep data and the risk events in the floor area are reported independently, without either waiting for the other.

## Stable, Interference-Resistant, Privacy-Preserving

The product captures no images and records no audio. Sensing results come only from radar echoes, which makes it suitable for bedrooms, bathroom entrances and other areas where a camera is not acceptable. 60 GHz radar does not depend on ambient light, and combined with the on-device processing it delivers stable indoor sensing with strong interference resistance.

Installation is ceiling-mounted with the sensing face pointing down, 2.4–3.0 m above the floor. Installed height, detection area and fall thresholds are all configurable remotely from the platform to match different room layouts.

## Wi-Fi Transport and Open Integration

Event information and device status are transmitted over Wi-Fi. The device reports over MQTT, connecting to a public broker by default, and an MQTT command switches it to the customer's own server. For volume deliveries the server address can be pre-configured at the factory, so on-site work is limited to network provisioning.

Reports include trajectory information — for each target, an index, position, height, speed, posture and projected radius — eight posture classes, live headcount, a comprehensive sleep status, sleep quality analysis and an end-of-night sleep report. The posture classes are fall, crouching, sitting on a chair, standing, low posture, prolonged stillness, lying in bed and sitting up in bed.

## Application Scenarios

**Elderly care.** Night-time fall alerts: a person remaining below the height threshold for 30 seconds triggers the first alarm, after which the device continues monitoring for two minutes. Also available are prolonged-stillness and low-posture notifications, high/low respiration and heart rate alerts, weak vital signs and abnormal struggling alerts, and prolonged-absence alerts. Overnight reports cover sleep stages, get-ups, turning over, apnea counts and a quality score, and can be used for long-term trend observation.

**Security and care management.** Room entry and exit can trigger notifications based on time away, and live headcount is reported — useful when a carer is present or someone remains in an area for an unusually long time. Multi-target position trajectories provide a data basis for space usage and abnormal dwell patterns.

## Key Specifications

| Item | Value |
|---|---|
| Radar | 60 GHz mmWave, 4T4R / FMCW |
| Detection coverage | 5 m × 4 m, about 20 m² |
| Detection angle | Horizontal 100°, vertical 90° |
| Installation | Ceiling-mounted, face down, 2.4–3.0 m above floor |
| Fall / sleep / respiration distance | 2 m each (projection radius, ceiling mounting) |
| Power / network | USB 5 V / 2 A, Type-C; Wi-Fi |
| Size / net weight | 89 × 21 mm / 54.2 g |
| Start-up stabilisation | ≥ 30 s |

## Summary

The HT-W brings five types of sensing — headcount, position, heart rate and respiration, falls, and sleep quality — into one ceiling-mounted device. Region labelling lets multiple targets and multiple tasks run in parallel, and Wi-Fi carries event information and device state outward, serving elderly care and security scenarios without compromising privacy.

For protocol documents, integration evaluation, or confirmation of the function set for a specific project, see the [developer documentation]({{ site.baseurl }}/), or contact us directly.

*This is product and engineering documentation, not a medical claim, safety certification, or performance guarantee for any particular installation. mmWave sensing carries an inherent false-alarm probability; any alerting product built on it should be designed on the assumption that it will sometimes be wrong. Some functions depend on the configuration of the specific model — refer to the manual for the model in question.*
