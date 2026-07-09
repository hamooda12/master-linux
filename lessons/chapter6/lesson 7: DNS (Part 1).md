# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 07 --- DNS (Part 1)

> **Goal:** Understand what DNS is, why it exists, and how Linux
> converts human-readable domain names into IP addresses.

------------------------------------------------------------------------

# 📚 Overview

Humans remember names more easily than numbers.

Imagine trying to remember the IP address of every website you visit:

``` text
142.250.190.78
151.101.1.69
104.18.32.47
```

Instead, we use names:

-   google.com
-   github.com
-   openai.com

The system responsible for translating these names into IP addresses is
called the **Domain Name System (DNS)**.

Without DNS, using the Internet would be much more difficult.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what DNS is.
-   Understand why DNS exists.
-   Describe the DNS resolution process.
-   Distinguish between domain names and IP addresses.
-   Understand the role of DNS servers.

------------------------------------------------------------------------

# 🌍 What Is DNS?

DNS stands for **Domain Name System**.

It is a distributed naming system that translates domain names into IP
addresses.

Example:

``` text
google.com

↓

142.250.x.x
```

Applications communicate using IP addresses, while users work with
domain names.

------------------------------------------------------------------------

# Why Do We Need DNS?

Routers and servers cannot route packets using names.

They require IP addresses.

DNS acts like a phone book.

    Name

    ↓

    DNS

    ↓

    IP Address

Once the IP address is known, communication begins.

------------------------------------------------------------------------

# DNS Resolution

Suppose you visit:

``` text
https://github.com
```

The sequence is:

``` text
Browser
    │
    ▼
DNS Query
    │
    ▼
DNS Server
    │
    ▼
IP Address
    │
    ▼
TCP Connection
    │
    ▼
HTTP/HTTPS Request
```

DNS happens before the web request.

------------------------------------------------------------------------

# Domain Names

A domain name is a human-readable identifier.

Examples:

``` text
example.com
api.example.com
mail.example.com
```

These names are easier to remember than numerical addresses.

------------------------------------------------------------------------

# DNS Server

A DNS server stores or retrieves DNS information.

Its job is to answer questions such as:

    What is the IP address of example.com?

The reply contains one or more IP addresses.

------------------------------------------------------------------------

# Linux Perspective

Linux applications perform DNS lookups automatically.

Later you'll inspect DNS using:

``` bash
cat /etc/resolv.conf
host
dig
nslookup
```

------------------------------------------------------------------------

# Production Example

Users report:

> "The website cannot be reached."

Investigation shows the server is healthy.

The application is running.

The network is working.

The actual problem:

DNS resolution is failing.

Without DNS, clients cannot discover the server's IP address.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What DNS is.
-   Why DNS exists.
-   Domain names.
-   DNS servers.
-   Basic DNS resolution.

------------------------------------------------------------------------

# ➡️ Next Part

We'll explore DNS records, including A, AAAA, CNAME, MX, and TXT
records, and understand what each record is used for.
