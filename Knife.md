# HTB Knife

Difficulty - Easy

Machine Status - Retired

I decided to do this retired machine first as it looks nice and easy and is a good place to start. Plus it has a cool name!

## Enumeration

First of all, I started scanning all ports with Nmap. I like using the -A -T4 flags to start with and if they produce nothing then I dig deeper. We have ports 22 (SSH) and 80 (HTTP) open. Port 80 is running Apache httpd 2.4.41. I did a quick google search for any good exploits for this version however couldn’t find anything that interesting.

https://httpd.apache.org/security/vulnerabilities_24.html

<img width="516" alt="9abc324a0460427395664d2bfccff1c4" src="https://user-images.githubusercontent.com/114961392/194970418-37749233-5f95-4bd4-a569-2986d1d96224.png">

Next, I fired up Burp Suite and went to the web page. After probing around I didn’t find anything interesting as there are no links on the page. I decided to probe using dirsearch however there was nothing of great interest. 

img

I went back to Burp Suite to check out the requests and responses. I did a google search for exploits on the PHP version PHP/8.1.0-dev and found something useful. I found an interesting exploit on exploit-db with a link to a GitHub page. 

https://www.exploit-db.com/exploits/49933
https://github.com/flast101/php-8.1.0-dev-backdoor-rce

After using git clone to copy the exploits from GitHub, I decided to run the reverse shell exploit.
First, I set up a netcat listener on port 4444, ran the exploits help menu for correct syntax, then ran the exploit.

img

This successfully gained entry into the system under the user “James”. Use cat to expose the user flag.

Now that we are in the system, let’s try to escalate our privileges. 

Typing sudo -l reveals that we can run a binary called knife from root. Let’s run the binary and do a google search to find out what it is. After doing some research I discovered that it is a command-line tool that provides an interface between a local chef-repo and the Chef Infra Server. After digging much deeper for an exploit, I found out that knife has a function called exec which can run ruby scripts. After doing some more digging around on google I eventually found some interesting code on GTFOBins which should be useful. 

I ran the sudo command and BOOM! We have root. Now we can capture the root flag.




