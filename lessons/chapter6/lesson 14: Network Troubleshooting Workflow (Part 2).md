# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 14 --- Network Troubleshooting Workflow (Part 2)

------------------------------------------------------------------------

# Step 6 --- Verify Basic Connectivity

After confirming the network configuration, verify whether the
destination is reachable.

Test by IP address first:

``` bash
ping 8.8.8.8
```

If this succeeds, basic Layer 3 connectivity exists.

Then test using a hostname:

``` bash
ping google.com
```

If the IP works but the hostname fails, investigate DNS.

------------------------------------------------------------------------

# Step 7 --- Identify the Network Path

If connectivity fails or is intermittent, determine where packets stop.

Use:

``` bash
tracepath google.com
```

or

``` bash
traceroute google.com
```

These commands display every hop between your system and the
destination.

If the route stops unexpectedly, the problem may exist between routers
rather than on either endpoint.

------------------------------------------------------------------------

# Step 8 --- Verify Firewall Rules

A healthy network does not guarantee that traffic is allowed.

Check the firewall:

``` bash
sudo ufw status
```

Verify:

-   Is the firewall active?
-   Is the required port allowed?
-   Is the correct protocol (TCP/UDP) permitted?

Always compare firewall rules with the application's listening ports.

------------------------------------------------------------------------

# Step 9 --- Verify Listening Services

Determine whether the application is accepting connections.

``` bash
ss -tulpn
```

Questions to answer:

-   Is the application listening?
-   Which port is it using?
-   Which process owns the socket?
-   Is it listening on the correct interface?

Example:

``` text
tcp LISTEN 0 128 0.0.0.0:8080
```

------------------------------------------------------------------------

# Step 10 --- Test the Service

Finally, interact with the application exactly as a client would.

Example:

``` bash
curl http://localhost:8080
```

Possible results:

``` text
200 OK
```

Application is responding correctly.

Or:

``` text
500 Internal Server Error
```

Networking is healthy, but the application has failed internally.

------------------------------------------------------------------------

# Complete Investigation Flow

``` text
Problem Report
      │
      ▼
ip link
      │
      ▼
ip addr
      │
      ▼
ip route
      │
      ▼
DNS
      │
      ▼
ping
      │
      ▼
tracepath / traceroute
      │
      ▼
ufw
      │
      ▼
ss
      │
      ▼
curl
```

Each step builds on the previous one.

Never skip directly to application debugging without verifying the lower
layers.

------------------------------------------------------------------------

# Linux Perspective

Most production networking issues are isolated quickly by following this
sequence.

The goal is not to memorize commands---it is to investigate logically
and collect evidence before making changes.

------------------------------------------------------------------------

# Production Example

Users report that a web application is unavailable.

Investigation:

-   `ip addr` → Correct IP ✅
-   `ip route` → Gateway present ✅
-   `ping` → Reachable ✅
-   `ss` → Application listening ✅
-   `ufw` → Port 8080 blocked ❌

Root cause: firewall configuration.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Connectivity verification
-   Route analysis
-   Firewall verification
-   Listening socket verification
-   Service testing
-   Complete investigation flow

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the networking chapter with realistic production
scenarios, interview questions, hands-on labs, challenge exercises, and
a comprehensive networking troubleshooting cheat sheet.
