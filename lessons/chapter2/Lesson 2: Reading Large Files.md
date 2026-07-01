# Chapter 02 -- Lesson 02

# Reading Large Files

> **Commands Covered:** `head`, `tail`, `tail -f`

------------------------------------------------------------------------

# Overview

Large log files are common on Linux servers. Reading an entire file is
often unnecessary and inefficient. Linux provides commands that let you
inspect only the beginning or the end of a file, making troubleshooting
much faster.

------------------------------------------------------------------------

# Learning Objectives

After this lesson, you will be be able to:

-   Display the first lines of a file
-   Display the last lines of a file
-   Show a custom number of lines
-   Monitor a file in real time
-   Choose the correct command for log analysis

------------------------------------------------------------------------

# Theory

System logs continuously grow as services run. Most recent events are
usually written at the end of the file, which is why administrators
frequently use `tail` instead of opening the entire file.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson02
cd ~/chapter02/lesson02

seq 1 100 > numbers.txt
```

Verify:

``` bash
wc -l numbers.txt
```

Expected output:

``` text
100 numbers.txt
```

------------------------------------------------------------------------

# Command: head

## Purpose

Display the first lines of a file.

## Syntax

``` bash
head [OPTIONS] FILE
```

Default:

``` bash
head numbers.txt
```

Displays the first 10 lines.

### Show a specific number of lines

``` bash
head -5 numbers.txt
```

------------------------------------------------------------------------

# Command: tail

## Purpose

Display the last lines of a file.

## Syntax

``` bash
tail [OPTIONS] FILE
```

Default:

``` bash
tail numbers.txt
```

Displays the last 10 lines.

### Show the last 15 lines

``` bash
tail -15 numbers.txt
```

------------------------------------------------------------------------

# Command: tail -f

## Purpose

Follow a file as new content is appended.

``` bash
tail -f server.log
```

Open another terminal and append data:

``` bash
echo "Application started" >> server.log
echo "Database connected" >> server.log
```

The first terminal immediately displays the new lines.

Stop monitoring with:

``` text
Ctrl + C
```

------------------------------------------------------------------------

# Practical Scenario

A web application suddenly stops responding.

Instead of opening a huge log file:

``` bash
cat /var/log/app.log
```

Use:

``` bash
tail -50 /var/log/app.log
```

If you are waiting for new requests:

``` bash
tail -f /var/log/app.log
```

------------------------------------------------------------------------

# Comparison

  Command     Best Use
  ----------- ----------------------
  `head`      Inspect file headers
  `tail`      View latest entries
  `tail -f`   Live monitoring

------------------------------------------------------------------------

# Hands-on Lab

1.  Create `numbers.txt` with 100 lines.
2.  Display the first 10 lines.
3.  Display the first 3 lines.
4.  Display the last 10 lines.
5.  Display the last 20 lines.
6.  Create `server.log`.
7.  Monitor it with `tail -f`.
8.  Append messages from another terminal.

------------------------------------------------------------------------

# Challenge

Create a file with 500 numbers.

-   Show the first 25 lines.
-   Show the last 25 lines.
-   Monitor a growing log file while appending five new messages.

------------------------------------------------------------------------

# Common Mistakes

-   Using `cat` for very large logs.
-   Forgetting that `tail -f` continues running until interrupted.
-   Monitoring the wrong file path.

------------------------------------------------------------------------

# Best Practices

-   Use `head` to inspect file headers.
-   Use `tail` for recent events.
-   Use `tail -f` for live troubleshooting.
-   Combine these commands with `grep` in later lessons.

------------------------------------------------------------------------

# Interview Questions

1.  What does `head` do?
2.  What is the default number of lines shown by `head` and `tail`?
3.  When would you use `tail -f`?
4.  How do you stop `tail -f`?
5.  Why is `tail` preferred for log analysis?

------------------------------------------------------------------------

# Summary

You learned how to inspect the beginning and end of files efficiently
and how to monitor files in real time using `tail -f`.

------------------------------------------------------------------------

# Cheat Sheet

``` text
head file.txt
head -5 file.txt

tail file.txt
tail -20 file.txt
tail -f server.log
```

------------------------------------------------------------------------

# Lab Assets

Save screenshots in:

``` text
assets/chapter-02/
```

Recommended files:

-   lesson-02-head.png
-   lesson-02-tail.png
-   lesson-02-tail-follow.png
-   lesson-02-log-monitoring.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 03 -- Searching Text with `grep`**

You'll learn how to locate specific words, patterns, and log entries
efficiently.
