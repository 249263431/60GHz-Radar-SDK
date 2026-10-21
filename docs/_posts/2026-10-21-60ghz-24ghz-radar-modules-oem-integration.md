---
layout: post
title: "60 GHz & 24 GHz Radar Modules: Specifications for OEM Integration (Modular-Approval Ready)"
date: 2026-10-21 09:00:00 +0800
categories: [guide]
tags: [mmwave, radar, module, oem]
---

We supply mmWave radar **modules** to system integrators and OEMs who build their own products. This page is the at-a-glance specification for our three production modules: **presence** (24 GHz), **fall detection** (60 GHz), and **sleep / breathing monitoring** (60 GHz).

- All three are **bare modules** with a UART interface and 3.3 V TTL I/O.
- A finished **Wi-Fi + MQTT** variant of each function exists for customers who do not integrate firmware themselves. The MQTT path is broker-agnostic (you point it at your own broker; no third-party cloud or app is required, and OTA is served from your own web server).
- We act as a **component supplier**. If your product needs regulatory certification, see the [Certification support](#certification-support) section.

## Which module do you need?

| | **R24AVD1** | **R60AFD1-V** | **R60ABD1** |
|---|---|---|---|
| Primary sensing | Presence / occupancy | Fall-event detection | Sleep & breathing |
| Frequency band | 24 GHz | 60 GHz (61–62.5 GHz) | 60 GHz FMCW |
| Beam (H × V) | 90° × 60° | 100° × 100° | 40° × 40° |
| Typical range | Motion ≈ 12 m diameter area (≈8×12 m); static ≈ 6 m; sleep ≈ 3 m | Fall / static / motion radius ≈ 1.5 m (see note) | Respiration ≤ 1.5 m; presence / sleep ≤ 2.5 m |
| Interface | UART 3.3 V TTL + S1/S2 outputs | UART 3.3 V TTL 115200/8N1 + GP1/GP2 outputs | UART 3.3 V TTL 115200/8N1 + spare GPIO |
| Power | 3.3 V I/O (see datasheet for VCC) | 4.5–5.5 V (typ. 5.0 V), ≤100 mA avg / ≤300 mA peak | 4.5–5.5 V (typ. 5.0 V), ≥200 mA |
| Mounting | Ceiling, height ≤ 2.75 m | Ceiling, height 2.2–2.8 m | Above bed 0.8–1.0 m, tilt 30°–45° |
| Module size | — (see datasheet) | 35 × 31 × 7.5 mm | — (see datasheet) |

> **Note on R60AFD1-V range:** the fall-module datasheet and its quick-start guide report the detection radius slightly differently (datasheet: 1.5 m for motion/static/fall at 2.8 m mount; quick-start: motion ≈ 2.5 m, fall ≤ 1.5 m). Treat **1.5 m** as the conservative figure for fall and static-dwell, and confirm against your own ceiling height during evaluation.

## Two form factors, two protocols

This is the single most important distinction before you design:

- **Module** — only UART. It has no network. You parse binary frames (header `0x53 0x59`, tail `0x54 0x43`, 8-bit sum checksum, low byte) or read the GPIO outputs.
- **Finished radar** — Wi-Fi + MQTT (JSON over a broker). It is not designed to be wired to a microcontroller.

The full explanation and first-five-minutes setup are in [Two Architectures, Two Protocols: MQTT vs UART]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %}).

---

## R24AVD1 — 24 GHz Presence / Occupancy Module

A cost-effective presence and motion sensor for lighting, HVAC, security, and "is someone in the room" logic. Unaffected by temperature, humidity, light, or a completely motionless person.

**Key parameters**

| Parameter | Value |
|---|---|
| Beam | Horizontal 90°, vertical 60° |
| Motion detection (diameter) | ≈ 12 m (≈ 8 m × 12 m area) |
| Static / micro-motion detection | ≈ 6 m diameter |
| Sleep detection | ≈ 3 m diameter (≈ 3 m × 2 m area) |
| Mounting | Ceiling, height ≤ 2.75 m, horizontal tilt ≤ 3° |
| I/O level | 3.3 V TTL |
| Outputs | **S1**: high = occupied, low = unoccupied; **S2**: high = active, low = static; GP1–GP4 redefinable |
| Baud | 115200 bps, 8N1 (family default; confirm in protocol manual) |

