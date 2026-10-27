---
layout: post
title: "Distance and Speed from a Radar Echo: Two Equations, and What the Module Datasheets Publish"
date: 2026-10-28 08:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, fmcw, doppler, 24GHz, 60GHz, wavelength]
---

A radar echo carries two kinematic quantities: **range** — how far away the reflecting target is — and **radial velocity**, the component of the target's motion along the line between the radar and the target. Both are recovered from the same received signal, by two different routes. Range comes from timing, expressed through frequency. Velocity comes from the Doppler effect, expressed through frequency shift.

This article sets out the two equations, derives the quantities that follow from them for the two bands we ship — **24 GHz** and **61–62.5 GHz** — and states which parameters the module datasheets publish and which they do not.

The conceptual overview, without arithmetic, is in [How mmWave Radar Works: From FMCW to Breathing Detection]({{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %}).

## Range

### The underlying idea: round-trip time

The radar transmits, the wave travels to the target, part of the energy returns, and the receiver sees it. Electromagnetic waves propagate at `c ≈ 3×10⁸ m/s`, and the wave covers the distance twice, so the round-trip time `t` gives the range:

```
d = c · t / 2
```

The division by two accounts for the outbound and return legs.

### Why the time is not measured directly

For a target at 1 m the round trip is 2 m, which takes about **6.7 ns**. Measuring delay at nanosecond resolution directly is not practical in a module of this cost and size.

### How it is done instead: FMCW

The modules use **FMCW** (frequency-modulated continuous wave). The transmit frequency sweeps linearly with time, so the echo received at any instant is a delayed copy of an earlier transmit frequency. Mixing the received signal with the current transmit signal produces a **beat frequency** proportional to that delay, and therefore to range.

The published radar type confirms this: the fall module and the sleep module are both specified as **60 GHz FMCW**, and the fall module as **1T3R** (one transmit, three receive channels).

The round-trip-time equation remains the correct way to understand where range comes from. The beat frequency is the engineering route to the same number.

### What sets how finely two targets can be separated

Range resolution is set by the swept bandwidth `B`, not by the carrier frequency:

```
Δ d = c / 2B
```

Where a datasheet publishes the transmit frequency band, a figure follows. The fall module publishes a transmit frequency of **61–62.5 GHz**, a span of 1.5 GHz:

```
Δd = c / (2B) = 3x10^8 / (2 x 1.5x10^9) = 0.10 m
```

So about **10 cm**, on the assumption that the published operating band is also the swept span. Two notes on that figure:

- It is the separation at which two targets appear as two. The accuracy on a single target is finer, because a single return is localised by interpolation and by phase, not by which resolution cell it lands in.
- The 24 GHz datasheet we supply does not publish a swept bandwidth, so no equivalent figure is stated here.

This explains the distinction that is often blurred: a resolution cell of roughly 10 cm, and the detection of chest-wall displacements below 1 mm, are not the same measurement. The second comes from **phase** within a cell, covered below.

## Velocity

### The Doppler effect

When target and radar move relative to each other, the echo returns at a frequency offset from the transmitted one. Approaching shifts it upward, receding shifts it downward, and the sign gives the direction.

### The equation

With wavelength `λ` and Doppler shift `Δf`:

```
v = Δ f · λ / 2, Δ f = 2v / λ
```

The factor of two again comes from the two-way path.

### Wavelength from the published frequency

Wavelength follows from the carrier: `λ = c / f`.

| Module | Published band | Wavelength |
|---|---|---|
| R24AVD1 | 24 GHz | 12.5 mm |
| R60ABD1, R60AFD1-V, R60ABD2-H | 61–62.5 GHz | 4.80–4.92 mm |

For every 1 m/s of radial velocity, the shift is therefore:

- **24 GHz**: `2 / 0.0125` ≈ **160 Hz per m/s**
- **61–62.5 GHz**: ≈ **410 Hz per m/s**

### Shift magnitudes in practice

| Target | Radial velocity | Δf at 24 GHz | Δf at 61–62.5 GHz |
|---|---|---|---|
| Chest wall during breathing | ≈ 0.0075 m/s | ≈ 1.2 Hz | ≈ 3 Hz |
| Walking | 1.5 m/s | ≈ 240 Hz | ≈ 610 Hz |
| Running | 5 m/s | ≈ 800 Hz | ≈ 2.0 kHz |

The shifts fall in the hertz to kilohertz range. They are small fractions of the carrier, and they are recovered by coherent processing rather than by comparing nominal frequencies.

### Where sub-millimetre sensitivity comes from

Phase is the more sensitive quantity. A displacement `Δx` produces a round-trip phase change of:

```
Δ φ = 4\pi · Δ x / λ
```

For a 1 mm displacement this is about **58°** at 24 GHz and about **144°** at 61–62.5 GHz. The shorter wavelength gives roughly 2.5 times more phase per millimetre, which is the quantitative form of the statement made in [How mmWave Radar Works]({{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %}) — that 60 GHz is the band to use when breathing or heart rate is required.

## Three parameters that set precision and span

Each of the following is a sampling constraint, so each is a trade rather than a setting that can be widened independently.

**Range resolution** follows the swept bandwidth, `Δd = c / 2B`. More span gives finer separation.

**Velocity resolution** follows the observation time `T_obs` over which the phase history is collected:

```
Δv = λ / (2 · T_obs)
```

For a one-second observation this gives about 6.3 mm/s at 24 GHz and about 2.4 mm/s at 61–62.5 GHz. Longer observation gives finer velocity resolution, which is why slow periodic motion — breathing — is measured over intervals rather than from single frames.

**Unambiguous velocity** follows the chirp repetition interval `T_c`:

```
v_max = λ / 4 · T_c
```

This is the one quantity where the shorter wavelength works in the opposite direction. At the same chirp interval, the 61–62.5 GHz band reaches a smaller unambiguous velocity than 24 GHz. A radar tuned to resolve sub-millimetre chest motion and a radar built to measure vehicle speed are therefore sampling the same physics with incompatible timing.

**Not published:** the modules' internal chirp timing and coherent processing interval do not appear in the datasheets, so no `T_c` or `T_obs` value is given here, and the worked figures above are labelled as illustrations of the equations.

## Published output cadences

Separate from the radio parameters above are the intervals at which results are reported. These are published, and they are output cadences, not integration times:

| Report | R24AVD1 (24 GHz) | 60 GHz finished units |
|---|---|---|
| Occupied → unoccupied | 40 s | about 40 s |
| Unoccupied → occupied | < 0.5 s | — |
| Static / active change | < 0.5 s | — |
| Approach / leave / no lateral movement | 2 s | — |
| Body-movement amplitude 0–100 | 5 s | 1 s |
| Breathing rate, heart rate | — | about 3 s |

The asymmetry — under half a second to declare a room occupied, about 40 seconds to declare it empty — is a deliberate stability choice, described in [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %}).

Distance and position are reported by the finished units in centimetres and as x/y/z, both about every 2 s, and both documented as **non-critical functions with limited accuracy**. They are useful while commissioning, as set out in [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %}), and are not a basis for a product feature.

