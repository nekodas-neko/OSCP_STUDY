---
tags:
  - enumeration
  - gobuster
  - phase/enumeration
  - web
---

# Directory Brute Force with Gobuster

> [!tip] Quick Reference — Gobuster
> | Goal | Command |
> |------|---------|
> | Dir brute force | `gobuster dir -u http://<IP>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt` |
> | With extensions | `gobuster dir -u http://<IP>/ -w common.txt -x php,txt,html,bak` |
> | Find documents by type (e.g. PDFs) | `gobuster dir -u http://<IP>/ -w common.txt -x pdf` |
> | Vhost fuzzing | `gobuster vhost -u http://<domain>/ -w subdomains.txt --append-domain` |
> | DNS subdomains | `gobuster dns -d <domain> -w subdomains.txt` |
> | With auth | `gobuster dir -u http://<IP>/ -w common.txt -U admin -P password` |
> | Quiet + output | `gobuster dir -u http://<IP>/ -w common.txt -q -o gobuster.txt` |

## Decision Tree

```
Web server found?
├── Start with common wordlist
│   └── gobuster dir -u http://<IP>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,txt
├── Found /admin, /login, /api etc → investigate manually
├── Got a domain name (not just IP)?
│   └── Try vhost enumeration
│       └── gobuster vhost -u http://<domain>/ -w subdomains-top1million-5000.txt --append-domain
├── Interesting directory found → recurse into it
│   └── gobuster dir -u http://<IP>/<dir>/ -w big.txt -x php,txt
└── API endpoint found?
    └── gobuster dir -u http://<IP>/api/ -w api-endpoints.txt
```

## Useful Wordlists (SecLists)
- `Discovery/Web-Content/common.txt` — quick, covers most basics
- `Discovery/Web-Content/directory-list-2.3-medium.txt` — thorough
- `Discovery/Web-Content/raft-large-files.txt` — file-focused
- `Discovery/DNS/subdomains-top1million-5000.txt` — vhost/dns

## Visual Flow

```mermaid
flowchart TD
    A[Web server found] --> B["gobuster dir -u http://IP -w common.txt"]
    B --> C{Status codes returned?}
    C -->|"200/301/302 e.g. /admin (301)"| D[Visit + inspect those paths]
    C -->|nothing found| E["Add extensions: -x php,txt,html,bak"]
    E --> F[Still nothing? Use bigger wordlist directory-list-2.3-medium.txt]
    D --> G{Got a hostname, not just IP?}
    G -->|yes| H["gobuster vhost -u http://domain -w subdomains.txt --append-domain"]
    D --> I[Interesting dir? Recurse into it]
```

> [!success] What success looks like
> Gobuster prints found paths with their status, e.g. `/css (Status: 301)`, `/index.php (Status: 302) [--> /login.php]`, `/uploads (Status: 301)`. The 200/301/302 entries are real pages — those are your next targets.

> [!danger] Common errors
> - `wordlist file does not exist` → point `-w` at a real path, e.g. `/usr/share/wordlists/dirb/common.txt` or a SecLists file.
> - HTTPS target: `unknown certificate` / TLS error → add `-k` to skip cert checks.
> - Every path returns the same status (false positives) → blacklist it, e.g. `-b 301,404`, or filter by size with `--exclude-length`.
> - Missing scheme error → include `http://` (or `https://`) in the `-u` URL on newer Gobuster versions.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> **Directory brute forcing** guesses hidden pages and folders from a wordlist, one request per word. Most web wins start by finding an unlinked page like `/admin` or `/backup` that the site never links to.

## Finding documents for metadata OSINT

`-x` isn't just for web shells and backup files — filtering to a document type (`pdf`, `docx`, `xlsx`) turns gobuster into a **document-discovery tool** for client-side recon:

```bash
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirb/common.txt -x pdf
```

If a common wordlist doesn't surface anything, step up to a filename-focused list:
```bash
gobuster dir -u http://<IP>/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -x pdf
```

Any PDF this finds is a candidate for `exiftool` — check every field, not just Author/Producer, since useful data (or a lab flag) can hide in `Keywords`, `Comment`, or `Subject`. See [[Information gathering]] for the full metadata-analysis workflow this feeds into.

> [!warning] This is the noisy alternative
> Unlike a `site:target.com filetype:pdf` Google dork, this sends real requests straight to the target and shows up in their logs. Use it when a document isn't linked anywhere Google could have indexed it — otherwise, dork first.

## Resources
- [HackTricks — Web Fuzzing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web#directories-and-files)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [ffuf (faster alternative)](https://github.com/ffuf/ffuf)


Once we have discovered an application running on a web server, our next step is to map all its publicly accessible files and directories. To do this, we would need to perform multiple queries against the target to discover any hidden paths. Gobuster is a tool (written in Go language) that can help us with this sort of enumeration. It uses wordlists to discover directories and files on a server through brute forcing.
[https://www.kali.org/tools/gobuster/](https://www.kali.org/tools/gobuster/)

> [!warning] Caution
> Because it brute forces one request per wordlist entry, Gobuster is noisy and generally unsuitable for stealth engagements.

Gobuster supports various enumeration modes, including fuzzing and DN
S, but for now, we’ll focus solely on the dir mode, which enumerates files and directories. 
We need to specify the target IP using the

## -u

parameter and a wordlist with

## -w

. The default running threads are 10; we can reduce the amount of traffic by setting a lower number via the

## -t

parameter.



gobuster dir -u 192.168.165.16 -w /usr/share/wordlists/dirb/common.txt -b 301 -t 5   

> Used -b 301 to ignore results for 301's

Using the `common.txt` wordlist found ten resources. Four (e.g. `/.htaccess`, `/.htpasswd`, `/server-status`) returned `403` and are inaccessible; the remaining six — `/css`, `/db`, `/images`, `/index.php` (302 → `/login.php`), `/js`, `/uploads` — are accessible and worth investigating.

```sh
gobuster dir -u 192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5
```

---
%% graph-links %%
## Related
- [[Security Testing with Burp Suite]]
- [[Inspecting HTTP Response Headers and Sitemaps]]
- [[Local file inclusion (LFI)]]

> [!info] Navigation
> Section: [[Web Applications/Application Assesment Tools/_index|Application Assesment Tools]] · Home: [[🏠 Home]]

