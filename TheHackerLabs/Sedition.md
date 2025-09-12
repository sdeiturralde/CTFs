---
tags:
  - CTF
  - Cybersecurity
  - TheHackerLabs
Name: "[[Sedition]]"
System: Linux (Debian)
Difficulty: Easy
Focus: Exploitation through SMB, ZIP cracking, Escalation with /sed
---



<div class="os-banner">
  <div class="os-border"></div>
  <img src="https://labs.thehackerslabs.com/static/uploads/machines/sedition.jpg" 
       alt="systemd" 
       class="os-icon3">
  <div class="os-border"></div>
</div>

<br>

**Machine name**: Sedition<br>
**System**: Linux(Debian)<br>
**Difficulty**: Easy<br>
**Focus**: Exploitation through SMB, ZIP cracking, Escalation with `/sed`
***

<br>
<br>
<br>

## 1. Port Scanning
```bash wrap
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 192.168.0.110 -oX nmap_TCP.xml && xsltproc nmap_TCP.xml -o Sedition_nmap_TCP.html && open Sedition_nmap_TCP.html &>/dev/null & disown
```

<iframe src="file:///C:/Users/Skyveck/Documents/Documentos/Obsidian/Obsidian/02 - Studies/Security/CTFs/The Hacker Labs/Imgs/Sedition_nmap_TCP.html" style="width:100%; height:500px;"></iframe>

<br>

Detected services:
- **139** &rarr; Samba server (NetBIOS session)
- **445** &rarr; Samba server (SMB file service)
- **65535** &rarr; SSH
***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. User enumeration
Since we have a **Samba** server we can use that to enumerate users.
```bash
enum4linux -a 192.168.0.110
```
![[Sedition 1.png]]
![[Sedition 2.png]]
![[Sedition 3.png]]
From this we can gather much information. We can check it [[Common services and basic advices#SMB|Here]]<br>
First we can see that the identifier **RID** of an user `cowboy`. That same user is spotted in the last part with a security identifier **SID**, in this case **SID S-1-5...** this is usually the standard prefix in `Windows NT-based` environments such as `Samba`. So we can confirm that the user `cowboy` is a `Samba` user.<br>
We can also see another security identifier **SID S-1-22-1** and this a well-known **SID** for the "Unix Users". So with this we can confirm that in the system there are 2 users `cowboy` and `debian`.<br>
We can also see a list of the **file system** within SMB.

<br>
<br>

Users enumerated:
- **cowboy** &rarr; Samba and system.
- **debian** &rarr; System.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 3. Exploitation
With our new user discovered we proceed to find the password through a **dictionary attack**.
```bash
crackmapexec smb 192.168.0.110 -u cowboy -p /usr/share/wordlists/rockyou.txt
```
![[Sedition 4.png]]
![[Sedition 5.png]]

<br>

With the password in our hands we can already access to the `SMB` file system.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 4. Discovering of the zip file (post-exploitation)
First we check the file system to see if the information that we got before is correct.
![[Sedition 6.png]]

<br>

We check the 2 disks that the user has access to.<br>
In the `cowboy` disk we don't fine anything but hidden configuration files.
![[Sedition 7.png]]

<br>

But in the `backup` disk we find a secret `.zip` file.
![[Sedition 8.png]]

<br>

We download it into our system for analysis.
![[Sedition 9.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Cracking and analysis of the `.zip` file
We start by listing the files inside the `.zip` file
![[Sedition 10.png]]There's a file called `password` inside,

<br>

Now we proceed to see information about the file itself.
![[Sedition 11.png]]
As we can see the `.zip` file is encrypted. We need a password.

<br>

We use the tool `john` to help us find the password.
![[Sedition 12.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 6. Access as **user**
With our password in our control we can unlock the `.zip` file.
![[Sedition 13.png]]

<br>

And inside we find a password.
![[Sedition 14.png]]

<br>

We connect through `SSH` with our new password.
![[Sedition 15.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 7. Credentials discovering and decoding
After checking everything in the system we find an internal database running on port **3306**.
![[Sedition 16.png]]
We tried the same password that allowed us to log as the user and it worked. <br>
There was another way that we didn't discover until later that it was using the `history` command to check the previous commands executed by the user.
![[Sedition 27.png]]

<br>

Inside this database we discovered the hash for the other user in the system.
![[Sedition 17.png]]
![[Sedition 18.png]]

<br>

We confirm the algorithm used to hash it with **Haiti**
![[Sedition 19.png]]

<br>

Knowing the algorithm we can find an online decrypter to do the job for us
![[Sedition 20.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 8. Access as **user** 2
With our new password we log in as `debian`
![[Sedition 21.png]]

<br>

And in the `/home` folder of this user we find our first flag. <br>
<span style="color:cyan">User Flag</span>
![[Sedition 22.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 9. Privilege escalation
After checking the sudo permission for this user we find a vulnerable binary: `/sed`
![[Sedition 23.png]]

<br>

After searching in the website [GTFOBins](https://gtfobins.github.io/#tar) we discover the right command for that allow us to executed privilege escalation.
![[Sedition 24.png]]

<br>

And as we can see it's a success
![[Sedition 25.png]]

<br>

And finally we can go to the `/root` folder to find our last flag. <br>
<span style="color:red">Root Flag</span>
![[Sedition 26.png]]
***
