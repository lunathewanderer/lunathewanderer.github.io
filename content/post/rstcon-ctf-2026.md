---
title: RSTCON CTF 2026
description: "I want you to live a normal life and die a normal death."
draft: false
date: 2026-10-26T06:02:33.342Z
tags:
  - Network Forensics
  - Steganography
categories:
  - Writeup
cover: "https://i.pinimg.com/1200x/25/7c/b4/257cb41be751e844b0ec245069ce96a9.jpg"
banner: "https://i.pinimg.com/1200x/25/7c/b4/257cb41be751e844b0ec245069ce96a9.jpg"
lastmod: 2026-10-26T06:29:41.818Z
---

## 1. Layered Catch

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1792994790262-46d2d1.png)

`xxd` suggests that there is a hidden ZIP, alongside a passphrase (which we can see more clearly via exiftool)

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1792994749744-f09e67.png)

Indeed, `binwalk -e` provides a ZIP (password required) containing `flag.txt`

`rstcon{p0lyglot_png_h1des_a_zip}`

## 2. Low Bit Leeve

Throw the image onto `AperiSolve` for the flag, with the actual method being `zsteg`.

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1792995126921-85490c.png)

`rstcon{least_significant_secret_bits}`

## 3. Life Savings

We are presented with a corrupted PNG file, so we first inspect its hexadecimal dump.

![image](/images/posts/rstcon-ctf-2026-f495bb/image-1792995683429-a9f5c5.png)

We can see that the problem lies in the missing width and height bytes.


## 4.
