# Chapter 02 -- Lesson 03 (Revised)

# Searching Text with `grep`

> **Command Covered:** `grep`

------------------------------------------------------------------------

# Overview

`grep` is one of the most frequently used Linux commands. It searches
text for patterns and is used daily by system administrators,
developers, security engineers, and DevOps teams.

Typical use cases include:

-   Searching log files
-   Finding configuration settings
-   Looking for users in `/etc/passwd`
-   Filtering command output
-   Investigating incidents

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Search for words and phrases
-   Match whole words and whole lines
-   Ignore case
-   Display line numbers
-   Count matches
-   Print only matching text
-   Search directories recursively
-   Show filenames that match (or don't)
-   Display context around matches
-   Understand basic regular expressions
-   Build useful pipelines with `grep`

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
root Administrator
ERROR Database failed
ERROR Timeout
EOF
```

------------------------------------------------------------------------

# Syntax

``` bash
grep [OPTIONS] PATTERN FILE
```

------------------------------------------------------------------------

# Basic Search

``` bash
grep "Engineering" employees.txt
```

------------------------------------------------------------------------

# Ignore Case (`-i`)

``` bash
grep -i engineering employees.txt
```

------------------------------------------------------------------------

# Show Line Numbers (`-n`)

``` bash
grep -n HR employees.txt
```

------------------------------------------------------------------------

# Count Matches (`-c`)

``` bash
grep -c ERROR employees.txt
```

------------------------------------------------------------------------

# Invert Match (`-v`)

``` bash
grep -v HR employees.txt
```

------------------------------------------------------------------------

# Match Whole Words (`-w`)

Without `-w`, searching for `root` may also match words containing
`root`.

``` bash
grep -w root employees.txt
```

------------------------------------------------------------------------

# Match Entire Lines (`-x`)

``` bash
grep -x "Emma HR" employees.txt
```

The entire line must match exactly.

------------------------------------------------------------------------

# Show Matching Filenames (`-l`)

``` bash
grep -l ERROR *.log
```

Useful when searching many log files.

------------------------------------------------------------------------

# Show Files Without Matches (`-L`)

``` bash
grep -L ERROR *.log
```

------------------------------------------------------------------------

# Print Only the Match (`-o`)

``` bash
grep -o ERROR employees.txt
```

Output:

``` text
ERROR
ERROR
```

------------------------------------------------------------------------

# Recursive Search (`-r`)

``` bash
grep -r TODO .
```

Search every file below the current directory.

------------------------------------------------------------------------

# Context Options

Show lines **after** a match:

``` bash
grep -A 2 ERROR application.log
```

Show lines **before** a match:

``` bash
grep -B 2 ERROR application.log
```

Show lines before **and** after:

``` bash
grep -C 2 ERROR application.log
```

These options are extremely useful when troubleshooting logs.

------------------------------------------------------------------------

# Basic Regular Expressions

## Dot (`.`)

Matches any single character.

``` bash
grep "c.t" words.txt
```

Matches:

``` text
cat
cot
cut
```

------------------------------------------------------------------------

## Asterisk (`*`)

Repeats the previous character zero or more times.

Example:

``` bash
grep "ab*c" words.txt
```

Matches:

``` text
ac
abc
abbc
abbbc
```

------------------------------------------------------------------------

## Caret (`^`)

Beginning of a line.

``` bash
grep "^root" /etc/passwd
```

------------------------------------------------------------------------

## Dollar Sign (`$`)

End of a line.

``` bash
grep "failed$" log.txt
```

------------------------------------------------------------------------

## Character Classes (`[]`)

``` bash
grep "[0-9]" numbers.txt
```

``` bash
grep "[AEIOU]" names.txt
```

------------------------------------------------------------------------

# Real Linux Examples

Find the root account:

``` bash
grep "^root" /etc/passwd
```

Search groups:

``` bash
grep sudo /etc/group
```

Search host entries:

``` bash
grep localhost /etc/hosts
```

Count failed SSH logins:

``` bash
grep "Failed password" /var/log/auth.log | wc -l
```

------------------------------------------------------------------------

# Pipelines

Running SSH processes:

``` bash
ps -ef | grep ssh
```

Find the most common error:

``` bash
grep ERROR application.log | sort | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Practical Scenario

An application crashes repeatedly.

Investigate:

``` bash
grep -C 3 ERROR application.log
```

This displays the error together with surrounding events.

------------------------------------------------------------------------

# Hands-on Lab

1.  Search for Engineering.
2.  Ignore case.
3.  Show line numbers.
4.  Match whole words.
5.  Match an entire line.
6.  Print only matching text.
7.  Search recursively.
8.  Display two lines of context around ERROR.
9.  Search `/etc/passwd` for the root account.

------------------------------------------------------------------------

# Challenge

Create three log files.

Use:

-   `grep -l`
-   `grep -L`
-   `grep -A`
-   `grep -B`
-   `grep -C`
-   `grep -o`

Then explain the output of each command.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting quotes around patterns containing spaces.
-   Using `uniq` without sorting first.
-   Confusing shell wildcards (`*`) with regular expressions.

------------------------------------------------------------------------

# Best Practices

-   Use `-i` when capitalization is uncertain.
-   Use `-n` during debugging.
-   Use `-C` when investigating logs.
-   Combine `grep` with pipes for efficient analysis.

------------------------------------------------------------------------

# Interview Questions

1.  What does `grep` do?
2.  What is the difference between `-w` and `-x`?
3.  When would you use `-A`, `-B`, and `-C`?
4.  What does `grep -o` print?
5.  Explain the meaning of `.`, `*`, `^`, `$`, and `[]`.

------------------------------------------------------------------------

# Cheat Sheet

``` text
grep text file
grep -i text file
grep -n text file
grep -c text file
grep -v text file
grep -w text file
grep -x "line" file
grep -o pattern file
grep -l pattern *.log
grep -L pattern *.log
grep -A 2 pattern file
grep -B 2 pattern file
grep -C 2 pattern file
grep -r pattern .
```

------------------------------------------------------------------------

# Lab Assets

Capture:

-   lesson-03-basic-search.png
-   lesson-03-context-search.png
-   lesson-03-regex.png
-   lesson-03-passwd-search.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 04 -- Counting Text with `wc`**
