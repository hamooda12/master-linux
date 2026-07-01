# Linux Cheat Sheet

> Personal Linux reference built during my Linux Practice journey.

------------------------------------------------------------------------

# File Viewing

  Command         Description
  --------------- ---------------------
  `cat file`      Display entire file
  `cat -n file`   Number lines
  `more file`     View page by page
  `less file`     Interactive viewer

------------------------------------------------------------------------

# Large Files

  Command           Description
  ----------------- ---------------------
  `head file`       First 10 lines
  `head -20 file`   First 20 lines
  `tail file`       Last 10 lines
  `tail -50 file`   Last 50 lines
  `tail -f file`    Follow live updates

------------------------------------------------------------------------

# Searching

  Command               Description
  --------------------- ------------------
  `grep text file`      Search text
  `grep -i text file`   Ignore case
  `grep -n text file`   Line numbers
  `grep -v text file`   Invert match
  `grep -c text file`   Count matches
  `grep -r text .`      Recursive search

------------------------------------------------------------------------

# Counting

  Command        Description
  -------------- ---------------------
  `wc file`      Lines, words, bytes
  `wc -l file`   Lines
  `wc -w file`   Words
  `wc -m file`   Characters
  `wc -c file`   Bytes

------------------------------------------------------------------------

# Sorting

  Command                              Description
  ------------------------------------ ----------------------------
  `sort file`                          Alphabetical sort
  `sort -r file`                       Reverse sort
  `sort -n file`                       Numeric sort
  `uniq file`                          Remove adjacent duplicates
  `uniq -c file`                       Count duplicates
  `sort file \| uniq -c \| sort -nr`   Frequency table

------------------------------------------------------------------------

# Comparing Files

  Command         Description
  --------------- -------------------------
  `diff a b`      Line-by-line comparison
  `diff -y a b`   Side-by-side
  `cmp a b`       Byte-by-byte comparison

------------------------------------------------------------------------

# Redirection

  Operator   Purpose
  ---------- --------------------------
  `>`        Overwrite output
  `>>`       Append output
  `<`        Redirect input
  `2>`       Redirect errors
  `2>>`      Append errors
  `&>`       Redirect output + errors

------------------------------------------------------------------------

# Pipes

``` bash
command1 | command2
```

Examples:

``` bash
ls | wc -l
ps -ef | grep ssh
grep ERROR app.log | wc -l
sort file | uniq
sort file | uniq -c | sort -nr
```
