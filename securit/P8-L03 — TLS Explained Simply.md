# DevOps Task #13

## P8-L03 — TLS Explained Simply

**Student:** Hamad Tarawa  
**Stage:** Part 8 — Network Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Local Computer

---

## 1. Connection to the Previous Lesson

In **P8-L02 — HTTP and HTTPS**, you learned:

```text
HTTPS = HTTP + TLS
```

HTTP defines web requests and responses.

TLS protects those messages while they travel across the network.

You also learned that HTTPS provides:

- Confidentiality
- Integrity
- Server authentication

This lesson explains how TLS creates that protection without requiring
advanced cryptography.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain what TLS means.
- Explain the relationship between TLS and HTTPS.
- Describe confidentiality, integrity, and server authentication.
- Explain the TLS handshake at a beginner level.
- Explain why TLS creates session keys.
- Describe where TLS protection starts and ends.
- Explain why outdated TLS versions should be disabled.
- Identify basic TLS evidence in `curl` output.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer may configure TLS on:

- Nginx
- Apache
- A load balancer
- A Kubernetes Ingress or Gateway
- An API gateway
- A cloud application endpoint
- Internal service connections

The engineer may also be responsible for:

- Installing certificates.
- Protecting certificate private keys.
- Renewing certificates.
- Enabling supported TLS versions.
- Redirecting HTTP to HTTPS.
- Troubleshooting failed secure connections.
- Deciding where TLS terminates.

You do not need to become a cryptography specialist.

You do need a correct mental model of what TLS protects and what it does not
protect.

---

## 4. Simple Explanation

### 4.1 What does TLS mean?

TLS means:

```text
Transport Layer Security
```

TLS is a security protocol that protects communication between two network
endpoints.

For HTTPS:

```text
Browser
   |
   | TLS-protected HTTP communication
   v
HTTPS server
```

The HTTPS server might be:

- Nginx
- A load balancer
- A reverse proxy
- An API gateway
- The application itself

---

### 4.2 TLS is not HTTP

HTTP and TLS have different jobs.

| Technology | Main job |
|---|---|
| HTTP | Defines requests, responses, methods, paths, headers, and bodies |
| TLS | Protects network communication between endpoints |
| HTTPS | HTTP communication carried through TLS |

Example:

```text
GET /api/courses
Authorization: Bearer token
```

HTTP defines the request.

TLS protects that request while it travels between the TLS client and TLS
server.

---

## 5. Protection 1 — Confidentiality

Confidentiality means:

> Unauthorized observers should not be able to read the communication.

Without TLS:

```text
Client -------- unprotected HTTP data --------> Server
```

With TLS:

```text
Client ===== encrypted communication ========> Server
```

Sensitive data may include:

- Passwords
- JWT access tokens
- Session cookies
- Personal information
- API request bodies
- API responses

The client and server can read the application data because they are the
communication endpoints.

An observer between them should see protected encrypted traffic instead of the
original readable HTTP message.

---

## 6. Protection 2 — Integrity

Integrity means:

> Unauthorized changes to the communication should be detected.

Suppose a server sends:

```json
{
  "account": "hamad",
  "role": "USER"
}
```

TLS helps prevent someone in the network path from silently changing it to:

```json
{
  "account": "hamad",
  "role": "ADMIN"
}
```

and delivering the modified message as if nothing happened.

If protected TLS data is altered incorrectly during transmission, the
integrity checks fail and the communication is rejected.

TLS integrity protects data in transit.

It does not prevent the authorized server itself from returning incorrect
data due to an application bug.

---

## 7. Protection 3 — Server Authentication

Server authentication helps the client answer:

> Am I communicating with the server for the domain I requested?

The server presents a digital certificate.

The client checks information such as:

- Is the certificate valid for this domain?
- Is it currently valid?
- Is it connected to a trusted certificate chain?
- Has the certificate validation succeeded?

Simplified example:

```text
Browser requests:
student.example.com
        |
        v
Server presents certificate
        |
        v
Browser verifies certificate for student.example.com
        |
    +---+---+
    |       |
 Valid   Invalid
    |       |
    v       v
Continue  Warning or failure
```

Certificates and certificate trust will be explained in detail in:

**P8-L04 — Certificates and Trust**

---

## 8. The TLS Handshake

Before protected application communication begins, the client and server
perform a TLS handshake.

The handshake is a setup conversation.

Its main purposes are:

1. Agree on secure communication settings.
2. Authenticate the server using its certificate.
3. Establish shared session keys.
4. Begin protected communication.

