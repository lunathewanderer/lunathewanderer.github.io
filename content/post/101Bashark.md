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

# 1. `WannaGame Recruit 2026 - enum or exfil?`

Reference: https://blog.devobass-will.win/posts/wannagame-recruit-2026/#enum-or-exfil

To extract the data exfiltrated via ICMP tunneling:

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

Since these are hexadecimal bytes, `cat exfil.txt | xxd -r -p` to decode it into `sneaky_network`. 

If it were to be decimal, such that:
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

`cat exfil.txt | awk '{printf "%c", $1} END {print ""}'` would return `sneaky_network`.

> `awk '{printf "%c", $1} END {print ""}' exfil.txt` is a slightly cleaner version.










