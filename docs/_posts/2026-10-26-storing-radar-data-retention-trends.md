---
layout: post
title: "Keeping Months of Radar Data: Retention, Downsampling, and Why Trends Beat Instant Values"
date: 2026-10-26 08:00:00 +0800
categories: [guide]
tags: [mqtt, data storage, time series, radar, retention]
---

The [previous article]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %}) ends with a dict full of the latest values. That is a working state, not a record: restart the process and everything before the restart is gone. This article is the step after — deciding what to keep, for how long, at what resolution, and which numbers actually answer a question anyone cares about.

It matters more than it sounds, because a single instant reading is nearly useless. "Occupied, still, right now" is a fact you could get by looking into the room. The facts that are worth keeping are the ones nobody can observe: how long she slept, how often she got up, whether tonight's activity looks like last month's. Those only exist if you stored the stream.

## Start from what actually arrives

Recapping [the JSON contract]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %}): finished units publish `{"version":"1.0","method":"post","params":{...}}` where values are strings and **only changed fields are present**, while a UART module behind your own gateway publishes whatever flat shape your firmware chose.

Three properties of that stream decide your storage design:

1. **It is sparse.** Most messages carry one or two fields. A row-per-field and a row-per-message schema are very different things.
2. **It is heterogeneous.** Presence changes on a human timescale; breathing updates every few seconds; a fall event happens once in a blue moon. One retention rule cannot serve all three.
3. **It is a stream of states, not a stream of samples.** Between two messages nothing changed. Storing the interval matters more than storing the moment.

## Four questions before you write a line of code

**How long do you need each kind of data?** Presence events might be worth two years; breathing rate might be worth thirty days. A fall alarm is an incident record — keep it as long as you keep any incident record.

**At what resolution?** Nobody reads three-second breathing samples from four months ago. They read the nightly average.

**Who is allowed to see it?** This is intimate data, and a single mammography of detail is not needed to answer any of the questions above. The safe default is: aggregate early, keep raw shortest, and store it where you own it — see the [self-hosted broker]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %}) rather than a cloud queue.

**Which fields are trustworthy?** Distance and position are documented as **non-critical functions with limited accuracy**, on the sleep and presence radars alike — see [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %}). If you store them, store them as commissioning notes, not as a data product.

## A retention layout that matches cadence

| Tier | Data | Raw window | Then keep |
|---|---|---|---|
| Incidents | Fall events, alarms, alarm acknowledgements | Forever | Forever — this is an audit trail |
| Sessions | Sleep session start/end, score, stage changes | 90 days | Daily summary, indefinitely |
| Presence | In-bed / occupancy state changes | 90 days | Per-day occupancy seconds |
| Vitals | Breathing rate, heart rate | 30 days | Per-minute average, then per-night average |
| Diagnostics | Distance, position, motion level | 7 days | Nothing, except the install-day baseline |

Those numbers are a starting pattern, not a rule; the point is that one retention setting for everything guarantees either a sprawling database or a useless one.

## Cast before you store

The finished units send everything as strings — `{"someoneExists": "1"}`. Insert that into a numeric column and you will discover it later, at the worst possible moment. Normalise on the way in:

```python
INT_FIELDS = {"someoneExists", "getIntoBed", "heartRateValue", "breathValue", "sleepStatus"}

def normalise(params):
    out = {}
    for key, value in params.items():
        if key in INT_FIELDS:
            try:
                out[key] = int(value)
            except (TypeError, ValueError):
                continue          # the sensor sometimes reports an unusable value
        else:
            out[key] = value
    return out
```

Note the `continue`: a breathing sensor reports "no value" states when it cannot resolve a rate. Storing those as 0 invents a reading that did not happen. Skipping them, and letting your last-known value stand, keeps the database honest — the same rule as in memory, applied one layer down.

## Merge, then write

Because payloads are sparse, write only what changed and treat it as valid until something else arrives:

```python
def on_message(client, userdata, msg):
    params = json.loads(msg.payload.decode()).get("params", {})
    row = normalise(params)
    if not row:
        return
    row["ts"] = datetime.utcnow()
    insert_row(row)          # sparse: only the fields in this message
```

Reading back is then an "as of" query — the most recent value of each field at or before a timestamp — which most column stores handle with `last_value(...) OVER (PARTITION BY field ORDER BY ts)`.

## Downsample to what someone will actually read

A per-night row is worth more than the 28,800 three-second samples it came from. Keep a rollup table alongside the raw one and build it once an hour:

```
night_of      DATE       -- the night, not the timestamp: nights do not start at 00:00
device_id     TEXT
in_bed_at     TIMESTAMP  -- first sustained in-bed reading
out_of_bed_at TIMESTAMP
bed_exits     INT        -- count of in-bed -> not-in-bed transitions
away_seconds  INT        -- total time out of bed
avg_breath    NUMERIC    -- median, not mean: it resists outliers
min_breath    NUMERIC
max_breath    NUMERIC
still_ratio   NUMERIC    -- fraction of samples with body movement at 1 %
```

Every one of those columns came from values the radars already publish; none of them required a new sensor or a new field. That is the whole argument for keeping history: the interesting features are computed from ordinary state transitions.

Use **medians and counts, not means**, for vitals. One garbage reading in ninety thousand will drag a mean and leave a median untouched.

## How much space, roughly

Worth doing once so nobody is surprised. A sleep radar publishing breathing every 3 s produces about 20 messages a minute, roughly 1,200 an hour, roughly 29,000 a day. At a sparse payload of well under 200 bytes before indexing, that is single-digit megabytes per device per day in a naive row-oriented store — and an order of magnitude less once you drop the raw stream on schedule and keep only the nightly row.

Do the arithmetic with your own payload sizes rather than trusting that figure; it is arithmetic on documented cadences, not a benchmark.

## Trends worth computing

Genuinely useful, and derivable from nothing but occupancy and breathing:

- **Nightly time in bed and number of exits.** The classic sleep-quality proxy, and the change in it matters more than its value.
- **Baseline activity.** Once you have weeks, "unusually still today" becomes a computable statement rather than a feeling.
- **Room usage by hour.** An occupancy heat map tells you whether a room is used the way the household thinks it is.
- **Slow drift over months.** Some apparent change is the sensor, not the person — see [why radar position drifts]({{ site.baseurl }}{% post_url 2026-10-18-radar-position-drift %}). Keep an install-day baseline precisely so you can tell the two apart.

For an elderly-care setting these roll-up rows are also what a carer can digest: not a live graph, a weekly summary. The escalation logic that turns a summary into a call is described in [Building an Aging-in-Place Care System with Three 60GHz Radars]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %}).

## Checklist

- [ ] Empty-restart test: does anything survive an application restart?
- [ ] Every numeric field cast explicitly; unusable values dropped rather than stored as 0
- [ ] Sparse writes, with an "as of" read path that fills from the last known value
- [ ] One retention window per tier, not one for all data
- [ ] Rollup job scheduled, and verified to produce the same answer as yesterday's data would
- [ ] Distance and position marked as diagnostic, pruned aggressively
- [ ] Data stored where you control it, and not sent anywhere it does not need to go

## Where to go next

- Getting the data in the first place: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- What each field means: [presence]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}), [sleep]({{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}) and [fall]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}) guides
- Why some readings take seconds: [Radar Latency Explained]({{ site.baseurl }}{% post_url 2026-10-17-radar-latency-explained %})
- The broker underneath it: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %}) and [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
