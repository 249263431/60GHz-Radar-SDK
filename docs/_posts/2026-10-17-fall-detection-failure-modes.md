---
layout: post
title: "Why Fall Detection Is Harder Than It Looks: Three Failure Modes"
date: 2026-10-17 09:00:00 +0800
categories: [fundamentals]
tags: [fall detection, false alarm, missed detection, acceptance criteria, mmwave radar]
---

<!-- filename: 2026-10-17-fall-detection-failure-modes.md -->

Fall detection is one of the most requested features in elderly care sensing, and one of the least straightforward to deliver. The name suggests a single clear event — a person falls, an alarm fires — but the sensing problem behind it has three distinct failure modes, and each one behaves differently in a real room.

This article describes those three modes: two kinds of false alarm and one kind of missed event. It also covers a problem that sits above all of them, which is that "a fall" is not a precisely defined acceptance criterion. Understanding these four things is the difference between a fall radar that works in a deployment and one that generates complaints.

## Why the Demand Is Real

For an elderly person with reduced mobility — unsteady gait, a cane, limited strength, or partial loss of independence — a fall is not a health metric. It is an event where somebody needs to know, quickly, so that the person can be helped up.

That is a different requirement from sleep monitoring or presence detection. Nobody is harmed by a sleep score that arrives late. A person lying on a bathroom floor for forty minutes is harmed by an alarm that did not fire, and by an alarm that fired three times the previous night for no reason.

So the sensing requirements are strict in both directions at once: the false alarm rate has to be low enough that the alarm is believed, and the missed-event rate has to be low enough that the alarm is trusted. Those two requirements pull against each other, and that tension is the source of everything below.

## False Alarm 1: Nobody There, but the Radar Reports Someone

The first failure mode is an alarm raised in an empty room.

A bathroom is full of things that move without a person present. An open window lets a curtain or a towel move in the air flow. A leaking container drips. A toilet flushes. An extractor fan vibrates. Each of these produces a radar return, and a return from a low, stationary-looking target is exactly what a fall looks like.

The sequence is:

1. An interference source produces a return.
2. The radar classifies the return as a target rather than clutter.
3. The target is low and does not move.
4. A low, stationary target is the signature the fall detector is looking for.
5. An alarm is published, in an empty room.

This is the failure mode that matters most at scale. A single false alarm is an inconvenience. A deployment of hundreds of units, each producing occasional empty-room alarms, produces a continuous stream of notifications that nobody acts on — and once occupants learn that the alarm is usually wrong, they stop responding to it, including when it is right.

Reducing this mode is mostly about the room, not the algorithm. Knowing whether a person is actually present — through entry and exit tracking, or through a corroborating signal — removes most of the candidates before the posture judgement runs. That is why a single-door bathroom is a much easier sensing problem than an open room, and why the scene matters as much as the sensor.

## False Alarm 2: Someone There, but the Posture Is Misread

The second failure mode is an alarm raised while a person is present and doing something entirely normal.

Three examples, all of them ordinary activities:

| Activity | What the Radar Sees | What It Concludes |
|---|---|---|
| Squatting to wash clothes | Hand and arm movement at low height, head relatively still | A person struggling on the floor |
| Mopping the floor | A moving object at floor level, low and wide | A low, moving target |
| Sitting on the toilet, swinging a leg | Foot movement at low height, torso still | Movement near the floor without a head |

The common mechanism is a mismatch between what the radar weights and where the person's mass actually is. A radar that over-weights low-level movement and under-weights the position of the torso will read a squatting person as a collapsed one, because the hands are moving and the head is not.

This mode is harder to remove than the first one, because the person really is there. Entry and exit tracking does not help — the person never left. What helps is judging posture from the whole target rather than from a single region: height, centroid, and the ratio of movement at different heights together, rather than any one of them alone.

## Missed Events: When There Is No Descent

The third failure mode is the one nobody wants to talk about, because it is the one that matters most.

A fall detector's most reliable trigger is a rapid descent: a tall target collapses into a wide, low one within about a second. That transition is distinctive, and it is what most detection chains are built around.

