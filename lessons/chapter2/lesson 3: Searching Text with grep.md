# Chapter 02 -- Lesson 03

# Searching Text with grep

> **Commands Covered:** `grep`

------------------------------------------------------------------------

# Overview

Searching for information quickly is one of the most valuable Linux
skills. System administrators use `grep` every day to search
configuration files, logs, source code, and command output.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Search for words and phrases
-   Ignore letter case
-   Show line numbers
-   Invert search results
-   Count matches
-   Search recursively through directories
-   Use `grep` in command pipelines

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson03
cd ~/chapter02/lesson03

cat > employees.txt << EOF
Alice Engineering
Bob HR
Charlie Engineering
David Sales
Emma HR
EOF
```

------------------------------------------------------------------------

# Command Syntax

``` bash
grep [OPTIONS] PATTERN FILE
```

------------------------------------------------------------------------

# Basic Search

``` bash
grep "Engineering" employees.txt
```

Returns every line containing the word **Engineering**.

------------------------------------------------------------------------

# Ignore Case

``` bash
grep -i "engineering" employees.txt
```

Matches `Engineering`, `ENGINEERING`, and `engineering`.

------------------------------------------------------------------------

# Show Line Numbers

``` bash
grep -n "HR" employees.txt
```

Example:

``` text
2:Bob HR
5:Emma HR
```

------------------------------------------------------------------------

# Count Matches

``` bash
grep -c "HR" employees.txt
```

Output:

``` text
2
```

------------------------------------------------------------------------

# Invert the Match

``` bash
grep -v "HR" employees.txt
```

Displays every line that does **not** contain `HR`.

------------------------------------------------------------------------

# Search Recursively

``` bash
grep -r "TODO" .
```

Searches every file in the current directory tree.

------------------------------------------------------------------------

# grep in Pipelines

Find SSH processes:

``` bash
ps aux | grep ssh
```

Search recent authentication failures:

``` bash
tail -100 /var/log/auth.log | grep "Failed"
```

------------------------------------------------------------------------

# Practical Scenario

A production server reports database failures.

Instead of reading thousands of log lines:

``` bash
grep -i "error" application.log
```

or

``` bash
grep -i "database" application.log
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Search for Engineering.
2.  Search for HR.
3.  Count HR entries.
4.  Display line numbers.
5.  Ignore case.
6.  Exclude Sales.
7.  Create another file and search recursively.

------------------------------------------------------------------------

# Challenge

Create a log file with 20 entries containing INFO, WARNING, and ERROR.

Practice:

-   `grep ERROR`
-   `grep -c ERROR`
-   `grep -v INFO`
-   `grep -n WARNING`

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting quotes around patterns with spaces.
-   Searching the wrong directory.
-   Confusing `grep` with shell wildcards.

------------------------------------------------------------------------

# Best Practices

-   Use `-i` when capitalization is inconsistent.
-   Use `-n` while debugging.
-   Combine `grep` with `tail`, `head`, `cat`, and pipes.

------------------------------------------------------------------------

# Interview Questions

1.  What does `grep` do?
2.  What is the purpose of `-i`?
3.  What does `-v` do?
4.  How do you count matches?
5.  Why is `grep` commonly used with pipes?

------------------------------------------------------------------------

# Cheat Sheet

``` text
grep text file
grep -i text file
grep -n text file
grep -v text file
grep -c text file
grep -r text .
```

------------------------------------------------------------------------

# Lab Assets

Capture:

-   lesson-03-basic-grep.png
-   lesson-03-ignore-case.png
-   lesson-03-line-numbers.png
-   lesson-03-recursive-search.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 04 -- Counting Text with `wc`**
