---
layout: post
title: "Online Says It's Connected, HeartBeat Says It Works: Reading Two Status Signals Instead of Guessing From Silence"
date: 2026-10-27 08:00:00 +0800
categories: [guide]
tags: [mqtt, radar, monitoring, reliability, health]
---

Two fields in a radar's MQTT output get mistaken for each other constantly, and mistaking them is how a health monitor ends up trusted when it should not be.

They answer different questions:

- **`online` answers "is the link up?"**
- **`heartBeat` answers "is the radar itself working?"**

| | `online` | `heartBeat` |
|---|---|---|
| Question answered | Is this device reachable? | Is this device operating correctly? |
| Reported values | `"0"` offline, `"1"` online | `"0"` abnormal, `"1"` normal |
| Layer | MQTT connection and broker session | The radar's own operating state |
| Reported when | On state change only | On state change only |
| Available on | Sleep radar, fall radar, presence radar | Sleep radar, fall radar |

The presence radar publishes its online state on a dedicated topic — `/RadarHP/<DeviceID>/sys/online/post`; the sleep and fall radars publish `online` as an attribute in the same property payload as everything else. Note the asymmetry: **there is no `heartBeat` in the presence radar's protocol documentation.** The other two report it like this:

```json
{"version": "1.0", "method": "post", "params": {"online": "0"}}
{"version": "1.0", "method": "post", "params": {"heartBeat": "0"}}
```

## Why this distinction is not academic

Consider what has to be true for any `heartBeat` message to reach your code: the radar has power, it is associated with the Wi-Fi access point, its MQTT session is established, and you are subscribed to its topic. **A delivered `heartBeat` therefore already proves the link is working.**

From that, one conclusion follows immediately:

> **`heartBeat` cannot report a connectivity failure. Physically and logically, it cannot — a radar that has lost its link has no way to deliver that news.**

The reverse holds for `online`. An `online: "0"` message is not written by the radar that went offline; it is generated **by the broker** when the session it was holding stops answering. The radar contributes nothing to that judgement.

So neither field overlaps with the other, and neither can substitute for the other:

- If you only watch `online`, you will see "connected" while the radar sits there reporting a fault about itself.
- If you only watch `heartBeat`, every network outage looks like nothing at all.

## The trap: silence is not evidence

Both fields are documented as **reported on state change**, not on a timer. That single sentence is where most monitoring code goes wrong.

- **Occupancy.** The presence state is event-driven. A room whose state does not change for three hours produces no occupancy messages, and that is correct behaviour — not a fault.
- **Heartbeat.** A radar that boots normally and stays normal has no reason to send `heartBeat` at all, because no state change occurred.
- **The one periodic attribute.** In the sleep and fall protocols, the only attribute the documentation marks as periodically reported is `workDuration` (4 bytes, value range `0x3C`–`0xffffffff`, unit seconds) — and it is marked "not yet available" in the same line. `workDurationReportFrequency` carries the same note. **Do not build a keepalive on either of them.**

So the tempting pattern — "if I have not heard from the device in 60 seconds, it must be gone" — has no foundation here. Absence of messages is the normal state of a working installation.

## What you can do instead

| `online` | `heartBeat` | Conclusion | Action |
|---|---|---|---|
| 1 | 1 | Link up, radar reports normal | — |
| 1 | 0 | **Link up, radar reports a fault** | Raise the higher-severity alarm; this is the case only `heartBeat` can surface |
| 0 | — | Session dropped, as judged by the broker | Investigate the link; the radar's own state is unknown |
| no messages | no messages | Most likely nothing changed | **Nothing can be concluded** |

Treat the two as different classes of event with different severities. A lost link is an infrastructure ticket; a radar reporting itself abnormal is a hardware ticket, and it is the one that should wake someone up.

## Two rules you must not break

**1. Do not disable `online` or `heartBeat` with `limit_set`.** The Wi-Fi protocols expose a reporting-suppression command: `limit_set` / `limit_get`, where `1` means "suppress this attribute" and `0` means "allow it". It exists for the attributes that report frequently — `someoneExists`, `motionStatus`, `movementSigns`, `humanDistance`, `humanPosition` — and it is a legitimate way to cut traffic and noise. Both `online` and `heartBeat` appear in that list. Suppress them and you remove the only two signals that stand between you and a timeout guess.

**2. Do not try to poll `heartBeat`.** Every neighbouring attribute in those documents — `initFinish`, `workDuration`, `productMode` — carries the line "this attribute supports querying" followed by a `get` example. `heartBeat` does not. The only `?` it ever appears with is inside `limit_get`, and that question means "is reporting suppressed for this attribute", not "what is its current value". There is no documented way to ask the radar for that value on demand.

