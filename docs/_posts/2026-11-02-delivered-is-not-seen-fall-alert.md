---
layout: post
title: "Delivered Is Not Seen: What a Fall Alert Needs After the Message Arrives"
date: 2026-11-02 08:00:00 +0800
categories: [guide]
tags: [alerting, escalation, acknowledgment, elderly care, reliability]
description: "A fall alert has three states — delivered, seen, answered — and only two of them can be observed. What to design for once the message has already reached the phone."
---

<!-- filename: 2026-11-02-delivered-is-not-seen-fall-alert.md -->

The previous article in this series was about getting a fall alert onto the right phone: why the fallback layer should be SMS and a placed call rather than a push notification, and what each channel guarantees on the way there.

This one picks up where that left off, at the moment the message has already arrived. Because arriving is not the same as being noticed, and being noticed is not the same as someone taking responsibility. Most alert systems that fail in the field do not fail at delivery. They fail here.

## Three states, and only two of them are observable

A single alert passes through three states:

1. **Delivered** — the message reached the device.
2. **Seen** — a person actually looked at it.
3. **Answered** — a person took responsibility for it.

The first and the third can be observed. The middle one cannot, and this is the single most important thing to accept before designing anything.

**Delivered** is observable because the operator network reports it: a delivery report for SMS, an answered or unanswered state for a call. Both come from outside your system and both are real.

**Answered** is observable only because you built a way to record it — an acknowledge action, a digit pressed during a call, a reply. It exists because you decided it should.

**Seen** has no signal at all. Nothing in any messaging channel tells you a human being read a thing. Read receipts are a partial exception and a weak one: they report that an app was opened, not that a person understood anything, and they are not available on the channels a fall alert actually uses.

So the practical consequence is this: **you cannot measure attention, so you must design for a proxy.** The only trustworthy proxy is an acknowledge action. Everything in the rest of this article follows from that one sentence.

## Why "seen" fails

It is worth being concrete about the ways an alert arrives and is not noticed, because they are ordinary and they are not exotic failures:

- **Notification grouping.** Smartphones sort messages into categories. A message that is not from a person tends to land somewhere other than the place its owner looks first.
- **Focus and quiet modes.** Scheduled automatically overnight — which is a large fraction of the hours a fall happens in.
- **The phone is not with the person.** Charging in another room, face down, on silent.
- **The alert went to the wrong member of the household.** One device was enrolled, several people live there, and the one who was enrolled is not the one who is available.
- **Several people were notified, and each assumed another was handling it.** When a group is told, responsibility spreads across the group and can end up nowhere. This is a well-documented pattern in team coordination, and a family is not immune to it.

None of these is a channel problem. Every one of them happens *after* a perfectly successful delivery. That is why a better channel does not fix them, and why they need to be designed against rather than engineered away.

## Alert fatigue: the one failure that disables every channel at once

Every channel discussed in the previous article — SMS, a placed call, push — works on the assumption that the recipient treats an incoming alert as worth attention. That assumption is the load-bearing part, and it is the part that breaks.

If alerts arrive often enough that they stop meaning anything, the recipient stops looking. Once that happens, **every channel fails simultaneously.** It does not matter that the SMS was delivered by a carrier-grade store-and-forward network, or that the call was placed by a retry queue you tuned carefully. A message nobody opens is indistinguishable from a message that never arrived.

This is the reason to treat false alarms as an alerting problem and not only a sensing problem. The sources of false alarms, and what has been done about them at the algorithm layer, are covered in [Real Demand, Hard Physics]({{ site.baseurl }}{% post_url 2026-10-19-fall-alarm-market-demand-false-alarms %}). This article is about the other half: what a false alarm does to the delivery chain downstream.

Two design rules follow:

**The value of an alert is its rarity, not its volume.** An alert that fires ten times a day carries almost no information on the eleventh. An alert that fires once a month is read immediately.

**Severity should control intrusiveness, not the reverse.** Not every event deserves a phone call. A static-dwell or out-of-bed notice is not the same class of event as a fall. If everything arrives at maximum urgency, urgency stops being a signal.

## The cancel path: an alert needs a way to be withdrawn

An alert that cannot be withdrawn is an alert that will eventually be muted.

There are two places a cancellation can come from, and a complete design has both:

**From the person on the floor.** If the resident is fine, they should be able to say so before anyone is disturbed. On the finished fall radar this already exists: the unit speaks after a fall and accepts a spoken reply, and a spoken confirmation stands the local alarm down — see the [fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}) for the phrase set and its limits. If you build on a module instead, the local cancel has to be your own hardware or firmware.

**From a person who received it.** Whoever gets the message needs a way to say "I am handling this" — a digit pressed during the call, a reply, an acknowledge in whatever interface you ship.

Two things are easy to get wrong here:

**A cancellation that nobody else hears about is not a cancellation.** If four people were notified and one acknowledges, the other three are still worried. Notify the rest that it is handled. This is cheap and it is the difference between a system people trust and one they find exhausting.

**The window has to be long enough to be usable and short enough to matter.** Too short and nobody can react before the escalation fires. Too long and the escalation is pointless. This is a product decision that belongs in configuration, visible to whoever commissions the installation — not a constant buried in your service.

