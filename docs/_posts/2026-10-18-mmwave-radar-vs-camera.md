---
layout: post
title: "mmWave Radar vs. a Camera: What Each One Actually Measures"
date: 2026-10-18 06:00:00 +0800
categories: [fundamentals]
tags: [mmwave radar, camera, comparison, micro-motion, privacy]
---
<!-- filename: 2026-10-18-mmwave-radar-vs-camera.md -->

The first question we get asked about indoor sensing is almost always the same: *why not just use a camera?*

It is a fair question. A camera is cheaper, familiar, and produces output a human can look at and understand immediately. But a camera and a mmWave radar do not measure the same thing, and each is clearly better at a different job.

The short version: **a camera tells you roughly what a person looks like and what posture they are in. A radar measures motion far too small for a camera to register at all.** Understanding that distinction is what determines which one you need.

## What a Camera Measures

A camera records reflected visible light. From those pixels you get shape, colour, texture, and — with modern processing — a rough skeletal pose.

The strength of this is that the output is rich and directly interpretable. If you want to know whether a person is roughly standing, sitting, or lying down, a camera gives you that more clearly and more cheaply than a radar does. You can look at the frame and confirm it yourself, which makes cameras easy to install and easy to debug.

The weakness is that everything downstream depends on light and on a clear line of sight. A camera needs illumination, or infrared in the dark. It needs a lens that stays clean. It stops working behind steam, smoke, or a blanket. And it produces identifiable video, which is exactly why installing one in a bedroom or a bathroom is a non-starter for most people.

## What mmWave Radar Measures

Radar transmits radio energy and listens for what comes back. From the returned signal it extracts three things:

- **Range** — distance to the target, from the round-trip time of the chirp.
- **Velocity** — radial motion, from the Doppler shift.
- **Micro-motion** — very small displacements, from the phase of the returned signal.

The third one is the one that matters for indoor sensing, and it is the one a camera cannot do.

## The Decisive Difference: Micro-Motion

A 60 GHz radar resolves movement down to a fraction of a millimetre. That is not a marginal improvement over a camera — it is a different regime entirely.

Breathing moves the chest wall by an amount well within that resolution, and the heartbeat moves it by less — but still by enough to be measured. A person lying still under a blanket produces no visible change whatsoever in a video frame, while a radar pointed at the same person measures the chest movement continuously, through the blanket, in complete darkness.

This is the entire reason sleep monitoring — breathing rate, heart rate, sleep staging — is a radar application and not a camera application. It is not that radar is a better camera. It is that the quantity being measured is invisible to a camera.

## Where the Camera Is Genuinely Better

It is worth being blunt about this, because the marketing in this category often is not.

**Radar is not a camera replacement for seeing shape.** The return from a human body is a sparse set of reflection points, not a picture. If your question is "roughly what posture is this person in," or "what is in the room," a camera answers it more directly and more reliably.

Radar does not identify who a person is. It does not read facial expressions. It does not tell you what object someone is holding. It cannot show you the room. If any of those are requirements, you need a camera, or both sensors together.

A realistic way to think about it: radar tells you *that* something is moving and *how* it is moving at a very fine resolution. A camera tells you *what* it is.

## Practical Differences

| Factor | mmWave radar | Camera |
|---|---|---|
| Complete darkness | Works — generates its own signal | Needs light or infrared |
| Privacy | No image, no lens, nothing identifiable | Produces identifiable video |
| Through blankets, steam, smoke | Sees through them | Blocked |
| Identifies a person | No | Yes |
| Rough posture | Inferred, coarser | Clearer and more direct |
| Breathing / heart rate | Measurable | Not measurable |
| Output | Abstract data (range, velocity, phase) | An image a human can read |
| Typical cost | Higher per unit | Lower per unit |
| Debugging | Harder — output is not visual | Easy — look at the frame |

## How to Choose

The decision usually comes down to two questions.

**First, is privacy or darkness a requirement?** If the sensor goes in a bedroom, a bathroom, or anywhere a camera would not be acceptable, the choice is made for you. Radar produces no image, which is the whole argument.

**Second, what is the smallest motion you need to detect?** If the answer is breathing, heartbeat, or a person lying still, only radar can measure it. If the answer is "a person walked into the room" or "someone is sitting at the table," a camera does that job more cheaply, and radar does it without the privacy problem.

For fall detection, sleep monitoring, and presence sensing in private spaces, radar is the practical answer. For "show me what is happening in the room," it is not.

## What Neither One Is

One more thing worth stating plainly: radar does not identify individuals and does not produce images. It measures motion, distance, and presence. A consumer radar that reports breathing rate or sleep stages is a wellness device — it is not a medical instrument, and it does not diagnose anything.

## Summary

- A camera is better at **shape and rough posture**, and costs less. Its output is directly interpretable.
- A radar is better at **motion too small to see** — breathing, heartbeat, a person lying still — and at sensing **in darkness, through materials, without producing an image**.
- Radar does not replace a camera for seeing what is in a room, and a camera cannot do what radar does for micro-motion.
- For bedrooms and bathrooms, the deciding factor is usually that **radar produces no image at all**.

Pick based on what you are measuring, not on which sensor sounds more advanced.
