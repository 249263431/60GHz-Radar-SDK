---
layout: post
title: "Powering and Enclosing a 60GHz Radar Module: Ripple, Grounding, and Antenna Keep-Out"
date: 2026-10-24 08:00:00 +0800
categories: [guide]
tags: [power supply, EMC, enclosure, mmwave radar]
---

The [module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}) tells you the electrical and mechanical numbers. This article is about what to do with them, because in our experience the first time a radar product fails a test lab, the cause is almost never the sensor — it is the supply, the ground, or the enclosure sitting too close to the antenna.

Everything below applies to the three bare modules we sell. Where a value differs per module it is stated; where the datasheet is the authority, it says so.

## Power: the window, the current, and the ripple

| Module | Supply | Current | Ripple |
|---|---|---|---|
| R60AFD1-V (fall, 60 GHz) | 4.5–5.5 V, typ. 5.0 V | ≤ 100 mA avg, ≤ 300 mA peak | ≤ 100 mV |
| R60ABD1 (sleep & breathing, 60 GHz) | 4.5–5.5 V, typ. 5.0 V | ≥ 200 mA | ≤ 100 mV |
| R24AVD1 (presence, 24 GHz) | **see datasheet** | **see datasheet** | **see datasheet** |

Three things to take from that table:

- **The 3.3 V figure on the pinout is the I/O level, not the supply.** UART, S1/S2 and the GP pins are 3.3 V TTL logic. Powering the module from 3.3 V because "the pins say 3.3 V" is a classic mistake.
- **Ripple ≤ 100 mV is a real constraint.** A bare switching regulator at the edge of its tolerance will not meet it. Use an LDO after the switcher, or a linear supply where heat allows.
- **Budget for peaks, not averages.** The fall module draws up to 300 mA in peaks while averaging 100 mA; the sleep module needs 200 mA available. A supply that sags on the peak is a supply that resets the module at the worst moment.

Practical layout order: bulk reservoir → LDO → module VCC, with the decoupling capacitor as close to the module pins as the layout allows. Keep the supply trace short and direct; do not route it past a relay coil or a motor driver.

## Two power domains, and they stay separate

If your product switches a load — a light, an HVAC contactor, a buzzer — the module and the load live in **separate power domains**, as shown in the [presence relay article]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}) and the [PLC dry-contact article]({{ site.baseurl }}{% post_url 2026-10-21-fall-gp1-plc-dry-contact %}):

```
   Module domain                 Load domain
   5 V, clean  ──► radar          5/12 V ──► relay ──► load
        │                              │
        └──────── single-point ────────┘
                  ground
```

- **One ground point where the two domains meet.** Two grounds joined in two places make a loop, and the loop current rides on your ground reference exactly where the sensor is most sensitive.
- **The GPIO pins are weak.** S1, S2 and GP1/GP2 are 3.3 V logic at milliamp level. They drive a transistor gate, never a relay coil or a lamp directly — with a flyback diode across any coil you switch.
- **Opto-isolation is available when the load side is nasty.** A high-voltage or long-cable load is worth an optocoupler; the GPIO then only has to drive an LED.

## The enclosure is part of the antenna

The 60 GHz modules radiate through their own PCB. The housing around them is part of the RF path, which is why the mounting notes in the guides read like RF instructions:

- **Non-metallic over the antenna face.** Plastic, not aluminium. A metal face in front of the antenna changes the beam and can detune it.
- **Flat, not curved.** A curved surface over the antenna is not supported; keep the face flat and parallel to the board.
- **3–5 mm from the antenna face to the inner shell surface.** Closer loads the antenna; further away wastes the space you needed for the lens or window.
- **Keep the radar component at least 1 mm above the surrounding PCB parts.** The antenna must not be shadowed by a taller component or a ground pour cut that was not planned.
- **Nothing metallic directly behind or beside the antenna** — mounting screws, brackets, and heatsinks all count.

If your product has a lens or radome over the antenna, treat its material and thickness as a design variable and confirm the effect with your own measurements rather than assuming.

## EMC basics that matter for this sensor

The radar is a sensitive receiver sitting next to whatever else your product contains. In order of impact:

1. **Distance from switching power.** Keep the switcher, its inductor, and its loop area away from the antenna end of the board. This is the single biggest source of broadband noise in a compact product.
2. **Motors and solenoids.** A buzzer, a lock, or a fan in the same enclosure couples straight into the measurement. If they must share a board, keep them at the far end and give them their own return path.
3. **Cable routing.** Keep the UART pair and any GPIO wires away from the supply wires and from anything switching current. Twist the UART pair if the run is long.
4. **Antenna area is keep-out.** No traces, no vias, no pours under or beside the antenna unless your design rules explicitly allow it.
5. **Ground discipline.** One solid ground reference for the analog/sensor domain; noisy return currents kept off it.

## Why your lab will ask about all of this

These are the same items that decide whether the product passes. The enclosure and the supply determine radiated emissions and band-edge behaviour, and the lab will want your technical file to reflect the configuration you actually ship. The full picture is in [Certification Support: Get CE and FCC Made Out to Your Own Name]({{ site.baseurl }}{% post_url 2026-10-23-certification-support-ce-fcc-own-brand %}) — including what we supply and what stays yours.

Two items from this article are worth telling your lab up front, because they are the ones most often discovered late: the **enclosure must be non-metallic and flat over the antenna face**, and the modules specify **power-supply ripple limits**.

## A pre-layout checklist

- [ ] Supply inside the module's voltage window, with ripple ≤ 100 mV at the module pins
- [ ] Peak current budgeted, with a reservoir for the fall module's 300 mA peaks
- [ ] LDO after any switcher, decoupling close to the pins
- [ ] Module domain and load domain separately supplied, single-point ground
- [ ] GPIO driving a transistor, never a coil directly; flyback diode on every coil
- [ ] Antenna face over non-metallic, flat material, 3–5 mm from the inner shell
- [ ] Radar component ≥ 1 mm above surrounding PCB parts; no metal behind the antenna
- [ ] Switcher, inductor and motors at the opposite end of the board
- [ ] Mounting height and tilt per the product guide for the variant you chose

## Where to go next

- All three modules' numbers: [60 GHz / 24 GHz module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- Wiring a load without writing a parser: [presence relay wiring]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}), [fall alarm relay]({{ site.baseurl }}{% post_url 2026-10-21-fall-alarm-relay-from-gp1 %}), [PLC dry contact]({{ site.baseurl }}{% post_url 2026-10-21-fall-gp1-plc-dry-contact %})
- Certification path: [CE / FCC support]({{ site.baseurl }}{% post_url 2026-10-23-certification-support-ce-fcc-own-brand %})
- Getting first frames: [Bring-Up Checklist]({{ site.baseurl }}{% post_url 2026-10-22-radar-module-bring-up-checklist %})

Modules and evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
