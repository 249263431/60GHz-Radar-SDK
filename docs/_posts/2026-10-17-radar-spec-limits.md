---
layout: post
title: "Three mmWave Radar Specs That Are Almost Always Exaggerated"
date: 2026-10-17 09:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, specification, detection angle, detection range, speed, procurement]
---

<!-- filename: 2026-10-17-radar-spec-limits.md -->

Anyone who has compared millimeter wave radar quotations has seen the same thing. A spreadsheet arrives with columns for detection angle, detection distance, and speed, and some rows look far better than the rest. Angles of 130° and 140°. Distances of 8 m and 10 m. Speed described only as "fast".

Most of those numbers are not achievable, and the reason is not commercial. They are blocked by antenna physics, transmit power, and processing capacity. This article sets out the practical limits for home and building millimeter wave radar, explains why each one cannot be moved by better algorithms, and gives a short checklist for reading a spec sheet.

## Why Inflated Specs Hurt Everyone

An exaggerated specification is usually not written to mislead the buyer. It is written to exclude competitors. If the tender requires 140° of coverage, every honest supplier is disqualified before the comparison even starts.

The consequences land on all three parties:

- **The buyer** receives hardware that does not meet the number it was purchased against, and has no recourse at acceptance because the specification was never achievable in the first place.
- **The honest supplier** is filtered out by a threshold it could not meet without lying.
- **The vendor who signed the inflated specification** ships a product that underperforms on site, and carries the warranty and reputation cost.

A spec sheet is a contract. Numbers that no hardware can reach do not protect the buyer; they only move the failure to a later date.

## Limit 1: Detection Angle

A horizontal detection angle of **130° or 140°** is common in tender documents. For a home or building mmWave radar it is not achievable.

A mmWave radar antenna — whether etched on a PCB or built as an AiP module — transmits into a bounded spatial region and listens for the return. Outside that region the signal is too weak to be useful. The geometry is set by the antenna, not by software.

The practical figures:

| Measure | Value |
|---|---|
| Maximum angle per side | ~60° |
| Absolute maximum total angle | ~120° |
| Usable total angle | ~100° |

At 120° the mmWave radar is already working at the edge of the antenna pattern, where the signal is weak and the detection becomes unreliable. In practice, the usable angle is closer to **100°**. Anything beyond that is either a specially designed antenna for a specific application, or a number that was written down rather than measured.

This applies to general chip-based solutions and PCB-based products, which is what nearly all home mmWave radars are. A finished home product does not exceed these figures.

**This is an antenna limit, not an algorithm limit.** No amount of signal processing creates energy that the antenna never radiated in that direction. A specification of 130° or 140° cannot be met by any supplier using standard hardware, which is precisely why writing it is an effective way to remove the competition — and precisely why it should not be written.

## Limit 2: Detection Distance

A detection distance of **8 m or 10 m** appears regularly in comparison tables. For a home mmWave radar, the reliable figure is around **6 to 7 m**.

The reason is transmit power. Home mmWave radar antennas transmit at low power — a few dBm — because the product is meant to radiate as little as possible into a living space. That power sets the range at which a human target still returns a usable signal.

Increasing the range means increasing the power, and that creates a second problem. More power brings in more of everything else in the room: noise, clutter, and weak returns from distant objects. The mmWave radar then has to filter the clutter out while keeping the person, and those two requirements pull against each other. Push the range far enough and the mmWave radar either loses the person or keeps the clutter.

This has been measured repeatedly across many rooms. The consistent result is that **6 to 7 m** is where a home mmWave radar reliably detects a person, and beyond that the margin disappears.

A distance of 8 m or 10 m is a theoretical figure. It may describe what the radio can do in an empty anechoic chamber. It does not describe what the product does in a room with furniture, a fan, and a curtain. If a product genuinely detected a person reliably at 10 m in a real room, its anti-interference performance would have to be extremely poor, because it would be reporting everything it received.

## Limit 3: Speed

