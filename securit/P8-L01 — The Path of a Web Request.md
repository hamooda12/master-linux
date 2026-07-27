# DevOps Task #13

## P8-L01 — The Path of a Web Request

**Student:** Hamad Tarawa  
**Stage:** Part 8 — Network Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Local Computer

---

## 1. Connection to the Previous Lesson

In Part 7, you learned how Linux protects one machine using:

- Users and groups
- File permissions and ownership
- `sudo`
- SSH security
- Firewalls
- Process isolation

Part 8 moves from security **inside a Linux machine** to security **between
systems communicating over a network**.

When a user opens a web application, the request may travel through several
components before the application returns a response.

The simplified path is:

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
Frontend or Backend Service
  |
  v
Database
```

This lesson explains the responsibility and security role of every component.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain what happens when a user opens a website.
- Describe the role of DNS.
- Explain how a request travels across a network.
- Explain where a firewall protects the system.
- Describe the role of a reverse proxy.
- Distinguish a frontend request from a backend API request.
- Explain when the backend communicates with a database.
- Identify the security control protecting each stage.
- Draw a simple secure web-request architecture.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer connects all parts of an application:

- Domain name
- DNS records
- Network
- Firewall
- TLS certificate
- Reverse proxy
- Frontend
- Backend API
- Database

When a request fails, the problem may exist at any one of these stages.

Examples:

```text
Wrong DNS record
Firewall blocks port 443
Expired TLS certificate
Nginx cannot reach Spring Boot
Spring Boot cannot connect to PostgreSQL
```

Understanding the request path helps a DevOps engineer:

- Design secure systems.
- Configure network access.
- Troubleshoot failures in a logical order.
- Avoid exposing internal services.
- Explain where each security control operates.

---

## 4. Simple Explanation

Imagine that a user enters:

```text
https://app.example.com
```

into a browser.

The browser cannot communicate using the domain name alone. It must discover
the server address, establish a network connection, pass through the permitted
network controls, reach the application, and wait for a response.

A simplified sequence is:

```text
1. The browser asks DNS for the server IP address.
2. The browser connects to that address across the network.
3. Network security rules decide whether the connection is allowed.
4. A reverse proxy receives the web request.
5. The reverse proxy sends it to the correct service.
6. The backend may read or change data in the database.
7. The response travels back to the browser.
```

This is a simplified mental model. Real production systems may also contain:

- Load balancers
- Content Delivery Networks (`CDNs`)
- Web Application Firewalls (`WAFs`)
- API gateways
- Multiple application instances
- Caches

Those additions do not change the main beginner idea: a request passes through
several layers, and each layer has a specific responsibility.

---

## 5. Important Terms

| Term | Simple meaning |
|---|---|
| Web request | A message sent by a client asking a web server for something |
| Client | The browser, mobile application, or program sending the request |
| Server | A system that receives a request and returns a response |
| DNS | System that translates a domain name into an IP address |
| IP address | Network address used to locate a system |
| Port | Number identifying a network service |
| Firewall | Control that allows or blocks network traffic |
| Reverse proxy | Server that receives external requests and forwards them internally |
| Frontend | User-facing application, such as a React application |
| Backend | Server-side application, such as a Spring Boot API |
| Database | System that stores application data |
| HTTP | Protocol used for web requests and responses |
| HTTPS | HTTP protected with TLS encryption |
| Internal service | Service intended for private application communication |

---

## 6. Stage 1 — The User and Browser

The request begins with a client.

Examples of clients:

- Web browser
- Mobile application
- React frontend
- Postman
- `curl`
- Another backend service

Suppose the user opens:

```text
https://student.example.com/api/courses
```

The URL contains several useful pieces:

```text
https://                  -> protocol
student.example.com       -> domain name
/api/courses              -> requested path
```

Because the URL uses HTTPS, the browser normally connects to TCP port `443`.

At this point, the browser knows the domain name but may not yet know its IP
address.

---

## 7. Stage 2 — DNS Resolution

DNS means:

```text
Domain Name System
```

DNS translates a human-readable domain into an IP address.

Example:

```text
student.example.com
          |
          | DNS lookup
          v
