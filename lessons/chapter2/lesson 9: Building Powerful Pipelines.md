# Chapter 02 -- Lesson 09

# Building Powerful Pipelines

> **Topics Covered:** Combining `grep`, `sort`, `uniq`, `wc`, `head`,
> `tail`, pipes, and redirection

------------------------------------------------------------------------

# Overview

A single Linux command can solve simple tasks.

A **pipeline** combines multiple commands into one workflow capable of
processing thousands or even millions of lines efficiently.

This is how Linux administrators, DevOps engineers, and SREs analyze
logs and automate repetitive work.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Build multi-stage pipelines
-   Analyze log files efficiently
-   Count matching records
-   Find the most common values
-   Save pipeline results to files
-   Read complex pipelines from left to right

------------------------------------------------------------------------

# Pipeline Design

Think of a pipeline as an assembly line.

``` text
Input File
    │
    ▼
 grep
    │
    ▼
 sort
    │
    ▼
 uniq -c
    │
    ▼
 sort -nr
    │
    ▼
 head
```

Each command performs one job.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson09
cd ~/chapter02/lesson09

cat > access.log << EOF
INFO Login Alice
ERROR Database
INFO Login Bob
WARNING Disk
ERROR Database
INFO Login Alice
ERROR Timeout
INFO Login Bob
ERROR Database
EOF
```

------------------------------------------------------------------------

# Example 1 -- Count Total Log Entries

``` bash
wc -l access.log
```

------------------------------------------------------------------------

# Example 2 -- Show Only Errors

``` bash
grep ERROR access.log
```

------------------------------------------------------------------------

# Example 3 -- Count Error Entries

``` bash
grep ERROR access.log | wc -l
```

------------------------------------------------------------------------

# Example 4 -- Unique Error Messages

``` bash
grep ERROR access.log | sort | uniq
```

------------------------------------------------------------------------

# Example 5 -- Count Each Error Type

``` bash
grep ERROR access.log | sort | uniq -c
```

Example output:

``` text
3 ERROR Database
1 ERROR Timeout
```

------------------------------------------------------------------------

# Example 6 -- Most Common Errors

``` bash
grep ERROR access.log | sort | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Example 7 -- Top Result Only

``` bash
grep ERROR access.log | sort | uniq -c | sort -nr | head -1
```

------------------------------------------------------------------------

# Example 8 -- Save Report

``` bash
grep ERROR access.log | sort | uniq -c | sort -nr > error-report.txt
```

------------------------------------------------------------------------

# Practical Scenario

A production server generated a 5 GB log overnight.

Management asks:

-   How many errors occurred?
-   Which error happened most?
-   Save the report.

One possible solution:

``` bash
grep ERROR app.log | sort | uniq -c | sort -nr > report.txt
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Count all log entries.
2.  Display only ERROR records.
3.  Count ERROR records.
4.  Find unique errors.
5.  Count each error type.
6.  Display the most common error.
7.  Save the report to a file.

------------------------------------------------------------------------

# Challenge

Create a larger log with INFO, WARNING, and ERROR messages.

Build pipelines that:

-   Count warnings.
-   Count unique users.
-   Show the five most common messages.
-   Save every report into the `reports/` directory.

------------------------------------------------------------------------

# Common Mistakes

-   Building very long pipelines without testing each stage.
-   Forgetting that order matters.
-   Skipping `sort` before `uniq`.

------------------------------------------------------------------------

# Best Practices

-   Test one command at a time.
-   Read pipelines left to right.
-   Redirect final output instead of intermediate results.
-   Add comments when using complex pipelines in scripts.

------------------------------------------------------------------------

# Interview Questions

1.  Why are pipelines powerful?
2.  Why is `sort` usually placed before `uniq`?
3.  How would you count all ERROR lines?
4.  How would you identify the most frequent error?
5.  Why is `head` useful at the end of a pipeline?

------------------------------------------------------------------------

# Summary

You learned how to combine multiple Linux utilities into efficient
workflows for searching, filtering, counting, sorting, and reporting.

------------------------------------------------------------------------

# Cheat Sheet

``` text
grep ERROR app.log | wc -l
grep ERROR app.log | sort | uniq
grep ERROR app.log | sort | uniq -c
grep ERROR app.log | sort | uniq -c | sort -nr
grep ERROR app.log | sort | uniq -c | sort -nr | head -1
grep ERROR app.log | sort | uniq -c | sort -nr > report.txt
```

------------------------------------------------------------------------

# Lab Assets

Save screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-09-error-search.png
-   lesson-09-error-count.png
-   lesson-09-frequency-report.png
-   lesson-09-saved-report.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 10 -- Mini Log Analysis Project**

Apply everything from Chapter 02 in a realistic administrator workflow.
