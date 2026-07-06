# Chapter 05 --- Users, Groups & Permissions

# Lesson 10 --- Access Control Lists (ACL)

## Overview

Traditional Linux permissions allow access to be assigned only to the
**owner**, **group**, and **others**. Sometimes this is not flexible
enough. Access Control Lists (ACLs) let you grant permissions to
specific users or groups without changing the file owner or primary
group.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Explain why ACLs exist.
-   View ACL entries.
-   Grant and remove ACL permissions.
-   Configure default ACLs on directories.
-   Troubleshoot ACL-related permission issues.

------------------------------------------------------------------------

# Why Use ACL?

Suppose:

-   Owner: `hamad`
-   Group: `developers`

You want **saeed** to edit one file without changing its owner or group.

Traditional permissions cannot do this cleanly.

ACLs solve the problem.

------------------------------------------------------------------------

# Viewing ACLs

Display ACL information:

``` bash
getfacl report.txt
```

Example:

``` text
# file: report.txt
# owner: hamad
# group: developers
user::rw-
user:saeed:rw-
group::r--
mask::rw-
other::---
```

------------------------------------------------------------------------

# Granting Permissions

Give `saeed` read/write access:

``` bash
sudo setfacl -m u:saeed:rw report.txt
```

Grant a group access:

``` bash
sudo setfacl -m g:qa:r report.txt
```

Verify:

``` bash
getfacl report.txt
```

------------------------------------------------------------------------

# Removing ACL Entries

Remove a user's ACL:

``` bash
sudo setfacl -x u:saeed report.txt
```

Remove a group's ACL:

``` bash
sudo setfacl -x g:qa report.txt
```

Remove all extended ACLs:

``` bash
sudo setfacl -b report.txt
```

------------------------------------------------------------------------

# Default ACLs

Apply a default ACL to a directory so new files inherit it.

``` bash
sudo setfacl -d -m u:saeed:rwx shared/
```

View it:

``` bash
getfacl shared/
```

Default entries begin with:

``` text
default:
```

------------------------------------------------------------------------

# ACL Mask

The ACL **mask** defines the maximum effective permissions for named
users and groups.

Example:

``` text
mask::r--
```

Even if a user has:

``` text
user:saeed:rw-
```

the effective permission becomes read-only because of the mask.

------------------------------------------------------------------------

# Production Scenario

The Finance department owns:

``` text
/reports
```

The security team needs read-only access to one report.

Instead of changing ownership or group:

``` bash
sudo setfacl -m u:security:r reports/payroll.xlsx
```

No other permissions are affected.

------------------------------------------------------------------------

# Hands-on Lab

Create a test directory:

``` bash
mkdir acl-lab
touch acl-lab/report.txt
```

Grant access:

``` bash
sudo setfacl -m u:$USER:rw acl-lab/report.txt
```

Inspect:

``` bash
getfacl acl-lab/report.txt
```

Remove the ACL:

``` bash
sudo setfacl -x u:$USER acl-lab/report.txt
```

Reset:

``` bash
sudo setfacl -b acl-lab/report.txt
```

------------------------------------------------------------------------

# Best Practices

-   Use ACLs only when traditional permissions are insufficient.
-   Document important ACLs.
-   Audit ACLs periodically.
-   Remember that ACLs add complexity.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to verify ACLs with `getfacl`.
-   Ignoring the ACL mask.
-   Using ACLs where simple group permissions would be enough.

------------------------------------------------------------------------

# Interview Questions

1.  Why were ACLs introduced?
2.  What is the difference between `chmod` and `setfacl`?
3.  How do you remove all ACL entries from a file?
4.  What is a default ACL?
5.  What does the ACL mask control?

------------------------------------------------------------------------

# Quick Reference

  Task              Command
  ----------------- -------------------------------
  View ACL          `getfacl file`
  Add user ACL      `setfacl -m u:user:rw file`
  Add group ACL     `setfacl -m g:group:r file`
  Remove ACL        `setfacl -x u:user file`
  Remove all ACLs   `setfacl -b file`
  Default ACL       `setfacl -d -m ... directory`

------------------------------------------------------------------------

# Summary

You learned how ACLs extend the traditional Linux permission model by
granting fine-grained access to specific users and groups without
changing ownership.

------------------------------------------------------------------------

# Next Lesson

**Lesson 14 --- sudo and visudo**