203.0.113.20
```

The IP address tells the client where to send the network connection.

### Simplified DNS process

```text
Browser asks:
"What is the IP address of student.example.com?"
                    |
                    v
DNS provides an address
                    |
                    v
Browser uses that address to connect
```

The browser or operating system may already have a cached DNS result. In that
case, it may not perform a complete new DNS lookup for every request.

### DNS security role

DNS must direct users to the correct system.

If an attacker changes a DNS record, users may be sent to the wrong server.

Important protections include:

- Strong domain-registrar account security
- Multi-factor authentication
- Restricted permission to modify DNS records
- Monitoring unexpected DNS changes
- DNSSEC where appropriate

DNS security will be explained in detail in **P8-L06 — DNS Security**.

---

## 8. Stage 3 — The Network and Internet

After obtaining the IP address, the client sends network traffic toward the
server.

```text
User's computer
      |
      | Local network
      v
Router / Internet provider
      |
      | Internet
      v
Server network
```

The internet is not one single machine. It is a large collection of connected
networks and routers.

Network devices forward the packets toward their destination.

### What is protected here?

Traffic crossing networks may be observed or modified if it is not protected.

HTTPS uses TLS to provide:

- Confidentiality: unauthorized observers should not read the data.
- Integrity: unauthorized changes should be detected.
- Server authentication: the client verifies the server identity using a
  certificate.

HTTPS and TLS will be explained in the next lessons.

---

## 9. Stage 4 — Firewall

Before traffic reaches the application, one or more firewall layers may
inspect it.

Examples:

- Router firewall
- Cloud firewall or security group
- Network firewall
- Linux host firewall
- Windows Firewall in a WSL environment

The firewall asks:

```text
Does this traffic match an allow rule?
```

Example policy:

| Traffic | Decision |
|---|---|
| Public HTTPS to port `443` | Allow |
| Public HTTP to port `80` | Allow only if needed, often redirect to HTTPS |
| SSH to port `22` from approved network | Allow |
| Public Spring Boot access to port `8080` | Deny |
| Public PostgreSQL access to port `5432` | Deny |

### Firewall security role

The firewall reduces unnecessary network exposure.

It should allow users to reach the public entrance while keeping internal
application and database services private.

```text
Internet
   |
   | 443 allowed
   v
Reverse proxy

Internet --X--> Spring Boot :8080
Internet --X--> PostgreSQL :5432
```

---

## 10. Stage 5 — Reverse Proxy

A reverse proxy receives requests on behalf of internal services.

Common reverse proxies include:

- Nginx
- Apache HTTP Server
- HAProxy
- Traefik

Example:

```text
Internet
   |
   | HTTPS :443
   v
Nginx reverse proxy
   |
   | Internal HTTP :8080
   v
Spring Boot
```

The user connects to Nginx, not directly to Spring Boot.

### What can a reverse proxy do?

A reverse proxy can:

- Receive external traffic.
- Terminate TLS.
- Forward requests to an internal service.
- Route different paths to different services.
- Hide internal ports.
- Add security headers.
- Apply request-size limits.
- Apply rate limiting.
- Distribute traffic across multiple application instances.

### Example routing

```text
/                -> React frontend
/api             -> Spring Boot backend
/health          -> Health endpoint
```

The reverse proxy selects the correct destination based on its configuration.

### Security role

The reverse proxy creates one controlled entrance to the application.

Instead of exposing every internal service:

```text
Public:
Nginx :443

