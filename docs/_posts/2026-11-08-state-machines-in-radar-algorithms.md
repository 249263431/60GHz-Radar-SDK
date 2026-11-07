---
layout: post
title: "State Machines in Radar Algorithms: States, Guards, and Dwell Timers"
date: 2026-11-08 08:00:00 +0800
categories: [fundamentals]
tags: [radar, mmwave, state machine, algorithms, fall detection, mqtt]
description: "Nearly every property in the HT-W protocol is reported on state change. What that contract reveals about states, guards, dwell timers, asymmetric hysteresis, and why false-alarm control is explicit rather than adaptive."
---

<!-- filename: 2026-11-08-state-machines-in-radar-algorithms.md -->

**Starting from one line in the interface contract: *reported on state change*.**

Nearly every property in the HT-W Wi-Fi protocol document carries the same phrase: **reported on state change**. `someoneExists`, `motionStatus`, `fallStatus`, `getIntoBed`, `online`, `heartBeat`, `nobodyTimeStatus` and `realTimeHeadCount` are all described that way.

That is not a note about field formatting. It is an interface contract. What the device sends is not a stream of measurements but notifications that some state machine has made a transition. This article does not describe how the firmware is implemented. It reads the public output backwards and asks why a decision algorithm working on continuous, noisy signals that fluctuate from frame to frame is organised around state machines at all — and what roles states, guards and timers play.

## From per-frame classification to transitions with memory

Every frame can produce a result: is anyone present, what is the target height, how large is the motion. If each frame were classified independently and reported directly, one anomalous frame would produce one alarm — a brief pause while someone turns around, an echo disappearing behind furniture, the momentary height drop of standing up.

A state machine contributes **memory**. The current state is a conclusion accumulated over time, not a reading of the current frame. A transition requires its condition to hold and to keep holding long enough. The classification problem is thereby replaced by a transition problem with guards: the algorithm no longer asks "is this frame a fall?" but "what state am I in, and has this frame satisfied the condition for leaving it?"

Presence, motion, posture and sleep stage are all **persistent states**, not instantaneous quantities. Describing them as states fits the physics better than per-frame classification, and it is the reason the protocol reports changes rather than repeating full snapshots on a clock.

## Several orthogonal state machines, not one large one

One device handles presence, falls, prolonged stillness, respiration, heart rate, sleep and entry/exit counting. If all of those variables were merged into a single state machine, the number of states would be the product of the values in each layer: eight postures times three motion states times two presence states times three sleep states times the region labels. The combination count becomes unmaintainable very quickly. That multiplication is illustrative, not a published figure.

The protocol presents a different organisation: several **concurrent, orthogonal** state machines, each transitioning and reporting on its own.

| Layer | State set | Reporting |
|---|---|---|
| Device | Initialising, online / offline; heartbeat normal / abnormal | On state change |
| Presence | Absent / present | On state change |
| Motion | None / still / active | On state change |
| Posture | Eight posture classes; fall / no fall | On state change |
| Region | Inside bed / outside bed / doorway; entering / leaving | On state change |
| Sleep | Out of bed / in bed, then awake / light / deep, then sleep ended | Periodic, plus an end-of-night report |
| Headcount and tracks | Live headcount, target trajectories, target disappearance | On state change |

The practical benefit is that a transition in one layer does not rebuild the state of the others. A person turning over in bed changes the motion layer; it does not require the presence layer or the device layer to re-enter their decisions. Region labels for a bed, a doorway and a sofa are themselves a form of state partition — they decide which chain consumes a given frame of echoes, which matters especially in a multi-function product.

## Guards are mostly timers

The core of a state machine is not its states but its **guard conditions**. In this protocol, most guards are not boolean thresholds but dwell timers: a condition holding for a defined interval.

