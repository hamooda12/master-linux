# Chapter 01 --- Lesson 5: Creating Files and Directories

## Overview

Creating files and directories is one of the first practical skills
every Linux user learns. Whether you are writing code, organizing
projects, or managing servers, you will constantly create new files and
folders.

This lesson introduces the commands used to create directories and empty
files safely and efficiently.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Create directories with `mkdir`
-   Create nested directories
-   Create empty files with `touch`
-   Create multiple files at once
-   Understand the difference between files and directories

------------------------------------------------------------------------

# Prerequisites

-   Linux filesystem basics
-   Navigation using `pwd` and `cd`
-   Listing directory contents with `ls`

------------------------------------------------------------------------

# Theory

Directories organize information, while files store information.

A well-organized Linux system depends on creating a logical directory
structure before adding files.

------------------------------------------------------------------------

# Command: `mkdir`

`mkdir` stands for **Make Directory**.

## Syntax

``` bash
mkdir directory_name
```

## Example

``` bash
mkdir projects
```

Verify it was created:

``` bash
ls
```

------------------------------------------------------------------------

## Create Multiple Directories

``` bash
mkdir docs scripts backups
```

------------------------------------------------------------------------

## Create Nested Directories

Use the `-p` option.

``` bash
mkdir -p projects/web/frontend
```

This creates every missing directory automatically.

------------------------------------------------------------------------

# Command: `touch`

`touch` creates an empty file if it does not already exist.

## Syntax

``` bash
touch filename
```

## Example

``` bash
touch notes.txt
```

Verify:

``` bash
ls
```

------------------------------------------------------------------------

## Create Multiple Files

``` bash
touch app.py README.md config.yml
```

------------------------------------------------------------------------

## Update File Timestamp

If the file already exists, `touch` updates its modification timestamp.

``` bash
touch README.md
```

------------------------------------------------------------------------

# Practical Scenario

You are starting a new software project.

``` bash
mkdir -p web-app/src
mkdir docs
touch README.md
touch web-app/src/main.py
```

Your project structure becomes:

``` text
.
├── docs
├── README.md
└── web-app
    └── src
        └── main.py
```

------------------------------------------------------------------------

# Hands-on Lab

Execute the following:

``` bash
cd ~/linux-practice

mkdir practice

cd practice

mkdir -p projects/project1

mkdir backups

touch notes.txt

touch todo.txt

touch projects/project1/main.py

ls -R
```

------------------------------------------------------------------------

# Challenge

1.  Create a directory named `training`.
2.  Inside it, create `lesson1` and `lesson2`.
3.  Create `README.md` inside `training`.
4.  Display the complete structure.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can create a directory.
-   [ ] I can create nested directories.
-   [ ] I can create an empty file.
-   [ ] I can create multiple files.
-   [ ] I understand the purpose of `mkdir -p`.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting the `-p` option when creating nested directories.
-   Using spaces in file names without quotes.
-   Confusing files with directories.

------------------------------------------------------------------------

# Best Practices

-   Use meaningful directory names.
-   Group related files together.
-   Prefer lowercase names with hyphens when appropriate.
-   Create the directory structure before adding project files.

------------------------------------------------------------------------

# Interview Questions

1.  What is the purpose of `mkdir`?
2.  What does `mkdir -p` do?
3.  What is the purpose of `touch`?
4.  Can `touch` modify an existing file?
5.  What is the difference between a file and a directory?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I build a project directory from scratch?
-   Do I understand when to use `mkdir` and `touch`?
-   Can I organize files logically?

------------------------------------------------------------------------

# Summary

The `mkdir` and `touch` commands form the foundation of filesystem
management. Mastering them allows you to build organized project
structures and prepare files for future work.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                     Capture
  ---------------------------- ---------------------------------
  `13-mkdir-basic.png`         Creating a single directory
  `14-mkdir-nested.png`        Using `mkdir -p`
  `15-touch-files.png`         Creating multiple files
  `16-project-structure.png`   Output of `ls -R` after the lab
