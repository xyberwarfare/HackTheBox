## HackTheBox - Sanitize (Challenge)

Sanitize is the second challenge in the OWASP Top 10 track.  
Vulnerability: SQL Injection  

### Reconnaissance
The web page has a basic username and password login form. 

<img width="550" alt="image" src="https://user-images.githubusercontent.com/114961392/202932334-ac3327d1-d8ed-461a-9269-b3ef53cf04e7.png">

Let's try entering **test** in both fields.  

<img width="556" alt="image" src="https://user-images.githubusercontent.com/114961392/202933030-75acd21d-a4b7-45b0-bfca-f762055fb78f.png">

The error messages shows us that this should be vulnerable to SQL Injection.

### Exploitation

Let's try to bypass the login form by injecting some standard SQLi techniques. First I will try **' or 1=1#**.  

<img width="558" alt="image" src="https://user-images.githubusercontent.com/114961392/202932471-806f5ce9-6416-4a61-bdac-8afa8a8c4ca8.png">

This did not work. Let's try some more injection techniques such as **' or 1=1--**.  

<img width="552" alt="image" src="https://user-images.githubusercontent.com/114961392/202932592-359c86a9-5c80-4bc1-98d7-f57bf5c40318.png">

Wooshkah! This worked and now we can see the flag.