Speed is often listed as "fast" or omitted entirely. For a home mmWave radar detecting a person, the honest figure is that it handles movement up to roughly **3 m/s**.

A home mmWave radar does far more than measure speed. It has to run the detection chain, extract targets, and perform point cloud and posture analysis on every frame. All of that computation takes time, and it limits how fast a moving target can be tracked reliably.

A speed-measuring radar has a simpler job: it measures speed and nothing else, so it can be very fast. A presence or fall radar also has to decide what the target is doing — standing, sitting, lying, falling — and that judgement is what caps the usable speed.

So a home mmWave radar is not the right instrument for tracking a fast-moving object, and any specification implying otherwise is describing a different product.

## Why These Are Hardware Limits

The three limits share a structure. Each is fixed before any algorithm runs.

| Parameter | Fixed by | Can an algorithm improve it? |
|---|---|---|
| Detection angle | Antenna size and array geometry | No — the energy was never radiated in that direction |
| Detection distance | Transmit power and signal-to-noise ratio | Partly, but only by trading away anti-interference |
| Speed | Frame rate and processing load | No — the computation has to happen before the result exists |

The angle is set when the antenna is laid out. The distance is set when the transmit power is chosen. The speed is set when the processing budget per frame is allocated. By the time the detection chain receives a frame, all three are already decided.

This is why a supplier cannot sign a specification of 140°, 10 m, and high speed and then engineer around it later. The hardware does not support it, and no firmware release changes an antenna.

## How to Evaluate a Spec Sheet

Six checks, in order of how quickly they eliminate a misleading claim.

1. **Check the angle against 120°.** Anything above 120° total is beyond standard antenna geometry. Anything above 130° is not achievable with a general-purpose mmWave radar.
2. **Check the distance against 7 m.** Ask what the test environment was. A distance quoted without a room description, an installation height, and a target description is a theoretical number.
3. **Check the speed unit.** If the specification says "fast" instead of a figure in m/s, ask for the figure. For home posture detection, expect something in the region of 3 m/s.
4. **Ask for the test conditions, not just the number.** Installation height, room size, target posture, and interference sources present. Two mmWave radars with the same quoted range can behave completely differently once these are stated.
5. **Ask for a recorded test in the target scene.** A short video of the mmWave radar running in a room of the size and shape you intend to deploy is worth more than any table.
6. **Write the acceptance criteria as scenarios, not numbers.** "Detects a lying person within 3 s in a 2 m × 3 m bathroom with the door closed" is verifiable. "10 m detection range" is not.

The last point matters most. A scenario-based acceptance criterion can be tested on delivery. A headline number usually cannot.

## What We Publish Instead

The specifications we publish are the values we have measured, stated with the conditions they were measured under.

- Detection angles are given as usable figures, not antenna-pattern extremes.
- Distances are given for the scene the product is specified for, with the installation height that produces them.
- Speeds are given as figures in m/s, with the posture analysis they assume.
- Where a value depends on the scene — and most of them do — the article that covers it says so.

This costs us some rows in a comparison table. It also means that a mmWave radar bought against our specification behaves the way the specification said it would, which is the only outcome a procurement process is supposed to produce.

## Summary

Three numbers in a millimeter wave radar specification are almost always inflated, and all three are fixed by hardware rather than by software.

The usable detection angle is around **100°**, with **120°** as the absolute maximum of a standard antenna. The reliable detection distance for a home mmWave radar is **6 to 7 m**, because the transmit power is deliberately low and raising it destroys anti-interference. The usable speed for posture detection is around **3 m/s**, because the processing budget per frame is spent on deciding what the target is doing.

None of these can be improved by a better algorithm, because the antenna, the power, and the frame budget are all decided before the algorithm receives a frame. A specification that asks for 140°, 10 m, and high speed is not asking for a better product. It is asking every honest supplier to disqualify itself.

Read a spec sheet for the test conditions behind each number. If the conditions are missing, the number is decoration.
