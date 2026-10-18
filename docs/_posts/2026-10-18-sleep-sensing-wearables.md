---
layout: post
title: "Sleep Sensing in Wearables: Principles, Vendor Approaches, and Where mmWave Radar Fits"
date: 2026-10-18 08:00:00 +0800
categories: [fundamentals]
tags: [wearables, sleep tracking, mmwave radar, PPG, accelerometer]
---
<!-- filename: 2026-10-18-sleep-sensing-wearables.md -->

Sleep tracking has quietly become a baseline feature across modern wearables. But how do these devices actually know you're asleep — and how accurate is that claim? This article walks through the three measurement principles behind consumer sleep sensing, how the major vendors implement them, the problems that still aren't solved, and where mmWave radar offers a fundamentally different angle.

## The three core measurement principles

**1. Accelerometer (motion).** An accelerometer measures acceleration, which in practice means it senses changes in your motion. The amplitude and duration of movement within a given time tell the device whether you're active or still. Long stretches without movement are interpreted as sleep.

**2. Heart rate / optical sensing (PPG).** Most wearables shine a light into your skin. As blood flows, it changes how much light is reflected back; each pulse of blood corresponds to one heartbeat. From the heart-rate stream the device can derive heart-rate variability (HRV) and, by watching how your heart rate speeds up and slows down, estimate your breathing amplitude.

**3. Acoustic sensing.** A microphone can pick up snoring, steady breathing, and wheezing. Listening to sound gives clues about breathing quality and potential interruptions.

## How wearables are built around these

Vendors weight these signals differently:

- **Accelerometer-first — Xiaomi, Apple.** These rely mostly on motion: how long it's been since your last movement, and how large those movements were. They average this into a sleep duration. (You may not keep your arm perfectly still while asleep, so the accelerometer essentially computes an average.)
- **Heart-rate-first — Huawei, VIVO.** They watch HR/HRV changes to derive breathing amplitude and produce a sleep-quality score.
- **Fusion — Samsung, Honor, Garmin, OPPO.** These combine accelerometer and heart rate, and sometimes sound, into a single picture.

Everyone is converging toward fusion, and devices increasingly pair with phones. A same-brand watch knows when you're actively using your phone, so it won't mistake that for sleep.

## The hard problems

Even with fusion, consumer sleep tracking hits walls:

- **Light vs. deep sleep is the core difficulty.** The pattern that separates them isn't obvious in wrist-scale signals, so staging is probabilistic.
- **REM (rapid eye movement) is estimated, not measured.** Devices infer it rather than observe it directly.
- **Form factor matters.** Rings such as Oura press tightly against the skin and sit closer to a blood vessel, giving a cleaner heart-rate signal. Watches can be worn loose or tight; pressure can blanch the skin and disrupt the optical reflection, causing dropouts.
- **The gold standard is PSG (polysomnography).** It records the body's electrical signals directly. Every consumer method falls short of its accuracy.

## Where mmWave radar fits

mmWave radar's principle is similar to the accelerometer approach — it detects motion amplitude and timing (you move, it knows) — but contactlessly, without anything on your body.

- **Breathing is measured directly and more accurately.** Wearables can only *compute* breathing because the wrist has no respiration of its own. mmWave observes chest displacement, so the respiration signal is stronger and more reliable.
- **Continuous, no-wear monitoring.** Because nothing needs to be worn or charged, it can watch over long periods and learn life patterns: prolonged lying down, how many times you got up at night, and so on. For elderly or unwell users who spend most of the day in bed, it can gently flag a change in routine.
- **Limitation.** It only works when you're inside the monitored space — analogous to "no watch means no measurement."

## Takeaways

Sleep is mostly about breathing — sleep well and you breathe steadily and amply. Health is mostly about heart rate. Different tools emphasize different things, and each technology has distinct strengths that suit different users.

No consumer method matches PSG. mmWave radar is best understood as a contactless, continuous complement that emphasizes respiration and daily routine — a different instrument for a different need.
