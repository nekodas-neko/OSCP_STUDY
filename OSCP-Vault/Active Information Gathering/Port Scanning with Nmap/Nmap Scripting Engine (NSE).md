---
tags:
  - enumeration
  - nmap
  - nse
  - phase/enumeration
---

# Nmap Scripting Engine (NSE)

> [!tip] Quick Reference — Nmap
> | Goal | Command |
> |------|---------|
> | Fast SYN scan all ports | `sudo nmap -sS -p- --min-rate 5000 <IP>` |
> | Full version + scripts | `sudo nmap -sC -sV -p <ports> <IP>` |
> | UDP top 20 | `sudo nmap -sU --top-ports 20 <IP>` |
> | OS detection | `sudo nmap -O --osscan-guess <IP>` |
> | Output all formats | `sudo nmap -sC -sV -oA scan <IP>` |
> | Sweep live hosts | `nmap -sn 10.10.10.0/24` |
> | Run a vuln-scan category | `sudo nmap --script vuln -p <ports> <IP>` |
> | Pass script arguments | `nmap --script smb-enum-shares --script-args smbuser=admin,smbpass=Passw0rd -p 445 <IP>` |
> | Run every script in a category | `nmap --script "http-*" -p 80 <IP>` |
> | Update the script database | `sudo nmap --script-updatedb` |
> | Raise per-script timeout | `nmap --script <name> --script-timeout 30s <IP>` |

## Decision Tree

```
Target identified?
├── Single host → sudo nmap -sS -p- --min-rate 5000 <IP> -oA full
│   └── Ports found → sudo nmap -sC -sV -p <open-ports> <IP> -oA detail
└── Subnet → nmap -sn 10.x.x.0/24 | grep "report for" | awk '{print $5}'
    └── For each live host → repeat single host flow

Service on unusual port?
└── Always run -sV to confirm what's actually listening

Firewall dropping packets?
└── Try -Pn (skip host discovery) and --scan-delay 1s
```

## Resources
- [HackTricks — Nmap](https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/nmap-cheatsheet)
- [Nmap NSE scripts list](https://nmap.org/nsedoc/)

## Visual Flow

```mermaid
flowchart TD
    A[Open port found] --> B{"Need a quick<br/>safe sweep?"}
    B -->|Yes| C["Default scripts<br/>sudo nmap -sC -sV -p port IP"]
    B -->|Targeted| D["Pick a script<br/>nmap --script http-headers IP"]
    D --> E["Unsure what it does?<br/>nmap --script-help name"]
    C --> F[🏁 Versions, headers, vulns]
    D --> F
```

> [!success] What success looks like
> The scan prints normal port lines plus extra indented `|` output from the script, e.g. an `http-headers` run shows `Server: Apache/2.4.41 (Ubuntu)`. That banner/version is what you feed into searchsploit.

> [!danger] Common errors
> - `'http-headers' did not match a category, filename, or directory` → typo in the script name; list options with `ls /usr/share/nmap/scripts/`.
> - Scripts run but return nothing → the service was not actually that protocol; confirm with `-sV` first.
> - `-sC` feels slow / noisy → it runs the whole `default` category; narrow to one script with `--script <name>` on real exams.
> - A single script stalls the whole scan for minutes → some scripts (e.g. `smb-vuln-*`, `vuln` category) have long internal retries; cap it with `--script-timeout 30s` or drop that one script.
> - `--script-args` values with special characters get mangled → wrap the whole argument list in quotes: `--script-args 'smbuser=admin,smbpass=P@ss!'`.
> - `NSOCK ERROR` / script errors about sockets → usually a firewall or IPS resetting connections mid-script; re-run with `-Pn` and/or a lower `--script-timeout`, or accept the port is filtered.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> **NSE** = Nmap Scripting Engine: small Lua scripts (in `/usr/share/nmap/scripts`) that do deeper work than a plain port scan, like grabbing HTTP headers or testing for known vulns. The shortcut `-sC` just means "run the default set of safe scripts" — pair it with `-sV` for version detection.

We can use the NSE to launch user-created scripts to automate various scanning tasks. These scripts perform a broad range of functions including DNS enumeration, brute force attacks, and even vulnerability identification. NSE scripts are in the /usr/share/nmap/scripts directory.

> [!example] The `http-headers` script connects to a target's HTTP service and lists the response headers — useful for identifying the server software:
> ```sh
> nmap --script http-headers 192.168.50.6
> ```
> The script output appears under the open port, e.g. `| Server: Apache/2.4.41 (Ubuntu)` — feed that banner into searchsploit.


> [!example] Use `--script-help` to see a script's description, categories, and a docs URL before running it:
> ```sh
> nmap --script-help http-headers
> ```

> [!example] Scripts often take extra options via `--script-args` — a `key=value,key2=value2` list. Here `smb-enum-shares` is given credentials so it can enumerate shares that reject anonymous/null sessions:
> ```sh
> nmap --script smb-enum-shares --script-args smbuser=admin,smbpass=Passw0rd -p 445 192.168.50.152
> ```

> [!info] Handy scripts by service
> | Service | Script(s) |
> |---------|-----------|
> | FTP (21) | `ftp-anon`, `ftp-syst` |
> | SSH (22) | `ssh2-enum-algos`, `ssh-auth-methods` |
> | HTTP(S) (80/443) | `http-enum`, `http-title`, `http-methods`, `http-headers` |
> | SMB (139/445) | `smb-enum-shares`, `smb-enum-users`, `smb-os-discovery`, `smb-vuln-ms17-010` |
> | SMTP (25) | `smtp-commands`, `smtp-enum-users`, `smtp-open-relay` |
> | SNMP (161) | `snmp-sysdescr`, `snmp-processes`, `snmp-netstat` |
> | MySQL (3306) | `mysql-info`, `mysql-empty-password` |
> | Generic | `vuln` category (`--script vuln`), `vulners` (needs internet + CPE match) |

> [!tip] Nmap's scripts are just categorized Lua files
> Categories like `vuln`, `default`, `discovery`, `auth`, and `brute` group related scripts — run `--script <category>` to fire a whole group at once. `-sC` is shorthand for `--script default`.

---
%% graph-links %%
## Related
- [[TCPUDP Port Scanning Theory]]
- [[FingerPrinting with Nmap]]
- [[NMAP]]
- [[SMB Enumeration]]

> [!info] Navigation
> Section: [[Active Information Gathering/Port Scanning with Nmap/_index|Port Scanning with Nmap]] · Home: [[🏠 Home]]

