---
title: CozyHosting Writeup
date: 2023-09-15
categories: htb,pentesting
tags: htb,pentesting     # TAG names should always be lowercase
---
## Machine Info
* Difficulty: Easy


## Network Scanning

### Nmap
After we got the IP address of the target machine, we run nmap to scan all ports and enable OS detection, version detection, script scanning, and traceroute to discover the open ports and services that are running on the target.

```bash
sudo nmap 10.10.11.230 -sC -sV -T 4 -p-
```

![nmap](/img/cozyhosting/nmapscan.png)

Dont forget to add the domain name to the /etc/hosts file as follows so that you can view the site.

![nmap](/img/cozyhosting/hosts.png)
## Enumeration
A good thing to always practice instead is viewing every page, checking the source code to gain more information on what you're going up against, the only thing of intrest that we were able to find though is a login page

![nmap](/img/cozyhosting/indexpage.png)

![nmap](/img/cozyhosting/loginpage.png)

I tried a couple of default admin credentials but none of them worked.

![nmap](/img/cozyhosting/failadmin.png)

After no credential was successful, I continued to dig around to see what I can find before I run a directory scan, checking the network tab in devloper mode to see if any hidden files were being loaded, or even the debugger thinking that there may be a login system that exposes a way to change our cookies and get elevated privileges.

![nmap](/img/cozyhosting/network.png)

![nmap](/img/cozyhosting/debugger.png)

No luck on either or, pitty. Time to run a directory scan

```bash
wfuzz -c -f,/home/kali/SecLists/Discovery/Web-Content/raft-large-directories.txt --hc 404 -u "http://cozyhosting.htb"
```

![nmap](/img/cozyhosting/wfuzz.png)

After finishing my directory scans I became a little stumped, I wasn't able to find any interesting files or directories. I decided to view the /error directory just to see if it would give me any extra information that would help me.

![nmap](/img/cozyhosting/whitelabel.png)

Ahah, I've never seen this kind of error page before, its usually a "403 forbidden", "Page doesn't exist" or things relative to that, So after looking up what exactly is "White Label" and where does it come from I was able to find out that it is part of the "Spring Boot" Framework
