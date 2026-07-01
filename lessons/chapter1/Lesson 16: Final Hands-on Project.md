# Chapter 01 --- Lesson 16: Final Hands-on Project

## Overview

This lesson combines everything you have learned throughout Chapter 01
into a single practical project. Instead of practicing commands
individually, you will complete a realistic Linux filesystem management
task similar to those performed by developers and junior Linux
administrators.

------------------------------------------------------------------------

# Learning Objectives

After completing this project, you will be able to:

-   Navigate the filesystem efficiently
-   Create and organize directories
-   Create, copy, move, and rename files
-   Delete files and directories safely
-   Search for files
-   Use wildcards
-   View file contents
-   Inspect and modify permissions
-   Inspect and modify ownership (if permitted)

------------------------------------------------------------------------

# Project Scenario

You have joined a software company as a Junior Linux Administrator.

Your manager asks you to prepare a project workspace for a new
application.

The workspace should have the following structure:

``` text
company-project/
├── backups/
├── config/
├── docs/
├── logs/
├── scripts/
│   └── deploy.sh
├── src/
│   ├── app.py
│   └── utils.py
└── README.md
```

------------------------------------------------------------------------

# Tasks

## Part 1 --- Create the Structure

Create the complete directory tree and all required files.

------------------------------------------------------------------------

## Part 2 --- Verify the Structure

Use appropriate commands to verify:

-   Current location
-   Directory contents
-   Recursive directory tree

------------------------------------------------------------------------

## Part 3 --- Organize Files

1.  Create a backup of `README.md`.
2.  Rename `deploy.sh` to `deploy-production.sh`.
3.  Move the backup into the `backups` directory.

------------------------------------------------------------------------

## Part 4 --- View Files

Use:

-   `cat`
-   `head`
-   `tail`
-   `less`

to inspect `README.md`.

------------------------------------------------------------------------

## Part 5 --- Search

Find:

-   Every Markdown file
-   Every Python file
-   Every directory
-   Every regular file

------------------------------------------------------------------------

## Part 6 --- Wildcards

Practice:

``` bash
ls *.md
ls *.py
ls src/*.py
```

------------------------------------------------------------------------

## Part 7 --- Permissions

1.  Make `deploy-production.sh` executable.
2.  Set `README.md` to `644`.
3.  Create `secret.txt`.
4.  Set `secret.txt` to `600`.

------------------------------------------------------------------------

## Part 8 --- Ownership (Optional)

If you have administrative privileges:

``` bash
sudo chown root README.md
sudo chgrp root README.md
```

Verify using:

``` bash
ls -l
```

------------------------------------------------------------------------

# Final Verification Checklist

-   [ ] Filesystem navigation
-   [ ] Directory creation
-   [ ] File creation
-   [ ] Copying files
-   [ ] Moving files
-   [ ] Renaming files
-   [ ] Deleting files
-   [ ] Viewing file contents
-   [ ] Searching with `find`
-   [ ] Using wildcards
-   [ ] Reading permissions
-   [ ] Changing permissions
-   [ ] Understanding ownership

------------------------------------------------------------------------

# Self-Assessment

Rate yourself from **1--5**:

  Skill             Rating
  ----------------- --------
  Navigation        
  File Management   
  Search            
  Permissions       
  Documentation     
  Confidence        

------------------------------------------------------------------------

# Challenge

Without looking at previous lessons, rebuild the entire project from
scratch in a new directory.

If you can complete the project successfully, you have mastered the
fundamentals of Chapter 01.

------------------------------------------------------------------------

# Summary

Congratulations!

You have completed the practical component of Chapter 01. You can now
navigate the Linux filesystem, manage files and directories, inspect
documentation, search efficiently, and work with permissions and
ownership---core skills expected from every Linux user.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -----------------------------------------------------------------------
  Filename                            Capture
  ----------------------------------- -----------------------------------
  `67-project-structure.png`          Final project tree

  `68-file-management.png`            Copying, moving, and renaming files

  `69-find-results.png`               Results of `find` commands

  `70-permissions.png`                Final permission settings

  `71-final-project-complete.png`     Final terminal showing the
                                      completed project
  -----------------------------------------------------------------------
