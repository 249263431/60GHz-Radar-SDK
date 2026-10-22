---
layout: post
title: "How mmWave Radar Works: From FMCW to Breathing Detection"
date: 2026-10-17 08:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, fmcw, 60GHz, breathing detection, presence detection, privacy]
---

<!-- filename: 2026-10-17-how-mmwave-radar-works.md -->

Millimeter wave radar detects human presence, breathing, heart rate, body movement, and sleep stages without any physical contact. It does not need a camera, and the person does not need to wear anything. This makes it a practical sensing technology for homes, elderly care, hotels, and offices.

This article explains how millimeter wave radar works, starting from the FMCW principle, and why the 60GHz band is able to detect chest movement smaller than one millimeter. It also compares millimeter wave radar with PIR sensors, cameras, and wearable devices.

## What Is Millimeter Wave Radar

Millimeter wave refers to electromagnetic waves with a wavelength between **1 mm and 10 mm**, which corresponds to a frequency range of roughly **30 GHz to 300 GHz**. It is simply a segment of the electromagnetic spectrum, in the same way as infrared or visible light.

Millimeter wave has three useful properties for indoor sensing:

- Large bandwidth, which allows fine resolution
- Narrow beam, which allows accurate direction finding
- Good penetration through smoke, dust, and non-metallic materials

For indoor presence detection, sleep monitoring, and fall detection, **60 GHz** is the most widely validated and cost-effective choice. At 60 GHz the wavelength is approximately **5 mm**, which is short enough to build a compact antenna array and fine enough to resolve small targets.

## How mmWave Radar Works: The FMCW Principle

Most indoor millimeter wave sensors use **FMCW (Frequency Modulated Continuous Wave)**. Instead of sending a short pulse, the radar transmits a continuous signal whose frequency sweeps linearly up and down over time. The processing chain has five steps.

### 1. Transmit

The radar transmits a 60 GHz FMCW chirp through its antenna array. Because the wavelength is only about 5 mm, the antenna can be small enough to fit inside a ceiling-mounted housing.

### 2. Reflect

The wave hits the target and part of the energy is reflected back. The echo carries information about range, velocity, and angle. A human chest reflects the signal continuously, even when the person is completely still, because breathing moves the chest wall.

### 3. Receive

The received echo is mixed with the transmitted signal. The mixing produces a beat frequency that is proportional to the target range. The signal is then filtered and sampled by an ADC.

### 4. Digital Signal Processing

An FFT separates targets in the range domain, and Doppler and phase differences reveal velocity and micro-movement. This is the step that separates a real human from a moving curtain: a curtain has movement but no stable phase structure or breathing rhythm.

### 5. Output Application Data

The processed result is published as application data: presence status, body movement level, human distance, breathing rate, heart rate, sleep stage, or fall alarm.

## Why 60GHz Can Detect Breathing

Breathing moves the chest wall by **less than 1 mm**. Detecting this requires very fine range resolution and a stable phase reference, and this is where the 60 GHz band performs well.

The reasons:

- Wavelength of approximately **5 mm** gives centimeter-level range resolution
- Narrow, focused beam concentrates energy on the chest area
- High sensitivity to micro-motion compared with the 24 GHz band
- Small antenna size, which allows compact product design

For comparison, the 24 GHz band has a wavelength of about **12.5 mm**, which makes it less sensitive to micro-motion and requires a larger antenna. 24 GHz radar is well suited to presence detection over a wide area, while 60 GHz radar is the better choice when breathing, heart rate, or sleep stages are required.

## What mmWave Radar Can Detect

| Signal | What It Measures | Typical Range | Report Frequency |
|---|---|---|---|
| Human presence | Occupied / unoccupied | Up to several meters | On change |
| Motion status | Static / active | — | On change |
| Body movement | Movement level 0–100 | — | Every 1 s |
| Human distance | 0–65535 cm | Non-critical function | Every 2 s |
| Breathing rate | 0–45 breaths/min | ~1.5 m | Every 3 s |
| Heart rate | 50–120 bpm | ~1.5 m | Every 3 s |
| Sleep stage | Awake / light / deep | ~2.5 m | Every 10 min |
| Sleep score | 0–100 | — | On sleep end |
| Fall alarm | Fall detected | ~2.5 m | On change |
| Dwell alarm | Static dwell detected | ~2.5 m | On change |

### Body Movement Levels

The movement parameter is reported as a value from 0 to 100, with fixed meaning:

| Value | Meaning |
|---|---|
| 0% | No person in range |
| 1% | Still (breathing only, no limb movement) |
| 2%–30% | Micro movement (small head or limb motion) |
| 31%–60% | Walking or fast limb movement |
| 61%–100% | Running or large close-range movement |

### Sleep Stages and Sleep Score

