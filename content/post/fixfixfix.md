---
title: FixFixFix
draft: false
date: 2026-11-27T14:45:27.400Z
categories:
  - Writeup
lastmod: 2026-11-28T04:06:07.456Z
description: 'Nỗ lực ở nơi không ai đoái hoài, rồi tỏa sáng ở nơi vạn người chú ý'
tags:
  - Network Forensics
  - Endpoint Forensics
cover: /images/posts/fixfixfix-e674c0/frie2-1795792930520-7cb2c8.jpg
banner: /images/posts/fixfixfix-e674c0/frie2-1795792930520-7cb2c8.jpg
---

> **Author: nh0kt1g3r12**
>
>*Dear IR Team,*
>
> *A client reported system issues after encountering an online verification page during normal web browsing. Following interaction with this interface, their workstation showed signs of compromise.*
> 
> ![image](/images/posts/fixfixfix-e674c0/image-1795791649775-76b8b8.png)
>
> *We detected suspicious activities including unusual process executions, external network connections, and system configuration changes. Evidence suggests malware installation with persistence mechanisms and credential harvesting capabilities.*
>
> *The device is isolated. Please investigate (note that the timestamp format is YYYY-MM-DD HH:MM:SS and the timezone in UTC.):*

**Q1: What initial access technique was used to lure the user to deploy the malware?**

The given image is familiar in the phising technique [ClickFix](https://www.microsoft.com/en-us/security/blog/2025/08/21/think-before-you-clickfix-analyzing-the-clickfix-social-engineering-technique/)

> *"The ClickFix technique attempts to trick users into running malicious commands on their devices by taking advantage of their target’s tendency to solve minor technical issues and other seemingly benign interactions, such as human verification and CAPTCHA checks. It typically gives the users instructions that involve clicking prompts and copying, pasting, and running commands directly in the Windows Run dialog box, Windows Terminal, or Windows PowerShell. It’s often combined with delivery vectors such as phishing, malvertising, and drive-by compromises, most of which even impersonate legitimate brands and organizations to further reduce suspicion from their targets." - Microsoft*

`Answer: ClickFix`

**Q2: What is the exact date and time of initial compromise?**

**Q3. What is the MITRE ATT&CK ID of the 1st stage downloader?**

**Q4. When was the malware first executed on the system?**

**Q5. What is the malware family?**

**Q6. When was the malware first found in the wild?**

**Q7. What is the full path of persistence?**

**Q8. What is the MITRE ATT&CK ID of the persistence technique?**

**Q9. What IP address and port was used for C2 communications?**

**Q10. When was the first C2 command executed?**

**Q11. How many different commands did the attacker execute?**

**Q12. What is the full command used to download and exfiltrate user credentials?**

**Q13. Where did the attacker exfiltrate browser credentials to?**

**Q14. What are the exfiltrated credentials? (username:password)**
