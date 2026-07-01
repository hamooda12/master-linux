# Chapter 02 -- Lesson 06

# Comparing Files

> **Commands Covered:** `diff`, `cmp`

------------------------------------------------------------------------

# Overview

Comparing files is a common task for Linux administrators and
developers. Whether you are reviewing configuration changes, validating
backups, or comparing source code, Linux provides two classic tools:

-   `diff` -- Compare files line by line
-   `cmp` -- Compare files byte by byte

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will be able to:

-   Compare text files
-   Understand line-based vs byte-based comparison
-   Read `diff` output
-   Detect whether two files are identical
-   Use comparison tools in troubleshooting

------------------------------------------------------------------------

# Theory

Although both commands compare files, they serve different purposes.

  Command   Best For
  --------- ---------------------------------
  `diff`    Text files
  `cmp`     Binary or exact file comparison

------------------------------------------------------------------------

# Lab Setup

``` bash
mkdir -p ~/chapter02/lesson06
cd ~/chapter02/lesson06

cat > file1.txt << EOF
Apple
Banana
Orange
Mango
EOF

cat > file2.txt << EOF
Apple
Banana
Grapes
Mango
EOF
```

------------------------------------------------------------------------

# Command: diff

## Syntax

``` bash
diff file1.txt file2.txt
```

Example output:

``` text
3c3
< Orange
---
> Grapes
```

Meaning:

-   Line 3 changed.
-   `Orange` exists in the first file.
-   `Grapes` exists in the second.

------------------------------------------------------------------------

## Side-by-side Comparison

``` bash
diff -y file1.txt file2.txt
```

------------------------------------------------------------------------

## Ignore Letter Case

``` bash
diff -i file1.txt file2.txt
```

------------------------------------------------------------------------

## Ignore Blank Lines

``` bash
diff -B file1.txt file2.txt
```

------------------------------------------------------------------------

# Command: cmp

## Syntax

``` bash
cmp file1.txt file2.txt
```

If the files differ:

``` text
file1.txt file2.txt differ: byte 15, line 3
```

If they are identical, `cmp` produces no output and returns exit status
0.

------------------------------------------------------------------------

# When to Use

Use `diff` when:

-   Comparing configuration files
-   Reviewing source code
-   Understanding changes

Use `cmp` when:

-   Verifying copied files
-   Comparing binary files
-   Checking exact equality

------------------------------------------------------------------------

# Practical Scenario

Before replacing a production configuration file:

``` bash
diff old.conf new.conf
```

To verify a copied backup:

``` bash
cmp backup.conf original.conf
```

------------------------------------------------------------------------

# Hands-on Lab

1.  Compare `file1.txt` and `file2.txt`.
2.  Modify one file and compare again.
3.  Use side-by-side mode.
4.  Create identical files and test `cmp`.
5.  Change one character and rerun `cmp`.

------------------------------------------------------------------------

# Challenge

Create two configuration files with five differences.

Practice:

-   `diff`
-   `diff -y`
-   `diff -i`
-   `cmp`

Explain every reported difference.

------------------------------------------------------------------------

# Common Mistakes

-   Using `cmp` when a readable difference report is needed.
-   Forgetting that identical files produce no output with `cmp`.
-   Comparing the wrong file versions.

------------------------------------------------------------------------

# Best Practices

-   Use `diff` for text.
-   Use `cmp` for exact verification.
-   Review configuration changes before deployment.
-   Keep original backups before editing production files.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `diff` and `cmp`?
2.  Which command compares files line by line?
3.  What does `cmp` output for identical files?
4.  When would you use `diff -y`?
5.  Why is file comparison useful in system administration?

------------------------------------------------------------------------

# Summary

You learned how to compare files using both line-based and byte-based
methods and when each command is most appropriate.

------------------------------------------------------------------------

# Cheat Sheet

``` text
diff file1 file2
diff -y file1 file2
diff -i file1 file2
diff -B file1 file2

cmp file1 file2
```

------------------------------------------------------------------------

# Lab Assets

Save screenshots in:

``` text
assets/chapter-02/
```

Recommended captures:

-   lesson-06-diff.png
-   lesson-06-diff-side-by-side.png
-   lesson-06-cmp-identical.png
-   lesson-06-cmp-different.png

------------------------------------------------------------------------

# Next Lesson

**Lesson 07 -- Input and Output Redirection**

Topics:

-   `>`
-   `>>`
-   `<`
-   `2>`
-   `2>>`
-   `&>`
