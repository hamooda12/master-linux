# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 14 --- Network Troubleshooting Workflow (Part 1)

> **Goal:** Learn a systematic process for diagnosing Linux networking
> problems without guessing or making unnecessary changes.

------------------------------------------------------------------------

# 📚 Overview

Professional Linux administrators rarely solve networking problems by
trial and error.

Instead, they investigate one layer at a time.

A structured troubleshooting workflow saves time, reduces mistakes, and
minimizes production downtime.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Apply a structured troubleshooting methodology.
-   Identify the correct layer to investigate first.
-   Use the networking tools learned throughout this chapter together.
-   Avoid common troubleshooting mistakes.

------------------------------------------------------------------------

# The Golden Rule

> **Always collect evidence before making changes.**

Do **not** restart services, edit configuration files, or reboot a
server until you understand the problem.

------------------------------------------------------------------------

# Step 1 --- Define the Problem

Ask questions such as:

-   What exactly is failing?
-   Does the problem affect one user or everyone?
-   Did it ever work?
-   What changed recently?

Clearly defining the problem prevents unnecessary investigation.

------------------------------------------------------------------------

# Step 2 --- Verify Physical & Interface Status

Check that the interface exists and is operational.

``` bash
ip link
```

Questions to answer:

-   Is the interface present?
-   Is it UP?
-   Is the expected interface being used?

------------------------------------------------------------------------

# Step 3 --- Verify IP Configuration

``` bash
ip addr
```

Confirm:

-   Correct IP address
-   Correct subnet prefix
-   Correct interface

Without a valid IP address, higher-level troubleshooting is pointless.

------------------------------------------------------------------------

# Step 4 --- Verify Routing

``` bash
ip route
```

Confirm:

-   Default gateway
-   Local routes
-   Static routes (if applicable)

A missing or incorrect route prevents communication outside the local
network.

------------------------------------------------------------------------

# Step 5 --- Verify DNS

``` bash
cat /etc/resolv.conf
```

Then test:

``` bash
host example.com
```

or

``` bash
dig example.com
```

Remember:

If `ping 8.8.8.8` works but `ping google.com` fails, DNS is a likely
cause.

------------------------------------------------------------------------

# Production Example

A web application appears offline.

Investigation shows:

-   Interface is UP ✅
-   IP address is correct ✅
-   Default gateway exists ✅
-   DNS lookup fails ❌

The root cause is DNS---not the application.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Why troubleshooting should be systematic.
-   The first five investigation steps.
-   How to combine Linux networking commands effectively.

------------------------------------------------------------------------

# ➡️ Next Part

We'll continue the workflow by verifying connectivity, firewall rules,
listening services, and application health.
