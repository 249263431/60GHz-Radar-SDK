---
layout: post
title: "60 GHz / 24 GHz Radar Module Bring-Up Checklist: Power, UART, Mounting, First Good Frame"
date: 2026-10-22 09:00:00 +0800
categories: [guide]
tags: [mmwave, radar, bring-up, gpio]
---

A field checklist to get a module outputting good data on the first try. It covers the **R60AFD1-V** (60 GHz fall), **R24AVD1** (24 GHz presence), and the SIP-S serial family. Every item is taken from the module datasheets / user manuals — nothing assumed. Pair it with the [minimal UART parser]({{ site.baseurl }}{% post_url 2026-10-22-minimal-uart-parser-sip-s %}) to go from "wires connected" to "data decoded".

## 1. Power — do this before anything else

- [ ] Supply **4.5–5.5 V** (typ 5.0 V) for the 60 GHz modules; ripple must be **≤ 100 mV** (R60AFD1-V datasheet §5.2, §9.4).
- [ ] Budget for **≥ 300 mA peak** (R60AFD1-V: average ≤ 100 mA, peak ≤ 300 mA). Use a supply with real current and transient headroom — not a weak LDO.
- [ ] **Ground well.** Radar performance drops and false-alarm rate rises with ground noise; keep the module ground solid and away from noisy loads (R60AFD1-V §9.4).
- [ ] For **R24AVD1**, VCC/current are **not published in its getting-started guide** — power it per its datasheet; only the 3.3 V I/O level is given there.

## 2. Serial / UART

- [ ] **115200 bps, 8 data bits, 1 stop bit, no parity, no flow control** (SIP-S family; R60ABD1 manual §7; R60AFD1-V lists SIP-S V1.0).
- [ ] **3.3 V TTL** levels. Cross-connect: module **TX → MCU/adapter RX**, module **RX ← MCU TX**. Never feed 5 V into the module's RX/TX.
- [ ] Use a 3.3 V USB-TTL adapter (CP2102 / CH340); open a terminal to confirm bytes arrive after boot.
- [ ] Expect the SIP-S V1.0 frame: header `0x53 0x59` … footer `0x54 0x43`, checksum = low 8 bits of the sum of all preceding bytes (see the parser article).

## 3. GPIO (if you use the hard outputs)

- [ ] R60AFD1-V **GP1 / GP2** and R24AVD1 **S1 / S2** are **3.3 V logic, weak (mA-level)**. Never connect a load directly.
- [ ] Drive a transistor/MOSFET, which switches the relay/buzzer/PLC. See [Local fall alarm from GP1]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}), [GP1 → PLC dry contact]({{ site.baseurl }}{% post_url 2026-10-21-fall-gp1-plc-dry-contact %}), [S1 → relay occupancy]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}).
- [ ] Pin meaning: R60AFD1-V **GP1 High = fall alarm, GP2 High = occupied**; R24AVD1 **S1 High = occupied, S2 High = active**.

## 4. Mounting & enclosure

- [ ] **Fall (R60AFD1-V):** ceiling, vertical down; height **2.2–2.8 m**. Set the module's height parameter to the real height (default **2.3 m**) or detection degrades.
- [ ] **Presence (R24AVD1):** ceiling; height **≤ 2.75 m**, horizontal tilt **≤ 3°**; aim the board's long side toward the door.
- [ ] Keep the antenna **3–5 mm** from a **non-metal, flat** enclosure face; keep the radar's own components **≥ 1 mm** above other PCBA parts (R24AVD1 guide §10).
- [ ] Keep metal out of the beam; a curved or metallized shell degrades the scan.

## 5. First-good-frame verification

- [ ] After power-on, wait **≥ 30 s** for the module to stabilize before trusting output (R60AFD1-V §9.1).
- [ ] Confirm a repeating frame stream: header `0x53 0x59`, footer `0x54 0x43`, and a checksum that validates. A heartbeat (control 0x01) is the easiest first sign of life.
- [ ] On the fall module, check **GP1** = 0 V when clear and 3.3 V on a test fall; **GP2** = 3.3 V with a person in coverage.
- [ ] Before field use, set **fall report time** (default 15 s; normal projects ≥ 60 s) and **static-dwell time** (default 300 s; normal ≥ 30 min) per the datasheet.

## 6. Avoiding false alarms (environment)

- [ ] Keep the beam away from: oscillating fans, swinging curtains, flowing liquid, pets near the radar, and **air-conditioner internal motors** — aim parallel to, not at, the AC (R24AVD1 guide §11; R60AFD1-V §10).
- [ ] For presence, pick a **scene mode** (DP6) and **sensitivity** (DP5, default 7) that fits the room; lower sensitivity rejects curtain/plant sway in corridors/parking.

## 7. Safety

- [ ] **Not a medical device.** A sensing building block, not a life-safety system; not for diagnosis or treatment.
- [ ] For any high-reliability use (medical life-support, aerospace, etc.), contact the vendor first and implement **fail-safe / redundant** design (R60AFD1-V §11).

## Where to go next

- The GPIO/UART split: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}).
- Decode the bytes: [Minimal UART parser (SIP-S V1.0)]({{ site.baseurl }}{% post_url 2026-10-22-minimal-uart-parser-sip-s %}).
- Local fall alarm from GP1: [relay/buzzer]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}).
- GP1 → PLC dry contact: [opto-isolated]({{ site.baseurl }}{% post_url 2026-10-21-fall-gp1-plc-dry-contact %}).
- S1 → relay occupancy: [lighting/HVAC]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}).
- Full specs: [Module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- Tuning: [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}), [Presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}).

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
