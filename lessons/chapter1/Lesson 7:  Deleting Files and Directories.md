# Chapter 01 --- Lesson 7: Deleting Files and Directories

## Overview

Creating and managing files is only part of filesystem administration.
You must also know how to safely remove files and directories when they
are no longer needed.

In Linux, deleting is a permanent operation by default. Unlike many
graphical desktop environments, the terminal does not send deleted files
to a recycle bin.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Delete files using `rm`
-   Delete empty directories using `rmdir`
-   Delete directories recursively with `rm -r`
-   Force deletion with `rm -f`
-   Understand safe file deletion practices

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Using `ls`
-   Creating files and directories
-   Copying and moving files

------------------------------------------------------------------------

# Theory

Linux gives you full control over your filesystem. That power comes with
responsibility.

A mistaken deletion can remove important files instantly, so
understanding deletion commands is essential before using them on
production systems.

------------------------------------------------------------------------

# Command: `rm`

`rm` stands for **Remove**.

## Syntax

``` bash
rm filename
```

## Example

``` bash
touch notes.txt
ls
rm notes.txt
ls
```

The file is permanently removed.

------------------------------------------------------------------------

## Remove Multiple Files

``` bash
rm file1.txt file2.txt file3.txt
```

------------------------------------------------------------------------

## Force Removal

``` bash
rm -f notes.txt
```

The `-f` option forces deletion without prompting if permissions allow.

------------------------------------------------------------------------

## Remove a Directory Recursively

``` bash
rm -r project
```

This removes the directory and everything inside it.

------------------------------------------------------------------------

## Force Recursive Removal

``` bash
rm -rf project
```

> **Warning:** `rm -rf` is one of the most powerful and dangerous Linux
> commands. Always verify the target before pressing Enter.

------------------------------------------------------------------------

# Command: `rmdir`

`rmdir` removes **empty** directories only.

## Syntax

``` bash
rmdir directory_name
```

## Example

``` bash
mkdir empty-folder
rmdir empty-folder
```

If the directory contains files, the command fails.

------------------------------------------------------------------------

# Practical Scenario

You finished testing a temporary project and want to clean your
workspace.

``` bash
cd ~/linux-practice/practice

ls

rm final-report.txt

rm -r project-copy

ls
```

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

mkdir temp

touch temp/file1.txt

touch temp/file2.txt

ls -R

rm temp/file1.txt

rm temp/file2.txt

rmdir temp

mkdir demo

touch demo/test.txt

rm -r demo

ls
```

------------------------------------------------------------------------

# Challenge

1.  Create a directory called `sandbox`.
2.  Create three files inside it.
3.  Delete one file.
4.  Delete the remaining directory and its contents.
5.  Verify that nothing remains.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can delete a file.
-   [ ] I can remove an empty directory.
-   [ ] I can delete a directory recursively.
-   [ ] I understand the purpose of `rm -f`.
-   [ ] I understand why `rm -rf` must be used carefully.

------------------------------------------------------------------------

# Common Mistakes

-   Using `rmdir` on a non-empty directory.
-   Running `rm -rf` without checking the path.
-   Deleting the wrong file because the current directory was not
    verified.

------------------------------------------------------------------------

# Best Practices

-   Run `pwd` before deleting important files.
-   Use `ls` to verify what will be removed.
-   Prefer deleting individual files before removing entire directories.
-   Avoid using `rm -rf` unless it is truly necessary.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `rm` and `rmdir`?
2.  Why does `rmdir` sometimes fail?
3.  What does the `-r` option do?
4.  What is the purpose of the `-f` option?
5.  Why is `rm -rf` considered dangerous?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I safely remove files?
-   Do I know when to use `rmdir` instead of `rm -r`?
-   Do I understand the risks of recursive deletion?

------------------------------------------------------------------------

# Summary

The `rm` and `rmdir` commands are essential for maintaining a clean
Linux filesystem. Used correctly, they help remove unnecessary files and
directories efficiently while reinforcing safe system administration
practices.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                 Capture
  ------------------------ -----------------------------------------
  `22-rm-file.png`         Removing a single file
  `23-rmdir-empty.png`     Removing an empty directory
  `24-rm-recursive.png`    Using `rm -r` on a directory
  `25-final-cleanup.png`   Final terminal after completing the lab
