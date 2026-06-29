---
tags:
  - dns
  - enumeration
  - phase/enumeration
---

# DNS Enumeration

> [!tip] Quick Reference — DNS
> | Goal | Command |
> |------|---------|
> | Basic lookup | `host <domain>` |
> | Find name servers | `host -t ns <domain>` |
> | Find mail servers | `host -t mx <domain>` |
> | Zone transfer attempt | `host -l <domain> <nameserver>` |
> | dnsrecon sweep | `dnsrecon -d <domain> -t std` |
> | Zone transfer (dnsrecon) | `dnsrecon -d <domain> -t axfr` |
> | Brute force subdomains | `dnsrecon -d <domain> -D /usr/share/wordlists/dnsmap.txt -t brt` |
> | dnsenum full | `dnsenum <domain>` |

## Decision Tree

```
Have a domain name?
├── Try zone transfer first (low-hanging fruit)
│   └── host -l <domain> <ns1.domain>
│       ├── SUCCESS → massive recon win, map every host
│       └── FAIL    → continue below
├── Enumerate standard records
│   └── dnsrecon -d <domain> -t std
├── Brute force subdomains
│   └── dnsrecon -d <domain> -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t brt
└── Reverse lookup on IP range
    └── dnsrecon -r <IP>/24 -n <nameserver>
```

## Resources
- [HackTricks — DNS](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns)
- [SecLists DNS wordlists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)


The Domain Name System (DNS) is a distributed database responsible for translating user-friendly domain names into IP addresses. It's one of the most critical systems on the internet. This is facilitated by a hierarchical structure that is divided into several zones, starting with the top-level root zone.

> [!note]- Screenshot
> ```
> Each domain can use different types of DNS records. Some of the most common types
> 
> of DNS records include:
> 
> + NS: Nameserver records contain the name of the authoritative servers hosting the
> DNS records for a domain.
> 
> + A: Also known as a host record, the "a record" contains the IPv4 address of a
> hostname (such as www.megacorpone.com).
> 
> + AAAA: Also known as a quad A host record, the "aaaa record" contains the IPv6
> address of a hostname (such as www.megacorpone.com).
> 
> + MX: Mail Exchange records contain the names of the servers responsible for
> handling email for the domain. A domain can contain multiple MX records.
> 
> + PTR: Pointer Records are used in reverse lookup zones and can find the records
> associated with an IP address.
> 
> + CNAME: Canonical Name Records are used to create aliases for other host records.
> 
> + TXT: Text records can contain any arbitrary data and be used for various purposes,
> such as domain ownership verification.
> 
> Due to the wealth of information contained within DNS, it is often a lucrative target for
> 
> active information gathering.
> ```


> [!note]- Screenshot
> ```
> Let's demonstrate this by using the host command to find the IP address of
> ‘www.megacorpone.com.
> kaligkali:~$ host ww.megacorpone.com
> ‘wam.megacorpone.com has address 149.56.244.87
> Listing 13 - Using hast to find the A host racord for www.megacorpone.com
> ```


> [!note]- Screenshot
> ```
> By default, the host command searches for an A record, but we can also query other
> fields, such as MX or TXT records, by specifying the record type in our query using the
> -t option.
> 
> kaligkali:~$ host -t mc megacorpone.com
> 
> rmegacorpone.com mail is handled by 10 #b.mail.gandi.net.
> 
> megacorpone.com mail is handled by 20 spool.mail.gandi.net.
> 
> megacorpone.com mail is handled by 5@ mail.megacorpone.con.
> 
> megacorpone.com mail is handled by 60 mail2.megacorpone.com.
> 
> Listing 14 - Using host to find the MX records for megacorpone.com
> ```

In this case, we first ran the host command to fetch only megacorpone.com MX records, which returned four different mail server records. Each server has a different priority (10, 20, 50, 60) and the server with the lowest priority number will be used first to forward mail addressed to the megacorpone.com domain (fb.mail.gandi.net).

> [!note]- Screenshot
> ```
> We then ran the host command again to retrieve only the megacorpone.com TXT
> records, which returned two entries.
> kaligkali:~$ host -t txt megacorpone.com
> megacorpone.com descriptive text “Try Harder”
> megacorpone.com descriptive text "google-site-
> verification-U7B_b@HNeBtY4qYGQZNSEYX#CI32N"NV3GECOMigSpA"
> sting 15 - Using host to find the TXT records for megacorpone.com
> ```


