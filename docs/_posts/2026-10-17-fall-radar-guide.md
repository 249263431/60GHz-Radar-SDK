---
layout: post
title: "The Fall Detection Radar: How Alarms Work, How to Tune Them, and What to Expect"
date: 2026-10-17 09:00:00 +0800
categories: [product]
tags: [fall radar, 60ghz, 24ghz, fall detection, dwell alarm, bathroom, mqtt]
---

<!-- filename: 2026-10-17-fall-radar-guide.md -->

The fall detection radar watches a space for two things: a person who has fallen, and a person who has not moved for too long. It reports both as alarms over MQTT, together with the position where the fall happened and the height the person was at.

This article covers how the detection works, what every alarm field means, how to tune the timing and sensitivity, where to mount the radar, and how to read the output. All timings, ranges, and values are the ones used by the models sold in this store.

**This is a sensing device, not a medical alarm. It does not diagnose, treat, or prevent any condition, and it is not a substitute for human care, a call system, or any emergency service.**

## Two Variants, Two Protocols

This radar is available in two forms, and they use completely different protocols. Check which one you have before reading further.

| Variant | Frequency | Interface | Protocol | Configuration |
|---|---|---|---|---|
| Fall radar module | 24 GHz | UART only | Frame protocol at 115200 bps | Wired, no network setup |
| Fall radar | 60 GHz | 2.4 GHz Wi-Fi + MQTT | JSON property protocol over MQTT topics | Web page for MQTT, Android app for Wi-Fi |

Both share the same sensing principle, so the outputs mean the same thing. The alarm fields, the sensitivity scale, and the timing ranges differ substantially, and every value in this article is marked with the variant it belongs to.

The **fall radar module** is the bare sensing core for integrators who embed it in their own product; its output is described under Over UART. The **fall radar** is the finished ceiling-mounted unit that joins your network; its output is described under Over Wi-Fi and MQTT.

Setting up the fall radar, the Wi-Fi variant, is covered step by step in the <a href="{{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}">setup guide</a>. The module needs no network setup at all.

## What This Radar Is

The **fall radar** is a 60 GHz millimeter wave sensor in a housing of approximately 87 × 87 × 24 mm, designed to be mounted on a ceiling looking straight down. It is built for small, high-risk spaces: bathrooms, showers, and toilets, where a camera is unacceptable and a call button is often out of reach.

It covers a low-posture detection area of about **3 m × 2 m**, mounts at a height of **2.8 m or less** with the sensor face pointing vertically down, and reports fall alarms, dwell alarms, presence, movement level, and the height of the person.

The finished unit also carries a light, a microphone, and a speaker: an LED night light that switches on when someone is present, and a voice function that speaks after a fall and listens for a spoken reply. The voice language is selectable; see Voice Call and Voice Reply.

The **fall radar module** is a 24 GHz sensing core without the housing and without the Wi-Fi section. It exposes a UART interface, reports a simpler set of alarm fields, and is intended to be built into another product such as a ceiling light or a wall panel.

## How Fall Detection Works

A fall is a fast, large change in the shape and position of the radar echo. Over roughly a second, a standing person's reflection collapses from a tall target at chest height to a wide, low target on the floor. The radar detects that collapse and the height that follows it.

Three properties make this workable indoors:

- **Height.** The radar measures the height of the reflecting target, so it can tell a person standing at 1.7 m from a person lying at 0.3 m.
- **Speed.** The transition is fast, which separates a fall from someone crouching down deliberately.
- **Persistence.** A fallen person stays low. The radar keeps reporting the low height until the person gets up.

The 60 GHz fall radar also exposes the **height distribution**: the proportion of time the detected target spent in the bands 0–0.5 m, 0.5–1.0 m, 1.0–1.5 m, and 1.5–2.0 m. A fall shows up as a sharp shift of that distribution toward the lowest band.

The finished unit adds a fixed rule on top of the algorithm: a body held below **70 cm** in a static posture triggers the fast alarm after **30 seconds**, and a person who stays completely still triggers the alarm after **5 minutes**.

## What It Measures

### Presence and Motion

| Property | Values |
|---|---|
| `someoneExists` | 0 no person, 1 person present |
| `motionStatus` | 0 no movement, 1 still, 2 active |
| `movementSigns` | Body movement level, 0–100 |

