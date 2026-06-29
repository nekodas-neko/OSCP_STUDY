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

> [!note]- Screenshot
> ```
> Q Tip
> 
> The -Pn option in Nmap disables host discovery (ping scan) and treats all targets as
> online, useful for scanning hosts that might appear to be down because they block
> ICMP requests.
> kaligkali:-$ map 192.168.50.149
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 @5:12 EST
> Nmap scan report for 192.168.509.149
> Host is up (@.10s latency).
> Not shown: 989 closed tcp ports (conn-refused)
> PORT STATE SERVICE
> 53/tcp open domain
> 88/tcp open kerberos-sec
> 135/tep open msrpc
> 239/tep open netbios-ssn
> 389/tep open dap
> 4as/tcp open microsoft-ds
> 464/tcp open kpasswdS
> 593/tep open http-rpc-epmap
> 636/tcp open Idapssi
> 3268/tcp open globalcatLDAP
> 3269/tcp open globalcatLOAPss1
> Nmap done: 1 IP address (1 host up) scanned in 10.95 seconds
> 
> Listing 31 - Scanning an IP forthe 1000 most popular TCP ports
> ```


```sh
nmap 192.168.50.149
```


## Stealth Scanning

The most popular Nmap scanning technique is SYN, or "stealth" scanning. There are many benefits to using a SYN scan and as such, it is the default scan option used when no scan option is specified in an nmap command and the user has the required raw socket privileges.

> [!note]- Screenshot
> ```
> ‘SYN scanning is a TCP port scanning method that involves sending SYN packets to
> various ports on a target machine without completing a TCP handshake. If a TCP port is
> ‘open, a SYN-ACK should be sent back from the target machine, informing us that the
> port is open. At this point, the port scanner does not bother to send the final ACK to
> complete the three-way handshake.
> 
> kaligeali:-$ sudo nmap -ss 192.168.50.149
> 
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 06:31 EST
> 
> Nnap scan report for 192.168.50.149
> 
> Host is up (@.11s latency).
> 
> Not shown: 989 closed tcp ports (reset)
> 
> PORT STATE SERVICE
> 
> 53/tep open domain
> 
> 88/tcp open Kerberos-sec
> 
> 135/tcp_ open msrpc
> 
> 139/tcp_ open netbios-ssn
> 
> 389/tcp open Idap
> 
> 4as/tcp open microsoft-ds
> 
> 464/tcp open kpassud5
> 
> 593/tcp open http-rpc-epmap
> 
> 636/tcp open Idapss1
> 
> 3268/tcp open globalcat LDAP
> 
> 3269/tcp open globalcatLDAPss1
> 
> Listing 34 - Using nmap to perform a SYN scan
> 
> Because the three-way handshake is never completed, the information is not passed to
> the application layer and as a result, will not appear in any application logs. A SYN scan
> is also faster and more efficient because fewer packets are sent and received.
> ```


```sh
sudo nmap -sS 192.168.50.149
```


## TCP Connect Scanning

When a user running nmap does not have raw socket privileges, Nmap will default to the TCP connect scan technique. Since an Nmap TCP connect scan makes use of the Berkeley sockets API to perform the three-way handshake, it does not require elevated privileges. However, because Nmap must wait for the connection to complete before the API will return the status of the connection, a TCP connect scan takes much longer to complete than a SYN scan.

> [!note]- Screenshot
> ```
> We may occasionally need to perform a connect scan using nmap, such as when
> ‘scanning via certain types of proxies. We can use the -st option to start a connect scan.
> 
> kaligkali:~$ nmap -sT 192.168.50.149
> 
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 06:44 EST
> 
> Nmap scan report for 192.168.50.149
> 
> Host is up (0.11 latency).
> 
> Not shown: 989 closed tcp ports (conn-refused)
> 
> PORT STATE SERVICE
> 
> 53/tcp open domain
> 
> 88/tcp open _kerberos-sec
> 
> 135/tcp open msrpc
> 
> 139/tcp_ open netbios-ssn
> 
> 389/tcp open 1dap
> 
> 4a5/tcp open microsoft-ds
> 
> 464/tcp open kpasswdS
> 
> '593/tcp open http-rpc-epmap
> 
> 636/tcp open Idapss1
> 
> 3268/tcp open globalcatLOAP
> 
> +3269/tcp open globalcatLDAPss1
> 
> Listing 35 - Using nmap to perform a TCP connect scan
> ```


```sh
nmap -sT 192.168.50.149
```