While a person is in bed, the radar evaluates the sleep stage **every 10 minutes** and reports awake, light sleep, or deep sleep. When the sleep session ends, it reports a sleep score from 0 to 100:

| Score | Rating |
|---|---|
| 1–60 | Poor |
| 61–75 | Fair |
| 76–100 | Good |

The sleep report includes total sleep duration, awake / light / deep percentages, time out of bed, number of times out of bed, turn count, average breathing rate, average heart rate, and apnea count.

The radar also reports sleep anomalies:

- Sleep duration shorter than **4 hours** (insufficient sleep)
- Sleep duration longer than **12 hours** (excessive sleep)
- No presence detected for **30 minutes** while the person should be in bed

## mmWave Radar vs PIR, Cameras, and Wearables

| Technology | Detects a Stationary Person | Works in Darkness | Measures Breathing | Privacy Risk | Wearing Required |
|---|---|---|---|---|---|
| PIR sensor | No | Yes | No | None | No |
| Camera | Yes | Needs infrared | No | High | No |
| Wearable device | Yes | Yes | Yes | Low | Yes |
| mmWave radar | Yes | Yes | Yes | None | No |

A PIR sensor only reacts to movement, so a person sitting still or sleeping is invisible to it. A camera captures images, which raises privacy concerns in bedrooms and bathrooms and needs infrared illumination in the dark. A wearable device measures breathing well, but it must be worn and charged, and many users stop wearing it after a few weeks.

Millimeter wave radar has none of these limitations, and it produces no image data at all. The output is a set of numbers.

## What Penetrates and What Blocks the Signal

Millimeter wave passes through most non-metallic materials. It also reacts to any movement inside the detection area.

Penetrates without affecting detection:

- Clothing and blankets
- Curtains
- Thin wooden boards
- Glass

Causes interference or false alarms:

- Electric fans (moving blades)
- Air conditioners (internal motor)
- Curtains moving with wind
- Metal objects in front of the antenna
- Flowing water and water pipes
- Pets moving inside the detection area

**Note:** Do not aim the radar directly at an air conditioner, or install it facing the same direction as the air flow. The internal motor causes false detection.

## Where mmWave Radar Is Used

- **Home sleep monitoring**: nightly breathing, heart rate, and sleep score without wearing a device
- **Elderly care and nursing homes**: fall alarm, long absence from bed, reduced activity over several days, no cameras in rooms
- **Bathrooms**: low posture detection and fall alarm in wet environments where a camera is unacceptable
- **Hotels and offices**: presence-based lighting and HVAC control, occupancy statistics
- **Smart home platforms**: standard MQTT integration, including Home Assistant auto-discovery

## Common Misconceptions

| Misconception | Reality | Solution |
|---|---|---|
| "It is just a motion sensor" | It resolves chest displacement below 1 mm through phase analysis | Aim the main beam at the chest area |
| "60GHz and 24GHz are the same" | 60GHz is far more sensitive to micro-motion and needs a smaller antenna | Use 60GHz for breathing, heart rate, and sleep |
| "Higher sensitivity is always better" | High sensitivity picks up curtains, fans, and pets | Match sensitivity to room size and interference |
| "It cannot work through a blanket" | Non-metallic materials do not block the signal | Keep metal away from the antenna face |
| "A camera gives better data" | Cameras fail in darkness without infrared and create privacy risk | Use radar where privacy matters |

## Getting Reliable Data

Signal quality depends mostly on geometry. For a sleep monitoring radar:

1. Identify the main sleep area and the bed head position.
2. Mark a point **0.8–1 m above the bed surface** on the wall or ceiling.
3. Tilt the radar downward at **30°–45°**.
4. Make sure the main beam covers the chest area of the person lying on the bed.
5. Check the detection area for interference sources and remove them.
6. Fix the radar in place, power it on, and verify the data.

## Testing After Installation

1. Lie on the bed in a normal sleeping position.
2. Stay still for at least **10 minutes**.
3. Check that breathing rate and heart rate are reported.
4. Check that the sleep stage changes from awake to light sleep.
5. If the data is unstable, adjust the tilt angle or the installation height.

A simple test: confirm that the breathing waveform is visible in the app. If the waveform is flat or missing, the radar is not aimed at the chest correctly.

## Summary

Millimeter wave radar uses FMCW to transmit a sweeping 60 GHz signal, mixes the echo with the transmitted signal to obtain range, and uses FFT and phase analysis to extract micro-movement. At a wavelength of about 5 mm, it resolves chest displacement below 1 mm, which makes breathing rate, heart rate, and sleep stages measurable without contact.

Keep three points in mind: use the **60 GHz** band for breathing and sleep, keep the radar within **1.5 m** for breathing detection and **2.5 m** for sleep and presence detection, and keep fans, air conditioners, moving curtains, and metal out of the beam.

A radar that is correctly aimed produces stable data. If the numbers look unstable, check the installation position and angle first.
