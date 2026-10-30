---
layout: post
title: "What Data-Driven Algorithms Changed in Contactless Sleep Sensing, and What They Could Not"
date: 2026-10-31 08:00:00 +0800
categories: [fundamentals]
tags: [sleep staging, radar, algorithm, mmwave, contactless sensing]
---

The largest change in contactless sleep sensing over the last few years has not been in the radio. The same bands, similar front-ends, comparable antenna layouts — what changed is the layer that turns motion, respiration and heart rate into a statement about sleep.

That layer used to be written by hand. It is increasingly learned from data. This article is about what that migration actually bought, and — more importantly — about the part of the problem it does not touch.

## The hand-tuned era

A rule-based sleep stage classifier looks like a set of thresholds. Movement below a value for a number of minutes means light sleep; a longer quiet interval with a slower, more regular breathing rate means deep sleep; a burst of movement followed by stillness means a transition. Each threshold is chosen by a person, tested on a handful of nights, and adjusted.

The weakness is not that the rules are wrong. It is that the variables they would need are not expressible as constants:

- **Individual differences.** Resting heart rate, breathing depth and the amount of movement during sleep vary substantially between people, and between an eighty-year-old and a thirty-year-old.
- **Bedding and season.** A thick winter quilt changes both the movement signature reaching the radar and the thermal environment. So does a fan running in August.
- **One person or two.** Two bodies in the same bed produce a composite signal no single-person rule describes.
- **Body position.** Lying on the back, on the side, or curled up changes which parts of the torso move and how much of that motion reaches the antenna.

Each of these would need its own branch, and the branches interact. A system built this way works in the conditions it was tuned in and degrades quietly outside them.

## What learning from data changed

The difference is not that a model is smarter than a person. It is that the decision boundary is fitted rather than guessed.

Instead of one person choosing where to put a threshold, a large number of labelled nights are used to find which combinations of features actually separate the states. The features are the same ones the hand-tuned rules used — movement over time, breathing rate and its regularity, heart rate and its variability, time since getting into bed, transitions between positions — but they are considered together, as a pattern over a window rather than as independent conditions.

Three practical consequences follow.

**Stability improved first.** The most visible change is not a more accurate stage label but fewer of them. A hand-tuned classifier can flicker between light and deep several times in a minute when a value sits near a boundary. A fitted boundary produces a smoother sequence, which is what makes a night's output readable at all.

**The edges got better.** Falling asleep and waking up are transitions with recognisable signatures — breathing rate changes, the movement pattern changes, and then posture settles. Learned models handle these boundaries better than a rule that waits for a fixed number of quiet minutes.

**Individual baselines became usable.** Once the model is fitted against many nights from many people, it can adapt to a particular person's typical pattern rather than comparing everyone against one absolute threshold. This is where most of the practical gain has come from.

## What did not change

This is the part worth being precise about, because the improvement in stability is easily read as an improvement in accuracy against ground truth.

**The inputs did not change.** A contactless radar observes body movement, respiration, heart rate, and — depending on the configuration — position and posture. It does not observe brain activity, eye movement, or muscle tone. Those are the signals that sleep stages are defined by.

**The clinical definition did not change.** Sleep staging comes from polysomnography: EEG for cortical activity, EOG for eye movement, EMG for muscle tone. A stage label produced without any of those three is an inference from correlates, and it stays an inference no matter how the inference is computed. The reasoning is set out in [Choosing a Sleep Sensing Approach]({{ site.baseurl }}{% post_url 2026-10-19-sleep-sensing-ring-watch-radar-module-infrared %}).

**Agreement with a wearable is not accuracy against reality.** This distinction matters more as models become better. If a model is trained or tuned so that its output matches a wristband or a ring, what improves is **agreement with that device**. The wearable is itself estimating stages from an accelerometer and an optical pulse sensor, and its own agreement with polysomnography is limited. Two devices agreeing with each other tells you they are consistent. It does not tell you either is right.

That is not an argument against the practice — comparing against a wearable is often the only large-scale source of labels available outside a sleep laboratory. It is an argument for keeping the two statements separate when describing what a device does.

**The population in the training data is still the population it works on.** A model fitted on one set of people, in one set of beds, with one range of body types, carries those conditions with it. The failure mode is not random error; it is systematic error on whoever was under-represented.

## What the radar actually publishes

The published output for the sleep radar makes the boundary visible, which is useful: the limits are on the surface rather than buried in an opaque score.

| Field | Content |
|---|---|
| `sleepStatus` | Sleep state |
| `getIntoBed` | In bed / out of bed |
| `awakeDuration` | Time awake |
| `lightSleepDuration` | Time in light sleep |
| `deepSleepDuration` | Time in deep sleep |
| `sleepScore` | Sleep score |
| `sleepComprehensiveStatus` | Composite sleep state |
| `sleepQualityAnalysis` | Sleep quality analysis |
| `abnormalSleep` | Sleep abnormality flag |

Awake, light and deep are present. **REM is not.** There is no rapid-eye-movement stage in the published protocol, which is the honest outcome of the constraint described above: eye movement is not among the measurable inputs, so a REM label would have to be inferred from heart-rate and movement patterns alone.

The supporting measurements arrive at their own cadences — breathing rate and heart rate roughly every three seconds, the body-movement index about once a second on the finished units — which is why staging is computed over windows rather than from a moment.

## How to read the output

Given all of the above, the output is best used as it was built to be used:

- **As a trend, not a measurement.** Nightly time in bed, number of times out of bed, and the distribution across states are meaningful when compared against the same person's previous weeks. A single night's absolute number is much weaker. This is the same principle as [Storing Radar Data: Retention, Downsampling, and Why Trends Beat Instant Values]({{ site.baseurl }}{% post_url 2026-10-26-storing-radar-data-retention-trends %}).
- **As a prompt, not a diagnosis.** `abnormalSleep` and `sleepScore` are flags that something in the night differed from the usual pattern. What follows from a flag is a person checking on a person.
- **With the stage labels coarse.** Three states plus an awake duration describe a night at a resolution that is defensible. Finer claims are not supported by the inputs.

Contactless sensing is useful precisely because nobody has to wear or charge anything, and because it works night after night without anyone remembering to enable it. That is worth a great deal. It is a different thing from a sleep study, and it is better for the comparison to be stated than left implied.

## Checklist

- [ ] Stage output treated as an inference from movement, respiration and heart rate — not as a measurement of sleep physiology
- [ ] Agreement with a wearable understood as consistency, not as validation against polysomnography
- [ ] Nightly values read as a trend over weeks rather than as absolute figures
- [ ] Score and abnormality flags used as prompts for a person to check, not as findings
- [ ] Bedding, season, second occupant and body position recorded, because all four move the signal
- [ ] Any change in a person's pattern confirmed by a person before it is acted on

## Where to go next

- The comparison across four sensing approaches, and where polysomnography sits: [Choosing a Sleep Sensing Approach]({{ site.baseurl }}{% post_url 2026-10-19-sleep-sensing-ring-watch-radar-module-infrared %})
- What the sleep radar publishes, field by field: [Sleep Radar Guide]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %})
- Keeping months of it, and why the trend is the signal: [Storing Radar Data]({{ site.baseurl }}{% post_url 2026-10-26-storing-radar-data-retention-trends %})
- Reading the stream directly: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- Treating quoted specifications with care: [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ site.baseurl }}{% post_url 2026-10-17-radar-spec-limits %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
