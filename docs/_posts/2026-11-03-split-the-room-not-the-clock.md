---
layout: post
title: "At 3 A.M., One Ceiling Sensor Has Five Jobs"
date: 2026-11-03 08:00:00 +0800
categories: [fundamentals]
tags: [radar, mmwave, multi-target, tracking, fall detection, sleep]
description: "The HT-W whole-home AI monitor tracks people, watches a sleeper's breathing and calls a fall from one ceiling-mounted 60 GHz radar. What it actually outputs, the numbers behind it, and where those numbers stop."
---

<!-- filename: 2026-11-03-split-the-room-not-the-clock.md -->

At 3 a.m. an 82-year-old gets out of bed. She crosses two metres of dark floor towards the bathroom, and somewhere between the bed and the doorway she goes down.

Nobody sees it. There is no camera in the room — by design — and nothing on her wrist, because she took the watch off to charge. On the ceiling, one small device has been listening to the room all night.

In the next thirty seconds it has five jobs to do. And it has to do all of them while the bed she just left is still occupied.

## Meet the HT-W

The **HT-W whole-home AI monitor** is a ceiling-mounted 60 GHz mmWave radar that reports events and device state over Wi-Fi. It does not form images and it does not record audio. What it works with is the echo: motion, position, and the sub-centimetre movement of a chest wall.

It is a 4T4R FMCW design — sixteen virtual channels — but the interesting part is not the antenna count. It is that the device is allowed to ask more than one question of the same frame of data.

Every radar we have written about here until now answered one question at a time. The presence radar answered *is anyone here*. The sleep radar answered *what was this person's night like*. The fall radar answered *has someone gone down*. Each was allowed to assume **one room, one person, one question** — and throwing away everything else is most of what made them cheap and reliable.

The HT-W does not get to throw anything away.

## Five jobs, five different ceilings

The product description lists five functions. Read the capacity column carefully, because it is not one number:

| Job | What it reports | Capacity |
|---|---|---|
| People and position tracking | Live positions and trajectories | **0–5 people** |
| Fall and prolonged-stillness monitoring | Fall, stillness, low-posture alarms | **0–3 people** |
| Respiration and heart rate | Breathing and heart rate values, high/low alarms | **1 person**, one frame every 3 s |
| Sleep detection | Awake / light / deep, get-ups, turning over, quality score | **1 person** |
| Bed and room entry / exit | Someone entering or leaving the bed or the room | **0–3 people** |

Five people can be tracked. Only three of them can be monitored for falls. Only one can have their breathing measured, and only one their sleep staged — and those two have to be the person inside the bed label.

A brochure that says "up to 5 people" is not lying, but a design built on it is. The number changes with the question you are asking.

## Split the room, not the clock

Two of these jobs want opposite things from the same signal.

Breathing is a sub-centimetre displacement that only becomes visible when everything else is still: to that pipeline, body movement is **noise to suppress**. A fall is a fast drop in tracked height followed by no recovery: to that pipeline, movement is **the signal**, and the stillness afterwards is what makes it suspicious.

Gain-staged for a collapse, the chest wall disappears into the noise floor. Tuned to resolve breathing, a fall arrives as an overload and gets clipped. Running the two in alternating time slices is worse — the events do not take turns, and alternating halves the update rate of both.

So the field of view is split in **space**, not time. Regions are configured as polygons in the sensor's own coordinate system — a bed, a doorway, a sofa — rather than fixed at manufacture:

| Region | What runs inside it |
|---|---|
| Inside the bed label | Breathing, heart rate, sleep stage |
| Outside the bed label | Fall, prolonged stillness, presence |
| Doorway label | Entry and exit counting |

The sleeper occupies the bed polygon; the floor is outside it. One frame of echoes feeds both pipelines, because the two pipelines are looking at different parts of the same scene. Nothing is lost in update rate — the data was already there. What changed is who is allowed to read it.

## The bed edge is where it earns its keep

That clean split breaks at the boundary, which is exactly where it matters.

Getting out of bed and falling out of bed look identical for the first second: a target leaves the bed polygon while its height drops. One is routine; the other is the reason the product exists.

What separates them is **height plus dwell time**. A person who stays below the configured height for longer than the configured fall duration — **30 seconds by default**, configurable — triggers the first alarm, and the device keeps checking for **two minutes** afterwards. Someone who stands up and walks away climbs back above the threshold within a second or two and never qualifies.

