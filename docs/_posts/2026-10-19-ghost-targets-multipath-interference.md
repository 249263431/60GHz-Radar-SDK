---
layout: post
title: "Ghost Targets and Glass Walls: Multipath Interference in Home Radar Care"
date: 2026-10-19 09:00:00 +0800
categories: [guide]
tags: [multipath, ghost targets, interference, mmwave radar, elderly care, fall detection, presence radar, scene mode, sensitivity, deployment]
description: "Why multipath reflections and clutter make people-counting and fall detection unreliable in home radar — and how scene mode, zone detection, and sensitivity tuning suppress them."
---

A ceiling radar watching an elderly person's room promises something simple: know whether someone is there, notice when they fall, and do it without a camera. Then comes the first confusing week.

The room is empty and the dashboard says occupied. A person sits in one chair and the activity trace suggests motion on the other side of the room. Nobody counts heads, yet something keeps behaving like a second occupant.

Almost every one of those stories traces back to the same physics: **multipath interference** — and the two judgements it damages most are **how many people are in the room** and **whether someone has fallen**. This article is built around that fact: what multipath and clutter actually are, how each failure mode happens, why modern interiors make both worse, and which real settings suppress them.

## Why wavelength changes everything

Start with the physical layer, because it explains both the strength and the weakness.

At **60 GHz** the wavelength is roughly **5 mm** — short enough to build a compact antenna array, fine enough to resolve small targets, and long enough to behave very differently from light. Millimetre waves pass through **smoke, dust, and non-metallic materials** where optical methods struggle. A camera is blinded by darkness, steam, or fog; radar generates its own illumination and does not care.

Radar also produces no image. There is no lens, no video, nothing identifiable — which is precisely why it is acceptable in bedrooms and bathrooms where a camera never would be. That property is what makes it viable for dignified elderly monitoring in the first place.

The price of that robustness is that the same radio energy that fills the room does not politely stop at the person.

## What multipath is

Radar transmits a chirp and listens for what comes back. Ideally, energy travels to the person and straight back. In reality, the space is full of surfaces, and a smooth hard one — glass, glazed tile, polished stone, metal — behaves much like a mirror.

So the receiver hears several things:

- **The direct return**: energy that bounced off the person and came straight back.
- **Reflected returns**: energy that reached the person by bouncing off a wall first, or returned via another surface, arriving later and from a different apparent direction.
- **Structural returns**: energy that never touched a person at all, arriving from fixed objects in the room.

The receiver sums all of them. A reflected copy carries real energy at a real delay, so it can land at a distance and angle that correspond to no physical object. That is a **ghost target** — a phantom person the radar genuinely "sees" because something genuinely reflected.

This is not a defect that can be engineered away. It is like light casting a shadow: unavoidable wherever reflective geometry exists. The engineering question is not how to eliminate it, but how to keep it out of the decisions that matter.

## Where it bites hardest

Some interiors are far worse than others. In practice these are the repeat offenders:

- **Glass**: partitions, curtain walls, large windows, shower screens, glass cabinet doors.
- **Hard flat surfaces**: glazed or polished tile, polished concrete, marble.
- **Metal**: frames, fixtures, appliances, light fittings, metal ceiling panels.
- **Water**: water pipes, and especially **flowing water**, are documented interference sources — moving reflective targets are the worst kind.
- **Moving non-persons**: curtains, fans, and pets all produce returns that resemble a human.
- **The person themselves**: arms and legs move independently of the torso, so a single body can legitimately generate several moving returns at once.

Modern interiors trend toward open plans with glass balustrades, glazed partitions, and polished floors. That aesthetic is close to a worst case for radar, and it is worth knowing before the install begins.

## The two kinds of noise you actually see

It helps to sort unwanted returns into two classes, because they behave differently and need different answers.

