---
layout: post
title: "Reading Radar Data in Your Own Application: JSON Contract, QoS, Last Will, and Sparse Payloads"
date: 2026-10-25 08:00:00 +0800
categories: [guide]
tags: [mqtt, json, backend, radar, integration]
---

Home Assistant is one way to consume radar data, and it is a good one — see the [integration overview]({{ site.baseurl }}{% post_url 2026-10-18-home-assistant-radar-integration %}). But if you are building a product, a care platform, or simply a small service of your own, you will read the broker directly. Nothing in the radar's output is Home Assistant specific: it is JSON over MQTT, on topics you already know.

This article is the contract layer — what the messages look like, what arrives when, and which MQTT settings actually matter. It assumes a broker is running (see the [Artemis setup guide]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})) and that at least one radar is configured (see the [setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %})).

## Two payload shapes

**Finished units** publish a JSON property protocol on their report topic. The DeviceID in the path is the device's MAC address, printed on the label and shown on the configuration page — see [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %}) for the difference between DeviceID and ClientID.

```json
{"version": "1.0", "method": "post", "params": {"someoneExists": "1"}}
```

Two things to internalise: the values arrive as **strings**, and only the fields that changed are present.

**A UART-only module behind your own gateway** publishes whatever your firmware sends. The [ESP32-S3 gateway]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %}) publishes a flat payload:

```json
{"breath": 22, "sleep": 1}
```

Because you own that code, you also own the contract — but the advice below applies to it just the same.

## What arrives when, and why that matters

The radars do not send every field in every message. The cadences are documented per model in the product guides; the pattern is always the same:

| Data | Typical cadence |
|---|---|
| Presence, motion status | On change, within about 0.5 s |
| Breathing rate, heart rate | Every 3 s |
| Body movement | Every 1 s (finished) / every 5 s (module) |
| Sleep stage | Every 10 min while in bed |
| Sleep score and report | Once, when the sleep session ends |
| Fall status and position | On a fall event |

The consequence is the single most important rule in this article:

> **Never assume a field is present.** Read only what arrived, and keep the last known value for everything else.

A consumer written as `payload["params"]["heartRateValue"]` will throw on the first message that carries only presence. That is exactly the failure the Home Assistant guides avoid with their `if defined … else this.state` pattern, and it applies to your own code in the same way.

## Topic patterns

- Everything from one device: `/Radar60SP/<MAC>/#`
- One function across all sleep radars: `/Radar60SP/+/sys/property/post`
- Commands to the device: `.../sys/property/set`

Some models publish extra topics worth knowing. The presence radar, for example, reports **online/offline** on `/RadarHP/<DeviceID>/sys/online/post` and its version on `.../sys/version/post` — use those for device health instead of inferring it from whether data happens to be arriving.

## MQTT settings that matter

- **QoS.** QoS 0 is fine for high-frequency vitals on a LAN, where the next reading arrives in three seconds anyway. Use QoS 1 when a missed message is expensive — a fall event, for instance. QoS 2 is unnecessary here and adds round trips.
- **Persistent session.** If your backend must not miss messages while it restarts, connect with a fixed client ID, `clean_session=False`, and subscribe at QoS 1. The broker then queues what you missed.
- **Keepalive.** 60 s is comfortable on a LAN. It only sets how fast a dead connection is noticed.
- **Last Will and Testament.** Register an LWT so the broker publishes an offline status for your service if it drops. Where the radar publishes its own online topic, prefer that for the device's own health and use LWT for your service.
- **Retained.** Do not retain high-frequency report topics — you do not want a broker to hand a new subscriber a three-second-old breathing rate as if it were current state. Retaining is reasonable for a "last known status" topic you publish yourself.
- **Version field.** Finished units include `version` in the JSON. Read it, log it, and ignore unknown keys rather than failing: new fields can appear in later firmware.

## A minimal consumer

Python with `paho-mqtt`, keeping last-known values and reconnecting:

```python
import json
import time
import paho.mqtt.client as mqtt

BROKER = "192.168.1.2"
PORT = 1883
USERNAME = "backend"
PASSWORD = "..."
TOPIC = "/Radar60SP/<MAC>/sys/property/post"

state = {}

def on_connect(client, userdata, flags, rc):
    client.subscribe(TOPIC, qos=1)

def on_message(client, userdata, msg):
    try:
        payload = json.loads(msg.payload.decode())
    except ValueError:
        return
    params = payload.get("params", {})
    # Sparse payload: only the fields that changed are present.
    if "breathValue" in params:
        state["breath"] = params["breathValue"]
    if "heartRateValue" in params:
        state["heart"] = params["heartRateValue"]
    if "sleepStatus" in params:
        state["stage"] = params["sleepStatus"]
    if "getIntoBed" in params:
        state["in_bed"] = params["getIntoBed"]
    state["updated"] = time.time()

client = mqtt.Client(client_id="radar-backend-01", clean_session=False)
client.username_pw_set(USERNAME, PASSWORD)
client.on_connect = on_connect
client.on_message = on_message
client.will_set("radar/backend/status", "offline", qos=1, retain=True)

while True:
    try:
        client.connect(BROKER, PORT, keepalive=60)
        client.publish("radar/backend/status", "online", qos=1, retain=True)
        client.loop_forever()
    except OSError:
        time.sleep(5)
```

That is the whole shape: **subscribe, merge what arrived, reconnect.** Replace `<MAC>` with the DeviceID from the device label, and keep one `state` dict per device if you have several.

## From here

What you do with `state` is your product, not the radar's job. The fall radar detects and reports; escalating to a notification is step five of the alarm chain and belongs to your system — see [Building an Aging-in-Place Care System with Three 60GHz Radars]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %}) for one worked escalation design.

## Where to go next

- Prerequisites: [setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}) and [running your own broker]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- Identifiers and topics per device: [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %})
- The Home Assistant path instead: [integration overview]({{ site.baseurl }}{% post_url 2026-10-18-home-assistant-radar-integration %}) and [R60ABD1 in Home Assistant]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-home-assistant-mqtt %})
- UART-only modules: [ESP32-S3 gateway]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %}) and [Minimal UART Parser]({{ site.baseurl }}{% post_url 2026-10-22-minimal-uart-parser-sip-s %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
