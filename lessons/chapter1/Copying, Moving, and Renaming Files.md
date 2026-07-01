# Chapter 01 --- Lesson 6: Copying, Moving, and Renaming Files

## Overview

Managing files is one of the most common daily tasks in Linux. In this
lesson, you will learn how to copy files, move them between directories,
and rename both files and directories using the `cp` and `mv` commands.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Copy files with `cp`
-   Copy directories recursively
-   Move files between directories
-   Rename files and directories
-   Understand the difference between copying and moving

------------------------------------------------------------------------

# Prerequisites

-   Navigation (`pwd`, `cd`)
-   Listing files (`ls`)
-   Creating files and directories (`mkdir`, `touch`)

------------------------------------------------------------------------

# Theory

When working on Linux systems you often need to:

-   Back up important files
-   Organize project folders
-   Rename files
-   Move files into different locations

Linux provides simple but powerful commands for these tasks.

------------------------------------------------------------------------

# Command: `cp`

`cp` stands for **Copy**.

## Syntax

``` bash
cp source destination
```

## Example

``` bash
cp notes.txt notes_backup.txt
```

Verify:

``` bash
ls
```

------------------------------------------------------------------------

## Copy a File to Another Directory

``` bash
cp notes.txt backups/
```

------------------------------------------------------------------------

## Copy Multiple Files

``` bash
cp file1.txt file2.txt backups/
```

------------------------------------------------------------------------

## Copy a Directory

Use the recursive option.

``` bash
cp -r project project_backup
```

Without `-r`, Linux cannot copy directories.

------------------------------------------------------------------------

# Command: `mv`

`mv` stands for **Move**.

## Move a File

``` bash
mv notes.txt Documents/
```

------------------------------------------------------------------------

## Rename a File

``` bash
mv notes.txt meeting-notes.txt
```

------------------------------------------------------------------------

## Rename a Directory

``` bash
mv project project-v2
```

The same command is used for both moving and renaming.

------------------------------------------------------------------------

# Practical Scenario

Create a backup of your project before making major changes.

``` bash
mkdir backups

cp -r project backups/

mv project current-project
```

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

touch report.txt

mkdir backups

cp report.txt backups/

mv report.txt final-report.txt

mkdir project

touch project/main.py

cp -r project project-copy

ls -R
```

------------------------------------------------------------------------

# Challenge

1.  Create a directory called `photos`.
2.  Create two empty image files.
3.  Copy both files into a new directory named `backup`.
4.  Rename one image.
5.  Display the final directory structure.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can copy a file.
-   [ ] I can copy a directory.
-   [ ] I can move a file.
-   [ ] I can rename files.
-   [ ] I understand the difference between `cp` and `mv`.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting `-r` when copying directories.
-   Accidentally overwriting existing files.
-   Confusing moving with copying.

------------------------------------------------------------------------

# Best Practices

-   Create backups before editing important files.
-   Verify your destination before moving files.
-   Use descriptive filenames.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `cp` and `mv`?
2.  Why is `cp -r` required?
3.  How do you rename a file in Linux?
4.  Does `mv` create a copy?
5.  When should you create backups?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I safely organize files?
-   Do I know when to copy instead of move?
-   Can I rename files confidently?

------------------------------------------------------------------------

# Summary

The `cp` and `mv` commands are essential for file management. Together
they allow you to organize projects, create backups, and maintain a
clean filesystem.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                       Capture
  ------------------------------ ------------------------------------
  `17-copy-file.png`             Copying a file with `cp`
  `18-copy-directory.png`        Using `cp -r`
  `19-move-file.png`             Moving a file to another directory
  `20-rename-file.png`           Renaming a file with `mv`
  `21-file-management-lab.png`   Final `ls -R` output after the lab
