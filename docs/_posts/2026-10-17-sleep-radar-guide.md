---
layout: post
title: "The Sleep Monitoring Radar: What It Measures, How to Install It, and How to Read the Data"
date: 2026-10-17 09:00:00 +0800
categories: [product]
tags: [sleep radar, 60ghz, breathing, heart rate, sleep stages, installation, mqtt]
---

<!-- filename: 2026-10-17-sleep-radar-guide.md -->

The sleep monitoring radar measures presence in bed, breathing rate, heart rate, body movement, sleep stages, and a sleep score, all without contact and without a camera. It is mounted once above or beside the bed, and it reports data continuously through the night.

This article covers everything about this radar: what each data point means, how a night is processed, where to install it, and how to read the output over MQTT or UART. It is written for the radar sold in this store, so all ranges, frequencies, and values are the ones this specific model uses.

**This is a consumer wellness device. It is not a medical device and does not diagnose, treat, or prevent any condition.**

## Two Variants, Two Protocols

This radar is available in two forms, and they use completely different protocols. Check which one you have before reading further.

| Variant | Interface | Protocol | Configuration |
|---|---|---|---|
| Sleep radar module | UART only | Frame protocol at 115200 bps | Wired, no network setup |
| Sleep radar | 2.4 GHz Wi-Fi + MQTT | JSON property protocol over MQTT topics | Web page for MQTT, Android app for Wi-Fi |

Both variants share the same sensing core, so the measured values mean the same thing. The ranges, the commands, and the reporting details differ, and every value in this article is marked with the variant it belongs to.

The **sleep radar module** is the bare sensing core for integrators who embed it in their own product; its output is described under Over UART. The **sleep radar** is the finished unit that joins your network; its output is described under Over Wi-Fi and MQTT.

## What This Radar Is

The **sleep radar** is a 60 GHz millimeter wave sensor in a housing of approximately 80 × 80 × 51.8 mm. It is designed for fixed installation at a bed: either side-wall mounted or mounted above the bed head and tilted downward.

The **sleep radar module** is the same sensing core without the housing and without the Wi-Fi section. It exposes a UART interface and is intended to be built into another product.

Both measure, through a blanket and in complete darkness:

- Whether a person is in bed or out of bed
- Breathing rate and breathing state
- Heart rate
- Body movement level
- Sleep stage: awake, light sleep, or deep sleep
- Time spent in each stage, and a sleep score at the end of the night

Neither measures who the person is, and neither records images, video, or audio. The sleep radar publishes numbers over Wi-Fi; the sleep radar module publishes them over a serial line.

## How It Sees You Through a Blanket

The radar transmits a 60 GHz FMCW signal and analyses the echo. Breathing moves the chest wall by less than one millimeter, and the 60 GHz band resolves displacements of that scale. A blanket is a non-metallic material, so the signal passes through it and reflects off the chest underneath.

The beam covers a horizontal angle of **40°** and a vertical angle of **40°**, so the radar must be aimed at the chest area rather than at the whole room. A full explanation of the FMCW principle is in the companion article <a href="{{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %}">How mmWave Radar Works</a>.

## What It Measures

