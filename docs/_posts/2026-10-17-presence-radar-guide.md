---
layout: post
title: "The Presence Detection Radar: Range, Sensitivity, and Scene Modes"
date: 2026-10-17 09:00:00 +0800
categories: [product]
tags: [presence radar, 60ghz, 24ghz, occupancy, scene mode, sensitivity, mqtt]
---

<!-- filename: 2026-10-17-presence-radar-guide.md -->

The presence detection radar answers one question continuously: is there a person in this space, and are they moving or still. Unlike a PIR sensor, it detects a person who is sitting, standing, or sleeping without any movement at all. Unlike a camera, it produces no image and works in complete darkness.

This article covers the full range of this radar, how sensitivity and scene modes change what it sees, where to mount it, and how to read the output. All ranges, timings, and values are the ones used by the models sold in this store.

**This is a consumer and commercial sensing device. It is not a medical device and does not diagnose, treat, or prevent any condition.**

## Two Variants, Two Protocols

This radar is available in two forms, and they use completely different protocols. Check which one you have before reading further.

| Variant | Frequency | Interface | Protocol | Configuration |
|---|---|---|---|---|
| Presence radar module | 24 GHz | UART only | Frame protocol at 115200 bps | Wired, no network setup |
| Presence radar | 60 GHz | 2.4 GHz Wi-Fi + MQTT | JSON property protocol over MQTT topics | Web page for MQTT, Android app for Wi-Fi |

Both share the same sensing principle, so the outputs mean the same thing. The coverage, the commands, and the reporting details differ, and every value in this article is marked with the variant it belongs to.

The **presence radar module** is the bare sensing core for integrators who embed it in their own product; its output is described under Over UART. The **presence radar** is the finished ceiling-mounted unit that joins your network; its output is described under Over Wi-Fi and MQTT.

Setting up the presence radar, the Wi-Fi variant, is covered step by step in the [setup guide]({% post_url 2026-10-17-radar-setup-guide %}). The module needs no network setup at all.

## What This Radar Is

The **presence radar** is a 60 GHz millimeter wave sensor in a housing of approximately 29.5 × 73 × 86 mm, designed to be recessed into a ceiling. It is powered over USB at 5 V and covers a radius of about **2.7 m** when ceiling-mounted, with a beam of **100° horizontal** and **80° vertical**. It is built for small to medium rooms: bathrooms, bedrooms, offices, and hotel rooms.

The **presence radar module** is a 24 GHz sensing core without the housing and without the Wi-Fi section. It is designed for ceiling mounting up to **2.75 m** and covers a much wider area, up to about **12 m** across for a moving person. It is intended for corridors, open-plan floors, and large rooms where one unit has to cover everything.

Both detect a stationary person. Both expose a body movement value from 0 to 100 that tells you not just whether someone is there, but what they are doing.

## How It Detects a Person Who Is Not Moving

A PIR sensor needs movement. It reacts to a change in infrared pattern, so a person who sits still for a few seconds becomes invisible to it. That is why PIR lights switch off while you are still in the room.

A millimeter wave radar does not need movement. It transmits a continuous wave and analyses the echo, and a human body reflects that signal whether the person is walking, sitting, or asleep. Even a perfectly still person has a chest that rises and falls, and the radar resolves displacements far smaller than that.

The practical consequence is the timing:

| Transition | Time to Report |
|---|---|
| No person to person present | Within **0.5 s** |
| Person present to no person | About **40 s** |

The asymmetry is deliberate. Declaring a room occupied takes one glance, because a false negative leaves a light off or a room marked empty. Declaring it empty takes 40 seconds of confirmed absence, because the radar keeps checking for micro-movement and breathing before it commits. A PIR sensor has the opposite bias, which is why it gives up on you.

## What It Measures

### Presence

| Value | Meaning |
|---|---|
| 0 | No person |
| 1 | Person present |

Reported on change, within 0.5 s of a person appearing and about 40 s after the last confirmed trace disappears.

### Motion Status

| Value | Meaning |
|---|---|
| 0 | No movement |
| 1 | Still (static) |
| 2 | Active |

Reported on change, within 0.5 s.

### Approach and Leave Direction

The presence radar module also reports which way a person is moving relative to the radar:

| Value | Meaning |
|---|---|
| 1 | No directional movement |
| 2 | Approaching |
| 3 | Moving away |
| 4 | Continuously approaching |
| 5 | Continuously moving away |

Reported on change, roughly every 2 s while movement continues. This is the value that lets a corridor radar know which direction someone walked, without any camera.

