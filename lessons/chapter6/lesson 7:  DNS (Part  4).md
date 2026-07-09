# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 07 --- DNS (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- DNS Server Unreachable

Users cannot access websites using domain names.

Running:

``` bash
dig example.com
```

returns:

``` text
;; connection timed out; no servers could be reached
```

### Investigation

-   Verify network connectivity.
-   Check `/etc/resolv.conf`.
-   Ensure the configured DNS server is reachable.

------------------------------------------------------------------------

## Scenario 2 --- Incorrect DNS Server

``` text
nameserver 192.168.1.250
```

The configured server does not exist.

Result:

-   DNS lookups fail.
-   Applications cannot resolve hostnames.

------------------------------------------------------------------------

## Scenario 3 --- Missing DNS Record

``` bash
host api.example.com
```

Output:

``` text
Host api.example.com not found: 3(NXDOMAIN)
```

### Investigation

The domain exists, but the requested hostname has no DNS record.

------------------------------------------------------------------------

## Scenario 4 --- Application Works by IP but Not by Name

The following succeeds:

``` text
http://192.168.1.20
```

But:

``` text
http://app.example.com
```

fails.

### Root Cause

Networking is functioning.

The failure is DNS resolution.

------------------------------------------------------------------------

# DNS Troubleshooting Workflow

``` text
Check network connectivity
        │
        ▼
Verify /etc/resolv.conf
        │
        ▼
Query with dig or host
        │
        ▼
Verify returned records
        │
        ▼
Test application
```

Always prove that DNS is the problem before changing application
settings.

------------------------------------------------------------------------

# Common Mistakes

## Assuming Every Connection Failure Is DNS

Applications may fail because of routing, firewalls, or incorrect ports.

Always verify DNS before drawing conclusions.

------------------------------------------------------------------------

## Editing the Wrong DNS Server

Changing DNS configuration without confirming the correct server often
creates additional problems.

------------------------------------------------------------------------

## Ignoring Cached Results

Some applications cache DNS responses.

A DNS record may be corrected while an application still uses an older
cached address.

------------------------------------------------------------------------

# Best Practices

-   Use `dig` for detailed investigations.
-   Keep DNS server configuration documented.
-   Verify records after making DNS changes.
-   Test both hostname resolution and application connectivity.
-   Troubleshoot systematically instead of guessing.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is DNS?
2.  Why do computers need DNS?
3.  What is an A record?

## Intermediate

1.  Compare A and AAAA records.
2.  What is a CNAME record?
3.  What information is stored in `/etc/resolv.conf`?

## Advanced

1.  A website works by IP but not by hostname. What would you
    investigate?
2.  Explain the difference between a DNS failure and a routing failure.

------------------------------------------------------------------------

# Hands-on Lab

Run the following commands:

``` bash
hostname
```

``` bash
cat /etc/resolv.conf
```

``` bash
host example.com
```

``` bash
dig example.com
```

Answer:

1.  What hostname does your system use?
2.  Which DNS servers are configured?
3.  What IPv4 address is returned?
4.  How long did the DNS query take?

------------------------------------------------------------------------

# Challenge Exercise

A Linux server can successfully:

``` text
ping 8.8.8.8
```

but cannot reach:

``` text
google.com
```

Explain why this indicates a DNS problem rather than a general network
connectivity problem.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 07 you understand:

-   DNS fundamentals
-   DNS resolution
-   DNS records
-   Linux DNS configuration
-   `host`
-   `dig`
-   `nslookup`
-   `/etc/resolv.conf`
-   DNS troubleshooting methodology

These skills are essential for diagnosing connectivity problems in Linux
and cloud environments.

------------------------------------------------------------------------

# Cheat Sheet

  Command                  Purpose
  ------------------------ -----------------------------
  `hostname`               Display system hostname
  `cat /etc/resolv.conf`   View configured DNS servers
  `host domain`            Simple DNS lookup
  `dig domain`             Detailed DNS query
  `nslookup domain`        DNS lookup utility

------------------------------------------------------------------------

# What's Next

Lesson 08 --- Network Connectivity

You'll learn how to test connectivity using `ping`, `traceroute`,
`tracepath`, and `mtr`, understand ICMP, packet loss, latency, and
diagnose network path failures like a Linux administrator.
