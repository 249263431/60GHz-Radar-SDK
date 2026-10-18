---
layout: post
title: "Two Architectures, Two Protocols: MQTT vs UART on mmWave Radar"
date: 2026-10-18 06:00:00 +0800
categories: [fundamentals]
tags: [architecture, mqtt, uart, protocol, mosquitto]
---
<!-- filename: 2026-10-18-radar-architecture-mqtt-vs-uart.md -->

The most common reason a radar "does not work" is not a defect. It is that the buyer followed the wrong set of instructions.

Our products come in two fundamentally different architectures, and they do not share a protocol. One joins your Wi-Fi network and speaks MQTT. The other has no network at all and speaks binary frames over a serial cable. Both are radar. Almost nothing else about them is the same.

This article explains both, tells you how to recognise which one you have, and shows what you need in order to see the first message come out of it.

## Start Here: Which One Do You Have?

| You have | How to recognise it | Interface | Protocol |
|---|---|---|---|
| **Finished radar** | Enclosed housing, no exposed pin header, has Wi-Fi | 2.4 GHz Wi-Fi | MQTT (JSON over IP) |
| **Radar module** | Bare PCB with a pin header, no Wi-Fi, no network to configure | Wired UART | Binary frames |

The fastest test: **if it has a row of pins and no Wi-Fi, it is a module. If it has a case and you configure Wi-Fi, it is the finished radar.**

That distinction decides which documentation applies to you:

- A **module has no MQTT**. It cannot reach a broker, because it has no way onto an IP network.
- A **finished radar has no exposed serial protocol**. It is not designed to be wired to a microcontroller.

Do not mix the two. Reading the module instructions for a finished radar (or the reverse) is where most of the confusion starts.

## Architecture A: The Finished Radar (MQTT)

### Three things you need

1. **A 2.4 GHz Wi-Fi network.** 5 GHz and 5.8 GHz cannot be used.
2. **An MQTT broker.** The radar publishes everything to it.
3. **An MQTT client.** You read the data with it.

### The one idea that matters: the broker sits in the middle

This is the part people miss. The radar does **not** send data to your phone, your laptop, or an app. It publishes to a broker. Everything else — your dashboard, your own script, any client you open — connects to **the same broker** and reads from there.

```
    radar  ──Wi-Fi──▶  router  ──▶  MQTT broker  ◀──  your MQTT client
                                    (Mosquitto /       (MQTT Explorer /
                                     Artemis)           mosquitto_sub /
                                                        MQTTX)

    The radar talks only to the broker.
    Your client talks only to the broker.
    They never talk to each other directly.
```

So when the radar is online but you see no data, the first question is always the same:

> **Are the radar and my client pointed at the same broker?**

In most cases they are not. The radar was configured with one address, and the client was opened with another — often a public test broker used during a first experiment.

### Choosing a broker

Any broker that accepts plain MQTT over TCP works. The radar is configured with a `tcp://` address, normally port 1883.

| Broker | Good for | Notes |
|---|---|---|
| **Eclipse Mosquitto** | Getting started, Raspberry Pi, home servers | A few megabytes, one command to install, the most widely documented broker in the IoT world. If you just want your first message to appear, start here. |
| **Apache Artemis** | Larger deployments, existing Java infrastructure | A full message broker that also speaks MQTT. This is what we run and what the radar has been verified against. |
| Managed cloud broker | Production without running your own server | Confirm it offers a plain TCP endpoint (port 1883), not only WebSocket |
| `broker.emqx.io` | First smoke test only | Public. Anyone can read your data. Never use it for a real installation. |

**Our recommendation:** if you already run a broker, use it. If you are starting from nothing, install Mosquitto — it is the shortest path from "radar on the desk" to "first message on screen".

The radar asks for four values, entered once through its built-in web page:

| Field | Limit | Note |
|---|---|---|
| Server address | Up to 64 characters | `tcp://host:1883` |
| ClientID | Up to 32 characters | **Must be unique per radar** |
| Username | Up to 32 characters | |
| Password | Up to 32 characters | |

Two radars sharing one ClientID will keep disconnecting each other. Give every unit its own.

### Choosing a client

You do not need any particular client. Any MQTT client works, so pick whichever you can install most easily.

| Client | Why you might use it |
|---|---|
| **MQTT Explorer** | Shows the topic tree automatically, so you can see which topics exist without typing them. The friendliest first client. |
| **mosquitto_sub** | Ships with Mosquitto, so if you installed Mosquitto as your broker you already have it. No extra download. |
| **MQTTX** | A cross-platform GUI client. |

### The topics the radar uses

The full report topic is:

```
<prefix>/<DeviceID>/sys/property/post
```

The prefix depends on the model:

| Model | Topic prefix |
|---|---|
| 60 GHz sleep radar | `/Radar60SP/` |
| 60 GHz fall radar | `/Radar60FL/` |
| 24 GHz presence radar | `/RadarHP/` |
| 24 GHz sleep radar | `/RadarSP/` |
| 24 GHz fall radar | `/RadarFL/` |

The DeviceID is the value shown on the radar's own configuration page. A healthy radar publishes messages like:

```json
{"version": "1.0", "method": "post", "params": {"someoneExists": "1"}}
```

The `method` field tells you what kind of message it is: `post` is a report from the radar, `get` queries a value, and `set` changes one. The radar answers queries on the report topic.

### Your first five minutes

