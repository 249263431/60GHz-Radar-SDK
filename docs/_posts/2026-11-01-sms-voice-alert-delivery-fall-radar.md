---
layout: post
title: "What SMS and Voice Calls Give a Fall Alert That Push Notifications Cannot"
date: 2026-11-01 08:00:00 +0800
categories: [guide]
tags: [alerting, sms, escalation, elderly care, reliability, fall detection]
description: "Step five of the fall alarm chain is yours to build. Why the fallback layer should be SMS and a placed call rather than push, what each channel actually guarantees, and what neither can solve."
---

<!-- filename: 2026-11-01-sms-voice-alert-delivery-fall-radar.md -->

Every fall-alarm article on this site ends the same way: **step five is yours.** The radar detects the collapse, waits out its configured delay, publishes `fallStatus`, and on the finished units even speaks to the person on the floor. Then it stops. Who gets told, how, and what happens when nobody answers is not in any datasheet, because none of it belongs to the sensor.

This article is about that fifth step — specifically the last leg of it, getting the alert onto the right phone. The argument is unfashionable but practical: **the fallback layer of a fall alert should be SMS and a placed call, not a push notification.** Here is what each approach actually buys you, and what neither ever will.

> This is engineering guidance for integrators. It is not a medical claim and not a safety certification. See the closing note.

## Step five is a chain, not a message

"Escalation" gets drawn as one box in an architecture diagram, and then implemented as one call in code — `send_notification()`. It is really three legs, and they fail differently:

1. **Transport** — the alarm travels from the radar to your backend (MQTT).
2. **Reach** — your backend gets the alert onto the recipient's device.
3. **Acknowledgment** — a human learns about it and takes responsibility.

Most engineering effort goes into leg 1, because it is the leg the datasheet describes. Most field complaints come from legs 2 and 3. "The radar missed it" is very often "the radar reported correctly and nobody found out."

And for our part of the chain, the detail that changes everything: **the buyer is usually not the user.** An adult child installs this for a parent. The question is not "will our users install our app" — it is whether the person who must be reachable has done their part before the one day it counts.

## Two networks, not two vendors

The choice between push and telecom is usually framed as picking a service provider. It is not. It is picking which **network** carries the last leg.

| | App push | SMS / placed call |
|---|---|---|
| Carried on | The internet — Wi-Fi or mobile data | The operator's signalling and voice networks |
| Needs a working data connection | Yes | No |
| Requires an app installed | Yes | No |
| Requires notification permission | Yes | No |
| Requires an account on the recipient's side | Yes | No |
| Who retries when the device is unreachable | You, or nobody | The operator, for SMS |
| Delivery evidence | Vendor receipt | Operator delivery report / answered-call state |

Two consequences follow directly, and they are the whole argument:

**Push asks something of the recipient first.** There must be an installed app, a granted permission, and a live session with a push service before your alert has anywhere to land. Every one of those is a condition you do not control.

**An alert path that only works when everything else is also working is not a fallback.** It is a second copy of the same dependency.

## What the telecom path gives you

Six properties, roughly in order of how much they matter here.

### 1. Nothing is required of the recipient

A phone number is enough. No install step, no sign-up, no permission granted weeks earlier, no app version quietly gone stale. A number works on a decade-old handset and for a relative who visits once a year and appears in nobody's contact graph.

It also removes the most expensive class of support ticket: the one where the alert never arrived because notifications were muted, an app was signed out, or an account was never created. None of those failure modes exist on a path that only needs a number.

### 2. It does not depend on a data connection

SMS travels on signalling; a call travels on the voice network. Neither needs an active IP session. The cases this covers are ordinary ones:

- The recipient has **no data allowance left** this month.
- The recipient is somewhere with **one bar of signal** — enough for a text and a call, not enough for anything else to load.
- The phone is in **battery-saving mode**, which throttles background data and leaves calls alone.

### 3. The retry belongs to someone else

This is the one property no application-layer approach can replicate. SMS is **store-and-forward**: if the handset is off, out of coverage, or underground, the message is held by the network and delivered when the device returns.

