# Write-up of swagshop HTB

We will start off with nmap scan of the ip 10.10.10.140 

```
[sarthak@sarthak swagshop]$ nmap -sV 10.10.10.140 -v -Pn
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-19 11:54 IST
NSE: Loaded 43 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 11:54
Completed Parallel DNS resolution of 1 host. at 11:54, 0.01s elapsed
Initiating Connect Scan at 11:54
Scanning 10.10.10.140 [1000 ports]
Discovered open port 22/tcp on 10.10.10.140
Discovered open port 80/tcp on 10.10.10.140
Completed Connect Scan at 11:55, 34.33s elapsed (1000 total ports)
Initiating Service scan at 11:55
Scanning 2 services on 10.10.10.140
Completed Service scan at 11:55, 6.52s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.140.
Initiating NSE at 11:55
Completed NSE at 11:55, 1.12s elapsed
Initiating NSE at 11:55
Completed NSE at 11:55, 0.00s elapsed
Nmap scan report for 10.10.10.140
Host is up (0.25s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
1045/tcp filtered fpitp
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.42 seconds
```
