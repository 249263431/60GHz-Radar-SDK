---
layout: post
title: "Run Your Own MQTT Broker for 60GHz Radar: An Apache Artemis Setup Guide"
date: 2026-10-23 08:00:00 +0800
categories: [guide]
tags: [mqtt, apache artemis, broker, radar, home-assistant, provisioning]
---

The [radar setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}) opens with a hard prerequisite: **the broker has to be ready before the radar is configured**. It tells you which four values to type into the module — server address, ClientID, username, password — and lists your options (self-hosted EMQX or Mosquitto, a managed cloud broker, or a public test broker for a first try). What it does not do is stand one up.

This article is that missing step. It builds an MQTT broker you control, using **Apache Artemis** — the broker we use ourselves while debugging radar prototypes. If Home Assistant already talks to *some* broker, you can skip to the sections you need.

> Written against Apache Artemis **2.50.0**. The CLI and `broker.xml` layout are stable across the 2.x line, but always diff your own copy before editing.

## What you need

- A machine on your LAN that stays on (mini-PC, NAS, or server) with a Java 17 runtime
- About 1 GB of free RAM
- A broker account plan: **one account per client**, which we create below

Our radars talk over **plain TCP**, so everything here uses port **1883**.

## Install Artemis

Grab the current release from the official download page <https://artemis.apache.org/components/artemis/download/>, then create an instance:

```bash
tar zxvf apache-artemis-2.50.0-bin.tar.gz
cd apache-artemis-2.50.0/bin
./artemis create /opt/radar-broker
```

The `create` step asks for a console username and password, and whether to allow anonymous login. Answer **N** to anonymous — every client gets its own credentials.

Start it in the foreground to check it comes up:

```bash
/opt/radar-broker/bin/artemis run
```

For a machine that reboots, install it as a service instead:

```bash
/opt/radar-broker/bin/artemis-service install
/opt/radar-broker/bin/artemis-service start
```

Log into the console at `http://<host>:8161/console`.

**That port, 8161, is the management console. It is not the MQTT port.** Typing it into the radar's configuration page is the most common reason a module "connects" but nothing arrives.

## Enable MQTT (the step that is not the default)

Artemis is a multi-protocol broker. Its default acceptor serves its own core protocol on **61616**; MQTT is something you ask for explicitly.

Stop the broker, edit `/opt/radar-broker/etc/broker.xml`, and add one acceptor inside `<acceptors>`:

```xml
<acceptor name="mqtt">tcp://0.0.0.0:1883?protocols=MQTT</acceptor>
```

Restart and confirm it listens:

```bash
ss -ltnp | grep 1883
```

Keep three ports straight:

| Port | What it is |
|---|---|
| 8161 | Management console (HTTP) — humans |
| 61616 | Artemis core protocol — not MQTT |
| 1883 | MQTT — what your radars and Home Assistant use |

Variations use the same pattern: MQTT over WebSocket (`protocols=MQTT,WS` on another port) or TLS on 8883. Both are documented for your version; start with plain 1883 on a trusted LAN.

## Create one account per client

With the default properties-based security, add users in `etc/artemis-users.properties` and map them to roles in `etc/artemis-roles.properties`. Create three:

- `radar` — for finished 60 GHz modules
- `radar-gw` — for the ESP32 gateway
- `ha` — for Home Assistant

Then restart. Because anonymous access is off, any client that connects without credentials is rejected — the second most common cause of "the radar shows no data".

If you want isolation between them, add a `<security-setting>` block in `broker.xml` restricting each role to its own topic prefix.

## Point the finished modules at it

You now have exactly the four values the setup guide asks for:

| Field | Value |
|---|---|
| Server address | `tcp://<Artemis host IP>:1883` |
| ClientID | **one unique ID per radar** |
| Username / Password | the account you created above |

Now go back to the [setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}): hold the button for 10 s, join the `RadarConfig` hotspot, open `192.168.0.1`, enter them, then use `BK_BLE5.apk` to send your **2.4 GHz** Wi-Fi credentials.

Verify by subscribing to the model's report topic with any MQTT client — `/Radar60SP/{DeviceID}/#` (sleep), `/Radar60FL/` (fall), `/RadarHP/` (presence) — exactly as in Step 5 of the setup guide.

## Point the ESP32 gateway at it

The R60ABD1 has no network of its own; the [ESP32-S3 gateway]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %}) reads its UART frames and publishes them. On the firmware side the broker is three lines of configuration:

```cpp
const char* mqtt_server   = "<Artemis host IP>";
const int   mqtt_port     = 1883;
const char* mqtt_username = "radar-gw";
const char* mqtt_password = "<password>";
// connect with a unique client id, then:
client.publish("radar/R60ABD1/state", payload.c_str());
```

Artemis speaks MQTT 3.1.1, which is what `PubSubClient` uses. Give each gateway a unique ClientID.

## Point Home Assistant at it

Add the MQTT integration with the same host, port 1883, and the `ha` account. Then declare entities as shown in the [R60ABD1 Home Assistant guide]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-home-assistant-mqtt %}) for UART-only sensors, or in the per-model guides for finished modules.

## Verify end to end

1. Console → **connections**: one line per radar, one for the gateway, one for Home Assistant.
2. Subscribe to a report topic and watch raw JSON arrive.
3. In Home Assistant, check **Developer → States** for your entities.

## Troubleshooting

| Symptom | Usual cause |
|---|---|
| Connection refused on 1883 | MQTT acceptor missing from `broker.xml`, or broker stopped |
| Console works, MQTT clients rejected | Anonymous disabled but client sent no credentials |
| Module online, no data | Port configured as 8161 or 61616 instead of 1883 |
| Radars disconnecting each other | Duplicate ClientID |
| Console unreachable but data flowing | 8161 firewalled (independent of 1883) |
| Broker will not start | Missing Java, or see `log/artemis.log` |

## Where to go next

- The two-Step 2.4 GHz / MQTT device setup: [radar setup guide]({{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %})
- The UART-only path: [R60ABD1 + ESP32-S3 gateway]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-esp32-s3-sip-s-mqtt %})
- Consuming it: [R60ABD1 in Home Assistant]({{ site.baseurl }}{% post_url 2026-10-22-r60abd1-home-assistant-mqtt %})
- Which module does what: [60 GHz / 24 GHz module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})

Both the finished 60 GHz radars and the bare modules are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