**Static clutter** is everything that comes back from objects rather than from people: a glazed tile corner, a window frame, a metal light fitting, a pipe. The documentation gives this a name — **dry interference**, returns that arrive from fixed structures rather than from a person. Because those targets never move, a well-tuned detection chain learns them and removes them. Until it has learned them, or when the geometry changes because a chair moved or a blind came down, the practical symptom is phantom occupancy: the room reads occupied when nobody is there.

**Dynamic clutter** moves, which is exactly what makes it dangerous. It comes from two places:

- **Things that are not people**: fans, curtains, pets, flowing water. All of them produce returns that resemble a person, and unlike structural returns they genuinely change from frame to frame.
- **Parts of the person themselves**: arms and legs move independently of the torso, so one body legitimately throws several moving returns at once.

Static clutter mostly creates phantom occupancy. Dynamic clutter is what corrupts the harder judgements — how many people there are, and what they are doing.

Stated plainly, then, the algorithm's real job is **separating a person from their own echoes**: deciding which returns belong to one body and which belong to the room. Everything below is a way of making that separation easier.

## The counter-intuitive part: reflections are not always the enemy

Here is the twist that surprises most people, and our own bathroom-tuned fall radar illustrates it well. In a small tiled bathroom, the radar performs **better** than in a quiet empty room, and the tiling is the reason.

Hard, flat, reflective surfaces create strong multipath, so the radar receives the same target through several paths simultaneously. The return from a person lying still becomes strong and stable — exactly the case that matters for fall detection.

The very same reflections also produce what can be called **dry interference**: returns arriving from fixed structures rather than from a person. The detection chain filters those out, keeping the useful signal and discarding the structural returns. In that room the trade is clearly favourable.

Step into an open-plan space and the situation reverses. More volume, longer distances, absorptive materials like carpet and plasterboard — weaker returns overall, and the reflections that remain come from surfaces outside the area you actually care about. Reflections are not inherently bad. **Unbounded reflections in a space larger than the one you are watching are the problem.**

## Why people counting goes wrong

This is the first judgement multipath breaks, and the failure runs in two directions.

**Phantom count — one person read as two.** Energy from a single body reaches the receiver along several paths. A strong reflection off glass or tile can arrive with enough delay and enough energy to separate from the true return, and the receiver then resolves what looks like a second target standing nearby. Add dynamic clutter from swinging arms and legs, and the picture gets busier still: several moving returns belong to one body, and anything counting apparent targets will report several people.

This is precisely the "one person displayed as two" symptom — and note that it is not the radar malfunctioning. It received energy along several paths, and every one of those paths carried something real.

**Merged count — two people read as one.** The opposite failure is documented directly in the presence radar's limitations: **two people in the area read the same as one.** When bodies are close together their return clusters overlap, and no amount of care in post-processing reliably separates them from radio alone.

Both errors share one root cause: the radar reports *what it received*, and what it received is energy along multiple paths — not a list of people.

That is why our presence radar is specified the way it is. It reports an **occupancy state**, not a head count:

| State | Meaning |
|---|---|
| Unoccupied | No person detected |
| Occupied, still | Someone is present and not moving |
| Occupied, moving | Someone is present and moving |

So the honest engineering position is not "it counts people and occasionally gets the number wrong" — it is that presence and movement are what this class of sensor can promise, while reliable head counting is not among them. The suppression techniques below exist precisely so the occupancy judgement stays trustworthy even though a head count never could be.

For elderly care that is usually the right trade anyway. "Someone is in the room and has been motionless for an unusual length of time" is the question worth answering, and it needs no headcount.

## Why fall detection goes wrong

Fall detection is the more demanding application, because it rests on a **judgement rather than a threshold**: the radar has to decide what the body is doing — standing, sitting, lying, falling. Multipath and dynamic clutter attack that judgement in three specific ways.