Reported on change. These are the same fields as the presence radar and are reported alongside every alarm.

### Fall Status

| Property | Values |
|---|---|
| `fallStatus` | 0 no fall, 1 fall |

Reported on change. This is the primary alarm.

### Fall Position

`fallPosition` reports the x and y coordinates of the fall, each in the range 0–65535, at the moment the fall is detected. In a single-radar room this identifies roughly where in the coverage area the person went down, which is what lets a care dashboard show a floor plan rather than a room number.

### Fall Duration

`fallDuration` is configurable from **0 to 65535 seconds**. It is the delay between the radar deciding a fall happened and the alarm being published. Setting it to 0 reports immediately; setting it to 30 means a fall is only reported if the person is still down half a minute later, which filters out someone who stumbled and got straight back up.

### Fall Sensitivity

The 60 GHz fall radar exposes `fallSensitivity` from **0 to 3**.

The 24 GHz fall radar module uses a completely different scale: **0 to 9, with 6 as the default**, where a higher number is more sensitive. Do not carry a value from one radar to the other.

### Dwell Status and Duration

A dwell alarm covers the person who does not fall but stops moving: someone who faints, sits down and does not get up, or lies still for hours.

| Property | Values |
|---|---|
| `residentStatus` | 0 no dwell, 1 dwell detected |
| `residentWarningDuration` | 0–65535 seconds |
| `residentWarningDurationSwitch` | 1 on, 0 off |

The finished unit applies a fixed 5-minute dwell rule in addition to the configurable one.

The 24 GHz fall radar module reports dwell as a staged alarm instead:

| Value | Meaning |
|---|---|
| 0 | No dwell alarm |
| 1 | First alarm, 5 minutes after movement stopped |
| 2 | Second alarm, 10 minutes after movement stopped |
| 3 | Third alarm, 30 minutes after movement stopped |
| 4 | Fourth alarm, 60 minutes after movement stopped |

### Fall Alarm

The 24 GHz fall radar module reports `fallAlarm` with three states rather than two:

| Value | Meaning |
|---|---|
| 0 | Suspected fall |
| 1 | Fall |
| 2 | No fall |

The suspected state exists so an integrator can raise a soft notification before committing to a full alarm.

### Low Posture and Height Distribution

Two properties describe how low the person is:

- `fallingAndBreakingHeight`, 0–150 cm, sets the height below which a target counts as fallen. Ceiling mounting only.
- `proportionOfHeight` reports the share of time spent in four height bands: 0–0.5 m, 0.5–1.0 m, 1.0–1.5 m, and 1.5–2.0 m, together with the total sample count.

### Unmanned Time

`unmannedTime`, from **10 to 1800 seconds**, sets how long the area must be empty before the radar reports it. Ceiling mounting only. This is the field that lets a bathroom radar tell "the person left" from "the person is on the floor and no longer moving", which is the distinction a dwell alarm depends on.

### Presence Energy and Threshold

On ceiling-mounted units the radar exposes `humanPresenceJudgmentThreshold` and `humanPresenceEnergyValue`, plus a switch for reporting the energy value. These are tuning fields for integrators dealing with an unusual room; the defaults are correct for normal bathrooms.

## Reporting Rhythm

| Data Point | Range | Reported |
|---|---|---|
| Presence | 0 / 1 | On change |
| Motion status | 0 / 1 / 2 | On change |
| Body movement | 0–100 | On change |
| Fall status | 0 / 1 | On change |
| Fall position | x, y, 0–65535 | On fall |
| Dwell status | 0 / 1 | On change |
| Height distribution | 4 bands, 0–100 | On change |
| Fall alarm | 0 / 1 / 2 | On change (module) |
| Dwell alarm | 0–4 staged | On change (module) |

## Detection Range and Installation

### Coverage

| Parameter | Value |
|---|---|
| Low-posture detection area | ~3 m × 2 m |
| Fall and dwell detection area | ~3 m × 2.5 m |
| Mounting height | 2.8 m or less |
| Orientation | Sensor face pointing vertically down |

### Mounting

Both variants are **ceiling-mounted**, looking straight down. A tilted or wall-mounted fall radar does not produce a usable height measurement, and the height measurement is what the whole detection depends on.

