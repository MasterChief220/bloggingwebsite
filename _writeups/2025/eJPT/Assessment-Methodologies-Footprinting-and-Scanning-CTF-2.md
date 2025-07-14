---
layout: writeup
category: eJPT
chall_description: This phase involves actively or passively collecting data such as server headers, open ports, exposed directories, and system configurations. Techniques like scanning, querying DNS records, examining web application files (e.g., robots.txt), and analyzing response headers help uncover critical information that can aid in later exploitation phases. Effective reconnaissance allows testers to map the attack surface, prioritize targets, and plan their approach with minimal detection by the system's defenses.This lab is designed to test your knowledge and skills in performing reconnaissance and identifying hidden information on a target web server.
points: 40
solves: 40
tags: Pentesting, Recon, Nmap, Footprinting
date: 2025-07-14
comments: true
---

# Assessment Methodologies: Footprinting and Scanning
## Task

In this lab environment, you will be provided with GUI access to a Kali Linux machine. The target machine will be accessible at **http://target.ine.local**.

**Objective:** Perform reconnaissance on the target and capture all the flags hidden within the environment.

**Flags to Capture:**

- **Flag 1**: The server proudly announces its identity in every response. Look closely; you might find something unusual.
- **Flag 2**: The gatekeeper's instructions often reveal what should remain unseen. Don't forget to read between the lines.
- **Flag 3**: Anonymous access sometimes leads to forgotten treasures. Connect and explore the directory; you might stumble upon something valuable.
- **Flag 4**: A well-named database can be quite revealing. Peek at the configurations to discover the hidden treasure.

The best tools for this lab are:

- Nmap
- FTP
- MySQL

## Flag 1

We will start the scan by running the ping command on the target. 

```bash
┌──(root㉿INE)-[~]
└─# ping target.ine.local
PING target.ine.local (192.22.131.3) 56(84) bytes of data.
64 bytes from target.ine.local (192.22.131.3): icmp_seq=1 ttl=64 time=0.107 ms
64 bytes from target.ine.local (192.22.131.3): icmp_seq=2 ttl=64 time=0.076 ms
64 bytes from target.ine.local (192.22.131.3): icmp_seq=3 ttl=64 time=0.069 ms
^C
--- target.ine.local ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2037ms
rtt min/avg/max/mdev = 0.069/0.084/0.107/0.016 ms
```
The scan results show that the target is up hence we can now run a default nmap scan on the target and see what that reveals. 

```bash

┌──(root㉿INE)-[~]
└─# nmap target.ine.local
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-14 12:06 IST
Nmap scan report for target.ine.local (192.22.131.3)
Host is up (0.000030s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
143/tcp  open  imap
993/tcp  open  imaps
3306/tcp open  mysql
MAC Address: 02:42:C0:16:83:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.17 seconds
```

These results show that a lot of ports are open but unfortunately there's nothing that looks like a flag. 
The flag says that the server lists it's identity in every response. Maybe using curl on the http port might yield something? 

Unfortunately it doesn't yield anything useful. I now think maybe I should run a nmap scan but also see the packets in wireshark, specifically in the responses. 


![Image1]({{ site.baseurl }}/assets/ejpt/eJPT-Assessment%20Methodologies-%20Footprinting%20and%20Scanning-%20Assessment%20Metholodologies%20CTF.png)



However the response shows no flag. 
I now run a service version detection and a script scan. 

```bash
(root㉿INE)-[~]
└─# nmap -sV target.ine.local -sC
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-14 12:22 IST
Stats: 0:01:26 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 12:24 (0:00:00 remaining)
Nmap scan report for target.ine.local (192.22.131.3)
Host is up (0.000027s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              22 Oct 28  2024 creds.txt
|_-rw-r--r--    1 0        0              39 Jul 14 06:32 flag.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.22.131.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a5:93:0f:6b:5a:77:f1:77:e8:2e:c9:31:e7:df:66:06 (ECDSA)
|_  256 b6:0d:e4:92:36:30:79:b7:31:91:3b:a0:1f:c1:ee:85 (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: localhost.members.linode.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost
| Not valid before: 2024-10-28T06:10:50
|_Not valid after:  2034-10-26T06:10:50
80/tcp   open  http     Werkzeug/3.0.6 Python/3.10.12
| http-robots.txt: 3 disallowed entries 
|_/photos /secret-info/ /data/
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.6 Python/3.10.12
|     Date: Mon, 14 Jul 2025 06:52:40 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2557
|     Server: FLAG1_8cad490b16dc4571b7701d5b6381913d
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <link rel="shortcut icon" href="#">
|     <title>CTF Challenge</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     margin: 0;
|     padding: 0;
|     background-color: #1c1c1c;
|     color: #fff;
|_Not valid after:  2034-10-26T06:10:50
80/tcp   open  http     Werkzeug/3.0.6 Python/3.10.12
| http-robots.txt: 3 disallowed entries 
|_/photos /secret-info/ /data/
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.6 Python/3.10.12
|     Date: Mon, 14 Jul 2025 06:52:40 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2557
|     Server: FLAG1_8cad490b16dc4571b7701d5b6381913d
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <link rel="shortcut icon" href="#">
|     <title>CTF Challenge</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     margin: 0;
|     padding: 0;
|     background-color: #1c1c1c;
|     color: #fff;
|     background-color: #333;
|     padding: 15px;
|     text-align: center;
|     list-style: none;
|     margin: 0;
|     padding: 0;
|     display:
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.6 Python/3.10.12
|     Date: Mon, 14 Jul 2025 06:52:40 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Server: FLAG1_8cad490b16dc4571b7701d5b6381913d
|     Content-Length: 0
|_    Connection: close
|_http-title: CTF Challenge
|_http-server-header: Werkzeug/3.0.6 Python/3.10.12
143/tcp  open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost
| Not valid before: 2024-10-28T06:10:50
|_Not valid after:  2034-10-26T06:10:50
|_imap-capabilities: ID more capabilities STARTTLS have post-login SASL-IR listed ENABLE IDLE Pre-login OK LOGINDISABLEDA0001 LITERAL+ LOGIN-REFERRALS IMAP4rev1
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost
| Not valid before: 2024-10-28T06:10:50
|_Not valid after:  2034-10-26T06:10:50
|_ssl-date: TLS randomness does not represent time

```
There's also other output but we've gotten our flag in the output. 

