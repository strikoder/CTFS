# Cap - HTB Walkthrough
# Video: https://youtu.be/AuwYzh-vI_4

```bash

#############################################
# 1. HTTP: bulk download
#############################################
# downloads /download/-1 .. /download/100
for i in $(seq -1 100); do wget "http://10.10.10.245/download/$i"; done

#############################################
# 2. FTP: connect to target
#############################################
# replace $user and $IP
ftp $user@$IP

#############################################
# 3. SSH: connect to target
#############################################
# replace $user and $IP
ssh $user@$IP

#############################################
# 4. Local: find files with capabilities
#############################################
# recursive scan from root (suppress errors)
getcap -r / 2>/dev/null
```