The output shows that the connect scan resulted in a few open services that are only active on the Windows-based host, especially Domain Controllers, as we'll cover shortly. One major takeaway, even from this simple scan, is that we can already infer the underlying OS and role of the target host.

## UDP


## Scan


> [!note]- Screenshot
> ```
> When performing a UDP scan, Nmap will use a combination of two different methods to
> determine if a port is open or closed. For most ports, it will use the standard "ICMP port
> unreachable" method described earlier by sending an empty packet to a given port.
> However, for common ports, such as port 161, which is used by SNMP, it will send a
> protocol-specific SNMP packet to get a response from an application bound to that port.
> To perform a UDP scan, we'll use the -su option, with sudo required to access raw
> sockets.
> 
> kaligeali:~$ sudo nmap -sU 192.168.509.149
> 
> ‘Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-04 11:46 EST
> 
> Nnap scan report for 192.168.131.149
> 
> Host is up (@.11s latency).
> 
> Not shown: 97 closed udp ports (port-unreach)
> 
> Port STATE ‘SERVICE
> 
> 123/udp open. ntp
> 
> (389/udp open Adap
> 
> =) done: 1 IP address (1 host up) scanned in 22.49 seconds
> 
> Listing 36 = Using nmap to perform a UDP scan
> ```


```sh
sudo nmap -sU 192.168.50.149
```


> [!note]- Screenshot
> ```
> ‘The UDP scan (-:u) can also be used in conjunction with a TCP SYN scan (-ss) to build
> a more complete picture of our target.
> 
> kaligkali:~$ sudo nmap -sU -s5 192.168.50.149
> 
> ‘Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 08:16 EST
> 
> Nnap scan report for 192.168.50.149
> 
> Host is up (@.10s latency).
> 
> Not shown: 989 closed tcp ports (reset), 977 closed udp ports (port-unreach)
> 
> PoRT STATE ‘SERVICE
> 
> 53/tcp open ‘domain
> 
> 88/tcp open kerberos-sec
> 
> 335/tcp open msrpc
> 
> 139/tcp open netbios-ssn
> 
> 389/tcp open ldap
> 
> aas/tcp open icrosoft-ds
> 
> ‘aeaytcp open passwd
> 
> 593/tcp open http-rpc-epmap
> 
> 636/tcp open aps
> 
> 3268/tcp open lobalcatLDAP
> 
> 3269/tcp open elobalcatLDAPss1
> 
> 53/udp open ‘domain
> 
> 123/udp open. ntp
> 
> (389/udp open Adap
> 
> Listing 37 - Using nmap to perform a combined UDP and SYN scan
> ```


```sh
sudo nmap -sU -sS 192.168.50.149
```


## Network Sweeping

To deal with large volumes of hosts, or to otherwise try to conserve network traffic, we can attempt to probe targets using Network Sweeping techniques in which we begin with broad scans, then use more specific scans against hosts of interest.

> [!note]- Screenshot
> ```
> When performing a network sweep with Nmap using the -=n option, the host discovery
> process consists of more than just sending an ICMP echo request. Nmap also sends a
> TCP SYN packet to port 443, a TCP ACK packet to port 80, and an ICMP timestamp
> request to verify whether a host is available.
> 
> ali@calis-$ map ~sn 192.168.50.1-253
> 
> Starting tap 7.92 ( https://nmap.org ) at 2022-03-10 03:19 EST
> 
> map scan report for 192.168.50.6
> 
> Host is up (0.125 latency).
> 
> map scan report for 192.168.50.8
> 
> Host is up (0.125 2atency).
> 
> nap done: 254 IP addresses (13 hosts up) scanned in 3.74 seconds
> 
> eo ee
> ```


```sh
nmap -sn 192.168.50.1-253
```


> [!note]- Screenshot
> ```
> Searching for live machines using the grep command on a standard nmap output can be
> cumbersome. Instead, let's use Nmap's "greppable" output parameter, -0s, to save
> these results in a more manageable format.
> 
> kaligkali:~$ nmap -v -sn 192.168.50.1-253 -0G ping-sweep.txt
> 
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 @3:21 EST
> 
> Initiating Ping Scan at 03:21
> 
> Read data files from: /usr/bin/../share/nmap
> 
> Nmap done: 254 IP addresses (13 hosts up) scanned in 3.74 seconds
> 
> kaligkali:~$ grep Up ping-sweep.txt | cut -" " -f 2
> 
> 192.168.50.6
> 
> 192.168.50.8
> 
> 192.168.50.9
> 
> Listing 39 - Using nmap to perform a network sweep and then using grep to find live hosts
> ```