## Escalation parameters: what actually decides whether someone acts

Escalation is usually described as a sequence: notify, wait, escalate. The interesting decisions are all in the parameters.

**Escalate on the absence of an acknowledge, not on elapsed time alone.** Elapsed time is a fallback for when your acknowledge mechanism is unavailable. If you have an acknowledge mechanism, the absence of one within the window is the real trigger, because it is the one signal that means "nobody has taken this".

**Parallel notification spreads responsibility; serial escalation concentrates it.** Telling everyone at once is fast, and it is the right choice for the first tier — cheap channels, low intrusion, everybody informed. But if nobody responds, continuing to tell everyone at once rarely helps. Moving to one person at a time, in a defined order, means someone specific is being asked. Stop at the first answer.

**Intrusiveness should climb one step at a time.** A text that nobody reads is not a reason to immediately place a call; the call is the step after the text has had a fair chance. Escalating straight to the most intrusive channel burns the channel's effectiveness for the genuine emergency.

**Decide in advance where escalation stops.** Every loop needs a terminus: a maximum number of attempts, a maximum number of people, or both. Without a ceiling, a misconfigured sensor turns into a repeated interruption, and repeated interruption is exactly what produces the fatigue described above.

**Say who is responsible when nobody answers.** If the chain reaches its terminus with no acknowledge, that state needs to be visible somewhere — a record, an entry, an unresolved item — rather than simply ending. An alert that nobody saw and an alert that nobody acknowledged are the same outcome, and both should leave a trace.

## Three kinds of receipt, and which one you can trust

Worth separating clearly, because they get conflated and the conflation produces bad logic:

| Receipt | Where it comes from | What it actually proves | Trust it? |
|---|---|---|---|
| **Delivery receipt** | The operator network | The message reached the handset | Yes |
| **Read receipt** | The receiving application, when one exists | An app was opened | No — not evidence of comprehension, and not available on SMS or calls |
| **Acknowledge** | Your own design | A person acted | Yes — the only one you control |

The delivery receipt tells you your half worked. The acknowledge tells you the whole loop worked. The read receipt tells you very little, and on the channels that matter for a fall alert it does not exist at all.

The practical rule: **build your logic on delivery receipts and acknowledges.** Anything that depends on inferring attention from silence is guessing.

## Monitoring the chain itself

One alert failing is a small problem. The chain failing silently is a large one, because it looks identical to a quiet day.

The counters worth keeping, and they are all simple:

- **Alerts emitted** — how many events your system decided to tell someone about
- **Delivered** — how many the network confirmed
- **Acknowledged** — how many a person took responsibility for
- **Time to acknowledge** — the distribution, not the average
- **Unresolved** — reached the end of the chain with nobody answering

A collapsing acknowledge rate is a much earlier warning than a complaint from a customer. It is the number that tells you the system is on its way to becoming background noise.

Two related traps, both covered in detail elsewhere:

- **Do not conclude anything from silence.** Absence of messages is the normal state of a working installation, because these radars report on change. See [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %}).
- **The alerting service is itself a single point of failure.** A device that has lost its network link cannot tell you, and neither can a forwarding service that has stopped. Treat the health of the chain as its own monitored object, with the same discipline described in [Online and HeartBeat]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %}).

## Checklist

- [ ] The three states written down separately, and no logic anywhere treats delivered as seen
- [ ] An acknowledge mechanism that exists on every channel, and requires no app install
- [ ] Escalation triggered by absence of acknowledge, with elapsed time only as fallback
- [ ] First tier parallel, later tiers serial, stopping at the first answer
- [ ] Intrusiveness matched to event class — not every event becomes a call
- [ ] A local cancel path, in hardware or firmware, before anything leaves the building
- [ ] Cancellations propagated to everyone who was notified
- [ ] A defined terminus: maximum attempts and maximum contacts per event
- [ ] Unresolved alerts recorded, not silently dropped
- [ ] Counters for emitted, delivered, acknowledged, time-to-acknowledge, unresolved
- [ ] The alerting service monitored as its own failure domain

## Where to go next

- The other half of the chain, choosing the channel: [What SMS and Voice Calls Give a Fall Alert That Push Notifications Cannot]({{ site.baseurl }}{% post_url 2026-11-01-sms-voice-alert-delivery-fall-radar %})
- Where false alarms come from, and what has been fixed: [Real Demand, Hard Physics]({{ site.baseurl }}{% post_url 2026-10-19-fall-alarm-market-demand-false-alarms %})
- The hand-off itself, including the local voice confirmation: [the fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %})
- Escalation across three radars in one home: [Building an Aging-in-Place Care System]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %})
- Device health before you alert on it: [Online and HeartBeat]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

## A note on what this is

A fall radar is a sensing device. It does not diagnose, does not treat, does not summon emergency services, and does not replace human checks or a call system. It detects motion it cannot promise to interpret correctly in every room, and it will produce false alarms.

Everything above concerns what happens after a reported event reaches a person. Good alert design makes sure a real event is noticed and owned. It cannot make an undetected event appear, and no amount of escalation logic improves the sensing underneath it.

Evaluation units and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
