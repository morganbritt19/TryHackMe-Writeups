# [Source THM Room](https://tryhackme.com/room/source)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial [IP]
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2024-02-16 16:10 GMT
Nmap scan report for ip-10-10-185-90.eu-west-1.compute.internal ([IP])
Host is up (0.00054s latency).
Not shown: 998 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
|   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
|_  256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (EdDSA)
10000/tcp open  http    MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
MAC Address: 02:68:61:27:30:B9 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.06 seconds
```

## Enumeration
- When we visit the IP address through a web browser at port 10000, we are greeted with the following screen:

![Prompt to Navigate to Another IP Address](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/ebad64d5-69e9-4ea3-b32e-3d8efae1e6d3)


- Okay, let's follow that redirect and see what we can see.

![Webmin Portal](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/e0a1cd98-9a2a-4fdc-badb-960727f86db0)


- We can do the basic testing things like looking for comments in the page source, inspecting the web page, and searching for default credentials. In the case of Webmin, the default credentials are `admin/admin`, but this doesn't yield any results for us. 
- Since we know the name of the technology we are looking to exploit, we can search around on the internet and see if there are any known and published vulnerabilities. As luck would have it, there is: [Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit) - Linux webapps Exploit (exploit-db.com)](https://www.exploit-db.com/exploits/47330)
- In our Nmap scan, we can see that the version of Webmin that is being used it `1.890`, which means it is likely vulnerable.

## System Hacking
- According to Exploit-DB, there is a Metasploit module for this vulnerability. So we can start up the `msfconsole` and search for exploits related to Webmin.

![Webmin Exploit MSFConsole](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/a809857a-15a2-4829-98aa-66bd90f92268)


- Exploit 7 sounds promising, since its described as a backdoor, which will give us full control over the machine if it works. Let's `use 7` and look at the options that we need to set.

![Exploit Options](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/45323863-b47d-4657-826c-44bf069a0c55)


- When the options are set as needed, we can launch the exploit with `exploit`. If everything is done correctly, we should be able to enter commands. It is important to note that there isn't a prompt like `root#>`. However, we can use commands like `pwd` and `whoami` to see where we are and who we are.

## Privilege Escalation
- Interestingly, utilizing this Metasploit module will give us `root` permissions right away. With that, we can `cat /root/root.txt` and get the root flag. Now we have to backtrack a little to get the user flag. A good way to find what other users are on this machine is to `cat /etc/passwd`, which is exactly what I did.

![etc passwd For Another Username](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/e32078cb-c425-4d91-90f5-d68c8e3fa268)


- At the bottom, we can see that there is a `dark` user. We can try to switch to that user using `su dark`. 
- That seems to work, and now we can `cat ~/user.txt` to get the user flag. 

