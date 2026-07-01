# Chapter 01 --- Lesson 14: Changing Permissions with `chmod`

## Overview

In the previous lesson, you learned how to read Linux file permissions.
In this lesson, you will learn how to change those permissions using the
`chmod` command.

Changing permissions is an essential Linux administration skill. It
controls who can read, modify, or execute files and directories.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Change file permissions using `chmod`
-   Understand symbolic permission mode
-   Understand numeric permission mode
-   Add and remove execute permission
-   Apply permissions safely to files and directories

------------------------------------------------------------------------

# Prerequisites

-   Understanding `ls -l`
-   Basic Linux permissions
-   File ownership concepts
-   Creating files and directories

------------------------------------------------------------------------

# Theory

Linux permissions control access for three groups:

``` text
Owner   Group   Others
```

Each group can have:

``` text
r = read
w = write
x = execute
```

Example:

``` text
-rw-r--r--
```

This means:

``` text
Owner:  read + write
Group:  read only
Others: read only
```

The `chmod` command changes these permissions.

------------------------------------------------------------------------

# Command: `chmod`

`chmod` stands for **Change Mode**.

## Syntax

``` bash
chmod permissions filename
```

------------------------------------------------------------------------

# Symbolic Mode

Symbolic mode uses letters to change permissions.

## Permission targets

  Symbol   Meaning
  -------- --------------
  `u`      User / owner
  `g`      Group
  `o`      Others
  `a`      All users

## Permission operators

  Symbol   Meaning
  -------- ----------------------
  `+`      Add permission
  `-`      Remove permission
  `=`      Set exact permission

## Permission types

  Symbol   Meaning
  -------- ---------
  `r`      Read
  `w`      Write
  `x`      Execute

------------------------------------------------------------------------

## Add Execute Permission

``` bash
chmod u+x script.sh
```

This gives the owner permission to execute the file.

Check:

``` bash
ls -l script.sh
```

Example:

``` text
-rwxr--r-- 1 hamad hamad 0 Jul 1 12:00 script.sh
```

------------------------------------------------------------------------

## Remove Write Permission

``` bash
chmod g-w notes.txt
```

This removes write permission from the group.

------------------------------------------------------------------------

## Give Everyone Read Permission

``` bash
chmod a+r notes.txt
```

------------------------------------------------------------------------

## Set Exact Permissions

``` bash
chmod u=rw,g=r,o=r notes.txt
```

This sets:

``` text
Owner: read + write
Group: read
Others: read
```

------------------------------------------------------------------------

# Numeric Mode

Numeric mode uses numbers to represent permissions.

  Permission      Value
  --------------- -------
  Read (`r`)      4
  Write (`w`)     2
  Execute (`x`)   1

Add the values together.

  Permission Set   Numeric Value
  ---------------- ---------------
  `r--`            4
  `rw-`            6
  `r-x`            5
  `rwx`            7

------------------------------------------------------------------------

## Common Numeric Permissions

  Mode    Meaning
  ------- -------------------------------------------------------
  `644`   Owner can read/write, others can read
  `755`   Owner can read/write/execute, others can read/execute
  `600`   Owner can read/write only
  `700`   Owner has full access only

------------------------------------------------------------------------

## Example: `chmod 644`

``` bash
chmod 644 notes.txt
```

Result:

``` text
-rw-r--r--
```

------------------------------------------------------------------------

## Example: `chmod 755`

``` bash
chmod 755 script.sh
```

Result:

``` text
-rwxr-xr-x
```

This is common for executable scripts.

------------------------------------------------------------------------

# Practical Scenario

You create a shell script:

``` bash
touch backup.sh
```

At first, it cannot be executed:

``` bash
ls -l backup.sh
```

Example:

``` text
-rw-r--r-- backup.sh
```

Add execute permission:

``` bash
chmod u+x backup.sh
```

Now the owner can run it.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

touch script.sh

ls -l script.sh

chmod u+x script.sh

ls -l script.sh

chmod 644 script.sh

ls -l script.sh

chmod 755 script.sh

ls -l script.sh

touch private.txt

chmod 600 private.txt

ls -l private.txt
```

------------------------------------------------------------------------

# Challenge

1.  Create a file named `deploy.sh`.
2.  Give the owner execute permission using symbolic mode.
3.  Change the file permission to `755`.
4.  Create a file named `secret.txt`.
5.  Set `secret.txt` permission to `600`.
6.  Display the final permissions using `ls -l`.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can use `chmod`.
-   [ ] I understand symbolic mode.
-   [ ] I understand numeric mode.
-   [ ] I know what `755` means.
-   [ ] I know what `644` means.
-   [ ] I can add execute permission to a script.

------------------------------------------------------------------------

# Common Mistakes

-   Giving execute permission to files that should not be executable.
-   Using `chmod 777` without understanding the security risk.
-   Forgetting to check permissions after changing them.
-   Confusing symbolic mode with numeric mode.

------------------------------------------------------------------------

# Best Practices

-   Use `chmod 755` for scripts that should be executable.
-   Use `chmod 644` for normal readable files.
-   Use `chmod 600` for private files.
-   Avoid `chmod 777` unless you fully understand the risk.
-   Always verify changes with `ls -l`.

------------------------------------------------------------------------

# Interview Questions

1.  What does `chmod` do?
2.  What is the difference between symbolic mode and numeric mode?
3.  What does `chmod u+x file.sh` mean?
4.  What does permission `755` mean?
5.  Why is `chmod 777` dangerous?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I change file permissions confidently?
-   Do I understand when a file needs execute permission?
-   Can I explain numeric permissions to another beginner?

------------------------------------------------------------------------

# Summary

The `chmod` command is used to change file and directory permissions. By
understanding symbolic and numeric permission modes, you can control
access safely and prepare for real Linux administration tasks.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -----------------------------------------------------------------------
  Filename                            Capture
  ----------------------------------- -----------------------------------
  `56-chmod-before.png`               Permission output before using
                                      `chmod`

  `57-chmod-symbolic.png`             Adding execute permission with
                                      `chmod u+x`

  `58-chmod-644.png`                  Applying `chmod 644`

  `59-chmod-755.png`                  Applying `chmod 755`

  `60-chmod-private-file.png`         Applying `chmod 600` to a private
                                      file

  `61-chmod-lab-complete.png`         Final terminal after completing the
                                      lab
  -----------------------------------------------------------------------