It is time-limited — the retention window is set by the operator and by your provider's configuration, so **confirm it rather than assume it**. But the retry loop runs inside the operator's infrastructure, unattended, whether or not your server is awake. With push, the same offline device is your problem, and whatever the platform holds, it holds briefly and on conditions you do not set.

Know the asymmetry going in: **a call has no store-and-forward.** An unanswered call is simply unanswered. You build the retry queue, you choose the intervals, and you decide how many contacts deep it goes.

### 4. An answered call is an acknowledgment

Push and SMS can both report "delivered". Neither tells you a human engaged. An alert sitting unread in a notification tray and an alert nobody received are indistinguishable from your backend's point of view.

An answered call is different in kind: a person picked up. That is the closest thing to proof-of-reach available without shipping an app, and it is why the call belongs at the *end* of the chain rather than the start.

You can extend that signal to everyone rather than only the first person to answer, and still ship no application: **prompt for a digit press during the call and treat it as an acknowledge.** Works on any handset that can receive the call, needs no account, and gives your backend a real event to record.

### 5. Delivery you can measure

SMS returns a delivery report; a call returns answered or unanswered. Both come from the operator's side. So your escalation logic can be built on **evidence** instead of a timeout guess:

```
fall event arrives ──> SMS to all contacts in parallel
                  ──> read delivery reports
                  ──> wait a bounded window for an acknowledge
                  ──> none received: place calls in order, stop at first answer
                  ──> nobody answered: next contact, bounded retries
```

Every branch above is triggered by something arriving. That is worth contrasting with the alternative, where "probably delivered" is the strongest signal you get and silence has to be interpreted.

### 6. The alerting logic stays yours

When the notification path is somebody else's app, the alert experience — who is told, in what order, what happens when nobody responds — is largely defined by that platform. When you place SMS and calls from your own service, the contact list, ordering, retry policy, and audit trail are software you own.

For an integrator that is the difference between a feature you tune per customer and a feature you request. It is also the part that can be priced and supported, rather than inherited.

## What it looks like assembled

Cheapest and most reliable first, escalating only on the absence of an answer.

**Layer 0 — on site, with or without a network.** Part of this already exists in hardware. The finished fall radar speaks after a fall and accepts a spoken reply, so a resident who says "I am OK" stands the local alarm down — see the phrase set and its limits in the [fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}). If you are building on a module instead, **R60AFD1-V** raises `GP1` the moment a fall is concluded, which is enough to drive a local buzzer or relay with no parser at all — see [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}) and [driving a relay from GP1]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}). Know the variant difference up front: the **24 GHz fall module has no hardware alarm output** — its `S1`/`S2` are presence and motion only, so for that variant Layer 0 has to be your firmware's job.

Keep this layer even though it looks redundant. It is the only one that still works when the home router has failed, which is not an exotic scenario.

> **Verify this one on the bench.** In the documented alarm chain, `fallStatus` is published *before* the local voice step. Confirm what your broker actually sees when a resident cancels locally — if that cancellation does not reach your system, then the stand-down decision lives entirely in your own logic, which is exactly why Layer 2 exists.

**Layer 1 — reach.** SMS to every contact, in parallel.

**Layer 2 — acknowledgment.** Wait a bounded window. No acknowledge → place calls, first contact first, stopping at the first answer.

**The acknowledge mechanism.** Do not skip it. Without a way for the receiving person to say "I am handling it", carers learn to treat alerts as background noise, and that defeats every layer above. The digit press during the call is the cheapest version that needs no app; a reply keyword works too, if your number can receive.

Set the wait window honestly: long enough that someone can read a text and respond, short enough to still be useful. It is a product decision and belongs in the configuration, visible to whoever commissions the installation.

## The first half of the chain is not immune

Everything above assumes the fall reached your backend. Worth a short section, because a telecom fallback tends to get blamed for failures upstream of it.

