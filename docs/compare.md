---
layout: default
title: Which 60GHz Radar Should I Buy?
---

# Which 60GHz Radar Should I Buy?

We offer three 60GHz mmWave radar models for different scenarios. Pick by use case so you don't order the wrong one.

| Radar | Primary use | Core capabilities | Typical scenes | Who it's for |
|-------|-------------|-------------------|----------------|--------------|
| **Fall Detection Radar** | Elderly / bathroom fall alerts | Fall alarm, dwell alarm, local voice call | Bathroom, bedroom, elder care / living alone | Families caring for seniors, care homes |
| **Sleep Monitoring Radar** | Sleep quality tracking | Breathing / heart rate, sleep stages, in-bed / out-of-bed | Bedside in bedroom | Anyone focused on sleep health |
| **Presence Detection Radar** | Area occupancy & presence | Presence sensing, zone division, scene modes, sensitivity tuning | Study, meeting room, hallway (lights on when someone enters) | Smart-home / energy saving |

## How to choose

- **Worried a family member may fall in the shower** → Fall Detection Radar (local voice alarm, no cloud needed)
- **Want to see how well you sleep** → Sleep Monitoring Radar
- **Want "lights on when someone's there" / occupancy automation** → Presence Detection Radar
- **Need all three** → Buy separately; all share the same MQTT / Home Assistant pipeline, and configs are interchangeable

## Detailed guides

- [Fall Detection Radar Guide]({{ "/product/2026/10/17/fall-radar-guide.html" | relative_url }})
- [Sleep Monitoring Radar Guide]({{ "/product/2026/10/17/sleep-radar-guide.html" | relative_url }})
- [Presence Detection Radar Guide]({{ "/product/2026/10/17/presence-radar-guide.html" | relative_url }})

## Common prerequisites

All three connect over **2.4GHz Wi-Fi + MQTT**. Setup and integration:

- [Wi-Fi Provisioning & MQTT Setup Guide]({{ "/guide/2026/10/17/radar-setup-guide.html" | relative_url }})
- [Home Assistant Integration (Verified)]({{ "/integration/2026/10/18/home-assistant-radar-integration.html" | relative_url }})

Data stays on your local broker by default — privacy friendly.

## Provisioning: how to connect to Wi-Fi

Two quick setups, both done once and stored in the module:

1. **Set the MQTT broker** — hold the button for **10 s**, join the `RadarConfig` hotspot, open `192.168.0.1`, and enter your broker address, port, ClientID, username, and password.
2. **Send the Wi-Fi credentials** — install `BK_BLE5.apk` (Android only), hold the button for **5 s** until the LED flashes, then grant location permission in the app and send your **2.4 GHz** Wi-Fi SSID and password over Bluetooth.

Notes: 2.4 GHz only (no 5/5.8 GHz); **90-second** timeout; the app is **Android-only**. Full walkthrough with screenshots and a troubleshooting table: [Wi-Fi Provisioning & MQTT Setup Guide]({{ "/guide/2026/10/17/radar-setup-guide.html" | relative_url }}).

## Buy

👉 [View / order on Tindie](https://www.tindie.com/stores/fuxinsuo/){:target="_blank"}
