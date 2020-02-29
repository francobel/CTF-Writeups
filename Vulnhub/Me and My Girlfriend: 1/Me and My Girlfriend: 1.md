# [Me and My Girlfriend: 1](https://www.vulnhub.com/entry/me-and-my-girlfriend-1,409/)
Date release: 13 Dec 2019\
Author: TW1C3\
Difficulty Level: Beginner

## Recon
1. **Find the targets IP Address**\
**Command: netdiscover -r 10.10.10.10/24**\
![netdiscover](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/1.png "netdiscover")

2. **Scan the targets ports**\
**Command: nmap -sC -sV 10.10.10.2**\
**-sC**: Performs a script scan using the default set of scripts on each open port found.\
**-sV**: Enumerates the versions of the open ports.\
![nmap](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/2.png "nmap")\
Ports 22(SSH) and 80(HTTP) are open.

## Threat Modeling
1. **Analyzing port 80**\
We use a web browser to analyze the website the target is running.\
![http](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/3.png "http")\
There seems to be nothing on the site but we get a big hint, the website is only accessible localy.

2. **Spoofing as local IP**\
First we use Burpsuite to intercept our GET request to 10.10.10.2:80.\
We send our intercepted request to burp's repeater and add the X-Forwarded-For header.\
After we add the header we can see that the response is trying to redirect us to a site ending in: ?page=index\
![burp](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/4.png "burp")\
The **X-Forwarded-For** header lets us pretend that our request is coming from the targets local network, giving us access to the real site.\
\
We use burp again to create a custom GET request to the site **10.10.10.2/?page=index** with the X-Forwarded-For:127.0.0.1 header.\
![burp](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/5.png "burp")

3. **Enumeration**\
Now we have access to the real site.\
![http](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/6.png "http")\
From here on out I will be intercepting every GET request I make and manually add the X-Forwarded-For header to my requests. Without it you can't navigate the site.\
\
The first thing I do is register an account and log in.\
**Register:**\
![reg](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/8.png "reg")\
\
**Log in:**\
![login](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/9.png "login")\
Now that we're logged in we have 3 menu options, Dashboard, Profile, and Logout. Dashboard doesn't give us any useful info and logout wont either. It looks like the profile option lets you edit your accounts credentials.\
![profile](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/11.png "profile")\
If you pay attention to the URL it looks like our user ID is #13.\
\
If we edit this number and send the request it gives us access to another user's login credentials.\
![profile](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/12.png "profile")\
\
I sent the request to burp's repeater to speed up the process of enumerating all of the user's credentials.\
![bur](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/13.png "burp")\
On the response's HTML you can see the values for the username and password.\
\
After going through users 1 - 12 these are the credentials that were found.\
![creds](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/14.png "creds")

## Exploitation
1. **Getting user shell**\
Since port 22(SSH) was open, I tried all of the credentials to log into the target's machine through SSH.\
Only the account with the credentials **alice:4lic3** allowed us to log in.\
**Command: ssh alice@10.10.10.2**\
![ssh](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/15.png "ssh")\
Now we have a user shell on the target!\
\
After listing the files in alice's home directory I navigate to her "my_secret" folder and get the first flag.\
![flag1](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/20.png "flag1")

2. **Enumerating Target Machine**\
After getting a user shell, the first thing I do in my enumartion process is check the root priviledges of the user I'm logged in as with the following command.\
**Command: sudo -l**\
**-l**: lists the commands the user has root priviledges with.\
![sudo](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/16.png "sudo")\
The second to last line tells us the user has root access to the "php" command with no need for a password.\
Now we have enough info to get a root shell on the target's machine.

3. **Exploitation / Priviledge Escalation**\
Since I know alice has root privs with php, any php code I create can be executed with root priviledges.\
I wrote a quick php script that will change the user I'm logged in as from alice to root.
![su](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/17.png "su")\
\
And executed it\
**Command: sudo php exploit.php**\
![change](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/18.png "root")\
Now we are logged in as **root**.\
\
The final step is to navigate to the root directory and read the final flag.\
**Command: cat flag2.txt > $(tty)**\
![x](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/19.png "flag2")\
I had to redirect cat's output to tty because it wasn't showing up without it. I'm assuming it was to add a level of difficulty.
