---
title: 60GHz mmWave Radar SDK
description: 60GHz mmWave radar SDK for fall detection, sleep monitoring, and presence detection. Wi-Fi + MQTT, Home Assistant compatible, with UART protocol and voice command reference.
keywords: 60GHz radar, mmWave radar, fall detection, sleep monitoring, presence detection, Wi-Fi, MQTT, Home Assistant, UART
---

# 60GHz mmWave Radar SDK

60GHz mmWave radar SDK for fall detection, sleep monitoring, and presence detection. Supports Wi-Fi + MQTT and works with Home Assistant. Includes UART protocol, MQTT topics, installation guide, and voice command reference for 60GHz radar sensors.

## Latest Articles

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    <small>{{ post.date | date: "%Y-%m-%d" }}</small>
  </li>
{% endfor %}
</ul>

## Overview

This SDK provides complete documentation and communication protocols for 60GHz mmWave radar sensors. It covers three main radar categories:

- **Presence Radar** – Human presence, motion, distance, and direction detection
- **Fall Detection Radar** – Fall alarm, dwell alarm, and voice call for help
- **Sleep Radar** – Breathing, heart rate, sleep state, and sleep quality monitoring

The radar uses millimeter wave technology to detect human presence and movement without physical contact. It connects via Wi-Fi to a cloud platform and can be controlled through a mobile app for scene linkage and status reporting.

## Key Features

### Presence Detection
- Human presence/absence status
- Motion status (static / active)
- Body movement intensity (0–100)
- Human distance and position (x, y, z)
- Approach / leave detection
- Environmental status (unoccupied / static / moving)

### Fall Detection
- Fall alarm with configurable sensitivity and timing
- Static dwell alarm
- Voice call for help with customizable wake words
- Night light linkage
- History records for presence and fall events

### Sleep Monitoring
- Heart rate and breathing rate
- Heart rate and breathing waveform
- Sleep state (awake / light sleep / deep sleep)
- Sleep duration statistics
- Sleep quality score and analysis
- Sleep abnormality alerts

### Voice Interaction
- Voice call for help
- Customizable wake words and voice commands
- Local language learning mode
- Dialect and multi-language support

### App Control
- Device pairing and network configuration
- Real-time status monitoring
- Parameter configuration (installation height, sensitivity, timing)
- Alarm push notifications
- History records
- Voice learning and reset

## Communication Protocols

### Wi-Fi + MQTT
The radar devices use MQTT for data reporting and configuration. Data is transmitted in JSON format.

Two main topics are used:

- **Report Topic**: `/Radar{Type}/{DeviceID}/sys/property/post`
- **Set Topic**: `/Radar{Type}/{DeviceID}/sys/property/set`

Replace `{Type}` with the appropriate radar type identifier and `{DeviceID}` with the actual device ID.

### UART Protocol
For direct module integration, a standard UART protocol is available.

- **Baud Rate**: 115200 bps
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: None
- **Level**: TTL

Frame structure:

| Header | Control | Command | Length | Data | Checksum | Tail |
|--------|---------|---------|--------|------|----------|------|
| 0x53 0x59 | 1 Byte | 1 Byte | 2 Bytes | n Bytes | 1 Byte | 0x54 0x43 |

Checksum = (Header + Control + Command + Length + Data) sum, take low 8 bits.

## Installation Notes

### Wi-Fi Provisioning
- The device only supports the **2.4GHz** Wi-Fi band.
- The **5.8GHz (5G)** band cannot complete network configuration.
- Please double-check your Wi-Fi band before proceeding.

### Radar Installation
- Install the radar horizontally at the top of the detection area.
- Recommended installation height: **2.2–2.8 m** (2.4 m recommended).
- Keep the radar away from vibration sources, exhaust hoods, fans, flowing water, and vents.
- Avoid moving objects within the detection range.

### Sleep Radar Installation
- Recommended installation: above the bed head, tilted downward at **30°–45°**.
- Installation height: **0.8–1 m** above the bed surface.
- Maximum detection distance:
  - Human presence: ~2.5 m
  - Sleep detection: ~2.5 m
  - Breathing rate: ~1.5 m

## Detection Range

| Detection Type | Range |
|----------------|-------|
| Human presence | ~4 m × 3 m |
| Static dwell alert | ~3 m × 2.5 m |
| Fall detection | ~3 m × 2.5 m |
| Wide angle | 100° |
| Narrow angle | 90° |

## Main Data Points

| Data Point | Description | Report Frequency |
|------------|-------------|------------------|
| Human presence | Occupied / unoccupied | On change |
| Motion status | Static / active | On change |
| Body movement | 0–100 | Every 1 s |
| Human distance | 0–65535 cm | Every 2 s |
| Human position | x, y, z coordinates | Every 2 s |
| Heart rate | 50–120 bpm | Every 3 s |
| Breathing rate | 0–45 breaths/min | Every 3 s |
| Sleep state | Awake / light / deep | Every 10 min |
| Sleep score | 0–100 | On sleep end |
| Fall alarm | Fall detected | On change |
| Dwell alarm | Static dwell detected | On change |

## Voice Commands

| Function | Voice Command |
|----------|---------------|
| Call for help | "Help me please", "Can you help me", "I need help" |
| Confirm fall | "I have fallen down", "I fell down" |
| Confirm OK | "I am OK", "I'm alright", "I didn't fall" |
| Turn on light | "Turn on the light" |
| Turn off light | "Turn off the light" |
| Enter learning mode | "Enter learning mode" |
| Start learning | "Start learning" |
| Exit learning mode | "Stop learning" |
| Factory reset | "Restore default factory settings" |

## Learning Mode

The device supports local language learning for custom voice commands.

1. Wake the device with a help command.
2. Enter learning mode via voice or app.
3. Follow the device prompts to record custom phrases.
4. The device confirms "Learning Success" when both inputs match.
5. Up to three voice commands can be learned.
6. Use "Voice Reset" in the app or factory reset to restore defaults.

## Power Supply

| Parameter | Value |
|-----------|-------|
| Rated input | 5V / 3A |
| Radar frequency | 60GHz |
| Wireless connection | 2.4GHz Wi-Fi |
| Operating temperature | -10°C to +60°C |
| Working humidity | 0–95% RH, no condensation |

## Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Cannot connect to Wi-Fi | Wrong frequency band | Use 2.4GHz Wi-Fi only |
| False alarms | Moving objects in range | Remove interference sources |
| No presence detection | Installation height incorrect | Adjust to 2.2–2.8 m |
| Sleep data inaccurate | Radar not aligned | Tilt 30°–45° above bed head |
| Voice command not working | Learning mode not entered | Enter learning mode first |

## Documentation

- [Setup Guide: MQTT + Wi-Fi]({{ site.baseurl }}/guide/2026/10/17/radar-setup-guide.html)
- [Wi-Fi Provisioning Guide]({{ site.baseurl }}/guide/2026/05/07/wifi-provisioning-tips.html)
- [Installation Guide](#installation-notes)
- [Communication Protocols](#communication-protocols)
- [Voice Commands](#voice-commands)

