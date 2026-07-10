# 🐧 Linux Service Testing Command Reference (DevOps Edition)

> Companion guide for **Chapter 06 -- Testing Network Services**
>
> This handbook explains how Linux administrators verify that
> **applications** work from the client's perspective.

------------------------------------------------------------------------

# Service Testing Workflow

    1. ping
    2. dig / nslookup
    3. ss -tulpn (server)
    4. nc
    5. curl
    6. wget
    7. Check application logs

------------------------------------------------------------------------

# curl

## Purpose

Test HTTP/HTTPS services and REST APIs.

## Basic

``` bash
curl http://example.com
curl https://example.com
```

## Show headers

``` bash
curl -I https://example.com
```

## Verbose

``` bash
curl -v https://example.com
```

## Follow redirects

``` bash
curl -L https://example.com
```

## Download

``` bash
curl -O https://example.com/file.zip
```

## Save with custom name

``` bash
curl -o app.zip https://example.com/file.zip
```

## POST JSON

``` bash
curl -X POST http://localhost:8080/api/users \
-H "Content-Type: application/json" \
-d '{"name":"Alice"}'
```

## PUT

``` bash
curl -X PUT http://localhost:8080/api/users/1 \
-H "Content-Type: application/json" \
-d '{"name":"Bob"}'
```

## DELETE

``` bash
curl -X DELETE http://localhost:8080/api/users/1
```

## Authentication

``` bash
curl -u user:password http://server
curl -H "Authorization: Bearer TOKEN" http://server/api
```

## Useful options

  Option   Purpose
  -------- -------------------
  -I       Headers only
  -v       Verbose
  -L       Follow redirects
  -o       Save file
  -O       Original filename
  -X       HTTP method
  -H       HTTP header
  -d       Request body

------------------------------------------------------------------------

# wget

## Purpose

Download files and websites.

``` bash
wget https://example.com/file.iso
wget -O ubuntu.iso https://example.com/file.iso
wget -c https://example.com/file.iso
wget -b https://example.com/file.iso
wget --mirror https://example.com
```

Common options:

  Option     Purpose
  ---------- -----------------------
  -O         Save with custom name
  -c         Resume download
  -b         Background download
  --mirror   Mirror website

------------------------------------------------------------------------

# nc (Netcat)

## Purpose

Test TCP/UDP connectivity.

``` bash
nc -zv google.com 80
nc -zv localhost 8080
nc -zvu dns.google 53
```

Listen mode:

``` bash
nc -l 9000
```

Connect manually:

``` bash
nc localhost 8080
```

Useful options:

  Option   Purpose
  -------- ---------------------------
  -z       Scan without sending data
  -v       Verbose
  -u       UDP
  -l       Listen

------------------------------------------------------------------------

# telnet

## Purpose

Manual TCP testing (legacy but still useful).

``` bash
telnet google.com 80
telnet localhost 25
```

Manual HTTP example:

``` text
GET / HTTP/1.1
Host: example.com
```

------------------------------------------------------------------------

# Production Troubleshooting

## Ping works, website doesn't

``` bash
ping api.company.com
curl http://api.company.com:8080/health
```

If curl returns **500**, networking is healthy; the application is
failing.

------------------------------------------------------------------------

## Port check

``` bash
nc -zv api.company.com 8080
```

-   Connected → service listening
-   Connection refused → service stopped
-   Timed out → firewall/routing

------------------------------------------------------------------------

## Download verification

``` bash
wget https://repo/file.tar.gz
```

Confirms DNS, routing, TLS, and HTTP download.

------------------------------------------------------------------------

# DevOps Cheat Sheet

  Command   Purpose
  --------- ------------------------
  curl      Test APIs and websites
  wget      Download resources
  nc        Test TCP/UDP ports
  telnet    Manual TCP session

------------------------------------------------------------------------

# Interview Notes

-   `curl` is the primary HTTP troubleshooting tool.
-   `wget` focuses on downloading.
-   `nc` is ideal for checking whether a port is reachable.
-   `telnet` is useful for manually speaking simple text protocols.
-   Always distinguish **network connectivity** from **application
    availability**.
