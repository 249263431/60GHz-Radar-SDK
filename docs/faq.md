---
layout: default
title: FAQ
---

# FAQ

**Who is this for?**

- **I bought a finished radar** → Setup & network, Voice, Power & mounting
- **I want a dashboard or automations** → Data & integration
- **I'm an OEM embedding the sensor** → OEM & certification, Data & integration

## Buying & selection

### Which one should I buy?
See [Which 60GHz Radar Should I Buy?]({{ "/compare.html" | relative_url }}).

### What is the difference between a finished radar and a bare module?
The **finished unit** has a housing, a 2.4 GHz Wi-Fi radio and an MQTT stack; you configure it and read it from a broker. The **bare module** is the sensing core only: UART, no network, built into your own product. Compare: [Two Architectures, Two Protocols: MQTT vs UART]({{ "/fundamentals/2026/10/17/radar-architecture-mqtt-vs-uart.html" | relative_url }}).

### Is this a medical device?
**No.** These are sensing devices, not medical devices. They do not diagnose, treat, or prevent any condition, and they are not a substitute for human care, a call system, or any emergency service. Treat every reading as a safety and wellness indicator.

### Can it count how many people are in the room?
**No.** The presence radar reports occupancy and motion state, not a person count. Several people inside one beam are ambiguous — design around presence, not counting.

### Does it measure REM sleep?
**No.** The sleep radar reports awake / light sleep / deep sleep / out-of-bed, the time spent in each, and a score — all derived from movement and breathing statistics. True REM is identified by eye movement and muscle tone, which no non-contact sensor measures. See [Building a Sleep Product: rings, watches, radar modules and infrared]({{ "/guide/2026/10/19/sleep-sensing-ring-watch-radar-module-infrared.html" | relative_url }}).

## Setup & network

### Can the radar see through walls?
Millimeter wave penetrates non-metallic materials like curtains and thin wooden doors to some degree, but **concrete walls and metal cause severe attenuation**. Mount the radar facing the monitored area; don't place it behind a load-bearing wall.

### Must it use 2.4GHz Wi-Fi?
Yes. Provisioning uses **2.4GHz Wi-Fi** only, not 5GHz. Make sure your router has the 2.4G band enabled (many dual-band routers merge bands by default and need to be split in settings).

### How do I connect the radar to Wi-Fi (provisioning)?

Two short setups, both done once and stored in the module:

1. **Set the MQTT broker** — press the button for **10 s**, join the `RadarConfig` Wi-Fi hotspot, open `192.168.0.1`, and enter your broker address, port, ClientID, username, and password. (The Android app has no MQTT fields, so this must be done on the hotspot page.)
2. **Send the Wi-Fi credentials** — install `BK_BLE5.apk` (Android only; there is no iOS app), press the button for **5 s** until the LED flashes, then in the app grant location permission and send your **2.4 GHz** Wi-Fi SSID and password over Bluetooth.

Gotchas:
- **2.4 GHz only** — 5 GHz / 5.8 GHz cannot be used. If your router merges bands, split them or use a separate 2.4 GHz network.
- **90-second timeout** — if provisioning does not finish in time, hold the button again to retry.
- The app is **Android-only**; the WeChat mini-program used in China is not available outside China, so use the Android app.

Full walkthrough with screenshots and a troubleshooting table: [Wi-Fi Provisioning & MQTT Setup Guide]({{ "/guide/2026/10/17/radar-setup-guide.html" | relative_url }}).

### Is there an iOS app?
**No.** The configuration app is Android only (`BK_BLE5.apk`). The MQTT settings are entered on the module's own hotspot page, so no app is needed for that half of the setup — you do need an Android phone to push the Wi-Fi credentials over Bluetooth.

### Does it need the cloud? Where does data go?
No. The radar reports over a **local MQTT broker** (e.g. Mosquitto / Artemis) and by default **never leaves your home network**. Going to the cloud is optional and up to you.

### What happens if the internet or the power goes out?
The finished units talk to your **local broker over the LAN** — no internet or cloud is needed for normal operation, so a WAN outage changes nothing. A power cut stops sensing (there is no internal battery); data resumes when power returns. Put the broker and Home Assistant on a UPS if outages are frequent.

