# Chapter 02 -- Lesson 05

# Sorting and Removing Duplicate Data

> **Commands Covered:** `sort`, `uniq`

------------------------------------------------------------------------

# Overview

When working with logs, configuration files, reports, or datasets,
information is often unordered and may contain duplicate entries. Linux
provides two powerful utilities---`sort` and `uniq`---to organize data
and remove repeated lines efficiently.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Sort text alphabetically and numerically
-   Sort in ascending and descending order
-   Remove duplicate lines
-   Count duplicate entries
-   Combine `sort` and `uniq` in pipelines

------------------------------------------------------------------------

# Theory

`uniq` only removes **adjacent** duplicate lines.

Because of this, `uniq` is almost always used together with `sort`.

Example:

``` text
Apple
Banana
Apple
```

Running only:

``` bash
uniq fruits.txt
```

will **not** remove both `Apple` entries because they are not next to
each other.

Instead:

``` bash
sort fruits.txt | uniq
```

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson05
cd ~/chapter02/lesson05

cat > fruits.txt << EOF
Orange
Apple
Banana
Apple
Mango
Banana
Orange
EOF
```

------------------------------------------------------------------------

# Command: sort

## Syntax

``` bash
sort [OPTIONS] FILE
```

Sort alphabetically:

``` bash
sort fruits.txt
```

Sort in reverse order:

``` bash
sort -r fruits.txt
```

Sort numbers:

``` bash
sort -n numbers.txt
```

Ignore letter case:

``` bash
sort -f names.txt
```

------------------------------------------------------------------------

# Command: uniq

## Syntax

``` bash
uniq [OPTIONS] FILE
```

Remove adjacent duplicates:

``` bash
uniq sorted.txt
```

Count duplicates:

``` bash
uniq -c sorted.txt
```

Show only duplicate lines:

``` bash
uniq -d sorted.txt
```

Show only unique lines:

``` bash
uniq -u sorted.txt
```

------------------------------------------------------------------------

# Combining sort and uniq

Remove duplicates correctly:

``` bash
sort fruits.txt | uniq
```

Count duplicate entries:

``` bash
sort fruits.txt | uniq -c
```

Find the most common values:

``` bash
sort fruits.txt | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Practical Scenario

A server log contains thousands of IP addresses.

To identify the most frequent client IPs:

``` bash
sort access.log | uniq -c | sort -nr
```

This produces a frequency table with the busiest IP addresses first.

------------------------------------------------------------------------

# Hands-on Lab

1.  Sort `fruits.txt`.
2.  Sort in reverse order.
3.  Create a numeric file and sort it.
4.  Remove duplicate fruit names.
5.  Count duplicate entries.
6.  Display only duplicated values.
7.  Display only unique values.

------------------------------------------------------------------------

# Challenge

Create a file with 30 names where several names appear multiple times.

Practice:

-   `sort`
-   `sort -r`
-   `sort -f`
-   `uniq`
-   `uniq -c`
-   `sort | uniq -c | sort -nr`

------------------------------------------------------------------------

# Common Mistakes

-   Running `uniq` on an unsorted file.
-   Forgetting that `uniq` compares adjacent lines only.
-   Using alphabetical sort for numeric data instead of `sort -n`.

------------------------------------------------------------------------

# Best Practices

-   Always use `sort` before `uniq` unless the file is already sorted.
-   Use `sort -n` for numbers.
-   Use `uniq -c` to generate quick statistics.
-   Build pipelines instead of creating temporary files.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `sort` and `uniq`?
2.  Why is `uniq` usually combined with `sort`?
3.  What does `sort -n` do?
4.  What does `uniq -c` display?
5.  How would you find the most common value in a file?

------------------------------------------------------------------------

# Summary

You learned how to organize text, remove duplicates, count repeated
entries, and build pipelines for basic data analysis.

------------------------------------------------------------------------

# Cheat Sheet

``` text
sort file.txt
sort -r file.txt
sort -n numbers.txt
sort -f names.txt

uniq file.txt
uniq -c file.txt
uniq -d file.txt
uniq -u file.txt

sort file.txt | uniq
sort file.txt | uniq -c
sort file.txt | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-05-sort.png
-   lesson-05-sort-reverse.png
-   lesson-05-uniq.png
-   lesson-05-uniq-count.png
-   lesson-05-frequency-table.png

------------------------------------------------------------------------
# Chapter 02 -- Lesson 05 (Revised)

# Additional `sort` Features

## Sort by Column

``` bash
sort -k2 scores.txt
```

Example:

``` text
Alice 90
Bob 75
Charlie 100
```

Sorts using the second column.

------------------------------------------------------------------------

## Remove Duplicates Directly

``` bash
sort -u names.txt
```

Equivalent to sorting and removing duplicates in one step.

------------------------------------------------------------------------

## Sort Human-Readable Sizes

``` bash
sort -h sizes.txt
```

Useful for values such as:

``` text
800K
20M
3G
```

------------------------------------------------------------------------

## Real Example

``` bash
du -sh * | sort -h
```

Displays directories ordered by size.

# Next Lesson

**Lesson 06 -- Comparing Files**

Commands:

-   `diff`
-   `cmp`