> [!note]- Screenshot
> ```
> Let's run host against this hostname.
> kaligkali:~$ host uw.megacorpone.com
> wam.megacorpone.com has address 149.56.244.87
> Listing 16 - Using host to search for a valid host
> ```


> [!note]- Screenshot
> ```
> Next, let's determine if megacorpone.com has a server with the hostname "idontexist".
> We'll observe the difference between the query outputs.
> kali@kali:~$ host idontexist.megacorpone.com
> Host idontexist.megacorpone.com not found: 3(NXDOMAIN)
> sting 17- Using host to search for an invalid host
> ```

Having learned the basics of DNS enumeration, we can develop DNS brute-forcing techniques to speed up our research.

Brute forcing is a trial-and-error technique that seeks to find valid information such as directories on a web server, username, and password combinations, or in this case, valid DNS records. By using a wordlist containing common hostnames, we can attempt to guess DNS records and check the response for valid hostnames.

In the examples so far, we used forward lookups, which request the IP address of a hostname to query both a valid and an invalid hostname. If host successfully resolves a name to an IP, this could be an indication of a functional server.

> [!note]- Screenshot
> ```
> We can automate the forward DNS-lookup of common hostnames using the host
> command in a Bash one-liner.
> First, let's build a list of possible hostnames:
> 
> kaligkali:~$ cat list.txt
> 
> ftp
> 
> mail
> 
> proxy
> 
> router
> 
> sting 18 - A small ist of possible hostnames
> ```


> [!note]- Screenshot
> ```
> ‘Next, we can use a Bash one-liner to attempt to resolve each hostname.
> 
> kaligkali:~$ for ip in $(cat list.txt); do host $ip.megacorpone.com; done
> 
> we megacorpone.com has address 149.56.244.87
> 
> Host ftp.megacorpone.com not found: 3(NXDOMAIN)
> 
> mail.megacorpone.com has address 167.114.21.68
> 
> ‘Host owa.megacorpone.com not found: 3(NXDOMAIN)
> 
> ‘Host proxy.megacorpone.com not found: 3(NXDOMAIN)
> 
> router.megacorpone.com has address 167.114.21.70
> 
> Listing 19- Using Bash to brute force forward DNS name lookups
> 
> Using this simplified wordlist, we discovered entries for "www", "mail", and "router". The
> hostnames "ftp", "owa", and "proxy", however, were not found. Much more
> comprehensive wordlists are available as part of the SecLists project. These wordlists
> can be installed to the /usr/share/seclists directory using the sudo apt install seclists
> command.
> ```

> for ip in $(cat list.txt); do host $ip.megacorpone.com; done
[https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)

## These wordlists can be installed to the /usr/share/seclists directory using the sudo apt install seclists command.

Except for the www record, our DNS-forward brute force enumeration revealed a set of scattered IP addresses in the same approximate range (167.114.21.X). If the DNS administrator of megacorpone.com configured PTR records for the domain, we could scan the approximate range with reverse lookups to request the hostname for each IP.

> [!note]- Screenshot
> ```
> Let's use a loop to scan IP addresses 167.114.21.64 through 167.114.21.79. We will filter
> 
> out invalid results (using grep -Ev), showing only entries that do not contain "not found"
> 
> or "timed out".
> kali@kali:~$ for ip in $(seq 64 79); do host 167.114.21.Sip; done | grep -Ev “not
> found| timed out™
> 64.21.114.167.in-addr.arpa domain name pointer admin.megacorpone.com.
> 65.21.114.167.in-addr.arpa domain name pointer beta.megacorpone.com.
> (66.21.114.167-in-addr-arpa domain name pointer fs1.megacorpone.com.
> (67.21.114.167-in-addr-arpa domain name pointer intranet.megacorpone.com.
> 68.21.114.167.in-addr.arpa domain name pointer mail.megacorpone.com.
> 69.21.114.167.in-addr.arpa domain name pointer mail2.megacorpone.com.
> 7@.21.114.167.in-addr.arpa domain name pointer router.megacorpone.com.
> ‘71.21.114.167.in-addr.arpa domain name pointer siem.megacorpone.com.
> ‘72.21.114.167.in-addr.arpa domain name pointer snmp.megacorpone.com.
> ‘73.21.114.167.in-addr.arpa domain name pointer syslog.megacorpone.com.
> ‘74.21.114.167.in-addr.arpa domain name pointer support.megacorpone.com.
> ‘75.21.114.167.in-addr.arpa domain name pointer test.megacorpone.com.
> 76.21.114.167-in-addr.arpa domain name pointer vpn.megacorpone.com.
> ‘7.21.114.167.in-addr.arpa domain name pointer vpn2.megacorpone.com.
> ‘78.21.114.167.in-addr.arpa domain name pointer vpndev.megacorpone.com.
> ‘79.21.114.167.in-addr.arpa domain name pointer vpnprod.megacorpone.com.
> 
> Listing 20 - Using Bash to brute force reverse DNS names
> ```

