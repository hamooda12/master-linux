# Chapter 01 --- Lesson 4: Listing Files and Directories

## Overview

After learning how to move around the Linux filesystem, the next
essential skill is inspecting the contents of directories. The `ls`
command allows you to view files, directories, and important information
about them.

Understanding how to use `ls` efficiently is a fundamental Linux skill
used by developers, system administrators, and DevOps engineers every
day.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   List files and directories
-   Display hidden files
-   Read detailed file information
-   Display human-readable file sizes
-   Recursively list directories
-   Sort files by modification time

------------------------------------------------------------------------

# Prerequisites

-   Basic Linux filesystem knowledge
-   Navigation using `pwd` and `cd`

------------------------------------------------------------------------

# Theory

Everything in Linux is organized into directories. Before opening,
editing, copying, or deleting files, you usually inspect the directory
first.

The `ls` command provides a snapshot of a directory's contents.

------------------------------------------------------------------------

# Command: `ls`

## Syntax

``` bash
ls
```

## Example

``` bash
$ ls
Documents  Downloads  Pictures  Projects
```

By default, `ls` displays the visible contents of the current directory.

------------------------------------------------------------------------

# Common Options

## `ls -l`

Displays files in the long listing format.

``` bash
ls -l
```

Example:

``` text
drwxr-xr-x 2 hamad hamad 4096 Jul 1 10:15 Documents
-rw-r--r-- 1 hamad hamad  842 Jul 1 09:20 notes.txt
```

### Understanding the Output

  -----------------------------------------------------------------------
  Field                               Description
  ----------------------------------- -----------------------------------
  `d` / `-`                           File type (`d` = directory, `-` =
                                      regular file)

  `rwxr-xr-x`                         Permissions

  `2`                                 Number of hard links

  `hamad`                             Owner

  `hamad`                             Group

  `4096`                              File size (bytes for files,
                                      directory metadata for directories)

  `Jul 1 10:15`                       Last modification time

  `Documents`                         File or directory name
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## `ls -a`

Shows hidden files.

``` bash
ls -a
```

Example:

``` text
.
..
.bashrc
.profile
.config
Documents
Downloads
```

Hidden files begin with a dot (`.`).

------------------------------------------------------------------------

## `ls -la`

Combines long listing and hidden files.

``` bash
ls -la
```

------------------------------------------------------------------------

## `ls -lh`

Displays human-readable file sizes.

``` bash
ls -lh
```

Example:

``` text
-rw-r--r-- 1 hamad hamad 2.1K Jul 1 09:20 notes.txt
```

------------------------------------------------------------------------

## `ls -R`

Lists directories recursively.

``` bash
ls -R
```

------------------------------------------------------------------------

## `ls -t`

Sorts files by modification time, newest first.

``` bash
ls -t
```

------------------------------------------------------------------------

# Practical Scenario

You log into a Linux server and need to inspect a project directory
before making changes.

``` bash
cd ~/linux-practice
ls -la
```

You immediately identify hidden configuration files, directories, and
recently modified files.

------------------------------------------------------------------------

# Hands-on Lab

Run the following commands:

``` bash
pwd

ls

ls -l

ls -a

ls -la

ls -lh

ls -R

ls -t
```

Observe how the output changes with each option.

------------------------------------------------------------------------

# Challenge

1.  Navigate to your `linux-practice` directory.
2.  Display all hidden files.
3.  Show the long listing format.
4.  Display human-readable file sizes.
5.  List files sorted by modification time.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can use `ls`.
-   [ ] I can display hidden files.
-   [ ] I can explain the columns in `ls -l`.
-   [ ] I can display human-readable sizes.
-   [ ] I can recursively list directories.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting that `ls` does not show hidden files by default.
-   Confusing `ls -l` with `ls -la`.
-   Assuming `ls -R` only lists the current directory.

------------------------------------------------------------------------

# Best Practices

-   Use `ls -la` when troubleshooting configuration issues.
-   Use `ls -lh` when checking file sizes.
-   Review directory contents before modifying files.

------------------------------------------------------------------------

# Interview Questions

1.  What does the `ls` command do?
2.  What is the difference between `ls` and `ls -a`?
3.  Why is `ls -lh` useful?
4.  What information does `ls -l` display?
5.  What does `ls -R` do?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I inspect any directory confidently?
-   Can I identify hidden files?
-   Can I explain the output of `ls -l`?

------------------------------------------------------------------------

# Summary

The `ls` command is one of the most frequently used Linux commands.
Mastering its options allows you to inspect files efficiently and
understand the structure of any directory.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                     Capture
  ---------------------------- --------------------
  `07-ls-basic.png`            Output of `ls`
  `08-ls-long-format.png`      Output of `ls -l`
  `09-ls-hidden-files.png`     Output of `ls -a`
  `10-ls-long-hidden.png`      Output of `ls -la`
  `11-ls-human-readable.png`   Output of `ls -lh`
  `12-ls-recursive.png`        Output of `ls -R`