```sh
nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt
grep Up ping-sweep.txt | cut -d " " -f 2
```


> [!note]- Screenshot
> ```
> We can also sweep for specific TCP or UDP ports across the network, probing for
> common services and ports to locate systems that may be useful or have known
> vulnerabilities. This scan tends to be more accurate than a ping sweep.
> 
> kaligkali:~$ nmap -p 80 192.168.50.1-253 -0G web-sweep. txt
> 
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 03:50 EST
> 
> Nmap scan report for 192.168.50.6
> 
> Host is up (@.11s latency).
> 
> PORT STATE SERVICE
> 
> 80/tcp open http
> 
> Nmap scan report for 192.168.50.8
> 
> Host is up (@.11s latency).
> 
> PORT STATE SERVICE
> 
> 80/tcp closed http
> 
> kaligkali:~$ grep open web-sweep.txt | cut -d" “ -£2
> 
> 192.168.50.6
> 
> 192.168.50.20
> 
> 192.168.5.21
> 
> Listing 40 - Using nmap to scan for web servers using port 80
> ```


```sh
nmap -p 80 192.168.50.1-253 -oG web-sweep.txt
grep open web-sweep.txt | cut -d" " -f2
```

To save time and network resources, we can also scan multiple IPs, probing for a short list of common ports. For example, let's conduct a TCP connect scan for the top 20 TCP ports with the --top-ports option and enable OS version detection, script scanning, and traceroute with -A.

> [!note]- Screenshot
> ```
> kaligkali:~$ nmap -sT -A --top-ports=20 192.168.50.1-253 -0G top-port-sweep. txt
> Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 04:04 EST
> Nmap scan report for 192.168.50.6
> Host is up (0.125 latency).
> PORT STATE SERVICE VERSION
> 21/tcp closed ftp
> 22/tcp openssh OpenSSH 8.2p1 Ubuntu 4ubuntud.3 (Ubuntu Linux; protocol.
> 2.0)
> | ssh-hostkey:
> | 3072 56:57:11:b5:de:1:13:d3:50:88:b8:ab:a9:83:e2:29 (RSA)
> | 256 4F:1d:#2:55:cb:40:e0:76:b4:36:90:19:a2:ba:#0:44 (ECDSA)
> |_ 256 67:46:b3:97:26:a9:e3:a8:4d: eb: 20:b3:9b:8d:7a:32 (£D25519)
> 23/tcp closed telnet
> 25/tcp closed smtp
> 53/tcp closed domain
> 80/tcp open http Apache httpd 2.4.41 ((Ubuntu))
> |_http-server-header: Apache/2.4.41 (Ubuntu)
> |_nttp-title: Under Construction
> 110/tcp closed pop3
> 111/tep closed rpcbind
> Listing 41 - Using nmap to perform a top twenty port scan, saving the output in greppable format
> ```


```sh
nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt
```

