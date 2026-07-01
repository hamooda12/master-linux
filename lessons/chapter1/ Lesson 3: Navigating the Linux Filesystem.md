# Chapter 01 --- Lesson 3: Navigating the Linux Filesystem

## Overview

Understanding the Linux filesystem is only the first step. To work
efficiently in Linux, you must know how to move through directories,
inspect your current location, and understand the difference between
**absolute** and **relative** paths.

Navigation is one of the most frequently used skills in Linux. Whether
you're managing servers, deploying applications, writing scripts, or
troubleshooting systems, you'll constantly move between directories.

By the end of this lesson, you will be able to navigate the Linux
filesystem confidently using the terminal.

------------------------------------------------------------------------

# Learning Objectives

-   Understand the Current Working Directory (CWD)
-   Display your current location
-   Move between directories
-   Return to your home directory
-   Navigate using absolute and relative paths
-   Understand `.`, `..`, `~`, and `/`

------------------------------------------------------------------------

# Prerequisites

-   Linux filesystem hierarchy
-   Root directory (`/`)
-   Common Linux directories

------------------------------------------------------------------------

# Theory

Every command in Linux runs from a location called the **Current Working
Directory (CWD)**.

``` text
                    Linux Filesystem

                        /
                        │
         ┌──────────────┼──────────────┐
         │              │              │
      home            etc            var
         │
      hamad
         │
    linux-practice
         │
      projects

Current Working Directory
             ▲
             │
        You are here
```

Knowing your current location helps prevent mistakes when creating,
deleting, or modifying files.

------------------------------------------------------------------------

# Command: `pwd`

`pwd` stands for **Print Working Directory**.

## Syntax

``` bash
pwd
```

## Example

``` bash
$ pwd
/home/hamad/linux-practice
```

Use `pwd` whenever you want to verify your current location.

------------------------------------------------------------------------

# Command: `cd`

`cd` means **Change Directory**.

## Enter a directory

``` bash
cd Documents
```

## Go to your home directory

``` bash
cd
```

or

``` bash
cd ~
```

## Go to the parent directory

``` bash
cd ..
```

## Go to the root directory

``` bash
cd /
```

Always verify your location afterwards:

``` bash
pwd
```

------------------------------------------------------------------------

# Absolute vs Relative Paths

## Absolute Path

Always starts with `/`.

Example:

``` text
/home/hamad/Documents/report.txt
```

Works no matter where you currently are.

## Relative Path

Starts from your current location.

Example:

``` bash
cd Documents
```

If your current directory is:

``` text
/home/hamad
```

Linux interprets it as:

``` text
/home/hamad/Documents
```

### Comparison

  Absolute Path         Relative Path
  --------------------- -------------------------------
  Starts with `/`       Starts from current directory
  Works from anywhere   Depends on current location
  Longer                Shorter

------------------------------------------------------------------------

# Special Directory Symbols

  Symbol   Meaning
  -------- -------------------
  `.`      Current directory
  `..`     Parent directory
  `~`      Home directory
  `/`      Root directory

------------------------------------------------------------------------

# Practical Scenario

A system administrator needs to inspect log files.

``` bash
pwd
cd /
cd var
cd log
pwd
```

Expected output:

``` text
/var/log
```

------------------------------------------------------------------------

# Hands-on Lab

``` bash
pwd

cd /

pwd

cd home

pwd

cd ~

pwd

cd ..

pwd

cd /

pwd
```

Observe how the current working directory changes after each command.

------------------------------------------------------------------------

# Challenge

1.  Go to the root directory.
2.  Navigate to your home directory.
3.  Enter the `Documents` directory (if it exists).
4.  Return to your home directory.
5.  Print your current location.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand the Current Working Directory.
-   [ ] I can use `pwd`.
-   [ ] I can use `cd`.
-   [ ] I understand absolute paths.
-   [ ] I understand relative paths.
-   [ ] I know the purpose of `.`, `..`, `~`, and `/`.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to check the current directory with `pwd`
-   Mixing absolute and relative paths
-   Assuming `cd` modifies files (it only changes directories)

------------------------------------------------------------------------

# Best Practices

-   Verify your location before important operations.
-   Prefer absolute paths in scripts.
-   Use relative paths for everyday navigation.
-   Use **Tab** for auto-completion.

------------------------------------------------------------------------

# Interview Questions

1.  What is the Current Working Directory?
2.  What does `pwd` stand for?
3.  What is the difference between an absolute path and a relative path?
4.  What does `cd ..` do?
5.  What does `~` represent?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I move confidently around the filesystem?
-   Can I explain absolute and relative paths?
-   Do I understand where commands execute?

------------------------------------------------------------------------

# Summary

Navigation is one of the core Linux skills. Mastering `pwd`, `cd`, and
filesystem paths provides the foundation for every future Linux task.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                           Description
  ---------------------------------- ---------------------------------
  `01-pwd-home.png`                  `pwd` from the home directory
  `02-root-directory.png`            `cd /` followed by `pwd`
  `03-home-navigation.png`           Navigating to `/home`
  `04-parent-directory.png`          Demonstrating `cd ..`
  `05-absolute-vs-relative.png`      Showing both navigation methods
  `06-navigation-lab-complete.png`   Completed hands-on lab