| Transition / action | Guard | Value | Source |
|---|---|---|---|
| Absent to present | Motion detected and sustained | Reported within 0.5 s | User manual v1.2, §6.1 DP1 |
| Present to absent | No valid target, sustained | About 30 s | User manual v1.2, §6.1 DP1 |
| Motion trigger / absence detection | — | ≤0.5 s / ≤40 s | Datasheet V1.1, §6.2 |
| Prolonged-stillness alarm | Abnormal stillness sustained | 5 min | User manual v1.2, §6.1 DP3 |
| Fall alarm | Fall condition met | Immediate | User manual v1.2, §6.1 DP2 |
| Sleep stage output | While in bed | Once every 10 min | User manual v1.2, §6.1 DP5 |
| Respiration and heart rate | While in bed | Once per 1 s (module) / one frame per 3 s (finished unit) | User manual DP7 / product manual |
| Sleep report | Sleep ended | Event-triggered | User manual v1.2, §6.1 DP6 |
| Power-on initialisation | Reset complete and ambient noise assessed | ≥30 s | Datasheet V1.1, §10.1 |
| Weak vital-sign alarm | Sustained duration | 25–120 min, default 30 min | User manual v1.2, §8.2 |
| Fall decision | Below height threshold without getting up | First alarm at 30 s, then 2 min of continued checking | Product manual |

Placed together, these values reveal the design choice: **when a single frame cannot distinguish a fall from crouching down, time is traded for confidence**. The guard changes from "height below threshold" to "height below threshold for 30 s". False alarms fall, response slows. That is an explicit, adjustable trade-off, not a classifier accuracy problem.

The actions also fall into three categories: report on state change (edge-triggered), periodic report (1 s, 3 s, 10 min, driven by a state clock), and event-triggered report (sleep ended, fall condition met). The distinction matters to an integrator, because only the first category represents an actual state transition.

## Asymmetric hysteresis: 0.5 s in, 30 s out

The presence layer uses different guards in the two directions: absent to present is reported within 0.5 s, while present to absent takes about 30 s.

This is **hysteresis**. Entering must be fast, because someone entering the room should be sensed promptly. Leaving must be slow, because brief stillness, turning around or occlusion by furniture can make echoes vanish for a few frames. If both directions used the same short timer, the state would flap between present and absent and everything built on top of it would flap too. If both used the long timer, entry detection would lose its purpose.

Per-frame classification cannot express this difference — it applies the same criterion to every frame. A state machine can, because the guard belongs to the edge, to the transition from one state to another, and the two directions are naturally separate edges. The same applies to the fall height threshold (40–70 cm, default 40 cm) together with dwell time: they form the guard of one directed edge, while recovery to standing is a different edge.

One documentation inconsistency should be recorded rather than resolved here: the user manual §6.1 says "present to absent, about 30 s", while datasheet §6.2 gives "absence detection time ≤40 s". Both describe the same transition, with different values.

## A timer is part of the state, not a delay

The ≥30 s stabilisation period after power-on is often read as "slow to start". From a state-machine perspective it is an **initialisation state**: the internal circuitry must be fully reset and ambient noise adequately assessed before output parameters are valid. It has its own entry condition (power applied) and exit condition (timer elapsed and assessment complete). Output during that state should be discarded, not treated as a valid state by upper layers.

Fall handling follows the same pattern. After 30 s below the threshold triggers the first alarm, the device does not return immediately to idle — it keeps checking for two minutes. That is a "continued observation" state, not a one-shot boolean.

`trajectoryExistenceTime` (unit: s; examples in the protocol are 5 and 10) illustrates another use: when a target is temporarily lost, the trajectory state is retained for an interval. Without that timer, a single occlusion would be interpreted as "target disappeared, new target appeared", and multi-target identity would keep splitting.

The common point is that **a timeout is an event and a timer is part of the state**. Every entered state therefore needs an explicit exit path: how an alarm is cleared, how a target is removed, how initialisation ends. A state machine missing an exit path gets stuck — something to check edge by edge at design time.

## What you configure is the guard, not the state

The MQTT protocol exposes a set of readable and writable parameters: `fallDuration`, `residentWarningDuration` and `lowPostureDuration` (each 0–65535 s), `fallHeightThreshold` (40–70 cm, default 40 cm), installed height (default 240 cm, range 200–300 cm), detection range, region labels, track switch and real-time headcount reporting interval.

All of them modify **transition conditions**; the state set itself does not change. That also explains why adjusting a threshold changes the false-alarm rate rather than the recognition capability. The documentation is direct about `fallHeightThreshold`: raising it increases the fall reporting rate but may increase false alarms. The guard is relaxed, so transitions happen more easily — nothing more. Shortening `fallDuration` works the same way, and ordinary night-time get-ups begin to be classified as falls.

