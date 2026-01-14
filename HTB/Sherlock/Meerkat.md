# HackTheBox Sherlock - Meerkat
**Youtube video:** https://youtu.be/1Qq32NW9l-s
---
## Overview & Initial Analysis

**Analyze JSON alerts:**
```bash
# View all alerts
cat meerkat-alerts.json | jq '.[]' | less

# Extract and sort source IPs
jq .[].src_ip meerkat-alerts.json -r | sort | uniq -c | sort -nr > IPs.txt

# Map IPs to alert signatures
jq -r '.[] | "\(.src_ip) \(.alert.signature)"' meerkat-alerts.json | sort | uniq -c | sort -nr

# Extract alert signatures
jq .[].alert.signature meerkat-alerts.json                    # With quotes
jq -r .[].alert.signature meerkat-alerts.json                 # Raw strings
jq -r .[].alert.signature meerkat-alerts.json | sort | uniq -c | sort -nr > signs.txt
```

---
## Wireshark Analysis

**Q1: Application targeted**
- **Answer:** BonitaSoft (rihnolabs)

**Q2: Attacker reconnaissance**
```
# Filter suspicious IP
ip.addr==156...

# Apply HTTP filter and examine packet 2158
# Follow HTTP stream - shows username enumeration and password spraying

# Second attacker IP
ip.addr==138... + HTTP filter
```

**Q3: CVE exploited**
- **Answer:** (rihnolabs related)

**Q4: Endpoint attacked**
- **Answer:** i18ntranslation

**Q5: Number of POST requests**
```
# Filter POST requests in Wireshark
http.request.method==POST
```

**Q6: How many usernames attempted**
```
# Filter POST requests to login endpoint
http.request.uri=="/bonita/loginservice" && http.request.method==POST

# Export packets for analysis
```

**Extract usernames from PCAP:**
```bash
# Export filtered packets, then:
tcpdump -r users.pcapng -A | grep username > users.txt

# Parse unique usernames
cat users.txt | cut -d "=" -f2 | sort | uniq | grep -v username | cut -d "%" -f1 | cut -d "&" -f1 | wc -l
# Result: 56 unique usernames
```
- Examine packet 2918

**Q7: Successful credentials**
- Follow packet 2931 (HTTP 204 response indicates success)
- Save username and password combination

**Q8: Exfiltration filename**
- Packet 3652 - follow stream
- Shows pastes.io upload
- **Answer:** Provide filename from pastes.io

**Q9: SSH key evidence**
- Follow packet 3652
- SSH key visible in stream

**Q10: Persistence mechanism**
```
# Search for SSH-related activity
# Filter: ssh
# Look for: /home/ubuntu/.ssh/authorized_keys
```
- **Answer:** SSH authorized_keys persistence
