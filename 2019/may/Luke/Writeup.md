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
We have Ftp opened so first let's enumerate that 
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
So we have got a file 'for_Chihiro.txt' Let's see What this has...
```
[sarthak@sarthak ~]$ cat for_Chihiro.txt 
Dear Chihiro !!

As you told me that you wanted to learn Web Development and Frontend, I can give you a little push by showing the sources of 
the actual website I've created .
Normally you should know where to look but hurry up because I will delete them soon because of our security policies ! 

Derry  
```

Nothing much ..
