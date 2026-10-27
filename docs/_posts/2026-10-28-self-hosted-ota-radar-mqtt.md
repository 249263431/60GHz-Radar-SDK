---
layout: post
title: "Updating Radar Firmware Without a Cloud: Self-Hosted OTA over MQTT"
date: 2026-10-28 08:00:00 +0800
categories: [guide]
tags: [ota, firmware, mqtt, radar, selfhosted]
---

Most connected devices want you to push firmware through somebody's cloud. These do not ask for that. The sleep and fall radars fetch an image from a URL you hand them, and the command arrives on the same MQTT connection the telemetry already uses. Nothing has to leave your network, and there is no vendor account in the path.

This article is the practical shape of that: which units support it, the three commands involved, how to confirm an update actually landed, and the operational precautions the protocol document leaves to you.

> Everything below comes from the Wi-Fi protocol documents for the sleep and fall units. Where they do not specify something, this article says so rather than filling the gap.

## Scope first

| Unit | Over-the-air update via this protocol |
|---|---|
| Sleep radar (60 GHz, Wi-Fi) | Yes — topic family `/Radar60SP/<DeviceID>/sys/property/{post,set}` |
| Fall radar (60 GHz, Wi-Fi) | Yes — topic family `/Radar60FL/<DeviceID>/sys/property/{post,set}` |
| Presence radar (24 GHz, Wi-Fi) | **No such command appears in its protocol document** |

That last row matters. The presence unit's document specifies `online`, `version`, `active` and `passive` report topics plus a `property/set` command topic — and no update command at all. If you need to update those, that is a question for the hardware, not for this protocol.

Commands go to the unit's `.../sys/property/set` topic; everything it reports arrives on `.../sys/property/post`.

## Three commands

**1. Select the target module** — required before every update.

```json
{"version": "1.0", "method": "ota_set", "params": {"module": "0"}}
```

`0` is the Wi-Fi module, `1` is the radar module; the default is `0`. The documentation is explicit that this control command has to be sent before each update — it is not a one-time setting you configure and forget.

**2. Hand over the image location.**

```json
{"version": "1.0", "method": "ota_set",
 "params": {"otaInform": {"firmwareUrl": "http://192.168.31.88/app.rbl", "port": "8090"}}}
```

The device fetches the firmware from that URL and port and starts updating. A failure comes back as:

```json
{"version": "1.0", "opt": "otaInform", "res": "fail", "msg": "ota failed!"}
```

**3. Observe progress.** The report is described as an active report during update, sent with method `ota_post`:

```json
{"version": "1.0", "method": "ota_post", "params": {"downloadProgress": "0"}}
```

Values run `0`–`100`. Subscribe to the unit's report topic before you start; if your particular unit reports it somewhere else, record that during commissioning rather than assuming.

**After a successful update**, the module restarts and then reports its version number. Treat that report — not the arrival of `100` — as the completion signal.

## Confirming the result

The point of verification is that "the transfer finished" and "the unit now runs the new code" are two different statements.

- **Sleep and fall units** carry `firmwareVersion` and `firmwareVersionWiFi` in their property list, and the protocol states that the post-restart report carries the version number after updating. Compare against the value you recorded before starting.
- **Presence units** publish a version snapshot on `/RadarHP/<DeviceID>/sys/version/post` containing `radarID`, `softwareVersion`, `firmwareVersion` and `protocolVersion`. These are documented as read-only — they cannot be written, only read. There is also a request command for soliciting it on demand: publish `{"version":"1.0","method":"get_version","params":{}}` to `/RadarHP/<DeviceID>/sys/property/set`.

Keep the before-and-after version pair in the same commissioning record you keep for everything else. It is the only evidence you have later.

## Running an update