Mount the radar directly above the area where a person would fall, not above the door. In a bathroom that means above the toilet and the middle of the floor, not above the entrance.

### Tuning the Alarm

Four settings decide how the radar behaves, in order of impact:

1. **Fall duration.** Start with a short value and raise it only if you get alarms from people who stumbled and recovered. A short duration catches more real events; a long one produces fewer false ones.
2. **Dwell duration.** Set this to the longest period of stillness you consider normal in that room. In a bathroom, a few minutes is normal; in a bedroom, an hour is normal.
3. **Fall sensitivity.** Raise it if small or slow collapses are missed, lower it if the radar alarms on someone bending over. Remember that the two variants use different scales.
4. **Falling height.** Lower it if the radar alarms on people crouching or sitting on the floor; raise it if a collapse onto furniture is missed.

The finished unit applies its own fixed rules on top: 30 seconds for a body held below 70 cm, and 5 minutes for complete stillness. These are not configurable.

## How an Alarm Is Processed

1. **Detection.** The radar sees the height collapse and the low persistent target.
2. **Delay.** The configured fall duration elapses. If the person is upright again before it ends, nothing is published.
3. **Publication.** `fallStatus` changes to 1 and `fallPosition` reports the coordinates.
4. **Local output.** On the finished unit, the speaker asks whether the person is all right and waits for a spoken reply, and the night light switches on. See Voice Call and Voice Reply.
5. **Escalation.** Your system decides what happens next: a push notification, a call, a dashboard entry.

Steps 1 to 4 are the radar's job. Step 5 is yours, and the radar does not perform it. No message leaves your network unless your broker and your application send it.

## Voice Call and Voice Reply

The finished fall radar carries a microphone and a speaker, so a fall does not have to end in a silent alarm. When the radar detects a fall, it speaks first, asks whether the person is all right, and waits for a spoken answer. The person replies by voice, and the radar either announces that help is being called or stands the alarm down.

The device is woken with a call-for-help phrase, and the same phrase set works whether the unit is speaking Chinese or English:

| Step | Chinese | English |
|---|---|---|
| Wake the device | 来人来人 | "Help me please", "Can you help me", "Could you help me", "I need help" |
| Device asks after a fall | 请问是否跌倒？/ 你跌倒了吗？ | "Have you fallen down?" |
| Reply: confirm the fall | 我跌倒了 / 跌倒了 | "I have fallen down", "I fell down", "I fell" |
| Device response | 请稍等我马上通知监护人 / 已通知监护人 | "Notifying the guardian immediately", "Guardians notified, please wait a moment" |
| Reply: cancel the alarm | 我没有跌倒 / 没人跌倒 / 无人跌倒 | "I am OK", "I'm alright", "I didn't fall", "Nobody fell down" |
| Device response | 好的 | "OK!" |
| Turn the light on | 打开灯光 | "Turn on the light" |
| Turn the light off | 关闭灯光 | "Turn off the light" |
| Enter learning mode | 进入学习模式 | "Enter learning mode" |
| Start learning | 开始学习 | "Start learning" |
| Exit learning mode | 退出学习 | "Stop learning" |
| Factory reset | 恢复出厂设置 | "Restore default factory settings" |

### Choosing the Language

The voice language is set before the unit ships. **Specify Chinese or English when you place the order.**

If the wrong language arrives, or you want to move to a different one, the command words can be re-taught on the device. The unit supports local voice learning, including dialects and other languages, so the wake word and the two fall replies can be replaced with whatever suits the user.

### How Voice Learning Works

1. Make sure voice output is enabled in the app.
2. Wake the device with the wake word.
3. Say "进入学习模式" or "Enter learning mode", or tap the voice learning button in the app.
4. The device confirms and asks you to say "开始学习" or "Start learning".
5. The device prompts for each command in turn — the wake word, the fall confirmation, and the cancel phrase. Repeat the phrase you want; the device asks you to say it again, and two matching inputs count as learned.
6. The device announces that learning is complete.

Three points to know:

- **Only three commands can be learned at a time.** To change them, reset the device to factory settings and learn again, or use the voice reset button in the app.
- **Learning times out after 10 seconds of silence**, and you can exit early with "退出学习" or "Stop learning".
- **A quiet room matters.** Two inputs that do not match, or a noisy environment, cause the device to report a failed attempt.

