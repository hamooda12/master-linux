# Chapter 05 --- Users, Groups & Permissions

# Lesson 14 --- `sudo` and `visudo`

## Overview

Most Linux administrators do **not** log in as `root` for daily work.
Instead, they use `sudo` to execute only the commands that require
administrative privileges. This follows the **Principle of Least
Privilege**, reducing the risk of accidental or malicious system
changes.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Explain the purpose of `sudo`.
-   Understand how `sudo` works.
-   Inspect sudo privileges.
-   Edit the sudo configuration safely with `visudo`.
-   Apply least privilege in production.

------------------------------------------------------------------------

# Why Use sudo?

Instead of logging in as `root`:

``` bash
su -
```

Linux administrators usually run:

``` bash
sudo apt update
```

Only that command executes with elevated privileges.

Benefits:

-   Reduces accidental damage.
-   Creates an audit trail.
-   Allows fine-grained privilege delegation.
-   Encourages least privilege.

------------------------------------------------------------------------

# Running Commands with sudo

Examples:

``` bash
sudo systemctl restart nginx
sudo useradd hamad
sudo passwd saeed
```

------------------------------------------------------------------------

# Checking sudo Access

Show your permissions:

``` bash
sudo -l
```

Example output:

``` text
User hamad may run the following commands:
    (ALL : ALL) ALL
```

------------------------------------------------------------------------

# The sudoers File

Configuration is stored in:

``` text
/etc/sudoers
```

**Never edit this file directly** with editors like `nano` or `vim`.

Instead use:

``` bash
sudo visudo
```

`visudo`:

-   Validates the syntax.
-   Prevents simultaneous edits.
-   Reduces the chance of locking yourself out.

------------------------------------------------------------------------

# Granting sudo Access

Example entry:

``` text
hamad ALL=(ALL:ALL) ALL
```

Meaning:

-   User: `hamad`
-   Host: `ALL`
-   May act as: `ALL`
-   Commands: `ALL`

A common approach is to add users to the `sudo` group:

``` bash
sudo usermod -aG sudo saeed
```

Verify:

``` bash
groups saeed
```

------------------------------------------------------------------------

# sudo Timestamp

After entering your password once, `sudo` remembers your authentication
for a short period.

Force it to ask again:

``` bash
sudo -k
```

------------------------------------------------------------------------

# Production Scenario

A junior administrator needs to restart web services but should not have
unrestricted root access.

Instead of sharing the root password:

1.  Create a normal user.
2.  Grant controlled sudo privileges.
3.  Log all administrative actions.

------------------------------------------------------------------------

# Hands-on Lab

Check your privileges:

``` bash
sudo -l
```

Clear the cached credentials:

``` bash
sudo -k
```

Run:

``` bash
sudo whoami
```

Expected output:

``` text
root
```

Check your groups:

``` bash
groups
```

------------------------------------------------------------------------

# Best Practices

-   Use `sudo` instead of logging in as `root`.
-   Edit the sudoers file only with `visudo`.
-   Grant the minimum required privileges.
-   Review sudo access regularly.
-   Never share the root password.

------------------------------------------------------------------------

# Common Mistakes

-   Editing `/etc/sudoers` directly.
-   Giving every user unrestricted sudo access.
-   Logging in as `root` for routine administration.
-   Forgetting to verify sudo permissions with `sudo -l`.

------------------------------------------------------------------------

# Interview Questions

1.  Why is `sudo` preferred over logging in as `root`?
2.  What does `sudo -l` do?
3.  Why should you use `visudo` instead of `nano /etc/sudoers`?
4.  What does `sudo -k` do?
5.  What is the Principle of Least Privilege?

------------------------------------------------------------------------

# Quick Reference

  Task                     Command
  ------------------------ ------------------------------
  Run as root              `sudo command`
  Show permissions         `sudo -l`
  Clear cache              `sudo -k`
  Edit sudoers safely      `sudo visudo`
  Add user to sudo group   `sudo usermod -aG sudo user`

------------------------------------------------------------------------

# Summary

You learned how `sudo` safely provides administrative access, why
`visudo` is essential, and how to delegate privileges securely while
following the Principle of Least Privilege.

------------------------------------------------------------------------

# Next Lesson

**Lesson 16 --- File Attributes (`lsattr` and `chattr`)**
