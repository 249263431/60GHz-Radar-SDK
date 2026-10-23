---
layout: post
title: "Connect Your 60GHz Fall Detection Radar to Home Assistant (Verified Walkthrough)"
categories: [integration]
tags: [home assistant, mqtt, integration, fall radar, dashboard]
---

# Connect Your 60GHz Fall Detection Radar to Home Assistant (Verified Walkthrough)

If you bought our 60GHz fall detection radar and configured it for MQTT, the next step is to get that fall alarm and presence data into a dashboard — and, more importantly, into an alert you can actually rely on. Home Assistant (HA) is the obvious choice: it speaks MQTT natively, runs locally (your data never leaves your home), and is free and open source.

This is the radar we verified end to end. Every step below was run on a real Windows 11 machine with WSL2 and Home Assistant Core **2025.1.4**, against a live fall radar talking to an Apache Artemis broker. The two sensors in this guide were confirmed returning real values (`fall_status = 0`, `presence = 1`). No "should work in theory" — it is proven.

## The big picture

```
60GHz Fall Radar  --MQTT-->  Broker (Artemis / Mosquitto)  --MQTT-->  Home Assistant
```

Three actors: the radar publishes JSON over MQTT, the broker is the middleman (the radar does *not* talk to HA directly), and HA subscribes and turns the JSON into entities. If you have not set up the broker + radar MQTT side yet, do that first (see our MQTT architecture article and the fall radar setup guide).

## Before you start

You need:

1. A broker reachable on your network (e.g. `192.168.1.2:1883`) with a username and password.
2. Your fall radar configured for MQTT (broker IP, port 1883, credentials) — covered in the setup guide.
3. The device's **DeviceID**. On our radars the DeviceID *is* the device MAC address, which is also the **default** MQTT ClientID. It is printed on the device label and shown on the configuration page; you can change the ClientID, but keep it unique per device — two devices sharing one ClientID keep kicking each other off.

The fall radar uses the prefix `Radar60FL`:

- Report (radar → you): `/Radar60FL/{DeviceID}/sys/property/post`
- Command (you → radar): `/Radar60FL/{DeviceID}/sys/property/set`

## What is Home Assistant, briefly

HA is an open-source home automation hub. You run it on a box at home (Raspberry Pi, mini-PC, or even a Windows laptop via WSL2). It pulls data from hundreds of device types and shows everything in one dashboard. Because it is local-first, nothing is sent to the cloud.

Three ways to install it:

- **HA OS** — a dedicated image for Raspberry Pi / x86 mini-PCs. Easiest for pure users.
- **Docker** — one container, but you need to be comfortable with Docker.
- **HA Core** — run the Python app directly. This is what we verified, and it is the lightest path on Windows with **no Docker**. This guide uses HA Core on WSL2.

## Install HA Core on WSL2 (verified)

WSL2 (Windows Subsystem for Linux) lets you run a real Linux on Windows — and HA Core is happiest on Linux.

If WSL2 is not enabled yet, in an **administrator** PowerShell:

```powershell
wsl --install
```

Reboot when prompted. Ubuntu will set itself up and ask you for a Linux username and password.

Then, inside the Ubuntu terminal:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-venv python3-dev build-essential libssl-dev libffi-dev libjpeg-dev zlib1g-dev pkg-config