### What the Voice Does and Does Not Do

The spoken confirmation is produced by the speaker on the device. When the device says that the guardian is being notified, that is a local prompt for the person on the floor — the actual notification to a caregiver is delivered by your own system over MQTT, exactly as described in How an Alarm Is Processed. The radar does not place a call and does not contact anyone by itself.

## Getting the Data Out

### Over Wi-Fi and MQTT (fall radar)

Two topics are used, with the model prefix for this radar being `Radar60FL`:

- Report topic: `/Radar60FL/{DeviceID}/sys/property/post`
- Set topic: `/Radar60FL/{DeviceID}/sys/property/set`

Data is JSON. A typical fall report looks like:

```json
{"version": "1.0", "method": "post", "params": {"fallStatus": "1"}}
```

The properties you will use most:

| Property | Meaning | Access |
|---|---|---|
| `fallSwitch` | Fall function on / off | get, set |
| `fallStatus` | 0 no fall, 1 fall | post, get |
| `fallPosition` | x, y coordinates of the fall | post |
| `fallDuration` | Delay before the alarm, 0–65535 s | get, set |
| `fallSensitivity` | 0–3 | get, set |
| `fallingAndBreakingHeight` | Fall height threshold, 0–150 cm | get, set |
| `residentStatus` | 0 no dwell, 1 dwell | post, get |
| `residentWarningDuration` | Dwell delay, 0–65535 s | get, set |
| `residentWarningDurationSwitch` | Dwell function on / off | get, set |
| `someoneExists` | Presence | post, get |
| `motionStatus` | Motion status | post, get |
| `movementSigns` | Body movement, 0–100 | post |
| `humanDistance` | Distance, cm | post, get |
| `humanPosition` | x, y, z coordinates | post, get |
| `unmannedTime` | Empty confirmation, 10–1800 s | get, set |
| `proportionOfHeight` | Time share in four height bands | post |
| `installHeight` | Ceiling 220–280 cm, side 180–240 cm | get, set |
| `installAngle` | 0° or 45° supported | get, set |
| `sceneMode` | 1 living room, 2 bedroom, 3 bathroom | get, set |

Values are reported as strings. Most properties support a query with `"method": "get"` and a setting with `"method": "set"`; the device answers on the report topic.

The radar also supports `limit_set` and `limit_get` to forbid or allow individual reports, `qos_set` and `qos_get` for quality of service, and `ota_set` for firmware updates. Note that in this protocol a value of **1 forbids a report and 0 allows it**, which is the opposite of the 24 GHz radars.

### Over UART (fall radar module)

The fall radar module exposes a UART interface at **115200 bps**, 8 data bits, 1 stop bit, no parity, 3.3 V TTL level.

```
Header   Control  Command  Length   Data     Checksum  Tail
0x53 0x59  1 Byte   1 Byte   2 Bytes  n Bytes  1 Byte    0x54 0x43
```

The checksum is the sum of the header, control byte, command byte, length bytes, and data, taking the low 8 bits.

Control byte `0x80` covers human presence, including the fall and dwell alarm fields. Query commands are the same values with the high bit set. The module reports `fallAlarm` with three states and `residentAlarm` with the four staged timings described above.

The module also exposes the same two hardware outputs as the presence module:

| Pin | Output |
|---|---|
| S1 | High = person present, low = no person |
| S2 | High = active, low = still |

Both are 3.3 V logic. There is no hardware alarm output; a fall alarm has to be read from the protocol.

## Where It Is Used

**Bathrooms at home.** The highest-value single installation. A bathroom is where falls happen, where a camera is unacceptable, and where a call button is often unreachable. The radar needs no button and no worn device.

**Elderly care and nursing homes.** One radar per room gives a dashboard of which room has a fall or a dwell event, with the position inside the room. No cameras in any room, nothing for the resident to wear or remember.

**Accessible hotel and hospital rooms.** Same logic, with the advantage that nothing is visible and nothing has to be cleaned.

**Integrators.** The module drops into a ceiling light or a wall panel and reports alarms over a serial line to the host product.

## What It Cannot Do

This section matters more for this radar than for any other in the range.

