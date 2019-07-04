# Haystack (HACK THE BOX)

Hey Guys, Today we will be doing Haystack from HackTheBox
<br/>
<<logo.png>>
<br/>

## NMAP Scan

```
[sarthak@sarthak ~]$ nmap -sV 10.10.10.115 -Pn -v
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-05 02:21 IST
NSE: Loaded 43 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 02:21
Completed Parallel DNS resolution of 1 host. at 02:21, 0.01s elapsed
Initiating Connect Scan at 02:21
Scanning 10.10.10.115 [1000 ports]
Discovered open port 22/tcp on 10.10.10.115
Discovered open port 80/tcp on 10.10.10.115
Discovered open port 9200/tcp on 10.10.10.115
Completed Connect Scan at 02:21, 8.14s elapsed (1000 total ports)
Initiating Service scan at 02:21
Scanning 3 services on 10.10.10.115
Completed Service scan at 02:22, 11.56s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.10.115.
Initiating NSE at 02:22
Completed NSE at 02:22, 0.81s elapsed
Initiating NSE at 02:22
Completed NSE at 02:22, 0.00s elapsed
Nmap scan report for 10.10.10.115
Host is up (0.18s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    nginx 1.12.2
9200/tcp open  http    nginx 1.12.2

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.92 seconds
```
We have 3 services running so let's start enumerating the port 80 first...

## Webserver Enumeration
<br/>
<selection 007>
<br/>
So we have a image only so let's download it and analyse it first...

#### Clues in image

We did strings on the image and find a base64 in it
```
[sarthak@sarthak haystack]$ strings needle.jpg | tail -5
#=pMr
BN2I
,'*'
I$f2/<-iy
bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==
```
which decodes to this:-
```
[sarthak@sarthak haystack]$ echo "bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==" | base64 -d

la aguja en el pajar es "clave"
```
So after a quick google i found that clave means key but we will just keep it as currently it doesn't make any sense

## Enumeration of Elastic-Search

On port 9200 Elastic Search is running so let's quickly find out all the indices and their contents for that i used [this](https://www.bmc.com/blogs/elasticsearch-commands/) cheatsheet..

<br/>
<selection 008>
<br/>
We got 3 indices so from which ```.kibana``` is the default of elastic search so we will enumerate b/w  ```bank``` and ```quotes``` 

### Finding the "Key"

Let's see all the content of ```quotes``` index first using ```http://10.10.10.115:9200/quotes/_search/?size=1000```

<br/>
<selection 009>
<br/>
We got this data and from here if we search the key or *clave* in spanish we will find 2 strings...

```
"quote":"Esta clave no se puede perder, la guardo aca: cGFzczogc3BhbmlzaC5pcy5rZXk="
Tengo que guardar la clave para la maquina: dXNlcjogc2VjdXJpdHkg "}
```
So we can see let's decode the base64 and we'll see what we get...
```
user: security
pass: spanish.is.key
```

## Low priv shell

we used this creds to login to ssh 
```
[sarthak@sarthak haystack]$ ssh security@10.10.10.115
The authenticity of host '10.10.10.115 (10.10.10.115)' can't be established.
ECDSA key fingerprint is SHA256:ihn2fPA4jrn1hytN0y9Z3vKpIKuL4YYe3yuESD76JeA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.115' (ECDSA) to the list of known hosts.
security@10.10.10.115's password: 
Last login: Thu Jul  4 12:28:57 2019 from 10.10.14.6
[security@haystack ~]$ id
uid=1000(security) gid=1000(security) grupos=1000(security) contexto=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[security@haystack ~]$ 
```

