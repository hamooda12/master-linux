# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 10 --- Testing Network Services (Part 1)

> **Goal:** Learn how Linux administrators verify that network services
> are actually working by testing them from a client's perspective.

------------------------------------------------------------------------

# 📚 Overview

A server may be:

-   Powered on
-   Connected to the network
-   Reachable by ping
-   Listening on the correct port

...yet the application may still not work.

Instead of only checking the server itself, administrators test the
service exactly as a client would.

Linux provides several tools for this purpose:

-   curl
-   wget
-   nc (Netcat)
-   telnet

These tools help answer one simple question:

> **"Can I successfully communicate with the service?"**

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain why service testing is important.
-   Understand the difference between network connectivity and service
    availability.
-   Know when to use curl, wget, nc, and telnet.
-   Test HTTP services and REST APIs from Linux.

------------------------------------------------------------------------

# 🌍 Connectivity vs Service Availability

A server responding to `ping` does **not** mean the application is
working.

Example:

    Laptop
        |
     ping ✓
        |
    Server
        |
    Spring Boot ✗

The network is healthy, but the application has failed.

Always test the service itself.

------------------------------------------------------------------------

# Why Test Services?

Suppose users report:

    https://api.example.com

is unavailable.

Questions to answer:

-   Can the server be reached?
-   Is the application listening?
-   Does it return a valid response?
-   Is the correct port open?

Service testing provides these answers.

------------------------------------------------------------------------

# Common Service Testing Tools

  Tool     Primary Purpose
  -------- -------------------------------
  curl     Test HTTP/HTTPS APIs
  wget     Download web resources
  nc       Test TCP/UDP connectivity
  telnet   Manual TCP connection testing

Each tool serves a different role.

------------------------------------------------------------------------

# Linux Perspective

These tools behave like clients.

Instead of inspecting the server internally, they attempt to communicate
with it over the network.

This closely matches the experience of real users.

------------------------------------------------------------------------

# Production Example

A Spring Boot application appears healthy.

The process is running.

The port is listening.

However, users still receive errors.

Running:

``` bash
curl http://server:8080
```

returns:

``` text
HTTP/1.1 500 Internal Server Error
```

The networking is functioning correctly.

The problem exists inside the application.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Why service testing matters.
-   Connectivity vs application availability.
-   The purpose of curl, wget, nc, and telnet.
-   Why administrators test services from the client perspective.

------------------------------------------------------------------------

# ➡️ Next Part

We'll begin with the `curl` command and learn how Linux administrators
test websites and REST APIs in production.
