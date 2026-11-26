---
title: "CyberDefenders' Labs: Endpoint Forensics"
description: "「月が綺麗ですね」"
draft: false
date: 2026-11-22T13:12:54.976Z
tags:
  - Endpoint Forensics
categories:
  - Writeup
lastmod: 2026-11-26T10:22:02.012Z
cover: /images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/emilia-scale2-1795443901781-2c6fe3.jpg
banner: /images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/emilia-scale2-1795443901781-2c6fe3.jpg
---

# 1. Insider
- **Date**: Nov 20, 2026
- **Category**: Endpoint > Disk Forensics
- **Tools used**: FTK Imager

We are presented with an `.AD1` file. 

**Q1. Which Linux distribution is being used on this machine?**

`/boot` is a directory in the Linux filesystem that contains essential files needed to start the operating system. When you turn on your computer, these files help load the Linux kernel into RAM and begin system initialization.

**Common contents of `/boot`**

| File or directory | Purpose |
|---|---|
| `vmlinuz` | The compressed Linux kernel |
| `initrd.img` or `initramfs.img` | A temporary filesystem containing drivers and tools needed during early startup |
| `grub/` | GRUB bootloader configuration and supporting files |
| `System.map` | A mapping between kernel symbols and their memory addresses |
| `config-*` | Configuration options used when compiling the kernel |
| `efi/` | A common mount point for the EFI System Partition on UEFI systems |

> **GNU GRUB (GRand Unified Bootloader)** is the standard boot loader package from the **GNU Project** used by most Linux distributions to load the operating system kernel into memory after firmware **(BIOS or UEFI)** initialization. It presents a boot menu so you can choose between operating systems or kernel versions, and supports features like disk encryption, network booting, and command-line rescue tools.

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795611223301-da7326.png)

Go to `/boot` and it is visible that the distro used is `kali`.

`Answer: kali`

**Q2. What is the MD5 hash of the Apache access.log file?**

Apache `access.log` is a log file maintained by the Apache HTTP Server that records the HTTP requests received by a web server. Whenever a user visits a webpage, searches for a product, submits a form, or downloads a file, their browser sends HTTP requests to the server. Apache can record information about each of these requests in `access.log`.

Kali Linux is a Debian-based distro. The default path of that file in Debian is `/var/log/apache2/access.log`.

**Apache's Combined Log Format Example**

| Field | Value | Meaning |
|---|---|---|
| Client IP | `192.168.1.100` | IP address of the requesting client |
| Remote identity | `-` | Identity information is unavailable |
| Authenticated user | `-` | No HTTP authentication username recorded |
| Timestamp | `25/Nov/2026:10:30:45 +0700` | Date, time and timezone of the request |
| HTTP method | `GET` | Method used to request a resource |
| Request URI | `/search.php?search=book` | Requested resource and query string |
| HTTP version | `HTTP/1.1` | HTTP protocol version |
| Status code | `200` | Result of the HTTP request |
| Response size | `1450` | Response body size in bytes |
| Referer | `-` | No referring page recorded |
| User-Agent | `Mozilla/5.0` | Client-reported browser or software information |

**`access.log` vs. `error.log`**

| Feature | `access.log` | `error.log` |
|---|---|---|
| Purpose | Records HTTP request activity | Records server errors and diagnostic messages |
| Evidence | IP addresses, URIs, timestamps, status codes | Errors, warnings, configuration problems |
| Forensic value | Reconstructing web activity | Understanding server behavior and failures |

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795614710125-36abdf.png)

However, the log is an empty file. At first, I thought this could be a mistake, but this is actually intended and pretty much normal (and may suggest that Apache was never run on this system). It turns out that an empty (0-byte) file can have the MD5 hash, which is `d41d8cd98f00b204e9800998ecf8427e`

`Answer: d41d8cd98f00b204e9800998ecf8427e`

