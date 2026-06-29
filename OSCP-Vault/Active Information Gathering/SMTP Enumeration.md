---
tags:
  - enumeration
  - phase/enumeration
  - smtp
---

# SMTP Enumeration

> [!tip] Quick Reference — SMTP
> | Goal | Command |
> |------|---------|
> | Banner grab | `nc -nv <IP> 25` |
> | Verify user exists | `VRFY <username>` (in nc session) |
> | Expand mailing list | `EXPN <list>` (in nc session) |
> | Nmap scripts | `nmap -p 25 --script smtp-enum-users,smtp-commands <IP>` |
> | smtp-user-enum tool | `smtp-user-enum -M VRFY -U /usr/share/wordlists/users.txt -t <IP>` |

## Decision Tree

```
Port 25/465/587 open?
├── Banner grab → nc -nv <IP> 25
│   └── Note software version → searchsploit it
├── User enumeration
│   ├── VRFY supported? → smtp-user-enum -M VRFY -U users.txt -t <IP>
│   ├── EXPN supported? → smtp-user-enum -M EXPN -U users.txt -t <IP>
│   └── RCPT TO supported? → smtp-user-enum -M RCPT -U users.txt -t <IP>
└── Found users? → feed into password attacks
```

## Resources
- [HackTricks — SMTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp)


The Simple Mail Transport Protocol (SMTP) supports several interesting commands, such as VRFY and EXPN. A VRFY request asks the server to verify an email address, while EXPN asks the server for the membership of a mailing list. These can often be abused to verify existing users on a mail server, which is useful information during a penetration test.

> [!note]- Screenshot
> ```
> kaligkali:~$ nc -nv 192.168.50.8 25
> (UNKNOWN) [192.168.50.8] 25 (smtp) open
> 220 mail ESMTP Postfix (Ubuntu)
> VRFY root
> 252 2.0.0 root
> VRFY idontexist
> 550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient
> table
> “
> sting 54 - Using nc to validate SMTP users
> ```


```sh
nc -nv 192.168.50.8 25
```

We can observe how the success and error messages differ. The 252 SMTP response code does not verify the root user exists but will accept and attempt delivery of any messages. Response code 550 indicates the mailbox is unavailable. This procedure can be used to help guess valid usernames in an automated fashion. Next, let's consider the following Python script, which opens a TCP socket, connects to the SMTP server, and issues a VRFY command for a given username:

> [!note]- Screenshot
> ```
> #1/usr/bin/python
> import socket
> import sys
> if len(sys.argv) != 3:
> print("Usage: vrfy.py <username> <target_ip>")
> sys.exit(@)
> # Create a Socket
> 5s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
> # Connect to the Server
> ip = sys-argv[2]
> connect = s.connect((ip,25))
> # Receive the banner
> banner = s.recv(1024)
> print (banner)
> # VRFY a user
> user = (sys-argv[1]) -encode()
> s.send(bVRFY * + user + b*\r\n")
> result = s.recv(1024)
> print (result)
> # Close the socket
> s.close()
> Listing 55 - Using Python to script the SMTP user enumeration
> ```


```sh
#!/usr/bin/python

import socket
import sys

if len(sys.argv) != 3:
        print("Usage: vrfy.py <username> <target_ip>")
        sys.exit(0)

# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the Server
ip = sys.argv[2]
connect = s.connect((ip,25))

# Receive the banner
banner = s.recv(1024)

print(banner)

# VRFY a user
user = (sys.argv[1]).encode()
s.send(b'VRFY ' + user + b'\r\n')
result = s.recv(1024)

print(result)

# Close the socket
s.close()
```


> [!note]- Screenshot
> ```
> We can run the script by providing the username to be tested as a first argument and
> the target IP as a second argument.
> 
> | kaligeali:~/Desktop$ python? smtp.py root 192.168.50.8 ai
> | b'220 mail ESMTP Postfix (Ubuntu)\r\n" i
> { b'252 2.0.0 root\r\n* H
> | \aligkali:~/Desktop$ python3 smtp.py johndoe 192.168.50.8 H
> | b'22@ mail ESMTP Postfix (Ubuntu)\r\n* H
> | b'550 5.1.1 <johndoe>: Recipient address rejected: User unknown in local recipient i
> | table\r\n" i
> Se PT Le ee
> ```


```sh
python3 smtp.py root 192.168.50.8
python3 smtp.py root 192.168.50.8
```


> [!note]- Screenshot
> ```
> Similarly, we can obtain SMTP information about our target from the Windows 11 client,
> as we did previously:
> 
> PS C:\Users\student> Test-NetConnection -Port 25 192.168.50.8
> 
> ComputerName = 192.168.50.8
> 
> RenoteAddress  : 192.168.50.8
> 
> RenotePort 125
> 
> InterfaceAlias : Ethernet
> 
> SourceAddress : 192.168.50.152
> 
> TepTestSucceeded : True
> 
> Listing 57 - Port scanning SMTP via PowerShell
> ```


```sh
Test-NetConnection -Port 25 192.168.50.8
```


> [!note]- Screenshot
> ```
> Unfortunately, with Test-NetConnection we are prevented from fully interacting with the
> ‘SMTP service. Nevertheless, if not already enabled, we can install the Microsoft version
> of the Telnet client, as shown:
> PS C:\Windows\systen32> dism /online /Enable-Feature /FeatureNane:TelnetClient
> Listing 58 - Installing the Telnet client
> ```


```sh
dism /online /Enable-Feature /FeatureName:TelnetClient
```


> [!note]- Screenshot
> ```
> C:\Windows\systen32> telnet 192.168.50.8 25
> 
> 220 mail ESMTP Postfix (Ubuntu)
> 
> VRFY goofy
> 
> 550 5.1.1 <goofy>: Recipient address rejected: User unknown in local recipient table
> 
> VRFY root
> 
> 252 2.0.0 root
> 
> Listing 59 - interacting with the SMTP service via Telnet on Windows
> 
> The above output depicts yet another example of enumeration that we can perform from
> a compromised Windows host when Kali is not available.
> ```


```sh
telnet 192.168.50.8 25
```
