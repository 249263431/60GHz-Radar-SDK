---
layout: post
title: "Feeding a PLC From a 60 GHz Fall Module: GP1 as an Opto-Isolated Dry Contact"
date: 2026-10-21 10:30:00 +0800
categories: [guide]
tags: [mmwave, radar, gpio, plc]
---

PLCs and building-management systems in care homes and commercial buildings do not speak MQTT or JSON. They want a **dry contact** — a switch that closes. The 60 GHz fall module (**R60AFD1-V**) gives you exactly that opportunity through **GP1** (active-high: High = fall detected). With one optocoupler you turn GP1's 3.3 V logic into an isolated contact the PLC can read, with **no microcontroller and no network** in between.

This is the module, not the finished Wi-Fi radar. The finished unit has no exposed GPIO.

## What GP1 is, and why isolation matters

| Pin | Level | Meaning |
|---|---|---|
| GP1 | **High = fall detected**, Low = clear | Active-high fall alarm |
| GP2 | High = someone present, Low = empty | Occupancy (optional) |

Two properties drive the design:

- **GP1 is a 3.3 V TTL logic output, weak (mA-level).** It cannot source a PLC input directly, and it must never see the PLC's 24 V.
- **The PLC side is typically 24 V and a different ground domain.** Connecting the two grounds directly invites ground loops and a 24 V fault that destroys the 3.3 V module.

An **optocoupler** solves both: the radar's GP1 drives the opto's LED; the opto's phototransistor switches the PLC input. The two sides share no electrical connection.

## Bill of materials

| Qty | Part | Notes |
|---|---|---|
| 1 | R60AFD1-V module | 60 GHz fall module, UART + GPIO |
| 1 | Optocoupler, e.g. **PC817 / LTV-817** | NPN phototransistor output, VCEO ≥ 35 V |
| 1 | Resistor **220–330 Ω** | Limits LED current from GP1 (3.3 V) |
| 1 | Pull-up **4.7 kΩ – 10 kΩ** | For the PLC side, if required by the input type |
| 1 | 5 V supply, ≥ 500 mA | Module peaks at 300 mA; ripple ≤ 100 mV |
| — | PLC digital input (24 V), wires | Sinking or sourcing per your PLC |

The phototransistor in a PC817 comfortably handles the few mA a PLC digital input draws, and its 35 V rating covers 24 V.

## Wiring (sinking PLC input — the common case)

Most PLC digital inputs are *sinking*: they sink current from an external 24 V through the field device.

```
   Radar side (3.3 V)                 PLC side (24 V, isolated)
 ┌──────────────┐
 │ R60AFD1-V    │
 │ GP1 ─┐       │
 │      ├──▱▱───┤─── opto LED anode (220–330 Ω)
 │   GND ├──────┤─── opto LED cathode
 └──────────────┘         │
                    opto phototransistor
                    collector ───► 24 V (via 4.7k pull-up if needed)
                    emitter   ───► PLC digital input (Ix)
                                    PLC COM / 0V ───► 0 V
```

Plain-text steps:

1. Power the module from **5 V** (4.5–5.5 V, ripple ≤ 100 mV); radar GND is its own domain.
2. **GP1** → 220–330 Ω → opto **LED anode**; LED **cathode** → radar **GND**.
3. Opto **phototransistor collector** → PLC **24 V** (or through a 4.7 kΩ pull-up to 24 V, depending on the input).
4. Opto **phototransistor emitter** → PLC **digital input (Ix)**.
5. PLC **COM / 0 V** → 0 V.

When GP1 goes High, the LED lights, the phototransistor conducts, and the PLC input energizes — your ladder logic sees a fall event and can trigger a call system, lighting, or HVAC. When the alarm clears (GP1 Low), the contact opens. **The two grounds never connect.**

If your PLC inputs are *sourcing* instead, swap the polarity: emitter → 0 V, collector → PLC input, and the PLC provides the 24 V internally. The opto still isolates.

