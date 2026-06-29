# Manual and automated code execution
%% moc %%
> [!abstract] Map of Content
> From SQLi to RCE — by hand and with sqlmap, plus labs.
> 
> ⬆ Up: [[SQL Injection Attacks/_index|SQL Injection Attacks]]

## Notes
- [[Automating the attack]]
- [[Labs]]
- [[Manual code execution]]

## Related Sections
- [[SQL Injection Attacks/Manual SQL exploitation/_index|Manual SQL exploitation]]

---

10.3. Manual and automated code execution
This Learning Unit covers the following Learning Objectives:

Exploit MSSQL Databases with xp_cmdshell
Automate SQL Injection with SQLmap
Depending on the operating system, service privileges, and filesystem permissions, SQL injection vulnerabilities can be used to read and write files on the underlying operating system.

Writing a carefully crafted file containing PHP code into the root directory of the web server could then be leveraged for full code execution.
