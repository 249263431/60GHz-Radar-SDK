---
layout: post
title: "Building a Sleep Product? What Rings, Watches, mmWave Radar Modules, and Infrared Actually Measure"
date: 2026-10-19 09:00:00 +0800
categories: [guide]
tags: [sleep monitoring, mmwave radar, radar module, infrared, wearable, ring, watch, ppg, accelerometer, hrv, rem, psg]
description: "A plain-language comparison of the four ways to sense sleep — rings, watches, mmWave radar modules, and infrared — covering what each measures directly and what it can only infer."
---

A growing number of teams are building sleep monitoring around a small mmWave radar **module**. It gets slipped into a bedside lamp, hidden inside an AI companion toy, built into an intercom, or mounted on the headboard or under the bed frame. The questions they are all trying to answer sound simple: *is this person asleep, and what are their vital signs doing?*

To answer those questions you have to choose a sensing technology, and there are four realistic options right now: a **ring**, a **watch or band**, a **mmWave radar module**, and **infrared**. They are not four versions of the same instrument. Each one directly measures one or two physical signals, and everything else it reports is *inferred* from those signals. If you are integrating one of them into a product, knowing exactly which is which is the difference between a feature you can stand behind and a number you are quietly guessing at.

This article compares all four at the sensing level: what each actually measures, what it is good at, where it breaks down, and how much of the sleep score behind it is real measurement versus calculation.

## The four approaches at a glance

| Option | Worn or embedded | What it measures directly | Good at | Weak at |
|---|---|---|---|---|
| **Ring** | Worn on finger | Movement (accelerometer), pulse wave at the skin (optical PPG) | Heart rate, HRV, sleep/wake timing; the most accurate wearable thanks to a tight, stable fit | Subtle breathing detail is limited; REM is inferred |
| **Watch / band** | Worn on wrist | Same two sensors as a ring | Heart-rate trends, movement, general activity | Accuracy drops when the fit is loose or inconsistent; gaps between sensor and skin ruin readings |
| **mmWave radar module** | Embedded — bare sensing core wired to your host MCU over **UART** (no Wi-Fi, no MQTT) | Chest displacement from breathing and heartbeat at a distance, presence, micro-motion | Contact-free breathing rate and presence in bed; nothing to wear, charge, or remember | Large posture changes are ambiguous on their own; separating "breathing" from "fidgeting" or "using a phone" takes your own logic |
| **Infrared** | Embedded or mounted | Body outline, heat signature, gross movement | Posture and big movements: rolled over, sat up, got out of bed | Largely insensitive to micro-motion like breathing; weak candidate for respiration by itself |

Notice the difference in form factor in that table. Rings and watches are finished devices a person agrees to wear. A **radar module** and an **infrared sensor** are components you build into something else. That distinction matters more than it looks, because it decides whose job it is to make sense of the signal.

## Wearables: the same two sensors, very different results

A ring and a wristband contain essentially the same measurement pair:

- **An accelerometer**, so any movement is immediately known. If the body moved, the device knows.
- **An optical heart-rate sensor (PPG)**, which shines light into the skin and watches how blood flow changes with each beat. Done well, this produces a genuinely usable heart rate.

The interesting part is that the same two sensors routinely produce better numbers on a finger than on a wrist, purely because of **fit**. A ring is held snugly against the skin and stays that way. People generally dislike wearing a watch tight enough to do the same job — it ends up slightly loose, or tight one hour and loose the next, and the moment the body shifts and opens a gap between the sensor and the skin, the reading degrades. Overall, ring accuracy tends to come out above watch accuracy for exactly this reason.

Wearables also give you **HRV (heart-rate variability)**, the small beat-to-beat changes in timing. The interpretation is worth stating plainly: variability is the healthy sign. A heart rate that speeds up and slows down from beat to beat reflects a responsive autonomic nervous system. A heart rate that stays mechanically identical is the flatter, less healthy pattern. A lot of consumer sleep and recovery metrics are built on top of this one signal.

What wearables are really answering is a discrimination problem: given that motion and pulse are both being watched, can we tell whether this person is **only breathing**, or **scrolling a phone**, or **making small restless movements**? Answer that cleanly and a large part of sleep and wake behaviour falls out of it.

## mmWave radar module: the micro-motion specialist

A mmWave radar module works without touching the subject and without a camera pointed at them. It fills the space with radio energy and reads the sub-millimetre chest movement produced by breathing and heartbeat. This is why it lands in sleep products so naturally: for a person in bed, nothing has to be worn, charged, or remembered.

Its clear strength is **micro-motion**. Breathing-rate-level movement and the presence of a living body in a defined space are things this sensing modality does well, quietly and continuously.

Its weakness sits at the other end of the scale. Large posture events — rolling over, sitting up, swinging legs out of bed — are much less distinctive when all you are looking at is reflected radio energy, and a raw radar return on its own struggles to separate "breathing normally" from "wide awake, holding a phone, thumb twitching." That separation is where the engineering goes.