Private:
React service
Spring Boot :8080
PostgreSQL :5432
```

---

## 11. Stage 6 — Frontend and Backend Services

### Frontend

The frontend is the user-facing part of the application.

For your stack, this may be a React application.

The browser may first request frontend files:

```text
index.html
JavaScript
CSS
Images
```

After React loads, it may send API requests to the backend.

Example:

```javascript
axios.get("/api/courses")
```

### Backend

The backend contains server-side application logic.

For your stack, this may be a Spring Boot API.

The backend may:

- Authenticate the user.
- Check authorization.
- Validate input.
- Perform business logic.
- Read or update database information.
- Return an HTTP response.

Example request:

```text
GET /api/courses
```

Example response:

```json
[
  {
    "id": 1,
    "name": "DevOps Fundamentals"
  }
]
```

### Security role

The backend must not trust every request simply because it passed through the
firewall and reverse proxy.

It still needs:

- Authentication
- Authorization
- Input validation
- Secure error handling
- Logging
- Least-privilege database access

This is another example of Defense in Depth.

---

## 12. Stage 7 — Database

The backend may need stored data to complete the request.

Example:

```text
Spring Boot receives:
GET /api/courses
        |
        v
Spring Boot queries PostgreSQL
        |
        v
PostgreSQL returns course records
        |
        v
Spring Boot creates an HTTP response
```

The database should normally not receive requests directly from public users.

The intended path is:

```text
User -> Reverse Proxy -> Backend -> Database
```

Not:

```text
User --------------------------> Database
```

### Database security role

The database connection should use:

- A dedicated database user.
- A strong secret.
- Least-privilege database permissions.
- Restricted network access.
- Encryption when required.
- Backups and monitoring.

### Important clarification

Not every web request reaches the database.

Examples that may not require a database:

- Loading a static image.
- Loading a CSS file.
- Returning a cached response.
- Requesting a simple health endpoint.

The database is contacted only when the application needs stored data.

---

## 13. The Response Path

After the request is processed, the response travels back through the system.

```text
Database result
      |
      v
Spring Boot response
      |
      v
Nginx reverse proxy
      |
      v
Internet
      |
      v
User's browser
```

For an HTTPS connection, data sent between the browser and the HTTPS endpoint
is protected by TLS.

The browser receives:

- An HTTP status code
- Response headers
- Response body

Example:

```text
HTTP/2 200 OK
Content-Type: application/json
```

The React application can then display the returned information to the user.

---

## 14. Complete Real-World Example

Consider your familiar stack:

```text
User
  |
  | Opens https://student.example.com/courses
  v
DNS
  |
  | Returns server IP address
  v
Internet
  |
  | HTTPS traffic
  v
Firewall
  |
  | Allows TCP port 443
  v
Nginx Reverse Proxy
  |
  | Serves React or forwards /api requests
  v
Spring Boot API
  |
  | Authenticates, validates, and processes
  v
PostgreSQL
  |
  | Returns required data
  v