python3 -m venv ~/hassenv
source ~/hassenv/bin/activate
pip install --upgrade pip
pip install homeassistant
```

The last step downloads a lot of packages — give it a few minutes. When it finishes, start HA:

```bash
hass
```

First launch initializes the config and database and takes several minutes. You'll know it is up when port 8123 responds. Open **http://localhost:8123** in your Windows browser and create your admin account.

> We tested on Ubuntu 24.04 with Python 3.12 and Home Assistant Core **2025.1.4**. HA requires Python 3.12+, so use a current Ubuntu.

To keep HA running after you close the terminal, start it detached:

```bash
setsid nohup hass > ~/hass.log 2>&1 < /dev/null &
```

## Connect Home Assistant to your broker

1. **Settings → Devices & Services → Add Integration**, search **MQTT**.
2. Enter your broker details:
   - Broker: `192.168.1.2` (your broker's IP)
   - Port: `1883`
   - Username / Password: your broker credentials
3. Save.

You can confirm the link from the broker side: Artemis' console will now show a new connection from Home Assistant.

## Configure the fall-radar sensors (the important part)

The fall radar reports `fallStatus` (0 = no fall, 1 = fall — the primary alarm), `someoneExists` (presence), `sceneMode` (1 living room, 2 bedroom, 3 bathroom), `motionStatus`, `fallPosition`, `residentStatus`, and more. We declare the ones we want as MQTT sensors in `configuration.yaml` (in your HA config folder, `~/.homeassistant/configuration.yaml` on Linux).

{% raw %}
```yaml
mqtt:
  sensor:
    - name: "Bathroom Fall Status"
      unique_id: "radar60fl_fallstatus"
      state_topic: "/Radar60FL/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.fallStatus is defined %}
          {{ value_json.params.fallStatus }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Bathroom Presence"
      unique_id: "radar60fl_presence"
      state_topic: "/Radar60FL/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.someoneExists is defined %}
          {{ value_json.params.someoneExists }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Bathroom Scene Mode"
      unique_id: "radar60fl_scene"
      state_topic: "/Radar60FL/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.sceneMode is defined %}
          {{ value_json.params.sceneMode }}
        {% else %}
          {{ this.state }}
        {% endif %}
```
{% endraw %}

Replace `{DeviceID}` with the number printed on your device.

**Why the `if defined … else this.state` pattern is mandatory here.** The radar does not send every field in every message. Sometimes it publishes `{"params":{"someoneExists":"1"}}`, sometimes `{"params":{"fallStatus":"1"}}`, and often just a couple of fields. If you wrote `{% raw %}{{ value_json.params.fallStatus }}{% endraw %}` directly, every message that lacks `fallStatus` would blank the sensor out. The template keeps the last known value when the field is absent — the standard, stable way to handle sparse MQTT payloads.

After editing, reload without a full restart:

- **Developer Tools → YAML → MQTT: Reload** (or call the `mqtt.reload` service).

## See your fall data in the UI

Four ways, fastest first:

1. **Developer Tools → States.** Type `bathroom` in the entity filter. You'll see the three sensors above with live values and last-updated time. Walk into the room and watch `Presence` flip.
2. **The default Overview dashboard.** New sensor entities auto-appear as cards.
3. **Raw MQTT.** Settings → Devices & Services → MQTT → Configure → Listen to a topic → enter `/Radar60FL/{DeviceID}/#` → Start Listening. Great for discovering every field your model reports.
4. **A history graph.** Edit the Overview dashboard, add a History card on `sensor.bathroom_presence`, and you get an occupancy square-wave over the day — useful for showing when a room was used.

## Pitfalls we actually hit (so you don't have to)

- **Don't use the web UI "Restart Home Assistant" button** when running HA Core from the command line. There is no supervisor to bring it back, so it stops and stays stopped. Use the `setsid nohup hass … &` command above instead. (We learned this the hard way.)
- **Two harmless log errors on WSL2:** `aiodns … getaddrinfo()` and `libpcap is not available`. Both are environment quirks (DNS resolver / DHCP discovery) and do not affect the web UI or MQTT. Ignore them.
- **`fall_status` shows `unknown` at first — that is normal.** `fallStatus` is only reported on a fall *event*, not continuously. Query it once with a `get` command — publish `{"version":"1.0","method":"get","params":{"fallStatus":""}}` to the `/set` topic — and it fills in with `0`.
- **HA will not auto-start after a PC reboot.** Re-run the `setsid nohup hass … &` command. For a permanent setup, HA OS or Docker (with restart policies) is the better long-term home.

## Wrapping up

That is the chain we proved end to end — **radar → broker → Home Assistant → dashboard** — with real `fall_status` and `presence` values flowing. From here you can push a phone notification when `fall_status` goes to `1`, or log daily room occupancy, all locally.

For the full list of reportable fall-radar fields and the install notes, see the fall radar guide and our MQTT architecture article.
