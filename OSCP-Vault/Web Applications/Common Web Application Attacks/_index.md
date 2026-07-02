# Common Web Application Attacks
%% moc %%
> [!abstract] Map of Content
> Command injection, directory traversal, file inclusion, and file upload.
> 
> ⬆ Up: [[Web Applications/_index|Web Applications]]

## Subsections
- 📁 [[Web Applications/Common Web Application Attacks/Directory Traversal/_index|Directory Traversal]]
- 📁 [[Web Applications/Common Web Application Attacks/File Inclusion Vulnerabilities/_index|File Inclusion Vulnerabilities]]
- 📁 [[Web Applications/Common Web Application Attacks/File Upload Vulnerabilities/_index|File Upload Vulnerabilities]]

## Notes
- [[Command Injection]]

## Wrapping up

Across this section: **directory traversal** displays file contents outside the web root, **file inclusion** goes further and executes those files within the app's running code, **file upload** abuses upload mechanisms with executable and non-executable payloads, and **command injection** goes straight to the underlying OS.

> [!success] Why this matters
> None of these vulnerabilities are tied to a specific language or framework, but their *exploitation* often is — always take a moment to fingerprint the web stack (see [[Web Applications/Application Assesment Tools/_index|Application Assessment Tools]]) before attempting any of them. Found on an internet-facing app, they're a foothold; found on an internal service, they're a lateral-movement vector.

## Related Sections
- [[Web Applications/Cross-Site Scripting/_index|Cross-Site Scripting]]
- [[SQL Injection Attacks/_index|SQL Injection Attacks]]
