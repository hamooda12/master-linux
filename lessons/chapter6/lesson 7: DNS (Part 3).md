# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 07 --- DNS (Part 3)

------------------------------------------------------------------------

# Linux DNS Configuration

Linux stores DNS configuration in several files.

The most important one is:

``` text
/etc/resolv.conf
```

This file tells Linux which DNS servers should be used when resolving
domain names.

------------------------------------------------------------------------

# Viewing DNS Configuration

``` bash
cat /etc/resolv.conf
```

Example:

``` text
nameserver 8.8.8.8
nameserver 1.1.1.1
```

Each `nameserver` entry represents a DNS server Linux may query.

------------------------------------------------------------------------

# The `hostname` Command

Display the system hostname:

``` bash
hostname
```

Example output:

``` text
web-server-01
```

The hostname identifies the computer on the network.

------------------------------------------------------------------------

# The `host` Command

Query DNS information for a domain.

``` bash
host example.com
```

Example:

``` text
example.com has address 93.184.216.34
```

Useful for quick DNS verification.

------------------------------------------------------------------------

# The `dig` Command

`dig` (Domain Information Groper) provides detailed DNS information.

``` bash
dig example.com
```

Useful information includes:

-   Answer section
-   DNS server used
-   Query time
-   Returned records

It is one of the most important DNS troubleshooting tools.

------------------------------------------------------------------------

# The `nslookup` Command

Another utility for querying DNS.

``` bash
nslookup example.com
```

Example:

``` text
Server: 8.8.8.8

Name: example.com

Address: 93.184.216.34
```

Although `dig` is generally preferred, `nslookup` is still widely
available.

------------------------------------------------------------------------

# Comparing DNS Tools

  Command                  Purpose
  ------------------------ -----------------------------
  `cat /etc/resolv.conf`   View configured DNS servers
  `hostname`               Display system hostname
  `host`                   Simple DNS query
  `dig`                    Detailed DNS query
  `nslookup`               DNS lookup utility

------------------------------------------------------------------------

# Linux Perspective

When an application requests:

``` text
https://example.com
```

Linux:

1.  Reads DNS configuration.
2.  Contacts a configured DNS server.
3.  Receives an IP address.
4.  Connects to that IP.

Applications rarely perform DNS lookups directly---they rely on the
operating system.

------------------------------------------------------------------------

# Production Example

Users report that:

``` text
https://api.example.com
```

cannot be reached.

Running:

``` bash
dig api.example.com
```

returns:

``` text
NXDOMAIN
```

This indicates the domain name cannot be resolved because the required
DNS record does not exist.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   `/etc/resolv.conf`
-   `hostname`
-   `host`
-   `dig`
-   `nslookup`
-   How Linux performs DNS lookups

------------------------------------------------------------------------

# ➡️ Next Part

We'll troubleshoot common DNS failures, analyze command output, and
complete the DNS lesson with labs, interview questions, and a cheat
sheet.
