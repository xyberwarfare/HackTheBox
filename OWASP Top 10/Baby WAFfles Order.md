## HackTheBox - Baby WAFfles Order (Challenge)
Baby WAFfles Order is the fifth challenge in the OWASP Top 10 track.  
Our WAFfles and ice scream are out of this world, come to our online WAFfles house and check out our super secure ordering system API!  
Vulnerability: XXE (XML External Entities)  

### Reconnaissance
HTB has presented us with a website and files to download for this challenge. First let's take a look at the website.  

<img width="521" alt="image" src="https://user-images.githubusercontent.com/114961392/202961268-e334042e-08e1-44f9-a9b4-8367ce581eb2.png">

There is not much on here except a simple form to order WAFfles or ice scream. Let's have a look at the downloaded files.  

<img width="350" alt="image" src="https://user-images.githubusercontent.com/114961392/202961916-a8fbfcd4-39d3-4bba-9593-db1d41e49e67.png">

In the **index.php** file, it looks like makes a POST request through an **OrderController** script. If we take a look at the OrderController script we can see that it accepts JSON content as well as XML.  

<img width="482" alt="image" src="https://user-images.githubusercontent.com/114961392/202962204-16059793-a649-403a-9065-a85d716d366e.png">

So for this challenge we will be playing around with an XXE (XML External Entity) vulnerability.  
Let's fire up Burp Suite and take a look at the requests. I will intercept an order request and send to repeater.  

<img width="598" alt="image" src="https://user-images.githubusercontent.com/114961392/202962572-ef332c18-c7f3-4864-a5c5-b487b4542d26.png">

Now let's try change the request to XML instead of JSON. First I will change the content type to **Content-Type: application/xml**.  
After crafting the request to XML format, I tested to see if it would accept it first and it did.  

<img width="482" alt="image" src="https://user-images.githubusercontent.com/114961392/202963267-9cd90513-cbea-4b8d-b184-ee1076657d05.png">

Now let's try adding the payload **<!DOCTYPE replace [<!ENTITY xwar SYSTEM "file:///etc/passwd"> ]>** and changing the food value to **&xwar;**.
This should show the /etc/passwd file.  

<img width="596" alt="image" src="https://user-images.githubusercontent.com/114961392/202963591-96ec2fc6-5bc5-48df-8469-ac098a5249c7.png">

Finally, we will change the directory to read the flag.

<img width="595" alt="image" src="https://user-images.githubusercontent.com/114961392/202963709-f97c66ff-c6dc-44f4-9b83-dc03aef8974b.png">
