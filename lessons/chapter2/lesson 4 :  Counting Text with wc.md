# Chapter 02 -- Lesson 04

# Counting Text with `wc`

> **Command Covered:** `wc`

------------------------------------------------------------------------

# Overview

The `wc` (word count) command is a simple but powerful utility used to
count lines, words, characters, and bytes in text files. It is
frequently used by Linux administrators, developers, and DevOps
engineers when analyzing logs, reports, and datasets.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Count lines in a file
-   Count words
-   Count characters and bytes
-   Count multiple files at once
-   Use `wc` with pipes

------------------------------------------------------------------------

# Theory

Many Linux tasks begin by answering simple questions:

-   How many log entries are there?
-   How many users are listed?
-   How many configuration lines exist?
-   How large is a text file?

Instead of manually counting, Linux provides `wc`.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson04
cd ~/chapter02/lesson04

cat > notes.txt << EOF
Linux is powerful.
Learning Linux takes practice.
Practice builds confidence.
EOF
```

------------------------------------------------------------------------

# Command Syntax

``` bash
wc [OPTIONS] FILE
```

Run:

``` bash
wc notes.txt
```

Example output:

``` text
3 9 68 notes.txt
```

The columns represent:

1.  Lines
2.  Words
3.  Bytes

------------------------------------------------------------------------

# Count Lines

``` bash
wc -l notes.txt
```

------------------------------------------------------------------------

# Count Words

``` bash
wc -w notes.txt
```

------------------------------------------------------------------------

# Count Characters

``` bash
wc -m notes.txt
```

------------------------------------------------------------------------

# Count Bytes

``` bash
wc -c notes.txt
```

> Characters and bytes are often the same for plain English text, but
> they may differ with Unicode characters.

------------------------------------------------------------------------

# Count Multiple Files

``` bash
wc file1.txt file2.txt
```

`wc` prints the statistics for each file followed by a total.

------------------------------------------------------------------------

# Using wc with Pipes

Count running processes:

``` bash
ps -ef | wc -l
```

Count Markdown files in the current directory:

``` bash
ls *.md | wc -l
```

Count failed login attempts from a log:

``` bash
grep "Failed" auth.log | wc -l
```

------------------------------------------------------------------------

# Practical Scenario

A security team asks:

> "How many failed SSH login attempts occurred today?"

You can answer quickly:

``` bash
grep "Failed password" auth.log | wc -l
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Count lines in `notes.txt`.
2.  Count words.
3.  Count characters.
4.  Count bytes.
5.  Create a second file and run `wc` on both.
6.  Count the number of files in your lesson directory using a pipeline.

------------------------------------------------------------------------

# Challenge

Create a file containing 50 lines.

Practice:

-   `wc`
-   `wc -l`
-   `wc -w`
-   `wc -m`
-   `wc -c`

Then compare the outputs.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing characters with bytes.
-   Assuming `wc` only counts words.
-   Forgetting that the default output contains three numeric columns.

------------------------------------------------------------------------

# Best Practices

-   Use `wc -l` when counting records.
-   Combine `grep` and `wc` for quick statistics.
-   Use pipelines to summarize command output.

------------------------------------------------------------------------

# Interview Questions

1.  What does `wc` stand for?
2.  What do the default output columns represent?
3.  What is the difference between `-m` and `-c`?
4.  Why is `wc -l` commonly used with `grep`?
5.  How do you count files using a pipeline?

------------------------------------------------------------------------

# Summary

You learned how to count lines, words, characters, and bytes, and how to
combine `wc` with other commands for fast analysis.

------------------------------------------------------------------------

# Cheat Sheet

``` text
wc file.txt
wc -l file.txt
wc -w file.txt
wc -m file.txt
wc -c file.txt

grep ERROR app.log | wc -l
ps -ef | wc -l
```

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-04-wc-default.png
-   lesson-04-wc-lines.png
-   lesson-04-wc-words.png
-   lesson-04-wc-pipeline.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 05 -- Sorting and Removing Duplicate Data**

Commands:

-   `sort`
-   `uniq`
