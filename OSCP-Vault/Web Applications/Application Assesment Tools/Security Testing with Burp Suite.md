---
tags:
  - burp-suite
  - phase/enumeration
  - web
---

# Security Testing with Burp Suite

> [!tip] Quick Reference — Burp Suite
> | Feature | Shortcut / How |
> |---------|----------------|
> | Send to Repeater | Ctrl+R |
> | Send to Intruder | Ctrl+I |
> | Forward request | Ctrl+F (in Intercept) |
> | Toggle intercept | Ctrl+T |
> | Search in response | Ctrl+F (in Repeater) |
> | Decode selection | Right-click → Send to Decoder |

## Workflow

```
Set up proxy → Firefox uses 127.0.0.1:8080
├── Intercept ON → modify requests on the fly
├── HTTP History → review all past requests
├── Repeater → replay + tweak individual requests
│   └── Best for: manual SQLi, XSS, auth testing
└── Intruder → automated fuzzing
    ├── Positions → mark injection points with §§
    ├── Payload sets → wordlist / numbers / brute
    └── Attack types:
        ├── Sniper    → one position, one payload list
        ├── Battering ram → same payload in all positions
        ├── Pitchfork → parallel lists per position
        └── Cluster bomb → all combinations (slow)
```

## Resources
- [PortSwigger Web Academy](https://portswigger.net/web-security) — free labs
- [HackTricks — Burp](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/burp-suite)


Burp Suite is a GUI-based integrated platform for web application security testing. It provides several different tools via the same user interface.

While the free Community Edition mainly contains tools used for manual testing, the commercial versions include additional features, including a formidable web application vulnerability scanner. Burp Suite has an extensive feature list and is worth investigating, but we will only explore a few basic functions in this section.

An important feature of Burp Suite is the ability to intercept HTTPS traffic. This requires using the Burp certificate, which is beyond the scope of this discussion.












Let's start with the Proxy tool. In general terms, a web proxy is any dedicated hardware or software meant to intercept requests and/or responses between the web client and the web server. This allows administrators and testers alike to modify any requests that are intercepted by the proxy, both manually and automatically.

With the Burp Proxy tool, we can intercept any request sent from the browser before it is passed on to the server. We can change almost anything about the request at this point, such as parameter names or form values. We can even add new headers. This lets us test how an application handles unexpected arbitrary input. For example, an input field might have a size limit of 20 characters, but we could use Burp Suite to modify a request to submit 30 characters.

To set up a proxy, we will first click the Proxy tab to reveal several sub-tabs. We'll also disable the Intercept tool, found under the Intercept tab.







By default, Burp Suite enables a proxy listener on localhost:8080. This is the host and port that our browser must connect to in order to proxy traffic through Burp Suite.



Let’s walk through configuring the Firefox browser on our local Kali machine to use Burp Suite as a proxy.

In Firefox, we can do this by navigating to about:preferences#general, scrolling down to Network Settings, then clicking Settings.

Let's choose the Manual option, setting the appropriate IP address and listening port. In our case, the proxy (Burp) and the browser reside on the same host, so we'll use the loopback IP address 127.0.0.1 and specify port 8080.



Finally, we also want to enable this proxy server for all protocol options to ensure that we can intercept every request while testing the target application.







By clicking on one of the requests, the entire dump of client requests and server responses is shown in the lower half of the Burp UI.





Besides the Proxy feature, the Repeater is another fundamental Burp tool. With the Repeater, we can craft new requests or easily modify the ones in History, resend them, and review the responses. To observe this in action, we can right-click a request from Proxy > HTTP History and select Send to Repeater.







The last feature we will cover is Intruder. First, we'll need to configure our local Kali's /etc/hosts file to statically assign the IP to the offsecwp website we are going to test.



This allows us to access the VM by hostname while bypassing DNS.




The Intruder Burp feature, as its name suggests, is designed to automate a variety of attack angles, from the simplest to more complex web application attacks. To learn more about this feature, let's simulate a password brute forcing attack.

Since we are dealing with a new target, we can start a new Burp session and configure the Proxy as we did before. Next, we'll navigate to
[http://offsecwp/wp-login.php](http://offsecwp/wp-login.php)
from Firefox. Then, we will type "admin" and "test" as respective username and password values, and click Log in.







We have now instructed the Intruder to modify only the password value on each new request. Before starting our attack, let's provide Intruder with a wordlist. Knowing that the correct password is "password", we can grab the first 10 values from the rockyou wordlist on Kali.




Moving to the Payloads sub-tab, we can paste the above wordlist into the Payload Options: [Simple list] area.



With everything ready to start the Intruder attack, let's click on the top right Start Attack button.

We can move past the Burp warning about restricted Intruder features, as this won't impact our attack. After we let the attack complete, we can observe that apart from the initial probing request, it performed 10 requests, one for each entry in the provided wordlist.



We'll notice that the WordPress application replied with a different Status code on the 4th request, hinting that this might be the correct password value. Our hypothesis is confirmed once we try to log in to the WordPress administrative console with the discovered password.

> [!note]- Screenshot
> ```
> We can find Burp Suite Community Edition in Kali under Applications > 03 Web
> Application Analysis > burpsuite.
> Aplications ~ Places ~ Mon’6:19 © oe @ 4on-
> 
> Favorites
> 
> 01 tnformatonGatherng
> 
> se vunsabinarane : EE cori
> 
> (04 Database Assessment >
> 
> Sore Sea
> 
> 06- Wireless Attacks , pares
> 
> 07 Reverse Engineering Bi aan
> 
> 08- Exploitation Toots
> 
> 09- Siting &Speting , ETE salmep
> 
> 10- Post xpotaton , ——webscarb
> 
> 11 Forensies ,
> 
> 1a epring Te B= ~
> 
> 13 - Social Engineering Tools
> 
> 14- System Services ,
> 
> Useal applicators ,
> 
> Activities Overview
> 
> Figure 2: Starting Burp Suite
> We can also launch it from the command line with burpsuite:
> kali@kali:~$
> Listing 4 - Starting Burp Suite from a terminal shell
> ```


```sh
burpsuite
```


> [!note]- Screenshot
> ```
> After our initial launch, we'll first notice a warning that Burp Suite has not been tested on
> our (JRE). Since the Kali team always tests Burp Suite on the
> Java version shipped with the OS, we can safely ignore this warning.
> i | Burp Suite Community Edition x
> 
> Your JRE appears to be version 1.0.11 from Debian
> 
> Burp has not been fully tested on this platform and you may experience
> 
> problems.
> 
> Don't show again for this JRE
> OK
> Figure 1: Burp Suite JRE warning
> ```


> [!note]- Screenshot
> ```
> Once it launches, we'll choose Temporary project and click Next.
> 5 Burp Suite Community Edition 2021103
> ® Welcome to Burp Suite Communit Edition, Use the options below tocreate or pena project Burp Suite
> Community Edition
> O Temporary project
> Name Fle
> Figure 3: Burp Startup
> ```


> [!note]- Screenshot
> ```
> We'll leave Use Burp defaults selected and click Start Burp.
> 5 Burp Suite Community Edition 202110.3
> © setectnecoiguation tat you woud eto ladforthspojec Burp Suite
> Community Edition
> © Use Burpdefauits
> f
> Load rom configuration file re
> File: Choose ie.
> Defauttothe aboveinfuture
> Disable extensions
> Canc Bock
> Figure 4: Burp Configuration
> ```


> [!note]- Screenshot
> ```
> After a few moments, the UI will load.
> Tate @902 ‘eich ere x
> Se | eee
> 20 CD ED CD =: :
> 
> Figure 5: Burp Suite User Interface a
> The initial four panes of the interface primarily serve as a summary for the Pro version
> scanner, so we can ignore them. Instead, we are going to focus on the features present
> on the tabs in the upper bar.
> ```


> [!note]- Screenshot
> ```
> Q Tip
> 
> When /ntercept is enabled, we have to manually click on Forward to send each
> request to its destination. Alternatively, we can click Drop to not send the request.
> There are times when we will want to intercept traffic and modify it, but when we
> are just browsing a site, having to click Forward on each request is very tedious.
> ```


> [!note]- Screenshot
> ```
> Spon brewer
> 
> Use Burp's embedded browser \
> a e
> 
> Tere'sno ned to config our roy settings oe =
> : vay
> ```


> [!note]- Screenshot
> ```
> Next, we can review the proxy listener settings. The Options sub-tab shows what ports
> are listening for proxy requests.
> 
> Figure 7: Proxy Listeners
> By default, Burp Suite enables a proxy listener on localhost:8080. This is the host and
> port that our browser must connect to in order to proxy traffic through Burp Suite.
> ```


> [!note]- Screenshot
> ```
> © Info
> 
> Burp Suite is now shipped with its own Chromium-based native web browser, which
> is preconfigured to work with all Burp's features. However, for this course we are
> going to exclusively rely on Kali's Firefox browser because it is a more flexible and
> modular option.
> ```


> [!note]- Screenshot
> ```
> © Info
> 
> In some testing scenarios, we might want to capture the traffic from multiple
> machines, so the proxy will be configured on a standalone IP. In such cases, we will
> configure the browser with the external IP address of the proxy.
> ```


> [!note]- Screenshot
> ```
> BS |e mh OE)! sy I Bape Coney. Seings— Masia Fietor) ait
> satis xT
> C4 W Fuel sboutpreferencesngeneral
> Bralunoc MKa Tons Wal Forums A XalDoce WNethurer ote Seey LMSFU ¢ Epot.DB 4 cHOB
> ComeconSetinge x
> Configure Proxy Accessto the Internet
> Nopron
> sto detect roy stings or tienen
> Use pony seinge
> © Hanus pony configuration
> socks Host 127.004 Por| 9060
> 0 s0cKS v4 s0cK5 5
> ‘tomatic roxy consguaton URL
> Ao prom or
> ample: mozilla.org neti, 523681.0728
> Connscton to lcs, 12.00.18 nd arene prone
> te cot
> Figure 8: Firefox Proxy Configuration.
> With Burp configured as a proxy on our browser, we can close any extra open Firefox
> tabs and browse to http://www.megacorpone.com. We should find the intercepted
> traffic in Burp Suite under Proxy > HTTP History.
> ```


> [!note]- Screenshot
> ```
> Figure 9: Burp Suite HTTP History
> We can now review the various requests our browser performed towards our target
> website.
> ```


> [!note]- Screenshot
> ```
> A Warning
> 
> If the browser hangs while loading the page, Intercept may be enabled. Switching it
> off will allow the traffic to flow uninterrupted. As we browse to additional pages, we
> should observe more requests in the HTTP History tab.
> ```


> [!note]- Screenshot
> ```
> _ Figure 10: Inspecting the first HTTP request.
> 
> On the left pane we can visualize the client request details, with the server response on
> the right pane. With this powerful Burp feature, we can inspect every detail of each
> request performed, along with the response. We'll make use of this feature often during
> upcoming Modules.
> ```


> [!note]- Screenshot
> ```
> Q Tip
> 
> Why does "detectportal.firefox.com" keep showing up in the proxy history?
> 
> A captive portalis a web page that serves as a sort of gateway page when
> attempting to browse the Internet. It is often displayed when accepting a user
> agreement or authenticating through a browser to a Wi-Fi network.
> 
> To ignore this, simply enter about:config in the address bar. Firefox will present a
> warning, but we can proceed by clicking / accept the risk!.
> 
> Finally, search for "network.captive-portal-service.enabled" and double-click it to
> change the value to "false". This will prevent these messages from appearing in the
> proxy history.
> ```


> [!note]- Screenshot
> ```
> Burp Project Intruder Repeater Window Help
> Dashboard Target —_—Proxy_—intruder «Repeater «Sequencer. _-—«‘Decoder_«=— Comparer
> Intercept __HTTPhistory __ WebSocketshistory Options
> Filter: Hiding CSS, image and general binary content
> # Host Method URL Params Edited
> jhttp://www.megacorpone.com GET,
> 2 http: www.megacorpone.com GET] _htp:/mmw.megacorpone.com/
> fs ttp-www.megacorpone.com GET
> + ttpeiwww.megacorpone.com  GET| _ Addtoscope
> 5 ttp-iAwww.megacorpone.com GET
> J ttpewww.megacorpone.com GET
> 7 ttp-Awww.megacorpone.com  GET| _Sendtointruder ct
> 1s ttp-www.megacorpone.com GET eR
> ‘Sendto Sequencer
> ‘Send to Comparer (request)
> Send to Comparer (response)
> Show response in browser
> Request in browser >
> Request Engagement tools [Proversion only] >
> Es. Gn = Show new history window
> 1 GET / HITP/1.1 ‘Add comment
> 2 Host: www.megacorpone, com Highlight 5
> User-Agent: Mozilla/S.o (x11; La DLOL Firefox/91.0
> 4 Accept Delete item
> text/html, application/xhtml+xml) — clearhistory >o4/4:q=0.8
> 5 Accept -Language: en-US,en:q=0.5
> Accept-Encoding: gzip, deflate Copy URL
> 7 Connection: close copy ascurt nd
> 8 Upgrade-Insecure-Requests: 1 ‘opyascurlcomma
> Tf-Modified-Since: Wed, 06 Nov 2 — Copylinks
> fio If-None-Match: "390b-596aedca79i
> 1 Cache-Control: max-age=0 Saeitem
> h2 Proxy history documentation
> Figure 11: Sending a Request to Repeater
> ```


> [!note]- Screenshot
> ```
> If we click on Repeater, we will observe one sub-tab with the request on the left side of
> the window. We can send multiple requests to Repeater, and it will display them using
> separate tabs. Let's send the request to the server by clicking Send.
> Cnaed —Tonget Pry Ider Repeater Samienar Deen Camprer Logarta Puectopns—Usropans Laan
> me ev =
> UStr-agents abestta/S.0 (odd; Linux x96_64; rv:52.0) cecko/2oi00101 Firefox/92.0
> & festUhet ppt scaionahenaatappctson/aan0,9,snnge/web 4/3 000.8
> scent ereodin: oF ps deflate
> Upsrade-Tnsecure-Requests: 1
> Shadi ried-since: ed, 65 Noy 2019 15:0814 ot
> 3¥-Nonestetch: "0b-SGeaede479700-o8ip"
> 1 Goche"eanerat: mow age=t
> Figure 12: Burp Suite Repeater
> ```


> [!note]- Screenshot
> ```
> Burp Suite will display the raw server response on the right side of the window, which
> includes the response headers and un-rendered response content.
> Diced —Taget hy rer peer) Sqn _Declr copa Laget_Aenreepens ropes Lem
> =n
> Request Response —
> UssrcAgents Sees e/S.0 TXDL; Linux 186.64; r¥:81.0) Gecko/20100L61 server! Apacke/2, 4.98 (Orbamnl
> rece cb soeredeaa7a0 gripe nt OT
> Set ees be as a:04 14 Bloomers
> } 2 TSE. aunt ss
> EEE tee cote
> — a ee
> Figure 13: Burp Suite Repeater with Request and Response
> ```


> [!note]- Screenshot
> ```
> © Info
> 
> Some web applications will include their hostname in links and redirects. If we don't
> have an entry in /etc/hosts that matches the web application's hostname, our
> browser and other tools may not be able to follow these links.
> ```


> [!note]- Screenshot
> ```
> kali@kali:~§ cat /etc/hosts
> 192.168.50.16 offsecup
> Listing 5 - Setting up our /etc/hosts file for offsecwp
> ```


```sh
cat /etc/hosts
```


> [!note]- Screenshot
> ```
> Log In. Offsec—WordPress x +
> e ca © & ©& offsecwp/wp-login.php &
> BUKaliLinux EVKali Tools AN KaliForums Kali Docs BNetHunter  OffensiveSecurity LMSFU 4 Exploit-DB # GHDB
> 
> Username or Email Address
> 
> admin
> 
> Password
> 
> ecco @
> 
> Remember Me
> Register| Lost your password?
> Figure 14: Simulating a failed WordPress login
> ```


> [!note]- Screenshot
> ```
> Returning to Burp, we'll navigate to Proxy > HTTP History, right-click on the POST
> request to /wp-login.php and select Send to Intruder.
> Usebons —Tavst_ ony __‘tcir—“Rpaser—Saguencer—Oeeter Camper Lager —ctoptons——Useroptns ssn
> merce —_ATTPhstny) _Websecetshston Opn
> Ferangs, image and generar etek
> tec cet eps " 200 607M slp Lig nttonccicet
> 3 hipsicon Get Japadminnagenaipessogsn. yeoman
> 4 tpt rest Papas Se pint pip Logneaqucofteces.
> Peposccupip gph
> SendtoRepeater 7
> Show responseln rose a:
> Pi om Ensigenent io [Rover >
> Post /up-Login.php HITP/2.1 Shon nw hwnd
> Gfercagents meitia/S.0 (x12; Linux 186.68; rv:5t.0) dscmmest avez cr
> Gecko/Boroox01 Forefon/sh.0 sigh
> 4 secent seem 5:00:00 okt
> ere /htsl, app icatson/shtal oun, spplication/eat:ar0.9,snage/v lstcrevalidate, wax-age-0
> recep Cangunge: en-U5.en/5°0.5 senator ee aa
> decept-sncoding: quip, deflate copyurt
> Goncent-Typesspplaestion/e-weeforn-urLenceded copycat command
> 1 Shelf cobs tne cookin oo
> Sordpresy Logged in, {fabsat fabesosanngo16sb2eSed0dce= -
> foleksle) : omutcres OBE] matches
> Figure 15: Sending the POST request to Intruder
> ```


> [!note]- Screenshot
> ```
> We can now select the /ntruder tab in the upper bar, choose the POST request we want
> to modify, and move to the Positions sub-tab. Knowing that the user adminis correct,
> we only need to brute force the password field. First, we'll press Clear on the right bar
> so that all fields are cleared. We can then select the value of the pwd key and press the
> Add button on the right.
> 
> —,
> ```


> [!note]- Screenshot
> ```
> kaligkali:~§ cat /usr/share/wordlists/rockyou.txt | head
> 123456
> 12345
> 123456789
> password
> iloveyou
> princess
> 1234567
> rockyou
> 12345678
> abc123
> Listing 6 - Copying the first 10 rockyou wordlist values
> ```


```sh
cat /usr/share/wordlists/rockyou.txt | head
```


> [!note]- Screenshot
> ```
> Burp Project Intruder Repeater. Window Help
> Dashboard Target «Proxy _—iintruder_—=éRepeater Sequencer Decoder
> 1x 3x
> Target Positions _—_Payloads Resource Pool Options
> You candefine one or more payload sets. The number of payload sets depends on the attacktype
> Payloadset: [7 Z| Payloadcount: 10
> Payloadtype: { Simple list Request count: 10
> ®
> This payload type lets you configure a simple list of strings that are used as payloads.
> Paste 123456
> 12345
> Load 123456789
> Remove) |Password
> iloveyou
> Clear princess
> 1234567
> Deduplicate _} | rockyou
> 12345678
> abet23
> Add
> Figure 15: Pasting the first 10 rockyou entries
> ```


> [!note]- Screenshot
> ```
> Attack Save Columns
> 
> Results Target Positions —Payloads Resource Pool Options
> Filter: Showing all items
> 
> Request Payload Status Error_Timeout Length Comment
> o 200 6462
> h 123456 200 6462
> 2 12345 200 6462
> 3 123456789 200 6462
> la password 302 1097
> 5 iloveyou 200 e462
> 6 princess 200 6462
> 7 1234567 200 6462
> 8 rockyou 200 6462
> 9 12345678 200 6462
> 10 abet23 200 6462
> 
> Figure 15: Inspecting Intruder's attack results
> ```


> [!note]- Screenshot
> ```
> j sa © & & offsecwp
> 
> BNKali Linux EXKaliTools HRKaliForums ff Kali Docs @ENetHunter Offensive Security J MSFU # Exploit-DB #GHDB
> 
> D fi ots O6 Po + he
> 
> = | salable
> 
> oa Dashboard
> 
> # Post This theme suggests the following plugin:
> 
> 9) Media q
> 
> IB Pages
> 
> B Comments Welcome to WordPress!
> 
> # Appearance Weve assembled some ln . ‘
> 
> rane Get Started Next Steps
> 
> © Users ce
> 
> # Tool: +
> 
> Bi Setting ° a
> 
> ° : co]
> Figure 15: Logging to the WP admin console
> ```


> [!note]- Screenshot
> ```
> © Caution
> If we set Firefox to use Burp as a proxy, and we close Burp, Firefox won't work
> correctly.
> ```
