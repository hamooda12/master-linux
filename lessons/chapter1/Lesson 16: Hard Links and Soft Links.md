# Chapter 01 --- Lesson 16: Hard Links and Soft Links

## Overview

Linux files are identified internally by **inodes**, not by their
filenames. A filename is simply a reference to an inode. This design
allows Linux to create multiple references to the same file through
**hard links** and **symbolic (soft) links**.

Understanding the difference between these two types of links is an
important filesystem concept for Linux users, administrators, and DevOps
engineers.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Understand what an inode is
-   Explain the purpose of file links
-   Create hard links
-   Create symbolic (soft) links
-   Identify the differences between hard and soft links
-   Inspect links using `ls -l`

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Creating files with `touch`
-   Listing files using `ls -l`

------------------------------------------------------------------------

# Theory

When you create a file:

``` bash
touch notes.txt
```

Linux creates:

-   An inode (stores the file's metadata and data location)
-   A filename (`notes.txt`) that points to that inode

Simplified view:

``` text
notes.txt
    │
    ▼
+---------+
| Inode   |
+---------+
| Data    |
+---------+
```

A link is simply another way of reaching the same data.

------------------------------------------------------------------------

# Hard Links

A **hard link** is another filename that points to the **same inode**.

``` text
notes.txt        notes_backup.txt
      │                 │
      └────────┬────────┘
               ▼
          +---------+
          | Inode   |
          +---------+
          | Data    |
          +---------+
```

Both names are equal. Neither is the "original."

------------------------------------------------------------------------

# Creating a Hard Link

## Syntax

``` bash
ln existing_file hard_link_name
```

## Example

``` bash
touch notes.txt

ln notes.txt notes_backup.txt
```

Verify:

``` bash
ls -li
```

Example:

``` text
105248 -rw-r--r-- 2 hamad hamad 0 Jul 1 notes.txt
105248 -rw-r--r-- 2 hamad hamad 0 Jul 1 notes_backup.txt
```

Notice:

-   Same inode number
-   Link count is `2`

------------------------------------------------------------------------

# Hard Link Behavior

Modify one file:

``` bash
echo "Linux" > notes.txt
cat notes_backup.txt
```

Output:

``` text
Linux
```

Both names reference the same data.

Delete one name:

``` bash
rm notes.txt
```

The data still exists because `notes_backup.txt` still points to the
inode.

------------------------------------------------------------------------

# Soft (Symbolic) Links

A **symbolic link** is a special file that stores the path to another
file.

``` text
shortcut.txt
      │
      ▼
notes.txt
      │
      ▼
+---------+
| Inode   |
+---------+
| Data    |
+---------+
```

Unlike a hard link, the symbolic link has its own inode.

------------------------------------------------------------------------

# Creating a Symbolic Link

## Syntax

``` bash
ln -s target_file symbolic_link
```

## Example

``` bash
ln -s notes.txt shortcut.txt
```

Verify:

``` bash
ls -l
```

Example:

``` text
lrwxrwxrwx 1 hamad hamad 9 Jul 1 shortcut.txt -> notes.txt
```

The leading `l` indicates a symbolic link.

------------------------------------------------------------------------

# Broken Symbolic Links

Delete the original file:

``` bash
rm notes.txt
```

The symbolic link remains but points to a missing target.

``` bash
cat shortcut.txt
```

Result:

``` text
No such file or directory
```

This is called a **broken symbolic link**.

------------------------------------------------------------------------

# Hard Link vs. Soft Link

  Feature                                  Hard Link      Soft Link
  ---------------------------------------- -------------- ---------------------
  Shares the same inode                    Yes            No
  Can link directories                     Generally No   Yes
  Works across filesystems                 No             Yes
  Survives deletion of original filename   Yes            No
  File type                                Regular file   Symbolic link (`l`)

------------------------------------------------------------------------

# Practical Scenario

A configuration file is shared by multiple applications.

Instead of creating duplicate copies, a system administrator creates
symbolic links so all applications reference the same configuration
file.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

touch notes.txt

echo "Linux Practice" > notes.txt

ln notes.txt notes-hard.txt

ln -s notes.txt notes-soft.txt

ls -li

cat notes-hard.txt

cat notes-soft.txt

rm notes.txt

ls -l

cat notes-hard.txt

cat notes-soft.txt
```

Observe what happens after deleting the original filename.

------------------------------------------------------------------------

# Challenge

1.  Create a file named `report.txt`.
2.  Create a hard link.
3.  Create a symbolic link.
4.  Compare their inode numbers.
5.  Delete the original filename and observe the behavior of both links.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand what an inode is.
-   [ ] I can create a hard link.
-   [ ] I can create a symbolic link.
-   [ ] I know how to identify a symbolic link using `ls -l`.
-   [ ] I understand the difference between hard and soft links.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing hard links with copies.
-   Expecting a symbolic link to work after the target file is removed.
-   Forgetting to use `-s` when creating a symbolic link.

------------------------------------------------------------------------

# Best Practices

-   Use symbolic links for shortcuts and shared paths.
-   Use hard links only when you understand inode behavior.
-   Verify links with `ls -li` and `ls -l`.

------------------------------------------------------------------------

# Interview Questions

1.  What is an inode?
2.  What is the difference between a hard link and a symbolic link?
3.  Why does a hard link survive deleting the original filename?
4.  What is a broken symbolic link?
5.  Which type of link can cross filesystems?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I explain why filenames are not the file itself?
-   Can I decide when to use a hard link or a symbolic link?
-   Do I understand why Linux uses inodes?

------------------------------------------------------------------------

# Summary

Hard links and symbolic links provide flexible ways to reference files.
Understanding how they interact with inodes is a key Linux filesystem
concept and an important foundation for advanced system administration.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  --------------------------------------------------------------------------
  Filename                               Capture
  -------------------------------------- -----------------------------------
  `62-hard-link-created.png`             Creating a hard link and viewing
                                         inode numbers

  `63-soft-link-created.png`             Creating a symbolic link

  `64-link-comparison.png`               Output of `ls -li` comparing links

  `65-broken-symbolic-link.png`          Symbolic link after deleting the
                                         target

  `66-hard-soft-link-lab-complete.png`   Final terminal after completing the
                                         lab
  --------------------------------------------------------------------------
