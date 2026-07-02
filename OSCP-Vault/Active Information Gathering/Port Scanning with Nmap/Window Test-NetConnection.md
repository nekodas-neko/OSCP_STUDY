---
tags:
  - phase/enumeration
  - enumeration
  - nmap
---

# Window:  Test-NetConnection

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

## Visual Flow

```mermaid
flowchart TD
    A["On a Windows host, no Kali/nmap"] --> B{"Check one port?"}
    B -->|Yes| C["Test-NetConnection -Port 445 IP"]
    B -->|Scan a range| D["1..1024 | % { TcpClient.Connect(IP,$_) ... }"]
    C --> E{"TcpTestSucceeded?"}
    E -->|True| F[🏁 Port open]
    E -->|False| G[Port closed or filtered]
    D --> F
```

> [!success] What success looks like
> The output ends with `TcpTestSucceeded : True` — that line is the whole point and means the TCP port is open. The 1..1024 one-liner prints `TCP port 88 is open` for each reachable port.

> [!danger] Common errors
> - `TcpTestSucceeded : False` with a ping reply → the host is up but that specific port is closed/filtered; try another port.
> - `WARNING: TCP connect ... failed` and it hangs → no route or firewall dropping; this is the normal "closed" case, not a tool bug.
> - One-liner floods red errors → that is expected for closed ports; the `2>$null` at the end is what suppresses them, so keep it.
> - `Test-NetConnection : The term 'Test-NetConnection' is not recognized` → the cmdlet only ships with PowerShell 4.0+ (Windows 8.1/Server 2012 R2 and later); on older boxes, fall back to the raw `TcpClient` one-liner, `Test-Connection` (ping only), or `telnet <IP> <port>`.
> - `Test-NetConnection` takes several seconds per port and feels slow → it also runs a traceroute-style path test by default; add `-WarningAction SilentlyContinue` and prefer the `TcpClient` loop for scanning many ports.
> - Command runs but nothing prints for the `TcpClient` one-liner even on ports you know are open → PowerShell's execution/language mode restrictions (Constrained Language Mode) can block `.Connect()`; check with `$ExecutionContext.SessionState.LanguageMode`.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> `Test-NetConnection` is the built-in PowerShell way to check a port when you are stuck on a Windows box with no nmap installed. It only checks **one port at a time**, so to scan many ports you fall back to the `TcpClient` one-liner. Always read the final `TcpTestSucceeded` field — ignore the ICMP/ping lines above it.

## Resources
- [HackTricks — Nmap](https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/nmap-cheatsheet)
- [Nmap NSE scripts list](https://nmap.org/nsedoc/)


The Test-NetConnection function checks if an IP responds to ICMP and whether a specified TCP port on the target host is open.

For instance, from the Windows 11 client, we can verify if the SMB port 445 is open on a domain controller, as follows:



The returned value in the TcpTestSucceeded parameter indicates that port 445 is open.

```sh
Test-NetConnection -Port 445 192.168.50.151
```


> [!info] To scan a range of ports, loop over `1..1024` and instantiate a raw `Net.Sockets.TcpClient` socket per port — this avoids the extra traffic `Test-NetConnection` sends. Each reachable port prints `TCP port <n> is open`; `2>$null` suppresses the errors from closed ports.

```sh
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
```

> [!tip] More PowerShell-native recon one-liners
> | Goal | Command |
> |------|---------|
> | Detailed single-port check | `Test-NetConnection -ComputerName <IP> -Port 445 -InformationLevel Detailed` |
> | Check the well-known ports in one shot | `Test-NetConnection -ComputerName <IP> -CommonTCPPort SMB` |
> | Suppress noisy DNS-resolution delay | `Test-NetConnection -ComputerName <IP> -Port 445 -WarningAction SilentlyContinue` |
> | ICMP-only reachability check | `Test-Connection -ComputerName <IP> -Count 2` |
> | Resolve a name (nslookup equivalent) | `Resolve-DnsName <hostname>` |
> | Quiet TCP test (bool only, scriptable) | `Test-NetConnection -ComputerName <IP> -Port 445 -InformationLevel Quiet` |

---
%% graph-links %%
## Related
- [[TCPUDP Port Scanning Theory]]
- [[Nmap Scripting Engine (NSE)]]

> [!info] Navigation
> Section: [[Active Information Gathering/Port Scanning with Nmap/_index|Port Scanning with Nmap]] · Home: [[🏠 Home]]

