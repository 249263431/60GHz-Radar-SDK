---
layout: post
title: "How to Set Up Your mmWave Radar: MQTT and Wi-Fi Configuration"
date: 2026-10-17 08:00:00 +0800
categories: [guide]
tags: [provisioning, mqtt, wifi, setup, 2.4GHz, android]
---

<!-- filename: 2026-10-17-radar-setup-guide.md -->

After receiving the radar, two things must be configured before it can send data: the MQTT connection and the Wi-Fi connection. The MQTT settings tell the radar where to send its data, and the Wi-Fi settings tell it how to reach your network. Both are configured once and stored inside the module.

This guide covers the complete setup for users outside China: preparing the MQTT environment, configuring the radar through its built-in web page, and connecting it to Wi-Fi with the Android configuration app. No serial cable and no WeChat account are required.

## What You Need Before You Start

| Item | Requirement | Notes |
|---|---|---|
| Android phone | Android 5.0 or later | The configuration app is Android-only; there is no iOS version |
| Wi-Fi network | **2.4 GHz** band | 5 GHz / 5.8 GHz cannot be used |
| Wi-Fi password | Known | Needed in Step 4 |
| MQTT broker | Address, port, ClientID, username, password | See Step 1 |
| Configuration app | `BK_BLE5.apk` (2.35 MB) | Download link in Step 3 |
| Power | 5 V USB adapter | The module boots automatically when powered |

Keep your phone's Wi-Fi switched on during the whole setup. You will connect it to the radar's own hotspot in Step 2 and back to your home network afterwards.

## Step 1: Prepare Your MQTT Environment

The radar does not send data to a cloud service by default. It publishes everything to the MQTT broker you specify, so the broker has to be ready **before** the radar is configured.

### What the Radar Needs

| Field | Example | Limit |
|---|---|---|
| Server address | `tcp://broker.emqx.io:1883` | Up to 64 characters |
| ClientID | `radar-livingroom-01` | Up to 32 characters |
| Username | `radar` | Up to 32 characters |
| Password | your password | Up to 32 characters |

Three points to check:

- **Use a plain TCP endpoint.** The module is configured with a `tcp://` address, so choose a broker endpoint that accepts plain TCP, normally port 1883.
- **Use a unique ClientID for every radar.** Two devices sharing one ClientID will keep disconnecting each other.
- **Open the port.** If the broker runs on a server outside your local network, make sure the port is reachable from the network the radar joins.

### Broker Options

| Option | Suitable for | Notes |
|---|---|---|
| Self-hosted (EMQX, Mosquitto) | Production and long-term use | Full control, needs a server or a home server |
| Managed cloud broker | Production without running a server | Check that a plain TCP endpoint is offered |
| Public test broker `tcp://broker.emqx.io:1883` | First test only | Anyone can read the data; never use it for real deployment |

Write the four values down before moving on. You will type them into the radar in the next step.

## Step 2: Configure MQTT on the Radar

The radar has a built-in web configuration page. This is the only place where the MQTT settings are entered — the Android app cannot do it.

1. Press and hold the button on the module for **more than 10 seconds**.
2. On your phone, open the Wi-Fi settings and connect to the hotspot named `RadarConfig-xxxxxxxxxxxx` (the last characters identify the device).
3. Open a browser and go to `192.168.0.1`. The configuration page opens and shows the device ID plus empty MQTT fields.
4. Fill in the server address, ClientID, username, and password from Step 1.
5. Tap **Modify** to save. The page refreshes and shows the values you entered.
6. Tap **Setup complete, exit**. The module reboots and leaves configuration mode.

The settings are stored inside the module, so this step is done once. If you change the broker later, repeat Step 2.

**Note:** while the phone is connected to the `RadarConfig` hotspot it has no internet access. That is normal — the page is served by the module itself.

## Step 3: Install the Configuration App

The radar is provisioned over Bluetooth. The WeChat mini-program used in China is not available outside China, so use the Android app instead.

1. Download `BK_BLE5.apk` (2.35 MB) from GitHub:
   `https://github.com/249263431/Android_BK_BLE_WIFI_NEW_2024031901/blob/main/BK_BLE5.apk`
2. If your phone asks, allow installing apps from this source (Settings → Apps → Special access → Install unknown apps).
3. Open the app. The main screen is titled **Network Test** and shows three fields: Bluetooth name, Wi-Fi name, and Wi-Fi password.

