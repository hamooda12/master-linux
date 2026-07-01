# Chapter 01 --- Lesson 15: Changing Ownership with `chown` and `chgrp`

## Overview

Permissions determine what users can do with a file, while ownership
determines who controls it. In this lesson, you will learn how to change
the owner and group of files and directories using the `chown` and
`chgrp` commands.

These commands are commonly used by Linux administrators when managing
shared systems and servers.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Understand file ownership
-   Change a file owner with `chown`
-   Change a file group with `chgrp`
-   Change both owner and group together
-   Apply ownership changes recursively

------------------------------------------------------------------------

# Prerequisites

-   Linux file permissions
-   Using `ls -l`
-   Understanding users and groups
-   Basic `chmod` usage

------------------------------------------------------------------------

# Theory

Every file in Linux belongs to:

-   An **owner**
-   A **group**

Example:

``` text
-rw-r--r-- 1 hamad developers 1200 Jul 1 report.txt
```

Here:

-   Owner: `hamad`
-   Group: `developers`

Ownership controls who is responsible for a file and who can manage its
permissions.

------------------------------------------------------------------------

# Viewing Ownership

``` bash
ls -l
```

Example:

``` text
-rw-r--r-- 1 hamad developers report.txt
```

------------------------------------------------------------------------

# Command: `chown`

`chown` stands for **Change Owner**.

## Syntax

``` bash
sudo chown owner filename
```

## Example

``` bash
sudo chown alice report.txt
```

The file owner becomes `alice`.

------------------------------------------------------------------------

## Change Owner and Group Together

``` bash
sudo chown alice:developers report.txt
```

This changes:

-   Owner → `alice`
-   Group → `developers`

------------------------------------------------------------------------

## Recursive Ownership Change

``` bash
sudo chown -R alice:developers project/
```

The `-R` option applies the change to every file and subdirectory.

------------------------------------------------------------------------

# Command: `chgrp`

`chgrp` stands for **Change Group**.

## Syntax

``` bash
sudo chgrp group_name filename
```

## Example

``` bash
sudo chgrp developers report.txt
```

Only the group changes.

------------------------------------------------------------------------

# Practical Scenario

A development team shares a project directory.

The administrator changes ownership:

``` bash
sudo chown -R alice:developers web-app
```

Now Alice owns the project, and every member of the `developers` group
can collaborate according to the assigned permissions.

------------------------------------------------------------------------

# Hands-on Lab

> **Note:** These commands require administrative privileges and
> additional users/groups. If your system does not have multiple users,
> observe the syntax and practice on a test environment.

``` bash
cd ~/linux-practice/practice

touch report.txt

ls -l report.txt

sudo chown root report.txt

ls -l report.txt

sudo chgrp root report.txt

ls -l report.txt
```

If available, create a test directory and apply recursive ownership:

``` bash
mkdir demo

touch demo/file1.txt

touch demo/file2.txt

sudo chown -R root:root demo

ls -lR demo
```

------------------------------------------------------------------------

# Challenge

1.  Create a test directory.
2.  Create two files inside it.
3.  Display the current owner and group.
4.  Change the owner (if permitted).
5.  Change the group.
6.  Verify the results using `ls -l`.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand file ownership.
-   [ ] I can identify the owner and group.
-   [ ] I know how to use `chown`.
-   [ ] I know how to use `chgrp`.
-   [ ] I understand recursive ownership changes.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting that changing ownership usually requires `sudo`.
-   Confusing permissions with ownership.
-   Applying recursive ownership changes to the wrong directory.

------------------------------------------------------------------------

# Best Practices

-   Verify ownership with `ls -l` before making changes.
-   Use recursive operations carefully.
-   Change ownership only when necessary.
-   Follow the principle of least privilege.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between ownership and permissions?
2.  What does `chown` do?
3.  What does `chgrp` do?
4.  Why is `sudo` often required with `chown`?
5.  What is the purpose of the `-R` option?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I identify the owner and group of a file?
-   Do I understand when ownership should be changed?
-   Can I explain the difference between `chmod` and `chown`?

------------------------------------------------------------------------

# Summary

Ownership is a fundamental part of Linux security. By using `chown` and
`chgrp`, administrators can assign responsibility for files and
directories while working together with Linux permissions to secure the
system.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -----------------------------------------------------------------------
  Filename                            Capture
  ----------------------------------- -----------------------------------
  `62-file-owner-before.png`          Ownership before changes

  `63-chown-example.png`              Changing the owner with `chown`

  `64-chgrp-example.png`              Changing the group with `chgrp`

  `65-recursive-chown.png`            Recursive ownership change

  `66-ownership-lab-complete.png`     Final terminal after completing the
                                      lab
  -----------------------------------------------------------------------
