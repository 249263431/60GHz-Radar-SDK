---
layout: post
title: "Using Distance and Position Output to Debug a Radar Installation"
date: 2026-10-24 08:00:00 +0800
categories: [guide]
tags: [installation, debugging, radar, commissioning]
---

Our product guides flag the distance and position outputs the same way every time: they are **non-critical functions with limited accuracy**. That sentence is often read as "ignore these fields". It means the opposite of what integrators need — these are the most useful numbers you have while *installing* the sensor, precisely because they are raw and unsmoothed.

This article is the method: what to subscribe to, what a healthy reading looks like, and how to decide between moving the sensor, changing a setting, or removing an interference source.

> These values are diagnostic only. Do not build a product feature on them — the documentation is explicit that their accuracy is limited, and a customer-visible feature needs the primary outputs (presence, motion, fall state, breathing) instead.

## What you can subscribe to

| Signal | Range / cadence | What it tells you |
|---|---|---|
| Human distance | 0–65535 cm, about every 2 s | Roughly how far the detected target is |
| Human position | x, y, z, about every 2 s | Where in the beam the target sits |
| Motion status | 0 none / 1 still / 2 active | Whether the target is moving at all |
| Body-movement level | 0–100 (5 s module, 1 s finished) | How much movement, in fixed bands |
| Approach / leave | 1–5, about every 2 s (presence module) | Direction of travel relative to the radar |
| Presence energy & threshold | fall radar, switchable | How strong the presence detection is firing |

The body-movement bands are worth memorising, because they turn a number into a sentence:

| Value | Meaning |
|---|---|
| 0 % | No person in range |
| 1 % | Still — breathing only, no limb movement |
| 2–30 % | Micro movement: small head or limb motion |
| 31–60 % | Walking or fast limb movement |
| 61–100 % | Running or large close-range movement |

A correctly aimed radar on a person lying still should sit at **1 %**. That single number answers the most common installation question — "is it actually seeing the person?" — without any interpretation.

## The method

### 1. Empty-room baseline

Before anyone is in the room, subscribe and watch for a minute. A healthy sensor in an empty room reports **no target**: distance absent, body movement at 0 %, motion status 0. If you see a distance, a position, or movement above 0 % with nobody present, you have interference — a fan, an air-conditioner motor, a curtain, or a reflective surface. Find it now, not after the customer calls.

### 2. Walk the room

Walk slowly from the door to the far wall, then back. Watch three things:

- **The distance trace** should rise and fall smoothly and end near the far wall's real distance.
- **Body movement** should read 31–60 % while walking and drop to 1 % when you stand still.
- **Motion status** should follow: 2 while walking, 1 while standing still.

If the distance stops changing while you keep walking, you have left the beam — that is your coverage edge.

### 3. Map the coverage edge

Repeat the walk along different paths and mark where the sensor stops reporting. Compare with the documented coverage for your variant: the finished presence radar covers roughly a 2.7 m radius; the 24 GHz presence module reaches about 12 m for a moving person, 6 m for someone sitting, 3 m for a sleeping person; the sleep radar's breathing detection degrades beyond about 1.5 m and disappears before presence detection does.

If the edge is short of where people actually are, that is a mounting or scene-mode problem, not a fault.

### 4. Verify the aim

For a bed installation, the beam must hit the chest, not the wall: the sleep radar is side-wall mounted at 1.8–2.2 m, or above the bed head tilted 30°–45°, with 45° recommended. Position x/y/z and the body-movement value together tell you whether you got it right — a target that only ever reports 1 % at the edge of the range is a target you are barely seeing.

### 5. Confirm the empty transition

Leave the room and time it. Declaring a room empty takes about **40 s** of confirmed absence, by design; declaring it occupied takes under 0.5 s. If your "empty" transition is much faster, something is being missed.

## Red flags and what they usually mean

| What you see | Likely cause | First move |
|---|---|---|
| Target reported in an empty room | Fan, AC motor, curtain, pet, reflective metal | Remove or re-aim away from the source |
| Distance jumps around with nobody moving | Multipath from a metal surface or a second radar | Check line of sight; see [ghost targets]({{ site.baseurl }}{% post_url 2026-10-19-ghost-targets-multipath-interference %}) |
| Distance pinned at a round number (0 or max) | Out of range or field saturated | Move the sensor closer / reduce range |
| Body movement never drops below 30 % | Beam too wide for the room, or interference | Use a smaller scene mode; lower sensitivity |
| Still person reads 0 % | Beam missing the chest, or sensitivity too low | Re-aim; raise sensitivity one step |
| Readings change over weeks with nothing moved | Position drift — new furniture, hanging objects | See [radar position drift]({{ site.baseurl }}{% post_url 2026-10-18-radar-position-drift %}) |

## Decide in this order

1. **Can I remove the interference?** Cheapest fix. Fans and air-conditioner motors in the beam cause most false readings.
2. **Can I move the sensor?** Coverage that misses the occupied area is usually a position problem.
3. **Can I shrink the range?** The smallest scene mode that still covers the room is the most effective single change — the mode table is in the [presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}).
4. **Only then change sensitivity.** One step at a time, and note the change. Higher sensitivity catches micro-movement and also picks up curtains and fans.
5. **Room-size adjustment.** Below about 15 m², drop sensitivity one step; above about 40 m², raise it one step — small rooms strengthen reflections, large rooms weaken them.

## Keep the baseline

Save the empty-room reading and the walk-through trace for every installation. It is the only objective record of what the sensor saw on day one, and it turns "it worked when you left" into a comparison you can actually make.

## Where to go next

- The fields in full: [presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}), [sleep radar guide]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}), [fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %})
- Why it behaves this way: [How mmWave Radar Works]({{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %}) and [Radar Latency Explained]({{ site.baseurl }}{% post_url 2026-10-17-radar-latency-explained %})
- Setting expectations on the numbers: [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %})
- Mounting geometry: [Sleep Radar Mounting Position]({{ site.baseurl }}{% post_url 2026-10-16-sleep-radar-installation-position %})
- Multiple radars in one home: [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
