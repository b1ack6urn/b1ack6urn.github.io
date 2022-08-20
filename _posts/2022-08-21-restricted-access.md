---
title: "Defense in depth: Allowing 3rd party Security Vendors in your network"
date: 2022-08-21
categories:
  - Research 
tags:
  - Security Research
  - DevOps
---


## User Creation Setup Guide

_Following steps should be run as a Root user. 
The articles assumes that the said Root user has already generated the Assymetric Key-Pair using OpenSSH ssh-keygen_


##### 1. Create a least privileged User

```bash
sudo adduser --home /restricted/home restrictedUser
#please remember the password and save it somewhere safe 
```

 
##### 2. Setup Public-Key Authentication for the above created user
```bash
# Authorize the Public key for ssh
sudo mkdir /restricted/home/.ssh
sudo bash -c 'echo "<paste your public key here>" >> /restricted/home/.ssh/authorized_keys'

# give permissions to the restrictedUser of the .ssh directory
sudo chown -R restrictedUser:restrictedUser /restricted 

```
##### 3. For IP Whitelisting and Public Key Authentication, ensure that `PubkeyAuthentication yes` is set in `/etc/ssh/sshd_config` and restart sshd
```bash
# all ssh configuration lies in this file
sudo vi /etc/ssh/sshd_config
`
PasswordAuthentication no #disable all ssh authentication by default
Match Address <whitelisted-ips>
        PubkeyAuthentication yes #enable ssh for above ips, hosts
        # uncomment the below line if Password auth is also used
        #PasswordAuthentication yes
`
sudo systemctl restart sshd
```

##### 4. Install Dependencies
```bash
sudo apt update
sudo apt install <packages required by the restrictedUser>
```

##### 5. Allow the user to run only two Scripts
```bash
# This will add entry in sudoers file to execute only two commands as ROOT
sudo bash -c 'echo "restrictedUser ALL=(ALL) NOPASSWD: /usr/bin/nano /etc/hosts" >> /etc/sudoers.d/restrictedUser'
sudo bash -c 'echo "restrictedUser ALL=(ALL) NOPASSWD: /usr/bin/find / -name *" >> /etc/sudoers.d/restrictedUser'

```


## Cleanup
##### 1. Revoking the user
```bash
sudo rm /etc/sudoers.d/restrictedUser 
sudo userdel restrictedUser 
sudo rm -r /restricted/home
```



