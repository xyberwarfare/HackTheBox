## HackTheBox - Full Stack Conf (Challenge)
Full Stack Conf is the eighth challenge in the OWASP Top 10 track. This one was also very easy and only took a minute.
Vulnerability: XSS Cross-Site Scripting

### Reconnaissance
The web page is an information page for a conference. Down the bottom it gives us a huge clue by saying **Stay up-to-date on Full Stack Conf or pop an alert() to get the flag**.  
Below that is an input field where you would add your email address.  
Let's input some basic XSS to get an alert. I will use <script>alert("xwar")</script>.  

<img width="684" alt="image" src="https://user-images.githubusercontent.com/114961392/203213162-279c829f-05c4-4562-ab8d-169ee3fdbed3.png">

It works and we can capture the flag!  

<img width="247" alt="image" src="https://user-images.githubusercontent.com/114961392/203213290-1c246442-8e7f-4bf8-a81a-c59e8c65644c.png">