1. Power the radar from a 5 V USB adapter.
2. Hold the button for **more than 10 seconds**, join the `RadarConfig-xxxxxxxxxxxx` hotspot, and open `192.168.0.1` to enter the broker address, ClientID, username, and password.
3. Install `BK_BLE5.apk`, hold the button for about **5 seconds** until the LED flashes, and send the radar your **2.4 GHz** Wi-Fi credentials over Bluetooth.
4. Subscribe from any client. With `mosquitto_sub`:
   ```
   mosquitto_sub -h <your-broker> -t "/Radar60FL/#" -v
   ```
   Or open MQTT Explorer and browse the topic tree.
5. Within seconds you should see JSON arriving.

If you can see that first message, the entire chain works. Everything after that — dashboards, automations, your own application — is built on top of a connection you have already proven. Full click-by-click steps are in the <a href="{{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}">setup guide</a>.

## Architecture B: The Radar Module (UART)

### What you need

1. **A USB-to-TTL serial adapter** — FTDI, CH340, or CP2102. Any of them works.
2. **Any serial terminal program.** There is nothing special to install.
3. **Four connections:** VCC, GND, TX, RX.

### The voltage warning

The module's UART is **3.3 V TTL logic level**.

Do not connect it directly to an RS-232 port, and do not feed it 5 V logic. Either can damage the module. If your adapter has a voltage jumper, set it to 3.3 V before wiring.

### Settings

| Setting | Value |
|---|---|
| Baud rate | 115200 |
| Data bits | 8 |
| Stop bits | 1 |
| Parity | None |
| Flow control | None |

That is the entire configuration. Any terminal that can be set to **115200 / 8N1** will talk to the module. There is no app, no account, and no driver beyond the one your USB-TTL adapter already needs.

```
    radar module            USB-TTL adapter          computer
    VCC GND TX RX  ──────▶   FTDI / CH340    ──USB──▶  any serial
                             / CP2102                   terminal
                           (3.3 V TTL, 115200 bps, 8N1)
```

### The frame format

```
Header      Control  Command  Length   Data     Checksum  Tail
0x53 0x59   1 Byte   1 Byte   2 Bytes  n Bytes  1 Byte    0x54 0x43
```

Every frame starts with `53 59` and ends with `54 43`. The checksum is the low 8 bits of the sum of the header, control byte, command byte, length bytes, and data.

You send a command frame, and the module answers with a report frame. The control and command bytes differ per product — the complete tables are in each product's guide: <a href="{{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}">sleep radar</a>, <a href="{{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}">presence radar</a>, and <a href="{{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}">fall radar</a>.

### Why MQTT is not available here

The module has no Wi-Fi radio and no network stack. It cannot reach a broker because it has no way onto an IP network at all.

If you need MQTT, you have two options: use the finished radar, or put your own microcontroller between them — read the UART frames and publish them yourself. That is exactly what the finished product does internally.

## The Two Are Not Interchangeable

| Question | Finished radar | Radar module |
|---|---|---|
| Joins Wi-Fi | Yes | No |
| Speaks MQTT | Yes | No |
| Exposes UART | No | Yes |
| Needs a broker | Yes | No |
| Needs a USB-TTL adapter | No | Yes |
| Configured with | Web page + Android app | Serial commands |
| Data format | JSON | Binary frames |
| Power | 5 V USB | 3.3 V (logic) |
| Typical use | Finished installation, dashboards, automations | Built into your own product |

## Troubleshooting

| Symptom | Likely cause | What to check |
|---|---|---|
| Radar is online but the client shows nothing | Radar and client on different brokers | Compare the address in the radar's web page with the one your client connects to |
| Two radars keep disconnecting each other | Same ClientID | Give every radar a unique ClientID |
| Wi-Fi setup always fails | 5 GHz network | Use 2.4 GHz only, or split the bands in your router |
| Provisioning times out | Took longer than 90 seconds | Hold the button again and retry |
| `192.168.0.1` will not open | Phone not connected to the `RadarConfig` hotspot | Reconnect to the hotspot. It has no internet access — that is normal |
| Serial terminal shows garbage | Wrong baud rate or framing | Set 115200, 8 data bits, 1 stop bit, no parity |
| Nothing at all from the module | TX/RX crossed, or wrong voltage | Swap TX and RX. Confirm 3.3 V TTL, not 5 V or RS-232 |
| Trying to set MQTT on a module | The module has no network | Use the finished radar for MQTT |
| App cannot find `Radar_BLE` | Module not in provisioning mode | Hold the button for 5 seconds until the LED flashes |

## Where to Go Next

- <a href="{{ site.baseurl }}{% post_url 2026-10-17-radar-setup-guide %}">How to Set Up Your mmWave Radar</a> — the click-by-click MQTT and Wi-Fi configuration
- <a href="{{ site.baseurl }}{% post_url 2026-10-17-sleep-radar-guide %}">Sleep radar guide</a> · <a href="{{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}">Presence radar guide</a> · <a href="{{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}">Fall radar guide</a>
- <a href="{{ site.baseurl }}{% post_url 2026-10-17-how-mmwave-radar-works %}">How mmWave Radar Works</a> — the sensing principle behind all of them

## Summary

Two architectures, two protocols.

The **finished radar** joins your 2.4 GHz Wi-Fi and publishes JSON to an MQTT broker. You read it with any MQTT client, and it never talks to your devices directly — the broker is always in the middle. If you see no data, check first that both ends use the same broker.

The **radar module** has no network at all. It speaks binary frames over a 3.3 V TTL UART at 115200 bps, 8N1, readable with any serial terminal and a USB-TTL adapter.

Identify which one you have first. Everything else in the documentation follows from that.
