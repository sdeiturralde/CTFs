---
tags:
  - CTF
  - Cybersecurity
  - Pwned
Name: "[[Dance-Samba]]"
System: Linux(Docker/Ubuntu)
Difficulty: Medium
Focus: Exploitation through SMB, Escalation with /file
---


<div class="os-banner">
  <div class="os-border"></div>
  <img src="https://pwn3d.es/img/Linux.svg" 
       alt="systemd" 
       class="os-icon3">
  <div class="os-border"></div>
</div>

<br>

**Machine name**: Dance-Samba<br>
**System**: Linux(Docker/Ubuntu)<br>
**Difficulty**: Medium<br>
**Focus**: Exploitation through SMB, Escalation with /file
***

<br>
<br>
<br>

## 1. Port Scanning

```bash wrap
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 172.18.0.2 -oX nmap_TCP.xml && xsltproc nmap_TCP.xml -o samba_nmap_TCP.html && open samba_nmap_TCP.html &>/dev/null & disown
```

<iframe src="file:///C:/Users/intel/Documents/Obsidian/Obsidian/02 - Studies/Security/CTFs/Pwned/Imgs/samba_nmap_TCP.html" style="width:100%; height:500px;"></iframe>

<br>

Detected services:
- **21** &rarr; FTP
- **22** &rarr; SSH
- **139** &rarr; Samba Server
- **445** &rarr; Samba Server
***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. Exploitation
With the nmap analysis we already know that we can connect through FTP as **`anonymous`** user and we use that to check what we can find.

Inside we find a note with a hint.
![[Samba-dance 1.png]]
We have 2 possible users in here and to test it out we try to use use a command to crawl and get all the **"shares"**, **directories** and **data** available for this **`anonymous`** user.
```bash wrap 
netexec -t 200 smb 172.18.0.2 -u 'anonymous' -p '' -M spider_plus -o READ_ONLY=true 2>/dev/null
```
![[Samba-dance 2.png]]
We can see that there's a folder called **`macarena`** which confirm us that there's indeed a user called as such.

We suspect that the other user is the password but we try a dictionary attack just in case.
```bash wrap
netexec -t 64 smb 172.18.0.2 -u macarena -p /usr/share/wordlists/rockyou.txt --continue-on-success --ignore-pw-decoding | grep -v 'STATUS_LOGON_FAILURE'
```
![[Samba-dance 3.png]]
And indeed our hunch was correct, the password is **`donald`**.
With that we can both confirm  the folder that we discovered earlier and get into that same folder.
![[Samba-dance 4.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. Access as user with SSH keys
Once inside we discover a text called **`user.txt`** that **is** the flag of the user but for some reason for us it was empty.<br><span style="color:cyan">User Flag</span>

Anyway once inside we notice due to the folders that we are inside the folder of a user in Linux so we test what we can do and we see that we can create new folder.

We create a hidden folder called **`.ssh`** and we move inside.

Now we create new new **ssh** key pair, both a public one and a private one.
```bash wrap
ssh-keygen -t rsa -b 4096
```
We change the name to our public key to **"authorized_keys"** and we upload it with **Samba** to the folder that we just created.
![[Samba-dance 5.png]]

Now with our private key we access through **ssh** without any password since we have the keys needed to enter.
```bash
ssh -i id_rsa macarena@172.18.0.2
```
![[Samba-dance 6.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 3. Getting credentials for user
Once inside we start looking for clues and we find in a folder called **`/secret`** in the **`/home`** directory a file containing a "hash" but with a quick glance we noticed that is a Base64 string.
![[Samba-dance 7.png]]

With the help of the website called **cyberchef** on github we could confirm this and we found a password that is the password for our user **`macarena`**.
![[Samba-dance 8.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 4. Privilege escalation
with our new credentials we discover a command in **sudoers** that we can execute with elevated permissions.
![[Samba-dance 9.png]]

With a quick search in [GTFObins](https://gtfobins.org) we find that this binary is used to read another file so we need another file to use it.
![[Samba-dance 10.png]]

After a while searching all the system we find what we were looking for in the folder **`/opt`** there's a protected file that we can't read.
![[Samba-dance 11.png]]

We use the binary with the **sudo** permissions to read it and we find the credentials for the user **`root`**.
![[Samba-dance 12.png]]

And with that we find our final flag.<br><span style="color:red">Root Flag</span>
![[Samba-dance 13.png]]
***
