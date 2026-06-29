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
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> **NSE** = Nmap Scripting Engine: small Lua scripts (in `/usr/share/nmap/scripts`) that do deeper work than a plain port scan, like grabbing HTTP headers or testing for known vulns. The shortcut `-sC` just means "run the default set of safe scripts" — pair it with `-sV` for version detection.

We can use the NSE to launch user-created scripts to automate various scanning tasks. These scripts perform a broad range of functions including DNS enumeration, brute force attacks, and even vulnerability identification. NSE scripts are in the /usr/share/nmap/scripts directory.

> [!note]- Screenshot
> ```
> ‘The Attp-headers script, for example, attempts to connect to the HTTP service on a
> target system and determine the supported headers.
> 
> kali@cali:-$ nmap --script http-headers 192.168.50.6
> 
> Starting Nmap 7.92 ( https: //nmap.org ) at 2022-03-10 13:53 EST
> 
> Nmap scan report for 192.168.50.6
> 
> Host is up (@.145 latency).
> 
> Not shown: 998 closed tcp ports (conn-refused)
> 
> PORT STATE SERVICE
> 
> 22/tcp open ssh
> 
> 80/tcp open http
> 
> | nttp-headers:
> 
> | Date: Thu, 10 Mar 2022 18:53:29 GMT
> 
> | Server: Apache/2.4.41 (Ubuntu)
> 
> | Last-Modified: Thu, 10 Mar 222 18:51:54 GMT
> 
> | ETag: “d1-5d9e1b5371420"
> 
> | Accept-Ranges: bytes
> 
> | Content-Length: 209
> 
> | Vary: Accept-Encoding
> 
> | Connection: close
> 
> | Content-Type: text/html
> 
> |
> 
> IL Request type: HEAD)
> 
> Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds
> 
> Listing 45 - Using nmap's scripting engine (NSE) for OS fingerprinting
> ```


```sh
nmap --script http-headers 192.168.50.6
```


> [!note]- Screenshot
> ```
> To view more information about a script, we can use the --script-help option, which
> displays a description of the script and a URL where we can find more in-depth
> information, such as the script arguments and usage examples.
> 
> kaligcali:-$ map --script-help nttp-headers
> 
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 13:54 EST
> 
> hnetp-headers,
> 
> Categories: discovery safe
> 
> hit tps: //nmap.org/nsedoc/scripts/nttp-headershtal
> 
> Performs a HEAD request for the root folder ("/") of a web server and displays the
> HTTP headers returned.
> sting 46 = Using the -scrpt-help option to view more information about a script
> ```


```sh
nmap --script-help http-headers
```

---
%% graph-links %%
## Related
- [[TCPUDP Port Scanning Theory]]
- [[FingerPrinting with Nmap]]
- [[NMAP]]
- [[SMB Enumeration]]

> [!info] Navigation
> Section: [[Active Information Gathering/Port Scanning with Nmap/_index|Port Scanning with Nmap]] · Home: [[🏠 Home]]

