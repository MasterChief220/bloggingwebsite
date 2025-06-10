---
layout: writeup
category: eJPT
chall_description: This lab focuses on information gathering and reconnaissance techniques to analyze a target website. Participants will explore various aspects of the website to uncover potential vulnerabilities, sensitive files, and misconfigurations. By leveraging investigative skills, they will learn how to identify critical information that could assist in further penetration testing or exploitation.
points: 50
solves: 50
tags: Pentesting, Recon, Information-Gathering, eLearn-Security
date: 2025-06-10
comments: true
---

# Question
A website is accessible at **http://target.ine.local**. Perform reconnaissance and capture the following flags.

- **Flag 1:** This tells search engines what to and what not to avoid.
- **Flag 2:** What website is running on the target, and what is its version?
- **Flag 3:** Directory browsing might reveal where files are stored.
- **Flag 4:** An overlooked backup file in the webroot can be problematic if it reveals sensitive configuration details.
- **Flag 5:** Certain files may reveal something interesting when mirrored.

## Tools
- Firefox
- Curl
- HTTrack 

# First Flag

**Flag 1: This tells search engines what to and what not to avoid.** 

Now as we learned earlier that is the robots.txt file. Since that file exists on websites to tell search engines what to and what not to index. So I try adding robots.txt to the website path and enter it in the vm. And that gets us our first flag! 

```
ser-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

FLAG1{a9719d23af07465da817b4446e2f7f68} 
```

> One thing to note here is that it's not always that robots.txt would be in that path. Sometimes we will have to look for that in other directories.

# Second Flag

**Flag 2: What website is running on the target, and what is its version?**

Now we need to find the second flag. For that I think we should try running a nmap scan with service version detection. So I'll try running an aggressive nmap scan. 


```bash
sudo nmap -A -T4 target.ine.local                                                                                               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-09 22:05 IST
Nmap scan report for target.ine.local (192.250.249.3)
Host is up (0.000098s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-generator: WordPress 6.5.3 - FL@G2{c8e06611d99a4e90b8ae70747d8fa8dc}
|_http-title: INE
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
MAC Address: 02:42:C0:FA:F9:03 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.10 ms target.ine.local (192.250.249.3)
```

And that has yielded us with our second flag. 

# Third Flag
**Flag 3: Directory browsing might reveal where files are stored.**

Now we need to look within it's directories for where files are stored. 
We could just use bruteforcing with gobuster or disbuster but the tools we were told to use did not include that. Hence I will try to use HTTrack to download the site and mirror it. 

So I run the command 
`httrack http://target.ine.local -O ./mirror`
This will mirror the sites files and directory structure to the mirror folder. Now the flag asks us to look for a folder where files might be stored but looking through the folders in the mirror directory I don't see any place like that. 

I decide to fall back to brute-forcing since maybe httrack missed something since it's more passive. So I run gobuster which yields the following:

```bash
gobuster dir -u http://target.ine.local -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://target.ine.local
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 281]
/.htaccess            (Status: 403) [Size: 281]
/.htpasswd            (Status: 403) [Size: 281]
/index.php            (Status: 301) [Size: 0] [--> http://target.ine.local/]
/robots.txt           (Status: 200) [Size: 108]
/server-status        (Status: 403) [Size: 281]
/wp-admin             (Status: 301) [Size: 323] [--> http://target.ine.local/wp-admin/]
/wp-content           (Status: 301) [Size: 325] [--> http://target.ine.local/wp-content/]
/wp-includes          (Status: 301) [Size: 326] [--> http://target.ine.local/wp-includes/]
/xmlrpc.php           (Status: 405) [Size: 42]

===============================================================

```



Now I know wordpress files usually have an uploads folder in wp-content so we will try to navigate to that first


```bash
gobuster dir -u http://target.ine.local/wp-content -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://target.ine.local/wp-content
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 281]
/.htaccess            (Status: 403) [Size: 281]
/.htpasswd            (Status: 403) [Size: 281]
/index.php            (Status: 200) [Size: 0]
/plugins              (Status: 301) [Size: 333] [--> http://target.ine.local/wp-content/plugins/]
/themes               (Status: 301) [Size: 332] [--> http://target.ine.local/wp-content/themes/]
/uploads              (Status: 301) [Size: 333] [--> http://target.ine.local/wp-content/uploads/]
Progress: 4614 / 4615 (99.98%)

```

Well we can see a folder named uploads which was not visible while using httracks. We navigate to that in the browser and we find the flag.txt file there. 


# Flag 4

**Flag 4: An overlooked backup file in the webroot can be problematic if it reveals sensitive configuration details**

Now we need to look for a backup file. Backup files are usually with the bak extension. I'll try to use gobuster for this, this time as well and just use bak for now by running

`gobuster dir -u http://target.ine.local -w /usr/share/wordlists/dirb/common.txt -x bak
`
```bash
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 281]
/.htaccess            (Status: 403) [Size: 281]
/.hta.bak             (Status: 403) [Size: 281]
/.htpasswd            (Status: 403) [Size: 281]
/.htaccess.bak        (Status: 403) [Size: 281]
/.htpasswd.bak        (Status: 403) [Size: 281]
/index.php            (Status: 301) [Size: 0] [--> http://target.ine.local/]
/robots.txt           (Status: 200) [Size: 108]
/server-status        (Status: 403) [Size: 281]
/wp-admin             (Status: 301) [Size: 323] [--> http://target.ine.local/wp-admin/]
/wp-content           (Status: 301) [Size: 325] [--> http://target.ine.local/wp-content/]
/wp-config.bak        (Status: 200) [Size: 3438]
/wp-includes          (Status: 301) [Size: 326] [--> http://target.ine.local/wp-includes/]
Progress: 9228 / 9230 (99.98%)
/xmlrpc.php           (Status: 405) [Size: 42]

```



We found all these files and any one of them might have our flag. However the question mentions sensitive configuration details so it's likely a config file. Imo it's the wp-config.bak file so I will download it first using `curl http://target.ine.local/wp-config.bak -o wp-config.bak`. After that I run cat on it and inspect the file and just to my luck it reveals the fourth flag. 

# Flag 5

**Flag 5: Certain files may reveal something interesting when mirrored. ** 

Now the main thing here to note is the mirrored keyword. I remember httrack having a mirror parameter in it so I will go back to that. In the default directory for target.ine.local in the mirror directory I see a php file named **xmlrpc0db0.php**. What's a php file doing here? I open the file and that has the fifth flag: 
```php
<?xml version="1.0" encoding="UTF-8"?><rsd version="1.0" xmlns="http://archipelago.phrasewise.com/rsd">
	<service>
		<engineName>WordPress</engineName>
		<engineLink>https://wordpress.org/</engineLink>
		<homePageLink>http://target.ine.local</homePageLink>
		<apis>
			<api name="WordPress" blogID="1" preferred="true" apiLink="http://target.ine.local/xmlrpc.php" />
			<api name="Movable Type" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
			<api name="MetaWeblog" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
			<api name="Blogger" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
      			<api name="FLAG5{d61e9108373f49a19d3374a642a15133}" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
				<api name="WP-API" blogID="1" preferred="false" apiLink="http://target.ine.local/index.php/wp-json/" />
			</apis>
	</service>
</rsd>
```

With that the Assessment Methodologies course is completed. 