- **It is not a medical alarm and not an emergency service.** It does not call anyone. It publishes a message to your broker, and what happens next is entirely up to your system.
- **It does not detect every fall.** A fall against a wall, a fall behind a door, a slow slide rather than a collapse, and a fall outside the coverage area can all be missed. No sensor of this type catches all of them.
- **It will produce false alarms.** Someone crouching to reach a shelf, someone sitting on the floor deliberately, a pet, and a moving curtain can all trigger one. Tuning reduces this; it does not remove it.
- **It cannot tell a person from a large pet.** A dog moving at floor height looks similar to a person on the floor.
- **It cannot see through metal or water.** A metal shower screen, a metal light fitting, or a running shower between the radar and the person degrades the signal.
- **It does not know who fell.** Position is a coordinate inside the coverage area, not an identity.
- **It must not be the only safeguard.** For anyone at real risk, this radar is an addition to human checks and a call system, never a replacement.
- **The voice function is local.** The spoken confirmation comes from the speaker on the device. Delivering a real notification to a caregiver is done by your own system over MQTT.
- **The voice language is fixed at shipping.** Specify Chinese or English when you order. Changing it afterwards means re-teaching the command words on the device.

## Troubleshooting

| Symptom | Possible Cause | Solution |
|---|---|---|
| No fall alarm for a real fall | Fall duration too long | Shorten `fallDuration` |
| | Fall sensitivity too low | Raise `fallSensitivity` |
| | Radar not above the fall area | Move it above the toilet and floor centre |
| Alarm on someone bending over | Fall height threshold too high | Lower `fallingAndBreakingHeight` |
| | Fall sensitivity too high | Lower `fallSensitivity` |
| Alarm on a pet | Pet inside the coverage area | Restrict the scene mode or move the radar |
| Dwell alarm never fires | Dwell function switched off | Set `residentWarningDurationSwitch` to 1 |
| | Dwell duration longer than the stillness | Shorten `residentWarningDuration` |
| Reports unoccupied while a person lies still | Unmanned time too short | Raise `unmannedTime` |
| Height distribution always in the lowest band | Radar mounted too high or tilted | Mount at 2.8 m or less, face straight down |
| Alarm published late | Fall duration set high | Reduce the delay |

## Specifications

### Fall radar (60 GHz)

| Parameter | Value |
|---|---|
| Radar frequency | 60 GHz |
| Low-posture detection area | ~3 m × 2 m |
| Fall and dwell detection area | ~3 m × 2.5 m |
| Mounting | Ceiling, face down |
| Mounting height | 2.8 m or less |
| Fall height threshold | 0–150 cm |
| Fall duration | 0–65535 s |
| Fall sensitivity | 0–3 |
| Dwell duration | 0–65535 s |
| Unmanned time | 10–1800 s |
| Fixed rules | Below 70 cm static: 30 s; fully still: 5 min |
| Housing size | 87 × 87 × 24 mm |
| Supply | 5 V DC |
| Wireless | 2.4 GHz Wi-Fi |
| Data output | MQTT over Wi-Fi (JSON) |
| Local output | Night light, voice alarm, two-way voice reply |
| Voice language | Chinese or English, set before shipping |

### Fall radar module (24 GHz)

| Parameter | Value |
|---|---|
| Radar frequency | 24 GHz |
| Fall sensitivity | 0–9, default 6 |
| Fall alarm time | 0–9, representing 1–10 min |
| Fall alarm states | 0 suspected, 1 fall, 2 no fall |
| Dwell alarm | 4 stages: 5, 10, 30, 60 min |
| Hardware outputs | S1 presence, S2 motion, 3.3 V logic |
| Data output | UART, 115200 bps, 8N1, 3.3 V TTL |

## Summary

The fall detection radar watches a ceiling-mounted area of about 3 m × 2 m for a collapse in target height and for a person who stops moving. It publishes a fall alarm with the position where it happened, and a dwell alarm after a configurable period of stillness, together with presence, movement level, and the height distribution of the target.

Mount it straight down, directly above where a person would fall, at 2.8 m or less. Set the fall duration to the shortest value that does not alarm on stumbles, and the dwell duration to the longest stillness that is normal for that room. Keep the fall height threshold high enough that bending over does not trigger it.

This is a sensing device, not a medical alarm. It publishes to your broker and nothing else; calling anyone, lighting anything, and escalating to a person are your system's job. Used alongside human checks rather than instead of them, it covers the one room where a camera cannot go.
