# Chapter 02 -- Lesson 02 (Revised)

# Reading Large Files

## New Additions

### Explicit `-n` Syntax

The following commands are equivalent:

``` bash
head -5 file.txt
head -n 5 file.txt
```

``` bash
tail -20 file.txt
tail -n 20 file.txt
```

Using `-n` is often preferred in scripts because it is more explicit.

## Best Practice

-   Use `head -n` and `tail -n` in automation.
-   Use `tail -f` to monitor live log files.

These additions complement the original lesson.