- **Use QoS 1 on fall events** and a persistent session on your consumer, so a restarting backend does not miss the one message that mattered. The settings that actually help are listed in [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %}).
- **Payloads are sparse by design.** Everything is reported on change, so no news is the normal state, not a fault.
- **Device health has a hard limit.** An offline device cannot report that it is offline. `online` is generated by the broker when a session drops, which is real evidence; `heartBeat` can only ever come from a radar that still has a link to send it with. Read [Online and HeartBeat]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %}) before writing any timeout-based logic — it is the clearest treatment of why silence proves nothing, and it lists what to measure at commissioning instead.
- **A network outage removes all of it at once.** No choice of downstream channel recovers an alert that never left the house. Which is the argument for Layer 0.

## What telecom channels do not solve

Stated plainly, because overselling this would be worse than not writing it.

**Reach is not attention.** This article is about getting the alert onto the phone. Whether somebody notices within thirty seconds is a separate problem, solved by escalation design and by keeping alerts rare enough to still mean something — which is a false-alarm question at the sensor end, covered in [Real Demand, Hard Physics]({{ site.baseurl }}{% post_url 2026-10-19-fall-alarm-market-demand-false-alarms %}). A perfect SMS path delivering constant noise is still a product nobody trusts.

**Numbers get filtered.** Operator spam filtering and unknown-call screening are real, particularly for numbers that have carried other traffic before. Use a consistent, registered sender identity, and ask recipients to save the number to contacts during setup. Plan for this rather than discovering it.

**Templates and consent are paperwork.** Signatures and message templates need registration and approval, and voice broadcasting is regulated more tightly than SMS in most places. Get explicit opt-in from the contact at binding time — not buried in terms, but an affirmative "yes, call me for fall alarms". With someone's phone number, that is also simply the decent thing to do.

**Bound your retry policy.** Cost arrives per message and per call minute. Fine at true-fall frequency; not fine if a misconfigured dwell alarm starts placing calls every few minutes overnight. Put a ceiling on attempts per event.

**One regional caveat, stated rather than buried.** How wide the gap really is depends on your market. Where handsets carry a system-level push service that is exempt from power management, an app path can perform well and the argument narrows to "installs are still a barrier". Where handsets ship without one — which covers a large share of Android devices — app push depends on per-manufacturer channels and per-device background policy, and the gap is wide. **Verify this for the markets you actually ship to** rather than taking either sentence from us.

## Checklist

- [ ] Fall events consumed at QoS 1, with a persistent session
- [ ] No logic anywhere concludes anything from the absence of messages
- [ ] Device health driven by `online` and `heartBeat`, respecting their documented asymmetry
- [ ] Layer 0 present: hardware voice reply, `GP1`-driven local output, or equivalent in firmware
- [ ] Local cancellation behaviour verified on the bench, not assumed
- [ ] SMS to all contacts in parallel as the reach layer
- [ ] Bounded acknowledge window, then calls in order, stopping at first answer
- [ ] Acknowledge mechanism that requires no app — digit press or reply keyword
- [ ] Sender number and signature registered, templates approved
- [ ] Explicit opt-in recorded per contact
- [ ] Call retry bounded by maximum attempts per event
- [ ] Whole path tested with the handset switched off, then restored

## Where to go next

- Step five across three radars in one home: [Building an Aging-in-Place Care System]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %})
- The hand-off itself, step by step: [the fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %})
- What genuinely defeats detection: [fall detection failure modes]({{ site.baseurl }}{% post_url 2026-10-17-fall-detection-failure-modes %})
- Getting events reliably to your backend: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- Device health before you alert on it: [Online and HeartBeat]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %})
- The broker underneath: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

## A note on what this is

A fall radar is a sensing device. It does not diagnose, does not treat, does not summon emergency services, and does not replace human checks or a call system. It detects motion it cannot promise to interpret correctly in every room, and it will produce false alarms.

Everything above concerns making sure a reported event reaches a person. That is a communication problem, one layer removed from whether any given fall was caught at all — and no receiving channel, however reliable, improves the detection underneath it.

Evaluation units and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
