# Chapter 01 --- Lesson 11: Command History and Tab Completion

## Overview

The Linux terminal provides features that dramatically improve
productivity. Instead of retyping commands or full file paths, you can
reuse previous commands and let the shell automatically complete
filenames and directories.

Learning these features will make your work faster, more accurate, and
less repetitive.

------------------------------------------------------------------------

# Learning Objectives

After completing this lesson, you will be able to:

-   View command history
-   Re-run previous commands
-   Search command history
-   Use Tab completion
-   Increase productivity in the terminal

------------------------------------------------------------------------

# Prerequisites

-   Basic terminal navigation
-   Working with files and directories
-   Using common Linux commands

------------------------------------------------------------------------

# Theory

Every command you execute in the shell is stored in your command
history. The shell also helps complete filenames and directories
automatically using the **Tab** key.

These features save time and reduce typing mistakes.

------------------------------------------------------------------------

# Command: `history`

Display your command history.

## Syntax

``` bash
history
```

## Example

``` text
1  pwd
2  ls
3  cd Documents
4  mkdir project
5  touch notes.txt
```

Each command receives a history number.

------------------------------------------------------------------------

# Re-run a Command

Run a command again using its history number.

``` bash
!4
```

This executes command number 4.

Run the previous command again:

``` bash
!!
```

------------------------------------------------------------------------

# Search Command History

Press:

``` text
Ctrl + R
```

Begin typing part of a previous command.

Example:

``` text
(reverse-i-search)`mkdir':
```

Press **Enter** to execute the selected command.

------------------------------------------------------------------------

# Tab Completion

The shell can automatically complete filenames and directories.

Example:

Instead of typing:

``` bash
cd Documents
```

Type:

``` bash
cd Doc
```

Then press:

``` text
Tab
```

The shell completes the directory name if the match is unique.

------------------------------------------------------------------------

## Multiple Matches

If more than one match exists:

``` text
Documents
Downloads
```

Typing:

``` bash
cd Do
```

and pressing **Tab** twice displays all possible matches.

------------------------------------------------------------------------

# Practical Scenario

You are working inside a large project.

Instead of typing a long path repeatedly:

``` bash
cd ~/linux-practice/projects/backend/src/config
```

Type:

``` bash
cd ~/lin
```

Press **Tab** until the shell completes the remaining path.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
pwd

ls

mkdir history-demo

cd history-demo

touch file1.txt

history

!!

!3
```

Practice using:

-   Up Arrow
-   Down Arrow
-   Ctrl + R
-   Tab

------------------------------------------------------------------------

# Challenge

1.  Execute at least ten commands.
2.  Display your history.
3.  Re-run one command using `!number`.
4.  Search for a command using `Ctrl + R`.
5.  Use Tab completion to enter a directory.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] I can display command history.
-   [ ] I can repeat a command.
-   [ ] I can search history.
-   [ ] I can use Tab completion.
-   [ ] I understand how these features improve productivity.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting that history numbers change over time.
-   Running `!number` without checking which command it references.
-   Expecting Tab to complete when multiple matches exist.

------------------------------------------------------------------------

# Best Practices

-   Use Tab instead of typing long paths.
-   Use `Ctrl + R` to locate complex commands.
-   Review command history before repeating destructive commands.

------------------------------------------------------------------------

# Interview Questions

1.  What does the `history` command do?
2.  How do you repeat the previous command?
3.  What is `Ctrl + R` used for?
4.  Why is Tab completion useful?
5.  What happens when multiple filenames match a Tab completion?

------------------------------------------------------------------------

# Lesson Reflection

-   Can I reduce repetitive typing?
-   Do I know how to find commands I've already used?
-   Can I navigate the terminal more efficiently?

------------------------------------------------------------------------

# Summary

Command history and Tab completion are productivity features built into
the Linux shell. Mastering them saves time, reduces typing errors, and
improves your overall terminal workflow.

------------------------------------------------------------------------

# Lab Assets

Store screenshots in:

``` text
assets/chapter-01/
```

  -----------------------------------------------------------------------
  Filename                            Capture
  ----------------------------------- -----------------------------------
  `41-history-command.png`            Output of the `history` command

  `42-history-repeat.png`             Re-running a command with `!!` or
                                      `!number`

  `43-history-search.png`             Reverse search using `Ctrl + R`

  `44-tab-completion.png`             Demonstrating Tab completion

  `45-history-lab-complete.png`       Final terminal after completing the
                                      lab
  -----------------------------------------------------------------------
