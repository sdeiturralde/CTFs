---
<p align="center" > <kbd> <img src="https://labs.thehackerslabs.com/static/uploads/machines/sedition.jpg" width=250px> </kbd> </p>

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
We use the tool `nmap` do a port scanning **without** 3-way-handshake and we save the results in an html file.
```bash
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 192.168.0.110 -oX nmap_TCP.xml && \
xsltproc nmap_TCP.xml -o Sedition_nmap_TCP.html && \
open Sedition_nmap_TCP.html &>/dev/null & \
disown
```

<img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2028.png"  height:500px >

<br>
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
Since we have a **Samba** server we can take advantage of that to enumerate users.
```bash
enum4linux -a 192.168.0.110
```
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%201.png"  height:500px > 
 <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%202.png"  height:500px > 
 <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%203.png"  height:500px > </kbd>

<br>

From this we can gather much information.<br>

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
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%204.png"  height:500px >
<img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%205.png"  height:500px > </kbd>

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
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%206.png"  height:500px > </kbd>

<br>

We check the 2 disks that the user has access to.<br>
In the `cowboy` disk we don't fine anything but hidden configuration files.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%207.png"  height:500px > </kbd>

<br>

But in the `backup` disk we find a secret `.zip` file.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%208.png"  height:500px > </kbd>

<br>

We download it into our system for analysis.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%209.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Cracking and analysis of the `.zip` file
We start by listing the files inside the `.zip` file
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2010.png"  height:500px > </kbd>
There's another file called `password` inside.

<br>

Now we proceed to see information about the file itself.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2011.png"  height:500px > </kbd>
As we can see the `.zip` file is encrypted. We need a password.

<br>

We use the tool `john` to help us find the password.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2012.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 6. Access as **user**
With the password in our control we can unlock the `.zip` file.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2013.png"  height:500px > </kbd>

<br>

And inside we find a password.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2014.png"  height:500px > </kbd>

<br>

We connect through `SSH` with our new password.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2015.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 7. Credentials discovering and decoding
After checking everything in the system we find an internal database running on port **3306**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2016.png"  height:500px > </kbd>
We tried the same password that allowed us to log as the user and it worked. <br>
There was another way that we didn't discover until later that it was using the `history` command to check the previous commands executed by the user.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2027.png"  height:500px > </kbd>

<br>

Inside this database we discovered the hash for the other user in the system.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2017.png"  height:500px > </kbd>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2018.png"  height:500px > </kbd>

<br>

We confirm the algorithm used to hash it with **Haiti**
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2019.png"  height:500px > </kbd>

<br>

Knowing the algorithm we can find an online decrypter to do the job for us
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2020.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 8. Access as **user** 2
With our new password we log in as `debian`
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2021.png"  height:500px > </kbd>

<br>

And in the `/home` folder of this user we find our first flag. <br>
User Flag:
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2022.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 9. Privilege escalation
After checking the sudo permission for this user we find a vulnerable binary: `/sed`
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2023.png"  height:500px > </kbd>

<br>

After searching in the website [GTFOBins](https://gtfobins.github.io/#tar) we discover the right command for that allow us to executed privilege escalation.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2024.png"  height:500px > </kbd>

<br>

And as we can see it's a success
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2025.png"  height:500px > </kbd>

<br>

And finally we can go to the `/root` folder to find our last flag. <br>
Root Flag
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Sedition%2026.png"  height:500px > </kbd>
***
