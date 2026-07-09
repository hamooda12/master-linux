# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 08 --- Network Connectivity (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Host Does Not Respond to Ping

``` bash
ping server.example.com
```

Result:

``` text
100% packet loss
```

Possible causes:

-   Host is powered off
-   Network cable disconnected
-   Router failure
-   Firewall blocks ICMP
-   Wrong IP address

Do not immediately assume the server is down.

------------------------------------------------------------------------

## Scenario 2 --- Ping Works but Website Does Not

``` bash
ping app.example.com
```

Succeeds.

``` bash
curl http://app.example.com
```

Fails.

Possible causes:

-   Web server stopped
-   Wrong listening port
-   Reverse proxy issue
-   Firewall blocking HTTP
-   Application crash

The network is reachable, but the service is unavailable.

------------------------------------------------------------------------

## Scenario 3 --- High Packet Loss

``` text
20 packets transmitted
14 received
30% packet loss
```

Possible causes:

-   Wireless interference
-   Congested network
-   Faulty hardware
-   Router overload

Applications may become slow or disconnect intermittently.

------------------------------------------------------------------------

## Scenario 4 --- Route Failure

``` bash
traceroute app.example.com
```

Output stops after hop 5.

Investigation indicates a routing problem beyond the fifth router.

The application itself may be healthy.

------------------------------------------------------------------------

# Troubleshooting Workflow

``` text
Verify DNS
      │
      ▼
Run ping
      │
      ▼
Inspect latency & packet loss
      │
      ▼
Run traceroute / tracepath
      │
      ▼
Use mtr for long-running analysis
      │
      ▼
Verify application services
```

------------------------------------------------------------------------

# Common Mistakes

## Assuming Ping Proves Everything

A successful ping only proves that ICMP packets reached the destination.

HTTP, HTTPS, SSH, or database services may still be unavailable.

------------------------------------------------------------------------

## Ignoring Packet Loss

Even small packet loss can significantly affect applications such as
video calls and APIs.

------------------------------------------------------------------------

## Treating Every Timeout as a Network Failure

Some servers intentionally block ICMP requests.

Always verify application connectivity using appropriate tools.

------------------------------------------------------------------------

# Best Practices

-   Begin with simple connectivity tests.
-   Escalate to path analysis when necessary.
-   Collect evidence before making changes.
-   Compare latency over time.
-   Verify applications after confirming network health.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What does `ping` test?
2.  What protocol does `ping` use?
3.  What is packet loss?

## Intermediate

1.  Explain TTL.
2.  When would you use `traceroute`?
3.  What advantages does `mtr` provide?

## Advanced

1.  Why can a host respond to `ping` while a website remains
    unavailable?
2.  How would you identify the network hop introducing high latency?

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ping 8.8.8.8
```

``` bash
ping google.com
```

``` bash
tracepath google.com
```

If available:

``` bash
mtr google.com
```

Answer:

1.  What was the average latency?
2.  Was packet loss observed?
3.  How many hops were required?
4.  Did DNS resolution succeed?

------------------------------------------------------------------------

# Challenge Exercise

A user reports:

-   `ping` succeeds.
-   `curl` fails.
-   `traceroute` reaches the destination.

Explain why the issue is probably not basic network connectivity and
describe the next troubleshooting steps.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 08 you understand:

-   ICMP
-   ping
-   traceroute
-   tracepath
-   mtr
-   Latency
-   Packet loss
-   TTL
-   Hop-by-hop path analysis
-   Connectivity troubleshooting methodology

------------------------------------------------------------------------

# Cheat Sheet

  Command        Purpose
  -------------- -------------------------------
  `ping`         Test reachability and latency
  `traceroute`   Display packet path
  `tracepath`    Trace route and discover MTU
  `mtr`          Continuous route analysis
  `Ctrl+C`       Display ping statistics

------------------------------------------------------------------------

# What's Next

Lesson 09 --- Ports and Sockets

You'll learn how Linux applications listen for connections, understand
TCP and UDP ports, inspect sockets using `ss`, `netstat`, and `lsof`,
and troubleshoot why services are unreachable even when the network
itself is healthy.
