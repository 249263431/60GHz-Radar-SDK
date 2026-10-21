---
layout: post
title: "GPIO vs UART: Get a Fall Alarm Without Writing a Parser"
date: 2026-10-21 09:30:00 +0800
categories: [guide]
tags: [mmwave, radar, gpio, uart]
---

This is the shortest possible entry point to our 60 GHz fall-detection module. You can get a **fall alarm** out of it two completely different ways, and you do not have to choose one up front — many products use both.

- **GPIO** — read a voltage level. Zero protocol, zero parsing.
- **UART** — parse binary frames. Full data, full control.

The module is **R60AFD1-V** (and the same idea applies to the 24 GHz presence module **R24AVD1**, which exposes `S1`/`S2` the same way). For the full module families, see the [module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}). The form-factor / protocol split is explained in [Two Architectures, Two Protocols]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %}).

## The one-line difference

| | GPIO | UART |
|---|---|---|
| What you get | 1 bit: *did the event happen?* (high/low) | The full data set (DP1–DP8) |
| Code required | `digitalRead()` — that's it | A small frame parser |
| Best for | Alarms, relays, PLC, "just tell me yes/no" | Dashboards, logic, tuning, queries |

## Path 1 — GPIO (no parser at all)

The module already decides "is someone there" and "did someone fall" and presents each as a 3.3 V logic level on a dedicated pin.

**R60AFD1-V pinout (interface 1)**

| Pin | Name | Level | Meaning |
|---|---|---|---|
| GP1 | Fall output | **High = fall detected**, Low = clear | Drives high the moment a fall is concluded |
| GP2 | Presence output | **High = someone present**, Low = empty | Occupied / unoccupied |

**R24AVD1** does the same with named outputs: `S1` = occupied (High/Low), `S2` = active/static (High/Low); `GP1–GP4` are redefinable.

### Wiring options

1. **Straight to an MCU.** Connect GP1 to any digital input. The module drives the pin; no pull-up needed.
2. **Drive a buzzer / relay / LED.** The pin is weak (mA-level) — **never connect a load directly**. Use a transistor (e.g. NPN + base resistor) or a MOSFET, or an opto-isolator for a PLC digital input.

### Minimal sketch (ESP32 / Arduino)

```cpp
const int GP1 = 4;   // R60AFD1-V GP1 -> this pin
const int GP2 = 5;   // R60AFD1-V GP2 -> this pin

void setup() {
  pinMode(GP1, INPUT);
  pinMode(GP2, INPUT);
  Serial.begin(115200);
}

void loop() {
  bool fell    = digitalRead(GP1) == HIGH;   // High = fall alarm
  bool present = digitalRead(GP2) == HIGH;   // High = someone present
  // Act on these flags — e.g. trigger a buzzer, set a relay, send an alert.
  delay(100);
}
```

**Level warning:** the I/O is **3.3 V TTL**. A 3.3 V MCU (ESP32, STM32) matches directly. A 5 V MCU reading a 3.3 V high usually works but has little margin — prefer a level shifter or a native 3.3 V part. **Never drive 5 V into the module's pins**; it can destroy them.

## Path 2 — UART (the full data set)

If you want more than a yes/no — body-movement amplitude, presence/static state, install-height config, alarm on/off, query/set — you read the serial frames instead.

**Electrical / serial:** 3.3 V TTL, **115200 bps, 8N1**.

**Frame format (shared across the family)**

```
Header    Ctrl  Cmd  Len    Data    Checksum  Tail
0x53 0x59 1B    1B   2B     nB      1B        0x54 0x43
```

The **checksum** is the low 8 bits of the sum of every byte from the header through the data. Example: query presence (`80 81`) becomes `53 59 80 81 00 01 0F BD 54 43`.

**Fall module data points (DP1–DP8):** occupied/unoccupied, static/active, body-movement 0–100, **fall alarm**, **static-dwell alarm**, install-height setting, fall-alarm on/off, static-dwell on/off.

### Minimal parser skeleton (Python)

```python
def find_frame(buf):
    # buf: bytearray of incoming serial bytes
    i = buf.find(b'\x53\x59')
    if i < 0 or len(buf) < i + 6:
        return None, buf
    ctrl = buf[i+2]; cmd = buf[i+3]
    length = int.from_bytes(buf[i+4:i+6], 'big')
    end = i + 6 + length + 2  # data + checksum + 2-byte tail
    if len(buf) < end:
        return None, buf
    frame = buf[i:end]
    if frame[-2:] != b'\x54\x43':
        return None, buf[i+2:]   # resync past bad header
    body = frame[2:-3]           # ctrl,cmd,len,data
    csum = frame[-3]
    if (sum(body) & 0xFF) != csum:
        return None, buf[i+2:]
    return (ctrl, cmd, frame[6:6+length]), buf[end:]
```

That is the entire job: find the header, read the length, verify the tail and checksum, extract the payload.

## Which should you use?

- **GPIO** if your product is an alarm, a relay, or feeds a PLC / "dry contact" input. It is the lowest-risk, lowest-effort path and needs no firmware on your side.
- **UART** if you need the numbers, want to tune sensitivity/install height, or build a UI / cloud feed.
- Use **both**: drive an immediate relay from GP1 (fast, local) while UART streams the rich data to your dashboard.

When you later want the data off the module and onto a broker/cloud, that is the third tier — **MQTT over Wi-Fi** on the finished radar — covered in [Two Architectures, Two Protocols]({{ site.baseurl }}{% post_url 2026-10-18-radar-architecture-mqtt-vs-uart %}).

## Caveats (read before you ship)

- **Power:** 4.5–5.5 V supply, ripple ≤ 100 mV, good grounding. Allow **≥ 30 s** after power-on before trusting output.
- **False alarms are possible.** Moving non-living objects (fans, swinging curtains, flowing water) can trip the sensor; a person lying on their side/back may be missed because the chest micro-motion isn't facing the radar. Design redundancy and fail-safe behavior.
- **Not a medical device.** Do not use as a life-safety system without redundant design; not for diagnosis or treatment. See the [module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}) disclaimer.

## Where to start

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/). Volume orders, customization, and certification support: contact us by email (store page). For fall-specific setup, see the [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}).
