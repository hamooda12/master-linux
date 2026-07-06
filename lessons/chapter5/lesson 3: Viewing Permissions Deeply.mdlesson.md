# Chapter 05 --- Users, Groups & Permissions

# Lesson 3 --- Viewing Permissions Deeply

## Overview

Before changing permissions, a Linux administrator must know how to
inspect them accurately. This lesson explains how to read permission
information using `ls -l`, `stat`, and `namei`.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Interpret every column of `ls -l`.
-   Use `stat` to inspect detailed file metadata.
-   Use `namei` to troubleshoot permission problems in directory paths.
-   Identify ownership and permission issues quickly.

------------------------------------------------------------------------

# 1. Viewing Permissions with ls -l

Run:

``` bash
ls -l
```

Example:

``` text
-rwxr-x--- 1 hamad developers 2048 Jul 6 10:30 deploy.sh
```

Meaning:

  Field           Description
  --------------- --------------------------------------------------------------
  `-`             File type (`d` = directory, `-` = file, `l` = symbolic link)
  `rwx`           Owner permissions
  `r-x`           Group permissions
  `---`           Others permissions
  `1`             Number of hard links
  `hamad`         Owner
  `developers`    Group
  `2048`          File size (bytes)
  `Jul 6 10:30`   Last modification time
  `deploy.sh`     File name

------------------------------------------------------------------------

# 2. Viewing Detailed Metadata with stat

Run:

``` bash
stat deploy.sh
```

Useful information includes:

-   File size
-   File type
-   Owner
-   Group
-   Permission bits (symbolic and octal)
-   Access, Modify, and Change timestamps
-   Inode number

This is especially useful when debugging ownership or timestamp issues.

------------------------------------------------------------------------

# 3. Inspecting Every Directory with namei

Suppose a user cannot access:

``` text
/opt/company/app/config.yml
```

Run:

``` bash
namei -l /opt/company/app/config.yml
```

Example output:

``` text
drwxr-xr-x root root /
drwxr-xr-x root root opt
drwxr-x--- root devops company
drwxrwxr-x hamad developers app
-rw-r----- hamad developers config.yml
```

`namei` shows the permissions of **every directory** in the path, making
it easy to find where access is blocked.

------------------------------------------------------------------------

# Production Scenario

A developer says:

> "I have permission to read `config.yml`, but I still get 'Permission
> denied'."

The file permissions may be correct, but one parent directory might not
have execute (`x`) permission for that user.

`namei -l` quickly identifies the problem.

------------------------------------------------------------------------

# Best Practices

-   Inspect permissions before changing them.
-   Use `stat` when timestamps or ownership matter.
-   Use `namei` when troubleshooting path access.
-   Avoid guessing permission problems.

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ls -l
```

Run:

``` bash
stat ~/.bashrc
```

Run:

``` bash
namei -l ~/.bashrc
```

Answer:

1.  Who owns the file?
2.  Which group owns it?
3.  What are the permission bits?
4.  Can others read it?
5.  Which directories must be executable to reach it?

------------------------------------------------------------------------

# Common Mistakes

-   Looking only at the target file.
-   Forgetting that directories also require execute (`x`) permission.
-   Using `ls` when detailed metadata from `stat` is needed.

------------------------------------------------------------------------

# Interview Questions

1.  What information does `ls -l` display?
2.  When would you use `stat` instead of `ls -l`?
3.  What problem does `namei` solve?
4.  Why can a readable file still produce "Permission denied"?

------------------------------------------------------------------------

# Summary

You learned how to:

-   Read `ls -l` output.
-   Inspect detailed metadata with `stat`.
-   Troubleshoot path permissions using `namei`.
-   Diagnose common permission issues.

------------------------------------------------------------------------

# Next Lesson

**Lesson 7 --- Creating Users**
