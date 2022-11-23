## HackTheBox - Baby Website Rick (Challenge)
Baby Website Rick is the ninth challenge in the OWASP Top 10 track. This challenge was hard! I had to do a tonne of research on serialization and pickle before I got anywhere.  
Vulnerability: Serialization  

### Reconnaissance
The web page just has a picture from Rick and Morty with some text down the bottom which says **Don't play around with this serum morty!! <__main__.anti_pickle_serum object at 0x7fd800d1e850>**.  
This gives us a clue that we need to use pickle within python. In Python, the pickle module lets you serialize and deserialize data. Essentially, this means that you can convert a Python object into a stream of bytes and then reconstruct it (including the object’s internal structure) later in a different process or environment by loading that stream of bytes.  

<img width="779" alt="image" src="https://user-images.githubusercontent.com/114961392/203451891-4b047344-d301-4ac5-89d5-4655e7eead4c.png">

The source code has nothing of interest. Let's have a look at Burp Suite.  

<img width="418" alt="image" src="https://user-images.githubusercontent.com/114961392/203452307-5e2360cd-44ee-4da6-ab70-f930e0ea1614.png">

Let's try and decode the cookie to see if it gives us anything interesting.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/203452893-5811e942-fa60-49b4-a41d-5cf0d74ede55.png">

Using Base64, we can see it decodes to give us a few clues. I will save the cookie file first before moving on.  
Now let's have look at pickle tools within python.  

<img width="548" alt="image" src="https://user-images.githubusercontent.com/114961392/203456622-42067b96-d886-4d6d-9835-b6bc6c9020ae.png">

If we use the pickletool module in python to read the decoded cookie file (saved as pickle2) we can see a similar structure to the Base64 decoded version of the cookie.  
I will add the **-a** flag to the command to give us a description of each line.  

<img width="604" alt="image" src="https://user-images.githubusercontent.com/114961392/203457039-76fdf569-0e66-4089-9644-bc674b15db9d.png">

### Exploitation
Let's start crafting the exploit in python. The most useful guide for pickle exploitation by David Hamann is [here](https://davidhamann.de/2020/04/05/exploiting-python-pickle/).  
First I will make a basic script to test out pickles capabilities.  

<img width="914" alt="image" src="https://user-images.githubusercontent.com/114961392/203458251-baf0e96f-566b-4d85-80b6-727f2f8caa50.png">

Result:

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/203458330-fa7a5569-e149-4f48-a6dd-abc6a70f7791.png">

It looks like we need to add anti_pickle_serum so I will add it with the value 'test'.  

<img width="916" alt="image" src="https://user-images.githubusercontent.com/114961392/203458453-4741393f-d4e0-49dc-a5e9-6d95cfa5fdea.png">

This sorts out the last error however we are now presented with a new one.  

<img width="832" alt="image" src="https://user-images.githubusercontent.com/114961392/203458579-a5619504-d3d7-4f9e-88fc-f489f24b328c.png">

It looks like we need to pass anti_pickle_serum as an object. At the moment it is passing as a string. So I will create a **class** in python to do just that.  

<img width="917" alt="image" src="https://user-images.githubusercontent.com/114961392/203459070-b759886b-8ec4-45ec-a69e-fda20f875f0c.png">

Now it works without any errors:

<img width="343" alt="image" src="https://user-images.githubusercontent.com/114961392/203459122-2434c5ad-65a2-40eb-94e6-23aa5662fb4b.png">

Now let's go back and look at David Hamann's guide for the next steps. It looks like we add the pickletools module to the script and re-write it to get the same output with added structure.  

<img width="899" alt="image" src="https://user-images.githubusercontent.com/114961392/203459621-02db6150-cea1-4e20-a9f2-8b978e697201.png">

<img width="331" alt="image" src="https://user-images.githubusercontent.com/114961392/203459782-35554069-d494-451f-b57a-f5f736f68082.png">

Next the guide says, **Reading a bit further down in the docs we can see that implementing __reduce__ is exactly what we would need to get code execution, when viewed from an attacker’s perspective**.  
So we need to add the **__reduce__** function. I copied the __reduce__ function from the guide and adjusted it to suit our needs.  

<img width="896" alt="image" src="https://user-images.githubusercontent.com/114961392/203460766-56ed821c-bc5c-470e-b7bd-d64e90aa4891.png">

Result:

<img width="290" alt="image" src="https://user-images.githubusercontent.com/114961392/203460825-cb4ea316-b04a-4945-b51c-1de0540e26a0.png">

I tested this output by replacing the cookie and refreshing the page however it did not work. Looking back at the guide, we need to add the protocol value and add anti_pickle_serum to a dictionary.  

<img width="904" alt="image" src="https://user-images.githubusercontent.com/114961392/203461752-609ce9ad-6adb-40a7-9f1e-9917109bd6d8.png">

Result:

<img width="352" alt="image" src="https://user-images.githubusercontent.com/114961392/203461785-3a532e61-b7a6-4e4f-8177-6a57a0600670.png">

When replacing the cookie with this value:

<img width="552" alt="image" src="https://user-images.githubusercontent.com/114961392/203461830-ef40a778-d7c7-409b-a918-fd19265ddd67.png">

We are starting to get somewhere!  

Next, we will replace the **OS** module in python with the **subprocess** module. However this still did not work.  
This is where I got stuck. After many failed attempts, I realised it needs to be run from python2 not python3.  

<img width="428" alt="image" src="https://user-images.githubusercontent.com/114961392/203462342-ccc6968b-2e31-4bed-8056-927ba65c53a7.png">

<img width="548" alt="image" src="https://user-images.githubusercontent.com/114961392/203462360-f9216b5c-9549-47d5-9b2e-db2f754fa7b2.png">

Now we are getting somewhere! The **ls** command worked and we can now see a list of the directory with the flag there.  
Let's change the command section to **cat flag_wIp1b** to display the flag.  

<img width="744" alt="image" src="https://user-images.githubusercontent.com/114961392/203462561-c8bda7b3-caa1-4a01-aa37-928dda7857f7.png">

WOOSHKAH! Finally we have the flag.