The top 20 nmap ports are determined using the /usr/share/nmap/nmap-services file, which uses a simple format of three whitespace-separated columns. The first is the name of the service, the second contains the port number and protocol, and the third is the "port frequency". Everything after the third column is ignored but is typically used for comments as shown by the use of the pound sign (#). The port frequency is based on how often the port was found open during periodic research scans of the internet.

> [!note]- Screenshot
> ```
> kaligkali:~$ cat /usr/share/nap/nmap-services
> finger 79/udp 0.000956
> http —8@/sctp —-@.000000 © # waw-http | ww | World Wide Web HTTP
> http 80/tcp 0.484143 # World Wide Web HTTP
> http 80/udp@.035767_# World Wide Web HTTP
> hosts2-ns 81/tcp 0.012056 # HOSTS2 Name Server
> hosts2-ns 81/udp 0.001005 # HOSTS2 Name Server
> Listing 42 - The nmap-services file showing the open frequency of TCP port 80
> ```

OS fingerprinting can be enabled with the -O option. This feature attempts to guess the target's operating system by inspecting returned packets. This works because operating systems often use slightly different implementations of the TCP/IP stack (such as varying default Time To Live (TTL) values and TCP window sizes), and these slight variances create a fingerprint that Nmap can often identify. This affects how quickly DNS changes propagate — important for attackers or sysadmins.

Nmap will inspect the traffic received from the target machine and attempt to match the fingerprint to a known list. By default, Nmap will display the detected OS only if the retrieved fingerprint is very accurate. Since we want to get a rough idea of the target OS, we include the --osscan-guess option to force Nmap to print the guessed result, even if is not fully accurate.

> [!note]- Screenshot
> ```
> For example, let's consider this simple nmap OS fingerprint scan:
> kaligcalis~$ sudo nnap -0 192.168.50.14 --osscan-guess
> Running (JUST GUESSING): Microsoft Windows 2019|2012|10|2616|2@22|7|2008|8.1 (93%)
> 05 CPE: cpe:/o:microsoft:sindous_server_2012:r2 cpe: ormicrosoft windows 10
> pe: /ormicrosoft:windows_server_2016 cpe:/o:microsoft:windows_7
> cpe: /ormicrosoft:windows_server_2008 cpe: /o:microsoft:windows 8.1
> ‘Aggressive OS guesses: Microsoft Windows Server 2019 (93%), Microsoft Windows Server
> 2012 R2 (89%), Microsoft Windows 10 1909 (88%), Microsoft Windows Server 2012 Data
> Center (88%), Microsoft Windows Server 2016 (88%), Microsoft Windows Server 2622 (87%),
> Microsoft Windows 7 SP1 or Windows Server 2008 (85%), Microsoft Windows 7 Ultimate
> (85%), Microsoft Windows 8.1 (85%)
> No exact OS matches for host (If you know what OS is running on it, see
> nttps://nmap.org/submit/ ).
> 
> “isting 43 = Using nmap for O5 fingerprinting
> ```


```sh
sudo nmap -O 192.168.50.14 --osscan-guess
```

Once we have recognized the underlying operating system, we can go further and identify services running on specific ports by inspecting service banners with -A parameter, which also runs various OS and service enumeration scripts against the target.

> [!note]- Screenshot
> ```
> kaligkali:~$ nmap -sT -A 192.168.50.14
> 
> Nmap scan report for 192.168.50.14
> 
> Host is up (0.125 latency).
> 
> Not shown: 996 closed tcp ports (conn-refused)
> 
> PORT STATE SERVICE VERSION
> 
> 2a/tcp open Ftp?
> 
> | Fingerprint-strings:
> 
> | DNSStatusRequestTCP, DNSVersionBindReqICP, GenericLines, NULL, RPCCheck,
> 
> SSLSessionReq, TLSSessionReq, TerminalServerCookie:
> 
> i} 22@-FileZilla Server 1.2.0
> 
> | Please visit https://Filezilla-project.org/
> 
> | GetRequest:
> 
> i} 22@-FileZilla Server 1.2.0
> 
> | Please visit https://filezilla-project.org/
> 
> | hat are you trying to do? Go away.
> 
> | HTTPoptions, RTSPRequest:
> 
> i} 22@-FileZilla Server 1.2.0
> 
> | Please visit https://Filezilla-project.org/
> 
> i} ‘Wrong command.
> 
> | Help:
> 
> i} 22@-FileZilla Server 1.2.0
> 
> | Please visit https://Filezilla-project.org/
> 
> i} 214-The following commands are recognized.
> 
> i} USER TYPE SYST SIZE RNTO RNFR RMD REST QUIT
> 
> i} HELP XMKD MLST MKD EPSV XCWD NOOP AUTH OPTS DELE
> 
> i} COUP APPE STOR ALLO RETR PWD FEAT CLNT MFMT
> 
> i} ‘MODE XRMD PROT ADAT ABOR XPWD MDTM LIST MLSD PBSZ
> 
> i} NLST EPRT PASS STRU PASV STAT PORT
> 
> I Help ok.
> 
> | ftp-syst:
> 
> |_. SYST: UNIX emulated by FileZilla.
> 
> I ssl-cert: Subject: comonNane=filezilla-server self signed certificate
> 
> | Not valid before: 2022-61-@6T15:37:24
> 
> |_Not valid after: 2023-@1-@7715:42:24
> 
> |_ssl-date: TLS randomness does not represent time
> 
> 135/tcp open msrpc Microsoft Windows RPC
> 
> 139/tcp open netbios-ssn Microsoft Windows netbios-ssn
> 
> 4a5/tcp open microsoft-ds?
> 
> Nmap done: 1 IP address (1 host up) scanned in 55.67 seconds
> 
> sting 44 - Using nmap for banner grabbing and/or service enumeration
> 
> In the above example, we used the -a parameter to run a service scan with extra
> options. If we want to run a plain service nmap scan, we can do it by providing only the
> sv parameter.
> ```


```sh
nmap -sT -A 192.168.50.14
```
