# Chapter 01 --- Lesson 9: Finding Files and Directories

## Overview

As Linux systems grow, manually browsing directories becomes
inefficient. The `find` command allows you to quickly locate files and
directories based on their name, type, size, permissions, modification
time, and many other attributes.

Learning `find` is an essential skill for every Linux user, system
administrator, and DevOps engineer.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Search for files by name
-   Search for directories
-   Search using wildcards
-   Search by file type
-   Search by file extension
-   Search within a specific directory

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Creating files and directories
-   Viewing directory contents

------------------------------------------------------------------------

# Theory

Imagine a server containing thousands of files. Searching manually would
be slow and error-prone.

The `find` command scans the filesystem and returns files or directories
that match your search criteria.

------------------------------------------------------------------------

# Command: `find`

## Syntax

``` bash
find <starting-directory> <expression>
```

------------------------------------------------------------------------

## Find a File by Name

``` bash
find . -name "README.md"
```

`.` means "start searching from the current directory."

------------------------------------------------------------------------

## Search from the Root Directory

``` bash
find / -name "sshd_config"
```

> You may need `sudo` when searching system directories.

------------------------------------------------------------------------

## Find Directories Only

``` bash
find . -type d
```

------------------------------------------------------------------------

## Find Files Only

``` bash
find . -type f
```

------------------------------------------------------------------------

## Search Using Wildcards

Find all Markdown files:

``` bash
find . -name "*.md"
```

Find all Python files:

``` bash
find . -name "*.py"
```

------------------------------------------------------------------------

## Case-Insensitive Search

``` bash
find . -iname "readme.md"
```

`-iname` ignores letter case.

------------------------------------------------------------------------

# Practical Scenario

You remember creating a project README but forgot where you saved it.

``` bash
cd ~/linux-practice

find . -name "README.md"
```

The command quickly returns its location.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice

mkdir -p search-demo/docs

touch search-demo/README.md

touch search-demo/docs/manual.md

touch search-demo/script.py

find . -name "README.md"

find . -name "*.md"

find . -type d

find . -type f
```

------------------------------------------------------------------------

# Challenge

1.  Create three directories.
2.  Create five files with different extensions.
3.  Find all Markdown files.
4.  Find all directories.
5.  Find one file using a case-insensitive search.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can search for files by name.
-   [ ] I can search for directories.
-   [ ] I can use wildcards.
-   [ ] I understand `-type f` and `-type d`.
-   [ ] I can perform a case-insensitive search.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to quote wildcard patterns like `"*.md"`.
-   Starting the search from the wrong directory.
-   Confusing `-name` with `-iname`.

------------------------------------------------------------------------

# Best Practices

-   Start searches from the smallest directory possible.
-   Use wildcards to reduce typing.
-   Prefer `-iname` when you are unsure about capitalization.
-   Verify search results before modifying files.

------------------------------------------------------------------------

# Interview Questions

1.  What is the purpose of the `find` command?
2.  What does `.` represent in a `find` command?
3.  What is the difference between `-name` and `-iname`?
4.  What does `-type f` do?
5.  How do you find every Markdown file?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I quickly locate files without browsing manually?
-   Do I know how to search only for directories?
-   Can I use wildcard patterns effectively?

------------------------------------------------------------------------

# Summary

The `find` command is one of Linux's most powerful search utilities.
Mastering it allows you to locate files and directories quickly, making
filesystem management far more efficient.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                     Capture
  ---------------------------- ----------------------------------------
  `31-find-by-name.png`        Finding a file by name
  `32-find-markdown.png`       Searching for `*.md` files
  `33-find-directories.png`    Output of `find . -type d`
  `34-find-files.png`          Output of `find . -type f`
  `35-find-lab-complete.png`   Final results after completing the lab
