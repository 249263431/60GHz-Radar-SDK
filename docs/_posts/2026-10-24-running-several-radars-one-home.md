---
layout: post
title: "Running Several Radars in One Home: Two Kinds of ID, One Broker, and How to Group Them"
date: 2026-10-24 08:00:00 +0800
categories: [guide]
tags: [radar, mqtt, deployment, home-assistant]
---

Setting up one radar is a single sequence in the [setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}). Setting up five is not the same sequence five times — it is a small design exercise. Almost every multi-radar failure we see traces back to one of four things: a duplicated ClientID, two devices sharing one broker account, topics that cannot be told apart, or two radars looking at the same square metre of floor.

This article is the deployment view: which identifier does what, how to keep them from colliding, where room identity actually lives, and the order to commission in.

## Two kinds of ID, and they are not the same thing

| | **DeviceID** | **ClientID** |
|---|---|---|
| What it is | The device's **MAC address** | The MQTT client identifier |
| Where you see it | Printed on the device label; shown on the configuration page at `192.168.0.1` | Entered on the same configuration page |
| Can you change it | **No** | **Yes — it defaults to the MAC** |
| Where it appears | In the topic path: `/Radar60SP/{DeviceID}/sys/property/post` | Nowhere visible; it identifies the connection to the broker |
| Uniqueness rule | Unique by construction | **Must be unique.** Two devices sharing one ClientID keep kicking each other off the broker |

That last row is the whole reason this article exists. A shared ClientID does not fail loudly — it produces two devices that connect, disconnect, and reconnect in a loop, with data arriving in bursts and gaps that look like a radio problem.

## One broker, many clients

Point every device at the same broker, then give each one its own credentials. The [Artemis setup guide]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %}) shows the broker side: one account per client, anonymous access off. From the radar side it is the same four values as always — server address, port 1883, username, password — entered once per device on its own configuration page.

Before you power anything on, build the table. Fill it in from the labels:

| Device | DeviceID (MAC, from label) | ClientID (default = MAC) | Account | Room |
|---|---|---|---|---|
| Fall radar | `AA:BB:...` | default | `radar` | Bathroom |
| Sleep radar | `CC:DD:...` | default | `radar` | Bedroom |
| Presence radar | `EE:FF:...` | default | `radar` | Living room |

Two habits that save an evening:

- **Leave the ClientID at its default.** The default is already unique, and a custom value is one more thing to get wrong. If you do set one, make it encode the room (`bathroom-fall-01`) — but then the *label* is the only place the MAC is written, so record both.
- **One account per device, or one shared account with a deliberate decision.** A shared account is fine for a home; separate accounts matter when you want to revoke one device, or to restrict which topics each one can publish to.

## Topics are fixed, so put room identity in Home Assistant

Because the DeviceID is a MAC address, the topic path cannot be renamed per room:

```
/Radar60FL/<MAC>/sys/property/post      ← fall radar, bathroom
/Radar60SP/<MAC>/sys/property/post      ← sleep radar, bedroom
/RadarHP/<MAC>/sys/property/post        ← presence radar, living room
```

That is a feature, not a limitation: the MAC is unique forever, so nothing you do later can break the path. The room name belongs in the layer above. In Home Assistant:

1. Assign each device to the matching **area** (Settings → Devices & Services → Devices → area).
2. Rename entities to `Bathroom Fall Status`, `Bedroom Breathing Rate` — not `Sensor 1`, `Sensor 2`.
3. Use the device ID to correlate: when a dashboard shows a fall in the bathroom, the entity name and area already say which bathroom.

The result is that a topic full of MAC addresses is still readable to a human, because nobody ever reads the topic.

## The gateway is the exception

Everything above applies to finished units, which are configured from their own web page. The **ESP32 gateway** for UART-only modules is different: it is your firmware. Both the ClientID and the topic — `radar/R60ABD1/state` — are whatever you set in code, so you are free to name them. See the [ESP32-S3 gateway guide]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %}).

That freedom deserves a rule, not a free-for-all:

- Keep a naming scheme in one place (a comment block, a README, or a spreadsheet) so the next person can decode it.
- If you run several gateways, give each a unique ClientID in firmware and a topic that encodes the room, e.g. `radar/<room>/state`.
- Write the scheme on the enclosure label next to the MAC. Six months from now, that label is the only documentation anyone will read.

## Stop the radars from arguing with each other

Two radars in adjacent rooms can interfere in ways that look like faulty hardware. The mechanisms are covered in [Ghost Targets and Multipath Interference]({{ site.baseurl }}{% post_url 2026-10-19-ghost-targets-multipath-interference %}) and [Radar Position Drift]({{ site.baseurl }}{% post_url 2026-10-18-radar-position-drift %}); the practical rules are:

- **Check coverage before mounting.** A finished presence radar covers roughly a 2.7 m radius; the 24 GHz presence *module* reaches up to about 12 m for a moving person. Two modules in a small apartment will absolutely see each other's rooms.
- **Shrink the trigger range first.** Choosing the smallest scene mode that still covers the room is the single most effective change — see the [presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}) for the mode table.
- **Keep metal out of the beam path.** Metal doors, mirrors, and appliance housings in the line of sight create reflections that appear as targets.
- **Aim away from fans, air-conditioner motors, swaying curtains, and pets.** These are the usual false-alarm sources in every room.

## Commission in this order

1. **Broker first.** It has to be running before any radar is configured — that is the prerequisite the setup guide opens with. Confirm with a test client that it accepts a connection.
2. **One radar at a time.** Configure it, then verify it on its own: subscribe to `/prefix/<MAC>/#` with any MQTT client and watch data arrive. Do not add the next device until the first is clean.
3. **Fill in the table** (DeviceID, ClientID, account, room) as each device comes online — not afterwards from memory.
4. **Then Home Assistant.** Add the MQTT integration, confirm entities appear, assign areas, rename.
5. **Only then build automations.** Cross-room logic written before the entities are stable is logic you will rewrite.

## Acceptance checklist

- [ ] Every device's ClientID is unique (defaults are)
- [ ] Every device has credentials that work, and anonymous access is off
- [ ] Each device's topic is subscribed and reporting, verified against the MAC on its label
- [ ] Two devices left alone for an hour produce no disconnection loop on the broker
- [ ] Each room's entities are named and assigned to the right area
- [ ] Walk the home once and confirm occupancy, presence and in-bed states change as expected
- [ ] Note the mounting position and sensitivity of each unit in the same table

## Where to go next

- Single-unit setup: [Wi-Fi Provisioning & MQTT Setup Guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %})
- The broker: [Apache Artemis setup]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %})
- A worked three-radar example: [Aging-in-Place Care System]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %})
- Hardware wiring without a parser: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}) and [presence relay wiring]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %})
- Module specs: [60 GHz / 24 GHz module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})

Finished radars and modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
