## Shoppy - HackTheBox

<img width="414" alt="image" src="https://user-images.githubusercontent.com/114961392/201800750-2d1ce52a-d610-4c1e-a2c4-458b6b173249.png">

Machine: Shoppy  
Operating System: Linux  
Release Date: 17/09/2022  
Difficulty: Easy  

### Reconnaissance
Let's scan all ports with nmap.  
**nmap -p- --min-rate=10000 10.10.11.180**  

<img width="403" alt="image" src="https://user-images.githubusercontent.com/114961392/201801532-8199ac48-c46a-4da4-b71b-051b0f1c519b.png">

This shows 3 ports are open, port 22 (SSH), port 80 (HTTP), and port 9093 (copycat). Let's do a service and script scan on the open ports.  
**nmap -p22,80,9093 -sV -sC 10.10.11.180**

<img width="543" alt="image" src="https://user-images.githubusercontent.com/114961392/201801805-2405ddbd-ebb8-463a-8928-858d8bfcce73.png">

This shows that there is a redirect to shoppy.htb. Let's add shoppy.htb to the /etc/hosts file.  
Now we can take a look at the website. The website has very little information and no links. The source code does not show anything interesting either.  
Let's use dirsearch to enumerate any directories.

<img width="469" alt="image" src="https://user-images.githubusercontent.com/114961392/201807990-1e02b4e1-3607-4602-a47f-f3cb16e58fb9.png">

A few directories come up however it looks like most redirect to the /login page. This looks intersting so let's check it out in the next stage.  
Before moving onto the exploitation stage, let's search for any subdomains using FFUF.  

<img width="698" alt="image" src="https://user-images.githubusercontent.com/114961392/201810639-fde9e0d2-f55a-4cb3-9a89-74f4c5f2c92c.png">

This shows that the subdomain **mattermost** is active. I will add this to my /etc/hosts file and check out the page. It is another login page for mattermost. Doing a quick google search for mattermost tells us that Mattermost is an open-source, self-hostable online chat service with file sharing, search, and integrations. It is designed as an internal chat for organisations and companies, and mostly markets itself as an open-source alternative to Slack and Microsoft Teams.  
Alright let's start exploiting!  

## Exploitation

Going back to the shoppy.htb login page, I tried some basic username and password combinations however no luck. Next I will try some SQL injection techniques such as **' or 1=1#**. This seemed to do something on the backend as it hung for a long time before giving me an error page.  
After doing some research and checking https://book.hacktricks.xyz, I realised it might be running a NoSQL service such as Mongo. After playing around with some NoSQL injections from [hacktricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection), I crafted the payload **admin'||'1==1** and it worked!

<img width="218" alt="image" src="https://user-images.githubusercontent.com/114961392/201811384-adf3d20a-aa20-43c0-bca2-970971d6bdf8.png">

We are now presented with the admin page for shoppy.htb. The only interesting section here is the search bar. Let's see what happens if I enter the same payload as before.  
It worked again and shows a list of users with hashed passwords!  

<img width="250" alt="image" src="https://user-images.githubusercontent.com/114961392/201811824-87ee1736-43fc-4a99-8fc5-67f9b11a8038.png">

I will save the hash for user josh in a file and crack with hashcat against the rockyou password list.  
**hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -o result.txt**  

<img width="711" alt="image" src="https://user-images.githubusercontent.com/114961392/201812572-f791f278-9d8e-4a5f-ba1b-64e4e8a65cec.png">

This cracks the password which is **remembermethisway**. Now let's use these credentials at the mattermost site. It works and we are in.  
Looking at the chat history there are a few interesting points. Under the Development section there is a conversation where josh is discussing **docker, password manager, and a deploy machine**.
I will take note of this for future reference.  

<img width="722" alt="image" src="https://user-images.githubusercontent.com/114961392/201814897-6e4c726a-9fa4-4cb4-81bc-ed911f429c11.png">

In the next section, there are some credentials for the deploy machine. I will make note of these and try to SSH into the machine.  

<img width="307" alt="image" src="https://user-images.githubusercontent.com/114961392/201815301-568c94d8-8532-4fe3-b8c7-c833ab3b0957.png">

The credentials work and now we have access to jaeger account. From here I can go ahead and capture user flag.  

<img width="569" alt="image" src="https://user-images.githubusercontent.com/114961392/201815502-163f3024-d384-4a7b-845e-858e05018834.png">

## Privilege Escalation

Since we already have the password for this account, we can try the **sudo -l** technique to escalate privileges.  

<img width="541" alt="image" src="https://user-images.githubusercontent.com/114961392/201816009-f3d2eb88-37a9-41fe-a085-f179a52d6bf1.png">

This shows that the user is allowed to run **/home/deploy/password-manager**. Using cat to look at the file, its heavily encrypted however if you scroll down there is some clear text which tells you the master password, **Sample**.  

<img width="832" alt="image" src="https://user-images.githubusercontent.com/114961392/201816761-3e651408-15c6-4ab3-8745-79e53d6b4dde.png">

Now let's run the program with sudo and enter the master password when requested.  

<img width="314" alt="image" src="https://user-images.githubusercontent.com/114961392/201816957-becd1f03-00b6-4e4b-8869-b991fe5c73b6.png">

This displays credentials for the deploy machine. I will take note of these and SSH in with these credentials.  
Once we are in, I will upload linpeas to find the next vulnerability. Straight away we can see a docker vulnerability.  

<img width="828" alt="image" src="https://user-images.githubusercontent.com/114961392/201817277-e2b0ba15-6bee-46e7-998b-fbac54cbb2a3.png">

Let's search [GTFOBins](https://gtfobins.github.io/) for docker exploits. 

<img width="616" alt="image" src="https://user-images.githubusercontent.com/114961392/201817536-22046c0a-b9ac-4e80-832f-532bd3336e0e.png">

This looks good so let's try and see what happens.  

<img width="604" alt="image" src="https://user-images.githubusercontent.com/114961392/201817678-cede1115-f5ac-437d-895d-ea4134802492.png">

WOOSHKAH! We have root. From here we can go ahead and capture the root flag!  

<img width="169" alt="image" src="https://user-images.githubusercontent.com/114961392/201817795-1118b916-5c8c-42b8-9875-4ea77e33dfd9.png">
