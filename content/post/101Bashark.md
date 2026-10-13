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
tshark -r challenge.pcap -Y "icmp" -T fields -e data.data | cut -c1-2 | uniq > exfil.txt

cat exfil.txt

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

Expansion: if it were to be decimal, such that:

```bash
cat exfil.txt

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

Multiple HTTP Auth Reuqest (GET method, followed by ACK) consists of abnormally lengthy, seemingly base64 in the basic credentials field.

```bash
tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d | file -
/dev/stdin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 544x600, components 3

tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed 's/firefly\://g' | base64 -d > hello.jpg
```
- This time we inspect "http" > "http.authbasic".
- `sed 's/firefly\://g'`: deletes every instances of `firefly:`
> `sed 's/.../.../'` the substitution command.
> 
> `firefly\:` the pattern we are searching for, the backlash being used to escape the colon (rendering them literally `firefly:`)
> 
> `//` the replacement section being empty, whatever matches the search pattern will be deleted.
> 
> `g` the global flag, matches every instances, not just the first.
- `base64 -d > hello.jpg` decodes the extracted data and write them into the image.



