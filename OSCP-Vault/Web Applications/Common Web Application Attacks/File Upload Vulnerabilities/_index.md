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

---

# File Upload Vulnerabilities

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


Many web applications provide functionality to upload files. In this Learning Unit, we will learn how to identify, exploit, and leverage File Upload vulnerabilities to access the underlying system or execute code. In general, we can group File Upload vulnerabilities into three categories:

The first category consists of vulnerabilities enabling us to upload files that are executable by the web application. For example, if we can upload a PHP script to a web server where PHP is enabled, we can execute the script by accessing it via the browser or curl. As we observed in the File Inclusion Learning Unit, apart from PHP, we can also leverage this kind of vulnerability in other frameworks or server-side scripting languages.

The second category consists of vulnerabilities that require us to combine the file upload mechanism with another vulnerability, such as Directory Traversal. For example, if the web application is vulnerable to Directory Traversal, we can use a relative path in the file upload request and try to overwrite files like authorized_keys. Furthermore, we can also combine file upload mechanisms with XML External Entity (XXE) or Cross Site Scripting (XSS) attacks. For example, when we are allowed to upload an avatar to a profile with an SVG file type, we may embed an XXE attack to display file contents or even execute code.

The third category relies on user interaction. For example, when we discover an upload form for job applications, we can try to upload a CV in .docx format with malicious macros integrated. Since this category requires a person to access our uploaded file, we will focus on the other two kinds of file upload vulnerabilities in this Learning Unit.
