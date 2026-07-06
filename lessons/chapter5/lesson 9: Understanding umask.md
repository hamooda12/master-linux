# Chapter 05 --- Users, Groups & Permissions

# Lesson 9 --- Understanding `umask`

## Overview

Whenever you create a new file or directory, Linux assigns default
permissions automatically. The **umask** (user file-creation mode mask)
determines which permission bits are removed from those defaults.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Explain what `umask` is.
-   View and temporarily change the current umask.
-   Calculate default permissions for new files and directories.
-   Understand common umask values.
-   Apply umask in real-world administration.

------------------------------------------------------------------------

# What Is umask?

`umask` does **not** add permissions.

It **removes** permissions from the system defaults.

Default creation permissions are:

  Object      Default
  ----------- ---------------------
  File        `666` (`rw-rw-rw-`)
  Directory   `777` (`rwxrwxrwx`)

The umask subtracts permissions from these defaults.

------------------------------------------------------------------------

# Viewing the Current umask

``` bash
umask
```

Example:

``` text
0022
```

Symbolic form:

``` bash
umask -S
```

Example:

``` text
u=rwx,g=rx,o=rx
```

------------------------------------------------------------------------

# Calculating Permissions

## Example 1

    File default:      666
    umask:             022
    -----------------------
    Result:            644

New file:

``` text
-rw-r--r--
```

## Example 2

    Directory default: 777
    umask:             022
    -----------------------
    Result:            755

New directory:

``` text
drwxr-xr-x
```

------------------------------------------------------------------------

# Common umask Values

    umask  Files   Directories  Typical Use
  ------- ------- ------------- --------------------------
      022   644        755      Standard Linux systems
      002   664        775      Shared team environments
      027   640        750      More restrictive servers
      077   600        700      Private accounts

------------------------------------------------------------------------

# Temporarily Changing umask

``` bash
umask 027
```

Create a file:

``` bash
touch secret.txt
ls -l secret.txt
```

Restore:

``` bash
umask 022
```

------------------------------------------------------------------------

# Making umask Persistent

Common locations:

-   `~/.bashrc`
-   `~/.profile`
-   `/etc/profile`
-   `/etc/bash.bashrc` (distribution dependent)

Example:

``` bash
umask 027
```

------------------------------------------------------------------------

# Production Scenario

A company stores confidential configuration files.

Policy:

-   Owner: read/write
-   Group: read
-   Others: no access

Administrators set:

``` bash
umask 027
```

New files become:

``` text
-rw-r-----
```

New directories become:

``` text
drwxr-x---
```

without manually running `chmod` each time.

------------------------------------------------------------------------

# Hands-on Lab

Check your current mask:

``` bash
umask
umask -S
```

Create test objects:

``` bash
touch file1
mkdir dir1

ls -ld file1 dir1
```

Change the mask:

``` bash
umask 077

touch private.txt
mkdir private_dir

ls -ld private.txt private_dir
```

Restore:

``` bash
umask 022
```

------------------------------------------------------------------------

# Best Practices

-   Understand the difference between default permissions and umask.
-   Choose restrictive values for sensitive systems.
-   Use `002` for collaborative teams when appropriate.
-   Verify results with `ls -l`.

------------------------------------------------------------------------

# Common Mistakes

-   Thinking umask grants permissions.
-   Forgetting to restore a temporary umask.
-   Using overly permissive values on production systems.

------------------------------------------------------------------------

# Interview Questions

1.  What is the purpose of `umask`?
2.  Why are new files based on `666` instead of `777`?
3.  What permissions result from a umask of `027`?
4.  When would you use `002` instead of `022`?

------------------------------------------------------------------------

# Quick Reference

  Command        Purpose
  -------------- ----------------------------
  `umask`        Show current umask
  `umask -S`     Show symbolic form
  `umask 022`    Set temporary umask
  `touch file`   Test file permissions
  `mkdir dir`    Test directory permissions

------------------------------------------------------------------------

# Summary

You learned how Linux calculates default permissions, how `umask`
removes permission bits, and how to configure appropriate defaults for
personal, collaborative, and production environments.

------------------------------------------------------------------------

# Next Lesson

**Lesson 13 --- Access Control Lists (ACL)**
