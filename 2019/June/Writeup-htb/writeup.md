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
