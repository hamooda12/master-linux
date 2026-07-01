# Chapter 02 -- Lesson 07 (Revised)

# Input, Output, Error Redirection & `/dev/null`

> **Topics Covered:** `>`, `>>`, `<`, `2>`, `2>>`, `2>&1`, `&>`,
> `/dev/null`, `tee`

------------------------------------------------------------------------

# Overview

Every Linux program communicates using three standard streams:

  Stream   Number   Purpose
  -------- -------- ---------------
  stdin    0        Input
  stdout   1        Normal output
  stderr   2        Error output

Redirection lets you control these streams.

------------------------------------------------------------------------

# Standard Streams

``` text
Keyboard
   │
   ▼
stdin (0)
   │
   ▼
 Program
  │    │
  │    └──── stderr (2)
  ▼
stdout (1)
```

------------------------------------------------------------------------

# Redirect Standard Output

Overwrite:

``` bash
echo "Linux" > notes.txt
```

Append:

``` bash
echo "Practice" >> notes.txt
```

------------------------------------------------------------------------

# Redirect Standard Input

``` bash
wc -l < notes.txt
```

------------------------------------------------------------------------

# Redirect Errors

``` bash
ls missing.txt 2> errors.log
```

Append errors:

``` bash
ls missing.txt 2>> errors.log
```

------------------------------------------------------------------------

# Redirect stdout and stderr Separately

``` bash
find /etc -name "*.conf" > configs.txt 2> errors.txt
```

------------------------------------------------------------------------

# Redirect stderr to stdout (`2>&1`)

Normally:

``` text
stdout → terminal
stderr → terminal
```

Combine both streams:

``` bash
find /etc -name "*.conf" > output.log 2>&1
```

Everything is written to `output.log`.

> Order matters. `> output.log 2>&1` is not the same as
> `2>&1 > output.log`.

------------------------------------------------------------------------

# Redirect Everything (`&>`)

``` bash
find /etc -name "*.conf" &> result.log
```

Equivalent to redirecting both stdout and stderr into one file.

------------------------------------------------------------------------

# The Special File: `/dev/null`

`/dev/null` is often called the **black hole** of Linux.

Anything written to it disappears permanently.

Discard normal output:

``` bash
command > /dev/null
```

Discard errors:

``` bash
command 2> /dev/null
```

Discard everything:

``` bash
command &> /dev/null
```

Or:

``` bash
command > /dev/null 2>&1
```

------------------------------------------------------------------------

# Why `/dev/null`?

Sometimes you only care whether a command succeeds.

Example:

``` bash
grep root /etc/passwd > /dev/null
```

No output is displayed, but the command's exit status is preserved.

------------------------------------------------------------------------

# Introducing `tee`

Normally:

``` bash
ls > files.txt
```

You lose terminal output.

With `tee`:

``` bash
ls | tee files.txt
```

The output is:

-   Displayed on screen
-   Saved into `files.txt`

Append with:

``` bash
ls | tee -a files.txt
```

------------------------------------------------------------------------

# Practical Scenarios

Save successful output while hiding permission errors:

``` bash
find /etc -name "*.conf" 2> /dev/null
```

Create a report while viewing it:

``` bash
grep ERROR app.log | tee error-report.txt
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Create `notes.txt` using `>`.
2.  Append data using `>>`.
3.  Save errors into `errors.log`.
4.  Combine stdout and stderr with `2>&1`.
5.  Discard output using `/dev/null`.
6.  Save and display output using `tee`.

------------------------------------------------------------------------

# Challenge

Write one command that:

-   Searches `/etc`
-   Saves results
-   Suppresses permission errors
-   Displays the final report

------------------------------------------------------------------------

# Common Mistakes

-   Accidentally overwriting files with `>`.
-   Forgetting that `2>&1` depends on order.
-   Assuming `/dev/null` stores data.
-   Using `>` instead of `tee` when you also need terminal output.

------------------------------------------------------------------------

# Best Practices

-   Use `>>` for log files.
-   Use `/dev/null` to silence unwanted output.
-   Use `tee` in scripts and troubleshooting.
-   Keep stdout and stderr separate while debugging.

------------------------------------------------------------------------

# Interview Questions

1.  What are stdin, stdout, and stderr?
2.  What does `2>&1` do?
3.  What is `/dev/null`?
4.  When would you use `tee`?
5.  What is the difference between `>` and `tee`?

------------------------------------------------------------------------

# Cheat Sheet

``` text
command > file
command >> file
command < file
command 2> errors.log
command 2>> errors.log
command > output.log 2>&1
command &> output.log
command > /dev/null
command 2> /dev/null
command &> /dev/null

command | tee file
command | tee -a file
```

------------------------------------------------------------------------

# Lab Assets

Capture:

-   lesson-07-redirection.png
-   lesson-07-dev-null.png
-   lesson-07-tee.png
-   lesson-07-stdout-stderr.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 08 -- Pipes (Revised)**