**Reported data points (DP1–DP6)**

| DP | Meaning | Notes |
|---|---|---|
| DP1 | Occupied / unoccupied | unoccupied→occupied < 0.5 s; occupied→unoccupied 40 s |
| DP2 | Static / active | reports within 0.5 s of change |
| DP3 | Approaching / leaving / no lateral movement | 2 s cadence |
| DP4 | Body-movement amplitude 0–100 | 5 s cadence |
| DP5 | Sensitivity 1–10 (default 7) | 1 low … 10 high |
| DP6 | Scene mode | area / bathroom / hotel / bedroom / office / corridor |

**Sensitivity guidance (DP5):** high (7–10) for quiet, enclosed spaces; medium (4–6, default) for homes/hotels; low (1–3) for corridors/parking where curtain or plant sway would otherwise false-trigger.

**Scene / range presets (DP6):** max-area (12 m / 8 m / 6 m × …), office, hotel (default), living room, bedroom, area-detect, bathroom (1.8 m / 1.5 m).

**Installation & enclosure**

- Ceiling mount; aim the long side of the board toward the door.
- Enclosure: non-metal, flat (no curved face); keep **3–5 mm** between the antenna face and the inner shell surface.
- On the PCBA, the radar component must sit **≥ 1 mm** higher than surrounding parts.

**Known false-alarm sources (plan the install angle):** moving non-living objects — fans, pets near the radar, swinging metal curtains; and the internal motor of an air conditioner directly in the beam.

---

## R60AFD1-V — 60 GHz Fall-Detection Module

Designed for elderly-care and bathroom/kitchen monitoring. Detects a fall from posture and motion change, and raises a **static-dwell** alarm for someone who has not moved for a long time.

**Key parameters**

| Parameter | Value |
|---|---|
| Radar type | 60 GHz FMCW, 1T3R |
| TX frequency | 61–62.5 GHz |
| TX power | 8 dBm typ., 10 dBm max |
| Antenna gain | 8 dBi |
| Beam | Horizontal 100° × vertical 100° |
| Fall / static-dwell / motion radius | ≈ 1.5 m (see note above) |
| Power | 4.5–5.5 V (typ. 5.0 V); ≤100 mA avg, ≤300 mA peak; ripple ≤ 100 mV |
| Operating temp. | −20 to +70 °C (storage −40 to +85 °C) |
| Interface 1 | 5V, GND, RX, TX, **GP1 (fall alarm / clear)**, **GP2 (occupied / unoccupied)** |
| Interface 2 | 3V3, GND, SWC/SWD (programming), GP3–GP6 |
| UART | 3.3 V TTL, 115200 bps, 8N1 |
| Boot stabilization | ≥ 30 s after power-on |
| Size | 35 × 31 × 7.5 mm; 2.0 mm pitch dual-row headers (2×3 + 2×4) |

**Why integrators like this part**

- **GPIO outputs need no protocol parsing.** GP1 goes high on fall, GP2 goes high when someone is present. Wire them straight to an MCU input, a buzzer via a transistor, or an opto-isolated PLC input.
- **UART protocol is generic (SIP-S V1.0).** No Tuya binding is required to use the module.

**Reported data points (DP1–DP8)**

| DP | Meaning |
|---|---|
| DP1 | Occupied / unoccupied |
| DP2 | Static / active |
| DP3 | Body-movement amplitude 0–100 |
| DP4 | Fall alarm |
| DP5 | Static-dwell alarm (default after 5 min stillness) |
| DP6 | Install-height setting (default 2.3–2.4 m, adjustable 2.2–2.8 m) |
| DP7 | Fall-alarm on/off |
| DP8 | Static-dwell alarm on/off |

**Timing you can tune**

- Fall report: default **15 s**, adjustable **5–180 s**. Use < 30 s only for demos; for production use **≥ 60 s** to suppress false trips.
- Static-dwell: default **300 s (5 min)**; for production use **≥ 30 min**.

---