## Behavior and tuning

- **Boot:** allow **≥ 30 s** after power-on before trusting the output.
- **On a fall:** GP1 rises; the PLC input turns on within milliseconds.
- **Alarm duration:** fall report defaults to **15 s**; for production use **≥ 60 s** to reject someone who stumbled and recovered (configurable 5–180 s over UART).
- **Static-dwell (optional):** a second stage after long stillness; default 300 s, production **≥ 30 min**.

## Add GP2 to arm the contact

Feed **GP2** (High = someone present) into the loop so the PLC only registers a fall when a person is actually in the coverage area — preventing a stale trigger with the room empty.

## Caveats

- **Power quality first.** A weak or noisy 5 V supply is the #1 cause of unstable detection.
- **False alarms happen.** Moving non-living objects (fans, curtains, flowing water) can trip it; a side-lying person may be missed. Use GP1 for a fast local/open-contact signal and UART/MQTT for logging; design redundancy.
- **Not a medical device.** A sensing building block, not a life-safety system; not for diagnosis or treatment.

## Module specifications (R60AFD1-V datasheet)

Every figure below is taken from the R60AFD1-V datasheet v2.1 (MicRadar / 云帆瑞达). Nothing here is assumed.

**Interface 1 — the connector you actually use (2×3, 2.0 mm pitch):**

| Pin | Signal | Level | Notes |
|---|---|---|---|
| 1 | 5V | 5.0 V | Power input + |
| 2 | GND | — | Ground |
| 3 | RX | 3.3 V TTL | UART receive |
| 4 | TX | 3.3 V TTL | UART transmit |
| 5 | GP1 | 3.3 V / 0 V | **High = fall alarm, Low = clear** |
| 6 | GP2 | 3.3 V / 0 V | High = occupied, Low = empty |

Interface 2 (2×4) is the programming/debug port: 3V3, GND, SWC, SWD, GP3–GP6. GP3–GP6 are spare and user-redefinable.

**Electrical (operating):**

| Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|
| VCC | 4.5 | 5.0 | 5.5 | V |
| Average current (ICC) | — | — | 100 | mA |
| Peak current (Ipeak) | — | — | 300 | mA |
| Operating temperature | −20 | — | +70 | °C |
| Storage temperature | −40 | — | +85 | °C |

Supply ripple must be **≤ 100 mV**; the supply must deliver enough current and transient response for the 300 mA peak.

**RF and detection:**

| Parameter | Value |
|---|---|
| Frequency | 61 – 62.5 GHz |
| Tx power | 8 dBm typ, 10 dBm max |
| Antenna gain | 8 dBi |
| Detection radius (motion / fall / static dwell) | 1.5 m typ @ 2.8 m ceiling |
| Beam angle (horizontal / pitch) | 100° / 100° |
| Motion output latency | ≤ 100 ms |
| Nobody → nobody-absent output | 10 – 60 s |
| Fall detection latency | 5 – 20 s (15 s typ) |

**Install and configuration:**

- Ceiling mount, vertical down; install height **2.2 – 2.8 m** (default height setting **2.3 m** — must match the actual height).
- Fall report time: default **15 s**, adjustable **5 – 180 s**; normal projects use **≥ 60 s**.
- Static-dwell report time: default **300 s**; normal projects use **≥ 30 min**.
- Power-on stabilization: **≥ 30 s** before trusting the output.
- Module volume: 35 × 31 × 7.5 mm. Protocol: SIP-S V1.0 (generic UART) and the Tuya standard protocol.

The key point for this article: GP1 is a **3.3 V logic output, not 24 V** — which is exactly why the optocoupler is required between it and the PLC.

## Where to go next

- The GPIO/UART split: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}).
- Local relay/buzzer from GP1 (no PLC): [Local fall alarm]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}).
- Full specs: [Module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- Fall tuning: [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}).

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