### Body Movement Level

A single value from 0 to 100. The presence radar module reports it **every 5 s**; the presence radar reports it every second over MQTT. The ranges have fixed meanings:

| Value | Meaning |
|---|---|
| 0 % | No person in range |
| 1 % | Still: breathing only, no limb movement |
| 2 %–30 % | Micro movement: small head or limb motion |
| 31 %–60 % | Walking or fast limb movement |
| 61 %–100 % | Running or large close-range movement |

This is the most useful value for tuning an installation. In a room where someone is sitting at a desk, a correctly configured radar reports values in the 1 %–30 % band and never drops to 0.

### Human Distance and Position

The presence radar module can report distance in centimetres and position as x, y, and z coordinates, both about every 2 s. Both are marked as **non-critical functions with limited accuracy** in the documentation. They are useful for debugging an installation, but they should not be the basis of a product feature.

### Sensitivity

The presence radar module exposes a sensitivity setting from **1 to 10**, with **7 as the default**.

| Range | Intended for |
|---|---|
| 7–10 | Low-interference spaces where fine motion must be caught: storage rooms, vaults |
| 4–6 | Ordinary indoor spaces: homes, hotels, offices (default) |
| 1–3 | Spaces with heavy interference: corridors, parking areas, where only large movement should trigger |

Higher sensitivity catches smaller movements and also picks up more false triggers from curtains and fans. Lower sensitivity ignores a person who is barely moving.

### Scene Mode

The presence radar module ships with seven scene modes. Each mode changes the **trigger** range, that is, the distance at which a moving person is detected:

| Scene mode | Trigger range, wide angle | Trigger range, narrow angle | Static range |
|---|---|---|---|
| Maximum area | 12 m | 8 m | 6 m / 4 m |
| Office | 10 m | 7 m | 6 m / 4 m |
| Hotel (default) | 9 m | 6 m | 6 m / 4 m |
| Living room | 7 m | 5 m | 6 m / 4 m |
| Bedroom | 4 m | 3 m | 6 m / 4 m |
| Zone detection | 3 m | 2 m | 6 m / 4 m |
| Bathroom | 1.8 m | 1.5 m | 6 m / 4 m |

The static range is the same across modes; what changes is how far away a **moving** person is noticed. Choosing the smallest mode that still covers the room is the single most effective way to remove false triggers.

The presence radar exposes scene mode as an MQTT property, but with a different numbering: `0` default, `1` zone detection, `2` bathroom, `3` bedroom, `4` living room, `5` office, `6` hotel. Do not mix the two lists; they are different protocols.

### Forced Unmanned

The presence radar module can force an unoccupied output after a fixed time, regardless of what it detects:

| Value | Forced unoccupied after |
|---|---|
| 0 | Not used |
| 1 | 10 s |
| 2 | 30 s |
| 3 | 1 min |
| 4 | 2 min |
| 5 | 5 min |
| 6 | 10 min |
| 7 | 30 min |
| 8 | 60 min |

Use this where a lingering signal is worse than a missed one, for example a meeting room that must be released for the next booking.

## Reporting Rhythm

| Data Point | Range | Reported |
|---|---|---|
| Presence | 0 / 1 | On change, 0.5 s in, ~40 s out |
| Motion status | 0 / 1 / 2 | On change, 0.5 s |
| Approach / leave | 1–5 | On change, ~2 s |
| Body movement | 0–100 | Every 5 s (module), every 1 s (radar) |

## Detection Range

### Presence radar (60 GHz, ceiling-mounted)

| Parameter | Value |
|---|---|
| Detection radius | ~2.7 m |
| Beam, horizontal | 100° |
| Beam, vertical | 80° |
| Mounting | Recessed in ceiling |

### Presence radar module (24 GHz, ceiling-mounted)

| Signal | Maximum distance |
|---|---|
| Moving person | ~12 m |
| Sitting or micro-movement | ~6 m |
| Sleeping person | ~3 m |

In practice, the covered area is approximately **8 m × 12 m** for a moving person, **4 m × 6 m** for a stationary person, and **3 m × 2 m** for a sleeping person. The actual area is oval rather than circular.

## Installation

### Mounting

Both variants are **ceiling-mounted**. Mounting the module on a wall or at an angle changes the covered shape and is not supported.

For the presence radar module:

- Mount horizontally, with the radar's long edge facing the room entrance so a person entering crosses the beam head-on
- Keep the horizontal deviation within **3°**
- Mount at a height of **2.75 m** or less
- Keep the area in front of the radar clear of obstructions and covers