The threshold itself is `fallHeightThreshold`: range **40–70 cm**, **default 40 cm**. The documentation is unusually direct about the trade-off — raising it "increases the fall reporting rate, but may increase false alarms". Measure it in the actual room. A low bed and a thick duvet change what "in bed" and "on the floor" look like in height terms.

Shortening the dwell to report faster does the same thing in the other direction: ordinary night-time get-ups start being classified as falls. There is no correct value, only a value that matches your tolerance.

## What you actually receive

This is a Wi-Fi device, not a UART module. Out of the box it publishes to a public MQTT broker at `tcp://broker.emqx.io:1883`, on topics of the form `/Radar60SMF/{deviceID}/sys/property/post`. An MQTT command switches it to your own broker, and **factory pre-configuration is supported** — for a batch, the server address can be written in production so the site only needs network access.

The most useful topic is `trackingInfo`. Each tracked target carries:

```
1B index | 1B target size | 1B target feature
2B X | 2B Y | 2B height | 2B speed
1B posture | 4B projected radius (2B X + 2B Y)
```

And the posture set, from the protocol document:

| Code | Posture |
|---|---|
| 1 | Fall |
| 2 | Crouching |
| 3 | Sitting on a chair |
| 4 | Standing |
| 5 | Low posture |
| 6 | Prolonged stillness |
| 7 | Lying in bed |
| 8 | Sitting up in bed |

Sleep has three further reports: an 8-byte comprehensive status (presence, sleep state, average respiration, average heart rate, turning-over count, large- and small-movement ratios, apnea count), a 12-byte quality analysis (score, total duration, awake / light / deep ratios, time and count out of bed, average respiration and heart rate, apnea count), and an end-of-night report.

The combined respiration interface reports breathing state, confidence, value, heart rate, in/out of bed, getting up from bed, abnormal struggling, prolonged absence, weak vital signs and sleep state. **The breathing-distance waveform and the breathing waveform are marked as reserved interfaces** — more on that below.

One integration note that matters more than the parsing: this is a **list**, not a state. The target index is a slot, not a name. Never key durable state to it, rate-limit your reactions to a frame of identity churn, and treat a missing target as ambiguous rather than gone.

## The numbers

**Finished HT-W unit** (product manual)

| Item | Value |
|---|---|
| Radar operating frequency | 61–61.5 GHz |
| Detection coverage | 5 m × 4 m, about 20 m² |
| Detection angle | Horizontal 100°, vertical 90° |
| Installation | Ceiling-mounted, face down; 2.4–3.0 m |
| Installed-height setting | Default 240 cm, range 200–300 cm |
| Track area setting | Default radius 200 cm; X ±250 cm, Y ±200 cm |
| Power | USB 5 V / 2 A, Type-C |
| Wireless | Wi-Fi |
| Operating environment | -10 to +60 °C, 20–85% RH non-condensing |
| Size / net weight | 89 × 21 mm / 54.2 g |

**4T4R module** (R60BHC1-V datasheet V1.1, ceiling-mount version; distances are projection radii for ceiling mounting)

| Item | Value |
|---|---|
| Antennas / scheme | 4T4R / FMCW |
| Operating frequency fTX | 60–62 GHz (min 60, max 62) |
| Transmit power Pout | Typical 11 dBm |
| Detection angle | Horizontal ±50°, vertical ±40° |
| Range / angle / height accuracy | 0.08 m / 10° / 0.4 m |
| Motion trigger / dynamic / static distance | 1–3 m / 3–4 m / 2.5–4 m |
| Fall / sleep / respiration distance | 2 m each |
| Supply / current | 5.0–6 V / typical 500 mA, max 600 mA |
| Start-up stabilisation | ≥ 30 s |

The relationship between channel count and angular resolution is derived in [Antenna Channels, Virtual Aperture]({{ site.baseurl }}{% post_url 2026-10-30-antenna-channels-virtual-aperture %}); coverage geometry is worked through in [Where the Beam Actually Lands]({{ site.baseurl }}{% post_url 2026-10-29-beam-coverage-installation-geometry %}).

## Where the datasheets disagree

Four numbers do not line up across the finished-unit manual, the module datasheet and the quick-start guide. We are not going to pick one and pretend the others do not exist:

