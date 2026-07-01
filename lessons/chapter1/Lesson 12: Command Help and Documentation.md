# Chapter 01 --- Lesson 12: Command Help and Documentation

## Overview

No one memorizes every Linux command or every available option. Linux
provides built-in documentation tools that allow you to learn commands
directly from the terminal.

Knowing how to find reliable documentation is one of the most valuable
skills for any Linux user.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Use `man` to read command manuals
-   Use `--help` for quick command references
-   Find manual pages by keyword with `apropos`
-   Understand manual page sections
-   Learn unfamiliar commands independently

------------------------------------------------------------------------

# Prerequisites

-   Basic Linux terminal navigation
-   Running common Linux commands

------------------------------------------------------------------------

# Theory

Linux is designed to be self-documenting. Almost every command includes
documentation that explains:

-   What the command does
-   Available options
-   Usage examples
-   Related commands

Learning to read documentation will make you less dependent on tutorials
and help you solve problems on your own.

------------------------------------------------------------------------

# Command: `man`

`man` stands for **Manual**.

## Syntax

``` bash
man command
```

## Example

``` bash
man ls
```

Use these keys while reading:

  Key       Action
  --------- --------------------
  `Space`   Next page
  `b`       Previous page
  `/text`   Search
  `n`       Next search result
  `q`       Quit

------------------------------------------------------------------------

# Command: `--help`

Many commands provide a quick help summary.

## Syntax

``` bash
command --help
```

## Examples

``` bash
ls --help

cp --help

mkdir --help
```

This is useful when you need a quick reminder without opening the full
manual.

------------------------------------------------------------------------

# Command: `apropos`

Search the manual database by keyword.

## Syntax

``` bash
apropos keyword
```

## Example

``` bash
apropos copy
```

Example output:

``` text
cp (1)      - copy files and directories
scp (1)     - secure file copy
```

If `apropos` returns no results, update the manual database (if
supported by your distribution):

``` bash
sudo mandb
```

------------------------------------------------------------------------

# Understanding Manual Sections

Sometimes multiple manual pages exist for the same name.

Example:

``` bash
man 5 passwd
```

Common sections include:

  Section   Description
  --------- --------------------------------
  1         User commands
  5         Configuration files
  8         System administration commands

------------------------------------------------------------------------

# Practical Scenario

You forget how to copy directories.

Instead of searching the web, you run:

``` bash
man cp
```

Search for:

``` text
recursive
```

You discover the `-r` option and continue working.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
man ls

ls --help

man mkdir

mkdir --help

apropos directory

apropos copy
```

Practice navigating inside the manual with:

-   Space
-   b
-   /
-   n
-   q

------------------------------------------------------------------------

# Challenge

1.  Open the manual page for `mv`.
2.  Find the option that prevents overwriting files.
3.  Display the help page for `find`.
4.  Search the manual database for the keyword `search`.
5.  Exit every manual correctly.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can open a manual page.
-   [ ] I can search inside a manual page.
-   [ ] I can use `--help`.
-   [ ] I can search documentation with `apropos`.
-   [ ] I understand manual sections.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to quit the manual with `q`.
-   Assuming every command supports identical options.
-   Ignoring built-in documentation and searching externally first.

------------------------------------------------------------------------

# Best Practices

-   Read the manual before using unfamiliar commands.
-   Use `--help` for quick syntax reminders.
-   Search manual pages instead of memorizing every option.
-   Build the habit of learning directly from the terminal.

------------------------------------------------------------------------

# Interview Questions

1.  What does the `man` command do?
2.  What is the difference between `man` and `--help`?
3.  What is `apropos` used for?
4.  How do you search inside a manual page?
5.  Why are Linux manual pages important?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I find documentation without using a web browser?
-   Do I know how to search inside a manual page?
-   Am I becoming more independent when learning Linux?

------------------------------------------------------------------------

# Summary

Linux includes powerful built-in documentation tools. By mastering
`man`, `--help`, and `apropos`, you can confidently explore new commands
and solve problems using official documentation.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -------------------------------------------------------------------------
  Filename                              Capture
  ------------------------------------- -----------------------------------
  `46-man-ls.png`                       Opening the `ls` manual

  `47-man-search.png`                   Searching within a manual page

  `48-help-option.png`                  Output of `ls --help`

  `49-apropos-copy.png`                 Results of `apropos copy`

  `50-documentation-lab-complete.png`   Final terminal after completing the
                                        lab
  -------------------------------------------------------------------------
