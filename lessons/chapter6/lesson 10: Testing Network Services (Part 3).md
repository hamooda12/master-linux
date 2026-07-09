# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 10 --- Testing Network Services (Part 3)

------------------------------------------------------------------------

# The `wget` Command

## Purpose

`wget` is used to download files and web resources from HTTP, HTTPS, and
FTP servers.

Unlike `curl`, which is primarily used to interact with web services and
APIs, `wget` focuses on retrieving content and saving it to disk.

------------------------------------------------------------------------

# Basic Syntax

``` bash
wget <URL>
```

Example:

``` bash
wget https://example.com/index.html
```

The downloaded file is saved in the current directory.

------------------------------------------------------------------------

# Downloading a File with a Custom Name

``` bash
wget -O homepage.html https://example.com
```

The `-O` option specifies the output filename.

------------------------------------------------------------------------

# Recursive Downloads (Overview)

`wget` can download an entire website recursively.

Example:

``` bash
wget --recursive https://example.com
```

This feature is useful for offline copies but should be used responsibly
because it may generate significant traffic.

------------------------------------------------------------------------

# The `nc` (Netcat) Command

## Purpose

Netcat is often called the **Swiss Army knife of networking**.

It can:

-   Test whether a TCP or UDP port is open.
-   Send and receive data.
-   Create simple client/server connections.
-   Assist in troubleshooting network services.

------------------------------------------------------------------------

# Testing a TCP Port

Example:

``` bash
nc -zv localhost 8080
```

Possible output:

``` text
Connection to localhost 8080 port [tcp/*] succeeded!
```

If the port is closed:

``` text
Connection refused
```

This immediately tells you whether a service is accepting TCP
connections.

------------------------------------------------------------------------

# Testing Multiple Ports

``` bash
nc -zv localhost 22 80 443
```

Netcat checks each specified port.

------------------------------------------------------------------------

# The `telnet` Command

## Purpose

`telnet` was originally designed for remote terminal access.

Today it is commonly used as a simple way to verify that a TCP port
accepts connections.

Example:

``` bash
telnet example.com 80
```

If the connection succeeds:

``` text
Connected to example.com.
```

Although modern systems generally prefer `nc`, you may still encounter
`telnet` in documentation and older environments.

------------------------------------------------------------------------

# Comparing the Tools

  Command    Best Used For
  ---------- ------------------------------------
  `curl`     Test HTTP/HTTPS services and APIs
  `wget`     Download files and web resources
  `nc`       Test TCP/UDP ports and connections
  `telnet`   Basic TCP connectivity testing

Each tool serves a different purpose.

------------------------------------------------------------------------

# Linux Perspective

Professional administrators often use these commands together.

Example workflow:

1.  Verify the service responds with `curl`.
2.  Download resources using `wget`.
3.  Confirm the port is open using `nc`.
4.  Use `telnet` if required by legacy documentation.

------------------------------------------------------------------------

# Production Example

A monitoring system reports that an API is unavailable.

Running:

``` bash
nc -zv api.example.com 443
```

returns:

``` text
Connection succeeded.
```

However:

``` bash
curl https://api.example.com
```

returns:

``` text
HTTP/1.1 503 Service Unavailable
```

The network path and port are functioning correctly.

The application itself is reporting an error.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   `wget`
-   `nc`
-   `telnet`
-   Downloading files
-   Testing TCP ports
-   Comparing Linux service-testing tools

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the lesson with production troubleshooting scenarios,
interview questions, hands-on labs, challenge exercises, and a complete
cheat sheet for testing network services.