```python
import json
import paho.mqtt.client as mqtt

BROKER, PORT = "192.168.1.2", 1883
DEV = "<DeviceID>"                        # the MAC, printed on the label
SET_TOPIC  = f"/Radar60SP/{DEV}/sys/property/set"
POST_TOPIC = f"/Radar60SP/{DEV}/sys/property/post"

IMAGE_URL, IMAGE_PORT = "http://192.168.1.50:8090/app.rbl", "8090"
TARGET = "0"                              # "0" Wi-Fi module, "1" radar module


def command(client, method, params):
    client.publish(SET_TOPIC,
                   json.dumps({"version": "1.0", "method": method, "params": params}),
                   qos=1)


def on_message(client, userdata, msg):
    payload = json.loads(msg.payload.decode())
    params = payload.get("params", {})
    if "downloadProgress" in params:
        print("transfer:", params["downloadProgress"], "%")
    if payload.get("opt") == "otaInform" and payload.get("res") == "fail":
        print("update failed:", payload.get("msg"))
    if "firmwareVersion" in params:
        print("reported version after restart:", params["firmwareVersion"])


client = mqtt.Client()
client.username_pw_set("backend", "...")
client.on_message = on_message
client.connect(BROKER, PORT, keepalive=60)
client.subscribe(POST_TOPIC, qos=1)
client.loop_start()

command(client, "ota_set", {"module": TARGET})          # select the target first
command(client, "ota_set", {"otaInform": {"firmwareUrl": IMAGE_URL, "port": IMAGE_PORT}})
client.loop_forever()
```

That is the whole mechanism — two publishes and a subscription. The rest is procedure.

- **Declare a maintenance window.** The unit is unreachable while it rewrites and restarts. Your monitoring will interpret that as a failure unless you suppress it — see [Online Says It's Connected, HeartBeat Says It Works]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %}).
- **Select the target deliberately.** `module` defaults to the Wi-Fi module. Sending a radar image to it is a mistake the command will not catch for you.
- **Make sure the URL is reachable from the device.** Same subnet or routable, firewall permits that port, and never `localhost` or `127.0.0.1` — the address is resolved by the radar, not by the machine issuing the command.
- **Stage it.** One unit first. Watch `downloadProgress` reach `100`, wait for the restart, confirm the version changed, then proceed with the rest.
- **Keep the previous image served.** Given what is not documented below, this is your only fallback.
- **Do not batch-update overnight in a care deployment.** A unit that comes back with a different behaviour at 03:00 is discovered by nobody until morning.

## A minimal image host

You need nothing more than a static file server on the management network:

```
python -m http.server 8090 --directory ./firmware
```

which makes `http://<host>:8090/app.rbl` available to the units. Use it on a segregated network or behind whatever reverse proxy you already trust; it is plain HTTP with directory listing enabled, and it is a build artifact service, not a public endpoint.

## What the documents do not say

Stated plainly, because guessing here costs hardware:

- **Transport.** Every example uses `http://`. HTTPS support is not stated. Plan for plain HTTP on a network you control.
- **Rollback.** No rollback command appears. Assume you cannot command a unit back to the previous revision — which is why keeping the old image available and piloting one unit first are not optional.
- **Duration and interruption.** No update duration is given, and the documents describe failure only as the `ota failed!` response. Behaviour after power loss mid-update is undocumented.

The consequence is a procedural rule rather than a technical one: update when someone is present to observe, one unit first, with the previous image still served.

## Checklist

- [ ] Unit confirmed to be one of the two that document this command
- [ ] `module` selected explicitly before every update — Wi-Fi module vs radar module
- [ ] Image URL and port reachable from the device's network, not from yours only
- [ ] Subscribed to the report topic *before* publishing either command
- [ ] Monitoring suppression window opened, and closed only after the version report arrives
- [ ] `downloadProgress` observed to `100`, restart observed, version compared against the pre-update record
- [ ] Previous firmware image still available at a known URL
- [ ] Staged: one unit, verified, then the cohort

## Where to go next

- Not alarming yourself during the restart: [Online Says It's Connected, HeartBeat Says It Works]({{ site.baseurl }}{% post_url 2026-10-27-online-heartbeat-radar-health %})
- Reading the stream these commands travel on: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- Keeping device identities straight at scale: [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %})
- The broker underneath it: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
