# Chapter 01 --- Lesson 10: Wildcards (Globbing)

## Overview

Typing every filename manually becomes impractical as the number of
files grows. Linux provides **wildcards**, also called **globbing
patterns**, which allow a single command to match multiple files or
directories.

Wildcards are supported by many Linux commands such as `ls`, `cp`, `mv`,
`rm`, and `find`.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   Understand wildcard patterns
-   Use `*`, `?`, and `[]`
-   Match files by extension
-   Match files with specific names
-   Apply wildcards safely with common Linux commands

------------------------------------------------------------------------

# Prerequisites

-   Filesystem navigation
-   Creating files and directories
-   Listing files with `ls`

------------------------------------------------------------------------

# Theory

Instead of typing every filename individually, you can describe a
pattern and let the shell expand it into matching filenames before the
command runs.

Example:

``` text
notes.txt
report.txt
todo.txt
```

Instead of writing:

``` bash
cat notes.txt report.txt todo.txt
```

You can write:

``` bash
cat *.txt
```

------------------------------------------------------------------------

# Wildcard: `*`

The asterisk (`*`) matches **zero or more characters**.

## Examples

List all Markdown files:

``` bash
ls *.md
```

List all text files:

``` bash
ls *.txt
```

List everything beginning with `project`:

``` bash
ls project*
```

------------------------------------------------------------------------

# Wildcard: `?`

The question mark (`?`) matches **exactly one character**.

Example files:

``` text
a.txt
b.txt
ab.txt
```

Command:

``` bash
ls ?.txt
```

Matches:

``` text
a.txt
b.txt
```

------------------------------------------------------------------------

# Wildcard: `[]`

Square brackets match one character from a specified set.

Example:

``` bash
ls file[123].txt
```

Matches:

``` text
file1.txt
file2.txt
file3.txt
```

You can also specify ranges:

``` bash
ls file[1-5].txt
```

Or letters:

``` bash
ls report[a-c].txt
```

------------------------------------------------------------------------

# Practical Scenario

Your project directory contains:

``` text
README.md
notes.txt
main.py
config.yml
script.py
```

Display only Python files:

``` bash
ls *.py
```

Copy all Markdown files:

``` bash
cp *.md docs/
```

------------------------------------------------------------------------

# Hands-on Lab

``` bash
cd ~/linux-practice

mkdir wildcard-demo

cd wildcard-demo

touch file1.txt file2.txt file3.txt

touch notes.md

touch app.py

touch report.pdf

ls

ls *.txt

ls *.md

ls *.py

ls file?.txt

ls file[1-2].txt
```

------------------------------------------------------------------------

# Challenge

1.  Create five files with different extensions.
2.  Display only `.txt` files.
3.  Display only `.py` files.
4.  Display files beginning with `file`.
5.  Display only `file1.txt` and `file2.txt` using brackets.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I understand `*`.
-   [ ] I understand `?`.
-   [ ] I understand `[]`.
-   [ ] I can filter files by extension.
-   [ ] I know that the shell expands wildcards before running commands.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting that wildcards are expanded by the shell.
-   Running `rm *.txt` without checking the matching files first.
-   Expecting `?` to match multiple characters.

------------------------------------------------------------------------

# Best Practices

-   Preview matches with `ls` before using `cp`, `mv`, or `rm`.
-   Quote wildcard patterns only when you want to prevent expansion.
-   Use descriptive filenames to make wildcard matching easier.

------------------------------------------------------------------------

# Interview Questions

1.  What does `*` match?
2.  What does `?` match?
3.  What is the purpose of `[]`?
4.  Which Linux commands commonly use wildcards?
5.  Why should you verify wildcard matches before deleting files?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I use wildcards confidently?
-   Do I understand the difference between `*` and `?`?
-   Can I simplify repetitive file operations?

------------------------------------------------------------------------

# Summary

Wildcards are a powerful feature of the Linux shell. By mastering `*`,
`?`, and `[]`, you can work with groups of files efficiently and reduce
repetitive typing.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -----------------------------------------------------------------------
  Filename                            Capture
  ----------------------------------- -----------------------------------
  `36-wildcard-all-txt.png`           Output of `ls *.txt`

  `37-wildcard-python.png`            Output of `ls *.py`

  `38-wildcard-question.png`          Output of `ls file?.txt`

  `39-wildcard-brackets.png`          Output of `ls file[1-2].txt`

  `40-wildcard-lab-complete.png`      Final terminal after completing the
                                      lab
  -----------------------------------------------------------------------