Not every fall produces it.

A person sitting on a toilet who slowly slides sideways and comes to rest on the floor has not descended in the radar's sense. Their height changed, but gradually, and without the rapid motion the detector is looking for. The same applies to a slow slump against a wall, or a controlled lowering after dizziness.

The result is a missed event in exactly the scenario where the person is least able to get up.

This is a genuine limitation of the approach, not a tuning problem. A detector that requires a rapid descent will miss slow descents. A detector that does not require one will report every sitting-down and lying-down motion as a fall. There is no setting that gives both.

## Why "a Fall" Is Not an Acceptance Criterion

Above the three failure modes sits a problem that is easier to state than to solve: nobody agrees on what counts as a fall.

Consider the questions a real deployment has to answer:

- Lying flat on the floor is clearly a fall. Is lying prone, face down, also a fall?
- Is a person sitting on the floor against a wall a fall, or a rest?
- Is a slow slide from a seated position a fall, or a collapse?
- Is a person who is down and then gets up unaided a reportable event?

Every one of these is a real scenario, and every one is answered differently by different vendors, different products, and different care providers. The word "fall" in a specification covers all of them and none of them.

The practical consequence is that two products with the same quoted capability can behave completely differently on site, and a buyer cannot tell which from the specification. It also means that a requirement written as "detects falls" cannot be verified on delivery, because there is no shared definition of the thing being verified.

This is why scenario-based acceptance matters more in this category than in any other sensing category. "Detects a person lying flat on the floor within 10 seconds in a 2 m × 3 m bathroom with the door closed" is testable. "Fall detection" is not.

## How to Evaluate a Fall Radar

Five checks that separate a product with real deployment experience from one with a specification sheet.

1. **Ask for a test in an empty room with interference.** A running extractor fan, a hanging cloth, a dripping tap, and a toilet flush, with nobody present. If the radar alarms, the first failure mode is unaddressed.
2. **Ask for a test of normal low activity.** Squatting, mopping, sitting and swinging a leg. These are the second failure mode, and they are easy to demonstrate.
3. **Ask specifically about slow descents.** Not "does it detect falls" — ask whether it reports a person who slowly slides from a seated position to the floor. This is the question most vendors will not answer directly.
4. **Ask what environment the quoted false alarm rate was measured in.** A rate measured in an empty room with no interference is not a rate. Ask for the room, the installation height, and the interference sources present.
5. **Write the acceptance criteria as scenarios, not capabilities.** List the postures, the room, the timing, and the interference sources you expect. A product that passes a scenario list can be verified on delivery; a product that passes a capability list cannot.

The fifth point is the one that changes outcomes. It moves the conversation from what the product claims to what the product does in your room.

## What Is Still Open

Three things in this category do not have clean answers, and it is worth being explicit about them.

- **Slow descents remain the hardest case.** Any detector that requires a rapid descent will miss them, and relaxing that requirement raises the false alarm rate. The trade-off is inherent.
- **The boundary between a fall and a rest is genuinely ambiguous.** Sitting on the floor, lying on a sofa, and sliding against a wall are all real behaviours, and no sensing approach classifies them perfectly.
- **There is no industry-standard definition of a fall.** Until acceptance criteria are written as scenarios rather than capabilities, two products with the same specification will continue to behave differently on site.

None of these are solved by a better algorithm alone. They are solved by matching the detection logic to the room, by writing acceptance criteria that can actually be tested, and by being explicit about which cases a given product is and is not specified for.

## Summary

Fall detection has three failure modes, and they are different problems with different causes. Empty-room alarms come from interference that resembles a low, stationary person. Misread postures come from weighting low-level movement over the position of the torso. Missed events come from requiring a rapid descent that a slow collapse does not produce.

Above all three sits the definition problem: "a fall" is not a precisely defined event, so a specification that says "detects falls" cannot be verified.

Evaluate a fall radar by asking for the empty-room test, the normal-activity test, and the slow-descent test, and by writing your acceptance criteria as scenarios rather than capabilities. The products that survive those four questions are the ones that have been deployed rather than only specified.
