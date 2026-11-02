---
layout: post
title: "At 3 A.M., One Radar Has to Answer Three Questions at Once"
date: 2026-11-03 08:00:00 +0800
categories: [fundamentals]
tags: [radar, mmwave, multi-target, tracking, fall detection, sleep]
description: "At 3 a.m. one sensor has to answer three questions at once, and two of them want opposite things from the same signal. How a multi-target radar splits the room instead of the clock."
---

<!-- filename: 2026-11-03-split-the-room-not-the-clock.md -->

At 3 a.m. an 82-year-old gets out of bed. She crosses two metres of dark floor towards the bathroom, and somewhere between the bed and the doorway she goes down.

On the ceiling above her, one sensor has to answer three questions in the next thirty seconds — and it has to answer them while the bed she left is still occupied.

The questions are easy to state:

- Is the person on the floor still there, and still moving?
- Is that a fall, or someone who sat down to rest?
- Is there still someone in the bed?

What makes them hard is not any single answer. It is that all three have to come from **one device, one stream of echoes, at the same moment** — and two of them want opposite things from that stream.

This is the problem the **R60BHC1-V** was built for. It is a 4T4R 60 GHz module. The finished unit around it mounts on the ceiling, tracks several people at once, divides the room into labelled zones, and runs a different algorithm inside each one.

## The assumption every earlier radar was allowed to make

Until now, every module on this site answered one question at a time. The presence radar answered *is anyone here*. The sleep radar answered *what was this person's night like*. The fall radar answered *has someone gone down*.

Each of them assumed the same thing without ever writing it down: **one room, one person, one question.**

That assumption is not laziness. It is what made them cheap and reliable. A sensor that only has to answer one question can throw away everything that is not relevant to it, and the throwing away is most of the engineering.

The multi-function radar does not get that luxury. Everything is relevant — and some of it is relevant to two algorithms that disagree about what it means.

## Why you cannot simply run both

Breathing during sleep is a displacement of the chest wall in the sub-centimetre range, and it only becomes visible when everything else is still. To the sleep pipeline, body movement is **noise to suppress**.

A fall is the opposite signature: a fast drop in the tracked height of a target, followed by an absence of recovery. To that pipeline, movement is **the signal**, and what makes the event suspicious is the stillness that comes after it.

One chain cannot serve both from the same data. Gain-staged for a collapse, the chest wall disappears into the noise floor. Tuned to resolve breathing, a fall arrives as an overload that gets clipped and discarded.

The obvious fix — run the two in alternating time slices — is worse than it sounds. The events do not take turns. People fall while upright and moving; breathing has to be measured while they are still. Alternating halves the update rate of both, and guarantees you are looking at the wrong half at the moment it matters.

## So split the room instead

The solution is not temporal. It is spatial.

The field of view is cut into **labelled regions** — a bed, a doorway, a sofa — each one a polygon defined in the sensor's own coordinate system, configured from coordinates rather than fixed at manufacture.

| Region | What consumes it |
|---|---|
| Inside the bed label | Breathing, heart rate, sleep stage |
| Outside the bed label | Fall, prolonged stillness, presence |
| Doorway label | Entry and exit counting |

Two jobs that conflict in **time** do not conflict in **space**. The sleeper occupies the bed polygon; the floor is outside it. The same frame of radar data feeds both pipelines, because the two pipelines are looking at different parts of the same scene.

This costs nothing in update rate. The data was already there. What changed is who is allowed to read it.

## The bed edge is where the design earns its keep

That clean split breaks exactly where it matters most: the boundary.

Getting out of bed and falling out of bed look identical for the first second — a target leaving the bed polygon while its height drops. One is routine. The other is the reason the product exists.

What separates them is **height plus dwell time**. A target that stays below the configured height for longer than the configured fall duration — thirty seconds by default — is concluded as a fall. Someone who stands up and walks away climbs back above the threshold within a second or two, and never qualifies.

Two consequences for anyone integrating this:

- **The threshold has to be measured in the actual room.** A low bed and a thick duvet change what "on the floor" and "in bed" look like in height terms. The default is a starting point, not a setting.
- **Exit and fall are the same pattern, separated only by time.** Shorten the dwell to report falls faster and you begin classifying ordinary night-time get-ups as falls. That is a real trade-off, not a knob with a correct value.

## The output stops being a state

Here is the part that changes your code more than it changes the architecture.

A single-function module publishes a **state**: presence yes or no, sleep stage awake/light/deep, fall detected or not. One value per question, per room.

The multi-target module publishes a **list**. Each tracked target carries an index, a size, a feature code, X and Y position, height, speed, a posture code, and a projected radius. One object per person, updated as the scene moves.

The hard part of integration moves with it. It is no longer parsing — the frame format is documented and unremarkable. It is **identity**: deciding whether the target at index 2 this second is the same person as the target at index 1 a second ago.

