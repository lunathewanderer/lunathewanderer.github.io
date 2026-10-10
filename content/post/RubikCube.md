---
title: "BrunnerCTF 2026: Rubik's Cube"
description: "high dif rizz ૮(˶ㅠ︿ㅠ)ა"
draft: false
date: 2026-08-23
tags:
  - Network Forensics
categories:
  - Writeup
cover: "/images/kujou-banner.jpeg"
banner: "/images/kujou-banner.jpeg"
---

> Figure out what moves they executed and write them in standard notation separated by underscores and wrapped in `brunner{}`. Example: If the sequence was R B B U F' F' D', the flag would be `brunner{R_B2_U_F2_D'}`.

The challenge presented us with a packet capture, seemingly from a Bluetooth smart cube.

Upon initial inspection, we can see that in the packet, there are only a few distinct types of protocols, mostly `HCI_CMD`, `HCI_EVT`, `HCI_MON`, and `ATT`.

<img width="1917" height="1126" alt="image" src="https://github.com/user-attachments/assets/2382f8e6-3c67-4b5f-8bec-91d5af70b710" />

Let's take a dive into some of the terminologies:

> [!NOTE]
> `HCI` stands for (Bluetooth) Host Controller Interface. It is the way a computer's Bluetooth software talks to its Bluetooth chip. The labels describe different roles in the same Bluetooth capture:
> 
> `HCI_MON`: Linux's monitor wrapper, recording what type of Bluetooth activity follows.
> 
> `HCI_CMD`: Command (from the computer to adapter): the computer tells its Bluetooth adapter to do something.
> 
> `HCI_EVT`: Event (from the adapter to computer): the adapter reports a result or something it observed.
> 
> `The Attribute Protocol (ATT)` is a low-level data communication protocol used in Bluetooth Low Energy (BLE) to store, organize, and exchange small pieces of data between two connected devices.
> 
> `Bluetooth Low Energy (BLE, or Bluetooth LE)` is a wireless personal area network technology designed for very low power consumption and short-range communication
> 
> `L2CAP` (Logical Link Control and Adaptation Protocol) is a core layer in the Bluetooth protocol stack that bridges higher-level applications and lower-level baseband radio layers. It sits directly on top of the Host Controller Interface (HCI) on the host side.

Protocol Hierarchy:

<img width="1519" height="361" alt="image" src="https://github.com/user-attachments/assets/707e72c5-9dc1-49d1-9bea-efa58318f8d3" />

> In Wireshark, a frame is one numbered item in the capture's packet list.

The HCLs are just filler for this challenge, playing the role of initialization. What we should focus on is the ATT frames, where the real data hidden. Once the computer is connected to the cube, ATT lets it read a value, write a value, or receive an update. GATT is the system that organizes those values into services and characteristics; ATT carries the individual messages. To understand more about the relationship about ATT, GATT, and Bluetooth in general, read this [article](https://medium.com/@QuarkAndCode/bluetooth-ble-protocol-stack-gatt-streaming-and-wi-ble-explained-9103d6e94572).

The flow of packet capture itself is pretty straightforward to understand:

- Frames 1 - 990: Scan nearby Bluetooth devices
- Frames 991 - 1000: Connect to QiYi smart cube
- Frames 1001 - 1069: Discover
- Frames 1070 - 1074: Hello exchanged
- Frames 1082 - 1169: Turns made by the Rubik. There are 30 incoming ATT notifications containing move updates. Other packets between them are acknowledgements and Bluetooth bookkeeping. These 30 notifications are the rubik's moves in transmission.
- Frames 1172 - 1181: Cleanup. The computer disables notifications and disconnects.

Filter out the main content with `btatt.opcode == 0x1b && btatt.handle == 0x001a`.
- `btatt` refers to the Bluetooth Attribute Protocol.
- `btatt.opcode == 0x1b` isolates Bluetooth Low Energy (BLE) Handle Value Notification packets.
- `btatt.handle == 0x001a` filters for the handle in the packet capture.

<img width="1918" height="1126" alt="image" src="https://github.com/user-attachments/assets/b4eae98d-e981-4f51-b563-b98c847d165a" />

> In Bluetooth terms, "received" means received by the computer from the cube. A Handle Value Notification is the cube telling the computer that a characteristic's value has changed. In this capture, that characteristic is addressed by handle 0x001A. The ATT specification defines notifications as server to client; its packet contains an opcode, a handle, and a value.

In the "Source" and "Destination" of the ATT packets, we can see the conversation from and to a device with the name QY-QYSC-S-0D1E. This is the name of the label QiYi Smart Cube, so we know that this is the data from the rubik cube itself. Extract the message (a.k.a. the `value` row, or `bttat.value`) with tshark:

```bash
tshark -r rubiks.pcapng \
  -Y 'btatt.opcode == 0x1b && btatt.handle == 0x001a && len(btatt.value) == 96' \
  -T fields -e frame.number -e btatt.value