**Masked fall — the person goes quiet, but never quite still.** A fall alarm is not a single-frame reading. Breathing rate, heart rate, fall alarms, and dwell alarms all take **seconds to minutes**, because they measure a rhythm or require a confirmed decision. That confirmation depends on the return settling into a stable "lying, not moving" picture. Reflected energy arriving by a second path — a beam glancing off glass, or off a tiled wall — can keep adding small apparent motion to the target cluster. The picture never fully settles, so the decision takes longer, or does not complete at all. In a care scenario this is the failure that matters most, because it delays the alarm rather than adding a false one.

**False fall — motion that never happened.** The mirror image. A structural return that genuinely varies — a fan, a curtain, water running in a pipe — injects movement energy into the region. Depending on where it lands, that can look like a collapse happening in an empty room. The documented priority is blunt and correct: when reacting quickly conflicts with resisting interference, **protect interference resistance first.** A radar that occasionally triggers late is better than one that reports a person who is not there.

**Zone escape — the fall lands outside what the radar watches.** Fall detection only covers the area the radar is configured to judge. Our own bathroom-tuned fall radar states it plainly: the room's walls bound the detection area, so **a fall cannot land outside it**. Remove that bound by moving the same radar into an open-plan space, and the coverage no longer covers the whole floor. A fall in the unmonitored part is not a signal-chain failure at all; it is a person outside the region you asked the sensor to watch.

Which brings back the earlier point, now sharpened. The same reflections that corrupt all three are also what let the radar see a person lying perfectly still in the first place: in a small tiled bathroom, strong multipath means the return from a still body is strong and stable — precisely the case fall detection depends on. Reflections are not the enemy. **Unknown, unbounded reflections are.** Every mitigation that follows is a way of making the reflections known and bounded.

## Suppression tool 1: scene mode and zone detection

The most effective control is also the simplest: **shrink the world the radar listens to.**

Scene modes change the distance at which a moving person triggers detection. Static range stays the same across modes; what changes is how far away movement is noticed.

| Scene mode | Trigger range (wide) | Trigger range (narrow) |
|---|---|---|
| Maximum area | 12 m | 8 m |
| Office | 10 m | 7 m |
| Hotel (default) | 9 m | 6 m |
| Living room | 7 m | 5 m |
| Bedroom | 4 m | 3 m |
| **Zone detection** | **3 m** | **2 m** |
| Bathroom | 1.8 m | 1.5 m |

The documentation states it plainly: **choosing the smallest mode that still covers the room is the single most effective way to remove false triggers.** This is "boundary setting" turned into a concrete setting — a reflection from outside the locked zone never reaches the decision stage.

Zone detection mode exists precisely for this: reducing the footprint to a defined region when the room itself is larger than the area you need to watch, such as one chair beside a bed, or the clear floor next to a doorway.

One practical warning: the finished presence radar exposes scene mode with **different numbering** over MQTT (`0` default, `1` zone, `2` bathroom, `3` bedroom, `4` living room, `5` office, `6` hotel). Do not mix the two lists — they are different protocols.

## Suppression tool 2: sensitivity

If scene mode bounds the space, sensitivity sets how hard the radar listens inside it. The setting runs from **1 to 10, with 7 as the documented default**.

| Range | Intended for |
|---|---|
| 7–10 | Low-interference spaces where fine motion must be caught: storage rooms, vaults |
| 4–6 | Ordinary indoor spaces: homes, hotels, offices |
| 1–3 | Spaces with heavy interference: corridors, parking areas, where only large movement should trigger |

Higher sensitivity catches smaller movements — and also picks up more false triggers from curtains and fans. Lower sensitivity can ignore a person who is barely moving, which in a care scenario is the failure you least want.

Two documented adjustments matter for multipath specifically:

- **Small rooms (about 15 m² or less)**: reflections are stronger. Reduce sensitivity by one step to keep the unoccupied judgement stable.
- **Large rooms (about 40 m² or more)**: reflections are weaker. Raise by one step to keep the occupied judgement stable.

Sensitivity is not free in either direction. It is the dial that trades missed subtle motion against phantom motion.