### Simplified handshake

```text
1. Client contacts server
          |
          v
2. Server presents certificate
          |
          v
3. Client verifies certificate
          |
          v
4. Client and server establish session keys
          |
          v
5. Encrypted application communication begins
```

This is the correct beginner mental model.

Real TLS handshakes contain more technical messages and details, but you do not
need them for this DevOps lesson.

---

## 9. Step-by-Step Handshake Explanation

### Step 1 — The client starts the connection

The client contacts the HTTPS server, normally on TCP port `443`.

It indicates which secure communication options it supports.

```text
Client:
"I want to create a secure connection."
```

---

### Step 2 — The server responds

The server selects supported communication settings and presents its
certificate.

```text
Server:
"Here is my certificate and the selected secure settings."
```

---

### Step 3 — The client verifies the certificate

The client checks whether it should trust the certificate for the requested
domain.

If verification fails, the client should not silently continue as if the
connection were trustworthy.

Possible causes of failure include:

- Certificate expired.
- Wrong domain name.
- Unknown or untrusted issuer.
- Incomplete certificate chain.

---

### Step 4 — Session keys are established

The client and server securely establish shared session keys.

These keys are used to protect the application data exchanged during that
connection.

The keys are not the same thing as:

- The user's password.
- A JWT access token.
- The server certificate.
- The certificate private key.

They are temporary cryptographic material used for that secure session.

---

### Step 5 — Secure communication begins

After the handshake succeeds, HTTP requests and responses travel through the
TLS-protected connection.

```text
TLS handshake completes
          |
          v
GET /api/courses
          |
          v
Encrypted transmission
          |
          v
HTTP response
```

---

## 10. Why TLS Uses Session Keys

At a simple level, TLS uses different cryptographic techniques for different
jobs.

The certificate and related key mechanisms help with identity and secure
session setup.

After setup, temporary session keys efficiently protect the application data.

The beginner mental model is:

```text
Certificate
   |
   | helps authenticate server
   v
Secure handshake
   |
   | establishes temporary keys
   v
Session keys
   |
   | protect requests and responses
   v
Encrypted communication
```

You do not need to calculate or manually create the TLS session keys.

The TLS software handles this process automatically.

---

## 11. Where TLS Protection Starts and Ends

TLS protects communication between two TLS endpoints.

Consider:

```text
Browser
   |
   | HTTPS
   v
Nginx
   |
   | HTTP
   v
Spring Boot
```

In this architecture:

- TLS protects browser-to-Nginx traffic.
- TLS ends at Nginx.
- Nginx decrypts the request.
- Nginx forwards the request to Spring Boot using the configured internal
  connection.

This is called **TLS termination** at Nginx.

### Important security question

If the Nginx-to-Spring-Boot network is not fully trusted, the DevOps engineer
may also need TLS for the internal connection:

```text
Browser
   |
   | HTTPS
   v
Nginx
   |
   | HTTPS
   v
Spring Boot
```

The correct choice depends on the architecture and risk.

The important idea is:

> TLS protects a connection between its endpoints. It does not automatically
> protect every later connection in the system.

---

## 12. TLS and the Database

Suppose this application uses:

```text
Browser -> Nginx -> Spring Boot -> PostgreSQL
```

Browser HTTPS does not automatically encrypt the Spring Boot-to-PostgreSQL
connection.

Each connection has its own security decision:

| Connection | Possible protection |
|---|---|
| Browser to Nginx | HTTPS/TLS |
| Nginx to Spring Boot | Internal HTTP or HTTPS |
| Spring Boot to PostgreSQL | Database TLS where required |

This is why a DevOps engineer maps every network connection instead of only
checking whether the browser shows HTTPS.

---

## 13. Old and Unsupported TLS Versions

Security protocols improve over time.

Older SSL and TLS versions have known weaknesses and should not remain enabled
merely to support obsolete clients.

A secure production configuration should:

- Use currently supported TLS versions.
- Disable obsolete SSL and TLS versions.
- Use secure cipher configuration.
- Keep TLS software updated.
- Test compatibility before removing old support.

For modern environments, TLS 1.2 and TLS 1.3 are commonly supported.

The exact configuration should follow:

- Current vendor documentation.
- Organizational policy.
- The needs of supported clients.

Do not copy an old TLS configuration from an outdated tutorial without
reviewing it.

---

## 14. Important Terms

