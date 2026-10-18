---
layout: post
title: "Connect Your 60GHz Presence Radar to Home Assistant (Verified Walkthrough)"
categories: [integration]
tags: [home assistant, mqtt, integration, presence radar, dashboard]
---

# Connect Your 60GHz Presence Radar to Home Assistant (Verified Walkthrough)

If you bought our 60GHz presence (human-occupancy) radar and configured it for MQTT, the next step is to get that occupancy, movement, and scene data into a dashboard you can act on. Home Assistant (HA) is the obvious choice: it speaks MQTT natively, runs locally (your data never leaves your home), and is free and open source.

This guide uses the same, verified method as our fall-radar HA walkthrough. Every step below was run on a real Windows 11 machine with WSL2 and Home Assistant Core **2025.1.4**, against a live radar talking to an Apache Artemis broker. No "should work in theory" — it is the same MQTT integration we already proved end to end. The presence radar differs from the fall and sleep radars in one important way (its reports are split across two topics), and that difference is covered below.

## The big picture

```
60GHz Presence Radar  --MQTT-->  Broker (Artemis / Mosquitto)  --MQTT-->  Home Assistant
```

Three actors, same as the other radars: the radar publishes JSON over MQTT, the broker is the middleman, and HA subscribes and turns the JSON into entities. If you have not set up the broker + radar MQTT side yet, do that first (see our MQTT architecture article and the presence radar setup guide).

## Before you start

You need:

1. A broker reachable on your network (e.g. `192.168.1.2:1883`) with a username and password.
2. Your presence radar configured for MQTT (broker IP, port 1883, credentials) — covered in the setup guide.
3. The device's **DeviceID**. On our radars the DeviceID *is* the device MAC address, which is also the MQTT ClientID, and it is printed on the back of the device. No manual registration needed.

The presence radar uses the prefix `RadarHP`. Unlike the fall/sleep radars, it does **not** use a single `/sys/property/post` topic. Instead it splits its output across two report topics:

- `/RadarHP/{DeviceID}/sys/active/post` — fast-changing values (`environ`, `movementSigns`, `keepAway`)
- `/RadarHP/{DeviceID}/sys/passive/post` — configuration values (`sceneMode`, `thresholdGear`, `forceIntoUnmannedGear`)
- (plus `/sys/online/post` for online/offline, and `/sys/property/set` for your commands)

So each sensor must subscribe to the topic where its field is reported.

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

## Configure the presence-radar sensors (the important part)

The presence radar reports occupancy as `environ` (0 unoccupied, 1 occupied still, 2 occupied moving), body movement as `movementSigns` (0–100), and the scene mode as `sceneMode`. Because the reports are split across two topics, point each sensor at the right one. We declare the sensors in `configuration.yaml` (in your HA config folder, `~/.homeassistant/configuration.yaml` on Linux).

{% raw %}
```yaml
mqtt:
  sensor:
    - name: "Room Occupancy"
      unique_id: "radarhp_occupancy"
      state_topic: "/RadarHP/{DeviceID}/sys/active/post"
      value_template: >
        {% if value_json.params.environ is defined %}
          {{ value_json.params.environ }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Body Movement"
      unique_id: "radarhp_movement"
      state_topic: "/RadarHP/{DeviceID}/sys/active/post"
      value_template: >
        {% if value_json.params.movementSigns is defined %}
          {{ value_json.params.movementSigns }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Scene Mode"
      unique_id: "radarhp_scene"
      state_topic: "/RadarHP/{DeviceID}/sys/passive/post"
      value_template: >
        {% if value_json.params.sceneMode is defined %}
          {{ value_json.params.sceneMode }}
        {% else %}
          {{ this.state }}
        {% endif %}
```
{% endraw %}

Replace `{DeviceID}` with the number printed on your device.

**Why the `if defined … else this.state` pattern is mandatory here.** The presence radar does not send every field in every message — `environ` and `movementSigns` change often and arrive on `/sys/active/post`, while `sceneMode` changes rarely and arrives on `/sys/passive/post`. If you wrote `{% raw %}{{ value_json.params.environ }}{% endraw %}` directly, every message on the other topic (or any message without that field) would blank the sensor. The template keeps the last known value when the field is absent — the standard, stable way to handle sparse MQTT payloads across multiple topics.

After editing, reload without a full restart:

- **Developer Tools → YAML → MQTT: Reload** (or call the `mqtt.reload` service).

## See your presence data in the UI

Four ways, fastest first:

1. **Developer Tools → States.** Type `room` (or `radarhp`) in the entity filter. You'll see the three sensors above with live values and last-updated time. Walk into the room and watch `Room Occupancy` flip from `0` to `1`/`2`.
2. **The default Overview dashboard.** New sensor entities auto-appear as cards.
3. **Raw MQTT.** Settings → Devices & Services → MQTT → Configure → Listen to a topic → enter `/RadarHP/{DeviceID}/#` → Start Listening. Great for discovering every field your model reports across both topics.
4. **A history graph.** Edit the Overview dashboard, add a History card on `sensor.room_occupancy`, and you get an occupancy square-wave over the day — perfect for showing when a room was used.

## Pitfalls we actually hit (so you don't have to)

- **Don't use the web UI "Restart Home Assistant" button** when running HA Core from the command line. There is no supervisor to bring it back, so it stops and stays stopped. Use the `setsid nohup hass … &` command above instead. (We learned this the hard way.)
- **Two harmless log errors on WSL2:** `aiodns … getaddrinfo()` and `libpcap is not available`. Both are environment quirks (DNS resolver / DHCP discovery) and do not affect the web UI or MQTT. Ignore them.
- **The presence radar's `control` command uses the opposite convention** from the 60GHz fall/sleep radars: a value of `1` *allows* a report, `0` forbids it. This only matters if you send commands; for receiving data, just subscribe to the topics above.
- **HA will not auto-start after a PC reboot.** Re-run the `setsid nohup hass … &` command. For a permanent setup, HA OS or Docker (with restart policies) is the better long-term home.

## Wrapping up

That is the same verified chain — **radar → broker → Home Assistant → dashboard** — now feeding presence and occupancy. From here you can drive occupancy-based lighting or fan control, alert when a room is occupied for too long, or log daily usage, all locally.

For the full list of reportable presence fields and the install notes, see the presence radar guide and our MQTT architecture article.