**Q3. It is suspected that a credential dumping tool was downloaded. What is the name of the downloaded file?**

Downloads are often stored in the user's default download directory, which is `/root/Downloads`

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795616222381-ab299d.png)

`Answer: mimikatz_trunk.zip`

**Q4. A super-secret file was created. What is the absolute path to this file?**

`.bash_history` is a hidden file in Linux that stores the history of commands entered by a user through the Bash shell (Bourne Again Shell). When a user executes commands to navigate directories, install software, download files, or modify system configurations, those commands may be recorded in `.bash_history`

| User | Location |
|---|---|
| Current user | `~/.bash_history` |
| Regular user (e.g., alice) | `/home/alice/.bash_history` |
| Root user | `/root/.bash_history` |

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795616625950-8aaea2.png)

`Answer: /root/Desktop/SuperSecretFile.txt`

**Q5. What program used the file `didyouthinkwedmakeiteasy.jpg` during its execution?**

Keep on checking the bash history

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795616668782-553e77.png)

`Answer: binwalk`

**Q6. What is the third goal from the checklist Karen created?**

Users often store checklists and notes on their desktop, so `/root/Desktop` should be a good starting point

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795617457139-bb90bb.png)

`Answer: profit`

**Q7. How many times was Apache run?**

The log files in /var/log/apache2/ will show evidence of Apache's activity. Since the files are empty, it might indicate that Apache was not used.

`Answer: 0`

**Q8. This machine was used to launch an attack on another. Which file contains the evidence for this?**

There is an unusual image in `/root`. The image shows a command prompt with elevated privileges, likely pointing to an attack scenario.

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795618856052-d04af8.png)

`Answer: irZLAohL.jpeg`

**Q9. It is believed that Karen was taunting a fellow computer expert through a bash script within the Documents directory. Who was the expert that Karen was taunting?**

Wandering around, `/root/Documents/myfirsthack/firstscript_fixed` contains the information in the question.

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795621282434-b9e98f.png)

`Answer: Young`

**Q10. A user executed the su command to gain root access multiple times at 11:26. Who was the user?**

To find out who switched to root, we should examine the system's authentication logs. 

> `/var/log/auth.log` is a Debian Linux log file that records authentication and authorization-related activities.

**What does `auth.log` record?**

> - SSH brute-force attempts.
> - Unauthorized successful authentication.
> - Suspicious account creation.
> - Privilege escalation attempts.
> - Suspicious use of `sudo`.
> - Source IP addresses associated with login attempts.
> - Timelines of user sessions.

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795621520589-f868db.png)

In `auth.log`, there are messages around 11:26 about "successful `su` for `postgres` by `root`"

**Sidenote**
> - `daemon.log` records the activity, status, and errors of background services (daemons) that run without direct user interaction.
> - `dpkg.log` records a chronological history of every low-level package action - including installations, upgrades, configurations, and removals (whether triggered via apt, apt-get, dpkg, or GUI software centers).

`Answer: postgres`

**Q11. Based on the bash history, what is the current working directory?**

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795621446145-38223b.png)

The last `cd` was directed to `/root/Documents/myfirsthack/`

`Answer: /root/Documents/myfirsthack/`

# 2. The Crime
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Mobile Forensics (Android)
- **Tools used**: DB Browser for SQLite, ALEAPP

# 3. RedLine
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility, Strings

# 4. Ramnit
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 3, VirusTotal

# 5. Reveal
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 3

# 6. Silent Breach
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk Forensics
- **Tools used**: FTK Imager, SQLite Viewer, Strings, CyberChef

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795688431877-3b1d93.png)

![image](/images/posts/cyberdefenders-labs-endpoint-forensics-2a10be/image-1795688466630-c5c8fb.png)

# 7. KrakenKeylogger
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Windows Artifact and Log Analysis
- **Tools used**: DB Browser for SQLite, LECmd, Timeline Explorer