| Term | Simple meaning |
|---|---|
| TLS | Protocol that protects network communication |
| HTTPS | HTTP carried through TLS |
| Confidentiality | Preventing unauthorized reading of data |
| Integrity | Detecting unauthorized changes to data |
| Authentication | Verifying identity |
| TLS handshake | Setup process before protected communication |
| Certificate | Digital document used to help authenticate the server |
| Session key | Temporary key used to protect a secure session |
| Encryption | Transforming data so unauthorized observers cannot read it |
| TLS endpoint | System where TLS protection begins or ends |
| TLS termination | Decrypting TLS traffic at a selected endpoint |
| Cipher | Cryptographic method used to protect communication |

---

## 15. Simple Real-World Example

Consider your React and Spring Boot application:

```text
React in browser
       |
       | HTTPS :443
       v
Nginx reverse proxy
       |
       | Internal HTTP :8080
       v
Spring Boot
       |
       | Database connection :5432
       v
PostgreSQL
```

### What TLS protects

If Nginx terminates TLS:

```text
Protected:
Browser <==== TLS ====> Nginx

Not automatically protected by that TLS session:
Nginx ------> Spring Boot
Spring Boot -> PostgreSQL
```

### DevOps decisions

The DevOps engineer must:

1. Install the correct certificate on Nginx.
2. Protect the certificate private key.
3. Enable supported TLS versions.
4. Redirect HTTP traffic to HTTPS.
5. Decide whether internal connections also require TLS.
6. Monitor certificate expiration.
7. Renew the certificate before it expires.

---

## 16. Commands and Configuration

### Command 1 — Observe an HTTPS connection with curl

```bash
curl -v https://example.com
```

Purpose:

> Display detailed connection, TLS, request, and response information.

Look for lines mentioning:

- Port `443`
- TLS
- Certificate
- HTTP request
- HTTP response

The exact output depends on the installed `curl` and TLS library.

---

### Command 2 — Display only response headers

```bash
curl -I https://example.com
```

Purpose:

> Establish an HTTPS connection and display the HTTP response headers.

The TLS handshake occurs before `curl` receives the protected HTTP response.

---

### Command 3 — Require a supported TLS version

For a simple test, `curl` can request a minimum TLS version:

```bash
curl --tlsv1.2 -I https://example.com
```

Purpose:

> Request a connection using TLS 1.2 or newer, depending on the `curl`
> implementation and server negotiation.

This does not modify your system or the server.

Do not interpret this command as proof that the server supports only TLS 1.2.
The final negotiated version may be newer.

---

## 17. Safe Practical Lab

### Lab goal

Observe a real TLS-protected HTTPS connection without changing any
configuration.

### Step 1 — Send an HTTPS request

Run:

```bash
curl -I https://example.com
```

Record the HTTP status code.

### Step 2 — Display verbose TLS information

Run:

```bash
curl -v https://example.com
```

Look for:

- Connection to port `443`
- TLS version
- Certificate subject or domain information
- Certificate verification result
- HTTP response status

### Step 3 — Test a modern minimum version

Run:

```bash
curl --tlsv1.2 -I https://example.com
```

Record whether the connection succeeds.

### Step 4 — Build the handshake explanation

Write:

```text
1. My curl client contacted the HTTPS server.
2. The server presented its certificate.
3. The client verified the certificate.
4. TLS established temporary session keys.
5. The HTTP request and response travelled through the protected connection.
```

### Lab safety

This lab:

- Uses a public demonstration domain.
- Sends no credentials.
- Changes no local configuration.
- Changes no remote configuration.
- Performs no attack.

### Evidence warning

Do not run verbose `curl` commands containing:

- Real authorization tokens
- Session cookies
- API keys
- Passwords

Verbose output may display request headers.

---

## 18. Expected Result

Your verbose output may include information similar to:

```text
Connected to example.com (...) port 443
SSL connection using TLS...
server certificate verification OK
> GET / HTTP/...
< HTTP/... 200
```

The wording differs between:

- `curl` versions
- TLS libraries
- Operating systems
- Server configurations

The important evidence is:

```text
Connection to port 443
        |
        v
TLS negotiation
        |
        v
Certificate verification
        |
        v
Protected HTTP exchange
```

---

## 19. Evidence to Save

Save one screenshot containing non-sensitive output from:

```bash
curl -I https://example.com
curl --tlsv1.2 -I https://example.com
```

Optionally save selected TLS lines from:

```bash
curl -v https://example.com
```

### Suggested evidence title

```text
TLS-Protected HTTPS Connection
```

### Suggested evidence explanation

