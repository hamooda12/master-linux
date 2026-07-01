# Chapter 02 -- Lesson 01 Viewing Text Files

> **Commands Covered:** `cat`, `more`, `less`

------------------------------------------------------------------------

# Overview

Reading text files is one of the most common tasks performed by Linux
users, system administrators, developers, and DevOps engineers.

Linux stores configuration files, logs, scripts, documentation, and many
system files as plain text. Learning how to inspect these files
efficiently is a fundamental Linux skill.

This lesson introduces the three classic commands used to display file
contents:

-   `cat`
-   `more`
-   `less`

You'll learn not only how to use them, but **when** each one is the
right choice.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Explain what a text file is
-   Distinguish text files from binary files
-   Display file contents with `cat`
-   Read long files using `more`
-   Navigate large files efficiently using `less`
-   Choose the appropriate command for different situations

------------------------------------------------------------------------

# Prerequisites

You should have completed Chapter 01 and understand:

-   Basic filesystem navigation
-   Creating files and directories
-   Absolute and relative paths

------------------------------------------------------------------------

# Theory

## What is a Text File?

A text file stores human-readable characters rather than binary machine
data.

Examples:

``` text
/etc/passwd
/etc/hosts
README.md
notes.txt
script.sh
```

Most Linux configuration files are text files because they are easy to
inspect, edit, and version-control.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson01
cd ~/chapter02/lesson01

cat > fruits.txt << EOF
Apple
Banana
Orange
Mango
Grapes
EOF
```

------------------------------------------------------------------------

# Command: cat

## Purpose

Display the entire contents of one or more files.

## Syntax

``` bash
cat [OPTIONS] FILE...
```

## Example

``` bash
cat fruits.txt
```

Output

``` text
Apple
Banana
Orange
Mango
Grapes
```

## Common Options

  Option   Description
  -------- -----------------------------
  `-n`     Number all lines
  `-b`     Number non-empty lines
  `-E`     Show end-of-line marker `$`
  `-T`     Display tabs
  `-A`     Show all special characters

Example:

``` bash
cat -n fruits.txt
```

## When to Use

-   Small files
-   Quick inspection
-   Concatenating files
-   Redirecting output

## Limitations

Avoid using `cat` for very large log files because it prints everything
at once.

------------------------------------------------------------------------

# Command: more

## Purpose

Read large files one page at a time.

``` bash
more filename
```

### Navigation

  Key     Action
  ------- -----------
  Space   Next page
  Enter   Next line
  q       Quit

### Best For

Medium-sized files when simple forward navigation is sufficient.

------------------------------------------------------------------------

# Command: less

## Purpose

Read files interactively with forward and backward navigation.

``` bash
less filename
```

### Navigation

  Key        Action
  ---------- -------------------
  ↑ / ↓      Move one line
  Space      Next page
  b          Previous page
  g          Beginning of file
  G          End of file
  /pattern   Search
  n          Next match
  q          Quit

### Why "less" is Better

Unlike `more`, `less` supports:

-   Backward scrolling
-   Searching
-   Faster navigation
-   Better performance on huge files

------------------------------------------------------------------------

# Comparison

  Feature           cat   more      less
  ----------------- ----- --------- -----------
  Small files       ✅    ✅        ✅
  Huge files        ❌    ⚠️        ✅
  Search            ❌    ❌        ✅
  Scroll backward   ❌    ❌        ✅
  Interactive       ❌    Limited   Excellent

------------------------------------------------------------------------

# Practical Scenario

You receive a 2 GB application log.

-   ❌ `cat app.log`
-   ⚠️ `more app.log`
-   ✅ `less app.log`

Use `less` because it loads only the required portion of the file into
memory.

------------------------------------------------------------------------

# Hands-on Lab

1.  Display the file with `cat`.
2.  Number the lines using `cat -n`.
3.  Open the file with `more`.
4.  Exit using `q`.
5.  Open the file with `less`.
6.  Search for `Orange`.
7.  Jump to the end with `G`.
8.  Return to the beginning with `g`.

------------------------------------------------------------------------

# Challenge

Create a file named `animals.txt` containing ten animals.

-   Display it with `cat`
-   View it with `more`
-   Search for one animal using `less`

------------------------------------------------------------------------

# Common Mistakes

-   Using `cat` on very large files
-   Forgetting to quit `less` using `q`
-   Assuming `more` supports backward scrolling

------------------------------------------------------------------------

# Best Practices

-   Use `cat` for small files.
-   Use `less` for logs and configuration files.
-   Learn the keyboard shortcuts for `less`.
-   Avoid overwhelming the terminal with huge outputs.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `cat`, `more`, and `less`?
2.  Why is `less` preferred for log analysis?
3.  What does `cat -n` do?
4.  How do you search inside `less`?
5.  Which command would you choose for a multi-gigabyte log file?

------------------------------------------------------------------------

# Lesson Reflection

After this lesson you should be comfortable inspecting text files of any
size and selecting the most appropriate tool for the task.

------------------------------------------------------------------------

# Summary

You learned:

-   What text files are
-   How `cat` works
-   How `more` works
-   Why `less` is the preferred viewer
-   Basic navigation techniques
-   Real-world usage

------------------------------------------------------------------------

# Cheat Sheet

``` text
cat file.txt
cat -n file.txt
more file.txt
less file.txt

Inside less:
/text
n
g
G
b
Space
q
```

------------------------------------------------------------------------

# Lab Assets

Save these under:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-01-cat.png
-   lesson-01-cat-numbered.png
-   lesson-01-more.png
-   lesson-01-less-search.png
-   lesson-01-less-navigation.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 02 -- Reading Large Files**

Commands:

-   `head`
-   `tail`
-   `tail -f`