> for ip in $(seq 64 79); do host 167.114.21.$ip; done | grep -Ev "not found|timed out"

We have successfully managed to resolve several IP addresses to valid hosts using reverse DNS lookups. If we were performing an assessment, we could further extrapolate these results, and might scan for "mail2", "router", etc., and reverse-lookup positive results. These types of scans are often cyclical; we expand our search based on any information we receive at every round.

Now that we have developed our foundational DNS enumeration skills, let's explore how we can automate the process using a few applications.

There are several tools in Kali Linux that can automate DNS enumeration. Two notable examples are DNSRecon and DNSenum; let's explore their capabilities.

DNSRecon is an advanced DNS enumeration script written in Python. Let's run dnsrecon against megacorpone.com, using the -d option to specify a domain name and -t to specify the type of enumeration to perform (in this case, a standard scan).

> [!note]- Screenshot
> ```
> DNSReconis an advanced DNS enumeration script written in Python. Let's run dnsrecon
> 
> against megacorpone.com, using the -d option to specify a domain name and -t to
> 
> ‘specify the type of enumeration to perform (in this case, a standard scan).
> kaligkali:~$ dnsrecon -d megacorpone.com -t std
> 2025-11-05109:29:45.290363-0800 INFO Starting enumeration for domain: megacorpone.com
> 2025-11-05709:29:45.290668-0800 INFO std: Performing General Enumeration against:
> megacorpone.com. .-
> 2025-11-5709:29:45.519861-0800 ERROR No answer for DNSSEC query for megacorpone.com
> 2025-11-5709:29:45.882832-0800 INFO SOA ns1.megacorpone.com 51.79.37.18
> 2025-11-5709:29:46.741122-0800 INFO NS nsi.megacorpone.com 51.79.37.18
> 2025-11-05709:29:46.955101-0800 INFO Bind Version for 51.79.37.18 "9.18.24-1-
> Debian”
> 2025-11-5709:29:46.955579-0800 INFO NS ns3.megacorpone.com 66.70.207.150
> 2025-11-5709:29:47.159697-0800 INFO Bind Version for 66.70.207.180 "9.18.24-1-
> Debian”
> 2025-11-5709:29:47.160241-0800 INFO NS ns2.megacorpone.com 51.222.39.63
> 2025-11-@5709:29:47.360733-0800 INFO Bind Version for 51.222.39.63 "9.18.24-1-
> Debian”
> 2025-11-05709:29:48.214191-0800 INFO MX mail-megacorpone.com 167.114.21.68
> 2025-11-05109:29:48.214759-0800 INFO MK spool.mail.gandi-net 217.70.178.1
> 2025-11-5709:29:48.214859-0800 INFO MX mail2.megacorpone.com 167.114.21.69
> 2025-11-05709:29:48.214928-0800 INFO MX fb.mail.gandi.net 217.70.178.217
> 2025-11-05709:29:48.214989-0800 INFO MX fb.mail.gandi-net 217.70.178.215
> 2025-11-5709:29:48.215132-0800 INFO MX fb.mail.gandi.net 217.70.178.216
> 2025-11-5709:29:48.215216-0800 INFO NX spool.mail.gandi.net 2001:4b98:e00: :1
> 2025-11-05709:29:48.215285-0800 INFO NX fb.mail.gandi-net 2001:4b98:dc4:8: :216
> 2025-11-5709:29:48.215358-0800 INFO MX fb.mail.gandi-net 2001:4b98:dc4:8::215
> 2025-11-5709:29:48.215454-0800 INFO MX fb.mail.gandi-net 2001:4098:dc4:8::217
> 2025-11-05109:29:48.435467-0800 INFO A megacorpone.com 149.56.244.87
> 2025-11-5109:29:48.790016-0800 INFO TXT megacorpone..com google-site-
> verification-U7B_b@HNeBtV4qYGQZNSEYX#CI32N"NV3GECOuia5pA
> 2025-11-05709:29:48.790608-0800 INFO TXT megacorpone.com Try Harder
> 2025-11-05709:29:49.037780-0800 INFO Enumerating SRV Records
> 2025-11-5709:29:50.050759-0800 ERROR No SRV Records Found for megacorpone.com
> 2025-11-05109:29:50.051673-0800 INFO Completed enumeration for domain: megacorpone.com
> 
> sting 21 Using dsrecon to perform a standard scan
> ```

