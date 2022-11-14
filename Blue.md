# Blue - HackTheBox

<img width="422" alt="image" src="https://user-images.githubusercontent.com/114961392/200467220-2bd05b1c-bec9-4708-8ef2-7f8bd3d16e2b.png">
 
Machine: Blue  
Operating System: Windows  
Release Date: 28/07/2017  
Difficulty: Easy  

An old machine however is a great place to start since it has the infamous EternalBlue exploit which affected Windows 7 before the critical MS17-101 patch was released. 

## Enumeration

Let’s start by scanning all ports on target with Nmap as well as running service and script services. Port 445 (SMB) looks very interesting since it is open and running Windows 7. EternalBlue, or MS17-101, is a vulnerability within Microsoft’s Server Message Block (SMB) protocol. Windows 7, without the critical MS17-101 patch, is vulnerable to this exploit.

nmap -p- --min-rate=1000 -sV -sC 10.10.10.40

<img width="547" alt="image" src="https://user-images.githubusercontent.com/114961392/200467894-202569ef-f438-464b-92ef-290570df6504.png">

Now that we know port 445 is open and running Windows 7, let’s scan to see if it’s vulnerable to MS17-010 by using Nmap’s vulnerability scripts.

nmap -p445 –script vuln 10.10.10.40

<img width="518" alt="image" src="https://user-images.githubusercontent.com/114961392/200468163-5b24686a-7f00-449a-a90d-b1b052d76b30.png">

This indicates that indeed, the target is vulnerable to EternalBlue (MS17-010).

## Exploitation

Let’s check searchsploit to see what comes up. Since this vulnerability was released back in 2017, there should be some exploits available.

<img width="619" alt="image" src="https://user-images.githubusercontent.com/114961392/200468296-5e12b9d3-5c95-4dc8-a9d4-39453272dd15.png">

Searchsploit is showing some exploits in Metasploit so let's check in there too. Metasploit has an exploit called exploit/windows/smb/ms17_010_eternalblue which sounds interesting so let’s give that a go. First load the exploit, then show options to see what we need to input. Set RHOSTS to the target IP. We will keep the default payload settings (windows/x64/meterpreter/reverse_tcp). Set LHOST to attacker IP and LPORT to which port you want to receive the connection.

<img width="980" alt="image" src="https://user-images.githubusercontent.com/114961392/200469103-614419f3-2b82-4b41-b801-d57a0a999861.png">

After we have configured the options, let's run the exploit. It is successful and we get a meterpreter shell with nt authority\system privileges. 

<img width="628" alt="image" src="https://user-images.githubusercontent.com/114961392/200469930-463ac672-26c1-4e10-92fe-757bca071405.png">

From here, we can go to C:\Users\haris\Desktop>user.txt and capture the user flag. 

<img width="227" alt="image" src="https://user-images.githubusercontent.com/114961392/200470258-b0758db1-1b6e-4b26-a905-6e25515d788c.png">

The root flag is also accessible from C:\Users\Administrator\Desktop>user.txt without any further privilege escalation.

<img width="252" alt="image" src="https://user-images.githubusercontent.com/114961392/200470673-0c9ee1ed-0792-4067-a5bd-f2a1c9f958db.png">
