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

## Still have questions?
Leave a message on [our Tindie store](https://www.tindie.com/stores/fuxinsuo/){:target="_blank"} or open a GitHub Issue.
