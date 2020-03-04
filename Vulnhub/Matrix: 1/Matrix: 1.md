# [Matrix: 1](https://www.vulnhub.com/entry/matrix-1,259/)
Date release: 19 Aug 2018\
Author: Ajay Verma\
Difficulty Level: Intermediate

## Recon
1. **Find the targets IP Address**\
Command: **netdiscover -r 10.10.10.0/24**\
![netdiscover](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/1.png "netdiscover")

2. **Scan the targets ports**\
Command: **nmap -sC -sV 10.10.10.5**\
**-sC**: Performs a script scan using the default set of scripts on each open port found.\
**-sV**: Enumerates the versions of the open ports.\
![nmap](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/2.png "nmap")\
Ports 22(SSH), 80(HTTP), and 31337(HTTP) are open.
Port 31337 isn't usually used for HTTP so there might be something interesting there.

## Threat Modeling
1. **Analyzing port 80**\
We use a web browser to analyze the website the target is running.\
![http](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/3.png "http")\
The header on the site says "Follow the White Rabbit" and at the bottom of the page we see a white rabbit. Let's open that image in a new tab and see what we find.\
\
![rabbit](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/4.png "rabbit")\
From the URL we can see that the title of the image is "p0rt_31337". Let's "follow" that port since it's what the website instructed us.

2. **Analyzing port 31337**\
This site looks similar to the last one but it doesn't give us any more useful information.\
![31337](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/5.png "31337")\
\
If we analyze the site's source code line 71 stands out. It looks like base64 that we can decode.\
![base64](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/6.png "base64")\
\
When we decode the base64 we get a message that says:\
echo "Then you'll see, that it is not the spoon that bends, it is only yourself. " > Cypher.matrix\
![b64](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/7.png "b64")\
This gives us a hint that a file named Cypher.matrix was created in the server hosting the site.\
\
If we enter the URL, "10.10.10.5:31337/Cypher.matrix" into the web browser we get a file download.\
![dl](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/8.png "dl")\
Lets see what's in there\
\
When we open this file in a text editor it looks like nothing.\
What it really is, it's code written in [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck).
![bf](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/9.png "bf")\
\
When we enter this code into a Brainfuck decoder we get the following message:\
![bfdecoded](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/10.png "bfdecoded")\
Now we know we can enter the matrix with the username guest, and the password k1ll0rXX. Where the XX's could be any character.

3. **Bruteforce**\
I created a wordlist using python that we'll use to bruteforce the password. The list will contain every permutation of k1ll0rXX with every possible characters instead of the XX's.\
This is the script that created the list.\
![python](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/11.png "python")\
Output:\
![wl](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/12.png "wl")\
\
Now that we have the wordlist with the password within it, we use hydra to bruteforce the targets SSH port.\
Command: **hydra -l guest -P wordlist.txt ssh://10.10.10.5 -t 50**\
**-l**: login: guest\
**-P**: password list: wordlist.txt\
**-t**: threads utilized: 50\
![wl](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/13.png "wl")\
Hydra ran successfully, if you look at the second to last line you can see the login credentials.\
**login: guest password:k1ll0r7n**\
\
Now we have a user shell in the target's server.\
![sh](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/14.png "sh")\

## Exploitation
1. **Breaking out of the shell**\
The first thing I notice inside the server is that I'm restricted from using even the most basic commands.\
cd, cat, ls are all restricted, luckily 'echo' isn't.\
We use echo to check what our PATH is. In this case our path leads to /home/guest/prog.\
Our path dictates the folder that contains the executables we can use.\
Using the command, "$PATH/*" we see what the only executable we're allowed to use is vi.\
We use the command, "$SHELL" and we verify we're in a restricted bash shell. 
![sh2](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/15.png "sh2")\
\
Since we're allowed to use vi, I exploit it by using it to spawn a shell with the command "!/bin/bash"\
![sh2](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/16.png "sh2")\
\
Now we've successfully broken out the restricted shell.\
![x](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/x.png "x")\

2. **Exploitation/Priviledge Escalation**\
The first thing I do is change my path to a more generic one to have access to the regular commands.\
Command: **export PATH=/usr/bin:/bin**\
Next I check the user's root priviledges with the following command.\
Command: **sudo -l**\
**-l**: lists the commands the user has root priviledges with.\
With that we find out that this user has access to all commands with root priviledges.\
A simple "sudo su" turns our user from "guest" to "root".\
![root](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/17.png "root")\
\
Now we just navigate the root's home folder and capture the flag.\
![flag](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Matrix:%201/Images/18.png "flag")\










