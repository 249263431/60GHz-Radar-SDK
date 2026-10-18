# Connect Your 60GHz Sleep Radar to Home Assistant (Verified Walkthrough)

If you bought our 60GHz sleep monitoring radar and configured it for MQTT, the next step is to get that data — breathing rate, heart rate, sleep stage, in-bed status — into a dashboard you can actually watch. Home Assistant (HA) is the obvious choice: it speaks MQTT natively, runs locally (your data never leaves your home), and is free and open source.

This guide uses the same, verified method as our fall-radar HA walkthrough. Every step below was run on a real Windows 11 machine with WSL2 and Home Assistant Core **2025.1.4**, against a live radar talking to an Apache Artemis broker. No "should work in theory" — it is the same MQTT integration we already proved end to end.

## The big picture

```
60GHz Sleep Radar  --MQTT-->  Broker (Artemis / Mosquitto)  --MQTT-->  Home Assistant
```

Three actors, same as the fall radar: the radar publishes JSON over MQTT, the broker is the middleman, and HA subscribes and turns the JSON into entities. If you have not set up the broker + radar MQTT side yet, do that first (see our MQTT architecture article and the sleep radar setup guide).

## Before you start

You need:

1. A broker reachable on your network (e.g. `192.168.1.2:1883`) with a username and password.
2. Your sleep radar configured for MQTT (broker IP, port 1883, credentials) — covered in the setup guide.
3. The device's **DeviceID**. On our radars the DeviceID *is* the device MAC address, which is also the MQTT ClientID, and it is printed on the back of the device. No manual registration needed.

The sleep radar uses the prefix `Radar60SP`:

- Report (radar → you): `/Radar60SP/{DeviceID}/sys/property/post`
- Command (you → radar): `/Radar60SP/{DeviceID}/sys/property/set`

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

## Configure the sleep-radar sensors (the important part)

The sleep radar reports a rich set of fields: `getIntoBed` (in/out of bed), `sleepStatus` (stage), `heartRateValue`, `breathValue`, `sleepScore`, `sleepComprehensiveStatus`, and more. We declare the ones we want as MQTT sensors in `configuration.yaml` (in your HA config folder, `~/.homeassistant/configuration.yaml` on Linux).

{% raw %}
```yaml
mqtt:
  sensor:
    - name: "Bed In-Bed Status"
      unique_id: "radar60sp_inbed"
      state_topic: "/Radar60SP/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.getIntoBed is defined %}
          {{ value_json.params.getIntoBed }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Bed Heart Rate"
      unique_id: "radar60sp_hr"
      state_topic: "/Radar60SP/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.heartRateValue is defined %}
          {{ value_json.params.heartRateValue }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Bed Breathing Rate"
      unique_id: "radar60sp_br"
      state_topic: "/Radar60SP/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.breathValue is defined %}
          {{ value_json.params.breathValue }}
        {% else %}
          {{ this.state }}
        {% endif %}

    - name: "Bed Sleep Score"
      unique_id: "radar60sp_score"
      state_topic: "/Radar60SP/{DeviceID}/sys/property/post"
      value_template: >
        {% if value_json.params.sleepScore is defined %}
          {{ value_json.params.sleepScore }}
        {% else %}
          {{ this.state }}
        {% endif %}
```
{% endraw %}

Replace `{DeviceID}` with the number printed on your device.

**Why the `if defined … else this.state` pattern is mandatory here.** The sleep radar does not send every field in every message. Heart rate and breathing arrive every 3 s, the sleep stage every 10 minutes, the score only when the session ends. If you wrote `{% raw %}{{ value_json.params.heartRateValue }}{% endraw %}` directly, every message without that field would blank the sensor. The template keeps the last known value when the field is absent — the standard, stable way to handle sparse MQTT payloads.

After editing, reload without a full restart:

- **Developer Tools → YAML → MQTT: Reload** (or call the `mqtt.reload` service).

## See your sleep data in the UI

Four ways, fastest first:

1. **Developer Tools → States.** Type `bed` in the entity filter. You'll see the four sensors above with live values and last-updated time. Lie down and watch `In-Bed Status` flip.
2. **The default Overview dashboard.** New sensor entities auto-appear as cards.
3. **Raw MQTT.** Settings → Devices & Services → MQTT → Configure → Listen to a topic → enter `/Radar60SP/{DeviceID}/#` → Start Listening. Great for discovering every field your model reports.
4. **A history graph.** Edit the Overview dashboard, add a History card on `sensor.bed_heart_rate` (and breathing), and you get an overnight curve — great for demos and for checking sleep quality.

## Pitfalls we actually hit (so you don't have to)

- **Don't use the web UI "Restart Home Assistant" button** when running HA Core from the command line. There is no supervisor to bring it back, so it stops and stays stopped. Use the `setsid nohup hass … &` command above instead. (We learned this the hard way.)
- **Two harmless log errors on WSL2:** `aiodns … getaddrinfo()` and `libpcap is not available`. Both are environment quirks (DNS resolver / DHCP discovery) and do not affect the web UI or MQTT. Ignore them.
- **`sleep_score` shows `unknown` at first — that is normal.** The score is only published when a sleep session ends (and only for sessions of 4–12 hours). It fills in after the first complete night; query it once with a `get` if you want to seed it.
- **HA will not auto-start after a PC reboot.** Re-run the `setsid nohup hass … &` command. For a permanent setup, HA OS or Docker (with restart policies) is the better long-term home.

## Wrapping up

That is the same verified chain — **radar → broker → Home Assistant → dashboard** — now feeding sleep metrics. From here you can graph heart and breathing rate across the night, alert when the bed stays empty too long, or log the nightly sleep score, all locally.

For the full list of reportable sleep fields and the install notes, see the sleep radar guide and our MQTT architecture article.
