---
tags:
  - fingerprinting
  - phase/enumeration
  - web
---

# Technology Stack Identification with Wappalyzer

Along with the active information gathering we performed via Nmap, we can also passively fetch a wealth of information about the application technology stack via Wappalyzer.
[https://www.wappalyzer.com/](https://www.wappalyzer.com/)
This quick external analysis reveals information about the OS, UI framework, web server, and more. The findings also provide information about JavaScript libraries used by the web application - this can be valuable data, as some versions of JavaScript libraries are known to be affected by several vulnerabilities.

> [!note]- Screenshot
> ```
> Once we have registered a free account, we can perform a Technology Lookup on the
> megacorpone.com domain.
> € Cc a Oo Bt wappalyzer.com) gacore
> SE Kali Linux RX Kali Tools XX KaliForums [MJ KaliDocs @XNetHunter i Offensive Security J MSFU
> @ Technology stack
> Operating systems
> © Debian
> UI frameworks
> @ Bootstrap
> Web servers
> @ Spache
> CDN
> <> Google Hosted Libraries
> Font scripts
> E21 Font Awesome
> JavaScript libraries
> © jQuery @™ prettyPhoto
> ```

---
%% graph-links %%
## Related
- [[FingerPrinting with Nmap]]
- [[Debugging Page Content]]
- [[Inspecting HTTP Response Headers and Sitemaps]]

> [!info] Navigation
> Section: [[Web Applications/Application Assesment Tools/_index|Application Assesment Tools]] · Home: [[🏠 Home]]