Spring Boot -> Nginx -> Browser
```

### Protection map

| Component | Main job | Example security protection |
|---|---|---|
| User/browser | Sends request and displays response | Browser certificate validation |
| DNS | Converts domain to IP address | Protected registrar account and DNS changes |
| Internet/network | Carries packets | HTTPS/TLS encryption |
| Firewall | Filters network traffic | Allow `443`, restrict `22`, block public `8080` and `5432` |
| Reverse proxy | Controls external entrance and routing | TLS termination, headers, limits, rate limiting |
| React frontend | User interface | Safe handling of data and tokens |
| Spring Boot backend | Business logic and API | Authentication, authorization, validation |
| PostgreSQL | Stores data | Private network, limited database user, backups |

No single control protects the entire request path.

---

## 15. Commands and Configuration

### Command 1 — Resolve a domain name

```bash
nslookup example.com
```

Purpose:

> Ask DNS for information about `example.com`.

The output may show one or more IP addresses.

If `nslookup` is not installed, you may use:

```bash
getent hosts example.com
```

---

### Command 2 — Inspect an HTTPS response

```bash
curl -I https://example.com
```

Purpose:

> Send an HTTPS request and display only the response headers.

Important option:

- `-I`: Requests response headers without downloading the normal response
  body.

Possible result:

```text
HTTP/2 200
content-type: text/html
```

The exact HTTP version and headers may differ.

---

### Command 3 — Show connection details

```bash
curl -v https://example.com
```

Purpose:

> Display detailed information about the connection and request.

The verbose output may show:

- DNS-resolved address
- Connection to port `443`
- TLS information
- HTTP request headers
- HTTP response headers

Verbose output can be long. You do not need to understand every line in this
lesson.

Important warning:

> Do not use `curl -v` with real secrets or authorization tokens when saving
> public screenshots because verbose output may display request headers.

---

## 16. Safe Practical Lab

### Lab goal

Observe DNS resolution and an HTTPS request without changing any system
configuration.

### Step 1 — Resolve the domain

Run:

```bash
nslookup example.com
```

If `nslookup` is unavailable, run:

```bash
getent hosts example.com
```

Record one returned IP address.

### Step 2 — Request HTTPS headers

Run:

```bash
curl -I https://example.com
```

Record:

- HTTP status
- Content type
- Any server or caching headers shown

### Step 3 — Observe connection details

Run:

```bash
curl -v https://example.com
```

Look for evidence of:

- A connection to port `443`
- TLS negotiation
- An HTTP request
- An HTTP response

The verbose lines commonly use:

```text
*  Connection or TLS information
>  Data sent by the client
<  Data received from the server
```

### Step 4 — Map the observed path

Write:

```text
My computer used DNS to resolve example.com.
It connected to the resolved server using HTTPS on port 443.
The remote web service returned an HTTP response.
Internal reverse-proxy, backend, and database details are not necessarily
visible to an external client.
```

### Important lab limitation

The commands prove:

- DNS resolution occurred.
- An HTTPS connection was made.
- An HTTP response was returned.

They do not prove the remote site's complete internal architecture.

You cannot assume that `example.com` uses a particular reverse proxy,
application framework, or database unless reliable documentation confirms it.

---

## 17. Expected Result

The DNS command may return output similar to:

```text
Name:    example.com
Address: 93.184.216.34
```

The actual address may differ.

The HTTPS request may return:

```text
HTTP/2 200
content-type: text/html
```

The verbose request may include lines similar to:

```text
* Connected to example.com (...) port 443
* SSL connection using ...
> GET / HTTP/...
< HTTP/... 200
```

The exact result depends on:

- DNS response
- Network location
- `curl` version
- Server configuration
- HTTP version

---

## 18. Evidence to Save

Save one screenshot showing:

```bash
nslookup example.com
curl -I https://example.com
```

If `nslookup` is unavailable, show:

```bash
getent hosts example.com
curl -I https://example.com
```

Optionally save a second screenshot containing selected non-sensitive output
from:

```bash
curl -v https://example.com
```

### Suggested evidence title

```text
DNS Resolution and HTTPS Web Request
```

### Suggested evidence explanation

> I used DNS to resolve the domain name into an IP address. I then used
> `curl -I` to send an HTTPS request and inspect the response headers. This
> demonstrates the external part of a web request: DNS resolution, network
> connection to port 443, and an HTTP response protected by TLS.

### Architecture evidence

Create or include this diagram in the report:

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
Nginx Reverse Proxy :443
  |
  v
Spring Boot :8080
  |
  v
PostgreSQL :5432
```

Label:

- Port `443` as public.
- Port `8080` as internal.
- Port `5432` as private.

---

## 19. Common Mistakes

### Mistake 1 — Thinking DNS sends the complete web request

DNS normally helps the client find an IP address.

The web request is then sent using HTTP or HTTPS.

---

### Mistake 2 — Thinking the firewall processes application business logic

The firewall filters network traffic.