## Privacy & safety

### How is privacy? Compared to a camera?
The sensing output is point-cloud level — distance, motion, presence and state — with **no video and no image stream**, which is what makes it acceptable in bathrooms and bedrooms. The finished fall radar additionally carries a **speaker and microphone that operate on the device only**, for the local voice prompt and spoken reply; that audio is not streamed out or stored by the sensing path.

### Is this a medical device?
See the answer above: **no**, it is a sensing device, not a medical one. It supports care and awareness; it does not replace a caregiver, a call system, or emergency services.

## Data & integration

### How do I connect to Home Assistant?
See [Home Assistant Integration (Verified)]({{ "/integration/2026/10/18/home-assistant-radar-integration.html" | relative_url }}) and the per-model HA guides: [fall]({{ "/integration/2026/10/18/fall-radar-home-assistant.html" | relative_url }}), [presence]({{ "/integration/2026/10/18/presence-radar-home-assistant.html" | relative_url }}), [sleep]({{ "/integration/2026/10/18/sleep-radar-home-assistant.html" | relative_url }}), and [R60ABD1 via an ESP32 gateway]({{ "/integration/2026/10/22/r60abd1-home-assistant-mqtt.html" | relative_url }}).

### Can I use it without Home Assistant?
**Yes.** Anything that speaks MQTT works: a command-line client, Node-RED, your own backend. Home Assistant is one option, not a requirement.

### What protocol and interface does it use?

Two families ship from this store:

- **Finished 60 GHz radar** — connects over **2.4 GHz Wi-Fi** and reports a **JSON property protocol over MQTT topics**. Configure it from a web page (for MQTT) or the Android app (for Wi-Fi), then read it from any broker (Mosquitto / Artemis). See [Home Assistant Integration (Verified)]({{ "/integration/2026/10/18/home-assistant-radar-integration.html" | relative_url }}).
- **Bare module** — the sensing core only, with **UART only**, speaking a binary **frame protocol at 115200 bps, 8N1, 3.3 V TTL**. No Wi-Fi, no account; any serial terminal or USB-TTL adapter talks to it. This is the path for embedding the sensor into your own product.

Full comparison: [Two Architectures, Two Protocols: MQTT vs UART]({{ "/fundamentals/2026/10/17/radar-architecture-mqtt-vs-uart.html" | relative_url }}).

### How do I run my own MQTT broker?
Any MQTT 3.1/5 broker works. For a self-hosted Linux box, see [Run Your Own MQTT Broker for 60GHz Radar: An Apache Artemis Setup Guide]({{ "/guide/2026/10/23/apache-artemis-mqtt-broker-radar.html" | relative_url }}).

### Can I install several radars in one home?
Yes. Give every device a **unique ClientID** and its own broker account, and name the IDs by room (`fl-bathroom-01`). A worked three-radar example is in [Building an Aging-in-Place Care System with Three 60GHz Radars]({{ "/guide/2026/10/23/aging-in-place-care-system-three-radars.html" | relative_url }}).

## Accuracy & expectations

### Are the advertised detection range / angle numbers reliable?
See [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ "/fundamentals/2026/10/17/radar-spec-limits.html" | relative_url }}).

### Why does it false-alarm?
The usual sources are moving non-living objects: electric fans, the internal motor of an air conditioner, pets near the beam, swaying metal curtains, and water pipes. Keep them out of the main lobe and lower the sensitivity where the room is small. See [Fall Detection Failure Modes]({{ "/fundamentals/2026/10/17/fall-detection-failure-modes.html" | relative_url }}) and [Ghost Targets and Multipath Interference]({{ "/guide/2026/10/19/ghost-targets-multipath-interference.html" | relative_url }}).

### How much latency / how real-time?
See [Why mmWave Radar Takes Seconds to Respond]({{ "/fundamentals/2026/10/17/radar-latency-explained.html" | relative_url }}).

### How does it work? How does mmWave measure breathing / falls?
See [How mmWave Radar Works]({{ "/fundamentals/2026/10/17/how-mmwave-radar-works.html" | relative_url }}).

## Voice (finished fall radar)

