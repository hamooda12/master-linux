# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 07 --- DNS (Part 2)

------------------------------------------------------------------------

# DNS Records

A DNS server stores different types of records.

Each record has a specific purpose.

Understanding these records helps Linux administrators troubleshoot
websites, APIs, email systems, and cloud services.

------------------------------------------------------------------------

# A Record

## Definition

An **A (Address) Record** maps a domain name to an **IPv4 address**.

Example:

``` text
example.com

↓

192.168.1.20
```

When a browser requests `example.com`, the DNS server returns its IPv4
address.

------------------------------------------------------------------------

# AAAA Record

## Definition

An **AAAA Record** maps a domain name to an **IPv6 address**.

Example:

``` text
example.com

↓

2001:db8::20
```

It serves the same purpose as an A record but for IPv6.

------------------------------------------------------------------------

# CNAME Record

## Definition

A **Canonical Name (CNAME)** record creates an alias for another domain
name.

Example:

``` text
www.example.com

↓

example.com
```

Instead of storing another IP address, the alias points to the original
hostname.

This simplifies DNS management.

------------------------------------------------------------------------

# MX Record

## Definition

An **MX (Mail Exchange)** record identifies the mail server responsible
for receiving email for a domain.

Example:

``` text
example.com

↓

mail.example.com
```

Email systems consult MX records before delivering messages.

------------------------------------------------------------------------

# TXT Record

## Definition

A **TXT Record** stores arbitrary text information associated with a
domain.

Common uses include:

-   Domain verification
-   Email security
-   SPF
-   DKIM
-   DMARC

Example:

``` text
example.com

↓

"v=spf1 include:_spf.example.com ~all"
```

Modern cloud services frequently ask administrators to add TXT records
during setup.

------------------------------------------------------------------------

# Summary of Common DNS Records

  Record   Purpose
  -------- ---------------------------
  A        Domain → IPv4
  AAAA     Domain → IPv6
  CNAME    Alias to another hostname
  MX       Mail server
  TXT      Verification and metadata

------------------------------------------------------------------------

# Linux Perspective

Linux tools can query these records individually.

Later lessons will introduce:

``` bash
dig

host

nslookup
```

These commands help administrators verify DNS configuration.

------------------------------------------------------------------------

# Production Example

A company launches a new website.

Users can access:

``` text
example.com
```

but:

``` text
www.example.com
```

fails.

Investigation reveals the missing **CNAME** record.

Adding the correct alias immediately resolves the problem.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   A Records
-   AAAA Records
-   CNAME Records
-   MX Records
-   TXT Records
-   Typical production use cases

------------------------------------------------------------------------

# ➡️ Next Part

We'll inspect Linux DNS configuration, explore `/etc/resolv.conf`, and
begin using DNS troubleshooting commands.
