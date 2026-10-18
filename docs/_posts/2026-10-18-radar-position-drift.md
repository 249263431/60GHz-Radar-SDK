---
layout: post
title: 'Why Does a mmWave Radar''s "Position" Drift? Coordinate Jitter in Elderly Care and Whole-Home Sensing'
date: 2026-10-18 06:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, position drift, coordinate jitter, elderly monitoring, whole-home sensing, angle of arrival, sensor fusion, noise]
---

<!-- filename: 2026-10-18-radar-position-drift.md -->

If you have ever watched the raw position readout from a 60 GHz mmWave radar — for example while building an elderly-monitoring or whole-home sensing system — you may have noticed something strange. The coordinates of a person seem to "drift." Someone standing 10 cm from the edge of the bed may appear to be lying in it. Someone standing outside a window may be reported as inside. The system toggles between "in bed" and "got up" even when the person only shifted a little.

This article explains why that happens, where the jitter comes from, and how a well-designed radar deals with it. The short version: the "point" a radar outputs is not a fixed coordinate. It is a value that trembles every second. Understanding that is the first step to using radar correctly for care.

## How a radar measures position

A mmWave radar does not see a person the way a camera does. It senses distance and motion with radio waves, and it samples continuously.

- It splits each second into at least 10 slices, each 0.1 s.
- Each 0.1 s slice is divided again into 32 to 64 sub-samples — the "chirps" the chip emits.
- The radar averages those 32–64 sub-samples to estimate, for one frame, whether the target is approaching or receding and how large the motion amplitude is.
- It then aggregates frames over time, locally or up to the cloud, to build a trajectory.

So the position you see is an averaged, time-aggregated statistic — not a single clean measurement.

## Why the value drifts: noise

Two kinds of noise make each frame's distance value inconsistent.

**Power and chip noise.** Battery ripple and the chip's own noise floor mean the per-frame distance is never exactly the same twice.

**Taking the maximum.** To stay robust against noise, many implementations pick the peak or maximum value inside a window. The side effect is that, instead of centering on the true distance, the reported value swings back and forth by a few centimeters around it. At a true distance of 2 m, you may see it wander by several centimeters.

A good analogy is a laser toy "shooting" at a person: tut-tut-tut, you keep aiming at the same spot, but the toy itself vibrates, so each dot lands a little differently. The reflections come back scattered, and the computed position spreads into a patch instead of a single point.

## The angle drifts too

It is not only distance. The left–right angle (angle of arrival) and the elevation angle are also affected by phase mismatch between the transmit and receive antennas and by vibration.

Here is the catch: the farther the person is, the more a tiny angular error is magnified into a large lateral offset. That is why someone standing 10 cm from the bed edge can appear to drift into the bed when they are a bit farther away — the angular error, multiplied by distance, lands them "inside."

The height dimension is relatively more stable, but it still suffers from transmit/receive inconsistency.

## Why the drift matters

This is not cosmetic. Wrong coordinates directly break the features people buy radar for:

- **False bed-entry / get-up detection.** If the point drifts across the bed boundary, the system thinks the person got in or out of bed when they only moved slightly.
- **False region alarms.** Someone shown as inside a window when they are outside can trigger fall or activity false alarms.
- For any application built on posture recognition or zone triggers, a jittery single point is a hard limitation.

## How we fix it: multi-dimensional fusion

The key idea is simple: a single point is unreliable, so stop trusting a single point. Add more dimensions and let them constrain each other.

- **Height dimension** — a person is taller than furniture; height helps separate a standing human from a chair or a wall reflection.
- **Projection area dimension** — lying, sitting, and standing produce different silhouette areas.
- **Time dimension** — trajectory continuity, speed, and state history smooth out per-frame jitter.
- **Phase algorithms** correct the left–right and elevation angles.

The result: a good radar does not make the point "more accurate" in isolation. It uses more dimensions to *constrain* the trembling point. What the radar finally outputs to the outside world is still, at its core, that trembling thing — but wrapped in enough context that the application sees a stable, sensible state.

## Practical takeaways for developers

If you build on top of a radar SDK:

- **Do not consume the raw single-frame (x, y) coordinate for critical decisions.** Use the SDK's already-fused outputs: posture, zone, trajectory.
- **Add hysteresis and filtering.** A buffer zone around bed or zone boundaries prevents flip-flopping from tiny drifts.
- **Prefer the fused results over the bare point.** Our SDK already performs the multi-dimensional fusion; rely on the posture and zone events rather than the raw coordinate.
- See also: [Sleep radar installation position](https://249263431.github.io/60GHz-Radar-SDK/guide/2026/10/16/sleep-radar-installation-position.html) and [MQTT vs UART architecture](https://249263431.github.io/60GHz-Radar-SDK/fundamentals/2026/10/18/radar-architecture-mqtt-vs-uart.html).

## Wrapping up

Position drift is an inherent phenomenon of physics plus sampling — not a bug you can fully eliminate. The right response is not to chase a perfect point, but to understand it and design around it. Once you treat the radar's output as a trembling estimate and fuse multiple dimensions, elderly monitoring and whole-home sensing become stable and trustworthy.