## R60ABD1 — 60 GHz Sleep & Breathing Module

Non-contact respiration, heart-rate, and sleep-staging for beds. No camera, no image, all processing local — well suited to privacy-sensitive care and wellness products.

**Key parameters**

| Parameter | Value |
|---|---|
| Radar type | 60 GHz FMCW |
| Respiration detection | ≤ 1.5 m |
| Presence / sleep detection | ≤ 2.5 m |
| Beam | Horizontal 40° × vertical 40° |
| Power | 4.5–5.5 V (typ. 5.0 V); ≥ 200 mA; ripple ≤ 100 mV |
| Interface 1 | 5V, GND, RX, TX, GP1, GP2 (spare) |
| Interface 2 | 3V3, GND, SL, SD, GP3–GP6 (spare) |
| UART | 3.3 V TTL, 115200 bps, 8N1 |
| Mounting | Above bed 0.8–1.0 m; tilt 30°–45° (recommended 45°) |

**Frame format (shared with the family)**

```
Header    Ctrl  Cmd  Len     Data    Checksum  Tail
0x53 0x59 1B    1B   2B      nB      1B        0x54 0x43
```

Checksum = low 8 bits of (header + ctrl + cmd + len + data). Example query for presence: `53 59 80 81 00 01 0F BD 54 43`.

**Reported data (DP1–DP20, summary)**

- Presence / motion / distance / body-movement / orientation (DP1–DP5)
- Heart rate (50–120) + waveform, respiration (0–45) + waveform, respiration status (DP6–DP10)
- In-bed / out-of-bed, sleep stage (awake/light/deep), stage durations, sleep score 0–100, sleep summary, sleep-quality rating (DP11–DP16)
- Anomaly alarms: under/over-sleep, prolonged absence, struggle, nobody-timer (DP17–DP20)

**Enclosure / RF notes**

- Antenna-to-shell distance **2–5 mm**, non-metal, flat face.
- PCBA: radar taller than surrounding parts by **≥ 1 mm**.
- Avoid metal/electrolyte in front of the antenna; keep the beam off the AC indoor unit motor.

---

## Common integration notes (all modules)

These are the things that actually cause returns and field failures:

1. **Power quality.** 4.5–5.5 V, ripple **≤ 100 mV**, good grounding. Weak or noisy supplies are the #1 cause of unstable detection.
2. **Boot time.** Allow **≥ 30 s** stabilization after power-on before trusting output.
3. **UART level.** 3.3 V TTL only. Do not feed 5 V logic or RS-232; a 5 V source into an I/O pin can destroy the module.
4. **GPIO drive.** Outputs are weak (mA-level). Drive a buzzer/relay/LED through a transistor or opto-isolator, never directly.
5. **False-alarm reality.** Millimeter-wave sensing has an inherent false-alarm probability. Moving non-living objects (fans, curtains, flowing liquid) can trip it; a person lying on their side/back may be missed because the chest micro-motion is not facing the radar. **Design redundancy and fail-safe behavior into your product.**
6. **Not a medical device.** See the disclaimer below.

## Certification support

We support integrators building certified end products:

- We provide the **module RF characteristics** (band, TX power, antenna gain), **RF test reports**, **antenna integration guidelines** (keep-out, ground plane, shell material/distance), and **labeling conditions** (for example, the "Contains FCC ID: XXXX" requirement where applicable).
- If you reference our module's own modular approval in your filing, we supply the grant number and test data. If the module is not yet granted, we provide the test data and integration conditions so your certifier can proceed.
- We are **not** a certification body; we cooperate with your certifier. Tell us your target market (FCC / CE-RED / other) and we will send the relevant package.

## Where to start

- **Evaluation units:** on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
- **Volume orders, customization, and certification support:** contact us by email (see store page).

Further reading: [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}) and [Sleep radar guide]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}).

## Disclaimer

These modules are **not medical devices**. They are supplied for development, research, and wellness/reference use only. They must **not** be used as life-safety or emergency-alarm systems without redundant, fail-safe design, and must not be used for medical diagnosis or treatment. Millimeter-wave sensing has an inherent false-alarm probability; the user is responsible for all integration and application decisions.
