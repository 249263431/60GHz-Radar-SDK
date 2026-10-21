---
layout: post
title: "Local Fall Alarm From a 60 GHz Radar Module: Driving a Relay and Buzzer From GP1, No Firmware"
date: 2026-10-21 10:00:00 +0800
categories: [guide]
tags: [mmwave, radar, gpio, relay]
---

The 60 GHz fall-detection module (**R60AFD1-V**) already decides, on its own, whether a person has fallen. It exposes that decision as a single 3.3 V logic pin — **GP1 goes High the moment a fall is detected, and Low when the alarm clears**. This article shows how to turn that one pin into a local audible / visual alarm with **no microcontroller and no firmware**: a transistor, a relay, and a buzzer. No serial parser, no broker, no network.

This is the **module**, not the finished Wi-Fi radar. The finished unit has no exposed GPIO; this article is for integrators building the sensor into their own product or a local alarm.

## What GP1 is, exactly

| Pin | Level | Meaning |
|---|---|---|
| GP1 | **High = fall detected**, Low = clear | Active-high fall alarm |
| GP2 | High = someone present, Low = empty | Occupancy (optional here) |

Two hard facts about the pin:

- **It is a 3.3 V TTL logic output.** It swings 0 V / 3.3 V, not 5 V.
- **It is weak (mA-level).** You must never connect a load — relay coil, buzzer, LED string — directly to it. You switch the load through a transistor or MOSFET.

## The one rule that prevents burned boards

> GP1 switches a transistor. The transistor switches the load. GP1 never touches the load.

Nearly every "it doesn't work / the module died" case in this setup comes from skipping that rule.

## Bill of materials

| Qty | Part | Notes |
|---|---|---|
| 1 | R60AFD1-V module | 60 GHz fall module, UART + GPIO |
| 1 | Logic-level N-channel MOSFET (e.g. **AO3400**, **IRLZ44N**) | Gate fully on at 3.3 V. An NPN (S8050 / 2N2222) + 1 kΩ base resistor also works |
| 1 | 5 V relay, **or** active 5 V buzzer | Relay if you want a dry-contact output; buzzer for sound only |
| 1 | Flyback diode (**1N4007**) | Required across the relay coil; not needed for a buzzer |
| 1 | 10 kΩ resistor | Gate pull-down, keeps the MOSFET off at boot |
| 1 | 5 V supply, ≥ 500 mA | Module peaks at 300 mA; ripple ≤ 100 mV |
| — | Wires, breadboard / PCB | — |

If you use an NPN instead of a MOSFET, replace the 10 kΩ gate pull-down with a **1 kΩ base resistor** between GP1 and the base; emitter to GND, collector to the load.

## Wiring

```
        R60AFD1-V                       Load side (5 V)
      ┌──────────────┐
 GP1 ─┤ GP1          │
      │              │         10 kΩ        Gate
      │          GND ├───┬───────▱▱───────┤ G
      └──────────────┘   │               ├─ N-ch MOSFET (logic level)
                         │          D ────┤ D
                         │          S ────┴── GND (5 V supply −)
                         │
                   +5 V supply ──► relay coil ──► D (MOSFET drain)
                                   │
                                1N4007 (cathode to +5 V, anode to D)
                                   │
                        buzzer / NO contact ──► load ──► GND
```

Plain-text steps:

1. Connect the module **5V** and **GND** to your 5 V supply.
2. **GP1** → 10 kΩ pull-down → **MOSFET gate**. Tie the gate to GND through the 10 kΩ so it stays off until GP1 drives it.
3. **MOSFET source** → 5 V supply **GND**.
4. **MOSFET drain** → one end of the **relay coil** (or buzzer +).
5. Other end of the coil / buzzer → **+5 V**.
6. **Flyback diode (1N4007)** across the relay coil: cathode to +5 V, anode to the drain side.
7. The relay's **NO (normally open)** contacts switch your buzzer, siren, light, or any 5 V / 12 V load.

When GP1 goes High, the MOSFET turns on, the coil energizes, and the NO contacts close — the buzzer sounds. When the alarm clears (GP1 Low), the relay releases.

## Expected behavior

- **Power on:** allow **≥ 30 s** for the module to boot and stabilize before trusting the output.
- **On a fall:** GP1 rises to 3.3 V, the relay closes within milliseconds, the alarm sounds.
- **Alarm duration:** the module's fall report defaults to **15 s**; for production use **≥ 60 s** to suppress false trips from someone who stumbles and recovers. (Configurable 5–180 s over UART.)
- **Static-dwell (optional):** the dwell alarm can drive a second stage after long stillness; default 300 s, production **≥ 30 min**.

## Add GP2 for "only alarm when someone is present"

GP2 is High when someone is in the coverage area. You can use it to arm the alarm: feed GP2 into the relay coil's supply path (or a second transistor) so the buzzer only sounds when a person is actually present. This prevents the relay from latching on a stale state with nobody in the room.

## Power and false-alarm reality

- **Power quality matters most.** 4.5–5.5 V, ripple **≤ 100 mV**, solid ground. A weak or noisy supply is the #1 cause of unstable detection.
- **False alarms are possible.** Moving non-living objects (fans, swinging curtains, flowing water) can trip the sensor; a person lying on their side with the chest not facing the radar may be missed. Design redundancy: use GP1 for a fast local alarm and UART / MQTT for logging and confirmation.
- **Not a medical device.** This is a sensing building block, not a life-safety system. Do not use it as the sole safeguard; not for diagnosis or treatment.

## Where to go next

- The GPIO / UART split and the full data set: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}).
- Full module families and specs: [Module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- Fall detection tuning and expectations: [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}).

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