A state machine also needs an **external reset entry point**. `clearNumberOfPeopleInfor` exists for exactly that: after installation, the room is emptied and the command is issued to calibrate the headcount state. `clearLabel` clears label configuration. Without a reset path, one incorrect headcount or label state would propagate into every subsequent decision. Providing an explicit reset is a baseline requirement for keeping a state machine maintainable in the field.

## The cost of edge-triggered reporting: state alignment

Reporting on state change means the device transmits only on transitions and does not periodically repeat its current value. Traffic stays low and the semantics are clear. The cost is that **losing one message leaves you permanently out of sync**: if a transition happened during a network outage, the local mirror still holds the old state after reconnection and the device will not replay it.

Integrators therefore need to maintain their own state mirror and realign on reconnect — through queries of the reported properties, heartbeats and online status — rather than assume "no message means no change". At the device layer, `online` and `heartBeat` are two independent fields answering "is the link reachable" and "is the radar itself operating" respectively; conflating them makes state reconstruction unreliable.

## False-alarm control is explicit, not adaptive

The published protocol shows no baseline learning or per-household training mechanism. False alarms are controlled by three explicit means: dwell timers (30 s, 5 min, 25–120 min), thresholds (40–70 cm), and region labels (inside bed / outside bed / doorway). Their shared property is that they are **auditable, reproducible and adjustable per room**: the same input under the same configuration always produces the same transition.

One entry in the protocol table of contents, `timeForEliminatingFalsePositives`, is marked "second-phase implementation", yet the body of the document gives query and set examples for the property (example value 255). This is an internal inconsistency; until it is clarified, it should not be treated as a currently available capability.

## Summary

The role of state-machine principles in radar algorithm design comes down to four points:

1. **Memory buys stability.** Replacing per-frame classification with guarded transitions turns a continuous, noisy input into discrete, actionable output.
2. **Layers stay orthogonal.** Separate machines for device, presence, motion, posture, region, sleep and headcount avoid combinatorial explosion and let each layer transition independently.
3. **Time carries confidence.** Dwell timers and asymmetric hysteresis do most of the false-alarm work, converting "can it classify accurately" into "has it waited long enough".
4. **Guards and resets stay explicit.** Configurable transition conditions and resettable states make behaviour tunable, inspectable and reproducible.

The corresponding engineering cost is edge-triggered reporting: the caller must maintain a state mirror and handle realignment after reconnection. That is the most easily overlooked division of responsibility hidden behind the phrase *reported on state change*.

## Checklist

- [ ] Local state mirror maintained for every property reported on change
- [ ] Reconnect path rebuilt from queries and heartbeat, not from silence
- [ ] `online` and `heartBeat` treated as separate signals
- [ ] Output during the ≥30 s power-on initialisation discarded
- [ ] Guard timers reviewed per installation, not left at defaults
- [ ] Every entered state checked for an explicit exit and reset path
- [ ] Threshold changes evaluated for false-alarm impact before deployment

## Where to go next

- Why those seconds exist at all: [Why mmWave Radar Takes Seconds to Respond]({{ site.baseurl }}{% post_url 2026-10-17-radar-latency-explained %})
- How false alarms are classified: [Why Fall Detection Is Harder Than It Looks]({{ site.baseurl }}{% post_url 2026-10-17-fall-detection-failure-modes %})
- Hysteresis around zone boundaries: [Coordinate Jitter in Elderly Care and Whole-Home Sensing]({{ site.baseurl }}{% post_url 2026-10-18-radar-position-drift %})
- Sparse payloads, QoS and last will: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- Two health signals, two meanings: [Online Says It's Connected, HeartBeat Says It Works]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %})
- The product whose protocol this article reads: [Multi-Function Whole-Home AI Monitor (HT-W)]({{ site.baseurl }}{% post_url 2026-11-03-split-the-room-not-the-clock %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

## A note on what this is

This is engineering documentation based on the public descriptions in the HT-W product manual, the Whole-Home Guardian Wi-Fi communication protocol document V1.7, the R60BHC1 user manual v1.2 and the R60BHC1-V datasheet V1.1. It reasons backwards from the output protocol to state-machine design; it is not a description of the firmware's internal implementation. The two places where the documents give different values are noted rather than resolved. mmWave sensing carries an inherent false-alarm probability; nothing here is a medical claim, a safety certification, or a performance guarantee for a particular installation.
