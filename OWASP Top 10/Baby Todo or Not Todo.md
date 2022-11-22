## HackTheBox - Baby Todo or Not Todo (Challenge)
Baby Todo or Not Todo is the sixth challenge in the OWASP Top 10 track.  
Vulnerability: Broken Authentication  

### Reconnaissance
The web page has a simple input form where you can add an item to a "todo" list. I tried it out by entering **test** into the field and it put it in the list.  

<img width="1298" alt="image" src="https://user-images.githubusercontent.com/114961392/203195743-c48362de-61f2-40f0-a7c3-4637680c3b1f.png">

Looking at the page source code, there is some interesting text down the bottom.  

<img width="395" alt="image" src="https://user-images.githubusercontent.com/114961392/203196254-c29a3d15-ee85-40da-8788-87dca5577ce6.png">

**const update = () => getTasks('userCd9b26bf')**  
Looks like it is a small script to constantly update the tasks for a specific user which has been randomly generated.  

**update()**  
This calls the update function which should update the users tasks.  

**setInterval(update, 3000)**  
This specifies the interval in which the update function will be called every 3000 milliseconds (3 seconds).  

### Exploitation
Let's play around with some of these functions in the console section of web page inspector.  
Before I do this, I will fire up Burp Suite and add the website to the scope so it catches all the requests.  

First I will use **getstatus('all')** function however it tells me it is not defined.  
Next, I will use the **getTasks('userCd9b26bf')** function however it is also not defined.  
When I changed this to **getTasks('all')**, something interesting happened. The web page very quickly gives me a list of all tasks that have been entered however it goes away before I have time to properly read.  
This indicates that it is possible to show all tasks with a bit more messing around.  
Finally, I tired the **setInterval(update, 3000)** function with getTasks nestled in to make **setInterval(getTasks('all'), 3000)**. This showed me all the tasks entered for 3 seconds.  
I briefly saw that there is a flag in there!  

<img width="481" alt="image" src="https://user-images.githubusercontent.com/114961392/203199116-b6a948aa-12bc-42cb-998b-5dc02ab88161.png">

OK now we know that we have to somehow show all the tasks that have been uploaded. If I go back to Burp Suite and have a look at the requests, some interesting responses have been populated.  
If we have a look at the request http://161.35.162.182:30363/api/list/all/?secret=Cabd2dC2bAa630f, we can see that the response has all the tasks!  
If we enter the URL in the browser, it comes up showing all tasks and we can go ahead and capture the flag.  

<img width="737" alt="image" src="https://user-images.githubusercontent.com/114961392/203200889-6fdc6c29-5701-4b97-b5db-8e65dae8a8de.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/114961392/203200936-693b7e10-f087-41ee-b310-be8dd371c992.png">
