# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 10 --- Testing Network Services (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Ping Works but API Fails

``` bash
ping api.example.com
```

Succeeds.

``` bash
curl https://api.example.com/health
```

Returns:

``` text
HTTP/1.1 500 Internal Server Error
```

### Investigation

The network is functioning correctly.

The application encountered an internal error.

------------------------------------------------------------------------

## Scenario 2 --- Port Closed

``` bash
nc -zv server.example.com 443
```

Output:

``` text
Connection refused
```

### Investigation

Possible causes:

-   Service not running
-   Wrong listening port
-   Firewall configuration
-   Service startup failure

------------------------------------------------------------------------

## Scenario 3 --- File Download Fails

``` bash
wget https://example.com/file.zip
```

Output:

``` text
404 Not Found
```

### Investigation

The web server is reachable.

The requested resource does not exist at the specified URL.

------------------------------------------------------------------------

## Scenario 4 --- Redirect Loop

``` bash
curl -L https://example.com
```

Never completes successfully.

### Investigation

The web server is repeatedly redirecting the client.

Review the web server or reverse proxy configuration.

------------------------------------------------------------------------

# Troubleshooting Workflow

``` text
Verify DNS
      │
      ▼
Test connectivity
      │
      ▼
Verify listening port
      │
      ▼
Test service with curl
      │
      ▼
Download resource if needed
      │
      ▼
Review application logs
```

------------------------------------------------------------------------

# Common Mistakes

## Assuming an Open Port Means the Application Works

An application may accept TCP connections while returning HTTP errors.

Always verify the service response.

------------------------------------------------------------------------

## Using the Wrong Tool

-   Use `curl` for APIs.
-   Use `wget` for downloads.
-   Use `nc` for port testing.
-   Use `telnet` only when appropriate or in legacy environments.

------------------------------------------------------------------------

## Ignoring HTTP Status Codes

Status codes provide valuable information:

-   **200 OK** -- Request succeeded.
-   **301/302** -- Redirect.
-   **403** -- Access forbidden.
-   **404** -- Resource not found.
-   **500** -- Internal server error.
-   **503** -- Service unavailable.

------------------------------------------------------------------------

# Best Practices

-   Test services exactly as clients use them.
-   Verify both connectivity and application responses.
-   Check HTTP status codes before restarting services.
-   Combine networking tools to narrow down the root cause.
-   Base every troubleshooting step on evidence.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is `curl` used for?
2.  What is the difference between `curl` and `wget`?
3.  What does `nc -zv` test?

## Intermediate

1.  Why can `curl` fail while `ping` succeeds?
2.  When would you use `wget` instead of `curl`?
3.  Why is Netcat often called the "Swiss Army knife" of networking?

## Advanced

1.  How would you determine whether a failure is caused by the network
    or the application?
2.  Describe a troubleshooting workflow using `curl`, `nc`, and `ss`.

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
curl -I https://example.com
```

``` bash
wget https://example.com
```

``` bash
nc -zv google.com 443
```

If installed:

``` bash
telnet google.com 80
```

Answer:

1.  Which HTTP status code was returned?
2.  Was the file downloaded successfully?
3.  Was port 443 open?
4.  Did the TCP connection succeed?

------------------------------------------------------------------------

# Challenge Exercise

A user reports that a web application is unavailable.

Design an investigation using only:

-   `curl`
-   `wget`
-   `nc`
-   `ss`

Explain how each command helps identify the root cause.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 10 you understand:

-   Service testing fundamentals
-   `curl`
-   `wget`
-   `nc`
-   `telnet`
-   HTTP status codes
-   API testing
-   Port verification
-   Client-side troubleshooting methodology

------------------------------------------------------------------------

# Cheat Sheet

  Command              Purpose
  -------------------- ----------------------------
  `curl URL`           Test HTTP/HTTPS services
  `curl -I URL`        Display HTTP headers
  `curl -L URL`        Follow redirects
  `wget URL`           Download resources
  `wget -O file URL`   Save with custom filename
  `nc -zv host port`   Test TCP port connectivity
  `telnet host port`   Basic TCP connection test

------------------------------------------------------------------------

# What's Next

Lesson 11 --- Network Troubleshooting Workflow

You'll combine everything learned in this chapter into a systematic
troubleshooting methodology used by Linux administrators and DevOps
engineers to diagnose real production networking issues.
