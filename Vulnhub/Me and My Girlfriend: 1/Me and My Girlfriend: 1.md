# Me and My Girlfriend: 1
Date release: 13 Dec 2019\
Author: TW1C3\
Difficulty Level: Beginner

## Recon
1. **Find the targets IP Address**\
Command: netdiscover -r 10.10.10.10/24\
![netdiscover](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/1.png "netdiscover")

2. **Scan the targets ports**\
Command: nmap -sC -sV 10.10.10.2\
-sC: Performs a script scan using the default set of scripts on each open port found.\
-sV: Enumerates the versions of the open ports.\
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
After going through users 1 - 12 these are the credentials that were found.
![creds](https://github.com/francobel/CTF-Writeups/blob/master/Vulnhub/Me%20and%20My%20Girlfriend:%201/Images/14.png "creds")\


