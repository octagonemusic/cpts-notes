# Network Enumeration
## Default Nmap Command
```bash
nmap -sV -sC -p- <TARGET_IP>
```
What each flag does:
- `-sC`: Specifies that **Nmap scripts** should be used to try and obtain more detailed information
- `-sV`: Instruct Nmap to perform a **version scan**. This fingerprints the services on the target system and identifies the **service protocol,** **application name**, and **version**
- `-p-`: Instruct Nmap to **scan all 65,535 TCP ports**

## Nmap command to run custom script
```bash
nmap --script <SCRIPT_NAME> -p<PORT> <HOST>
```

## Banner Grabbing
```bash
nc -nv <TARGET_IP> <PORT>
```

## SMB Enumeration
```bash
smbclient -N -L \\\\10.129.250.155
```

```bash
smbclient -U bob \\\\10.129.250.155\\users
```

# Web Enumeration
## GoBuster
### Directory/File Enumeration
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### DNS Enumeration
```bash
gobuster dns --do <TARGET_DOMAIN> -w /usr/share/seclists/Discovery/DNS/namelist.txt 
```
- `--do`: domain
- `-w`: wordlists

## Banner Grabbing
```shell
curl -IL https://www.inlanefreight.com
```
- `-I`: headers only (changes req method from `GET` to `HEAD`)
- `-L`: location (follows http redirects)
**EyeWitness** can be used to take screenshots of target web apps. 

## Whatweb
Extracts version of web servers, supporting frameworks and applications
```bash
whatweb <TARGET_IP>
```

## Other Important Tips for Web Enumeration
- View **Certificates t**o reveal details like email address and company name
- `robots.txt` could provide valuable information such as the location of private files and admin pages
- Check for the **source code** by hitting `[CTRL + U]`



