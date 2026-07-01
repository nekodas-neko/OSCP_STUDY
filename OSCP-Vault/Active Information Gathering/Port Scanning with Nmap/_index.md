---
tags:
  - enumeration
  - nmap
  - phase/enumeration
---

# Port Scanning with Nmap
%% moc %%
> [!abstract] Map of Content
> Find open ports → version-scan → hand off to per-service enumeration.
> 
> ⬆ Up: [[Active Information Gathering/_index|Active Information Gathering]]

## Notes
- [[Nmap Scripting Engine (NSE)]]
- [[Window Test-NetConnection]]

## Related Sections
- [[Active Information Gathering/_index|Active Information Gathering]]
- [[Vulnerability Scanning/_index|Vulnerability Scanning]]

---

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


Nmap (written by Gordon Lyon, aka Fyodor) is one of the most popular, versatile, and robust port scanners available. It has been actively developed for over two decades and offers numerous features beyond port scanning.

Some of the Nmap example scans we'll cover in this Module are run using sudo. This is because quite a few Nmap scanning options require access to raw sockets, which in turn require root privileges. Raw sockets allow for surgical manipulation of TCP and UDP packets. Without access to raw sockets, Nmap is limited, as it falls back to crafting packets by using the standard Berkeley socket API.

> [!tip] -Pn
> The `-Pn` option disables host discovery (the ping scan) and treats all targets as online. Useful for hosts that appear down because they block ICMP.

A default `nmap <IP>` scans the 1000 most popular TCP ports. Against a Domain Controller this reveals services like 53, 88 (kerberos), 135, 139, 389, 445, 636, 3268/3269:

```sh
nmap 192.168.50.149
```


## Stealth Scanning

The most popular Nmap scanning technique is SYN, or "stealth" scanning. There are many benefits to using a SYN scan and as such, it is the default scan option used when no scan option is specified in an nmap command and the user has the required raw socket privileges.

> [!info] How SYN scanning works
> A SYN scan sends a SYN to each port but never completes the handshake. An open port replies with SYN-ACK, then the scanner drops the connection instead of sending the final ACK. Because the handshake never finishes, nothing reaches the application layer, so the scan rarely appears in application logs — and it's faster since fewer packets are exchanged.

Run a SYN scan with `-sS` (requires raw-socket privileges, hence `sudo`):

```sh
sudo nmap -sS 192.168.50.149
```


## TCP Connect Scanning

When a user running nmap does not have raw socket privileges, Nmap will default to the TCP connect scan technique. Since an Nmap TCP connect scan makes use of the Berkeley sockets API to perform the three-way handshake, it does not require elevated privileges. However, because Nmap must wait for the connection to complete before the API will return the status of the connection, a TCP connect scan takes much longer to complete than a SYN scan.

Use `-sT` for a full TCP connect scan. It needs no special privileges and works through proxies, but is slower because it completes the whole handshake:

```sh
nmap -sT 192.168.50.149
```

The output shows that the connect scan resulted in a few open services that are only active on the Windows-based host, especially Domain Controllers, as we'll cover shortly. One major takeaway, even from this simple scan, is that we can already infer the underlying OS and role of the target host.

## UDP


## Scan


> [!info] How UDP scanning works
> Nmap uses two methods for UDP. For most ports it sends an empty packet and infers state from an "ICMP port unreachable" reply. For common ports like 161/SNMP it sends a protocol-specific probe to elicit a response from the listening application.

Run a UDP scan with `-sU` (`sudo` required for raw sockets):

```sh
sudo nmap -sU 192.168.50.149
```


Combine a UDP scan (`-sU`) with a SYN scan (`-sS`) to cover both protocols in one run and build a fuller picture of the target:

```sh
sudo nmap -sU -sS 192.168.50.149
```


## Network Sweeping

To deal with large volumes of hosts, or to otherwise try to conserve network traffic, we can attempt to probe targets using Network Sweeping techniques in which we begin with broad scans, then use more specific scans against hosts of interest.

> [!info] What -sn does
> A ping sweep (`-sn`) does more than send an ICMP echo request: Nmap also sends a TCP SYN to port 443, a TCP ACK to port 80, and an ICMP timestamp request to decide whether a host is up.

Sweep a range for live hosts:

```sh
nmap -sn 192.168.50.1-253
```


Save the sweep in greppable format (`-oG`) so live hosts are easy to extract with `grep`/`cut`:

```sh
nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt
grep Up ping-sweep.txt | cut -d " " -f 2
```


Sweep a specific port across the network to find hosts running a given service (here web servers on port 80). A single-port sweep tends to be more accurate than a ping sweep:

```sh
nmap -p 80 192.168.50.1-253 -oG web-sweep.txt
grep open web-sweep.txt | cut -d" " -f2
```

To save time and network resources, we can also scan multiple IPs, probing for a short list of common ports. For example, let's conduct a TCP connect scan for the top 20 TCP ports with the --top-ports option and enable OS version detection, script scanning, and traceroute with -A.

Scan the 20 most common ports across a range with `--top-ports=20`, adding `-A` for OS/version detection, NSE scripts, and traceroute:

```sh
nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt
```

The top 20 nmap ports are determined using the /usr/share/nmap/nmap-services file, which uses a simple format of three whitespace-separated columns. The first is the name of the service, the second contains the port number and protocol, and the third is the "port frequency". Everything after the third column is ignored but is typically used for comments as shown by the use of the pound sign (#). The port frequency is based on how often the port was found open during periodic research scans of the internet.

> [!example] nmap-services format
> Each line is `service  port/proto  frequency  # comment`. The frequency is how often the port was found open in Internet-wide research scans — this is what `--top-ports` ranks by.
> ```
> http   80/tcp   0.484143   # World Wide Web HTTP
> ```

OS fingerprinting can be enabled with the -O option. This feature attempts to guess the target's operating system by inspecting returned packets. This works because operating systems often use slightly different implementations of the TCP/IP stack (such as varying default Time To Live (TTL) values and TCP window sizes), and these slight variances create a fingerprint that Nmap can often identify. This affects how quickly DNS changes propagate — important for attackers or sysadmins.

Nmap will inspect the traffic received from the target machine and attempt to match the fingerprint to a known list. By default, Nmap will display the detected OS only if the retrieved fingerprint is very accurate. Since we want to get a rough idea of the target OS, we include the --osscan-guess option to force Nmap to print the guessed result, even if is not fully accurate.

Fingerprint the OS with `-O`, adding `--osscan-guess` to print the best guess even when there's no exact match. Output ranks candidates by confidence (e.g. Windows Server 2019 at 93%):

```sh
sudo nmap -O 192.168.50.14 --osscan-guess
```

Once we have recognized the underlying operating system, we can go further and identify services running on specific ports by inspecting service banners with -A parameter, which also runs various OS and service enumeration scripts against the target.

Use `-A` to grab service banners and run enumeration scripts. Here it identifies the FTP service as `FileZilla Server 1.2.0` and pulls its SSL cert and supported commands. For a plain service scan without the extra scripts, use only `-sV`:

```sh
nmap -sT -A 192.168.50.14
```
