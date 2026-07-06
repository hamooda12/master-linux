# Chapter 05 --- Users, Groups & Permissions

# Lesson 12 --- File Attributes (`lsattr` and `chattr`)

## Overview

Linux file attributes provide an extra layer of protection beyond
standard permissions. They are commonly used to protect configuration
files and log files on production servers.

------------------------------------------------------------------------

# Learning Objectives

-   Understand file attributes.
-   View attributes with `lsattr`.
-   Modify attributes with `chattr`.
-   Use immutable (`+i`) and append-only (`+a`) attributes.
-   Apply file attributes in production.

------------------------------------------------------------------------

# Viewing Attributes

``` bash
lsattr
lsattr config.yml
```

Example:

``` text
----i--------- config.yml
```

`i` means the file is immutable.

------------------------------------------------------------------------

# Immutable Attribute

Enable:

``` bash
sudo chattr +i config.yml
```

Effects:

-   Cannot edit
-   Cannot rename
-   Cannot delete
-   Cannot truncate

Remove:

``` bash
sudo chattr -i config.yml
```

------------------------------------------------------------------------

# Append-only Attribute

Enable:

``` bash
sudo chattr +a application.log
```

Only new data may be appended.

Remove:

``` bash
sudo chattr -a application.log
```

------------------------------------------------------------------------

# Production Examples

Protect an important configuration:

``` bash
sudo chattr +i /etc/nginx/nginx.conf
```

Protect a log file:

``` bash
sudo chattr +a /var/log/app.log
```

------------------------------------------------------------------------

# Hands-on Lab

``` bash
touch demo.txt
lsattr demo.txt

sudo chattr +i demo.txt

echo test >> demo.txt
rm demo.txt

sudo chattr -i demo.txt
rm demo.txt
```

------------------------------------------------------------------------

# Best Practices

-   Use `+i` for critical configuration files.
-   Use `+a` for important logs.
-   Document protected files.
-   Remove attributes before planned maintenance.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting a file is immutable.
-   Expecting `chmod` to override attributes.
-   Protecting files that applications must modify.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between permissions and file attributes?
2.  What does `+i` do?
3.  What does `+a` do?
4.  Which command shows attributes?
5.  When would you use immutable files?

------------------------------------------------------------------------

# Quick Reference

  Task                 Command
  -------------------- ------------------
  View attributes      `lsattr`
  Immutable            `chattr +i file`
  Remove immutable     `chattr -i file`
  Append only          `chattr +a file`
  Remove append only   `chattr -a file`

------------------------------------------------------------------------

# Summary

You learned how to use Linux file attributes to protect files beyond
normal permissions.

------------------------------------------------------------------------

# Chapter 05 Complete ✅

Next: \*\*Chapter 05 Final Review, Practical Lab, Mini Production
Scenario, and Bootcamp.
