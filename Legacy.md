# Legacy - HackTheBox

<img width="418" alt="image" src="https://user-images.githubusercontent.com/114961392/201549829-b2dad99a-8d6b-45ae-a006-78c10766dace.png">
 
Machine: Legacy  
Operating System: Windows  
Release Date: 14/03/2017  
Difficulty: Easy  

Legacy is another old machine running Windows XP. It was a very easy machine that can be conpromised with Metasploit using the ms17_010_psexec module.  
However I will dig a bit deeper into these exploits to see how they work.  
Vulnerability: MS17-010 & MS08-067.

## Reconnaissance

Let's start by scanning all ports with nmap.  
nmap -p- --min-rate=10000 10.10.10.4  
<img width="410" alt="image" src="https://user-images.githubusercontent.com/114961392/201550152-fcaa0984-f3de-41a0-bc8a-50a829ccf0e4.png">

3 ports are open. Let's run service and script scan on these open ports to enumerate further. This shows that the system is running Windows XP and SMB requires a user account for authentication.  
nmap -p135,139,445 -sV -sC 10.10.10.4  
<img width="502" alt="image" src="https://user-images.githubusercontent.com/114961392/201550249-9698defd-16f6-4204-8d1b-d3a22f57f60b.png">

Now I will run nmap's vulnerability script scan to check if the system has any common vulnerabilities.  
nmap --script vuln 10.10.10.4  
<img width="533" alt="image" src="https://user-images.githubusercontent.com/114961392/201550406-56f2e710-e418-48a6-9267-badbf659bbe5.png">

2 common vulnerabilities are shown, MS17-010 and MS08-067.  

## Exploitation [with Metasploit]

Since I know there are some MS17-010 exploits in Metasploit, I am going to start there.  
Searching for MS17-010 comes up with 5 results.  
<img width="835" alt="image" src="https://user-images.githubusercontent.com/114961392/201551928-450b361c-97ee-4dfe-a1b5-f7951f3e642e.png">

The first one is the EternalBlue exploit. Since I have already covered this vulnerability in the Blue machine, I am do not think it will work however let's try anyway and see what happens.  
Set options and run exploit however exploit failed and no session was created. This is due to the target running Windows 5.1 86x (32-bit) and this exploit only works on targets that are running x64 (64-bit) machines.  
<img width="827" alt="image" src="https://user-images.githubusercontent.com/114961392/201552234-baf97d90-dd98-41cc-b4b0-0cc2afeeb3f0.png">

Let's try the /windows/smb/ms17_010_psexec exploit. Set the opeions and run the exploit. WOOSHKAH, we have a shell.  
<img width="502" alt="image" src="https://user-images.githubusercontent.com/114961392/201552561-ae3a4041-497a-495b-9035-5896ef8c81d5.png">

From here we can navigate to John's desktop to capture the user flag.  
<img width="272" alt="image" src="https://user-images.githubusercontent.com/114961392/201552659-6504b609-7efb-4917-93fa-d793007be575.png">

Then we can navigate straight to the Administrators directory and capture the root flag.  
<img width="314" alt="image" src="https://user-images.githubusercontent.com/114961392/201552784-74112d8f-10e2-4db6-835c-e98d919d6182.png">

