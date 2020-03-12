# [Toppo: 1](https://www.vulnhub.com/entry/toppo-1,245/)
Date release: 12 Jul 2018\
Author: @h4d3sw0rm\
Difficulty Level: Beginner

## Recon
1. **Scan the targets ports**\
Command: **nmap -sC -sV 10.10.10.8**\
**-sC**: Performs a script scan using the default set of scripts on each open port found.\
**-sV**: Enumerates the versions of the open ports.\
![nmap](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/1.png "nmap")\
Ports 22(SSH), 80(HTTP), and 111(rcpbind) are open.

## Threat Modeling
1. **Analyzing port 80**\
We use a web browser to analyze the website the target is running.\
The website doesn't look like anything special, needs further analyzing.\
![http](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/2.png "http")

2. **Dirbuster**\
We review the web server further by checking for vulnerable directories.\
Dirbuster allows us to bruteforce the directory structure using a wordlist made for finding directories.\
![dirb](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/3.png "dirb")

3. **Results**\
The second to last entry in the "Scan Information" tab reveals that in the web server there exists an "admin" folder.\
Let's see what it leads to\
![admin](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/4.png "admin")\
\
Once we enter "http://10.10.10.8/admin/" in our web browser we find out that within the admin folder there's a text file named notes.txt\
![txt](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/5.png "txt")\
\
The contents of the file seem to be someone leaving a not for themselves. Within their note they reveal their password, "12345ted234".\
![ted](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/6.png "ted")

## Exploitation
1. **SSH**\
With that information I assume the admin's username is ted. Now we know enough to successfully log into the server through SSH.\
![ssh](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/7.png "ssh")

2. **SUID**\
A search for all the executables with SUID bits reveals the following list of programs.\
(Programs with SUID bits let you execute them as a different user, specifically its owner.)\
Python sticks out to me because you don't normally see it with a set SUID bit.\
![suid](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/8.png "suid")\
\
Now we check who owns python's executable. We see that it's root.\
![own](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/11.png "own")

3. **Priviledge Escalation \ Exploitation**\
With that we have enough information to root the box.\
We spawn a root shell using python. This gives us a root shell because when we execute a command with python it's as if root was executing it due to the SUID bit.\
Command: **python2.7 -c 'import pty; pty.spawn("/bin/sh")'**
![root](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/11.png "root")\
\
Now all that is left is to go to the root folder and capture the flag.\
![flag](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Toppo:%201/Images/10.png "flag")



