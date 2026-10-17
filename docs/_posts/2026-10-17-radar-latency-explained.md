---
layout: post
title: "Why mmWave Radar Takes Seconds to Respond"
date: 2026-10-17 09:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, latency, frame, averaging, false alarm, detection rate, breathing, fall detection]
---

<!-- filename: 2026-10-17-radar-latency-explained.md -->

A common question from new users is why the radar feels slow. Breathing rate takes a while to settle. A fall is not reported the instant it happens. Heart rate seems to lag behind what the person is feeling. The suspicion is usually that the sensor is underpowered or the signal is weak.

The opposite is closer to the truth. The delay is the price of not reporting things that did not happen, and it is built into how millimeter wave radar processes a signal. This article explains where the delay comes from, which measurements are genuinely fast, and which are slow by design.

## The Trade-off Behind Every Delay

Every millimeter wave radar faces the same choice on every frame: report now, or confirm first.

Report immediately and the reporting rate approaches one hundred percent — nothing is missed. It also means every shower, every squat to pick something up, and every curtain movement becomes an alarm. Report only after confirmation and the false alarm rate collapses, at the cost of a few seconds of latency.

Neither extreme is a product. A sensor that reports everything is unusable because nobody trusts it, and a sensor that reports nothing is unusable because it reports nothing. The delay you notice is the interval the radar spends deciding which side of that line an event falls on.

So latency is not a defect to be engineered away. It is the setting that decides how many false alarms you are willing to accept in exchange for how quickly a real event is published.

## How a Second Is Built

The delay starts with how the radar divides time.

A second is split into ten intervals of **0.1 s**. Each of those is split again into several sub-frames. The radar does not evaluate a single sub-frame in isolation — it averages across the sub-frames inside one 0.1 s window, and then across the 0.1 s windows inside a second.

Within each window the radar computes:

- The average velocity of the target
- The average movement amplitude
- How many frames show the target approaching
- How many show it receding
- How many show large movement, and how many show small movement

Combining the 0.1 s windows is what produces a height change, a breathing amplitude, or a heart rate amplitude. Every one of those values is an average, not a measurement of one instant. An average needs several samples before it means anything, and that requirement is the origin of the delay.

## Why Frames Drift

The radar transmits and receives continuously. It is closer to a machine gun firing frames than a camera taking one picture at a time, and that has a consequence: no two frames are identical.

Because the transmission is continuous and the target is moving, each frame captures the target at a slightly different phase. The result is a small drift between frames that is not signal and not noise — it is the geometry of sampling a moving object.

To keep the output stable, the radar does not trust a single frame. It recomputes across many frames and publishes only what survives the recomputation. That repetition is the second source of delay, and it is unavoidable: a value computed from one frame would jump around, and a value that jumps around is worse than a value that arrives late.

## Why a Fall Takes Seconds

A fall is the clearest example of the trade-off, because the raw signal is unambiguous and the decision is not.

The radar sees the target collapse: a tall reflection at chest height becomes a wide, low reflection on the floor, within about a second. That alone is enough to raise an alarm, and raising it immediately would catch every fall.

It would also catch a person crouching to pick something up, a person sitting on the edge of the bathtub, and a person reaching for a towel on the floor. All of those produce the same collapse.

So the radar confirms before publishing. It checks whether the height change is sustained, whether the target stays low, and whether the posture persists across repeated detections. Only a target that is still down several seconds later is reported.

The finished unit applies fixed rules on top: a body held below **70 cm** in a static posture raises the alarm after **30 seconds**, and a person who stays completely still raises it after **5 minutes**. The configurable fall duration covers everything in between.

That is why a fall takes seconds. The radar already knows something happened; it is spending those seconds making sure it happened to a person and not to someone bending over.

## Why Breathing Takes Longer

Breathing has the same structure, plus one extra problem: the signal has natural gaps.

A human breath is not a smooth wave. It is an inhale, an exhale, and a pause, and the pause is the difficulty. During the pause there is no chest movement to measure, so the radar has to decide whether the person has stopped breathing or is simply between breaths.

A single pause is not enough information. The radar has to observe several complete breath cycles — inhale, exhale, pause — before it can establish a rate, and a full cycle takes several seconds. Only after seeing the same rhythm repeat does the published value settle.

This is why the breathing value arrives frequently but stabilises slowly. The radar publishes a number every few seconds from the very first frame, and that number keeps moving until enough cycles have been observed. A rate computed from two breaths is not a rate.

Heart rate follows the same logic, for the same reason: it is a rhythm with gaps, and a rhythm needs repetition before it means anything.

## What Is Actually Fast

Not everything is delayed, and knowing which values are fast is useful when designing a system around the radar.

| Data | Speed | Why |
|---|---|---|
| Body movement amplitude | Near real time, updates every second | A magnitude, not a rhythm — no averaging across cycles needed |
| Energy level | Near real time | The size of a change is measurable in a single frame |
| Human position | Within 1–2 seconds | A coordinate, not a pattern |
| Presence | Within 0.5 s of a person appearing | A state change, not a measurement |
| Motion status | Within 0.5 s | Same |
| Breathing rate | Several seconds to stabilise | Requires several full breath cycles |
| Heart rate | Several seconds to stabilise | Same |
| Fall alarm | Seconds, by design | Requires sustained-posture confirmation |
| Dwell alarm | Minutes, by design | Requires sustained stillness |
| Sleep stage | Every 10 minutes | Evaluated once per window |

The pattern is simple. Anything that measures **how much** is fast. Anything that measures **a rhythm** or **a decision** is slow, because both require repetition before they can be trusted.

## How to Work With the Delay

The latency is fixed by the detection chain, but a system can be designed around it.

1. **Do not treat a single reading as final.** Breathing and heart rate values move for several seconds before settling. Average them on your side, or display them with a settling indicator.
2. **Separate fast signals from slow ones.** Use presence and movement amplitude for anything that must feel instant — lighting, extractor fans, occupancy displays. Use breathing, heart rate, and sleep data only for trends and history, where a few seconds of delay is invisible.
3. **Set the fall duration to the event, not to the fastest possible reaction.** A short duration catches more real events and produces more false ones. A long duration does the opposite. The right value depends on how much a false alarm costs in your deployment.
4. **Design the escalation path around the dwell alarm, not the fall alarm.** The fall alarm is seconds. The dwell alarm is minutes, and it is the one that covers the person who does not fall but stops moving.
5. **Expect the sleep stage to move in steps.** It is evaluated once per ten-minute window, so it changes in steps rather than continuously. That is not lag; that is the reporting interval.

## Summary

Millimeter wave radar divides a second into ten intervals of 0.1 s, averages across sub-frames and across intervals, and recomputes across many frames to remove drift. Every published value is the product of that averaging, and averaging takes time.

The delay is not a limitation of the sensor. It is the setting that decides how many false alarms you accept in exchange for how fast a real event is published. A radar that reported everything instantly would be useless, because nobody would believe it.

Movement amplitude, energy, and position are near real time, because they measure a magnitude. Breathing rate, heart rate, fall alarms, and dwell alarms take seconds to minutes, because they measure a rhythm or require a confirmed decision.

Design your system with that split in mind: fast values for anything that must respond immediately, slow values for anything that only needs to be right.
