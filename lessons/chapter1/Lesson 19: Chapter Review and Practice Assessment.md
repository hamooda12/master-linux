# Chapter 01 --- Lesson 19: Chapter Review and Practice Assessment

## Overview

This lesson reviews every major concept covered in Chapter 01. Its
purpose is to reinforce your understanding and verify that you are ready
to continue to Chapter 02.

------------------------------------------------------------------------

# Learning Objectives

-   Recall the purpose of every command learned
-   Choose the correct command for common Linux tasks
-   Complete practical exercises without notes
-   Identify areas that require additional practice

------------------------------------------------------------------------

# Chapter 01 Command Summary

  Command     Purpose
  ----------- -------------------------------------
  `pwd`       Print the current working directory
  `cd`        Change directories
  `ls`        List directory contents
  `mkdir`     Create directories
  `touch`     Create empty files
  `cp`        Copy files and directories
  `mv`        Move or rename files
  `rm`        Remove files
  `rmdir`     Remove empty directories
  `cat`       Display file contents
  `less`      Read files page by page
  `head`      Show the beginning of a file
  `tail`      Show the end of a file
  `find`      Search for files and directories
  `history`   Show command history
  `man`       Open manual pages
  `chmod`     Change permissions
  `chown`     Change ownership
  `chgrp`     Change group ownership
  `ln`        Create hard links
  `ln -s`     Create symbolic links

------------------------------------------------------------------------

# Knowledge Check

1.  What is the difference between an absolute path and a relative path?
2.  What is an inode?
3.  What is the difference between a hard link and a symbolic link?
4.  What does permission `755` mean?
5.  Why is `chmod 777` discouraged?
6.  What is the purpose of the Sticky Bit?
7.  When would you use SetGID?
8.  What does `find . -type f` return?
9.  What is the difference between `cp` and `mv`?
10. How do you display the manual page for `find`?

------------------------------------------------------------------------

# Practical Assessment

Complete these tasks without referring to previous lessons.

-   Create a project structure with `docs`, `scripts`, `backups`,
    `shared`, and `public`.
-   Create `README.md`, `app.py`, and `deploy.sh`.
-   Create one hard link and one symbolic link to `README.md`.
-   Set `deploy.sh` to `755`.
-   Set `README.md` to `644`.
-   Enable SetGID on `shared`.
-   Enable the Sticky Bit on `public`.
-   Find all Markdown files.
-   Display inode numbers.
-   Verify permissions.

------------------------------------------------------------------------

# Common Mistakes

-   Running destructive commands without checking the current directory.
-   Confusing hard links with copies.
-   Misunderstanding special permissions.

------------------------------------------------------------------------

# Best Practices

-   Verify paths with `pwd`.
-   Check results using `ls -l` and `ls -li`.
-   Read documentation with `man`.
-   Create backups before making important changes.

------------------------------------------------------------------------

# Interview Questions

1.  Explain the Linux filesystem hierarchy.
2.  What is an inode?
3.  Compare hard links and symbolic links.
4.  Explain SetUID, SetGID, and the Sticky Bit.
5.  What does `chmod 644` do?
6.  What does `chmod 755` do?
7.  Why does `/tmp` have the Sticky Bit?
8.  How do you recursively change ownership?
9.  What does `mkdir -p` do?
10. How would you troubleshoot a "Permission denied" error?

------------------------------------------------------------------------

# Self-Assessment

  Topic                 Rating (1--5)
  --------------------- ---------------
  Navigation            
  File Management       
  Viewing Files         
  Searching             
  Links                 
  Permissions           
  Special Permissions   
  Ownership             
  Overall Confidence    

------------------------------------------------------------------------

# Chapter Summary

You have completed the Linux filesystem fundamentals:

-   Filesystem hierarchy
-   Navigation
-   File management
-   Viewing files
-   Searching
-   Wildcards
-   Documentation
-   Hard and symbolic links
-   Standard permissions
-   Special permissions
-   Ownership

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                       Capture
  ------------------------------ -------------------------------------
  `79-final-assessment.png`      Completed assessment tasks
  `80-links-review.png`          Hard and symbolic link verification
  `81-permissions-review.png`    Permission review
  `82-chapter-01-complete.png`   Final completed Chapter 01 terminal
