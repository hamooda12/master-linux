# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 14 --- Network Troubleshooting Workflow (Part 3)

------------------------------------------------------------------------

# Production Troubleshooting Scenarios

## Scenario 1 --- Cannot Reach the Internet

### Symptoms

-   Local network works.
-   Internet access fails.

### Investigation

``` bash
ip route
```

No default route exists.

### Root Cause

The system has no gateway for destinations outside the local network.

------------------------------------------------------------------------

## Scenario 2 --- IP Works, Domain Does Not

### Symptoms

``` bash
ping 8.8.8.8
```

Succeeds.

``` bash
ping google.com
```

Fails.

### Investigation

``` bash
cat /etc/resolv.conf
```

``` bash
dig google.com
```

### Root Cause

DNS configuration is incorrect or unavailable.

------------------------------------------------------------------------

## Scenario 3 --- Service Running but Unreachable

### Investigation

``` bash
ss -tulpn
```

Output:

``` text
tcp LISTEN 127.0.0.1:8080
```

### Root Cause

The application listens only on the loopback interface.

Remote systems cannot connect.

------------------------------------------------------------------------

## Scenario 4 --- Firewall Blocking Traffic

Application:

``` bash
ss -tulpn
```

Shows:

``` text
0.0.0.0:443
```

Firewall:

``` bash
sudo ufw status
```

Shows no rule allowing HTTPS.

### Root Cause

Firewall configuration blocks incoming traffic.

------------------------------------------------------------------------

## Scenario 5 --- Wrong Static Configuration

``` bash
ip addr
```

Shows:

``` text
10.10.20.50/24
```

Production network:

``` text
192.168.1.0/24
```

### Root Cause

Incorrect network configuration prevents communication.

------------------------------------------------------------------------

# Layer-by-Layer Thinking

Professional administrators troubleshoot from the bottom upward.

    Application
          ▲
    Service
          ▲
    Socket
          ▲
    Firewall
          ▲
    DNS
          ▲
    Routing
          ▲
    IP Address
          ▲
    Interface

Never investigate the upper layers before verifying the lower ones.

------------------------------------------------------------------------

# Evidence-Based Troubleshooting

Avoid statements such as:

> "The firewall must be broken."

Instead collect evidence:

``` bash
sudo ufw status
```

Avoid:

> "The application isn't running."

Verify:

``` bash
ss -tulpn
```

Evidence always comes before conclusions.

------------------------------------------------------------------------

# Production Example

A customer cannot access an internal API.

Investigation sequence:

1.  `ip link`
2.  `ip addr`
3.  `ip route`
4.  `dig api.internal`
5.  `ping`
6.  `ss`
7.  `ufw`
8.  `curl`

The investigation reveals that the application is healthy, but the
firewall blocks TCP port **8443**.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Real production networking failures.
-   Layer-by-layer troubleshooting.
-   Evidence-based investigation.
-   Common networking root causes.

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the networking chapter with interview questions, hands-on
labs, challenge exercises, a comprehensive networking cheat sheet, and a
final review of everything you've learned.
