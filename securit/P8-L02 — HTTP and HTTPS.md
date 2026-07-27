# DevOps Task #13

## P8-L02 — HTTP and HTTPS

**Student:** Hamad Tarawa  
**Stage:** Part 8 — Network Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Local Computer

---

## 1. Connection to the Previous Lesson

In **P8-L01 — The Path of a Web Request**, you followed this simplified path:

```text
User
  |
  v
DNS
  |
  v
Internet
  |
  v
Firewall
  |
  v
Reverse Proxy
  |
  v
Frontend or Backend
  |
  v
Database
```

You learned that the network carries the request between the client and
server.

This creates an important security question:

> Can another person read or modify the data while it travels across the
> network?

HTTP sends web requests and responses.

HTTPS uses HTTP together with TLS protection to secure the communication.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain the purpose of HTTP.
- Explain the purpose of HTTPS.
- Distinguish HTTP from HTTPS.
- Explain plaintext communication.
- Explain encrypted communication.
- Explain why passwords and access tokens must not travel over HTTP.
- Explain why web applications redirect HTTP traffic to HTTPS.
- Inspect HTTP and HTTPS response headers using `curl`.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer commonly configures:

- Nginx or another reverse proxy
- Domain names
- TLS certificates
- HTTP and HTTPS ports
- Redirect rules
- Application environment variables
- Load balancers and ingress controllers

Consider a login request:

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "hamad@example.com",
  "password": "my-password"
}
```

If this request is sent over unprotected HTTP, sensitive information travels
without TLS encryption.

If it is sent over HTTPS, TLS protects the communication while it travels
between the client and HTTPS endpoint.

Correct HTTPS configuration is therefore a basic DevOps responsibility.

---

## 4. Simple Explanation

### 4.1 What is HTTP?

HTTP means:

```text
Hypertext Transfer Protocol
```

HTTP defines how a client and web server exchange requests and responses.

The client might be:

- A browser
- A React application
- A mobile application
- Postman
- `curl`
- Another backend service

The server might be:

- Nginx
- Spring Boot
- Node.js
- An API gateway
- A cloud load balancer

### Simple HTTP exchange

```text
Client
  |
  | HTTP request
  v
Server
  |
  | HTTP response
  v
Client
```

HTTP describes the web message format, including:

- Method
- Path
- Headers
- Body
- Status code

---

### 4.2 What is an HTTP request?

An HTTP request asks the server to perform an action.

Example:

```http
GET /api/courses HTTP/1.1
Host: student.example.com
Accept: application/json
```

Important parts:

| Part | Example | Meaning |
|---|---|---|
| Method | `GET` | Requested action |
| Path | `/api/courses` | Requested resource |
| Protocol version | `HTTP/1.1` | HTTP version |
| Header | `Accept: application/json` | Additional request information |

A request may also contain a body.

Example:

```http
POST /api/courses HTTP/1.1
Content-Type: application/json

{
  "name": "DevOps Security"
}
```

---

### 4.3 What is an HTTP response?

An HTTP response contains the server's result.

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "DevOps Security"
  }
]
```

Important parts:

| Part | Example | Meaning |
|---|---|---|
| Status code | `200` | Request succeeded |
| Status text | `OK` | Human-readable result |
| Header | `Content-Type` | Type of returned data |
| Body | JSON data | Returned content |

HTTP itself defines these messages. It does not automatically encrypt them.

---

## 5. What is Plaintext Communication?

Plaintext means data is sent in a form that is not protected by encryption.

For a normal HTTP connection:

```text
Client
  |
  | HTTP data without TLS encryption
  v
Server
```

If an unauthorized person can observe the network traffic, unprotected data
may be readable.

Sensitive examples include:

- Usernames
- Passwords
- Session cookies
- JWT access tokens
- Personal information
- API request and response data

### Important clarification

Plaintext does not mean the user literally sees the message as plain text on
the screen.

It means the connection does not use TLS encryption to protect the HTTP data
while it travels across the network.

---

## 6. What is HTTPS?

HTTPS means:

```text
HTTP Secure
```

HTTPS is HTTP communication protected by TLS.

The relationship is:

```text
HTTPS = HTTP + TLS protection
```

HTTP still defines:

- Request methods
- Paths
- Headers
- Bodies
- Status codes

TLS adds network security around that communication.

```text
HTTP request and response
          |
          v
TLS protection
          |
          v
Network communication
```

---

## 7. What HTTPS Protects

HTTPS provides three major protections.

