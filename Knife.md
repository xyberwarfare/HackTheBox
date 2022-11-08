# Knife - HackTheBox Walkthrough

<img width="424" alt="image" src="https://user-images.githubusercontent.com/114961392/200475129-e59ba3db-5fa8-4134-9159-afee40f5d8e4.png">

Machine: Knife  
Operating System: Linux  
Release Date: 22/05/2021  
Difficulty: Easy

## Enumeration

Let' start by scanning all ports with nmap. We have ports 22 (SSH) and 80 (HTTP) open. Port 80 is running Apache httpd 2.4.41. I did a quick google search for any good exploits for this version however couldn’t find anything that interesting.  
https://httpd.apache.org/security/vulnerabilities_24.html

nmap -p- --min-rate=1000 -sV -sC 10.10.10.242

<img width="482" alt="image" src="https://user-images.githubusercontent.com/114961392/200475737-6a33c11b-eee0-4acd-8940-edc921451eaa.png">

Next, I decided to probe using dirsearch however there was nothing of great interest. 

<img width="477" alt="image" src="https://user-images.githubusercontent.com/114961392/200478733-667c32d5-ccc8-43fc-a6cd-cfa9a28434a0.png">

I checked the page source code however I still could not find anything that interesting. Let's fire up Burp Suite to inspect the requests and responses. This shows that the apache server is running PHP version 8.1.0-dev. I did a google search for exploits on the PHP version PHP/8.1.0-dev and found something useful. I found an interesting exploit on exploit-db with a link to a GitHub page. 

<img width="329" alt="f7ba7135554043ebae62b0f89446b963" src="https://user-images.githubusercontent.com/114961392/194975961-306f7508-a91d-46e9-843e-4197d1d277f4.png">

https://www.exploit-db.com/exploits/49933  
https://github.com/flast101/php-8.1.0-dev-backdoor-rce

## Exploitation

After using git clone to copy the exploits from GitHub, I decided to run the reverse shell exploit.
First, I set up a netcat listener on port 4444, ran the exploits help menu for correct syntax, then ran the exploit.

<img width="595" alt="image" src="https://user-images.githubusercontent.com/114961392/200478542-02b7364a-ee2f-4963-9b18-153cfe14b023.png">

<img width="410" alt="image" src="https://user-images.githubusercontent.com/114961392/200478592-9cea3f06-56ee-463c-a237-54fc870d345b.png">

This successfully gained entry into the system under the user “james”. Use cat to expose the user flag.

<img width="203" alt="5286299191b24311886c04599c1f6e2e" src="https://user-images.githubusercontent.com/114961392/194977976-10d3d717-c9dd-4f42-9e6b-acb13838fbf7.png">

## Privilege Escalation

Now that we are in the system, let’s try to escalate our privileges.  
Typing sudo -l reveals that we can run a binary called knife from root. Let’s run the binary and do a google search to find out what it is. 

<img width="470" alt="73e3bdf2c4f04c23bad7325fd9ed8017" src="https://user-images.githubusercontent.com/114961392/194976505-d7c55a0d-517b-41b9-b193-a2187593ae3c.png">

After doing some research I discovered that it is a command-line tool that provides an interface between a local chef-repo and the Chef Infra Server. 
After digging much deeper for an exploit, I found out that knife has a function called exec which can run ruby scripts. Eventually, after checking GTFOBins, I found a useful command.

sudo knife exec -E 'exec "/bin/sh"'

<img width="611" alt="39241dffe9a945d483ab2893d86da5ee" src="https://user-images.githubusercontent.com/114961392/194976598-4f8e1e46-98f0-4872-a766-a3e88b815256.png">

I ran the sudo command and WOOSHKAH! We have root. Now we can capture the root flag.

<img width="256" alt="65c4607ee1a549bdaf6832c7754a9d44" src="https://user-images.githubusercontent.com/114961392/194977747-0488a3e8-0539-48e4-b703-d85d8dacaca6.png">
