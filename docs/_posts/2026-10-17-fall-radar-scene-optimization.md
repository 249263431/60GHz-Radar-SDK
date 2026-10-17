---
layout: post
title: "Why the Same Fall Radar Behaves Differently in a Bathroom and an Open Office"
date: 2026-10-17 09:00:00 +0800
categories: [product]
tags: [fall radar, scene optimization, bathroom, false alarm, missed detection, installation]
---

<!-- filename: 2026-10-17-fall-radar-scene-optimization.md -->

A customer mounts a fall detection radar in an office corner, walks around, lies down on the floor, and gets a reliable alarm. Another customer mounts the same radar in a bathroom and gets a different result. Neither radar is faulty. The radar is tuned for one of those two rooms.

This article explains what "tuned for the bathroom" actually means, which three design decisions depend on it, and what changes when the radar is moved to an open space. It is written for anyone evaluating this radar, and for anyone who has tested it somewhere other than a bathroom and wondered why the behaviour changed.

## Why We Optimize for the Bathroom

A bathroom is a small, enclosed room with a single door, hard reflective surfaces, and a handful of predictable moving objects. That combination is unusual, and it cuts both ways.

The same properties that make a bathroom difficult — tiled walls, a waterproof floor, a toilet, an extractor fan, a shower curtain — also make it possible to tune a radar very precisely for it. A room of roughly **2 m × 3 m**, about **5 to 6 m²**, with one door and hard walls, gives the radar a bounded area with a known entry and exit path.

So the tuning is not a limitation added to the product. It is a set of deliberate design decisions that only work because the room has that shape.

## Interference Sources in a Bathroom

Four things in a typical bathroom produce radar returns that resemble a person.

| Source | Why It Causes a False Reading |
|---|---|
| Washing machine | A low, vibrating target can resemble a collapsed posture |
| Toilet flushing | The signal passes through tile; the changing water surface produces a strong return |
| Shower curtain or towel moving in air flow | A moving soft target looks similar to a moving person |
| Extractor fan vibration | Continuous micro-movement disturbs the stillness judgement |

Each of these has a different signature. A washing machine is low and constant. A flushing toilet produces a brief, strong return from a fixed position. A curtain moves without a consistent phase structure. A fan vibrates at a fixed frequency.

A general-purpose sensor sees all four as ambiguous. A sensor tuned for the room can be told what to expect.

## How Entry and Exit Direction Suppresses False Alarms

This is the most important of the three design decisions, and it is the one that stops working first when the radar leaves a bathroom.

The radar tracks which direction a person enters and leaves through. In a bathroom there is exactly one door, so the entry and exit paths are known and short. That single fact is enough to resolve most of the interference table above.

Here is the logic:

1. A person enters through the door. The radar registers an entry event.
2. While the person is inside, movement, presence, and posture are evaluated normally.
3. When the person leaves through the same door, the radar registers an exit event.
4. **From that moment, reports from the room are suppressed for a defined window.**

Step 4 is the key. Most false alarms in a bathroom happen at the moment of leaving: the toilet flushes after the person has gone, the curtain settles, the washing machine starts its spin cycle. Without the exit event, the radar has no way to know those returns are not a person on the floor. With it, they are simply discarded.

A washing machine that was a problem all day stops being a problem, because the radar knows nobody is in the room.

## Why an Open Room Breaks This Logic

The suppression window depends on there being one door. Remove the door, or add a second one, and two different failures appear.

**Failure one: the radar keeps reporting a person who has left.** In a corridor or an open office, a person enters from one side and leaves from another. The radar sees an entry without a matching exit, cannot conclude that the room is empty, and continues to treat the space as occupied. The state that should have cleared does not clear.

**Failure two: the radar clears a room that still has someone in it.** In a busy space, several people enter and leave through different paths. The radar concludes that the occupancy has ended and clears the state to unoccupied, while a person is still inside. The result is the opposite error: someone is present, and the radar reports nobody.

Neither failure is a defect. Both are the direct consequence of applying a single-door assumption to a room with no doors.

This is why the radar is specified for bathrooms and similar small enclosed rooms, and not for corridors, open-plan offices, or any space where people enter and leave through different paths.

## Signal Strength: Why Reflections Help

The second design decision is counter-intuitive: the radar performs better in a bathroom than in an empty room, and the reason is the tiling.

A bathroom is full of hard, flat, reflective surfaces. Tile walls, a glazed floor, and waterproof membranes create strong multi-path reflections, and the radar receives the same target through several paths at once. In a bathroom, that means the return from a person lying still is strong and stable.

The same reflections also create what can be called dry interference: returns that arrive from fixed structures rather than from a person. The detection chain filters those out, so the useful signal from the person is kept and the structural returns are removed. The room is reflective enough to give a strong signal, and the filtering is tuned to know which reflections to ignore.