The app configures Wi-Fi only. It contains no MQTT settings, which is why Step 2 comes first.

## Step 4: Connect the Radar to Wi-Fi

1. Press and hold the button for about **5 seconds**, until the LED turns off, then release. The LED starts flashing: the module is now in provisioning mode.
2. Make sure the phone's Wi-Fi is on, and grant the location permission if the phone asks for it. Android requires it to scan for Bluetooth and Wi-Fi devices.
3. In the app, tap **Choose bluetooth** and select `Radar_BLE` from the list. If the list is empty, wait 10–20 seconds for the scan to finish.
4. Tap **Choose SSID** and select your **2.4 GHz** network. Type the Wi-Fi password, and use **Show Password** to check what you typed.
5. Tap **Start confguring network** and wait. The whole process takes a few seconds.
6. When the app reports success, the radar is connected to your router and to your broker.

Two limits to remember:

- **2.4 GHz only.** A 5 GHz / 5.8 GHz network cannot be used. If your router broadcasts both bands under one name, split the bands in the router settings or use a separate 2.4 GHz network.
- **90 seconds.** If provisioning does not finish within 90 seconds it times out. Press and hold the button again to restart.

**Note:** every step in this section must be done while the LED is flashing. If the LED is not flashing, the attempt has failed and you need to start again.

## Step 5: Verify the Connection

Check three things, in order:

1. **Wi-Fi** — the app reports a successful configuration.
2. **Broker** — subscribe with any MQTT client to the radar's report topic. The topic prefix depends on the model:

| Model | Topic prefix |
|---|---|
| 60 GHz sleep radar | `/Radar60SP/` |
| 60 GHz fall radar | `/Radar60FL/` |
| 24 GHz presence radar | `/RadarHP/` |
| 24 GHz sleep radar | `/RadarSP/` |
| 24 GHz fall radar | `/RadarFL/` |

The full report topic is `<prefix>/<DeviceID>/sys/property/post`. The DeviceID is the value shown on the configuration page in Step 2. A healthy radar publishes JSON messages such as:

```json
{"version": "1.0", "method": "post", "params": {"someoneExists": "1"}}
```

3. **Data** — the messages change when the state changes, for example when someone enters or leaves the room.

## If Something Goes Wrong

| Issue | Possible Cause | Solution |
|---|---|---|
| No `Radar_BLE` in the app list | Module not in provisioning mode | Press and hold the button for 5 s until the LED flashes |
| | Bluetooth or location permission denied | Grant both permissions and scan again |
| Provisioning times out | Wrong Wi-Fi band | Use a 2.4 GHz network |
| | Wrong Wi-Fi password | Re-enter the password and check it with Show Password |
| Cannot open `192.168.0.1` | Phone not connected to the `RadarConfig` hotspot | Reconnect to the hotspot and reload the page |
| Radar online but no data | Broker address, port, or credentials wrong | Repeat Step 2 and check every field |
| | ClientID already used by another device | Give every radar a unique ClientID |
| | Firewall blocking the port | Open the port or use a reachable broker |
| Bluetooth scan fails on Android 12 or later | Nearby devices permission not granted | Allow the "Nearby devices" permission for the app |

## Common Mistakes

| Mistake | Result | Solution |
|---|---|---|
| Configuring Wi-Fi before MQTT | The app has no MQTT fields, so the broker is never set | Do Step 2 first |
| Using a 5 GHz network | Provisioning always fails | Use 2.4 GHz |
| Using the public test broker for real use | Data is visible to everyone | Use your own broker |
| Same ClientID on several radars | Devices disconnect each other | Use a unique ClientID per device |
| Assuming the app configures everything | The broker is never configured | Complete both steps |

## Summary

Setting up the radar takes two configurations and nothing else. Prepare the MQTT broker first and write down the address, port, ClientID, username, and password. Then press the button for 10 seconds, join the `RadarConfig` hotspot, and enter those values at `192.168.0.1`. Finally, install `BK_BLE5.apk`, press the button for 5 seconds until the LED flashes, and send the radar your 2.4 GHz Wi-Fi credentials over Bluetooth.

Use 2.4 GHz Wi-Fi, a unique ClientID for every radar, and a broker you control. Once both steps are done, the radar publishes its data to your broker and the setup is finished.
