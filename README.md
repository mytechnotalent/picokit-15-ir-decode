![picokit-15-ir-decode](https://raw.githubusercontent.com/mytechnotalent/picokit-15-ir-decode/main/picokit-15-ir-decode.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-15 IR DECODE

### VS1838B NEC Decode and Authenticated Heartbeat
#### Lesson 15 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The fifteenth Picokit lesson. The node decodes NEC infrared frames from a
VS1838B receiver on GP5 and prints each frame's raw command byte, and every
five seconds it transmits an authenticated heartbeat over LoRa to a Python
gateway that logs and displays it. It reuses the standard node shape and
adds a timed NEC pulse decoder on top of it.

<br>

## What it teaches

- Timing the mark and space edges of a NEC infrared frame.
- Classing a bit space into a zero or a one and validating the byte
  complements.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and sending it
  with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| VS1838B OUT | GP5 | NEC infrared input |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every 200 ms it polls the VS1838B
on GP5 and, on a valid NEC frame, prints the raw command byte. Every 5
seconds it seals `{"n":15,"s":<seq>,"i":<cmd>}` with the field key and
sends it over LoRa. The gateway authenticates each frame and only then
parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_15_ir_decode.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-15 IR DECODE // NEC DECODE + AUTHENTICATED HEARTBEAT ===
IR cmd=0x45
IR cmd=0x46
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=15 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-16-ir-keys](https://github.com/mytechnotalent/picokit-16-ir-keys)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-15-ir-decode/blob/main/LICENSE)
