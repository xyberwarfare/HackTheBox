## Timelapse - HackTheBox
<img width="419" alt="image" src="https://user-images.githubusercontent.com/114961392/204408532-32f23cde-ea5d-4173-97ee-1c7f6c2dfb47.png">

Machine: Timelapse  
Operating System: Windows  
Release Date: 26/03/2022  
Difficulty: Easy  

Vulnerability: SMB, Password Cracking (.zip file & .pfx file)  

### Reconnaissance
Scan all ports with nmap.  

<img width="445" alt="image" src="https://user-images.githubusercontent.com/114961392/204408747-4beb663e-7f79-445f-9a89-14b62cf7c090.png">

Tried to scan all open ports with service and script scan however ping seemed to be blocking the requests. I will add the flag **-Pn**.  

<img width="415" alt="image" src="https://user-images.githubusercontent.com/114961392/204408929-845da4d4-16d4-4cf2-b0dd-1b1f7d91164d.png">

<img width="659" alt="image" src="https://user-images.githubusercontent.com/114961392/204408958-861bd847-4f86-415d-904b-8fd48dbc1cdb.png">

I will add **timelapse.htb** to my /etc/hosts file since it comes up in the scan.  
Port 80 (HTTP) is not open so there is no webpage, let's have a look at SMB (port 445) with **smbmap**.  

<img width="505" alt="image" src="https://user-images.githubusercontent.com/114961392/204409201-a3a6852e-116b-4e1b-bd1d-5323dd751e10.png">

This shows that **IPC$** and **Shares** are readable.  
Let's dig further into these using **smbclient**. First IPC$, however nothing interesting.  

<img width="246" alt="image" src="https://user-images.githubusercontent.com/114961392/204409534-bdb5157d-9c1d-44b9-8a43-9bd529c14809.png">

**Shares** has some interesting files.  

<img width="387" alt="image" src="https://user-images.githubusercontent.com/114961392/204409762-b09692f1-1224-4d72-bdb5-4b259edfa618.png">

The **Dev** folder has a file called **winrm_backup.zip**, this looks interesting so I will grab it.  
Now let's try to unzip the winrm_backup.zip file. It is password protected so it does not load without a password.  

<img width="287" alt="image" src="https://user-images.githubusercontent.com/114961392/204410174-fcf08b63-ac48-4810-9866-c6fcbcbef469.png">

Let's have a look at what is inside the file.  

<img width="261" alt="image" src="https://user-images.githubusercontent.com/114961392/204417604-6e16eed5-0e4d-49ea-a914-39d5f1c354b1.png">

I will use **JohnTheRipper** to crack this. First, I will use the module **zip2john** which creates a hash file from the zip file which can be cracked.  

<img width="752" alt="image" src="https://user-images.githubusercontent.com/114961392/204410276-c053f07d-7dc1-48bc-8a10-0bc57a3c4381.png">

Now I will crack the hash file with **JohnTheRipper**.  

<img width="490" alt="image" src="https://user-images.githubusercontent.com/114961392/204410379-1085d577-9b2e-4c01-95af-ce144b5c36d4.png">

This cracks the password with is **supremelegacy**. I will use this password to unzip the contents.  
In the zip file is a pfx file called **legacyy_dev_auth.pfx**. A quick google search reveals PFX files are digital certificates that contain both the SSL certificate (public keys) and private key. They’re essential for establishing secure connections between two devices. PFX files are usually issued by a certificate authority and contain information about the issuing CA, the certificate holder, and the certificate’s public and private keys.  
[Here](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file) is instructions on how to extract the certificate and keys from the pfx file using **openssl**.  
However it requires a different password to extract the certificate.  

<img width="338" alt="image" src="https://user-images.githubusercontent.com/114961392/204418761-227c5abf-d12c-4f42-bc66-80332ae04886.png">

JohnTheRipper has another module for PFX files. I will use this to create a hash and crack the password.  

<img width="499" alt="image" src="https://user-images.githubusercontent.com/114961392/204421037-d300d6b1-d0b8-4f62-b505-35a3365f3d28.png">

