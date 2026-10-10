---
title: "V1TCTF 2026: BasicQnA"
description: "The forgotten past..."
draft: false
date: 2026-06-28
tags:
  - Network Forensics
categories:
  - Writeup
cover: "/images/kujou-banner.jpeg"
banner: "/images/kujou-banner.jpeg"
---

> Connect to [https://basicqna.v1t.site/](https://basicqna.v1t.site/) and answer the questions pertaining to the packet capture.

![.](/images/posts/0003.png)

Go to Statistics > Conversations > TCP. 

![.](/images/posts/0004.png)

Our answer is unfold, since it was packed with 172.29.9.159 as adress A and 13.212.67.96 as address B. 

`Answer Q1: 172.29.9.159,13.212.67.96`

![.](/images/posts/0005.png)

> The **Secure Shell (SSH) service** operates by default on **network port 22** to **provide encrypted, secure remote access to computer systems**. 

We filter the packet with `tcp.port == 22`

![.](/images/posts/0006.png)

`Answer Q2: SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.2`

![.](/images/posts/0007.png)

> Nmap is the most common and widely used tool for active network reconnaissance, port scanning, and host discovery.

And indeed, we can see its presence in the image above.

`Answer Q3: nmap`

![.](/images/posts/0008.png)

I tried to `Ctrl + F` and try to search for sussy keywords (by sussy, I mean "affirmative"). `true` returns me the right one!

![.](/images/posts/0009.png)

`Answer Q4: tcp.stream eq 4491`

![.](/images/posts/0010.png)

Look below the `"role": "admin", "success": true,...` we can see the line `â€œuserâ€:...` 

`Answer Q5: support_c30cde@corpvault.local`

![.](/images/posts/0011.png)

![.](/images/posts/0012.png)

![.](/images/posts/0013.png)

Scattering throughout the packet capture is the hint for privilege escalation vulnerabilities in the WordPress Maps Pro plugin. Furthermore, there are keywords such as `fc-call-nonce`, `wp_ajax_noprix`. [This is the article for the mentioned CVE](https://nvd.nist.gov/vuln/detail/cve-2026-8732).

`Answer Q6: CVE-2026-8732` 

![.](/images/posts/0015.png)

Search around for keyword `backup` 

![.](/images/posts/0016.png)

> RCE stands for **remote code execution**. It is a critical software security flaw that lets an attacker run malicious commands or code on a target computer or server over a network or the internet without any physical access. 

That said, this line seems suspicious: `<input name="backup_name" value="daily-contracts; echo HOME=$HOME; pwd; ls -la ~" placeholder="daily-contracts">`

`Answer Q7: backup_name` 

![.](/images/posts/0017.png)

Searching around, we can see that the attacker has accessed root privilege, executing command such as `whoami` or `ls` .

![.](/images/posts/0018.png)

I tried to search for `cat` , applying the â€œMultiple occurencesâ€ filter.

![.](/images/posts/0019.png)

![.](/images/posts/0020.png)

![.](/images/posts/0021.png)

`Answer Q8: /app/static/.env, /app/templates/.env`

![.](/images/posts/0022.png)

![.](/images/posts/0023.png)

![.](/images/posts/0024.png)

`Answer Q9: Ich1ck3nPlus`

![.](/images/posts/0025.png)

**`v1t{llm_c0uld_s0lv3_th1s_ez_chall3ng3!!!}`**
