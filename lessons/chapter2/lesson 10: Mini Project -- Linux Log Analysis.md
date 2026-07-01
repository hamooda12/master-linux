# Chapter 02 -- Lesson 10

# Mini Project -- Linux Log Analysis

> **Capstone Project:** Apply everything learned in Chapter 02

------------------------------------------------------------------------

# Project Overview

You have just joined a company's Infrastructure team.

A web server experienced several issues overnight, and your manager has
asked you to analyze the application log from the Linux terminal.

Your objective is to investigate the log and produce useful reports
using only command-line tools.

------------------------------------------------------------------------

# Skills Practiced

This project combines:

-   `cat`
-   `less`
-   `head`
-   `tail`
-   `grep`
-   `wc`
-   `sort`
-   `uniq`
-   `diff`
-   `cmp`
-   Redirection (`>`, `>>`, `2>`)
-   Pipes (`|`)

------------------------------------------------------------------------

# Learning Objectives

By completing this project you will:

-   Read large log files efficiently
-   Search for specific events
-   Count log entries
-   Build command pipelines
-   Generate reports
-   Think like a Linux administrator

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/final-project
cd ~/chapter02/final-project

cat > application.log << EOF
INFO Login Alice
INFO Login Bob
WARNING Disk Space Low
ERROR Database Connection Failed
INFO Login Alice
ERROR Timeout
INFO Logout Bob
ERROR Database Connection Failed
WARNING CPU High
INFO Login Charlie
ERROR Database Connection Failed
INFO Login Alice
EOF
```

------------------------------------------------------------------------

# Task 1 -- Inspect the Log

View the first lines.

``` bash
head application.log
```

View the last lines.

``` bash
tail application.log
```

Browse interactively.

``` bash
less application.log
```

------------------------------------------------------------------------

# Task 2 -- Count Entries

Count all log lines.

``` bash
wc -l application.log
```

------------------------------------------------------------------------

# Task 3 -- Find Errors

Display every error.

``` bash
grep ERROR application.log
```

Count the errors.

``` bash
grep ERROR application.log | wc -l
```

------------------------------------------------------------------------

# Task 4 -- Find Warnings

``` bash
grep WARNING application.log
```

------------------------------------------------------------------------

# Task 5 -- Most Frequent Error

``` bash
grep ERROR application.log | sort | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Task 6 -- Find Active Users

``` bash
grep "Login" application.log | awk '{print $3}' | sort | uniq -c | sort -nr
```

Expected result:

``` text
3 Alice
1 Bob
1 Charlie
```

------------------------------------------------------------------------

# Task 7 -- Generate Reports

Create an error report.

``` bash
grep ERROR application.log > error-report.txt
```

Create a summary report.

``` bash
{
echo "Total Entries:"
wc -l < application.log

echo
echo "Errors:"
grep ERROR application.log | wc -l

echo
echo "Warnings:"
grep WARNING application.log | wc -l
} > summary-report.txt
```
# Chapter 02 -- Lesson 10 (Revised)

## Bonus Exercise -- Analyze a Real System Log

Depending on your Linux distribution, inspect one of these files:

``` text
/var/log/syslog
```

or

``` text
/var/log/auth.log
```

Try the following tasks:

``` bash
tail -50 /var/log/syslog
```

``` bash
grep ERROR /var/log/syslog
```

``` bash
grep WARNING /var/log/syslog | wc -l
```

``` bash
grep "Failed password" /var/log/auth.log
```

Generate a report:

``` bash
grep ERROR /var/log/syslog | sort | uniq -c | sort -nr > system-error-report.txt
```

## Final Goal

By completing this project you should be comfortable investigating real
Linux logs using only command-line tools.
------------------------------------------------------------------------

# Verification Checklist

-   [ ] Read the log with `less`
-   [ ] Count all entries
-   [ ] Count ERROR entries
-   [ ] Count WARNING entries
-   [ ] Find the most common error
-   [ ] Find the most active user
-   [ ] Save reports to files

------------------------------------------------------------------------

# Challenge

Extend the project by:

1.  Adding 100 more log entries.
2.  Creating an `INFO` report.
3.  Comparing two reports using `diff`.
4.  Redirecting errors to a separate file.
5.  Creating one pipeline that shows the most common log message.

------------------------------------------------------------------------

# Common Mistakes

-   Using `cat` for large logs.
-   Forgetting to sort before `uniq`.
-   Overwriting reports with `>` accidentally.
-   Building long pipelines without testing each stage.

------------------------------------------------------------------------

# Best Practices

-   Test pipelines incrementally.
-   Keep reports in a dedicated directory.
-   Redirect output instead of copying from the terminal.
-   Prefer `less` for large logs.

------------------------------------------------------------------------

# Interview Questions

1.  How would you identify the most common error in a log?
2.  Why is `sort` required before `uniq`?
3.  When would you use `tail -f`?
4.  What is the difference between `>` and `>>`?
5.  How do pipes improve productivity?

------------------------------------------------------------------------

# Chapter 02 Summary

You have learned to:

-   Read text files
-   Inspect large logs
-   Search with `grep`
-   Count with `wc`
-   Sort and remove duplicates
-   Compare files
-   Redirect input and output
-   Build pipelines
-   Analyze logs like a Linux administrator

Congratulations! You have completed **Chapter 02 -- Working with Text
Files and Streams**.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-10-log-overview.png
-   lesson-10-error-search.png
-   lesson-10-user-report.png
-   lesson-10-summary-report.png
-   lesson-10-terminal-final.png

------------------------------------------------------------------------

# Next Chapter

**Chapter 03 -- Archives, Compression, and Backups**

You'll learn:

-   `gzip` / `gunzip`
-   `bzip2`
-   `xz`
-   `tar`
-   `zip` / `unzip`
-   Backup strategies
-   Building automated backup workflows