### Tuning the Scene Mode and Sensitivity

The two settings work together, and the module documentation gives a clear procedure:

1. Compare the room area with the scene mode table and pick a starting mode.
2. Confirm where people actually spend time; the centre of that area is where the radar belongs.
3. Find the room entrance and aim the radar's long edge at it.
4. Check the detection area for interference sources.
5. If interference exists, shrink the trigger range by moving to a smaller scene mode.
6. Decide the starting sensitivity from the room type.
7. If the room is small (**15 m² or less**), reduce the sensitivity by one step. Small rooms strengthen reflections, so lowering sensitivity keeps the unoccupied judgement stable.
8. If the room is large (**40 m² or more**), raise the sensitivity by one step. Large rooms weaken reflections, so raising sensitivity keeps the occupied judgement stable.

When the entrance trigger and the interference resistance conflict, the documentation recommends protecting interference resistance first. A radar that occasionally triggers late is better than one that reports a person who is not there.

### What Interferes

Avoid aiming the radar at:

- Electric fans
- Air conditioners (the internal motor causes false detection)
- Curtains that move with air flow
- Metal objects
- Water pipes or flowing water
- Pets moving in the area

### What the Signal Passes Through

The signal is not blocked by:

- Clothing
- Curtains
- Thin wooden boards
- Glass

## Getting the Data Out

### Over Wi-Fi and MQTT (presence radar)

The presence radar uses a set of topics per function rather than one pair:

| Topic | Direction |
|---|---|
| `/RadarHP/{DeviceID}/sys/online/post` | Device reports online / offline |
| `/RadarHP/{DeviceID}/sys/version/post` | Device reports version information |
| `/RadarHP/{DeviceID}/sys/property/set` | You send settings |
| `/RadarHP/{DeviceID}/sys/property/set_reply` | Device answers a setting |
| `/RadarHP/{DeviceID}/sys/active/post` | Device reports fast-changing values |
| `/RadarHP/{DeviceID}/sys/passive/post` | Device reports configuration values |

The main properties:

| Property | Meaning | Access |
|---|---|---|
| `online` | 0 offline, 1 online | post |
| `radarID` | Device identifier | post, get |
| `softwareVersion` | Software version | post, get |
| `firmwareVersion` | Firmware version | post, get |
| `protocolVersion` | Protocol version | post, get |
| `movementSigns` | Body movement, 0–100 | post |
| `keepAway` | 1 none, 2 approaching, 3 leaving, 4 continuous approaching, 5 continuous leaving | post |
| `environ` | 0 unoccupied, 1 occupied still, 2 occupied moving | post |
| `thresholdGear` | Sensitivity, 0–9 | post, set |
| `sceneMode` | 0 default, 1 zone, 2 bathroom, 3 bedroom, 4 living room, 5 office, 6 hotel | post, set |
| `forceIntoUnmannedGear` | Forced unoccupied, 0–8 | post, set |

Values are strings. Settings are sent with `"method": "set"` to the property set topic, and the device answers on the set reply topic.

Two control commands apply to the fast-changing values:

- `"method": "control"` turns individual reports on or off. A value of **1 allows the report, 0 forbids it**. Note that this is the opposite of the convention used by the 60 GHz radars, where 1 means forbidden.
- `"method": "control_qos"` sets the quality of service per topic, using keys such as `Qos_movementSigns`, `Qos_keepAway`, and `Qos_environ`. The default is QoS 1.

### Over UART (presence radar module)

The presence radar module exposes a UART interface at **115200 bps**, 8 data bits, 1 stop bit, no parity, 3.3 V TTL level.

```
Header   Control  Command  Length   Data     Checksum  Tail
0x53 0x59  1 Byte   1 Byte   2 Bytes  n Bytes  1 Byte    0x54 0x43
```

The checksum is the sum of the header, control byte, command byte, length bytes, and data, taking the low 8 bits.

Control byte `0x80` covers human presence. Command `0x01` reports presence, `0x02` reports motion status, `0x03` reports the body movement value, `0x04` reports distance, and `0x05` reports position. Query commands are the same values with the high bit set, for example `0x81` to query presence.

The module also has two hardware outputs that need no protocol at all:

| Pin | Output |
|---|---|
| S1 | High = person present, low = no person |
| S2 | High = active, low = still |

