---
tags:
  - home
  - index
---

# OSCP Notes — Home

## Attack Phases

| Phase | Notes |
|-------|-------|
| 🔍 Recon | [[Passive Information Gathering/_index\|Passive]] · [[Active Information Gathering/_index\|Active]] |
| 🌐 Web | [[Web Applications/_index\|Web Apps]] · [[SQL Injection Attacks/_index\|SQLi]] |
| 🔎 Vuln Scan | [[Vulnerability Scanning/_index\|Nessus / Nmap]] |
| 📝 Report | [[Report Writing]] |

## Most-Used Quick Refs

- [[Active Information Gathering/Port Scanning with Nmap/_index|Nmap]]
- [[Active Information Gathering/SMB Enumeration|SMB]]
- [[Active Information Gathering/DNS Enumeration|DNS]]
- [[Web Applications/Application Assesment Tools/Directory Brute Force with Gobuster|Gobuster]]
- [[Web Applications/Application Assesment Tools/Security Testing with Burp Suite|Burp Suite]]
- [[Web Applications/Common Web Application Attacks/File Inclusion Vulnerabilities/Local file inclusion (LFI)|LFI]]
- [[Web Applications/Common Web Application Attacks/Command Injection|Command Injection]]
- [[SQL Injection Attacks/Manual SQL exploitation/_index|SQLi]]

## External Resources

| Resource | URL |
|----------|-----|
| HackTricks | https://book.hacktricks.xyz |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| GTFOBins (Linux privesc) | https://gtfobins.github.io |
| LOLBAS (Windows privesc) | https://lolbas-project.github.io |
| RevShells | https://www.revshells.com |
| Exploit-DB | https://www.exploit-db.com |
| PortSwigger Web Academy | https://portswigger.net/web-security |
| CyberChef | https://gchq.github.io/CyberChef |
| IppSec (video search) | https://ippsec.rocks |
| SecLists | https://github.com/danielmiessler/SecLists |

## Exam Day Checklist

```
□ VPN connected
□ Obsidian open on this vault
□ Burp Suite running
□ Terminal with note-taking (screenshots of every flag)
□ Timer set for 23h 45m

For each machine:
□ nmap all ports → service scan → check each service
□ Web? → gobuster + manual enum + burp
□ Screenshot proof.txt + whoami + hostname + IP
□ Document attack chain for report
```
