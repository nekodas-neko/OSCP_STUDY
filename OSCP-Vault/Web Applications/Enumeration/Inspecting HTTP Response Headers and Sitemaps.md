---
tags:
  - phase/enumeration
  - enumeration
  - http
  - nmap
  - nse
  - web
---

# Inspecting HTTP Response Headers and Sitemaps

We can also search server responses for additional information. There are two types of tools we can use to accomplish this task. The first type is a proxy, like Burp Suite, which intercepts requests and responses between a client and a web server, and the other is the browser's own Network tool.

We can launch it from the Firefox Web Developer menu to review HTTP requests and responses. This tool shows network activity that occurs after it launches, so we must refresh the page to display traffic.







Historically, headers that started with "X-" were called non-standard HTTP headers. However, RFC6648 now deprecates the use of "X-" in favor of a clearer naming convention.

The names or values in the response header often reveal additional information about the technology stack used by the application. Some examples of non-standard headers include X-Powered-By, x-amz-cf-id, and X-Aspnet-Version. Further research into these names could reveal additional information, such as that the "x-amz-cf-id" header indicates the application uses Amazon CloudFront.

Sitemaps are another important element we should take into consideration when enumerating web applications.

Web applications can include sitemap files to help search engine bots crawl and index their sites. These files also include directives of which URLs not to crawl - typically sensitive pages or administrative consoles, which are exactly the sort of pages we are interested in.

Inclusive directives are performed with the sitemaps protocol, while robots.txt excludes URLs from being crawled.

For example, we can retrieve the robots.txt file from
[www.google.com](http://www.google.com)
with curl:



Allow and Disallow directives inform “polite” web crawlers which pages or directories may be accessed or should be avoided. In most cases, the listed pages and directories may not be interesting, and some may even be invalid. Nevertheless, sitemap files should not be overlooked because they may contain clues about the website layout or other interesting information, such as yet-unexplored portions of the target.

> [!info] Firefox Network tool
> Open the Network tab from the Firefox Web Developer menu, then refresh the page. It lists every HTTP request the browser makes so you can inspect them individually.


> [!info] Viewing response headers
> Click a request in the Network tab and open its Headers to see the response headers. The `Server` header usually reveals the web server software, and in many default configurations its version number too.


> [!info] Tip
> Not all headers come from the web server itself. Proxies, for example, insert `X-Forwarded-For` to pass the original client IP address on to the server.


```sh
curl https://www.google.com/robots.txt
```

Sample output:

```
User-agent: *
Disallow: /search
Allow: /search/about
Disallow: /groups
Disallow: /index.html?
...
```

## Visual Flow

```mermaid
flowchart TD
    A[Target web app] --> B["curl -I http://IP"]
    B --> C{Headers reveal tech?}
    C -->|"Server, X-Powered-By, X-Aspnet-Version"| D[Research the software/version]
    A --> E["curl http://IP/robots.txt"]
    E --> F{Disallow entries?}
    F -->|/admin, /backup| G[Browse those paths manually]
    A --> H[Firefox Network tab: refresh, click request]
    H --> I[Read Response Headers in browser]
```

> [!success] What success looks like
> `curl -I` returns headers such as `Server: Apache/2.4.41 (Ubuntu)` or `X-Powered-By: PHP/7.4`. `robots.txt` lists `Disallow:` paths like `/admin` — exactly the "hidden" pages worth visiting.

> [!danger] Common errors
> - `curl: (60) SSL certificate problem` on HTTPS labs → add `-k` to skip cert verification: `curl -Ik https://IP`.
> - `-I` shows nothing useful → some servers reject HEAD; try `curl -i http://IP` (lowercase, full GET with headers).
> - robots.txt returns 404 → not every site has one; that is fine, move on to gobuster.
> - Network tab empty → it only records after it opens; refresh the page (F5).
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> `-I` (capital i) asks only for the **headers**, not the page body — a fast way to see the `Server` line and other tech hints. `robots.txt` is a public file websites use to tell crawlers what to skip, which conveniently points you straight at sensitive areas.

---
%% graph-links %%
## Related
- [[Debugging Page Content]]
- [[Directory Brute Force with Gobuster]]
- [[Security Headers and SSLTLS]]

> [!info] Navigation
> Section: [[Web Applications/Enumeration/_index|Enumeration]] · Home: [[🏠 Home]]