Both are 3.3 V logic, so a microcontroller can read presence directly without parsing any frame. GP1 to GP4 are parameter selection pins that can be redefined for your own use.

## Where It Is Used

**Bathrooms and small rooms.** One 60 GHz unit covers a bathroom reliably, including a person sitting still on the toilet, which a PIR sensor misses entirely. This is the most common single use: occupancy-based lighting and extractor fans.

**Corridors and open-plan floors.** The 24 GHz module covers up to 12 m, so one unit watches a whole corridor or an open office, including which direction people walk.

**Meeting rooms and desks.** Presence plus the movement level distinguishes a room that is genuinely empty from one where someone is quietly working, which is exactly the distinction PIR-based systems get wrong.

**Hotels.** Occupancy-based climate and lighting control, with no camera in the room and nothing for the guest to object to.

**Integrators.** The module's S1 and S2 outputs give a two-wire presence signal for products that have no network at all.

## What It Cannot Do

- **It is not a medical device.** It reports occupancy and movement. It does not diagnose, treat, or prevent any condition.
- **It cannot count people reliably.** It reports whether at least one person is present, not how many. Two people in the area read the same as one.
- **It cannot identify people.** There is no way to tell who is in the room from the radar output alone.
- **It cannot see through metal.** A metal ceiling panel, a metal light fitting, or a metal cabinet between the radar and the person will degrade or block the signal.
- **It does not react instantly to an empty room.** The 40-second unoccupied delay is built in. A system that needs a room marked empty in 2 seconds will not get it from this radar without forcing it.
- **High sensitivity is not free.** Catching a person who is barely moving also means catching a curtain that is barely moving.
- **Scene mode is not cosmetic.** Leaving a large mode on a small room is the most common cause of a radar that reports people who are not there.

## Troubleshooting

| Symptom | Possible Cause | Solution |
|---|---|---|
| Reports unoccupied while someone sits still | Sensitivity too low | Raise the sensitivity by one step |
| | Radar too far from the person | Check the scene mode range against the room |
| Reports occupied in an empty room | Scene mode too large for the room | Move to a smaller scene mode |
| | Interference source in the beam | Remove fans, curtains, or pets from the area |
| Person entering is not detected | Radar's long edge not facing the entrance | Re-aim the long edge at the door |
| Unoccupied state flickers | Room too small for the sensitivity | Reduce sensitivity by one step |
| Movement value stuck at 0 | Person outside the detection area | Move the radar or reduce the scene mode |
| No data over MQTT | Report forbidden by the control command | Check the `control` settings and the QoS values |

## Specifications

### Presence radar (60 GHz)

| Parameter | Value |
|---|---|
| Radar frequency | 60 GHz |
| Detection radius | ~2.7 m |
| Beam angle | 100° horizontal, 80° vertical |
| Mounting | Recessed ceiling mount |
| Supply | 5 V over USB |
| Housing size | 29.5 × 73 × 86 mm |
| Wireless | 2.4 GHz Wi-Fi |
| Data output | MQTT over Wi-Fi (JSON) |

### Presence radar module (24 GHz)

| Parameter | Value |
|---|---|
| Radar frequency | 24 GHz |
| Beam angle | 90° horizontal, 60° vertical |
| Mounting | Ceiling, horizontal, deviation ≤ 3° |
| Mounting height | 2.75 m or less |
| Moving person range | ~12 m |
| Static or micro-movement range | ~6 m |
| Sleeping person range | ~3 m |
| Presence response | Within 0.5 s |
| Unoccupied confirmation | About 40 s |
| Body movement reporting | Every 5 s |
| Sensitivity | 1–10, default 7 |
| Scene modes | 7 |
| Hardware outputs | S1 presence, S2 motion, 3.3 V logic |
| Data output | UART, 115200 bps, 8N1, 3.3 V TTL |

## Summary

The presence detection radar reports whether a space is occupied and whether the person is still or moving, within 0.5 s of someone appearing and after about 40 s of confirmed absence. It detects a stationary person, which is the one thing a PIR sensor cannot do.

The 60 GHz presence radar covers about 2.7 m from a ceiling and suits bathrooms, bedrooms, and offices. The 24 GHz presence radar module covers up to 12 m and suits corridors and open floors, and it can report presence on two hardware pins with no protocol at all.

Mount it horizontally, aim the long edge at the entrance, choose the smallest scene mode that still covers the room, and set the sensitivity to match the room size. Keep fans, air conditioners, moving curtains, and metal out of the beam, and the radar will hold a stable occupied or unoccupied state for years.