> dnsrecon -d megacorpone.com -t std

> [!note]- Screenshot
> ```
> Let's try to brute force additional hostnames using the 1ist.txt file we created
> previously for forward lookups.
> 
> kaligkali:~$ cat list.txt
> 
> ftp
> 
> mail
> 
> proxy
> 
> router
> 
> isting 22 = List to be used for subdomain brute forcing using dnsrecon
> ```


> [!note]- Screenshot
> ```
> To perform our brute force attempt, we will use the -d option to specify a domain name,
> -p to specify a file name containing potential subdomain strings, and -t to specify the
> type of enumeration to perform, in this case brt for brute force.
> kaligkali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
> 2025-11-05709:31:17.105002-0800 INFO Using the dictionary file: /nome/kali/list.txt
> (provided by user)
> 2025-11-05T09:31:17.105225-0800 INFO Starting enumeration for domain: megacorpone.com
> 2025-11-05709:31:17.105415-0800 INFO brt: Performing host and subdomain brute force
> against megacorpone. com...
> 2025-11-@5709:31:17.472625-0800 INFO A router.megacorpone.com 167.114.21.78
> 2025-11-05709:31:17.473573-0800 INFO A www.megacorpone.com 149.56.244.87
> 2025-11-05709:31:17.518546-0800 INFO A mail.megacorpone.com 167.114.21.68
> 2025-11-@5709:31:17.519863-0800 INFO 3 Records Found
> 2025-11-05109:31:17.520273-080@ INFO Completed enumeration for domain: megacorpone.com
> Listing 23 - Brute forcing hostnames using dnsrecon
> ```

> dnsrecon -d megacorpone.com -D ~/list.txt -t brt


DNSEnum is another popular DNS enumeration tool that can be used to further automate DNS enumeration of the megacorpone.com domain. We can pass the tool a few options, but for the sake of this example, we'll only pass the target domain parameter:

