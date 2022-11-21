## HackTheBox - Baby Nginxatsu (Challenge)

Baby Nginxatsu is the fourth challenge in the OWASP Top 10 track.  
Vulnerability: Nginx Conf  

### Reconnaissance
The web page has another basic username and password login form with the option to create your own account.  

<img width="557" alt="image" src="https://user-images.githubusercontent.com/114961392/202946688-63937ca8-c47f-43ec-921d-3f0c1ee1abd5.png">

Let's click **Create a new account** and enter the following credentials:
- Name: test  
- Email: test@test.com
- Password: test  

<img width="564" alt="image" src="https://user-images.githubusercontent.com/114961392/202947441-79c76569-4070-41cf-9e14-b93fffed4db3.png">

Now let's login with those credentials.  
Once we are logged in, we can see a page which generates your own nginx config file. I will leave all fields in default and click **Generate Config** down the bottom.  
### Exploitation

<img width="563" alt="image" src="https://user-images.githubusercontent.com/114961392/202948747-8486ccb4-946e-47d5-a4cd-d725714d4c16.png">

After that, a file appears down the bottom under **Configs** called **51**. Upon clicking it, we are redirected to a raw config file.  

<img width="476" alt="image" src="https://user-images.githubusercontent.com/114961392/202949465-82e36ebd-65fa-43f7-baf7-8c919463b25c.png">

Looking at the config file, we can see there is a hashed out section which says **We sure hope so that we don't spill any secrets within the open directory on /storage**. This indicates that the directory /storage is open. Let's navigate to the web page and have a look.  

<img width="394" alt="image" src="https://user-images.githubusercontent.com/114961392/202950683-eedd2830-8e10-456d-963b-d093f983582d.png">

Down the bottom we can see a file which looks interesting, **v1_db_backup_1604123342.tar.gz**. Let's download it using **wget** and extract it's contents.  

<img width="818" alt="image" src="https://user-images.githubusercontent.com/114961392/202950843-44489b3d-5013-4df5-87c1-87386b1cefc5.png">

When extracted, there is a file called **database.sqlite**. This indicates that it is a sqlite file and can be read using **sqlite3** on kali.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/202951240-f7886a9a-1022-4716-89a8-e090ae69f79e.png">

By using sqlite3 to enumerate the database, we can get a list of the users and hashed passwords.  
Once we have a password hash, we can use [crackstation.net](https://crackstation.net/) to check the hash file and crack the hash. This shows the password is **adminadmin1**.  

<img width="765" alt="image" src="https://user-images.githubusercontent.com/114961392/202951638-3abdc906-a131-4a75-bf9a-2b0ae177a124.png">

Now let's try logging in with these credentials. It works and now we can capture the flag.  

<img width="541" alt="image" src="https://user-images.githubusercontent.com/114961392/202951909-b2c4318b-e39c-476f-bbba-845b53648bc5.png">
