---
layout: default
title: FAQ
---

# FAQ

## Can the radar see through walls?
Millimeter wave penetrates non-metallic materials like curtains and thin wooden doors to some degree, but **concrete walls and metal cause severe attenuation**. Mount the radar facing the monitored area; don't place it behind a load-bearing wall.

## Does it need the cloud? Where does data go?
No. The radar reports over a **local MQTT broker** (e.g. Mosquitto / Artemis) and by default **never leaves your home network**. Going to the cloud is optional and up to you.

## How is privacy? Compared to a camera?
The radar only outputs point-cloud-level info like distance, speed, and presence / fall state — **no video, no audio**, far more privacy-friendly than a camera. Great for bathrooms and bedrooms.

## Must it use 2.4GHz Wi-Fi?
Yes. Provisioning uses **2.4GHz Wi-Fi** only, not 5GHz. Make sure your router has the 2.4G band enabled (many dual-band routers merge bands by default and need to be split in settings).

## How do I connect the radar to Wi-Fi (provisioning)?

Two short setups, both done once and stored in the module:

1. **Set the MQTT broker** — press the button for **10 s**, join the `RadarConfig` Wi-Fi hotspot, open `192.168.0.1`, and enter your broker address, port, ClientID, username, and password. (The Android app has no MQTT fields, so this must be done on the hotspot page.)
2. **Send the Wi-Fi credentials** — install `BK_BLE5.apk` (Android only; there is no iOS app), press the button for **5 s** until the LED flashes, then in the app grant location permission and send your **2.4 GHz** Wi-Fi SSID and password over Bluetooth.

Gotchas:
- **2.4 GHz only** — 5 GHz / 5.8 GHz cannot be used. If your router merges bands, split them or use a separate 2.4 GHz network.
- **90-second timeout** — if provisioning does not finish in time, hold the button again to retry.
- The app is **Android-only**; the WeChat mini-program used in China is not available outside China, so use the Android app.

Full walkthrough with screenshots and a troubleshooting table: [Wi-Fi Provisioning & MQTT Setup Guide]({{ "/guide/2026/10/17/radar-setup-guide.html" | relative_url }}).

## How do I choose the voice / alarm language?
The firmware supports Chinese / English voice prompts; switch in the companion app or a config item. See each product guide.

## How do I connect to Home Assistant?
See [Home Assistant Integration (Verified)]({{ "/2026/10/18/home-assistant-radar-integration.html" | relative_url }}) and the per-model HA guides.

## What are the power and mounting requirements?
See [Sleep Radar Mounting Position]({{ "/installation/2026/10/16/sleep-radar-installation-position.html" | relative_url }}).

## How much latency / how real-time?
See [Why mmWave Radar Takes Seconds to Respond]({{ "/fundamentals/2026/10/17/radar-latency-explained.html" | relative_url }}).

## Are the advertised detection range / angle numbers reliable?
See [Three mmWave Radar Specs That Are Almost Always Exaggerated]({{ "/fundamentals/2026/10/17/radar-spec-limits.html" | relative_url }}).

## How does it work? How does mmWave measure breathing / falls?
See [How mmWave Radar Works]({{ "/fundamentals/2026/10/17/how-mmwave-radar-works.html" | relative_url }}).

## Which one should I buy?
See [Which 60GHz Radar Should I Buy?]({{ "/compare.html" | relative_url }}).

## Specifications: size, power supply, and wireless

All three finished 60 GHz units join your network over **2.4 GHz Wi-Fi + MQTT**. The bare 24 GHz modules use **UART only** (no Wi-Fi, no network to configure).

| Product | Housing size | Power | Wireless | Typical mounting |
|---|---|---|---|---|
| Fall Detection Radar | 87 × 87 × 24 mm | 5 V DC | 2.4 GHz Wi-Fi + MQTT | Ceiling, face down, ≤ 2.8 m |
| Presence Radar | 29.5 × 73 × 86 mm | 5 V over USB | 2.4 GHz Wi-Fi + MQTT | Recessed in ceiling; module ≤ 2.75 m |
| Sleep Radar | 80 × 80 × 51.8 mm | 4.5–5.5 V (≈5 V) | 2.4 GHz Wi-Fi + MQTT | Bedside: side-wall 1.8–2.2 m, or above bed head tilted 30°–45° down |

Each product guide has its own `## Specifications` table with the full parameter list (beam angle, current, detection range, etc.). Mounting detail: [Sleep Radar Mounting Position]({{ "/installation/2026/10/16/sleep-radar-installation-position.html" | relative_url }}).

## What protocol and interface does it use?

Two families ship from this store:

- **Finished 60 GHz radar** — connects over **2.4 GHz Wi-Fi** and reports a **JSON property protocol over MQTT topics**. Configure it from a web page (for MQTT) or the Android app (for Wi-Fi), then read it from any broker (Mosquitto / Artemis). See [Home Assistant Integration (Verified)]({{ "/2026/10/18/home-assistant-radar-integration.html" | relative_url }}).
- **24 GHz radar module** — a bare sensing core with **UART only**, speaking a binary **frame protocol at 115200 bps, 8N1, 3.3 V TTL**. No Wi-Fi, no account; any serial terminal or USB-TTL adapter talks to it. This is the path for embedding the sensor into your own product.

Full comparison: [Two Architectures, Two Protocols: MQTT vs UART]({{ "/guide/2026/10/18/radar-architecture-mqtt-vs-uart.html" | relative_url }}).

## Still have questions?
Leave a message on [our Tindie store](https://www.tindie.com/stores/fuxinsuo/){:target="_blank"} or open a GitHub Issue.