### How do I choose the voice / alarm language?
The language is **specified when you place the order** (Chinese or English) and is set at the factory. If the wrong language arrives, the wake word and the two fall replies can be **re-taught on the device** — dialects and other languages included. See the [fall radar guide]({{ "/product/2026/10/17/fall-radar-guide.html" | relative_url }}).

## Power & mounting

### What are the power and mounting requirements?
See [Sleep Radar Mounting Position]({{ "/guide/2026/10/16/sleep-radar-installation-position.html" | relative_url }}), and the tables below.

### Specifications: size, power supply, and wireless

**Finished 60 GHz units** — connect over **2.4 GHz Wi-Fi + MQTT**:

| Product | Housing size | Power | Wireless | Typical mounting |
|---|---|---|---|---|
| Fall Detection Radar | 87 × 87 × 24 mm | USB Type-C: 5V/1A | 2.4 GHz Wi-Fi + MQTT | Ceiling, face down, ≤ 2.8 m |
| Presence Radar | 29.5 × 73 × 86 mm | USB Type-C: 5V/1A | 2.4 GHz Wi-Fi + MQTT | Recessed in ceiling; module ≤ 2.75 m |
| Sleep Radar | 80 × 80 × 51.8 mm | USB Type-C: 5V/1A | 2.4 GHz Wi-Fi + MQTT | Bedside: side-wall 1.8–2.2 m, or above bed head tilted 30°–45° down |

**Bare modules** — UART only, no network to configure:

| Module | Function | Interface | Power | Typical mounting |
|---|---|---|---|---|
| R24AVD1 | Presence / occupancy, 24 GHz | UART 3.3 V TTL + S1/S2 outputs | 3.3 V I/O; VCC per datasheet | Ceiling, height ≤ 2.75 m |
| R60AFD1-V | Fall detection, 60 GHz | UART 115200 8N1 + GP1/GP2 | 4.5–5.5 V (typ. 5.0 V), ≤ 100 mA avg / ≤ 300 mA peak | Ceiling, 2.2–2.8 m |
| R60ABD1 | Sleep & breathing, 60 GHz | UART 115200 8N1 + spare GPIO | 4.5–5.5 V (typ. 5.0 V), ≥ 200 mA | Above bed 0.8–1.0 m, tilt 30°–45° |

Each product guide has its own `## Specifications` table with the full parameter list (beam angle, current, detection range, etc.). For the modules, see the [60 GHz / 24 GHz module spec sheet]({{ "/guide/2026/10/21/60ghz-24ghz-radar-modules-oem-integration.html" | relative_url }}).

## OEM & certification

### Does it have CE / FCC certification?
We supply **modules**, not certified end products. Certification is done by you, and the certificate is issued in **your company's name** — because you are the manufacturer placing the product on the market (CE) and the grantee (FCC). We support you with samples, RF and antenna test data, technical file material, and technical Q&A. Full details: [Certification Support: Get CE and FCC Made Out to Your Own Name]({{ "/guide/2026/10/23/certification-support-ce-fcc-own-brand.html" | relative_url }}).

### I want to embed your sensor in my own product — where do I start?
1. Pick the module from the table above and the [spec sheet]({{ "/guide/2026/10/21/60ghz-24ghz-radar-modules-oem-integration.html" | relative_url }}).
2. Understand the protocol split: [Two Architectures, Two Protocols]({{ "/fundamentals/2026/10/17/radar-architecture-mqtt-vs-uart.html" | relative_url }}).
3. Get your first frames: [Radar Module Bring-Up Checklist]({{ "/guide/2026/10/22/radar-module-bring-up-checklist.html" | relative_url }}) and [Minimal UART Parser (SIP-S V1.0)]({{ "/guide/2026/10/22/minimal-uart-parser-sip-s.html" | relative_url }}).
4. Or skip the firmware and reuse a working gateway: [R60ABD1 + ESP32-S3 SIP-S/MQTT]({{ "/guide/2026/10/22/r60abd1-esp32-s3-sip-s-mqtt.html" | relative_url }}).

## Still have questions?
Leave a message on [our Tindie store](https://www.tindie.com/stores/fuxinsuo/){:target="_blank"} or open a GitHub Issue.