### 7.1 Confidentiality

Confidentiality means unauthorized observers should not be able to read the
communication.

Example:

```text
Login password
      |
      | encrypted connection
      v
HTTPS server
```

---

### 7.2 Integrity

Integrity helps detect unauthorized changes to data while it travels.

The client should receive the response created by the server rather than a
response secretly modified during transmission.

---

### 7.3 Server authentication

The server presents a digital certificate.

The client checks whether the certificate is valid for the requested domain
and trusted under its certificate-validation rules.

This helps the client verify that it connected to the intended server.

Certificates will be explained in:

- **P8-L03 — TLS Explained Simply**
- **P8-L04 — Certificates and Trust**

---

## 8. HTTP vs HTTPS

| Question | HTTP | HTTPS |
|---|---|---|
| Full name | Hypertext Transfer Protocol | HTTP Secure |
| Common port | `80` | `443` |
| TLS encryption | No | Yes |
| Protects data in transit | No TLS protection | Yes, when correctly configured |
| Certificate used | No | Yes |
| Suitable for passwords and tokens | No | Yes |
| URL begins with | `http://` | `https://` |

### Mental model

```text
HTTP
Client -------- readable network data --------> Server

HTTPS
Client ======== TLS-protected connection ======> Server
```

The second line does not mean the application data becomes unreadable to the
client or server.

The client creates the request, and the server processes it.

TLS protects the data while it moves between the TLS endpoints.

---

## 9. Important Terms

| Term | Simple meaning |
|---|---|
| HTTP | Protocol for web requests and responses |
| HTTPS | HTTP protected by TLS |
| Plaintext | Data not protected by encryption |
| Encryption | Transforming data so unauthorized observers cannot read it |
| TLS | Security layer used by HTTPS |
| HTTP method | Requested action such as `GET` or `POST` |
| Header | Additional information attached to a request or response |
| Body | Main data carried by a request or response |
| Status code | Number describing the response result |
| Certificate | Digital document used to help authenticate the server |
| Redirect | Response instructing the client to use another URL |
| Data in transit | Data moving between systems across a network |

---

## 10. Simple Real-World Example

Suppose your React application sends a login request to Spring Boot.

### Unsafe design

```text
React
  |
  | HTTP POST /api/auth/login
  | email + password
  v
Spring Boot
```

The HTTP connection does not use TLS encryption.

### Safer design

```text
React browser client
       |
       | HTTPS :443
       v
Nginx reverse proxy
       |
       | Internal controlled connection
       v
Spring Boot :8080
```

Nginx may terminate TLS:

1. The browser connects securely to Nginx using HTTPS.
2. Nginx presents the certificate.
3. TLS protects the external network connection.
4. Nginx forwards the request to Spring Boot through the configured internal
   network.

The internal network must also be protected appropriately. A private network
does not automatically make every internal connection secure.

---

## 11. Why Passwords and Tokens Must Not Use HTTP

Your applications may send:

- Login passwords
- JWT access tokens
- Refresh tokens
- Session cookies
- API keys

Example authorization header:

```http
Authorization: Bearer eyJhbGciOi...
```

The token is a credential.

Anyone who obtains a valid token may be able to act as the authenticated user
until the token expires or is revoked.

Sending it over HTTP means the connection does not provide TLS protection.

Therefore:

> Credentials and sensitive application data must use HTTPS during network
> transmission.

### HTTPS does not fix token-storage mistakes

HTTPS protects data while it travels.

It does not automatically protect:

- A token stored insecurely in the browser.
- A token printed in application logs.
- A token committed to Git.
- A token exposed through vulnerable application code.

Those problems require additional security controls.

---

## 12. HTTP-to-HTTPS Redirection

A user may type:

```text
http://student.example.com
```

A web server can return a redirect telling the browser to use:

```text
https://student.example.com
```

### Simplified flow

```text
Browser requests HTTP
         |
         v
Server returns redirect
         |
         v
Browser requests HTTPS
         |
         v
Secure connection begins
```

### Example response

```http
HTTP/1.1 301 Moved Permanently
Location: https://student.example.com/
```

The `Location` header tells the client where to go.

### Important limitation

The first HTTP request is still an HTTP request.

Applications should:

- Use HTTPS links directly.
- Configure redirects.
- Avoid sending sensitive data to HTTP URLs.
- Use additional browser protections such as HSTS where appropriate.

You only need the redirect concept in this lesson.

---

## 13. Example Nginx Redirect Configuration

