---
layout: post
title: "Where mmWave Radar Is Used: From Parking Spaces to Oil Tanks"
date: 2026-10-18 06:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, applications, level measurement, parking, sonar]
---
<!-- filename: 2026-10-18-mmwave-radar-applications.md -->

When most people first hear about 60 GHz mmWave radar, they picture one thing: a small module mounted on a wall, telling a smart home system whether someone is in the room.

That is a real application, but it is a narrow one. The same sensing principle — transmit a frequency-modulated chirp, measure the round-trip time for range and the Doppler shift for velocity — is also used to measure the fuel level in a storage tank, hold a drone at a fixed altitude, time a car at an intersection, and map crop types from orbit.

The physics does not change between these applications. What changes is the frequency band, the antenna design, the beamwidth, and above all what the signal processing looks for in the returned signal.

This article walks through the main application families, and ends with an honest note about the one slice we build for.

## Automotive and Traffic

**Autonomous driving and driver assistance.** Radar measures the range and relative velocity of vehicles, cyclists, and pedestrians directly, rather than inferring them from pixels. This is the main reason radar stays dependable in rain, fog, glare, and darkness — conditions where a camera's input degrades sharply. A vision system and a radar are usually fused, because they fail in different conditions.

**Traffic monitoring.** Roadside radar measures the speed of passing vehicles and detects which lanes are occupied. Because it measures velocity through the Doppler shift rather than by comparing frames, it does not need to track an object across images to know how fast it is moving. The output feeds signal controllers and enforcement systems.

**Parking.** Per-space sensors detect whether a bay is occupied. Radar suits this because it can be mounted above or beside the space, is unaffected by lighting or weather, and does not require a clear line of sight in the way a camera does.

## Industrial Level Measurement

Large storage tanks — fuel, chemicals, water — are commonly measured with radar mounted at the top of the tank. The radar measures the distance to the liquid surface, and the level follows from the known tank geometry.

The reason radar is preferred here is that it is **non-contact**. Nothing touches the product, which matters when the contents are corrosive, hot, pressurised, or need to stay sealed. It is also unaffected by vapour, dust, temperature gradients, or the colour and transparency of the liquid. Precision at the millimetre level is achievable, and at the scale of a large tank, a millimetre of level error translates into a meaningful volume error, so that precision is worth paying for.

## Security Screening

Human tissue and metal return measurably different signals. Their dielectric properties and reflectivity differ, so the phase of the reflected signal changes depending on what the wave bounced off. Phase-sensitive radar exploits this difference to detect concealed items.

This is worth calling out as a separate family because it is a fundamentally different signal-processing problem from presence detection. Presence detection asks "is there a target that moves." Screening asks "what is this target made of." Those need different features extracted from the same returned signal.

## Drones and Light Vehicles

**Drones.** Radar is used for altitude hold, obstacle avoidance, and — in more capable systems — imaging radar that forms a coarse representation of the surrounding environment. The advantage over optical or ultrasonic sensing is operation in dust, fog, and darkness, plus direct velocity measurement.

**E-bikes and motorcycles.** Rear-facing radar provides approach and blind-spot warning for two-wheeled vehicles, where the rider cannot simply glance over a shoulder the way a car driver can.

## Weather and Agriculture

**Weather.** Radar measures precipitation — including rain density — by analysing the returned energy from hydrometeors across a volume.

**Agriculture.** The reflection signature of a dense insect swarm differs from that of bare soil or an individual plant. Radar can estimate pest density, including locust swarms, from the characteristics of the return. It is a coarse estimate, not a species-level identification, but it covers areas that are impractical to survey by hand.

## Satellite and Remote Sensing

Orbital radar observes surface structure and can distinguish classes of vegetation by their reflection characteristics. This is remote sensing rather than proximity sensing, but it is the same principle: transmit, receive, and interpret what came back.

## Underwater: The Most Common Misconception

This one comes up regularly, so it is worth addressing directly. The question is usually some version of: *can radar detect fish, or animals, underwater?*

**No. Underwater detection uses sonar, which is acoustic, not electromagnetic.**

Sound propagates through water; radio energy does not travel far in it. Water absorbs electromagnetic waves very rapidly, and in practical terms a radar submerged in water propagates only a very short distance before the signal is gone. If the target is underwater, the sensor is a sound-based one, and the signal processing works on acoustic echoes and their characteristics.

This is not a limitation of any particular radar — it is the medium. The moment the target is below the water surface, radar is the wrong tool.

## Indoor Presence, Counting, Posture, and Breathing

And then there is the slice we build for: rooms.

In an indoor setting the useful questions are narrow and specific:

- Is anyone in this room?
- How many people are there?
- What is the person's posture — standing, sitting, lying?
- Are they breathing, and at what rate?

These are answered with the same hardware used in the applications above, but tuned completely differently: short range, wide field of view, fine velocity resolution, and signal processing built around human-scale motion rather than vehicles or liquid surfaces.

Two properties make radar the right choice indoors. It works in complete darkness, because it generates its own illumination. And it produces no image at all — no lens, no video, nothing identifiable — which is why it is acceptable in bedrooms and bathrooms where a camera would not be.

## Why We Only Build One Slice

Every application above is its own engineering discipline. Automotive radar has to survive vibration, temperature extremes, and functional-safety certification. Tank-level radar has to be certified for hazardous atmospheres. Screening radar is a materials-classification problem. Drone radar is constrained by size and power in a way that wall-mounted sensors are not.

Doing all of them well is not realistic. Doing one of them properly is. We build indoor sensing modules, and we refer the other enquiries elsewhere.

## Summary

| Application | What the radar measures | Why radar fits |
|---|---|---|
| Automotive / ADAS | Range and relative velocity of road users | Works in rain, fog, darkness |
| Traffic enforcement | Vehicle speed, lane occupancy | Velocity comes from Doppler, not frame tracking |
| Parking | Bay occupancy | Lighting- and weather-independent |
| Tank level | Distance to liquid surface | Non-contact, millimetre precision |
| Security screening | Material differences via phase | Metal and tissue reflect differently |
| Drones | Altitude, obstacles | Operates in dust and darkness |
| Two-wheelers | Rear approach | Direct velocity measurement |
| Weather | Rain density | Volume measurement |
| Agriculture | Pest / swarm density | Covers large areas |
| Satellite | Surface and vegetation type | Works through cloud cover |
| Underwater | **Not applicable** | Use sonar; radio waves do not propagate in water |
| Indoor | Presence, count, posture, breathing | Darkness-capable, no camera, no image |

The technology is one. The applications are many, and they share almost nothing beyond the basic transmit-and-receive principle. Knowing which one you need — and which one you are buying — matters more than knowing the frequency.
