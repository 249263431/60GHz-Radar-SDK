---
layout: post
title: "Antenna Channels, Virtual Aperture, and What More of Them Actually Buys You"
date: 2026-10-30 08:00:00 +0800
categories: [fundamentals]
tags: [antenna, mimo, radar, mmwave, point cloud]
---

Channel counts are the specification most often quoted when radar modules are compared: one transmit and three receive, four and four, six and six. The comparison is meaningful, but it is often read as a linear quality scale, and it is not one. What a channel buys depends entirely on what the radar is being asked to work out.

This article sets out what channel count determines, what it does not, and how that maps onto the three jobs these modules are built for — presence, breathing and sleep, and fall detection.

## What a channel count means

With `T` transmit antennas and `R` receive antennas, time-division MIMO produces a **virtual array** of:

```
N = T x R
```

virtual channels. The virtual array, not the physical one, sets the angular capability. Two consequences follow directly, and a third follows from how the elements are placed.

### Angular resolution

With elements at half-wavelength spacing, the angular resolution at broadside is approximately:

```
theta_res ≈ 2 / N  radians  =  114.6 / N  degrees
```

| Configuration | Virtual channels | Approximate angular resolution |
|---|---|---|
| 1T1R | 1 | none — no angle at all |
| 1T3R | 3 | ≈ 38° |
| 2T4R | 8 | ≈ 14° |
| 4T4R | 16 | ≈ 7.2° |
| 6T6R | 36 | ≈ 3.2° |

These are theoretical figures, at broadside and with good signal-to-noise ratio. Real performance is worse, and it degrades further away from the antenna boresight.

### Coherent combining gain

Processing across `N` channels gives a combining gain of about:

```
G = 10 x log10(N)  dB
```

| Configuration | Combining gain |
|---|---|
| 1T3R | ≈ 4.8 dB |
| 2T4R | ≈ 9 dB |
| 4T4R | ≈ 12 dB |
| 6T6R | ≈ 15.6 dB |

Because detection range scales as the fourth root of signal-to-noise ratio, a gain difference of `G` dB changes range by a factor of `10^(G/40)`. Going from three virtual channels to sixteen — about 7 dB — extends range by roughly 1.5 times. That is a real improvement, and it is much smaller than the ratio of channel counts suggests.

### Elevation

Angular resolution in azimuth is not enough when the question is about height. Estimating elevation requires receive elements distributed in the **vertical** direction. A single row of elements gives azimuth only, no matter how many there are. An L-shaped or triangular arrangement gives both, at lower resolution than either.

This is the one place where the physical layout matters more than the count. Three elements in a triangle can say something about height; eight elements in a straight line cannot.

## What a channel count does not decide

Two of the most important capabilities in these applications are set by other parameters entirely.

**Range resolution** depends on swept bandwidth, not on channels:

```
Δd = c / (2B)
```

**Displacement sensitivity** depends on wavelength and phase, not on channels:

```
Δφ = 4π x Δx / λ
```

At 61–62.5 GHz the wavelength is about 4.9 mm, which gives roughly 144° of phase change per millimetre of displacement. That is what lets breathing be measured, and adding channels does not change it.

A worked example makes the boundary clear. Separating two fingers about 2 cm apart at a distance of 0.5 m needs an angular resolution near 2.3°, which means roughly 50 virtual channels — and it also needs a range resolution better than 2 cm, which needs about 7.5 GHz of bandwidth. Neither is available in a 60 GHz module. Fine spatial detail of that kind is imaging-radar territory, not the territory of a four-by-four or a six-by-six.

## What the three jobs actually require

| Job | Question being answered | Capability it actually rests on | Does it need angular resolution? |
|---|---|---|---|
| Presence (24 GHz) | Is anyone in the room, including someone sitting still? | Sensitivity, beam coverage, detection of small motion | No |
| Breathing and sleep (60 GHz) | Breathing rate, heart rate, sleep stage | Phase sensitivity at a short wavelength, signal-to-noise ratio | No |
| Fall detection (60 GHz) | Is this person upright, seated, or lying down? | **Change in height** — the elevation dimension | Yes, and it can be coarse |

Two of the three do not need angle at all. They need sensitivity and phase, which come from wavelength, bandwidth and combining gain.