An Nginx HTTP server block may redirect traffic to HTTPS:

```nginx
server {
    listen 80;
    server_name student.example.com;

    return 301 https://$host$request_uri;
}
```

### Explanation

```nginx
listen 80;
```

Receives HTTP traffic on port `80`.

```nginx
server_name student.example.com;
```

Applies the block to that domain.

```nginx
return 301 https://$host$request_uri;
```

Returns a permanent redirect to the equivalent HTTPS URL.

For example:

```text
http://student.example.com/api/courses?page=2
```

becomes:

```text
https://student.example.com/api/courses?page=2
```

This is a configuration example only. Do not apply it unless the domain,
certificate, and HTTPS server block are correctly configured.

---

## 14. What HTTPS Does Not Guarantee

HTTPS is essential, but it does not mean the entire application is safe.

HTTPS does not automatically prevent:

- Broken access control
- SQL injection
- Weak passwords
- Insecure code
- Malware on the user's computer
- Exposed secrets in Git
- Incorrect database permissions

HTTPS means the network connection to the verified HTTPS endpoint receives TLS
protection.

A malicious website may also use HTTPS.

Therefore, a browser lock icon does not mean:

> Everything on this website is trustworthy.

It mainly means:

> The connection is encrypted, and the certificate was accepted for this
> site.

Application security still requires other controls.

---

## 15. Commands and Configuration

### Command 1 — Inspect an HTTP response

```bash
curl -I http://example.com
```

Purpose:

> Send an HTTP request and display the response headers.

The response might:

- Return content over HTTP.
- Redirect to HTTPS.
- Be blocked by the network environment.

The exact result depends on the site and network.

---

### Command 2 — Inspect an HTTPS response

```bash
curl -I https://example.com
```

Purpose:

> Send an HTTPS request and display the response headers.

`curl` establishes TLS before exchanging the HTTP message.

---

### Command 3 — Follow redirects

```bash
curl -L -I http://example.com
```

Options:

- `-L`: Follow redirects.
- `-I`: Display response headers only.

If the site redirects HTTP to HTTPS, the output may display multiple response
header blocks:

```text
Initial HTTP response
        |
        v
Redirect response
        |
        v
Final HTTPS response
```

---

### Command 4 — Show verbose HTTPS connection details

```bash
curl -v https://example.com
```

The output may show:

- Connection to port `443`
- TLS negotiation
- Certificate information
- HTTP request
- HTTP response

Do not include real access tokens, passwords, or sensitive headers in verbose
commands used for public evidence.

---

## 16. Safe Practical Lab

### Lab goal

Compare HTTP and HTTPS behavior for a public demonstration domain.

### Step 1 — Inspect HTTP

Run:

```bash
curl -I http://example.com
```

Record:

- Status code
- Any `Location` header
- Whether the response redirects

### Step 2 — Inspect HTTPS

Run:

```bash
curl -I https://example.com
```

Record:

- Status code
- HTTP version
- Content type

### Step 3 — Follow any redirect

Run:

```bash
curl -L -I http://example.com
```

Observe whether the final URL uses HTTPS.

### Step 4 — Observe TLS use

Run:

```bash
curl -v https://example.com
```

Look for:

- Port `443`
- TLS or SSL connection information
- Certificate information
- Final HTTP response status

You do not need to understand the handshake details yet.

### Step 5 — Write the comparison

Complete:

```text
HTTP uses port: __________________
HTTPS uses port: _________________
HTTP TLS protection: Yes / No
HTTPS TLS protection: Yes / No
HTTP response redirected: Yes / No
Final response status: __________
```

### Lab safety

This lab:

- Uses a public demonstration domain.
- Sends no credentials.
- Changes no system configuration.
- Performs no security attack.

---

## 17. Expected Result

Possible HTTP output:

```text
HTTP/1.1 301 Moved Permanently
Location: https://example.com/
```

or:

```text
HTTP/1.1 200 OK
```

Possible HTTPS output:

```text
HTTP/2 200
content-type: text/html
```

The exact result may differ because the site's configuration can change.

The conceptual result remains:

```text
http://  -> HTTP without TLS
https:// -> HTTP protected by TLS
```

---

## 18. Evidence to Save

Save one screenshot showing:

```bash
curl -I http://example.com
curl -I https://example.com
curl -L -I http://example.com
```

Optionally save selected non-sensitive verbose output from:

```bash
curl -v https://example.com
```

### Suggested evidence title

```text
HTTP and HTTPS Response Comparison
```

### Suggested evidence explanation