## Flag 2

As we saw in the above output. There were 2 interesting things to note. 
These are this:

```
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              22 Oct 28  2024 creds.txt
|_-rw-r--r--    1 0        0              39 Jul 14 06:32 flag.txt


80/tcp   open  http     Werkzeug/3.0.6 Python/3.10.12
| http-robots.txt: 3 disallowed entries 
|_/photos /secret-info/ /data/
```

Now the second task hint was **The gatekeeper's instructions often reveal what should remain unseen. Don't forget to read between the lines.** 
In my opinion that means we should pay attention to port 80 since the third question seems more relevant to FTP. 

I navigate to the the *secret-info* directory in the browser since it seems interesting. That shows us this: 
![Image 2]({{ site.baseurl }}/assets/ejpt/eJPT-Assessment%20Methodologies-%20Footprinting%20and%20Scanning-%20CTF%20Flag%202.png)



Hence this shows us the flag.txt file. We can append it to the directory path and go there and we'll get out flag. 

## Flag 3

Now we will pay attention to the third task . 
It says something about anonymous access and the nmap output confirmed that as well. Now to get anonymous login we run the following commands:
```bash

┌──(root㉿INE)-[~]
└─# ftp target.ine.local
Connected to target.ine.local.
220 (vsFTPd 3.0.5)
Name (target.ine.local:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Basically we do not need a password and we can just login using anonymous as username and no password as the password. 

```bash
ftp> ls
229 Entering Extended Passive Mode (|||46790|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              22 Oct 28  2024 creds.txt
-rw-r--r--    1 0        0              39 Jul 14 06:32 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||65046|)
150 Opening BINARY mode data connection for flag.txt (39 bytes).
100% |****************************************************************************************************************|    39      264.48 KiB/s    00:00 ETA
226 Transfer complete.
39 bytes received in 00:00 (54.48 KiB/s)
ftp> get creds.txt
local: creds.txt remote: creds.txt
229 Entering Extended Passive Mode (|||26523|)
150 Opening BINARY mode data connection for creds.txt (22 bytes).
100% |****************************************************************************************************************|    22      190.12 KiB/s    00:00 ETA
226 Transfer complete.
22 bytes received in 00:00 (26.32 KiB/s)
ftp> exit
221 Goodbye.

```

We can then navigate within the machine and use the get command to get the flag.txt and creds.txt file. It will be downloaded to our home directory from where we can cat out the flag.txt file. 

## Flag 4
The fourth flag seems to be related to MySQL. However before we get to that I pipe out the output of the creds.txt file we got earlier. It shows this

```bash
┌──(root㉿INE)-[~]
└─# cat creds.txt                                                                                                                                            
db_admin:password@123

```
Based on this it gives us a username and password which seems to be credentials to the MySQL Database. 

The MySQL part of the nmap output was as follows:
```bash
3306/tcp open  mysql    MySQL 8.0.39-0ubuntu0.22.04.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=MySQL_Server_8.0.39_Auto_Generated_Server_Certificate
| Not valid before: 2024-10-28T06:11:13
|_Not valid after:  2034-10-26T06:11:13
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.39-0ubuntu0.22.04.1
|   Thread ID: 56
|   Capabilities flags: 65535
|   Some Capabilities: SwitchToSSLAfterHandshake, SupportsCompression, Speaks41ProtocolOld, InteractiveClient, SupportsTransactions, IgnoreSigpipes, FoundRows, LongPassword, DontAllowDatabaseTableColumn, SupportsLoadDataLocal, ConnectWithDatabase, Support41Auth, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, LongColumnFlag, ODBCClient, SupportsAuthPlugins, SupportsMultipleResults, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: \x03.\x01^km_N\x113&4"Od\x05\x05.j\x18
|_  Auth Plugin Name: caching_sha2_password

```

That does not seem helpful but anyways we go to the following [page](https://hackviser.com/tactics/pentesting/services/mysql) to look up more information on MySQL. 
This tells us how to connect to the database. 
```bash
┌──(root㉿INE)-[~]
└─# mysql -u db_admin -h target.ine.local -p                                                                                                                 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 67
Server version: 8.0.39-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```
This lets us in. Now we need to find the flag. 
Looking back at that article we find out that we can use the command `SHOW DATaBASES;` to list all databases. Here I was assuming that we'd need to find out the flag from a database but just running the command gives us this output: 
```MySQL
MySQL [(none)]> SHOW DATABASES;
+----------------------------------------+
| Database                               |
+----------------------------------------+
| FLAG4_b989257b04b542fe8c826bdda70cb499 |
| information_schema                     |
| mysql                                  |
| performance_schema                     |
| sys                                    |
+----------------------------------------+
5 rows in set (0.003 sec)


```

That concludes the ctf. 