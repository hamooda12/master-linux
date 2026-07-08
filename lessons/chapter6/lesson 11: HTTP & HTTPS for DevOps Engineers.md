# Chapter 06 — Networking Fundamentals

# Lesson 11 — HTTP & HTTPS for DevOps Engineers

> **Objective:** Understand how web applications communicate using HTTP and HTTPS, how browsers and servers exchange requests and responses, and why HTTPS is essential in production.

---

# Learning Objectives

By the end of this lesson you will be able to:

- Explain what HTTP and HTTPS are.
- Understand the request/response model.
- Identify common HTTP methods.
- Interpret HTTP status codes.
- Read HTTP headers.
- Explain the role of TLS in HTTPS.
- Use Linux tools to test HTTP services.

---

# What Is HTTP?

HTTP (**HyperText Transfer Protocol**) is the protocol used by clients and servers to exchange web requests and responses.

Example:

```text
Browser
   │ HTTP Request
   ▼
Web Server
   │ HTTP Response
   ▼
Browser
```

HTTP is **application-layer** protocol and usually runs over **TCP**.

Default port:

```text
80
```

---

# What Is HTTPS?

HTTPS is simply:

```text
HTTP + TLS Encryption
```

Default port:

```text
443
```

HTTPS provides:

- Encryption
- Authentication
- Integrity

Without HTTPS, anyone on the network could potentially read unencrypted traffic.

---

# Request / Response Model

Every HTTP conversation follows the same pattern:

```text
Client
   │
Request
   ▼
Server
   │
Response
   ▼
Client
```

---

# HTTP Methods

| Method | Purpose |
|---------|---------|
| GET | Retrieve data |
| POST | Create new data |
| PUT | Replace existing data |
| PATCH | Update part of existing data |
| DELETE | Remove data |

Examples:

```http
GET /products
```

```http
POST /users
```

---

# HTTP Status Codes

## 2xx – Success

```text
200 OK
201 Created
204 No Content
```

## 3xx – Redirection

```text
301 Moved Permanently
302 Found
304 Not Modified
```

## 4xx – Client Errors

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

## 5xx – Server Errors

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# HTTP Headers

Headers carry metadata.

Example request:

```http
GET / HTTP/1.1
Host: example.com
User-Agent: curl/8.0
Accept: */*
```

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 120
Server: nginx
```

---

# HTTPS and TLS

Before HTTP data is exchanged over HTTPS:

1. TCP connection is established.
2. TLS handshake occurs.
3. Certificates are verified.
4. Encryption keys are negotiated.
5. Encrypted HTTP traffic begins.

---

# Typical Production Flow

```text
Browser
   │ HTTPS
   ▼
Load Balancer
   │ HTTPS or HTTP
   ▼
Nginx
   │ HTTP
   ▼
Spring Boot
   │
   ▼
PostgreSQL
```

Often TLS terminates at the load balancer or reverse proxy.

---

# Linux Tools

Check HTTP headers:

```bash
curl -I https://example.com
```

Verbose request:

```bash
curl -v https://example.com
```

Download page:

```bash
curl https://example.com
```

---

# DevOps Troubleshooting

Problem:

```text
Users cannot access https://example.com
```

Possible causes:

- DNS failure
- Port 443 closed
- Firewall blocking HTTPS
- TLS certificate expired
- Nginx not listening
- Backend unavailable
- Load balancer unhealthy

---

# Hands-On Lab

Run:

```bash
curl -I https://example.com
curl -v https://example.com
curl http://example.com
```

Observe:

- Status code
- Headers
- Redirects
- TLS negotiation (verbose mode)

---

# Common Mistakes

- Confusing HTTP with HTTPS.
- Assuming HTTPS means the application is secure.
- Ignoring HTTP status codes.
- Using HTTP in production for sensitive data.

---

# Cheat Sheet

| Item | Value |
|------|-------|
| HTTP Port | 80 |
| HTTPS Port | 443 |
| GET | Read |
| POST | Create |
| PUT | Replace |
| PATCH | Update |
| DELETE | Remove |
| 200 | Success |
| 404 | Not Found |
| 500 | Server Error |

---

# Interview Questions

1. What is the difference between HTTP and HTTPS?
2. Why does HTTPS require TLS?
3. What is the default port for HTTPS?
4. What does status code 404 mean?
5. What command can test an HTTP endpoint from Linux?

---

# Summary

HTTP defines how web clients and servers communicate.

HTTPS adds TLS encryption, authentication and integrity, making it the standard protocol for secure web communication in modern production environments.

---

# Next Lesson

**Lesson 12 — Firewalls: Allowing and Denying Network Traffic**