> I used `curl` to establish an HTTPS connection to port 443. The verbose
> output showed TLS negotiation and certificate verification before the HTTP
> response was received. TLS provides confidentiality, integrity, and server
> authentication for data travelling between the client and HTTPS endpoint.

### Architecture evidence

Include:

```text
Browser <==== HTTPS/TLS ====> Nginx
Nginx  ------ internal -----> Spring Boot
```

Then explain:

> If TLS terminates at Nginx, the same TLS session does not automatically
> protect the later Nginx-to-Spring-Boot connection.

---

## 20. Common Mistakes

### Mistake 1 — Thinking TLS and HTTP are the same thing

HTTP defines application messages.

TLS protects network communication.

HTTPS combines them.

---

### Mistake 2 — Thinking TLS encrypts data everywhere

TLS protects communication between its endpoints.

After termination, later connections need their own protection decisions.

---

### Mistake 3 — Ignoring certificate errors

A certificate error may mean the client cannot verify the intended server.

Do not disable verification merely to remove the error.

---

### Mistake 4 — Believing encryption alone proves server identity

Certificate validation is needed to authenticate the server.

An encrypted connection to the wrong server is not the intended security
result.

---

### Mistake 5 — Keeping obsolete protocol versions enabled

Old versions should not remain active without a supported requirement and
security review.

---

### Mistake 6 — Confusing session keys with user credentials

TLS session keys protect network communication.

They are not passwords, JWTs, or API keys.

---

### Mistake 7 — Assuming TLS fixes application vulnerabilities

TLS does not fix:

- Broken access control
- SQL injection
- Insecure token storage
- Exposed secrets
- Vulnerable dependencies

It protects data in transit.

---

## 21. Task Report Notes

You may adapt the following text for your final assignment:

> TLS, or Transport Layer Security, is the security protocol used by HTTPS.
> HTTP defines the request and response messages, while TLS protects those
> messages during network transmission. TLS provides confidentiality,
> integrity, and server authentication.

> Before protected communication begins, the client and server perform a TLS
> handshake. The server presents a certificate, the client verifies it, and
> the two systems establish temporary session keys. These keys are then used
> to protect the HTTP requests and responses exchanged during the session.

> TLS protects communication only between its endpoints. If Nginx terminates
> TLS and forwards a request to Spring Boot using HTTP, the browser-to-Nginx
> TLS session does not automatically protect the internal connection. DevOps
> engineers must evaluate every connection and use supported TLS versions,
> valid certificates, protected private keys, and appropriate internal
> network security.

### Practical demonstration note

> I used `curl -v` to observe a TLS-protected connection to an HTTPS server on
> port 443. The output showed TLS negotiation and certificate verification
> before the HTTP response was returned. I also used `curl --tlsv1.2` to
> request a modern minimum TLS version without changing any configuration.

---

## 22. Five Review Questions

1. What is TLS, and how is it related to HTTPS?
2. What three main protections does TLS provide?
3. What happens during the simplified TLS handshake?
4. What is the purpose of a TLS session key?
5. If TLS terminates at Nginx, does it automatically protect the
   Nginx-to-Spring-Boot connection? Explain.

---

## 23. Lesson Summary

TLS means:

```text
Transport Layer Security
```

It protects network communication with:

- Confidentiality
- Integrity
- Server authentication

The simplified handshake is:

```text
Client contacts server
        |
        v
Server presents certificate
        |
        v
Client verifies certificate
        |
        v
Session keys are established
        |
        v
Protected communication begins
```

The most important DevOps idea is:

> TLS protects communication between its endpoints. Every later network
> connection must be evaluated separately.

Important commands:

```bash
curl -I https://example.com
curl -v https://example.com
curl --tlsv1.2 -I https://example.com
```

---

## 24. Progress

### Stage 1 — Part 7: Linux Security

- [x] P7-L01 through P7-L10 completed

### Stage 2 — Part 8: Network Security

- [x] P8-L01 — The Path of a Web Request
- [x] P8-L02 — HTTP and HTTPS
- [x] P8-L03 — TLS Explained Simply
- [ ] P8-L04 — Certificates and Trust
- [ ] P8-L05 — Inspecting HTTPS, TLS, and Certificates
- [ ] P8-L06 — DNS Security
- [ ] P8-L07 — Firewalls and Reverse Proxy Security
- [ ] P8-L08 — Common Network Attacks at a High Level
- [ ] P8-L09 — Part 8 Network Security Report

---

## Exact Next Lesson

**P8-L04 — Certificates and Trust**

Do not continue until the student writes:

```text
next
```