# 8. AzurePot
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk, Memory, and Log Forensics (Linux)
- **Tools used**: FTK Imager, `grep`, `awk`

# 9. Sysinternals
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Registry Forensics
- **Tools used**: Registry Explorer, Event Log Explorer, VirusTotal, FTK Imager, Autopsy

# 10. BlackEnergy
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility

# 11. Eli
- **Date**: xxx xx, 2026
- **Category**: Endpoint > ChromeOS and Browser Forensics
- **Tools used**: CyberChef, DCode, DB Browser for SQLite, Unfurl, CLEAPP, RLEAPP

# 12. MrGamer
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and User Artifact Forensics
- **Tools used**: CyberChef, DCode, DB Browser for SQLite, Unfurl, Arsenal Image Mounter, Autopsy

# 13. Seized
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics (Linux)
- **Tools used**: Volatility, CyberChef, `grep`

# 14. MrRobot
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 2, Volatility 3

# 15. l337 S4uc3
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory and Network Forensics
- **Tools used**: Volatility, Wireshark, NetworkMiner, Brim

# 16. Hacked
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk Forensics (Linux)
- **Tools used**: FTK Imager, `last`

# 17. LGDroid
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Mobile Forensics (Android)
- **Tools used**: DB Browser for SQLite

# 18. Brave
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 3, HxD

# 19. AfricanFalls
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Browser Forensics
- **Tools used**: FTK Imager, Autopsy

# 20. DumpMe
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 2

# 21. HireMe
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk Forensics
- **Tools used**: FTK Imager, Autopsy, Registry Explorer, LECmd, RegRipper

# 22. Phishy
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Browser Forensics
- **Tools used**: FTK Imager, Autopsy, Registry Explorer, DB Browser for SQLite, VirusTotal

# 23. Injector
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Memory Forensics
- **Tools used**: FTK Imager, Autopsy, Volatility, Registry Explorer, RegRipper

# 24. Hammered
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Log Analysis (Linux)
- **Tools used**: `grep`, `awk`, `sed`

# 25. Ulysses
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory and Log Forensics (Linux)
- **Tools used**: Volatility, 010 Editor, Autopsy, FTK Imager

# 26. Spotlight
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk Forensics (macOS)
- **Tools used**: Autopsy, `mac_apt`, SQLite

# 27. Jailbroken
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Mobile Forensics (iOS)
- **Tools used**: iLEAPP, Autopsy, `mac_apt`, DB Browser for SQLite

# 28. CorporateSecrets
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Event Log Forensics
- **Tools used**: FTK Imager, Registry Explorer, RegRipper, HxD, DB Browser for SQLite, Event Log Explorer

# 29. Szechuan Sauce
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk, Memory, Network, and Log Forensics
- **Tools used**: Registry Explorer, Brim, Wireshark, FTK Imager, Volatility 3, Event Log Explorer, VirusTotal

# 30. Hunter
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk Forensics
- **Tools used**: FTK Imager, Registry Explorer, RECmd, RegRipper, DCode, DB Browser for SQLite, 010 Editor, Autopsy, Arsenal Image Mounter

# 31. SpottedInTheWild
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Disk and Event Log Forensics
- **Tools used**: Arsenal Image Mounter, SQLite Viewer, Registry Explorer, Event Log Explorer, Strings, CyberChef

# 32. NintendoHunt
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 2, Volatility 3

# 33. Patrick
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Mobile Forensics (iOS)
- **Tools used**: CyberChef, iLEAPP, DCode, DB Browser for SQLite, Unfurl, `ios_apt`

# 34. TeamSpy
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics and Document Analysis
- **Tools used**: Volatility 2.6, OfficeMalScanner, VirusTotal

# 35. DeepDive
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility 2

# 36. BankingTroubles
- **Date**: xxx xx, 2026
- **Category**: Endpoint > Memory Forensics
- **Tools used**: Volatility, Strings