```

<img width="1540" height="594" alt="image" src="https://github.com/user-attachments/assets/2cd88e90-af0f-4f34-9c21-d6858f3b4f3d" />

However, we can not read it directly since QiYi encrypts the message. [With some researching](https://www.reddit.com/r/Cubers/comments/1dkgu8b/i_reverse_engineered_the_qiyi_smartcube_protocol/), we know that all messages sent to/received from the cube are encrypted using AES-128 in ECB mode with the fixed key `57b1f9abcd5ae8a79cb98ce7578c5108`. 

> This is a 128-bit key, there are 32 hexadecimal digits, or 16 bytes. AES works on 16-byte blocks, so a 96-byte `bttat.value` contains six sub-blocks. ECB means that each block can be decrypted with the key without an extra initialization value (IV). `57 b1 f9 ab cd 5a e8 a7 9c b9 8c e7 57 8c 51 08` = 16 bytes x 8 bits = 128 bits, hence the name AES-128.

Upon trying to decrypt a message with CyberChef:

<img width="1101" height="808" alt="image" src="https://github.com/user-attachments/assets/a1e245ec-f29c-440b-b0c3-69afd7efaa43" />

To understand the message of the Cube Update, we need to break down the components of [Qiyi's smartcube protocol: ](https://codeberg.org/Flying-Toast/qiyi_smartcube_protocol).

- `[0]`: `0xFE` the magic byte
- `[1]`: Length of the message
- `[2]`: Opcode (Operation code - 0x02 = Hello, 0x03 indicates a move)
- `[3:6]`: Timestamp
- `[7:34]`: Initial cube state
- `[34]`: Move byte
- `[35]`: Battery level
- `[36:90]`: Previous move
- `[91]`: Solved or Not solved
- `[92:93]`: Checksum

<img width="1906" height="1074" alt="image" src="https://github.com/user-attachments/assets/9791858c-906a-46c2-a79a-0c0eeb9193d3" />

> In the example I tried to decode using CyberChef, its byte number 34 is `0x0a`, or `F`. 

**Script solve**

**`WSL`**

```bash
python3 - "rubiks.pcapng" << 'PY'
import subprocess
import sys

pcap = sys.argv[1]
key = "57b1f9abcd5ae8a79cb98ce7578c5108"
display_filter = (
    "btatt.opcode == 0x1b && "
    "btatt.handle == 0x001a && "
    "len(btatt.value) == 96"
)

result = subprocess.run(
    [
        "tshark", "-r", pcap, "-Y", display_filter,
        "-T", "fields", "-E", "separator=/t", "-E", "occurrence=f",
        "-e", "frame.number", "-e", "btatt.value",
    ],
    capture_output=True, text=True, check=True,
)

for line in result.stdout.splitlines():
    frame, hex_value = line.split("\t", 1)
    ciphertext = bytes.fromhex(hex_value.replace(":", "").replace(" ", ""))

    plaintext = subprocess.run(
        ["openssl", "enc", "-aes-128-ecb", "-d", "-nopad", "-K", key],
        input=ciphertext, capture_output=True, check=True,
    ).stdout

    if plaintext[0] == 0xFE and plaintext[2] == 0x03:
        print(f"Frame {frame}: byte[34] = 0x{plaintext[34]:02x}")
PY
```

**`notifications.txt` and `sol.py`**

```py
import re
from pathlib import Path
from Crypto.Cipher import AES

KEY = bytes([
    87, 177, 249, 171, 205, 90, 232, 167,
    156, 185, 140, 231, 87, 140, 81, 8
])

MOVES = {
    0x01: "L'", 0x02: "L",
    0x03: "R'", 0x04: "R",
    0x05: "D'", 0x06: "D",
    0x07: "U'", 0x08: "U",
    0x09: "F'", 0x0A: "F",
    0x0B: "B'", 0x0C: "B",
}

def simplify(moves):
    result = []
    i = 0
    while i < len(moves):
        face = moves[i][0]
        amount = 0
        while i < len(moves) and moves[i][0] == face:
            amount += 3 if moves[i].endswith("'") else 1
            i += 1

        amount %= 4
        if amount == 1:
            result.append(face)
        elif amount == 2:
            result.append(face + "2")
        elif amount == 3:
            result.append(face + "'")
    return result

cipher = AES.new(KEY, AES.MODE_ECB)
seen = set()
moves = []

for line in (Path(__file__).resolve().parent / "notifications.txt").read_text().splitlines():
    encrypted = bytes.fromhex(re.sub(r"[^0-9a-fA-F]", "", line))
    if not encrypted or len(encrypted) % 16:
        continue

    plain = cipher.decrypt(encrypted)

    if plain[0] != 0xFE or plain[2] != 0x03:
        continue

    timestamp = int.from_bytes(plain[3:7], "big")
    move = MOVES.get(plain[34])

    if move and (timestamp, move) not in seen:
        seen.add((timestamp, move))
        moves.append(move)

print("Raw moves:", " ".join(moves))
print("Flag:", "brunner{" + "_".join(simplify(moves)) + "}")
```

**`brunner{F2_U2_B'_R2_B'_L_D2_R_F2_U_B2_D_R2_B2_D'_B2_U_F2_U_B}`**