> I sent one request using HTTP and another using HTTPS. HTTP does not provide
> TLS encryption, while HTTPS protects the HTTP communication using TLS. I
> also used `curl -L -I` to observe whether the server redirected an HTTP
> request to an HTTPS URL. Passwords, access tokens, and other sensitive data
> should never be sent through unprotected HTTP.

---

## 19. Common Mistakes

### Mistake 1 — Thinking HTTP and HTTPS are different application APIs

Both use HTTP request and response concepts.

HTTPS adds TLS protection around the communication.

---

### Mistake 2 — Sending login credentials over HTTP

Passwords, tokens, cookies, and personal data require HTTPS protection.

---

### Mistake 3 — Believing a redirect protects data already sent to HTTP

The initial HTTP request occurs before the redirect.

Clients should use HTTPS URLs directly for sensitive operations.

---

### Mistake 4 — Believing HTTPS makes application code secure

HTTPS protects data in transit.

The application still needs authentication, authorization, validation, secure
code, and protected secrets.

---

### Mistake 5 — Disabling certificate validation

Encryption without correct certificate validation may fail to confirm the
intended server.

Clients should not ignore certificate errors merely to make a connection work.

---

### Mistake 6 — Exposing the backend because HTTPS is enabled

HTTPS does not replace network segmentation.

The reverse proxy can remain public while backend and database ports remain
private.

---

### Mistake 7 — Including tokens in verbose evidence

Verbose HTTP tools can display request headers.

Never expose real authorization tokens or cookies in screenshots and reports.

---

## 20. Task Report Notes

You may adapt the following text for your final assignment:

> HTTP is the protocol used to exchange web requests and responses. An HTTP
> message can contain methods, paths, headers, bodies, and status codes.
> Standard HTTP does not use TLS encryption, so sensitive data should not be
> transmitted through an unprotected HTTP connection.

> HTTPS is HTTP protected by TLS. TLS provides confidentiality, integrity, and
> server authentication for data in transit. Login passwords, session cookies,
> JWT access tokens, API keys, and personal information should use HTTPS when
> transmitted over a network.

> Web servers commonly redirect requests from HTTP port 80 to HTTPS port 443.
> However, the initial HTTP request occurs before the redirect, so applications
> should use HTTPS URLs directly and must not send sensitive data to an HTTP
> endpoint.

### Practical demonstration note

> I compared `http://example.com` and `https://example.com` using `curl`. I
> inspected the status and response headers and used `curl -L -I` to follow any
> redirect. The demonstration showed the difference between an HTTP URL and an
> HTTPS connection protected with TLS.

---

## 21. Five Review Questions

1. What is the purpose of HTTP?
2. What protection does HTTPS add to HTTP?
3. Why must passwords and JWT access tokens not travel over HTTP?
4. What does an HTTP-to-HTTPS redirect do?
5. Does HTTPS guarantee that the application has no security vulnerabilities?
   Explain.

---

## 22. Lesson Summary

HTTP defines web requests and responses:

```text
Client -> HTTP request -> Server
Client <- HTTP response <- Server
```

HTTP alone does not provide TLS protection.

HTTPS adds TLS:

```text
HTTPS = HTTP + TLS
```

TLS provides:

- Confidentiality
- Integrity
- Server authentication

The most important rule is:

> Never send passwords, access tokens, session cookies, or other sensitive
> data through unprotected HTTP.

Web servers commonly redirect HTTP traffic to HTTPS, but applications should
use HTTPS directly for sensitive operations.

Important commands:

```bash
curl -I http://example.com
curl -I https://example.com
curl -L -I http://example.com
curl -v https://example.com
```

---

## 23. Progress

### Stage 1 — Part 7: Linux Security

- [x] P7-L01 through P7-L10 completed

### Stage 2 — Part 8: Network Security

- [x] P8-L01 — The Path of a Web Request
- [x] P8-L02 — HTTP and HTTPS
- [ ] P8-L03 — TLS Explained Simply
- [ ] P8-L04 — Certificates and Trust
- [ ] P8-L05 — Inspecting HTTPS, TLS, and Certificates
- [ ] P8-L06 — DNS Security
- [ ] P8-L07 — Firewalls and Reverse Proxy Security
- [ ] P8-L08 — Common Network Attacks at a High Level
- [ ] P8-L09 — Part 8 Network Security Report

---

## Exact Next Lesson

**P8-L03 — TLS Explained Simply**

Do not continue until the student writes:

```text
next
```