> [!note]- Screenshot
> ```
> kaligkali:~$ dnsenum megacorpone.com
> ‘dnsenum VERSION:1.3.1
> s---- megacorpone.com -----
> Host"s addresses:
> megacorpone.com. 3002«~«*N' A 149.56.244.87
> ‘Trying Zone Transfers and getting Bind Versions:
> megacorpone.com. 30002=SCIN nS. ‘ns1L.megacorpone.com.
> megacorpone.com. 30002=SOIN AS. ‘ns2.megacorpone.com.
> megacorpone.com. 3002=SOIN NS. 'ns3.megacorpone.com.
> ‘adnin.megacorpone.com. 3002«*IN' A 167.114.21.64
> ‘ai-megacorpone.com. 3002«*IN' A 149.56.244.87
> beta.megacorpone.com. 3002¢«*IN' A 167.114.21.65
> s1.megacorpone.com. 3002«*N' A 167.114.21.66
> intranet .megacorpone.com. 3002«*IN' A 167.114.21.67
> mail.megacorpone.com. 3002¢«*IN' A 167.114.21.68
> mail2.megacorpone.com. 3002«*N' A 167.114.21.69
> ‘ns1_megacorpone.com. 3002«*IN' A 51.79.3718
> 'ns2.megacorpone.com. 3002«*IN' A 51.222.39.63
> 'ns3.megacorpone..com. 3002¢«*IN' A 66.70.207.180
> router.megacorpone.com. 3002«*N' A 167.114.21.70
> ‘siem.megacorpone.com. 3002«*IN' A 167.114.21.71
> ‘snmp-megacorpone.com. 3002¢«*IN' A 167.114.21.72
> ‘support .megacorpone..com. 3002«*N' A 167.114.21.74
> syslog-megacorpone.com. 3002«*IN' A 167.114.21.73
> ‘test.megacorpone.com. 3002«*IN' A 167.114.21.75
> ‘vpn.megacorpone.com. 3002¢«*IN' A 167.114.21.76
> ‘vpn2.megacorpone.com. 3002«*N' A 167.114.21.7
> \vpndev.megacorpone.com. 3002«*IN' A 167.114.21.78
> ‘vpnprod.megacorpone.com. 3002¢«*IN' A 167.114.21.79
> ‘wum.megacorpone.com. 3002«*N' A 149.56.244.87
> ‘wi2.megacorpone.com. 3002«*IN' A 149.56.244.87
> megacorpone.com class C netranges:
> 51.79.37.0/24
> 51.222.39.0/24
> 66.70.207.0/24
> 149.56.244.0/24
> 167.114.21.0/24
> 
> Listing 24 - Using dnsenum to automate DNS enumeration
> ```

> dnsenum megacorpone.com

We have now discovered several previously-unknown hosts as a result of our extensive DNS enumeration. This set of hostnames can now be used for service enumeration, port scanning, or validating exposure. As mentioned at the beginning of this Module, information gathering has a cyclic pattern, so we'll need to perform all the other passive and active enumeration tasks on this new subset of hosts to disclose any new potential details.


> xfreerdp /u:student /p:lab /v:192.168.50.152


> nslookup mail.megacorptwo.com



In this example, we are specifically querying the 192.168.50.151 DNS server for any TXT record related to the info.megacorptwo.com host.

The nslookup utility is as versatile as the Linux host command and the queries can also be further automated through PowerShell or Batch scripting.

> [!note]- Screenshot
> ```
> Next, let's connect to our Windows 11 client using the xfreerdp command. The syntax is.
> /u: and the username, /p: and the password, and /v: and the ip address.
> kaligcali:-$ xfreerdp /u:student /p:lab /v:192.168.50.152
> sting 25 - Connecting to the Windows 11 client
> ```


> [!note]- Screenshot
> ```
> Let's use nslookup to resolve a domain name to an IP address.
> C:\Users\student> nslookup mail.megacorptwo.com
> 
> DNS request timed out.
> 
> ‘timeout was 2 seconds.
> 
> Server: Unknown
> 
> ‘Address: 192.168.50.151
> 
> Name: mail-megacorptwo.com
> 
> Address: 192.168.50.154
> 
> Listing 26 - Using nslookup to perform a simple host enumeration
> ```


> [!note]- Screenshot
> ```
> This confirms the DNS resolution succeeded and shows the domain's IP.
> The output above shows we queried the default DNS server (192.168.50.151) to resolve
> the IP address of mail.megacorptwo.com, which the DNS server then answered with
> "192.168.50.154".
> Similarly to the Linux host command, nslookup can perform more granular queries. For
> instance, we can query a given DNS about a TXT record that belongs to a specific host.
> 
> C2\Users\student> nslookup -type=TXT info.megacorptwo.com 192.168.50.151
> 
> Server: Unknown
> 
> ‘Address: 192.168.50.151
> 
> info.megacorptwo.com text =
> 
> “greetings from the TXT record body”
> Listing 27 - Using nslookup to perform 2 more specific query
> ```


```sh
> nslookup -type=TXT info.megacorptwo.com 192.168.50.151
```

---
%% graph-links %%
## Related
- [[WHOIS Enumeration]]
- [[SMB Enumeration]]
- [[Nmap Scripting Engine (NSE)]]

> [!info] Navigation
> Section: [[Active Information Gathering/_index|Active Information Gathering]] · Home: [[🏠 Home]]

