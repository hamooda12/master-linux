# Chapter 01 --- Lesson 13: Linux File Permissions Basics

## Overview

Every file and directory in Linux has permissions that control who can
read, modify, or execute it. Understanding permissions is essential for
protecting data, managing users, and administering Linux systems
securely.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Understand Linux permission concepts
-   Read permission strings from `ls -l`
-   Identify file owners and groups
-   Understand read, write, and execute permissions
-   Recognize file types in permission listings

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Listing files with `ls -l`
-   Creating files and directories

------------------------------------------------------------------------

# Theory

Linux uses a permission system to determine who can access files and
what actions they can perform.

Each file has:

-   An owner
-   A group
-   Permissions for:
    -   Owner
    -   Group
    -   Others

This model helps keep systems secure while allowing collaboration.

------------------------------------------------------------------------

# Viewing Permissions

Use:

``` bash
ls -l
```

Example:

``` text
-rwxr-xr-- 1 hamad developers 2048 Jul 1 10:30 script.sh
```

------------------------------------------------------------------------

# Understanding the Output

``` text
-rwxr-xr--
││  │  │
││  │  └── Others
││  └───── Group
│└──────── Owner
└───────── File Type
```

------------------------------------------------------------------------

# File Types

  Symbol   Meaning
  -------- ------------------
  `-`      Regular file
  `d`      Directory
  `l`      Symbolic link
  `c`      Character device
  `b`      Block device

------------------------------------------------------------------------

# Permission Symbols

  Symbol   Permission   Meaning
  -------- ------------ --------------------------------------
  `r`      Read         View file contents or list directory
  `w`      Write        Modify a file or directory contents
  `x`      Execute      Run a file or enter a directory

------------------------------------------------------------------------

# Permission Groups

Example:

``` text
-rwxr-xr--
```

Breakdown:

``` text
-  rwx  r-x  r--
   │    │    │
   │    │    └── Others
   │    └────── Group
   └─────────── Owner
```

Owner:

``` text
rwx
```

Group:

``` text
r-x
```

Others:

``` text
r--
```

------------------------------------------------------------------------

# Practical Scenario

You download a shell script.

``` bash
ls -l backup.sh
```

Output:

``` text
-rw-r--r-- backup.sh
```

You notice there is no `x` permission, meaning the script cannot be
executed yet.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

touch notes.txt

mkdir projects

ls -l

ls -ld projects

touch script.sh

ls -l script.sh
```

Observe:

-   File type
-   Owner
-   Group
-   Permission bits

------------------------------------------------------------------------

# Challenge

1.  Create two files.
2.  Create one directory.
3.  Display permissions using `ls -l`.
4.  Identify the owner, group, and permissions for each item.
5.  Explain the meaning of each permission string.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand owner, group, and others.
-   [ ] I can identify file types.
-   [ ] I know the meaning of `r`, `w`, and `x`.
-   [ ] I can read the output of `ls -l`.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing file permissions with ownership.
-   Assuming directories use permissions exactly like files.
-   Ignoring the first character that indicates the file type.

------------------------------------------------------------------------

# Best Practices

-   Check permissions before modifying system files.
-   Review ownership when troubleshooting access problems.
-   Use `ls -l` regularly when working with important files.

------------------------------------------------------------------------

# Interview Questions

1.  What do `r`, `w`, and `x` represent?
2.  Who are the owner, group, and others?
3.  What does the first character in `ls -l` indicate?
4.  How can you identify a directory from `ls -l` output?
5.  Why are Linux permissions important?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I interpret a permission string?
-   Can I identify file ownership?
-   Do I understand how permissions improve security?

------------------------------------------------------------------------

# Summary

Linux permissions are the foundation of system security. By
understanding ownership and permission bits, you can safely manage files
and prepare for changing permissions in the next lesson.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                            Capture
  ----------------------------------- ------------------------------
  `51-ls-long-permissions.png`        Output of `ls -l`
  `52-directory-permissions.png`      Output of `ls -ld projects`
  `53-file-permissions.png`           Permissions for `script.sh`
  `54-permission-analysis.png`        Annotated permission string
  `55-permissions-lab-complete.png`   Final terminal after the lab
