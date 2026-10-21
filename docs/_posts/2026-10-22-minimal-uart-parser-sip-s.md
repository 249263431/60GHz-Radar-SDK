---
layout: post
title: "A Minimal Runnable UART Parser for the SIP-S V1.0 Radar Protocol (Python)"
date: 2026-10-22 09:30:00 +0800
categories: [guide]
tags: [mmwave, radar, uart, parser]
---

The SIP-S V1.0 serial protocol is what the 60 GHz / 24 GHz modules stream over UART at **115200 / 8N1**. This article gives a **minimal, runnable Python parser** that finds frames, validates the checksum, and prints the decoded fields — the bridge from "wires connected" (see the [Bring-Up checklist]({{ site.baseurl }}{% post_url 2026-10-22-radar-module-bring-up-checklist %})) to "data in your code".

## Frame structure (SIP-S V1.0)

From the SIP-S V1.0 definition in the module user manuals (e.g. R60ABD1 user manual v3.6 §8.1):

| Field | Bytes | Value |
|---|---|---|
| Header 帧头 | 2 | `0x53`, `0x59` |
| Control 控制字 | 1 | function class |
| Command 命令字 | 1 | content tag |
| Length 长度标识 | 2 | number of data bytes (big-endian) |
| Data 数据 | n | payload |
| Checksum 校验码 | 1 | sum(header+control+command+length+data), low 8 bits |
| Footer 帧尾 | 2 | `0x54`, `0x43` |

Serial: 115200 bps, 8 data, 1 stop, no parity, no flow control; **3.3 V TTL**; module TX → MCU/adapter RX.

Control-byte classes (from the same manual): `0x01` heartbeat, `0x02` product info, `0x03` OTA, `0x05` work state, `0x07` detection-range info, `0x80` presence, `0x81` breathing, `0x84` sleep, `0x85` heart rate. **Semantics are module-specific** — the fall module defines its own command set; the parser below is protocol-generic.

## Minimal Python parser

```python
import serial

HEADER = b'\x53\x59'
FOOTER = b'\x54\x43'

def parse_frames(buf):
    """Yield (control, command, data) for each valid frame in buf."""
    i = 0
    while True:
        head = buf.find(HEADER, i)
        if head == -1:
            return
        if len(buf) - head < 9:   # header+ctrl+cmd+len+chk+footer minimum
            return
        ctrl = buf[head + 2]
        cmd = buf[head + 3]
        length = (buf[head + 4] << 8) | buf[head + 5]   # big-endian
        data_start = head + 6
        data_end = data_start + length
        frame_end = data_end + 1 + 2   # checksum + footer
        if len(buf) < frame_end:
            return
        data = buf[data_start:data_end]
        checksum = buf[data_end]
        footer = buf[data_end + 1:frame_end]
        expected = sum(buf[head:data_end]) & 0xFF
        if checksum == expected and footer == FOOTER:
            yield (ctrl, cmd, data)
        i = head + 2

ser = serial.Serial('/dev/ttyUSB0', 115200, timeout=1)  # Windows: 'COM3'
buf = b''
while True:
    chunk = ser.read(64)
    if not chunk:
        continue
    buf += chunk
    for ctrl, cmd, data in parse_frames(buf):
        print(f"ctrl=0x{ctrl:02X} cmd=0x{cmd:02X} len={len(data)} data={data.hex()}")
    buf = buf[-256:]   # keep only the tail to bound memory
```

## Test it without hardware

The manual (§4.5) gives a sample frame `53 59 80 81 00 01 0F` with checksum computed as `0x53+0x59+0x80+0x81+0x00+0x01+0x0F = 0x01BD`, so the checksum byte is **0xBD**. The full frame is `53 59 80 81 00 01 0F BD 54 43` (10 bytes). Feed it straight to the parser:

```python
sample = bytes.fromhex('5359808100010FBD5443')
print(list(parse_frames(sample)))
# -> [(0x80, 0x81, b'\x0f')]
```

(That sample is the manual's own test vector — a presence/breathing report from the sleep module. Your fall module's frames use its own command set, but the frame shape and checksum are identical.)

## Extending it

- Map `ctrl` to a name using the control-byte table above for logging.
- The `data` field's meaning depends on `cmd` and the module; consult that module's command table in its user manual before interpreting values.
- On an MCU the same loop ports directly: scan for `0x53 0x59`, read the length, accumulate, verify the checksum, then act on the payload.

## Why this matters

GPIO gives a 1-bit "something happened" with zero parsing. UART gives the full state — and this parser is the minimum to turn the byte stream into structured data. Use both: GPIO for an instant local alarm, UART for records and dashboards.

## Where to go next

- Get the wires right first: [Bring-Up checklist]({{ site.baseurl }}{% post_url 2026-10-22-radar-module-bring-up-checklist %}).
- The GPIO/UART split: [GPIO vs UART]({{ site.baseurl }}{% post_url 2026-10-21-gpio-vs-uart-fall-alarm-without-parser %}).
- Full specs: [Module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- Tuning: [Fall radar guide]({{ site.baseurl }}{% post_url 2026-10-17-fall-radar-guide %}), [Presence radar guide]({{ site.baseurl }}{% post_url 2026-10-17-presence-radar-guide %}).

Evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/).