That leaves one workable approach for the things you genuinely need on demand: **query the attributes that do support it.** If you must confirm a radar has finished starting up, `initFinish` (`0` not complete, `1` complete) is queryable. `heartBeat` is not, and no amount of polling will change that.

## A minimal health monitor

Subscribe, merge the sparse payload, and raise the two classes separately:

```python
import json, time
import paho.mqtt.client as mqtt

TOPIC = "/Radar60SP/<MAC>/#"

state = {}
last_any = {}


def on_message(client, userdata, msg):
    try:
        payload = json.loads(msg.payload.decode())
    except ValueError:
        return
    params = payload.get("params", {})
    if not params:
        return

    # Only the fields that changed are present.
    for k in ("online", "heartBeat", "initFinish"):
        if k in params:
            state[k] = params[k]

    last_any["ts"] = time.time()

    # Two different failures, two different severities.
    if params.get("online") == "0":
        raise_alarm("link", device=msg.topic, severity="infra")
    if params.get("heartBeat") == "0":
        raise_alarm("sensor", device=msg.topic, severity="high")
```

Note the shape: nothing here fires on elapsed time. Every action is triggered by an arriving value.

If you still want a timeout as a last resort, the threshold must come from measurement, not habit. Log the longest interval your real installation goes without any message during normal operation, then add margin to that. A number you made up will either fire constantly or never fire.

## Measure these at commissioning

Two things the documentation does not state, and that you should record per installation rather than assume:

- **Does the radar announce `heartBeat: "1"` on power-up?** A device that boots healthy may never have a reason to report it. Power one up on the bench with a subscriber attached and write down what actually arrives, and how long after power-up it arrives.
- **Does `online: "1"` get republished after a reconnect?** If it does not, your consumer must treat the absence of a change as "still online" rather than as unknown.

Both belong in the same commissioning note as the empty-room baseline from [Using Distance and Position Output to Debug a Radar Installation]({{ site.baseurl }}{% post_url 2026-10-24-distance-position-debug-installation %}).

## Maintenance looks like a failure

One consequence of all this that catches people out: the sleep and fall protocols support firmware update over the air — `ota_set` with `otaModule` (`0` Wi-Fi module, `1` radar module, and the command must be sent before every update), `otaInform` giving a firmware URL and port, and `downloadProgress` reporting `0`–`100`. On success the module restarts and then reports its new version number.

During that window the device is legitimately unreachable. If your monitoring treats a missing device as an alarm, a firmware update will page someone. Put planned maintenance in a suppression window, and use the version report afterwards — the presence radar's `/RadarSP/<DeviceID>/sys/version/post` topic, or the `firmwareVersion` attribute — to confirm the update actually landed.

The update server is one you host yourself; the documentation states plainly that you build the web server the firmware is fetched from.

## Two controls worth knowing about

- **Reporting suppression** (`limit_set` / `limit_get`): set `1` to suppress an attribute, `0` to allow it. Use it on the high-frequency attributes to cut traffic and clutter. Leave `online` and `heartBeat` alone.
- **Per-attribute QoS** (`control_qos` in the presence protocol, `qos_set` / `qos_get` with `Qos_post` in the sleep and fall protocols): the default is QoS 1, where `0` is QoS 0, `1` is QoS 1 and `2` is QoS 2. The presence protocol additionally exposes per-attribute keys — `Qos_online`, `Qos_movementSigns`, `Qos_breathRate`, and so on. This is worth knowing because it contradicts the usual assumption that QoS is decided only by the subscriber.

## Checklist

- [ ] `online` and `heartBeat` handled as two separate signals with two separate severities
- [ ] No logic concludes anything from the absence of messages
- [ ] `limit_set` used, if at all, only on high-frequency attributes — never on `online` or `heartBeat`
- [ ] No polling of `heartBeat`; anything needed on demand read from attributes that document query support
- [ ] Power-up behaviour of `heartBeat` measured and recorded
- [ ] Reconnect behaviour of `online` measured and recorded
- [ ] Any timeout threshold derived from measured worst-case silence, not assumed
- [ ] Firmware updates placed in a suppression window, and verified afterwards by version report

## Where to go next

- Getting the stream in the first place: [Reading Radar Data in Your Own Application]({{ site.baseurl }}{% post_url 2026-10-25-reading-radar-data-your-own-app %})
- The broker underneath it: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- Keeping several devices straight: [Running Several Radars in One Home]({{ site.baseurl }}{% post_url 2026-10-24-running-several-radars-one-home %})
- Keeping the history: [Storing Radar Data]({{ site.baseurl }}{% post_url 2026-10-26-storing-radar-data-retention-trends %})
- More questions answered: [the FAQ]({{ site.baseurl }}/faq.html)

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
