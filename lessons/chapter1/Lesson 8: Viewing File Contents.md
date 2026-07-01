# Chapter 01 --- Lesson 8: Viewing File Contents

## Overview

Before editing a file, you often need to inspect its contents. Linux
provides several commands for reading text files directly from the
terminal. These commands are essential for system administration,
troubleshooting, and software development.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Display an entire file with `cat`
-   Read long files page by page with `less`
-   View the beginning of a file with `head`
-   View the end of a file with `tail`
-   Monitor log files in real time using `tail -f`

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Creating files
-   Listing files

------------------------------------------------------------------------

# Theory

Linux stores configuration files, logs, scripts, and source code as
plain text. Instead of opening a graphical editor, administrators often
inspect these files directly from the terminal.

------------------------------------------------------------------------

# Command: `cat`

`cat` stands for **Concatenate**.

## Syntax

``` bash
cat filename
```

## Example

``` bash
cat README.md
```

Display multiple files:

``` bash
cat file1.txt file2.txt
```

------------------------------------------------------------------------

# Command: `less`

`less` lets you scroll through large files.

``` bash
less /etc/services
```

Useful keys:

  Key       Action
  --------- ---------------
  `Space`   Next page
  `b`       Previous page
  `/text`   Search
  `q`       Quit

------------------------------------------------------------------------

# Command: `head`

Displays the first 10 lines by default.

``` bash
head notes.txt
```

Display the first 20 lines:

``` bash
head -n 20 notes.txt
```

------------------------------------------------------------------------

# Command: `tail`

Displays the last 10 lines.

``` bash
tail logfile.log
```

Display the last 25 lines:

``` bash
tail -n 25 logfile.log
```

------------------------------------------------------------------------

## Monitor a File in Real Time

``` bash
tail -f /var/log/syslog
```

This is commonly used to watch application or system logs as they are
updated.

Stop monitoring with:

``` text
Ctrl + C
```

------------------------------------------------------------------------

# Practical Scenario

An application reports an error. You inspect the log:

``` bash
tail -f /var/log/syslog
```

As new events are written, they immediately appear on your screen,
helping you troubleshoot the problem.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice/practice

cat README.md

head README.md

tail README.md

less README.md

head -n 5 README.md

tail -n 5 README.md
```

If `README.md` does not exist, create it first:

``` bash
touch README.md
```

------------------------------------------------------------------------

# Challenge

1.  Create a file with several lines of text.
2.  Display the entire file.
3.  Display only the first three lines.
4.  Display only the last three lines.
5.  Open the file with `less` and search for a word.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can use `cat`.
-   [ ] I can use `less`.
-   [ ] I can display the beginning of a file.
-   [ ] I can display the end of a file.
-   [ ] I understand when to use `tail -f`.

------------------------------------------------------------------------

# Common Mistakes

-   Using `cat` on extremely large files.
-   Forgetting to exit `less` with `q`.
-   Confusing `head` and `tail`.

------------------------------------------------------------------------

# Best Practices

-   Use `less` for large files.
-   Use `head` to preview files.
-   Use `tail -f` for monitoring logs.
-   Avoid opening huge log files with `cat`.

------------------------------------------------------------------------

# Interview Questions

1.  What is the purpose of `cat`?
2.  When should you use `less` instead of `cat`?
3.  What does `head -n` do?
4.  What is `tail -f` commonly used for?
5.  How do you exit `less`?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I inspect any text file from the terminal?
-   Do I know which command is appropriate for large files?
-   Can I monitor logs in real time?

------------------------------------------------------------------------

# Summary

Linux provides several commands for viewing file contents. Mastering
`cat`, `less`, `head`, and `tail` enables you to inspect configuration
files, review logs, and troubleshoot systems efficiently.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  Filename                   Capture
  -------------------------- ----------------------------------------
  `26-cat-output.png`        Displaying a file with `cat`
  `27-less-navigation.png`   Viewing a file with `less`
  `28-head-command.png`      Using `head`
  `29-tail-command.png`      Using `tail`
  `30-tail-follow.png`       Demonstrating `tail -f` (if available)
