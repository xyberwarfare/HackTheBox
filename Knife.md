# Knife - HackTheBox Walkthrough
 
Difficulty: Easy

Machine Status: Retired

Comments: I decided to do this retired machine first as it looks nice and easy and is a good place to start. Plus it has a cool name!

## Enumeration

First of all, I started scanning all ports with Nmap. I like using the -A -T4 flags to start with and if they produce nothing then I dig deeper. We have ports 22 (SSH) and 80 (HTTP) open. Port 80 is running Apache httpd 2.4.41. I did a quick google search for any good exploits for this version however couldn’t find anything that interesting.

https://httpd.apache.org/security/vulnerabilities_24.html

<img width="516" alt="9abc324a0460427395664d2bfccff1c4" src="https://user-images.githubusercontent.com/114961392/194970418-37749233-5f95-4bd4-a569-2986d1d96224.png">

Next, I fired up Burp Suite and went to the web page. After probing around I didn’t find anything interesting as there are no links on the page. I decided to probe using dirsearch however there was nothing of great interest. 

<img width="600" alt="74f2b922bea14415927eac5559df7575" src="https://user-images.githubusercontent.com/114961392/194978719-0c5ec71a-f1e2-4ea6-bc64-71147dee5cf8.png">

<img width="472" alt="19e20ecacc5b4110b1d2ac89efa9e525" src="https://user-images.githubusercontent.com/114961392/194975850-4f261afd-fa8e-4933-a384-8b7c45b5e20b.png">

I went back to Burp Suite to check out the requests and responses. I did a google search for exploits on the PHP version PHP/8.1.0-dev and found something useful. I found an interesting exploit on exploit-db with a link to a GitHub page. 

<img width="329" alt="f7ba7135554043ebae62b0f89446b963" src="https://user-images.githubusercontent.com/114961392/194975961-306f7508-a91d-46e9-843e-4197d1d277f4.png">

https://www.exploit-db.com/exploits/49933

https://github.com/flast101/php-8.1.0-dev-backdoor-rce

## Initial Access

After using git clone to copy the exploits from GitHub, I decided to run the reverse shell exploit.
First, I set up a netcat listener on port 4444, ran the exploits help menu for correct syntax, then ran the exploit.

<img width="593" alt="ecb7376407e94d2388b9d5f8391a9e2b" src="https://user-images.githubusercontent.com/114961392/194976124-3c716afa-2c60-4b9a-a243-9d0d5d8f25ee.png">

<img width="398" alt="66225b1d4c1943adb0827a672644a01c" src="https://user-images.githubusercontent.com/114961392/194976221-f78482dd-8518-49d3-97dd-f32d147f5e67.png">

This successfully gained entry into the system under the user “james”. Use cat to expose the user flag.

<img width="203" alt="5286299191b24311886c04599c1f6e2e" src="https://user-images.githubusercontent.com/114961392/194977976-10d3d717-c9dd-4f42-9e6b-acb13838fbf7.png">

## Privilege Escalation

Now that we are in the system, let’s try to escalate our privileges. 
Before I continue, I should upgrade our shell. I am not sure if it is nessesary or not so I thought I would just try it. 

<img width="205" alt="1b648fce26ef409abf6abf4aa8dc6564" src="https://user-images.githubusercontent.com/114961392/194977604-94052ecb-ce1c-482d-91d9-0af77553c5b6.png">

Typing sudo -l reveals that we can run a binary called knife from root. Let’s run the binary and do a google search to find out what it is. 

<img width="470" alt="73e3bdf2c4f04c23bad7325fd9ed8017" src="https://user-images.githubusercontent.com/114961392/194976505-d7c55a0d-517b-41b9-b193-a2187593ae3c.png">

After doing some research I discovered that it is a command-line tool that provides an interface between a local chef-repo and the Chef Infra Server. 
After digging much deeper for an exploit, I found out that knife has a function called exec which can run ruby scripts. After doing some more digging around on google I eventually found some interesting code on GTFOBins which should be useful. 

<img width="611" alt="39241dffe9a945d483ab2893d86da5ee" src="https://user-images.githubusercontent.com/114961392/194976598-4f8e1e46-98f0-4872-a766-a3e88b815256.png">

I ran the sudo command and BOOM! We have root. Now we can capture the root flag.

<img width="256" alt="65c4607ee1a549bdaf6832c7754a9d44" src="https://user-images.githubusercontent.com/114961392/194977747-0488a3e8-0539-48e4-b703-d85d8dacaca6.png">