**One integration fact shapes everything else: this module speaks UART, not MQTT.** It is the bare sensing core without a housing and without the Wi-Fi section, and it reports binary frames over a **3.3 V TTL UART at 115200 bps, 8N1**. There is no network to configure and no broker in the picture — any serial terminal with a USB-TTL adapter can read it directly. MQTT and Wi-Fi belong to the *finished* unit, which is the same sensing core plus a housing and a network stack. So if your product needs to publish MQTT topics, that work lands on your own host microcontroller: read the UART frames on one side and publish them yourself, which is exactly what the finished product does internally. The two variants are compared in detail in [Two Architectures, Two Protocols: MQTT vs UART](https://249263431.github.io/60GHz-Radar-SDK/fundamentals/2026/10/17/radar-architecture-mqtt-vs-uart.html).

Because this is a **module** rather than a finished consumer product, you also gain an advantage here. The module hands you the sensing layer; you own the interpretation layer. Whether your product can tell those states apart — and whether it fuses radar with another modality — is up to the logic you write on top of it.

## Infrared: the posture specialist

Infrared sits almost exactly opposite radar on the micro-motion axis. It is largely **insensitive** to the tiny movements that encode breathing and pulse, which makes it a poor standalone choice for respiration. But it is comparatively **good at posture**: it can tell you whether the person rolled, sat up, or got out of bed, because those events change the silhouette and heat signature in obvious ways.

So neither technology is the whole answer:

- **Radar module** → micro-motion is reliable (breathing, heartbeat-level movement, presence)
- **Infrared** → posture is reliable (sat up, rolled over, left the bed)

Put them together and each covers the other's gap. If your product cares about both "is she breathing steadily?" and "did he just get out of bed?", a single modality will always leave you guessing about one of them.

## Where sleep staging actually comes from

Awake, light sleep, and deep sleep — whether they arrive from a wearable or from the radar's own report — are largely derived from **how often a person moves and how much they move**, together with breathing patterns. Movement becomes quieter and less frequent as sleep deepens, so counting and characterising motion events across the night produces a plausible three-stage picture.

The sleep radar is a concrete example of this. It publishes a sleep-stage value every 10 minutes — deep sleep, light sleep, awake, or out of bed — plus time spent in each stage, and a single sleep score when the night ends. Those are genuinely useful outputs, and every one of them is a derived judgement: the stage is evaluated over a 10-minute window rather than observed continuously, and the product documentation describes the score as computed from duration, stage proportions, movement, and breathing.

## REM: every one of these devices is guessing

True REM sleep is identified by what the eyes are doing and by muscle tone dropping away — which is exactly why none of these technologies measures it directly. A ring, a watch, a radar module, and an infrared sensor are not watching your eyes.

Radar has a particularly hard time here, because the physical correlates of REM are unhelpfully subtle to a radio sensor: muscles relax, occasional twitches appear, movement becomes rare. None of that produces a distinctive radar signature on its own.

The product-level evidence is right there in the radar's own output: its sleep-stage field covers deep sleep, light sleep, awake, and out of bed. There is **no REM value** to read. If even the sensor that watches a chest all night does not publish a REM stage, any REM number you see in an app came from a model's inference.

Wristbands and rings report REM too, but by **calculation**: if heart rate behaves in a certain way and the body has been perfectly still for a stretch, the model infers REM. Useful, often directionally right — still an inference rather than a measurement.

This is the single most over-trusted number on consumer sleep dashboards.

## The honest ceiling: PSG is the yardstick

Everything above is measured against, and falls far short of, **polysomnography (PSG)** — the wired laboratory study that records the body's actual electrical activity: brain waves, eye movement, muscle tone, heart rhythm, airflow. PSG is how sleep is really staged, and it is several levels of fidelity above anything discussed here.

That gap does not make consumer sensing useless. It means **different applications tolerate different precision**. It also means none of these four is a medical measurement, and its output should not be presented as one. Wanting to know whether an elderly person got out of bed at 3 a.m., or whether breathing stopped for a worrying stretch, is a completely different requirement from diagnosing a sleep disorder. Choosing among these four technologies is mostly a question of which failure modes your application can live with.

## How to choose

A practical way to frame the decision:

- **Contact-free presence and breathing for elderly care** → radar module. Nothing to wear is often the deciding requirement.
- **Personal sleep trends and HRV** → a ring, if the wearer tolerates it; a watch otherwise, accepting that looser fit means noisier data.
- **Posture and getting-out-of-bed events** → infrared, or radar fused with infrared.
- **Best overall coverage** → radar module for micro-motion plus infrared for posture, with your own logic combining them. No single sensor does all of it.

The recurring theme: **do not ask one sensor to do every job.** Decide which physical signal you actually need, then pick the technology that measures that signal directly.

## Takeaway

- Rings and watches both run an accelerometer plus an optical pulse sensor; **rings usually win on accuracy purely because they fit better**.
- **Radar modules win on micro-motion** — contact-free breathing and presence.
- **Infrared wins on posture** and is weak on micro-motion; the two are natural complements.
- Sleep **stages are computed** from movement statistics.
- **REM is inferred everywhere**, never directly measured outside a laboratory.
- **PSG** is the reference standard, and the gap is real — so match the precision to the application.

Every sleep device reports a mix of things it genuinely measured and things it calculated. Knowing which is which is the most useful thing you can take from this comparison.

- See also: [Sleep radar installation position](https://249263431.github.io/60GHz-Radar-SDK/guide/2026/10/16/sleep-radar-installation-position.html) and [why radar position drifts](https://249263431.github.io/60GHz-Radar-SDK/fundamentals/2026/10/18/radar-position-drift.html).
