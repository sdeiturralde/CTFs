---
tags:
  - CTF
  - TheHackerLabs
  - Cybersecurity
Name: "[[Uploader]]"
System: Linux (Ubuntu)
Difficulty: Easy
Focus: Reverse shell upload, ZIP cracking, Escalation with /tar
---



<div class="os-banner">
  <div class="os-border"></div>
  <img src="https://labs.thehackerslabs.com/static/uploads/machines/uploader.png" 
       alt="systemd" 
       class="os-icon3">
  <div class="os-border"></div>
</div>

<br>

**Machine name**: Uploader<br>
**System**: Linux (Ubuntu)<br>
**Difficulty**: Easy<br>
**Focus**: Reverse shell upload, ZIP cracking, Escalation with `/tar`
***

<br>
<br>
<br>

## 1. Port Scanning
```bash wrap
nmap --stats-every=5s -p 0-65535 --open --min-rate=5000 -T5 -A -sT -Pn -n -v 172.17.140.41 -oX nmap_TCP.xml && xsltproc nmap_TCP.xml -o nmap_TCP.html && open nmap_TCP.html &>/dev/null & disown
```

<iframe src="file:///C:/Users/Skyveck/Documents/Documentos/Obsidian/Obsidian/02 - Studies/Security/CTFs/The Hacker Labs/Imgs/Uploader_nmap_TCP.html" style="width:100%; height:500px;"></iframe>

<br>

Detected services:
- **80** &rarr; Apache HTTP Server

***

<br>
<br>
<br>
<br>
<br>
<br>

## 2. Web enumeration
We use `gobuster` to identify important directories.
```bash wrap
gobuster dir -r -k -t 200 --no-error --add-slash -b 404 -x html,php,xml,json,css,js,txt,md,pdf,zip,rar,bak,old,jpg,jpeg,png,gif,db,sql,log -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 172.17.136.32
```

![[Uploader 1.png]]

<br>

We find a `.php` file and a folder where we have the uploaded files (`/uploads/`)
***

<br>
<br>
<br>
<br>
<br>
<br>

## 3. Exploitation
Thanks to the port scanning earlier we know that there's an HTTP server running on the machine, so when we access through the IP we find a website.
![[Uploader 2.png]]

<br>

Also thanks to the web enumeration we also know that there's a page called `upload.php` and we can confirm this on the website.
![[Uploader 3.png]]

<br>

Here we can upload a file and there's no sanitation of any kind so we were able to upload a `.php` file directly.
![[Uploader 6.png]]
![[Uploader 4.png]]

<br>

We also discovered in the web enumeration phase the folder where this uploads are being stored so we could find our file.
![[Uploader 5.png]]

<br>

Here we can just click on the file to execute it and get the reverse shell that we previously prepared successfully exploiting the vulnerability.
![[Uploader 7.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 4. Post-Exploitation enumeration
After we got in we go to the `/home` folder and we find our user **operatorx**.
<img width=760 src="./Imgs/Uploader 8.png" />

<br>

We also find a `Readme.txt` file that tell us about a file `.zip` that we must find.
![[Uploader 9.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 5. Discovering and analysis of the `.zip` file
First we find the file in the system.
```bash 
find / -type f -name *.zip 2>/dev/null
```
<img width=900 src="./Imgs/Uploader 10.png" />

<br>
Once we find it we download it into our system for analysis.<br>To do that we open an HTTP server with python

<img width=630 src="./Imgs/Uploader 11.png" />
For some reason it was not working very well for us but we were still able to reach the file through the browser.
<img width=720 src="./Imgs/Uploader 12.png" />

<br>

Now we list the files within the `.zip` and we try to extract them.
```bash
7z l File.zip
```
<img width=720 src="./Imgs/Uploader 13.png" />

We found a directory with a text file within.
<br>

```bash
7z x File.zip
```
![[Uploader 14.png]]
But at the moment of extracting it we discover that the `.zip` has a password.

<br>

We display detailed info about the file.
```bash
zipinfo -v File.zip
```
![[Uploader 15.png]]
![[Uploader 16.png]]We can see detailed information about both files and we can see confirm that the second one (`Credentials.txt`) is **encrypted**.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 6. Cracking the `.zip` file with John
```bash
zip2john File.zip > hash
```
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
<img width=720 src="./Imgs/Uploader 17.png" />
We already did it before the screenshot so the password got stored.<br>We have the password for `.zip` file.
***

<br>
<br>
<br>
<br>
<br>
<br>

## 7. Credentials decoding
Now we can finally decompress the `.zip` file.
<img width=720 src="./Imgs/Uploader 19.png" />
<img width=720 src="./Imgs/Uploader 20.png" />
And inside we find a `.txt` file with the credentials of the user **operatorx**. The password is encrypted.

<br>

With the help of `haiti` we find what is the algorithm used to encrypt it.<br>In this case it was **MD5**.
<img width=720 src="./Imgs/Uploader 21.png" />

<br>

And finally with an online tool found online we get the user password.
<img width=720 src="./Imgs/Uploader 22.png" />
***

<br>
<br>
<br>
<br>
<br>
<br>

## 8. Access as **user**
We use our new credentials to access as **operatorx**.
![[Uploader 23.png]]
<br>

Once in here we discover our flag.<br>
<span style="color:cyan">User Flag</span>
![[Uploader 24.png]]
***

<br>
<br>
<br>
<br>
<br>
<br>

## 9. Privilege escalation
We discovered with out new user a vulnerable binary.
```bash
sudo -l
```
![[Uploader 25.png]]

<br>

After searching in the website [GTFOBins](https://gtfobins.github.io/#tar) we discover the right command for that allow us to executed privilege escalation.
<img width=720 src="./Imgs/Uploader 26.png" />

<br>

We can see that is a success.
![[Uploader 27.png]]

<br>

And finally we can claim our last flag.<br>
<span style="color:red">Root Flag</span>
<img width=720 src="./Imgs/Uploader 28.png" />
***
