---
tags:
  - file-upload
  - phase/enumeration
  - rce
  - vulnerability-scanning
  - web
---

# File Upload Vulnerabilities
%% moc %%
> [!abstract] Map of Content
> Turn an upload into a shell — executable and non-executable paths.
> 
> ⬆ Up: [[Web Applications/Common Web Application Attacks/_index|Common Web Application Attacks]]

## Notes
- [[Using executable files]]
- [[Using non-executable files]]

## Related Sections
- [[Web Applications/Common Web Application Attacks/File Inclusion Vulnerabilities/_index|File Inclusion Vulnerabilities]]

> [!info] The three categories of file upload vulnerability
> 1. **Directly executable** — upload a script (e.g. PHP) the web server will run, then browse to it. Covered in [[Using executable files]].
> 2. **Combined with another vuln** — e.g. Directory Traversal to overwrite `authorized_keys`, or XXE/XSS smuggled in via an uploaded SVG. Covered in [[Using non-executable files]].
> 3. **User-interaction-dependent** — e.g. a malicious macro document uploaded as a "CV," requiring someone else to open it. Out of scope here — see [[Client-Side Attacks/_index|Client-Side Attacks]] for that vector instead.
