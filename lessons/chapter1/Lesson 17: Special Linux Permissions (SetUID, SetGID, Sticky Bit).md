# Chapter 01 --- Lesson 17: Special Linux Permissions (SetUID, SetGID, Sticky Bit)

## Overview

Beyond the standard read (`r`), write (`w`), and execute (`x`)
permissions, Linux provides three **special permissions** that change
how files and directories behave.

These permissions are widely used on multi-user systems to improve
security and collaboration.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Understand SetUID, SetGID, and the Sticky Bit
-   Identify special permissions using `ls -l`
-   Apply special permissions using symbolic and numeric modes
-   Understand common real-world use cases
-   Recognize potential security risks

------------------------------------------------------------------------

# Prerequisites

-   Linux file permissions
-   `chmod`
-   File ownership

------------------------------------------------------------------------

# Theory

Normal permissions answer the question:

> "Who can read, write, or execute this file?"

Special permissions answer questions like:

-   Which user's privileges should a program use?
-   Which group should new files belong to?
-   Who is allowed to delete files in a shared directory?

------------------------------------------------------------------------

# Viewing Special Permissions

``` bash
ls -l
```

Example:

``` text
-rwsr-xr-x
drwxrwsr-x
drwxrwxrwt
```

The letters `s` and `t` indicate special permissions.

------------------------------------------------------------------------

# SetUID (Set User ID)

When SetUID is applied to an **executable file**, it runs with the
permissions of the file owner instead of the user executing it.

Example:

``` text
-rwsr-xr-x
```

The `s` replaces the owner's execute bit.

## Apply SetUID

Symbolic mode:

``` bash
chmod u+s program
```

Numeric mode:

``` bash
chmod 4755 program
```

### Real-world Example

The `passwd` program allows normal users to change their passwords
because it runs with elevated privileges.

------------------------------------------------------------------------

# SetGID (Set Group ID)

On an executable file, SetGID causes the program to run with the file's
group permissions.

On a **directory**, every new file created inside inherits the
directory's group.

Example:

``` text
drwxrwsr-x
```

## Apply SetGID

Symbolic mode:

``` bash
chmod g+s shared
```

Numeric mode:

``` bash
chmod 2755 shared
```

### Real-world Example

A shared development directory where every new file automatically
belongs to the `developers` group.

------------------------------------------------------------------------

# Sticky Bit

The Sticky Bit is mainly used on directories.

It allows users to create files in a shared directory but prevents them
from deleting files owned by other users.

Example:

``` text
drwxrwxrwt
```

The `t` replaces the execute permission for others.

## Apply the Sticky Bit

Symbolic mode:

``` bash
chmod +t shared
```

Numeric mode:

``` bash
chmod 1777 shared
```

### Real-world Example

The `/tmp` directory:

``` bash
ls -ld /tmp
```

Typical output:

``` text
drwxrwxrwt
```

Every user can create temporary files, but only the file owner (or root)
can delete them.

------------------------------------------------------------------------

# Numeric Values

  Permission     Numeric Value
  ------------ ---------------
  Sticky Bit                 1
  SetGID                     2
  SetUID                     4

Examples:

  Mode     Meaning
  -------- --------------------
  `4755`   SetUID + `755`
  `2755`   SetGID + `755`
  `1777`   Sticky Bit + `777`

------------------------------------------------------------------------

# Practical Scenario

A team shares a project directory.

The administrator enables SetGID so every newly created file belongs to
the same development group.

Another administrator verifies that `/tmp` has the Sticky Bit to prevent
users from deleting each other's files.

------------------------------------------------------------------------

# Hands-on Lab

> Some commands may require `sudo`.

``` bash
mkdir shared

chmod 775 shared

chmod g+s shared

ls -ld shared

mkdir public

chmod 1777 public

ls -ld public

ls -ld /tmp
```

Observe where the `s` and `t` appear.

------------------------------------------------------------------------

# Challenge

1.  Create a directory named `team-share`.
2.  Enable SetGID.
3.  Create another directory named `public-share`.
4.  Enable the Sticky Bit.
5.  Display permissions with `ls -ld`.
6.  Explain the meaning of each permission string.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand SetUID.
-   [ ] I understand SetGID.
-   [ ] I understand the Sticky Bit.
-   [ ] I can identify `s` and `t` in permission strings.
-   [ ] I know common use cases.

------------------------------------------------------------------------

# Common Mistakes

-   Applying SetUID to ordinary scripts without understanding the
    security implications.
-   Assuming the Sticky Bit prevents users from reading files.
-   Confusing SetGID on files with SetGID on directories.

------------------------------------------------------------------------

# Best Practices

-   Use SetUID sparingly.
-   Use SetGID for shared project directories.
-   Use the Sticky Bit on world-writable shared directories.
-   Verify changes with `ls -l` or `ls -ld`.

------------------------------------------------------------------------

# Interview Questions

1.  What is SetUID?
2.  What is the difference between SetGID on a file and a directory?
3.  Why does `/tmp` use the Sticky Bit?
4.  What do the numbers `4755`, `2755`, and `1777` represent?
5.  How can you identify special permissions in `ls -l` output?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I recognize special permissions in a permission string?
-   Do I understand when each special permission should be used?
-   Can I explain why they exist?

------------------------------------------------------------------------

# Summary

SetUID, SetGID, and the Sticky Bit extend the standard Linux permission
model. They are essential concepts for system administration and secure
multi-user environments.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -------------------------------------------------------------------------------
  Filename                                    Capture
  ------------------------------------------- -----------------------------------
  `67-setgid-directory.png`                   Directory with SetGID enabled

  `68-sticky-bit-directory.png`               Directory with Sticky Bit enabled

  `69-tmp-sticky-bit.png`                     Output of `ls -ld /tmp`

  `70-special-permissions-summary.png`        Comparing normal and special
                                              permissions

  `71-special-permissions-lab-complete.png`   Final terminal after completing the
                                              lab
  -------------------------------------------------------------------------------