Spring Boot performs the application business logic.

---

### Mistake 3 — Exposing every internal service

Users normally need access to the controlled public entrance, not direct
access to Spring Boot and PostgreSQL.

---

### Mistake 4 — Assuming HTTPS protects the database automatically

Browser-to-server HTTPS protects that network connection.

The database still requires its own:

- Network restrictions
- Authentication
- Permissions
- Encryption configuration where required

---

### Mistake 5 — Assuming every request reaches the database

Static files, cached results, and some health endpoints may not require a
database query.

---

### Mistake 6 — Confusing a reverse proxy with a firewall

A firewall primarily filters network traffic.

A reverse proxy receives application requests and forwards them to internal
services.

The two controls may work together but do different jobs.

---

### Mistake 7 — Claiming an internal architecture from `curl` output

An external response does not necessarily reveal every internal component.

Document the architecture you control or information confirmed by reliable
sources.

---

## 20. Task Report Notes

You may adapt the following text for your final assignment:

> A web request passes through several network and application layers. The
> client first uses DNS to resolve a domain name into an IP address. It then
> establishes a network connection to the destination. For HTTPS, TLS protects
> the communication by providing encryption, integrity, and server
> authentication.

> Firewalls allow required traffic and block unnecessary exposure. A public
> web application may allow HTTPS traffic on port 443 while keeping the
> Spring Boot application port and PostgreSQL database port private. A reverse
> proxy such as Nginx receives external traffic and forwards each request to
> the correct internal frontend or backend service.

> The backend performs authentication, authorization, validation, and business
> logic. It communicates with the database only when stored data is required.
> The database should accept connections only from approved internal services
> and use a limited database account.

### Practical demonstration note

> I used `nslookup` to resolve a domain name and `curl -I` to send an HTTPS
> request. The commands demonstrated the external request path from DNS
> resolution to an HTTP response over port 443. The external client could not
> prove the remote system's complete internal reverse-proxy, backend, or
> database architecture.

---

## 21. Five Review Questions

1. What is the role of DNS in a web request?
2. What is the difference between the role of a firewall and a reverse proxy?
3. Why should Spring Boot port `8080` and PostgreSQL port `5432` normally
   remain private?
4. Does every web request need to reach the database? Explain.
5. Which security control protects data travelling between a browser and an
   HTTPS endpoint?

---

## 22. Lesson Summary

The simplified web-request path is:

```text
User
  |
  | enters domain
  v
DNS
  |
  | returns IP address
  v
Internet
  |
  | carries HTTPS traffic
  v
Firewall
  |
  | allows required port
  v
Reverse Proxy
  |
  | routes the request
  v
Frontend or Backend
  |
  | requests stored data when needed
  v
Database
```

Each part has a different job:

- DNS locates the destination.
- The network carries the traffic.
- HTTPS/TLS protects data in transit.
- The firewall filters traffic.
- The reverse proxy controls the public application entrance.
- The backend performs security checks and business logic.
- The database stores information and remains private.

No single layer protects the complete system. Secure applications use several
connected controls.

---

## 23. Progress

### Stage 1 — Part 7: Linux Security

- [x] P7-L01 through P7-L10 completed

### Stage 2 — Part 8: Network Security

- [x] P8-L01 — The Path of a Web Request
- [ ] P8-L02 — HTTP and HTTPS
- [ ] P8-L03 — TLS Explained Simply
- [ ] P8-L04 — Certificates and Trust
- [ ] P8-L05 — Inspecting HTTPS, TLS, and Certificates
- [ ] P8-L06 — DNS Security
- [ ] P8-L07 — Firewalls and Reverse Proxy Security
- [ ] P8-L08 — Common Network Attacks at a High Level
- [ ] P8-L09 — Part 8 Network Security Report

---

## Exact Next Lesson

**P8-L02 — HTTP and HTTPS**

Do not continue until the student writes:

```text
next
```
