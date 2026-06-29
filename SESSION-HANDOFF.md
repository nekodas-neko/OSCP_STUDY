# OSCP Vault — Session Handoff

## What we're doing

Converting a CherryTree `.ctb` file (OSCP study notes) into an Obsidian markdown vault. The user is preparing for the OSCP exam and wants a fast, searchable, well-structured reference they can use during the exam on their Kali box. AI is not permitted during the exam so the vault needs to be self-contained and comprehensive.

---

## What's already been done

The initial migration is **complete**. The user has already downloaded `OSCP-Vault.zip` containing 63 `.md` files.

### Migration approach used
- Source: CherryTree SQLite `.ctb` file (62 nodes, 284 images, 89 code boxes)
- Images were extracted from the SQLite blob and OCR'd with `tesseract` — converted to `> [!note]- Screenshot` callout blocks (searchable text instead of dead images)
- Code boxes converted to fenced code blocks with syntax highlighting
- XML rich text parsed and converted to clean markdown
- YAML frontmatter tags added to every file

### Enhancements already added to existing content
Every technical file received:
- **Quick-reference table** at the top (most-used commands at a glance)
- **Decision tree** — "if this then do this" attack flows
- **Resource links** — HackTricks, PayloadsAllTheThings, PortSwigger, GTFOBins, RevShells etc.

Files enhanced so far: Nmap, SMB, DNS, SMTP, SNMP, Gobuster, LFI, RFI, PHP Wrappers, Command Injection, SQLi, XSS, File Upload, Burp Suite, Report Writing.

### Current vault structure
```
OSCP-Vault/
├── 🏠 Home.md                          ← dashboard with exam checklist + all links
├── Report Writing.md
├── Passive Information Gathering/
│   ├── WHOIS Enumeration.md
│   ├── Google Hacking.md
│   ├── Netcraft.md
│   ├── Open-Source Code.md
│   ├── Shodan.md
│   ├── Security Headers and SSLTLS.md
│   └── LLM-Powered Passive Information Gathering.md
├── Active Information Gathering/
│   ├── DNS Enumeration.md
│   ├── TCP/UDP Port Scanning Theory.md
│   ├── Port Scanning with Nmap/
│   │   ├── _index.md
│   │   ├── Nmap Scripting Engine (NSE).md
│   │   └── Window Test-NetConnection.md
│   ├── SMB Enumeration.md
│   ├── SMTP Enumeration.md
│   ├── SNMP Enumeration.md
│   └── Active LLM-Aided Enumeration.md
├── Vulnerability Scanning/
│   ├── Nessus.md
│   └── NMAP.md
├── Web Applications/
│   ├── Application Assessment Tools/
│   │   ├── FingerPrinting with Nmap.md
│   │   ├── Directory Brute Force with Gobuster.md
│   │   ├── Technology Stack Identification with Wappalyzer.md
│   │   └── Security Testing with Burp Suite.md
│   ├── Enumeration/
│   │   ├── Debugging Page Content.md
│   │   ├── Inspecting HTTP Response Headers and Sitemaps.md
│   │   └── Enumerating and Abusing APIs.md
│   ├── Cross-Site Scripting/
│   │   ├── Stored vs Reflected XSS Theory.md
│   │   ├── JavaScript Refresher.md
│   │   ├── Identifying XSS Vulnerabilities.md
│   │   ├── Basic XSS.md
│   │   └── Privilege Escalation via XSS.md
│   └── Common Web Application Attacks/
│       ├── Command Injection.md
│       ├── Directory Traversal/
│       │   ├── Absolute vs relative paths.md
│       │   ├── Identifying and exploiting directory traversals.md
│       │   └── Encoding special characters.md
│       ├── File Inclusion Vulnerabilities/
│       │   ├── Local file inclusion (LFI).md
│       │   ├── Remote file inclusion (RFI).md
│       │   └── PHP wrappers.md
│       └── File Upload Vulnerabilities/
│           ├── Using executable files.md
│           └── Using non-executable files.md
└── SQL Injection Attacks/
    ├── SQL theory and databases.md
    ├── Manual SQL exploitation/
    │   ├── UNION-based payloads.md
    │   ├── Identifying SQLi via error-based payloads.md
    │   ├── Blind SQL injections.md
    │   └── Labs.md
    └── Manual and automated code execution/
        ├── Manual code execution.md
        ├── Automating the attack.md
        └── Labs.md
```

---

## What's NOT in the vault yet (user is still writing these)

The user knows these are missing and will be adding them as they study. When they bring new CherryTree content or ask to add a topic, use the same format:

- Linux Privilege Escalation
- Windows Privilege Escalation  
- Active Directory attacks (Kerberoasting, Pass-the-Hash, BloodHound, Evil-WinRM)
- Password attacks (hashcat, john, hydra, credential spraying)
- Shells & Payloads (msfvenom, netcat, socat, shell stabilisation)
- Port Forwarding & Pivoting (chisel, ligolo-ng, ssh tunnels)
- Client-Side Attacks
- Antivirus Evasion
- Metasploit
- Post-Exploitation (mimikatz, lateral movement)
- Buffer Overflow basics

---

## Format standard (use this for all new content)

Every note follows this structure:

```markdown
---
tags:
  - topic-tag
  - phase/exploitation   # or phase/recon, phase/enumeration
---

# Note Title

> [!tip] Quick Reference — Topic
> | Goal | Command |
> |------|---------|
> | ... | `command here` |

## Decision Tree

\`\`\`
If X → do Y
├── Sub case → command
└── Sub case → command
\`\`\`

## [Content from user's notes]

## Resources
- [HackTricks — Topic](https://book.hacktricks.xyz/...)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/...)
```

### Tag convention
- `phase/recon`, `phase/enumeration`, `phase/exploitation`, `phase/post-exploitation`
- Tool tags: `nmap`, `gobuster`, `burp-suite`, `sqlmap`, `metasploit` etc.
- Vuln tags: `sqli`, `xss`, `lfi`, `rfi`, `rce`, `command-injection`, `file-upload` etc.
- `lab` / `exam-practice` for lab writeup nodes

---

## Key external resources to link in notes

| Resource | URL |
|----------|-----|
| HackTricks | https://book.hacktricks.xyz |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| GTFOBins | https://gtfobins.github.io |
| LOLBAS | https://lolbas-project.github.io |
| RevShells | https://www.revshells.com |
| Exploit-DB | https://www.exploit-db.com |
| PortSwigger Web Academy | https://portswigger.net/web-security |
| CyberChef | https://gchq.github.io/CyberChef |
| IppSec (HackTheBox video search) | https://ippsec.rocks |
| SecLists | https://github.com/danielmiessler/SecLists |
| PentestMonkey Cheatsheets | https://pentestmonkey.net/cheat-sheet |

---

## How to continue

The user will either:
1. **Paste new CherryTree content** (raw text or XML) → convert to the format above and give them back a `.md` file
2. **Ask to add a missing topic** → write a new note from scratch in the format above
3. **Ask to improve an existing note** → enhance it with better decision trees, more commands, more resources

The vault lives locally on their Kali box at whatever path they unzipped to. They are not using git sync — just local markdown files opened in Obsidian.