Now we can use **openssl** to extract the certificate and key. Remember to use the **-nodes** flag at the end of the command to stop asking for PEM password everytime.    

<img width="372" alt="image" src="https://user-images.githubusercontent.com/114961392/204424072-a48c6ff1-521d-4deb-8df2-325b94e26319.png">

<img width="338" alt="image" src="https://user-images.githubusercontent.com/114961392/204421560-7f9b0cdc-642b-4103-aa5a-9347ab05b184.png">

### Exploitation
I will be using **Evil-WinRM** to connect to the Windows target. Evil-WinRM is the best tool for connecting to Windows from Linux.  
**evil-winrm -i 10.10.11.152 -c cert.pem -k key.pem -S**

<img width="683" alt="image" src="https://user-images.githubusercontent.com/114961392/204422551-9d2dd688-582c-4987-9389-81977106e832.png">

From here we can go ahead can capture user flag.  

<img width="284" alt="image" src="https://user-images.githubusercontent.com/114961392/204422701-4115c61d-610a-4242-9dd3-61966e73ffdf.png">

### Privilege Escalation

From here I will upload **WinPEAS** to find privilege escalation path.  
A very important place to check on Windows computers is the powershell history. WinPEAS finds the file location.  

<img width="597" alt="image" src="https://user-images.githubusercontent.com/114961392/204425294-542b8c62-429c-4092-9179-6a90f4e14772.png">

The powershell history reveals that a user called **svc_deploy** was created with the password **E3R$Q62^12p7PLlC%KWaxuaV**.  

<img width="733" alt="image" src="https://user-images.githubusercontent.com/114961392/204425671-4b3154a7-a107-4f26-ad34-fcf79653adb9.png">

I will use these credentials to log in again with Evil-WinRM.  

<img width="689" alt="image" src="https://user-images.githubusercontent.com/114961392/204425885-e1b92d8e-92bd-459b-83e7-6460eaae7768.png">

Once I am lodded in, I will use command **net user svc_deploy** to check user information. This shows that svc_deploy is part of the **LAPS_Reader** group.  

<img width="359" alt="image" src="https://user-images.githubusercontent.com/114961392/204426513-334b39cf-e41c-4872-8279-8c62812ee57a.png">

A quick google search on LAPS reveals that Microsoft Local Administrator Password Solution (LAPS) provides a simple way to manage local administrator passwords on domain joined Windows Servers and endpoints, ensuring that each password is unique and is changed regularly.  
I need to use the following command to read the administrator password **Get-ADComputer DC01 -property 'ms-mcs-admpwd'**.  

<img width="476" alt="image" src="https://user-images.githubusercontent.com/114961392/204427320-207bed23-cf5d-4696-9b7c-8c5413a15e19.png">

This reveals that the password is **ff28+p,.@+!dZ50N1N.@5oK2**  
I will connect again with Evil-WinRM using Administrator credentials.  

<img width="686" alt="image" src="https://user-images.githubusercontent.com/114961392/204427787-c0b54f7b-fa0b-46c5-927f-36b1f8e4e21a.png">

This connects however I got stuck here as I could not find the root flag. It is not in the Desktop folder like it is normally.  
Having a look in the Users directory, I can see there is a user called **TRX**. I eventually found the root flag in this users Desktop directory.  

<img width="266" alt="image" src="https://user-images.githubusercontent.com/114961392/204428366-94fb0fca-1e8c-402b-9a1f-c42a7d657a39.png">

I did some research on why the flag was moved and found the following in **0xdf write-up**:  
HackTheBox has a system in place to prevent cheating referred to as flag rotation. Each time a VM is powered on, the system generates random flags, and sets the contents of user.txt and root.txt, storing those values associated with that instance in the database. Then when I go to submit my flag, it know what lab / machine I’m associated with, and checks if it’s correct.  
The system needs to know the password for an administrative user on the box. For systems where LAPS is in use, that’s not possible (as it changes the password periodically). The work around here is to add another user that the system can use for administrative access with a static password. This user exists only to enable flag rotation.  




