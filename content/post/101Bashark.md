---
title: "Digital Forensics Script Kiddie"
description: "The truth is, you know... I never went to school either."
draft: false
date: 2026-10-06
tags:
  - Programming
  - Network Forensics
categories:
  - Learning
cover: "https://i.pinimg.com/1200x/e6/b0/0b/e6b00bd9c0abc5abb852644c81fcfee0.jpg"
banner: "https://i.pinimg.com/1200x/e6/b0/0b/e6b00bd9c0abc5abb852644c81fcfee0.jpg"
---

# WannaGame Recruit 2026 - enum or exfil?

**Reference**: https://blog.devobass-will.win/posts/wannagame-recruit-2026/#enum-or-exfil

## Part 1

Data exfiltration via ICMP tunneling:

```bash
$ tshark -r challenge.pcap -Y "icmp" -T fields -e data.data | cut -c1-2 | uniq > exfil.txt

$ cat exfil.txt
73
6e
65
61
6b
79
5f
6e
65
74
77
6f
72
6b
```

- `tshark -r challange.pcap`: read packets from that capture file.
- -`Y "icmp"`: apply the display filter ICMP
- `-T fields`: output selected fields instead of the usual packet summary.
- `-e data.data`: select the packet payload field named `data.data`.
- `cut -c1-2`: keep characters 1 through 2 from each line.

For a hex payload, that 2 characters exfiltrated from the tunneling construct one byte. Since they are hexadecimal bytes, `cat exfil.txt | xxd -r -p` to decode it into `sneaky_network`. 

> `data.data` is a display filter field that targets the generic raw data payload layer (often referred to as the generic data or unparsed application data) when Wireshark/TShark cannot or does not dissect a packet into a higher-level protocol.
> 
> Read devobass's writeup to understand the structure of the ICMP tunneling!

**Food for thought**: If it were to be decimal, such that:

```bash
$ cat exfil.txt
115
110
101
097
107
121
095
110
101
116
119
111
114
107
```

`cat exfil.txt | awk '{printf "%c", $1}` would return `sneaky_network`.

> Note 1: To add a newline, append `END {print ""}'`.
> 
> Note 2: In the same vein, `awk '{printf "%c", $1} END {print ""}' exfil.txt` is a slightly cleaner version.

## Part 2

Data exfiltration via HTTP Auth Request.

Multiple HTTP Auth Reuqests (GET method, followed by ACK) consist of abnormally lengthy, seemingly base64 credentials (in the basic credentials field).

```bash
$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d | file -
/dev/stdin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 544x600, components 3

$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed 's/firefly\://g' | base64 -d > hello.jpg
```
- This time we inspect "http" > "http.authbasic".
- `sed 's/firefly\://g'`: deletes every instances of `firefly:`
> - `sed 's/.../.../'` the substitution command.
> 
> - `firefly\:` the pattern we are searching for, the backlash being used to escape the colon (rendering them literally `firefly:`)
> 
> - `//` the replacement section being empty, whatever matches the search pattern will be deleted.
> 
> - `g` the global flag, matches every instances, not just the first.
- `| base64 -d > hello.jpg` decodes the extracted data and write them into the image.
- `file -` tells `file` to read from the stdin.

# WannaGame Freshmen 2025

## Free Flag on Network

To reverse the `.png`, loop processes lines in ascending order, but reverses the bytes within each line.

```bash
#!/usr/bin/sh
for i in `seq 1 100`
do
	sed -n "$i"p flag-reverse.png | base64 -d | xxd -p -c1 | tac | xxd -p -r >> flag.png
done
```
- `for i in ...` starts a loop; i takes each value produced by the command inside the backticks.
- `seq 1 100` runs seq and substitutes its output: the numbers 1 through 100, inclusive. Its equivalent in Python is `for i in range (1, 101)
- `do ... done` mark the beginning and end of the loop body.
- `sed -n "$i"p flag-reverse.png` prints line number `i` from the file while suppressing normal output.
- `base64 -d` decodes that line from Base64 into bytes.
- `xxd -p -c1` displays the bytes as plain hexadecimal, with one byte per line. For example, ABC becomes 41, 42, 43, each on its own line.
- `tac` reverses the line order. Since each line is one byte, this reverses the byte order for the current input line.
- `xxd -p -r` converts those hex bytes back into raw bytes.

> **`tac`** is the reverse of **`cat`**

```python
from pathlib import Path
import base64

chunks = []
for i in range(100):
    encoded = Path(f"flag{i}.txt").read_text().splitlines()[3]
    chunks.append(base64.b64decode(encoded)[::-1])

Path("recovered_flag.png").write_bytes(b"".join(chunks))
```
- `from pathlib import Path` imports Python's path and file-handling class.
- `import base64` imports functions for decoding and encoding Base64 data.
- First, we create an empty list to store reversed byte chunk
- `encoded = Path(f"flag{i}.txt").read_text().splitlines()[3]`
> This line finds and reads one file, then selects one line from it:
> - `f"flag{i}.txt"` is an f-string. Python substitutes the current value of i into the filename, so it checks flag0.txt, flag1.txt, …, flag99.txt.
> - `Path(...)` represents that filename as a path.
> - `.read_text()` reads the file as text.
> - `.splitlines()` splits the text into a list of lines.
> - `[3]` selects the item at index 3. Python indexes from zero, so this is the fourth line, the line containing Base64.
> - The selected line is assigned to encoded.
- `chunks.append(base64.b64decode(encoded)[::-1])`
> - `base64.b64decode(encoded)` decodes the Base64 text into raw bytes.
> - `[::-1]` reverses the byte sequence. It reverses the bytes inside this chunk; it does not change the order of the chunks.
> - `chunks.append(...)` adds the reversed bytes to the end of the list.
- `Path("recovered_flag.png").write_bytes(b"".join(chunks))`
> - `b""` is an empty byte string.
> - `b"".join(chunks)` combines all byte chunks in list order, with no separator between them.
> - `Path(...).write_bytes(...)` writes the combined bytes to `recovered_flag.png`. It creates the file or overwrites it if it already exists.


