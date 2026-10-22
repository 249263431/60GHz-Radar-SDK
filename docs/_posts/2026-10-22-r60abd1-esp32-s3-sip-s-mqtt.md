---
layout: post
title: "R60ABD1 Sleep & Breathing Radar + ESP32-S3-WROOM-1: Parse SIP-S Frames and Publish to MQTT"
date: 2026-10-22 00:00:00 +0800
categories: [guide]
tags: [mmwave, radar, esp32, uart, mqtt, sip-s]
---

The R60ABD1 is a 60 GHz sleep and breathing radar module. It reports presence, breathing rate, sleep state and sleep quality over a **3.3 V TTL UART** using the **SIP-S V1.0** protocol — and it has **no Wi-Fi and no MQTT stack**. To get that data into a broker or a dashboard, you add a small gateway MCU.

This guide wires the R60ABD1 to the **ESP32-S3-WROOM-1 development board** (the native-USB board we sell) and shows how to parse the frames and, optionally, publish them to any MQTT broker. It is the bridge half of the two-architecture story in [MQTT vs UART]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %}): the module does not speak MQTT, the ESP32 does.

## What you need

- R60ABD1 module (60 GHz sleep/breathing radar, SIP-S V1.0 over UART)
- ESP32-S3-WROOM-1 dev board — [the one used here](https://www.tindie.com/products/fuxinsuo/esp32-s3-development-board-with-esp32-s3-wroom-1/) (DevKitC-1 compatible, 44 pins, native USB)
- Jumper wires
- 5 V from the board's `5V` pin (or a separate 5 V supply); shared `GND`
- Optional: an MQTT broker (Mosquitto, etc.) for the publish step

## Power (from the R60ABD1 manual)

The R60ABD1 hardware-design section (user manual v3.6, §3) states:

- Rated supply voltage **4.5–5.5 V** (typ 5 V)
- Required input current **≥ 200 mA** in normal operation
- Supply ripple **≤ 100 mV**

The board's `5V` pin can power it; if the radar's peak draw would exceed the board's 5 V budget, use a separate 5 V supply but **keep `GND` common** with the ESP32.

## ESP32-S3-WROOM-1 GPIO allocation

This board uses **native USB (GPIO19/GPIO20)** for Serial/JTAG and flashing — there is no external USB-to-serial bridge on GPIO43/44. Pinout is DevKitC-1 compatible, all 44 module pins broken out at 2.54 mm.

**Radar ↔ ESP32 wiring (locked):**

| R60ABD1 | ESP32-S3 | Note |
|---|---|---|
| `TX` (3.3 V out) | **GPIO18** (UART1 RX) | radar → ESP |
| `RX` (3.3 V in) | **GPIO17** (UART1 TX) | ESP → radar |
| `GND` | `GND` | common ground |
| `VCC` (5 V) | `5V` pin | per manual §3 |

Both UART lines are **3.3 V TTL**, so no level shifter is needed.

**Pins to avoid (so you do not lose the debugger or hit strapping):**
| GPIO | Why avoid |
|---|---|
| **19, 20** | native USB D+/D- (flash, log, JTAG) |
| 0 | strapping — boot mode |
| 3 | strapping — boot-log enable |
| 26–32 | flash / PSRAM (internal) |
| 45, 46 | internal flash-voltage strapping, not broken out |
| 43, 44 | UART0 — free at hardware level, but ESP-IDF's default console; leave it for logs |

We use **UART1 on GPIO17/18**: it is clear of every reserved function, and it leaves the native USB and the default console untouched.

## Wiring diagram

```
R60ABD1                 ESP32-S3-WROOM-1
 5V   ───────────────▶  5V
 GND  ───────────────▶  GND
 TX   ───────────────▶  GPIO18   (UART1 RX)
 RX   ───────────────▶  GPIO17   (UART1 TX)
```

## Firmware (ESP-IDF, Espressif IDE 3.1.0)

Develop in Espressif IDE with the ESP-IDF framework. Configure **UART1** at **115200 / 8N1** and bind it to GPIO17/18 with `uart_set_pin`. Then read bytes and run the SIP-S parser.

SIP-S V1.0 frame (from the R60ABD1 manual §7 / EN §4.2):

| Field | Bytes | Value |
|---|---|---|
| Header | 2 | `0x53`, `0x59` |
| Control | 1 | function class |
| Command | 1 | content tag |
| Length | 2 | data byte count (big-endian) |
| Data | n | payload |
| Checksum | 1 | low 8 bits of sum of all preceding bytes |
| Footer | 2 | `0x54`, `0x43` |

Control classes: `0x01` heartbeat, `0x02` product info, `0x03` OTA, `0x05` work state, `0x07` range info, `0x80` presence, `0x81` breathing, `0x84` sleep, `0x85` heart rate.

Minimal ESP-IDF parse + dispatch:

```c
#include "driver/uart.h"

#define RADAR_UART UART_NUM_1
#define TX_PIN     17
#define RX_PIN     18

void radar_init(void) {
    uart_config_t cfg = {
        .baud_rate = 115200,
        .data_bits = UART_DATA_8_BITS,
        .parity    = UART_PARITY_DISABLE,
        .stop_bits = UART_STOP_BITS_1,
        .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,
    };
    uart_param_config(RADAR_UART, &cfg);
    uart_set_pin(RADAR_UART, TX_PIN, RX_PIN, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE);
    uart_driver_install(RADAR_UART, 256, 256, 0, NULL, 0);
}

// Returns true on a valid frame; fills ctrl/cmd/data/dlen.
static bool parse_frame(const uint8_t *b, int n, uint8_t *ctrl, uint8_t *cmd,
                        const uint8_t **data, int *dlen) {
    for (int i = 0; i + 9 <= n; i++) {
        if (b[i] != 0x53 || b[i+1] != 0x59) continue;
        *ctrl = b[i+2]; *cmd = b[i+3];
        int len = (b[i+4] << 8) | b[i+5];
        int data_end = i + 6 + len;
        if (data_end + 3 > n) return false;
        int sum = 0; for (int k = i; k < data_end; k++) sum += b[k];
        if ((sum & 0xFF) != b[data_end]) continue;
        if (b[data_end+1] != 0x54 || b[data_end+2] != 0x43) continue;
        *data = &b[i+6]; *dlen = len;
        return true;
    }
    return false;
}
```

## Decoding the breathing and sleep frames

The R60ABD1 manual defines these report frames (manual v3.6, §7 command tables). After a valid frame, dispatch on `ctrl`/`cmd`:

| ctrl | cmd | Meaning | Data |
|---|---|---|---|
| `0x81` | `0x01` | breathing info (on change) | `01` normal, `02` high (>25/min), `03` low (<10/min), `04` none |
| `0x81` | `0x02` | breathing rate | 1 byte, 0–35 breaths/min (every 3 s) |
| `0x81` | `0x05` | breathing waveform | 5 bytes, sine, midline 128 (1 s) |
| `0x84` | `0x02` | sleep state (10 min) | `0` deep, `1` light, `2` awake, `3` off-bed/none |
| `0x84` | `0x06` | sleep quality score | 1 byte, 0–100 |
| `0x84` | `0x0C` | comprehensive sleep status | 8 bytes: presence(1/0), state(3/2/1/0), avg breath, avg heart, turns, large-motion %, small-motion %, apnea |

Example: a breathing-rate frame is `53 59 81 02 00 01 [rate] [chk] 54 43`. A sleep-state frame is `53 59 84 02 00 01 [state] [chk] 54 43`. The manual's own test vector `53 59 80 81 00 01 0F BD 54 43` (checksum `0xBD`) is a handy parser sanity check.

## Optional: publish to MQTT

The ESP32 publishes — the module never does. Using the `esp-mqtt` component, post a broker-agnostic topic such as `radar/R60ABD1/state` with JSON like `{"breath":22,"sleep":1}`. Point it at your own Mosquitto/EMQX/broker; nothing here is tied to a vendor cloud. This is exactly the "add your own microcontroller" path the architecture article describes for modules that lack networking.

## Notes and safety

- The module is **not a medical device**. Breathing/sleep values are indicators, not a diagnosis.
- Verify the 5 V budget before relying on the board's `5V` pin; use a separate supply if needed (keep `GND` common).
- Keep `GPIO19/20` clear — they are the native-USB debug path.
- Default ESP-IDF console stays on UART0; we left it alone, so logs and flashing keep working.

## Where to go next

- Two architectures explained: [MQTT vs UART]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %})
- The platform-agnostic parser: [Minimal UART parser (SIP-S V1.0)]({{ site.baseurl }}{% post_url 2026-10-22-minimal-uart-parser-sip-s %})
- First-good-frame checklist: [Bring-Up checklist]({{ site.baseurl }}{% post_url 2026-10-22-radar-module-bring-up-checklist %})
- Module specs: [60 GHz / 24 GHz module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %})
- The GPIO angle: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %})

Both the R60ABD1 module and the ESP32-S3 board are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
