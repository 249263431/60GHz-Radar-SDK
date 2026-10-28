---
layout: post
title: "Where the Beam Actually Lands: Coverage from Beam Angle and Mounting Height"
date: 2026-10-29 08:00:00 +0800
categories: [guide]
tags: [radar, installation, geometry, coverage, commissioning]
---

A datasheet gives you a beam angle and a mounting height. What you actually need to know is how much floor the radar covers, and how much of that footprint is usable. Those are two different questions, and the published figures answer them separately.

This article works the geometry out for the three modules we ship, using published beam angles and published mounting heights. It also deals with the gap that appears when you put the two published numbers next to each other — the geometric footprint and the quoted detection range often do not agree, and when they disagree, one of them is the harder bound.

## Two quantities, not one

| Quantity | Set by | Nature |
|---|---|---|
| **Geometric footprint** | Beam angle and mounting height | Geometry. Calculable, and a hard bound. |
| **Detection range** | Signal-to-noise ratio, target size, and the conditions of the measurement | Measured under stated conditions. |

The beam decides where energy can reach. Detection range decides whether what comes back is enough to make a decision. A target inside the beam but beyond the detection range is illuminated and still not reported.

## The geometry

### Axis pointing straight down

For a ceiling-mounted unit at height `h` with a full beam angle `θ` on the axis of interest, the footprint half-width on the floor is:

```
r = h x tan(θ / 2)
```

The full width is `2r`. `θ` is the **full** angle as published, not the half angle.

### Axis tilted

For a wall-mounted unit, the geometry is easier in terms of the depression angle `φ` below horizontal. With height `h` above the reference plane and beam angle `θ`:

```
near edge   = h / tan(φ + θ/2)
beam centre = h / tan(φ)
far edge    = h / tan(φ - θ/2)
```

The useful angles:

| Angle | tan |
|---|---|
| 10° | 0.176 |
| 25° | 0.466 |
| 30° | 0.577 |
| 45° | 1.000 |
| 50° | 1.192 |
| 65° | 2.145 |

## Presence module: R24AVD1, ceiling

Published: beam **90° horizontal × 60° vertical**, ceiling mounting to **2.75 m**, horizontal tilt ≤ 3°.

At 2.75 m:

```
width  = 2 x 2.75 x tan(45°) = 5.5 m
depth  = 2 x 2.75 x tan(30°) = 3.2 m
```

The footprint is therefore roughly **5.5 m × 3.2 m**, and it is not square — the vertical beam is narrower, so the depth of coverage is smaller than the width. Orienting the board with the long side toward the door, as the integration notes recommend, puts the wide axis where it is wanted.

The published detection figure for this module is much larger — a moving person at up to about 12 m across. That figure is measured for a moving target under stated conditions. The 5.5 × 3.2 m footprint is where the beam puts energy at 2.75 m. Both are real; they are not the same statement, and the geometry is the one you can rely on without qualification.

## Fall module: R60AFD1-V, ceiling

Published: beam **100° × 100°**, mounting **2.2–2.8 m**, fall / static-dwell / motion radius ≈ **1.5 m**.

At the ends and middle of the published mounting range:

| Mounting height | Footprint (square) |
|---|---|
| 2.2 m | 5.2 m × 5.2 m |
| 2.5 m | 6.0 m × 6.0 m |
| 2.8 m | 6.7 m × 6.7 m |

Against a detection radius of 1.5 m, the beam covers a floor area several times larger than the area in which a fall can be decided. Raising the unit from 2.2 m to 2.8 m widens the footprint from about 5.2 m to about 6.7 m without extending the radius in which the decision is possible — which is why the mounting height is a configured parameter (DP6, default 2.3–2.4 m) rather than a free choice.

The practical reading: size the room against the 1.5 m radius, not against the beam. A bathroom wider than about 3 m across will have corners the radar illuminates and cannot decide on.

## Sleep module: R60ABD1, tilted above the bed

