# Chapter 06 --- Networking Fundamentals

# Lesson 10 --- HTTP & HTTPS for DevOps Engineers (DevOps Deep Dive)

> **Objective:** Master how HTTP and HTTPS really work from the moment a
> user types a URL until a response is returned, and learn how to
> troubleshoot production web applications.

------------------------------------------------------------------------

# Why This Lesson Matters

Almost every DevOps engineer manages web applications.

Whether you deploy:

-   Nginx
-   Apache
-   Spring Boot
-   Node.js
-   Laravel
-   Kubernetes Ingress
-   AWS Load Balancer

they all communicate using **HTTP or HTTPS**.

If you understand HTTP deeply, troubleshooting production systems
becomes much easier.

------------------------------------------------------------------------

# The Complete Journey

When a user visits:

``` text
https://shop.example.com/products
```

the following happens:

``` text
User
 │
 │ 1. Enter URL
 ▼
Browser
 │
 │ 2. DNS Lookup
 ▼
DNS Server
 │
 │ 3. Return IP Address
 ▼
Browser
 │
 │ 4. TCP Three-Way Handshake
 ▼
Web Server
 │
 │ 5. TLS Handshake
 ▼
Secure Connection
 │
 │ 6. HTTP Request
 ▼
Nginx
 │
 │ 7. Reverse Proxy
 ▼
Spring Boot
 │
 │ 8. SQL Query
 ▼
PostgreSQL
 │
 │ 9. Response
 ▼
Browser
```

Every production request follows roughly this sequence.

------------------------------------------------------------------------

# HTTP Is an Application Layer Protocol

HTTP defines **how applications exchange data**.

It does **not** define:

-   routing
-   encryption
-   reliability

Instead it relies on lower layers.

``` text
HTTP
↓

TLS (HTTPS only)

↓

TCP

↓

IP

↓

Ethernet / Wi-Fi
```

------------------------------------------------------------------------

# HTTP Request Anatomy

``` http
GET /products?page=2 HTTP/1.1
Host: shop.example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer TOKEN

<optional body>
```

Parts:

-   Method
-   Path
-   HTTP Version
-   Headers
-   Optional Body

------------------------------------------------------------------------

# HTTP Response Anatomy

``` http
HTTP/1.1 200 OK

Content-Type: application/json
Content-Length: 231
Server: nginx

{
  "products": [...]
}
```

------------------------------------------------------------------------

# Common HTTP Methods

  Method   Meaning          Idempotent?   Typical Use
  -------- ---------------- ------------- -----------------
  GET      Read             Yes           Fetch data
  POST     Create           No            Create resource
  PUT      Replace          Yes           Replace object
  PATCH    Partial update   Usually       Update fields
  DELETE   Remove           Yes           Delete resource

------------------------------------------------------------------------

# Understanding Status Codes Like a DevOps Engineer

## 2xx

Request succeeded.

Examples:

-   200 OK
-   201 Created
-   204 No Content

------------------------------------------------------------------------

## 3xx

Redirection.

301 → permanent redirect

302 → temporary redirect

304 → browser cache still valid

------------------------------------------------------------------------

## 4xx

The client did something wrong.

400 Bad Request

401 Unauthorized

Authentication missing.

403 Forbidden

Authenticated but not allowed.

404 Not Found

Resource doesn't exist.

------------------------------------------------------------------------

## 5xx

Server-side failures.

500

Application crashed.

502

Gateway received invalid response from backend.

Usually:

Browser → Nginx → Spring Boot

Spring Boot crashed.

503

Service unavailable.

Often:

-   maintenance
-   overloaded
-   no healthy pods

504

Gateway timeout.

Backend took too long.

------------------------------------------------------------------------

# HTTP Headers Every DevOps Engineer Must Know

## Host

Determines which virtual host should serve the request.

## Content-Type

Tells how to interpret the body.

Examples:

-   application/json
-   text/html
-   multipart/form-data

## Authorization

Carries authentication credentials.

## User-Agent

Identifies the client.

## Accept

Specifies acceptable response formats.

## Location

Used in redirects.

## Set-Cookie

Creates browser cookies.

## X-Forwarded-For

Original client IP behind reverse proxies.

## X-Forwarded-Proto

Indicates whether the original request was HTTP or HTTPS.

------------------------------------------------------------------------

# What HTTPS Adds

HTTPS = HTTP + TLS

TLS provides:

-   Encryption
-   Authentication
-   Integrity

TLS handshake:

1.  TCP connection
2.  ClientHello
3.  ServerHello
4.  Certificate exchange
5.  Certificate validation
6.  Session keys created
7.  Encrypted HTTP begins

------------------------------------------------------------------------

# TLS Termination

``` text
Browser
   │ HTTPS
   ▼
Load Balancer
   │ HTTPS
   ▼
Nginx
   │ HTTP
   ▼
Spring Boot
```

The load balancer or reverse proxy often decrypts HTTPS and forwards
plain HTTP internally.

------------------------------------------------------------------------

# Real Production Troubleshooting

## Incident 1 --- 502 Bad Gateway

Symptoms:

-   Nginx returns 502.

Possible causes:

-   Backend process stopped.
-   Wrong upstream port.
-   Application crashed.

Commands:

``` bash
systemctl status backend
journalctl -u backend -n 50
ss -tulpen
```

------------------------------------------------------------------------

## Incident 2 --- HTTPS Certificate Expired

Symptoms:

Browser displays security warning.

Check:

``` bash
openssl s_client -connect example.com:443
```

------------------------------------------------------------------------

## Incident 3 --- Infinite Redirect Loop

Cause:

Nginx thinks requests are HTTP while the load balancer already
terminated TLS.

Check:

-   X-Forwarded-Proto
-   reverse proxy configuration

------------------------------------------------------------------------

# Linux Commands

``` bash
curl -I https://example.com
curl -v https://example.com
curl -L http://example.com
openssl s_client -connect example.com:443
ss -tulpen
```

------------------------------------------------------------------------

# DevOps Checklist

When a website is down, verify in order:

1.  DNS
2.  TCP connectivity
3.  Port 80/443
4.  Firewall
5.  TLS certificate
6.  Reverse proxy
7.  Backend application
8.  Database
9.  Logs

------------------------------------------------------------------------

# Key Takeaways

-   HTTP is an application protocol.
-   HTTPS is HTTP protected by TLS.
-   Every request follows DNS → TCP → TLS → HTTP.
-   Headers and status codes are critical troubleshooting tools.
-   Most production outages involve reverse proxies, certificates,
    networking, or backend availability---not HTTP itself.
