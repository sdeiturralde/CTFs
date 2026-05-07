---
<p align="center" > <kbd> <img src="https://labs.thehackerslabs.com/static/uploads/machines/Nike.png" width=250px> </kbd> </p>

<br>

**Machine name**: Nike<br>
**System**: Linux(Debian)<br>
**Difficulty**: Medium<br>
**Focus**: XXE exploitation, Escaping restricted shell, Horizontal escalation with `/java`, `/python3`, `/logrotate` and `/dd`, Privilege escalation with SUID custom binary
***

<br>
<br>
<br>

## 1. Port Scanning

We use the tool `nmap` do a port scanning **without** 3-way-handshake and we save the results in an html file.
```bash
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sS -Pn -n -v 192.168.0.110 -oX nmap_TCP.xml && \
xsltproc nmap_TCP.xml -o Nike_nmap_TCP.html && \
open Nike_nmap_TCP.html &>/dev/null & \
disown
```

<p align="center" > <kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2025.png" width=900px > </kbd> </p>

<br>

Detected services:
- **22** &rarr; SSH
- **80** &rarr; Apache HTTP Server
***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. Web enumeration
We know we have an **http** service so we checked it and we found a website that simulates a **nike store**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%201.png"  height:500px > </kbd>

As usual we proceed with a web enumeration and we find 2 hidden files **`upload.php`** and **`datos.php`**. We noticed that we have a **500** response with the latter so that's an error in the server. We can't access this file through the web.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%205.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 3. Expoitation
We focused con the **`upload.php`** file and we discover an empty file that is lacking any **xml** structure and it is expecting one.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%202.png"  height:500px > </kbd>

We try a **raw XXE** attack with **`curl`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%203.png"  height:500px > </kbd>
And as we can see we have access to the system and we discover **5** users: **`mike`**, **`n`**, **`pylon`**, **`macci`** and **`wvverez`**.

We remember that we also have a file with **500** code that it was forbidden to us called **`datos.php`** and with the same vulnerability we are now able to read it.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%204.png"  height:500px > </kbd>

This is apparently a file with all the **passwords** of the system.
But after trying each of them on the **ssh** we discover that only works the password for the user **`mike`**.

But once inside we discover a problem we are in a **restrict shell** we can do barely any command and our privileges are very little. we also didn't have any commands in our **PATH** it was empty.<br>
To try to escape that we closed the **ssh** and added **bash** to try to use a different **shell**. It didn't allow us to escape but allowed us to use the command **cd** and **sudo** where we discovered a **sudoers** permissions with **`java`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%206.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 4. Horizontal escalation
We can execute the binary **java** as the user **`n`**. To do that we create a malicious script in **java** in our local device.<br><br>
In the script we a main method that creates an **operating system subprocess** to open a **bash shell** with **`stdin`**, **`stdout`** and **`stderr`** and wait for the process (in this case the **shell**) to end.
<br>

**-> Opening a terminal with Java:**
```java wrap
public class Exploit {
    public static void main(String[] args) {
        try {
            Process p = new ProcessBuilder("/bin/bash").inheritIO().start();
            p.waitFor();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Once we have our script we download it in the vulnerable machine with **`wget`** and we execute it as the user **`n`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%207.png"  height:500px > </kbd>

Once as the user **`n`** we sanitized the terminal to work better and we checked the **sudoers** permits once again and we discover that we can execute the binary **python** and a **script** as the user **`pylon`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%208.png"  height:500px > </kbd>

We open the script with **nano** (that is just a simple script to add two numbers) and we some **lines** to open a **bash shell** with **python**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%209.png"  height:500px > </kbd>

We execute the **binary** and the modified script as the user **`pylon`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2010.png"  height:500px > </kbd>

We checked once again for the **sudoers** permits we once again found another one. This time we can execute **logrotate** as the user **`macci`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2011.png"  height:500px > </kbd>

This binary is designed to automate the management of **log files**. Its main purpose is to save you from having to manually handle **log files** that can grow indefinitely and potentially fill up your hard drive.<br><br>
We will use this to create a **reverse shell**. First we create a **`.log`** file larger than 1 **kilobyte** to ensure that **logrotate** checks this file. We do this reading **2000** bytes from the **kernel's** random number generator (**`/dev/urandom`**).
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2012.png"  height:500px > </kbd>

Then we make a **malicious** configuration file for **logrotate**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2013.png"  height:500px > </kbd>

This makes sure that the our **`.log`** is rotate daily (although this is overwritten by the size) and then we stablish the size to make logrotate rotate the file when it exceeds **1 kilobyte**. Then we create a command to run before that rotation. And after that we create a new **pipe** file to connect the **bash shell** that we want to run with our **nc** command.<br><br>
The command **cat** reads whatever is in the **pipe** file and passes it to the **bash shell**. Then the output that arrives through the **nc** is redirected to the **pipe** file.

And finally we just execute **logrotate** with a custom **state file** (where logrotate keeps track of when it last rotated each log) and our **malicious configuration** as the user **`macci`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2014.png"  height:500px >
<img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2015.png"  height:500px > </kbd>


Once inside as the user **`macci`** we discover that we have once again **sudoers** permits. This time is the binary **dd** that we can execute as the user **`wvverez`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2016.png"  height:500px > </kbd>

This binary is used for low-level copying and conversion of data. But it can also be used to **read** files.

And this is exactly what we do. Executing as the user **`wvverez`** we could read the **private ssh** key stored in the hidden folder **`/.ssh`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2017.png"  height:500px > </kbd>

Copying it into our system and using **ssh** to connect as the user gave us finally our first flag.
<span style="color:cyan">User Flag</span>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2018.png"  height:500px > </kbd>
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Privilege escalation
With this final user we didn't have any **sudoers** permits but we noticed something curious. The user was in another group aside from his **default** group. A group called **`ctf_admins`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2019.png"  height:500px > </kbd>

And this group also had a **SUID** custom binary called **`sys_monitor`**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2020.png"  height:500px > </kbd>

Investigating a bit we discover that this binary has **3** functions and it accepts an **input**.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2021.png"  height:500px > </kbd>

Investigating a bit more with **strings** we discover a bit more about how it works. We know for certain that the **first** function uses **ls** on the input that we gave it to it.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2022.png"  height:500px > </kbd>

With this info we create a test file where we discovered that the **second** function uses **cat** to read the input and the **third** one execute it. With this in mind we created a simple **bash script** that opens a terminal and we try it to execute it with the **3rd** function.
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2023.png"  height:500px > </kbd>

And we can see it worked. We now have our final flag.<br><span style="color:red">Root Flag</span>
<kbd> <img src="https://github.com/sdeiturralde/CTFs/blob/main/Imgs/Nike%2024.png"  height:500px > </kbd>
***

