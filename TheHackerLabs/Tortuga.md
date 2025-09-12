---
tags:
  - CTF
  - Cybersecurity
  - TheHackerLabs
Name: "[[Tortuga]]"
System: Linux (Debian)
Difficulty: Easy
Focus: SSH dictionary attack, horizontal escalation, Escalation through python3 capabilities
---


<div class="os-banner">
  <div class="os-border"></div>
  <img src="https://labs.thehackerslabs.com/static/uploads/machines/tortuga.png" 
       alt="systemd" 
       class="os-icon3">
  <div class="os-border"></div>
</div>

<br>

**Machine name**: Tortuga<br>
**System**: Linux(Debian)
**Difficulty**: Easy<br>
**Focus**: SSH dictionary attack, horizontal escalation, Escalation through python3 capabilities
***

<br>
<br>
<br>

## 1. Port Scanning
```bash wrap
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 172.17.143.186 -oX nmap_TCP.xml && xsltproc nmap_TCP.xml -o Tortuga_nmap_TCP.html && open Tortuga_nmap_TCP.html &>/dev/null & disown
```
<iframe src="file:///C:/Users/Skyveck/Documents/Documentos/Obsidian/Obsidian/02 - Studies/Security/CTFs/The Hacker Labs/Imgs/Tortuga_nmap_TCP.html" style="width:100%; height:500px;"></iframe>

<br>

Detected services:
- **22** &rarr; SSH
- **80** &rarr; Apache http server
***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. Web enumeration
```bash wrap
gobuster dir -r -k -t 200 --no-error --add-slash -b 404 -x html,php,xml,json,css,js,txt,md,pdf,zip,rar,bak,old,jpg,jpeg,png,gif,db,sql,log -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 172.17.143.186
```
![[Tortuga 1.png]]
<img width=1000 src="./Imgs/Tortuga 2.png" />
We tried a web enumeration but we only discover a normal file that we can access through the main website **`/mapa.php/`**.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 3. Exploitation
We know already about the existence of an `Apache` server thanks to the previos scans. So we access to it and we found a lot of clues.
<img width=720 src="./Imgs/Tortuga 3.png" />
The main website already tell us about secrets.
<br>
<br>
<img width=720 src="./Imgs/Tortuga 4.png" />
Then we find a map giving us a hint.<br>
It tell us about a **port** that is a reference to one of the service to enter in the pirate bay (`the machine`) and a buried treasure in between.
It also call us **`grumete`** and give us a reminder of a **hidden note**.

<br>

<img width=720 src="./Imgs/Tortuga 5.png" />
Then we have a list of the tribulation and we can confirm that **`grumete`** is part of the crew.

<br>
<br>

We also know that the machine has a `SSH` running and we are already aware of an user **`grumete`** so we try a **dictionary attack**.
<img width=720 src="./Imgs/Tortuga 6.png" />
And as we can see it is a success. We found a way in.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 4. Access as **user**
With our new credentials we use **SSH** to access into the machine.
<img width=720 src="./Imgs/Tortuga 7.png" />

<br>

And once we are inside we find our first flag.
<span style="color:cyan">User Flag</span>
<img width=920 src="./Imgs/Tortuga 8.png" />
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Horizontal escalation
We remember the advice that the captain left us in the website and we do a thorough search for hidden files and we found the **secret note**.
<img width=920 src="./Imgs/Tortuga 9.png" />
<img width=920 src="./Imgs/Tortuga 10.png" />

Hidden in this note we find a new credential for us to use and we already know the users in the folder **`/home`**
<img width=920 src="./Imgs/Tortuga 16.png" />

<br>

With our new credentials we access as the user **`captian`**
<img width=580 src="./Imgs/Tortuga 11.png" />
***

<br>
<br>
<br>
<br>
<br>
<br>

## 6. Privilege escalation
We proceed to check everywhere for anything out of place and we found it in the **capabilities**.
<img width=920 src="./Imgs/Tortuga 12.png" />
This reveals that the binary `python3.11` has the **capability** **`cap_setuid=ep`** that allows it to change the **UID** process.<br>
This means that Python can execute **`setuid(0)`** and become `root`.<br>
<br>

We search in the website [GTFOBins](https://gtfobins.github.io/#tar) and we find the correct command to exploit the vulnerability.
<img width=720 src="./Imgs/Tortuga 13.png" />

<br>

And after adapting it a lil bit to match our binary we can see that is a sucess.
<img width=920 src="./Imgs/Tortuga 14.png" />

<br>

Finally we find our last flag.<br>
<span style="color:red">Root Flag</span>
<img width=680 src="./Imgs/Tortuga 15.png" />
***
