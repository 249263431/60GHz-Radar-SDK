---
layout: post
title: "Occupancy Lighting and HVAC From a 24 GHz Radar Module: Switching a Relay With S1, No Gateway"
date: 2026-10-21 11:00:00 +0800
categories: [guide]
tags: [mmwave, radar, gpio, occupancy]
---

The 24 GHz presence module (**R24AVD1**) reports whether a space is occupied as a single logic pin: **S1 goes High when someone is present and Low when empty**. Wire S1 to a relay and you get occupancy-controlled lighting or HVAC with **no Wi-Fi, no cloud, no gateway** — the radar switches your load directly. This reuses the same transistor/relay topology as the fall-alarm article, applied to a lower-cost 24 GHz part.

## What S1 (and S2) are

| Pin | Level | Meaning |
|---|---|---|
| S1 | **High = occupied**, Low = unoccupied | Presence |
| S2 | High = active (moving), Low = static | Motion vs stillness |
| GP1–GP4 | redefinable | — |

Key behaviors from the module's protocol:

- Occupied → unoccupied is reported after a **40 s** stillness delay; occupied appears in **< 0.5 s**.
- **Sensitivity (DP5, 1–10, default 7)** and **scene mode (DP6)** tune range and false-trigger rejection. Use a lower sensitivity in corridors/parking where curtain or plant sway would otherwise false-trigger.
- **Power the module per its datasheet** (3.3 V I/O; VCC and current are specified there). The relay/load side uses its own 5 V/12 V supply, kept separate from the module.

## The same rule as always

> S1 switches a transistor; the transistor switches the load. S1 never touches the load.

The pin is 3.3 V TTL and weak (mA-level) — never drive a relay or lamp directly.

## Bill of materials

| Qty | Part | Notes |
|---|---|---|
| 1 | R24AVD1 module | 24 GHz presence module, UART + GPIO |
| 1 | Logic-level N-channel MOSFET (e.g. **AO3400**, **IRLZ44N**) | Gate on at 3.3 V; NPN + 1 kΩ also works |
| 1 | 5 V / 12 V relay (or contactor for HVAC) | Rating must exceed the load |
| 1 | Flyback diode (**1N4007**) | Across the relay coil |
| 1 | 10 kΩ resistor | Gate pull-down |
| 1 | Supply for the module (per datasheet) + supply for the load | Separate domains |
| — | Wires, enclosure | Non-metal, flat face; 3–5 mm from antenna to shell |

## Wiring

```
   R24AVD1 (3.3 V)                    Load side (5 V / 12 V)
 ┌──────────────┐
 │ S1 ─┐        │
 │     ├─▱▱─────┤── 10 kΩ pull-down ──► MOSFET gate
 │ GND ├────────┤────────────────────► MOSFET source ──► load supply 0 V
 └──────────────┘    MOSFET drain ──► relay coil ──► load +V
                                 1N4007 (cathode to +V, anode to drain)
                                 NO contacts ──► light / HVAC ──► 0 V
```

1. Power the **module** from its specified supply; power the **relay/load** from its own 5 V/12 V supply.
2. **S1** → 10 kΩ pull-down → **MOSFET gate**.
3. **MOSFET source** → load-supply **0 V**.
4. **MOSFET drain** → relay coil → load **+V**; flyback diode across the coil.
5. Relay **NO** contacts switch the light or HVAC circuit.

When S1 is High, the relay closes and the load runs; when the space goes empty (after the 40 s delay), it opens.

## Using S2 to avoid "lights off while still"

S2 is High only when the person is **moving**. If you drive the relay from S1 alone, a person who sits still keeps the lights on (good). If you prefer "on only while moving," use S2 instead — but then a still person darkens the room. Most installs want **S1** for occupancy and leave S2 for UART-based analytics.

## Mounting and false-trigger reality

- **Ceiling mount, height ≤ 2.75 m, tilt ≤ 3°**, long side of the board toward the door. Keep the antenna 3–5 mm from the enclosure's inner surface.
- **Known false-alarm sources:** fans, pets near the radar, swinging metal curtains, and the internal motor of an air conditioner directly in the beam. Aim the beam away from these.
- **Not a medical device.** A sensing building block; not for diagnosis or treatment.

## Where to go next

- The GPIO/UART split: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}).
- Same topology on the fall module: [Local fall alarm]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}).
- Full specs: [Module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- Presence tuning: [Presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}).

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