Targets merge when two people pass close to each other. They split when one is occluded by furniture and reappears. The device does its own association and it is good at it — but no association is perfect, and when it fails it fails **silently**. You get a plausible list with the wrong identities attached to it.

Three rules follow:

- **Never key durable state to a target index.** It is a slot, not a name.
- **Rate-limit your reactions.** One frame of identity churn should not create a new person in your model.
- **Treat a missing target as ambiguous, not as gone.** Absence is the normal state of a working sensor.

## Eight postures, and the point where the list stops

Each target carries a posture code. The published set:

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

Read that carefully before designing a feature on top of it. These are **behavioural** postures — what someone is doing — not **sleep positions** — how a body is lying. On the back, on the left side, on the right side: all of them are code 7.

That is not a gap in the software. It is what the geometry can support. The datasheet quotes a **height accuracy of 0.4 m** and an **angular accuracy of 10°**. Rotating a body from its back onto its side changes the cross-section the radar sees, but the resulting change in apparent height is a matter of centimetres — well inside a 0.4 m uncertainty. The two positions are not separable in the first place.

The 10° figure is worth setting beside the theoretical one. A 4T4R array yields sixteen virtual channels, and at half-wavelength spacing the broadside resolution works out to about 7°. The published 10° is an accuracy figure, which is the honest number to design against rather than the theoretical floor. How channel count, layout and angular resolution relate is worked through in [Antenna Channels, Virtual Aperture]({{ site.baseurl }}{% post_url 2026-10-30-antenna-channels-virtual-aperture %}).

One entry deserves a second look: code 5, *low posture*, is not the same as code 7. It is the ambiguous middle — someone down, but not clearly fallen and not clearly in a bed. It exists because forcing every ambiguous frame into a definite category produces confident answers that are wrong, and that is worse than an explicit "I don't know".

## The headcount numbers disagree

A detail that affects your margins: the datasheet quotes a tracking ceiling of **10 in section 1.3** and **8 in section 6.2**. Both sections agree on the figure that matters more — **0 to 3 reliable**.

Design against 0–3. Beyond that, the count depends on how the region is occupied: two people standing close together, or one behind another from the radar's viewpoint, merge into one. Both sections carry the same caveat in slightly different words.

A brochure will quote the larger number. A specification you build on should use the smaller one.

## What it comes down to

Three things separate this from the modules before it:

- **You route by region, not by function.** Part of the sensor's job becomes configuration: which polygon is the bed, which is the door.
- **You consume a list, not a state.** Identity handling moves from the vendor into your code.
- **Posture is behavioural.** Good for falls, stillness and getting up. Not a substitute for sleep-position classification, and it will not become one by tuning.

Back to the room at 3 a.m. The person on the floor is below the height threshold and has not moved for thirty seconds. The target in the bed is still there, breathing. Two answers, one sensor, one stream of echoes — and neither answer had to wait for the other.

That is the whole idea.

## Checklist

- [ ] Regions configured as polygons matching the actual room, not left at defaults
- [ ] Fall height threshold measured in the actual room, with its actual bed
- [ ] Fall dwell time chosen against your night-time false-alarm tolerance
- [ ] No durable state keyed to a target index
- [ ] Identity churn rate-limited before it reaches your data model
- [ ] Headcount designed for 0–3, with the rest treated as indication only
- [ ] Posture codes treated as behaviour, never as sleep position
- [ ] Output discarded for the first 30 seconds after power-on
- [ ] Target disappearance treated as ambiguous, not as absence

## Where to go next

- What channel count actually buys: [Antenna Channels, Virtual Aperture]({{ site.baseurl }}{% post_url 2026-10-30-antenna-channels-virtual-aperture %})
- Footprint versus detection range: [Where the Beam Actually Lands]({{ site.baseurl }}{% post_url 2026-10-29-beam-coverage-installation-geometry %})
- What sleep staging can and cannot claim: [What Data-Driven Algorithms Changed]({{ site.baseurl }}{% post_url 2026-10-31-ai-sleep-staging-radar-limits %})
- Reading the stream without drawing wrong conclusions: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- The chain that starts after detection: [What SMS and Voice Calls Give a Fall Alert]({{ site.baseurl }}{% post_url 2026-11-01-sms-voice-alert-delivery-fall-radar %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

## A note on what this is

This is engineering documentation for integrators, based on the published datasheet values for the R60BHC1-V module and the finished multi-function unit built on it. It is not a medical claim, not a safety certification, and not a performance guarantee for any particular installation. Where two datasheet sections disagree, both values are given and the more conservative one is recommended for design. mmWave sensing carries an inherent false-alarm probability; any alerting product built on it should be designed on the assumption that it will sometimes be wrong.
