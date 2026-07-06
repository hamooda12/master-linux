# Chapter 05 --- Users, Groups & Permissions

# Lesson 8 --- Understanding `/etc/passwd`, `/etc/shadow`, `/etc/group`, and `/etc/gshadow`

## Overview

Linux stores user and group information in several text files under
`/etc`. Understanding these files is essential for troubleshooting
authentication, permissions, and account management.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Explain the purpose of the four account database files.
-   Interpret every field in each file.
-   Inspect these files safely.
-   Understand how they work together.

------------------------------------------------------------------------

# 1. `/etc/passwd`

Stores basic user account information.

View it:

``` bash
cat /etc/passwd
```

Example:

``` text
hamad:x:1000:1000:Hamad:/home/hamad:/bin/bash
```

Fields:

  Position   Meaning
  ---------- ----------------------------
  1          Username
  2          Password placeholder (`x`)
  3          UID
  4          Primary GID
  5          GECOS (comment/full name)
  6          Home directory
  7          Login shell

------------------------------------------------------------------------

# 2. `/etc/shadow`

Stores password hashes and password policy.

View (root only):

``` bash
sudo cat /etc/shadow
```

Example:

``` text
hamad:$y$...:20545:0:90:14:::
```

Important fields:

1.  Username
2.  Password hash
3.  Last password change
4.  Minimum password age
5.  Maximum password age
6.  Warning period
7.  Inactive period
8.  Account expiration

A password beginning with `!` is usually locked.

------------------------------------------------------------------------

# 3. `/etc/group`

Stores group definitions.

``` bash
cat /etc/group
```

Example:

``` text
developers:x:1001:hamad,saeed,sara
```

Fields:

  Position   Meaning
  ---------- ----------------------
  1          Group name
  2          Password placeholder
  3          GID
  4          Members

------------------------------------------------------------------------

# 4. `/etc/gshadow`

Stores secure group information.

``` bash
sudo cat /etc/gshadow
```

Example:

``` text
developers:!::hamad,saeed
```

Contains:

-   Group password hash (rarely used)
-   Group administrators
-   Group members

------------------------------------------------------------------------

# Relationship Between the Files

``` text
/etc/passwd  ---> Basic user information
        |
        v
/etc/shadow  ---> Passwords & password policy

/etc/group   ---> Group definitions
        |
        v
/etc/gshadow ---> Secure group information
```

------------------------------------------------------------------------

# Useful Commands

``` bash
grep "^hamad:" /etc/passwd
grep "^hamad:" /etc/shadow
getent passwd hamad
getent group developers
```

Prefer `getent` because it also works with LDAP and other directory
services.

------------------------------------------------------------------------

# Production Scenario

A developer cannot log in.

Investigation:

1.  Check `/etc/passwd` for the shell and home directory.
2.  Check `/etc/shadow` for a locked password or expired account.
3.  Verify group membership with `id` or `getent group`.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
grep "^$USER:" /etc/passwd
sudo grep "^$USER:" /etc/shadow
grep "^sudo:" /etc/group
sudo grep "^sudo:" /etc/gshadow
getent passwd $USER
getent group sudo
```

Identify:

-   UID
-   GID
-   Home directory
-   Login shell
-   Password status

------------------------------------------------------------------------

# Best Practices

-   Never edit these files directly unless absolutely necessary.
-   Use tools like `useradd`, `usermod`, `groupadd`, and `passwd`.
-   Protect `/etc/shadow`.
-   Use `getent` when possible.

------------------------------------------------------------------------

# Common Mistakes

-   Assuming passwords are stored in `/etc/passwd`.
-   Editing account files manually.
-   Ignoring password aging fields in `/etc/shadow`.

------------------------------------------------------------------------

# Interview Questions

1.  Why is the password field in `/etc/passwd` usually `x`?
2.  Which file stores password hashes?
3.  What information is stored in `/etc/group`?
4.  Why should `getent` often be preferred over `cat`?

------------------------------------------------------------------------

# Quick Reference

  File             Purpose
  ---------------- ---------------------------
  `/etc/passwd`    User account information
  `/etc/shadow`    Password hashes and aging
  `/etc/group`     Group definitions
  `/etc/gshadow`   Secure group information

------------------------------------------------------------------------

# Summary

You now understand the four core Linux account databases, what each
stores, and how they work together during authentication and permission
checks.

------------------------------------------------------------------------

# Next Lesson

**Lesson 12 --- Understanding `umask`**