Published: beam **40° × 40°**, mounted above the bed at **0.8–1.0 m**, tilt **30°–45°**, with **45°** given as the recommended value. Respiration is specified to **1.5 m**, presence and sleep to **2.5 m**.

Take `h = 1.0 m` and `θ = 40°`, so `θ/2 = 20°`.

At the recommended 45°:

```
near edge   = 1.0 / tan(65°) = 0.47 m
beam centre = 1.0 / tan(45°) = 1.00 m
far edge    = 1.0 / tan(25°) = 2.14 m
```

At 30°:

```
near edge   = 1.0 / tan(50°) = 0.84 m
beam centre = 1.0 / tan(30°) = 1.73 m
far edge    = 1.0 / tan(10°) = 5.67 m
```

The geometry explains the published recommendation. At 45° the far edge sits at 2.14 m, inside the 2.5 m sleep range, so essentially all of the illuminated area is usable. At 30° the far edge runs out to 5.67 m, well past the 2.5 m figure — most of the beam is falling on floor that the radar cannot report on, and the beam centre has moved to 1.73 m, away from the chest.

Working the same numbers at the lower mounting height, `h = 0.8 m` with a 45° tilt, gives a near edge of about 0.37 m and a far edge of about 1.72 m — a tighter footprint concentrated on the upper body, which is what breathing detection wants.

For respiration specifically, the 1.5 m limit means the reliably measured band on the bed runs from about 0.47 m to 1.5 m from the wall at `h = 1.0 m` and 45°. That is the part of the bed to aim at.

## Why the two numbers diverge

Beam edge energy is lower than on-axis energy, and off-axis targets return a weaker echo at a worse incidence angle. Decisions that need more than "something is there" — is this person lying down, is this a fall — need more signal than simple presence does, so their usable radius is smaller even inside the same beam.

Published range figures are measured against a defined target under defined conditions. Comparing them directly against a geometric footprint is a category error in both directions: the footprint can be larger than the usable area, and a quoted range can be larger than the footprint at a low mounting height. When the two disagree, treat the geometry as the bound you can compute, and the quoted range as a figure you verify on site.

## What geometry does not tell you

The calculation assumes a clear line of sight. It says nothing about:

- **Occlusion** — a cabinet, a shower screen, a plant between the radar and the target
- **Reflections** — metal and glass surfaces that route energy elsewhere and produce returns from places no one is standing; see [Ghost Targets and Glass Walls]({{ site.baseurl }}{% post_url 2026-10-19-ghost-targets-multipath-interference %})
- **Target size** — a person standing returns more than a person lying down at the same distance
- **Interference** — moving non-living objects inside the beam, such as fans and curtain rails

## Verifying it on site

The finished units report distance and position, both about every 2 s and both documented as non-critical with limited accuracy. Walk the edge of the computed footprint and watch where the target is picked up and where it is lost; that gives you the real boundary, including occlusions the calculation cannot see. The method is set out in [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %}).

Record the result. It is the number your next installation of the same model starts from.

## Checklist

- [ ] Beam angle read as the **full** angle, and halved before use
- [ ] Footprint computed at the actual mounting height, not at the nominal one
- [ ] For ceiling units: width and depth computed separately when the beam is not square
- [ ] For tilted units: near edge, centre and far edge all computed, and the far edge checked against the published range
- [ ] Room sized against the **detection radius**, not the beam footprint
- [ ] Occlusions and reflective surfaces checked separately — the calculation assumes clear line of sight
- [ ] Boundary confirmed on site with the distance output, and recorded

## Where to go next

- Where to put a sleep radar, and what to avoid: [Sleep Radar Installation — Finding the Right Position]({{ site.baseurl }}{% post_url 2026-10-16-sleep-radar-installation-position %})
- Beam angles, mounting heights and ranges in full: [60 GHz & 24 GHz Radar Modules for OEM Integration]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- Why published ranges should be treated with care: [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %})
- Commissioning the result: [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
