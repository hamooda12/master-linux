# Chapter 02 -- Lesson 07

# Input and Output Redirection

> **Topics Covered:** `>`, `>>`, `<`, `2>`, `2>>`, `&>`

------------------------------------------------------------------------

# Overview

Every Linux command communicates using three standard streams:

-   **stdin (0)** -- Standard Input
-   **stdout (1)** -- Standard Output
-   **stderr (2)** -- Standard Error

Redirection lets you control where these streams come from and where
they go. It is one of the most important Linux concepts because it forms
the foundation of shell scripting, automation, and DevOps workflows.

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Redirect command output to files
-   Append output without overwriting
-   Redirect input from a file
-   Separate errors from normal output
-   Redirect both output and errors together
-   Understand the three standard streams

------------------------------------------------------------------------

# Theory

By default:

``` text
Keyboard ──> stdin ──> Program ──> stdout ──> Terminal
                           │
                           └──────────────> stderr ──> Terminal
```

Redirection changes these destinations.

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson07
cd ~/chapter02/lesson07
```

------------------------------------------------------------------------

# Redirect Output (`>`)

Overwrite a file with command output.

``` bash
echo "Linux Practice" > notes.txt
```

Display the file:

``` bash
cat notes.txt
```

**Important:** If the file already exists, `>` replaces its contents.

------------------------------------------------------------------------

# Append Output (`>>`)

Add new data without removing existing content.

``` bash
echo "Lesson 07" >> notes.txt
```

------------------------------------------------------------------------

# Redirect Input (`<`)

Read input from a file.

``` bash
wc -l < notes.txt
```

------------------------------------------------------------------------

# Redirect Errors (`2>`)

Store only error messages.

``` bash
ls missing-file.txt 2> errors.log
```

View the error:

``` bash
cat errors.log
```

------------------------------------------------------------------------

# Append Errors (`2>>`)

Append error messages instead of overwriting.

``` bash
ls missing1.txt 2>> errors.log
ls missing2.txt 2>> errors.log
```

------------------------------------------------------------------------

# Redirect Both Output and Errors (`&>`)

Save everything to one file.

``` bash
ls notes.txt missing.txt &> results.log
```

------------------------------------------------------------------------

# Practical Scenario

Save successful output while keeping errors separate.

``` bash
find /etc -name "*.conf" > configs.txt 2> errors.txt
```

Now:

-   `configs.txt` contains matching files.
-   `errors.txt` contains permission errors.

------------------------------------------------------------------------

# Hands-on Lab

1.  Create `notes.txt` with `>`.
2.  Append three more lines using `>>`.
3.  Count lines using `<`.
4.  Generate an error with `ls missing.txt`.
5.  Save the error using `2>`.
6.  Save both output and errors using `&>`.

------------------------------------------------------------------------

# Challenge

Create a script of commands that:

-   Creates a report.
-   Appends system information.
-   Saves errors separately.
-   Produces one combined log.

------------------------------------------------------------------------

# Common Mistakes

-   Accidentally overwriting files with `>`.
-   Forgetting to use `>>` when appending.
-   Mixing normal output with errors.
-   Assuming `2>` captures normal output.

------------------------------------------------------------------------

# Best Practices

-   Use `>>` for log files.
-   Keep errors in separate files when troubleshooting.
-   Double-check filenames before using `>`.
-   Redirect output instead of copying from the terminal.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `>` and `>>`?
2.  What does `2>` redirect?
3.  What are stdin, stdout, and stderr?
4.  When would you use `<`?
5.  What does `&>` do?

------------------------------------------------------------------------

# Summary

You learned how to control command input, output, and error streams
using Linux redirection operators.

------------------------------------------------------------------------

# Cheat Sheet

``` text
command > file
command >> file
command < file
command 2> errors.log
command 2>> errors.log
command &> output.log
```

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-07-output-redirection.png
-   lesson-07-append.png
-   lesson-07-error-redirection.png
-   lesson-07-both-streams.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 08 -- Pipes (`|`)**

Learn how to connect commands together so the output of one command
becomes the input of another.