## Suppression tool 3: time, geometry, and logic

The remaining defence is algorithmic, and it rests on one observation: **a real person behaves consistently over time; a reflection usually does not.**

- **Confirmation across frames.** Every published value is the product of averaging over intervals and frames. The same averaging that produces breathing rate or a fall decision is also what suppresses a return that flickers in and out.
- **Known geometry.** Where a room has a single door, entry and exit direction resolves most interference: something that appears without entering through the door is not a person. This logic is what makes our bathroom-tuned fall radar so effective there — and it is also the first thing that stops working when that radar is moved to an open space.
- **Accept the built-in delay.** The presence radar has roughly a **40-second unoccupied delay** by design. It is not sluggishness; it is the cost of being sure nobody is there.

The documented priority when two goals conflict is worth quoting: **protect interference resistance first.** A radar that occasionally triggers late is better than one that reports a person who is not there. For elderly monitoring, false alarms erode trust faster than delays do.

## The first few seconds

Expect a brief settling period. Right after power-on or a reset, the detection chain has not yet built its picture of which returns are fixed structures and which are live targets — remember that filtering structural "dry interference" is something it learns to separate. So short-lived odd readings while the model forms are normal, and they stabilise as frames accumulate.

Treat those first moments as commissioning behaviour, not as evidence about the installation. Judge the radar after it has watched the room for a while.

## Commissioning checklist

Distilled from the documented tuning procedure:

1. Compare the room area with the scene mode table and pick a starting mode.
2. Confirm where the person actually spends time — the centre of that area is where the radar belongs.
3. Aim the radar's long edge at the room entrance.
4. Survey for interference sources: glass, metal, water pipes, flowing water, fans, curtains, pets.
5. If interference exists, **shrink the trigger range** by moving to a smaller scene mode.
6. Set sensitivity from the room type; adjust one step down for small rooms, one step up for large ones.
7. Introduce interference **one source at a time** and confirm it does not produce occupancy while nobody is present.
8. Confirm the state clears after the person leaves (allowing for the ~40 s delay), and that later interference does not raise an alarm.

Step 7 is the one people skip, and it is the one that predicts success.

## Takeaway

- **Multipath is physics**, not a defect: hard smooth surfaces return real energy along extra paths, creating ghost targets.
- **Glass-heavy modern interiors are the worst case**, alongside metal, flowing water, fans, curtains, and pets.
- **Reflections can help**: in small tiled rooms strong multipath strengthens the return, and the detection chain filters the fixed structural part.
- **Counting people is the fragile claim**, and it fails in both directions: one body can split into apparent multiple targets via reflection and limb clutter, and two people can merge into one cluster.
- **Fall detection is a judgement, not a threshold** — masked falls (the picture never settles), false motion from moving clutter, and falls landing outside the configured zone are all the same multipath problem wearing different clothes.
- **Bound the space first**: smallest adequate scene mode, or Zone detection for a defined region.
- **Then tune sensitivity**, down in small reflective rooms, up in large absorptive ones.
- **Let time and geometry do the rest**, and accept that resisting false alarms costs you a little latency.

Reflections will always be there. The craft is deciding which ones are allowed to matter.

Finally, for elderly care specifically: this is not a medical device, and it does not replace human checks. It publishes alarms and state to your own system; what happens next is what you build. For anyone at real risk, radar is **an addition to a care routine, never a substitute for one**.

- See also: [Presence detection radar: range, sensitivity, and scene modes](https://249263431.github.io/60GHz-Radar-SDK/product/2026/10/17/presence-radar-guide.html), [why our fall radar is tuned for bathrooms](https://249263431.github.io/60GHz-Radar-SDK/product/2026/10/17/fall-radar-scene-optimization.html), and [why radar position drifts](https://249263431.github.io/60GHz-Radar-SDK/fundamentals/2026/10/18/radar-position-drift.html).