Every value below is reported either when the state changes or at a fixed interval. The exact property names for Wi-Fi and UART are given in the section [Getting the Data Out](#getting-the-data-out).

### Presence in Bed

| State | Meaning |
|---|---|
| 0 | Out of bed |
| 1 | In bed |

The radar reports a person within **0.5 s** of detection. It takes about **40 s** of confirmed absence before it reports unoccupied, because the radar checks for micro-movement and breathing before declaring the area empty.

The in-bed state is entered as soon as the energy condition is met. The out-of-bed state is output about **30 s** after the person leaves.

### Motion Status

| Value | Meaning |
|---|---|
| 0 | No movement |
| 1 | Still (static) |
| 2 | Active |

State changes are reported within **0.5 s**.

### Body Movement Level

A single value from 0 to 100, reported **every 1 s**. The ranges have fixed meanings:

| Value | Meaning |
|---|---|
| 0 % | No person in range |
| 1 % | Still: breathing only, no limb movement |
| 2 %–30 % | Micro movement: small head or limb motion |
| 31 %–60 % | Walking or fast limb movement |
| 61 %–100 % | Running or large close-range movement |

This is the most useful value for tuning an installation. A correctly aimed radar reports values around 1 % while a person lies still.

### Breathing Rate

Reported **every 3 s**. The range is **0–45 breaths per minute** on the sleep radar module and **0–30** on the sleep radar.

The breathing state is reported separately when it changes:

| State | Meaning |
|---|---|
| Normal | 10 to 25 breaths per minute |
| High | Above 25 breaths per minute |
| Low | Below 10 breaths per minute |
| None | No person detected (default state) |

The breathing waveform is available but **switched off by default**. When enabled, it reports 5 values every second, each in the range 0–255. Each value is the real measurement plus 128, which keeps the curve centred on the display axis.

### Heart Rate

Reported **every 3 s**. The range is **50–120 bpm** on the sleep radar module and **60–120 bpm** on the sleep radar, which is the corrected range.

The heart rate state is reported separately: normal, high, low, or detecting. The heart rate waveform follows the same rule as the breathing waveform: 5 values per second, 0–255, real value plus 128, off by default.

### Human Distance and Position

Both are reported every 2 s. Distance is in centimetres, 0–65535. Position is given as x, y, and z coordinates.

Both are marked as **non-critical functions with limited accuracy** in the sleep radar module documentation. They are useful for debugging an installation, but they should not be used as the basis of a product feature.

### Sleep Stage

Reported **every 10 minutes** while the person is in bed.

| Value | Meaning |
|---|---|
| 0 | Deep sleep |
| 1 | Light sleep |
| 2 | Awake |
| 3 | None (out of bed) |

The stage is evaluated once per 10-minute window, so the value changes in steps rather than continuously. Note that some older versions of the documentation list these values in a different order; the order above is the one used by the sleep radar module protocol.

### Stage Durations

Three separate values, in minutes: awake duration, light sleep duration, and deep sleep duration. Each ranges from 0 to 65535 and is updated together with the 10-minute sleep stage report.

### Sleep Score and Sleep Rating

The sleep score is reported **when the sleep session ends**, as a value from **0 to 100**.

A valid score is only produced when the sleep duration is between **4 and 12 hours**. Outside that range the session is treated as an anomaly and no meaningful score is output.

The radar also reports a rating derived from the score:

| Score | Rating |
|---|---|
| 1–60 | Poor |
| 61–75 | Fair |
| 76–100 | Good |

### The Sleep Report

At the end of a sleep session the radar publishes a report of 11 fields. This is the main output most integrations use.

| Field | Range |
|---|---|
| Sleep score | 0–100 |
| Total sleep duration | 0–65535 minutes |
| Awake percentage | 0–100 |
| Light sleep percentage | 0–100 |
| Deep sleep percentage | 0–100 |
| Time out of bed | 0–255 |
| Number of times out of bed | 0–255 |
| Turn count | 0–255 |
| Average breathing rate | 0–25 |
| Average heart rate | 0–100 |
| Breathing pauses | 0–10 |

Breathing pauses are counted as a raw sensor output. The radar counts pauses in the breathing waveform; it does not interpret them, and no medical meaning should be attached to the number.

### Sleep Comprehensive Status

A second, shorter report published **every 10 minutes** while the person is in bed, with 8 fields:

| Field | Range |
|---|---|
| Presence | 1 = present, 0 = absent |
| Sleep stage | 3 = out of bed, 2 = awake, 1 = light, 0 = deep |
| Average breathing rate | 10-minute average |
| Average heart rate | 10-minute average |
| Turn count | Turns during light or deep sleep |
| Large movement proportion | 0–100 |
| Small movement proportion | 0–100 |
| Breathing pauses | Count within 10 minutes |

This report is the practical choice for a live dashboard, because it arrives every 10 minutes whether or not the stage changes.

### Sleep Anomalies

Reported when the state changes:

| Value | Meaning |
|---|---|
| 0 | Sleep duration shorter than 4 hours |
| 1 | Sleep duration longer than 12 hours |
| 2 | No presence detected for 30 minutes while in a sleep state |
| 3 | None |

The 30-minute rule is the one most useful in care settings: if the radar is in a sleep state and detects nobody for half an hour, it raises the anomaly.

### Struggle and No-Person Timing

Two further functions are available on the sleep radar module protocol. Both are switches that must be enabled before they report anything.

- **Abnormal struggle**: reports none, normal, or abnormal struggle. The sensitivity of the judgement can be set to low, medium (default), or high.
- **No-person timing**: reports none, normal, or abnormal. The threshold is configurable; note that the documented range differs between the setting command and the query command, so treat the setting range as the reliable one.

## Reporting Rhythm

A single table of every value and when it arrives:

| Data Point | Range | Reported |
|---|---|---|
| Presence in bed | 0 / 1 | On change |
| Motion status | 0 / 1 / 2 | On change |
| Body movement | 0–100 | Every 1 s |
| Human distance | 0–65535 cm | Every 2 s |
| Human position | x, y, z | Every 2 s |
| Breathing rate | 0–45 | Every 3 s |
| Breathing state | Normal / high / low / none | On change |
| Heart rate | 50–120 bpm | Every 3 s |
| Heart rate state | Normal / high / low / detecting | On change |
| Breathing waveform | 5 values, 0–255 | Every 1 s, off by default |
| Heart rate waveform | 5 values, 0–255 | Every 1 s, off by default |
| Sleep stage | 0–3 | Every 10 min while in bed |
| Stage durations | 0–65535 min | With the 10-minute report |
| Sleep score | 0–100 | On sleep end |
| Sleep rating | Poor / fair / good | On change |
| Sleep report | 11 fields | On sleep end |
| Sleep comprehensive status | 8 fields | Every 10 min |
| Sleep anomaly | 0–3 | On change |

## Installation

### Two Mounting Methods

**Side-wall mounting (sleep radar).** Mount the radar on the wall at the side of the bed, at a height of **1.8–2.2 m**, pointing at the bed. This is the standard method for the finished sleep radar and needs no tilt adjustment.

**Tilted mounting above the bed head (sleep radar module).** Mount the radar above the bed head, tilted downward at **30°–45°**, with **45° recommended**, and **0.8–1 m above the bed surface**. This is the method described in the sleep radar module documentation and gives the strongest breathing signal, because the beam hits the chest directly instead of at an angle.

Use the tilted method whenever the physical layout allows it.

### Detection Range

| Signal | Maximum Distance |
|---|---|
| Breathing rate | ~1.5 m |
| Sleep detection | ~2.5 m |
| Human presence | ~2.5 m |

If the bed is large, or the radar ends up further away than these distances, the breathing signal degrades first and disappears before the presence detection does.

### What Interferes

Avoid installing the radar where the beam reaches:

- Electric fans (moving blades cause false readings)
- Air conditioners (the internal motor causes false detection)
- Curtains that move with air flow
- Metal objects
- Water pipes or flowing water
- Pets that move around the bed

Do not aim the radar directly at an air conditioner, and do not install it facing the same direction as the air flow.

### What the Signal Passes Through

The signal is not blocked by:

- Clothing and blankets
- Curtains
- Thin wooden boards
- Glass

### Installation Steps

1. Identify the bed head and the position where the sleeper's chest rests.
2. For side-wall mounting, mark a point 1.8–2.2 m high on the wall at the side of the bed. For tilted mounting, mark a point 0.8–1 m above the bed surface, above the bed head.
3. Aim the radar so the main beam covers the chest area of a person lying on the bed.
4. Check the whole detection area for the interference sources listed above and remove or avoid them.
5. Fix the radar in place and power it.
6. Wait for the initialization to complete, then verify the data.

### Verifying the Installation

Four tests, taken from the sleep radar module documentation:

1. **Sleep stage.** Lie still on the bed for 10 minutes. The reported stage should change from awake to light sleep.
2. **In-bed detection.** Approach the bed at a speed of at least 0.7 m/s and stop the moment the state changes. The radar should report in-bed.
3. **Out-of-bed detection.** Leave the area, at least 3 m away with no interference. The radar should report out-of-bed within 5 minutes.
4. **Breathing.** Sit still within 1.5 m in front of the radar, breathe calmly for 3 minutes, then hold your breath for 30–40 seconds. The breathing value should drop to 0 and the radar should report a breathing-hold event.

If the breathing waveform is flat or missing during test 4, the radar is not aimed at the chest. Adjust the angle or the height before looking at anything else.

## How a Night Is Processed

Understanding this sequence explains most of the odd-looking data.

1. **Power on.** The radar initializes and reports that initialization is complete.
2. **Mode selection.** The radar runs in one of two modes. Sleep detection mode is the default and the mode to use. Real-time detection mode exists for quick testing only and does not produce a sleep report.
3. **In-bed.** When the energy condition is met, the radar reports in-bed and starts evaluating.
4. **Stage evaluation.** Every 10 minutes, the radar judges the stage and reports it, together with the updated stage durations and the comprehensive status.
5. **Out of bed.** About 30 seconds after the person leaves, the radar reports out-of-bed.
6. **Session end.** If the person does not return, the radar treats the session as finished and computes the statistics.
7. **Report.** The sleep score and the full sleep report are published. The report is generated once the area has been unoccupied for about 5 minutes.

Two consequences worth knowing:

- **Switching modes clears the data.** If you switch between real-time and sleep detection mode, the accumulated data is discarded and detection restarts from zero.
- **A short night produces no score.** Sessions shorter than 4 hours or longer than 12 hours are reported as anomalies instead.

## Getting the Data Out

### Over Wi-Fi and MQTT (sleep radar)

Two topics are used, with the model prefix for the sleep radar being `Radar60SP`:

- Report topic: `/Radar60SP/{DeviceID}/sys/property/post`
- Set topic: `/Radar60SP/{DeviceID}/sys/property/set`

Data is JSON. A typical report looks like:

```json
{"version": "1.0", "method": "post", "params": {"sleepStatus": "0"}}
```

The properties you will use most:

| Property | Meaning | Access |
|---|---|---|
| `sleepSwitch` | Sleep function on / off | get, set |
| `sleepStatus` | Sleep stage | post, get |
| `getIntoBed` | In bed / out of bed | post, get |
| `awakeDuration` | Awake minutes | post, get |
| `lightSleepDuration` | Light sleep minutes | post, get |
| `deepSleepDuration` | Deep sleep minutes | post, get |
| `sleepScore` | Sleep score | post, get |
| `sleepComprehensiveStatus` | 8-field status report | post, get |
| `sleepQualityAnalysis` | 11-field sleep report | post, get |
| `abnormalSleep` | Sleep anomaly | post, get |
| `breathSwitch` | Breathing function on / off | get, set |
| `breathValue` | Breathing rate | post |
| `breathInform` | Breathing state | post, get |
| `heartRateSwitch` | Heart rate function on / off | get, set |
| `heartRateValue` | Heart rate | post |
| `heartRateInform` | Heart rate state | post, get |
| `someoneExists` | Presence | post, get |
| `motionStatus` | Motion status | post, get |
| `movementSigns` | Body movement level | post, get |
| `reportMode` | 1 = sleep mode, 0 = real-time mode | get, set |

Values are reported as strings. Most properties support a query with `"method": "get"` and a setting with `"method": "set"`; the device answers on the report topic.

### Over UART (sleep radar module)

The sleep radar module exposes a UART interface at **115200 bps**, 8 data bits, 1 stop bit, no parity, 3.3 V TTL level.

The frame structure is:

```
Header   Control  Command  Length   Data     Checksum  Tail
0x53 0x59  1 Byte   1 Byte   2 Bytes  n Bytes  1 Byte    0x54 0x43
```

The checksum is the sum of the header, control byte, command byte, length bytes, and data, taking the low 8 bits.

Control byte `0x84` covers sleep monitoring. Within it, command `0x0C` returns the comprehensive status, `0x0D` returns the sleep report, `0x0E` returns the sleep anomaly, and `0x10` returns the rating. Query commands are the same values with the high bit set, for example `0x8D` to query the comprehensive status.

### With Home Assistant

Because the radar publishes standard MQTT, it can be added to Home Assistant as an MQTT sensor. The practical approach is to subscribe to the report topic and map the fields you need. The comprehensive status report, arriving every 10 minutes, is usually enough for a dashboard; the full sleep report is best stored for history.

## Where It Is Used

**Home sleep tracking.** One radar above the bed gives a nightly record of stages, durations, and score with nothing to wear and nothing to charge. This is the most common individual use.

**Care facilities and nursing homes.** A room without a camera still needs to know whether the occupant is in bed, how long they were out of bed, and whether the night was unusually short or long. The anomaly report covers the last case directly.

**Hotels and long-stay apartments.** Presence-based control of lighting and climate, with the sleep report available as an optional guest feature.

**Solution integrators.** The radar plus a broker plus a dashboard is a complete bed-side monitoring unit. The UART interface allows the same sensor to be embedded in a product without Wi-Fi.

## What It Cannot Do

This section matters as much as the features.

- **It is not a medical device.** It does not diagnose, treat, or prevent any condition, and its output must not be presented as a medical measurement.
- **It does not identify people.** With two people in one bed the signals combine, and the stage judgement reflects the combined movement. Use one radar per bed, not per room.
- **It cannot see through metal.** A metal bed frame, a metal headboard, or a metal lamp between the radar and the chest will degrade the signal.
- **It is sensitive to movement in the beam.** A fan, a curtain, or a pet produces readings that look like a person. Installation is not optional.
- **A wrong angle produces unusable data.** If the beam misses the chest, the breathing value will be absent or unstable long before the presence detection fails.
- **The score is a device index.** It is computed from duration, stage proportions, movement, and breathing. It is not a health assessment and should not be compared with any clinical scale.
- **Breathing pauses are a raw count.** The radar counts pauses in the waveform. It does not classify them, and no medical conclusion follows from the number.

## Troubleshooting

| Symptom | Possible Cause | Solution |
|---|---|---|
| No breathing value | Beam missing the chest | Adjust the angle or height; run the breathing test |
| | Metal object in the beam path | Remove or relocate the radar |
| | Person too far away | Move within 1.5 m for breathing detection |
| Breathing value unstable | Interference source in the beam | Remove fans, curtains, or pets from the area |
| Always reports out of bed | Radar too high or too far | Lower to 0.8–1 m above the bed, or use side-wall mounting at 1.8–2.2 m |
| Sleep stage never changes | Person not still for 10 minutes | Wait a full 10-minute window |
| | Radar in real-time mode | Set `reportMode` to 1 (sleep mode) |
| No sleep score | Session shorter than 4 h or longer than 12 h | Check the anomaly report instead |
| Waveform empty | Waveform reporting is off by default | Enable the waveform switch |
| Data cleared unexpectedly | Mode was switched | Do not switch modes during a session |
| False presence at night | Moving object in the beam | Remove the interference source |

## Specifications

| Parameter | Value |
|---|---|
| Radar frequency | 60 GHz |
| Beam angle | 40° horizontal, 40° vertical |
| Housing size | 80 × 80 × 51.8 mm |
| Supply voltage | 4.5–5.5 V |
| Operating current | 200 mA minimum |
| Power consumption | 0.8 W minimum |
| Wireless | 2.4 GHz Wi-Fi |
| Data output | MQTT over Wi-Fi (JSON), UART |
| UART | 115200 bps, 8N1, 3.3 V TTL |
| Side-wall mounting height | 1.8–2.2 m |
| Tilted mounting | 30°–45° downward, 0.8–1 m above the bed |
| Breathing detection range | ~1.5 m |
| Sleep and presence range | ~2.5 m |
| Sleep stage interval | 10 minutes |
| Sleep score range | 0–100 |

## Summary

The sleep monitoring radar reports presence in bed, breathing rate and state, heart rate, body movement, sleep stage, stage durations, and a sleep score, at fixed intervals from every second to every 10 minutes, with the full report published when the session ends.

Mount it above the bed head tilted 30°–45° at 0.8–1 m above the bed surface, or side-wall mounted at 1.8–2.2 m. Keep it within 1.5 m for breathing and 2.5 m for sleep and presence. Keep fans, air conditioners, moving curtains, metal, and pets out of the beam.

It is a consumer wellness device that records what happens at the bed side. It does not diagnose anything, and the quality of its data depends almost entirely on where and how it is installed.
