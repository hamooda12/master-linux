# Chapter 02 -- Lesson 08

# Pipes

> **Operator Covered:** `|`

------------------------------------------------------------------------

# Overview

The pipe (`|`) is one of the most powerful ideas in Linux and Unix.

A pipe connects two commands together so that the **output (stdout)** of
the first command becomes the **input (stdin)** of the second command.

Instead of saving intermediate results to temporary files, commands
communicate directly.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Understand how pipes work
-   Connect multiple commands together
-   Build efficient command pipelines
-   Filter command output
-   Apply the Unix philosophy in practice

------------------------------------------------------------------------

# Theory

Without a pipe:

``` text
Command A ---> Terminal
```

With a pipe:

``` text
Command A ---> Command B ---> Terminal
```

Each command performs one small task, and the shell combines them into a
workflow.

------------------------------------------------------------------------

# Unix Philosophy

> Write programs that do one thing well.

Instead of creating one huge command, Linux encourages combining many
small tools.

Example:

``` bash
ps -ef | grep ssh
```

-   `ps -ef` lists processes.
-   `grep ssh` filters only SSH-related processes.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson08
cd ~/chapter02/lesson08

cat > users.txt << EOF
Alice
Bob
Charlie
David
Emma
Alice
Bob
EOF
```

------------------------------------------------------------------------

# Basic Pipe

Count files in the current directory:

``` bash
ls | wc -l
```

------------------------------------------------------------------------

# Search Command Output

``` bash
ls -l | grep ".txt"
```

------------------------------------------------------------------------

# Multiple Pipes

``` bash
cat users.txt | sort | uniq
```

Pipeline flow:

``` text
users.txt
    │
    ▼
 cat
    │
    ▼
 sort
    │
    ▼
 uniq
```

------------------------------------------------------------------------

# Count Unique Values

``` bash
cat users.txt | sort | uniq | wc -l
```

------------------------------------------------------------------------

# Most Frequent Entries

``` bash
cat users.txt | sort | uniq -c | sort -nr
```

Example output:

``` text
2 Alice
2 Bob
1 Charlie
1 David
1 Emma
```

------------------------------------------------------------------------

# Practical Scenarios

## Find Markdown files

``` bash
ls | grep ".md"
```

## Count running processes

``` bash
ps -ef | wc -l
```

## Search system logs

``` bash
tail -100 application.log | grep ERROR
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Count files with `ls | wc -l`.
2.  Filter only `.txt` files.
3.  Sort the names in `users.txt`.
4.  Remove duplicates.
5.  Count unique names.
6.  Display the frequency table.

------------------------------------------------------------------------

# Challenge

Create a file with 50 random names.

Build pipelines to:

-   Remove duplicates
-   Count unique values
-   Display the most common names
-   Count the final results

------------------------------------------------------------------------

# Common Mistakes

-   Confusing `|` with `>`.
-   Forgetting that pipes use stdout only.
-   Creating unnecessary temporary files.

------------------------------------------------------------------------

# Best Practices

-   Prefer pipelines over temporary files.
-   Keep each command focused on one task.
-   Read pipelines from left to right.
-   Build complex pipelines one command at a time.

------------------------------------------------------------------------

# Interview Questions

1.  What does the pipe operator do?
2.  What is passed through a pipe?
3.  What is the difference between `|` and `>`?
4.  Why are pipes central to Unix?
5.  Build a pipeline to count unique entries in a file.

------------------------------------------------------------------------

# Summary

You learned how to connect Linux commands together using pipes, creating
efficient workflows that are simple, reusable, and powerful.

------------------------------------------------------------------------

# Cheat Sheet

``` text
ls | wc -l
ps -ef | grep ssh
cat users.txt | sort
cat users.txt | sort | uniq
cat users.txt | sort | uniq | wc -l
cat users.txt | sort | uniq -c | sort -nr
```

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-08-basic-pipe.png
-   lesson-08-grep-pipe.png
-   lesson-08-sort-uniq.png
-   lesson-08-frequency-table.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 09 -- Building Powerful Pipelines**

You'll combine `head`, `tail`, `grep`, `sort`, `uniq`, `wc`,
redirection, and pipes to solve real Linux administration tasks.