The body-movement amplitude is a dimensionless index from 0 to 100 with defined bands, not a velocity. It is not converted to metres per second.

## The two bands side by side

| Quantity | 24 GHz | 61–62.5 GHz |
|---|---|---|
| Wavelength | 12.5 mm | 4.80–4.92 mm |
| Doppler shift per 1 m/s | ≈ 160 Hz | ≈ 410 Hz |
| Phase change per 1 mm displacement | ≈ 58° | ≈ 144° |
| Phase change per 1 mm, relative | 1× | ≈ 2.5× |
| Velocity resolution at 1 s observation | ≈ 6.3 mm/s | ≈ 2.4 mm/s |
| Unambiguous velocity at equal chirp interval | larger | smaller |
| Swept bandwidth published | no | 61–62.5 GHz (1.5 GHz span) |
| Derived range resolution | not stated | ≈ 10 cm |

## What this accounts for

- **Respiration and heart rate use 60 GHz.** Displacement sensitivity scales inversely with wavelength, so the shorter band returns about 2.5 times more phase per millimetre.
- **Presence over a wide area uses 24 GHz.** The longer wavelength and the wider beam cover a larger volume; the published beam for R24AVD1 is 90° horizontal by 60° vertical.
- **A posture radar is not a speed instrument.** The unambiguous velocity limit moves against the higher band, and the same processing budget is spent on deciding what the target is doing. The practical figure for a home posture radar is around 3 m/s, with the reasoning in [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %}).

## Summary

Range comes from round-trip time, recovered in practice from an FMCW beat frequency, and its resolution follows the swept bandwidth. Velocity comes from the Doppler shift, and its resolution and span follow observation time and chirp interval respectively.

The two bands differ by a factor of about 2.5 in wavelength, and that one ratio propagates through every sensitivity figure — phase per millimetre, Doppler shift per metre per second, velocity resolution — while moving the unambiguous velocity limit in the opposite direction.

Where a datasheet publishes a value, that value is used here. Where it does not, the equation is given and the absence is stated.

## Where to go next

- The conceptual version without arithmetic: [How mmWave Radar Works]({{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %})
- Practical limits on angle, distance and speed: [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %})
- Module specifications in full: [60 GHz & 24 GHz Radar Modules for OEM Integration]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- Using distance and position while commissioning: [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
