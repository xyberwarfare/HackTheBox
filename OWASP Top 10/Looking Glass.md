## HackTheBox - Looking Glass (Challenge)

Looking Glass is the first challenge in the OWASP Top 10 track.  
Vulnerability: Remote Code Execution  

### Reconnaissance

Looking at the web page, we can see that it has a Ping and Traceroute application. These should be vulnerable to an OS Command Injection attack.  

<img width="707" alt="image" src="https://user-images.githubusercontent.com/114961392/202931246-7a7cd257-760d-4448-9214-9d71f45d3111.png">

Let's test out the Ping function.  

<img width="709" alt="image" src="https://user-images.githubusercontent.com/114961392/202931343-8c7fc3bd-ac35-4bce-85a3-7b8cbc8fd4f3.png">

### Exploitation

Alright, let's try to inject some commands. I will try to see if I can read the /etc/passwd file. First I will add either **& ; or |** after the IP address, followed by command **cat /etc/passwd**.  

<img width="707" alt="image" src="https://user-images.githubusercontent.com/114961392/202931504-de8de812-4f9e-4259-ab1d-0d529020ca44.png">

This works and we can read the passwd file which confirms there is a remote code execution vulnerability. From here, we can do a number of different attacks such as a reverse shell however for this instance we are going to dig deeper to find the flag.  

<img width="706" alt="image" src="https://user-images.githubusercontent.com/114961392/202931598-9caa276d-2d5d-42f8-b0ac-a07fa0dd8418.png">

By using the **ls** command we can display the directory contents which just has the **index.php** file. Now let's use **ls /** to list the root directory.  

<img width="706" alt="image" src="https://user-images.githubusercontent.com/114961392/202931725-3a397ae4-0d09-422c-82f5-4a806c142505.png">

There is an intestering file here called **flag_VXMkY**. Let's try read it using **& cat /flag_VXMkY**.  

<img width="707" alt="image" src="https://user-images.githubusercontent.com/114961392/202931850-35e3d259-b2e7-47bc-becb-a3314a07a5c7.png">

Wooshkah! We now have the flag.
