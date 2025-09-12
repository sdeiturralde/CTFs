---

 <p align="center"> <kbd> <img src="https://labs.thehackerslabs.com/static/uploads/machines/tortuga.png" width=250px > </kbd> </p>

<br>

**Machine name**: Tortuga<br>
**System**: Linux(Debian) <br>
**Difficulty**: Easy<br>
**Focus**: SSH dictionary attack, Horizontal escalation, Escalation through python3 capabilities
***

<br>
<br>
<br>

## 1. Port Scanning
```bash
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 172.17.143.186 -oX nmap_TCP.xml && \
xsltproc nmap_TCP.xml -o Tortuga_nmap_TCP.html && \
open Tortuga_nmap_TCP.html &>/dev/null & \
disown
```
<img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2017.png" width=1200px >

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
```bash
gobuster dir -r -k -t 200 --no-error --add-slash -b 404 -x html,php,xml,json,css,js,txt,md,pdf,zip,rar,bak,old,jpg,jpeg,png,gif,db,sql,log \ 
-w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \ 
-u 172.17.143.186
```
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%201.png" width=1200px >
<img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%202.png" width=1200px > </kbd>

<br>

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
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%203.png" width=900px > </kbd>
The main website already tell us about secrets.
Here it says:<br>
<br>
"Welcome to Turtle Island. <br>
The corsair refuge. <br>
Here your sailing begins, sailor... Explore the corners of the island and find what is hidden in their shores." <br> <br>
And there are two links.

<br>
<br>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%204.png" width=900px > </kbd>
Then we find a map giving us a hint.<br>
Here it says: <br> <br>
"A wrinkled note that you can read in a corner... <br>
'Hey grumete (cabin boy) check the hidden note that i've left in your cabin.' " <br>
<br>

It tell us about a **port** that is a reference to one of the service to enter in the pirate bay (`the machine`) and a buried treasure in between.
It also call us **`grumete`** and give us a reminder of a **hidden note**.

<br>

<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%205.png" width=1200px > </kbd>
<br>

Then we have a list of the tribulation and we can confirm that **`grumete`** is part of the crew.

<br>
<br>

We also know that the machine has a `SSH` running and we are already aware of an user **`grumete`** so we try a **dictionary attack**. <br>

<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%206.png" width=1200px > </kbd>
<br>

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
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%207.png" width=1200px > </kbd>

<br>

And once we are inside we find our first flag. <br>
User Flag
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%208.png" width=1200px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Horizontal escalation
We remember the advice that the captain left us in the website and we do a thorough search for hidden files and we found the **secret note**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%209.png" width=1200px > </kbd>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2010.png" width=1200px > </kbd>
<br>
<br>
The content of this note is that the "captain" leaves the bote and trust our cabin boy with the password of the wheel.<br>
Hidden in this note we find a new credential for us to use and we already know the users in the folder **`/home`** <br> <br>

<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2016.png" width=1200px > </kbd>

<br>

With our new credentials we access as the user **`captain`** <br>

<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2011.png" width=700px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 6. Privilege escalation
We proceed to check everywhere for anything out of place and we found it in the **capabilities**. <br>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2012.png" width=1200px > </kbd>
This reveals that the binary `python3.11` has the **capability** **`cap_setuid=ep`** that allows it to change the **UID** process.<br>
This means that Python can execute **`setuid(0)`** and become `root`.<br>
<br>

We search in the website [GTFOBins](https://gtfobins.github.io/#tar) and we find the correct command to exploit the vulnerability.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2013.png" width=1200px > </kbd>

<br>

And after adapting it a lil bit to match our binary we can see that is a sucess.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2014.png" width=1200px > </kbd>

<br>

Finally we find our last flag.<br>
Root Flag<br>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Tortuga%2015.png" width=700px > </kbd>>
***
