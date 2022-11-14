## Photobomb - HackTheBox

<img width="416" alt="image" src="https://user-images.githubusercontent.com/114961392/201578781-8f2568c0-e6dd-4671-b374-ac5328570c05.png">

Machine: Photobomb  
Operating System: Linux  
Release Date: 08/10/2022  
Difficulty: Easy  

Photobomb is new machine however did not take to long to conpromise. It is a website which requires authorization, then you can download a selected photo.  
I conpromised the system by injecting code after intercepting the request when you download a picture. Privledge Escaltion was acheived by exploiting the /opt/cleanup.sh file.  

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
Once the hardcoded credentials have been entered, we can access the site. It is just a
