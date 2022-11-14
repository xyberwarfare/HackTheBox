# Lame - HackTheBox

<img width="417" alt="image" src="https://user-images.githubusercontent.com/114961392/201272878-722a2d2e-2ddc-400f-9190-6c772f04b4af.png">
 
Machine: Lame  
Operating System: Linux  
Release Date: 14/03/2017  
Difficulty: Easy  

Lame was the first box released on HackTheBox. It was nice and easy as it involves using Metasploit modules.  
Vulnerability: Samba 3.0.20

## Enumeration

Let's start by scanning all ports with nmap.  
nmap -p- --min-rate=10000 10.10.10.3  
<img width="326" alt="image" src="https://user-images.githubusercontent.com/114961392/201274639-279bc232-5e94-4fbf-95f9-3e7c6db9bf4e.png">

Let's do a version and script scan on the 5 open ports before we dive in.  
nmap -p21,22,139,445,3632 -sV -sC 10.10.10.3  
<img width="482" alt="image" src="https://user-images.githubusercontent.com/114961392/201274839-e41fbbdc-6666-4ac8-8fb6-cbc61df48db1.png">

### FTP - Port 21

Version: vsftpd 2.3.4  
Anonymous FTP login allowed. However nothing interesting.  
<img width="247" alt="image" src="https://user-images.githubusercontent.com/114961392/201275772-2b5245c6-d4e4-4af1-bec2-c050dbe21c13.png">

Let's search the version for exploits. 2 interesting exploits come up on searchsploit that we will try in the next stage.  
<img width="326" alt="image" src="https://user-images.githubusercontent.com/114961392/201276024-4f71cd30-cc21-43c1-b1d7-d3b3da139e61.png">

### SMB - Port 445

Version: Samba smbd 3.0.20-Debian  
Scanning with smbmap shows 1 accessible share.  
smbmap -H 10.10.10.3  
<img width="649" alt="image" src="https://user-images.githubusercontent.com/114961392/201276873-e7b9631d-6367-48b3-9bc5-6ccde2010801.png">

However looks like there's nothing interesting inside.  
<img width="382" alt="image" src="https://user-images.githubusercontent.com/114961392/201277108-608c7623-6e71-4752-95d8-87cf6911b0f6.png">

Searching for exploits reveals an interesting exploit in Metasploit that matches the version.  I will make a note of this and try in the exploitation stage.
searchsploit Samba 3.0.20  
<img width="418" alt="image" src="https://user-images.githubusercontent.com/114961392/201278037-16ad8c03-1f94-42d7-8efe-1bc24da1c0fc.png">

## Exploitation

### vsftpd 2.3.4 Exploit

Let's start by trying the Metasploit exploit for vsftpd. I loaded up msfconsole, set the exploit, set the options, and run the exploit however no session was created.  
<img width="661" alt="image" src="https://user-images.githubusercontent.com/114961392/201279261-0457848e-2649-40d2-ae3e-cbcfde80ab30.png">

Before giving up on this exploit, I decided to dig deeper and see what it does. After reviewing the exploit code, it looks like if you enter ":)" after the username when connected to port 21 FTP it will trigger. Then, if successfull it should connect back on port 6200. I tested this out using netcat to connect to FTP however it failed.  

### Samba 3.0.20 Exploit

This just leaves Samba to exploit. Let's start by opening the Metasploit module, setting the options, and running the exploit.  
<img width="661" alt="image" src="https://user-images.githubusercontent.com/114961392/201281483-1f134155-00f7-4b04-9fbf-379026a1b7a0.png">

Wooshka! We have a shell with root privileges. From here we can go ahead and get the user flag.  
cat /home/makis/user.txt  
<img width="196" alt="image" src="https://user-images.githubusercontent.com/114961392/201283776-beb0d33a-b5fe-410b-9b3c-e0a9e55b5529.png">

Followed by the root flag since we already have root privileges.  
cat /root/root.txt  
<img width="182" alt="image" src="https://user-images.githubusercontent.com/114961392/201284227-b3871bc0-6bbf-49c1-a7be-e2403b1e3ec6.png">
