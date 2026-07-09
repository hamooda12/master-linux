# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 10 --- Testing Network Services (Part 2)

------------------------------------------------------------------------

# The `curl` Command

## Purpose

`curl` is one of the most important networking tools for Linux
administrators and DevOps engineers.

It allows you to communicate directly with network services from the
command line.

Unlike a web browser, `curl` displays the server's response without
rendering HTML.

------------------------------------------------------------------------

# Basic Syntax

``` bash
curl <URL>
```

Examples:

``` bash
curl http://example.com
```

``` bash
curl https://example.com
```

------------------------------------------------------------------------

# Testing a Web Server

Suppose a Spring Boot application is running on port 8080.

``` bash
curl http://localhost:8080
```

Possible output:

``` text
Hello, World!
```

This confirms:

-   The application is running.
-   The service accepted the request.
-   A response was returned.

------------------------------------------------------------------------

# Viewing HTTP Headers

To display only the response headers:

``` bash
curl -I https://example.com
```

Example output:

``` text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 648
```

Headers provide useful diagnostic information without downloading the
full page.

------------------------------------------------------------------------

# Following Redirects

Some websites automatically redirect clients.

Example:

``` bash
curl http://example.com
```

may return:

``` text
301 Moved Permanently
```

To follow redirects automatically:

``` bash
curl -L http://example.com
```

------------------------------------------------------------------------

# Saving the Response

Instead of displaying the response on the terminal:

``` bash
curl -o page.html https://example.com
```

The output is saved to:

``` text
page.html
```

------------------------------------------------------------------------

# Testing REST APIs

Example:

``` bash
curl http://localhost:8080/api/users
```

Possible JSON response:

``` json
[
  {
    "id": 1,
    "name": "Alice"
  }
]
```

This makes `curl` indispensable for testing APIs.

------------------------------------------------------------------------

# Sending HTTP Methods

Retrieve data:

``` bash
curl -X GET http://localhost:8080/api/users
```

Create data:

``` bash
curl -X POST http://localhost:8080/api/users
```

Update data:

``` bash
curl -X PUT http://localhost:8080/api/users/1
```

Delete data:

``` bash
curl -X DELETE http://localhost:8080/api/users/1
```

------------------------------------------------------------------------

# Linux Perspective

Most API troubleshooting begins with `curl`.

It answers questions such as:

-   Does the server respond?
-   What HTTP status code is returned?
-   Is JSON returned correctly?
-   Is authentication required?

------------------------------------------------------------------------

# Production Example

Users report that an API is unavailable.

Running:

``` bash
curl http://api.example.com/health
```

returns:

``` text
HTTP/1.1 503 Service Unavailable
```

Networking is functioning.

The application itself reports that it cannot currently serve requests.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   `curl`
-   HTTP requests
-   Response headers
-   Redirects
-   REST API testing
-   Basic HTTP methods

------------------------------------------------------------------------

# ➡️ Next Part

We'll study `wget`, `nc` (Netcat), and `telnet`, and learn when each
tool is the best choice for testing network services.
