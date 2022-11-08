# Blue - HackTheBox Walkthrough

<img width="422" alt="image" src="https://user-images.githubusercontent.com/114961392/200467220-2bd05b1c-bec9-4708-8ef2-7f8bd3d16e2b.png">
 
Machine: Blue
Operating System: Windows
Release Date: 28/07/2017
Difficulty: Easy

An old machine however is a great place to start since it has the infamous EternalBlue exploit which affected Windows 7 & 10 before the critical MS17-101 patch was released. 

## Enumeration

Let’s start by scanning all ports on target with Nmap as well as running service and script services. Port 445 (SMB) looks very interesting since it is open and running Windows 7. EternalBlue, or MS17-101, is a vulnerability within Microsoft’s Server Message Block (SMB) protocol. Windows 7, without the critical MS17-101 patch, is vulnerable to this exploit.

nmap -p- --min-rate=1000 -sV -sC 10.10.10.40
