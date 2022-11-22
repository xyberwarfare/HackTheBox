## HackTheBox - Baby BoneChewerConh (Challenge)
Baby BoneChewerCon is the seventh challenge in the OWASP Top 10 track. This one is too easy, took me about 10 seconds to find the flag.  
Vulnerability: Debug  

### Reconnaissance
The web page is an event registration page. It allows you to enter your name and register.  

<img width="1226" alt="image" src="https://user-images.githubusercontent.com/114961392/203207251-81a1f37d-41a5-4145-bb61-cd270706ec85.png">

I started by entering **test** in the name field and clicked **Register me!**. This leads to a laravel debug page. Doing a quick google search for **laravel** reveals that Laravel is a free and open-source PHP web framework, created by Taylor Otwell and intended for the development of web applications following the model–view–controller architectural pattern and based on Symfony.  
Looking at the debug information we can easily find the flag straight away!  

<img width="941" alt="image" src="https://user-images.githubusercontent.com/114961392/203207859-220a85c4-6d56-4aa1-979a-7350e7c174fb.png">

There is also some interesting information on here such as **connection type, host, port, db username, and db password**.
