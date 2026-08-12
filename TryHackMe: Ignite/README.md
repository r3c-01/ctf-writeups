# Ignite Write-Up

This is an easy level CTF called Ignite. It specifically says "A new startup has a few issues with their web server." And we are tasked with getting two flags (user.txt and root.txt).

I started with an nmap scan to see which ports are available given the IP. This is a web application CTF, so I know there will be an http or https port open at the least.

After the scan we see that port 80 is the only port open. With the aggressive scan we can also see that the http title says "Welcome to Fuel CMS". This is a content management system and I know that there are vulnerabilities present on some of these systems from prior CTF challenges. Additionally, there is a robots.txt which disallows the directory /fuel. So we will check both of these out. 

I also ran a gobuster scan to see if there were any other directories of value; however, I did not find anything of importance.

Looking at the website we see that this is powered by Fuel CMS 1.4. And there is quite a lot of information on the website. There are admin login credentials for the fuel directory given. They are: admin and admin. So I went to test these out to see if they weren't changed. And sure enough, they worked. After looking into the admin panel I found a few things that could be valuable. There was an assets page where you could upload a file and there was a page that showed the permissions. This immediately made me think of a file upload RCE, so maybe we could get a web shell this way.

Before I went any deeper I decided to check searchsploit to see if there were any known vulnerabilities in Fuel CMS version 1.4. And I found that the CMS was vulnerable to remote code execution or RCE. Kali linux gives you a database of working exploits so I decided to use the python code available for the RCE. Make sure that you add the url to the end.

After executing the python code, we have a web shell. 