The third is the exception. Distinguishing lying from standing is a height question, so fall detection is the only one of the three that requires the elevation dimension — and it requires it only well enough to place a body in one of a few postures, not to resolve it in detail.

This is also visible in the published figures for the fall module: a fall and static-dwell radius of about 1.5 m against a beam of 100° × 100°. The decision radius is far smaller than the illuminated area. The decision is made from signal quality and height change within that radius, not from angular precision across the whole beam.

## Why more channels are not automatically better for home monitoring

The classification being asked for is coarse: unoccupied, lying, seated, standing, moving. That is a small number of classes, and it does not require a dense point cloud to separate them.

Adding channels past the point where those classes are separable buys detail that the application cannot use, and it costs in four places:

- **Cost and die area.** Every channel carries its own RF front-end circuitry.
- **Power.** More active channels per frame.
- **Frame duration.** In time-division MIMO the transmit antennas take turns, so the frame grows roughly linearly with the number of transmit channels. Using the relation from [Distance and Speed from a Radar Echo]({{ site.baseurl }}{% post_url 2026-10-28-distance-speed-two-equations %}), `v_max = λ / (4 · T_c)`, a longer per-antenna repetition interval lowers the unambiguous velocity. A configuration with four transmit channels reaches one quarter of the unambiguous velocity that one transmit channel would, at the same chirp duration. Fast motion then aliases, and a moving target that crosses resolution cells within one coherent interval produces a smeared result.
- **Array layout constraints.** Packing many elements into a small board pushes spacing towards or beyond half a wavelength, which produces grating lobes — false angular responses — and increases mutual coupling between elements. Both degrade the point cloud rather than improving it.

There is also a processing cost: more channels means more data per frame and more work per frame, on hardware that in a domestic product is expected to be inexpensive.

## When more channels are genuinely needed

- **Separating several people** in the same space, and tracking each one
- **Finer posture work** than lying, seated and standing
- **Imaging applications**, where the output is a shape rather than a class
- **Longer range**, where the combining gain is what buys the link budget

None of these is required by unattended presence, breathing monitoring, or fall detection in a single-occupant room.

## Choosing for the job

| If the product must… | Then the design pressure is on… |
|---|---|
| Say whether a room is occupied, including a still person | Sensitivity and beam coverage, not channel count |
| Report breathing rate and sleep stage | Wavelength, phase stability and signal quality |
| Decide whether someone has fallen | An elevation dimension, of modest resolution |
| Separate and track several people | Angular resolution — genuinely more channels |
| Produce a shape or a fine posture estimate | Many channels, and bandwidth to match |

## Checklist

- [ ] The question the radar answers written down before the channel count is chosen
- [ ] Whether an elevation dimension is needed at all, checked first
- [ ] Angular resolution computed as `114.6 / N` degrees, and compared against the actual separation required
- [ ] Range resolution computed as `c / (2B)` separately — channels do not affect it
- [ ] Combining gain converted into range, not read as a raw dB figure
- [ ] Frame duration checked against the fastest motion that must be captured
- [ ] Element spacing checked against half a wavelength
- [ ] Cost, power and processing budget checked at the higher count

## Summary

Channel count sets the virtual aperture, and therefore angular resolution, combining gain, and — if the elements are placed in two dimensions — the ability to estimate height. It does not set range resolution, which is bandwidth, and it does not set displacement sensitivity, which is wavelength and phase.

For presence, breathing and fall detection in a home, the decisions being made are a small number of coarse classes. Two of the three do not use angle at all, and the third needs it only coarsely. Beyond that point, additional channels cost frame duration, area, power and processing, and can degrade the result through grating lobes and motion within the coherent interval.

## Where to go next

- The equations behind range and velocity: [Distance and Speed from a Radar Echo]({{ site.baseurl }}{% post_url 2026-10-28-distance-speed-two-equations %})
- Turning beam angle and mounting height into floor coverage: [Where the Beam Actually Lands]({{ site.baseurl }}{% post_url 2026-10-29-beam-coverage-installation-geometry %})
- Beam angles, ranges and mounting heights in full: [60 GHz & 24 GHz Radar Modules for OEM Integration]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- Practical limits on quoted specifications: [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