An open room does not have that property. Fewer surfaces, longer distances, and absorptive materials such as carpet, fabric, and plasterboard mean weaker returns and fewer of them. A person lying completely still in an open area produces a weaker signal than the same person in a tiled bathroom, because there is simply less energy coming back.

So an empty office is not a cleaner test environment. It is a different one, with less signal to work with.

## Geometry: Why the Range Never Misses in a Bathroom

The third design decision is about shape rather than signal.

A bathroom is bounded. Walls on all sides, a typical floor area of 2 m × 3 m, and a ceiling height of 2.2 to 2.8 m. A fallen person is over a metre long, so within that space a fall lands well inside the detection area. The walls do the bounding for free: there is nowhere in the room for a body to fall outside the covered zone.

An open area has no such boundary. A person can fall with their feet inside the detection area and their torso outside it, or fall just beyond the edge of the coverage. The radar reports what it can see, and what it can see is a partial target at the edge of range.

The detection area of this radar is approximately **3 m × 2 m** for a low posture and **3 m × 2.5 m** for fall and dwell detection. In a bathroom of 2 m × 3 m, that covers the whole floor. In an open room of the same size or larger, it covers part of it.

This is the difference between a radar that is specified for a room and a radar that is specified for an area. This product is specified for a room.

## Periodic Re-Detection and Reporting Rate

Every millimeter wave radar faces the same trade-off. Reporting an event immediately risks reporting it wrongly; waiting to confirm risks missing it entirely.

The standard answer is periodic re-detection: the radar re-examines the target at fixed intervals rather than deciding once. A posture that persists across several consecutive checks is reported; one that does not is discarded. This is what raises the reporting rate, and it is also what keeps the false alarm rate down, because a transient return does not survive the re-checks.

Two parameters control the balance:

- **Fall duration** — how long the posture must persist before the alarm is published. Shortening it catches more events and produces more false ones; lengthening it does the opposite.
- **Dwell duration** — the same idea applied to a person who has stopped moving without falling.

In a bathroom, the entry and exit information removes most of the false candidates before the re-detection stage even runs, so the periodic checks can be tuned aggressively for sensitivity. In an open room, the false candidates are never removed, so the same tuning produces alarms that the re-checks then have to reject.

The result is a radar that feels responsive in the intended scene and hesitant outside it.

## How to Test It Properly

If you are evaluating this radar, test it in the environment it is designed for. A bathroom-like setup takes twenty minutes and removes every variable discussed above.

1. Build or use a space of roughly **2 m × 3 m**, with walls on all sides.
2. Fit a single door, and use it for every entry and exit during the test.
3. Mount the radar on the ceiling at **2.2 to 2.8 m**, sensor face pointing straight down, above the area where a person would fall.
4. Enter through the door, move around, and leave through the same door. Confirm the presence state follows you.
5. Test a fall with a real lying posture, held still. Confirm the alarm arrives within the configured fall duration.
6. Test a dwell: lie still without falling, and confirm the dwell alarm arrives after the configured dwell duration.
7. Introduce one interference source at a time — a running extractor fan, a hanging cloth, a filled container — and confirm it does not produce an alarm while nobody is in the room.
8. Leave the room and confirm the state clears, then confirm that subsequent interference does not raise an alarm.

If the radar passes those eight steps in a room of that shape, it will behave the same way in a real bathroom. If it is tested in a corridor instead, steps 4 and 8 will fail for the reasons described above, and the result will not predict the real-world behaviour.

## What This Radar Is Not

- **It is not a general-purpose presence sensor.** It is tuned for a small enclosed room with one door. For corridors and open spaces, use a presence radar instead.
- **It does not achieve a zero missed-event rate.** The design target is the highest practical reporting rate in the intended scene, and the tuning is directed at reducing missed events there. No sensor of this type eliminates them.
- **It will produce false alarms outside the intended scene.** Entry and exit suppression is what removes them, and that mechanism requires a single known door.
- **It is not a medical device and not an emergency service.** It publishes alarms to your MQTT broker. What happens next — a notification, a call, a dashboard entry — is built by your own system.
- **It does not replace human checks.** For anyone at real risk, it is an addition to a care routine, never a substitute for one.

## Summary

This radar is tuned for a bathroom, and the tuning rests on three things: a single door that makes entry and exit known, reflective surfaces that give a strong return which the detection chain then filters, and walls that bound the detection area so a fall cannot land outside it.

Move it to an open office and all three weaken at once. The entry and exit logic stops resolving, the signal gets weaker instead of stronger, and the coverage no longer covers the whole floor. The radar is not broken; it is being asked to work in a room it was not designed for.

Test it in a 2 m × 3 m space with one door, mounted at 2.2 to 2.8 m facing straight down, and use the door for every entry and exit. If it passes there, it will pass in a real bathroom.

Further detail on the alarm fields, timing parameters, and MQTT properties is in the companion article <a href="{{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}">The Fall Detection Radar: How Alarms Work, How to Tune Them, and What to Expect</a>.