| Item | Finished unit | Module / guide |
|---|---|---|
| Frequency | 61–61.5 GHz | 60–62 GHz |
| Vertical angle | 90° | ±40°, i.e. 80° |
| Installation height | 2.4–3.0 m; setting allows 2.0–3.0 m | 2.2–2.8 m |
| People | 0–5 position tracking, 0–3 falls, 1 sleeper | 0–3 accurate; 8 in section 6.2, 10 in section 1.3 |

Two practical rules follow. **Design against 0–3 people**, not 5, 8 or 10 — every source agrees on that figure, and the larger ones carry the same caveat about region occupancy. **Confirm the operating band with the manufacturer** before any certification work; 60–62 GHz and 61–61.5 GHz are not two ways of writing the same band.

## Three honest limits

**Posture is behaviour, not sleep position.** On the back, on the left side, on the right side — all of them are code 7, "lying in bed". With a height accuracy of 0.4 m, the difference in apparent height simply is not resolvable. The eight codes describe what a person is doing, not how the body is lying. No amount of tuning changes that.

**No raw waveform.** The waveform fields in the combined respiration interface are reserved. If you need breathing and heart-rate waveforms rather than values, that lives on the other product line: the R60ABD1-based 1T3R sleep radar outputs them at 5 points per second. The two lines are complementary, not generations of the same thing.

**Voice calls are a reserved capability.** The manual lists "phone call + voice prompt alarm" among the functions, but also states plainly that the hardware platform may reserve interfaces such as a SIM slot which **not every model supports or enables**. Do not put that in a specification until the specific SKU confirms it. The delivery chain behind an alert is its own problem, covered in [What SMS and Voice Calls Give a Fall Alert]({{ site.baseurl }}{% post_url 2026-11-01-sms-voice-alert-delivery-fall-radar %}) and [Delivered Is Not Seen]({{ site.baseurl }}{% post_url 2026-11-02-delivered-is-not-seen-fall-alert %}).

## What it comes down to

Three things change when you move from a single-function module to this one:

- **You route by region, not by function.** Configuring the bed and door polygons is part of the integration, not a factory default.
- **You consume a list, not a state.** Identity handling moves into your code.
- **Every function has its own capacity.** Five tracked, three watched for falls, one measured for sleep.

Back to the room at 3 a.m. The person on the floor is below the height threshold and has not moved for thirty seconds. The target in the bed is still there, breathing. The doorway count has not changed. Five jobs, one sensor, one stream of echoes — and none of them had to wait for the others.

That is the whole idea.

## Checklist

- [ ] Regions configured as polygons matching the actual room
- [ ] Installed-height setting matches the actual ceiling height
- [ ] `fallHeightThreshold` measured in the room, with its actual bed (default 40 cm)
- [ ] Fall dwell time chosen against your night-time false-alarm tolerance
- [ ] MQTT broker switched to your own server, or factory pre-configured
- [ ] No durable state keyed to a target index
- [ ] Headcount designed for 0–3, with 5 treated as tracking only
- [ ] Posture codes treated as behaviour, never as sleep position
- [ ] Output discarded for the first 30 seconds after power-on
- [ ] Voice-call capability confirmed against the specific SKU before quoting it

## Where to go next

- Channel count and angular resolution: [Antenna Channels, Virtual Aperture]({{ site.baseurl }}{% post_url 2026-10-30-antenna-channels-virtual-aperture %})
- Footprint versus detection range: [Where the Beam Actually Lands]({{ site.baseurl }}{% post_url 2026-10-29-beam-coverage-installation-geometry %})
- What sleep staging can and cannot claim: [What Data-Driven Algorithms Changed]({{ site.baseurl }}{% post_url 2026-10-31-ai-sleep-staging-radar-limits %})
- Reading the stream without drawing wrong conclusions: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- After detection: [What SMS and Voice Calls Give a Fall Alert]({{ site.baseurl }}{% post_url 2026-11-01-sms-voice-alert-delivery-fall-radar %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

## A note on what this is

This is engineering documentation for integrators, based on the HT-W product manual, the Wi-Fi communication protocol document V1.7 and the R60BHC1-V datasheet V1.1. It is not a medical claim, not a safety certification, and not a performance guarantee for any particular installation. Where the documents disagree, both values are given and the more conservative one is recommended for design. mmWave sensing carries an inherent false-alarm probability; any alerting product built on it should be designed on the assumption that it will sometimes be wrong.
