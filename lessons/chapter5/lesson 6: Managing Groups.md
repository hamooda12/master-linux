# Chapter 05 --- Users, Groups & Permissions

# Lesson 6 --- Managing Groups

## Overview

Groups allow administrators to manage permissions efficiently by
assigning access to teams instead of individual users.

------------------------------------------------------------------------

# Learning Objectives

-   Create, rename, and delete groups.
-   Add and remove users from groups.
-   Inspect group membership.
-   Apply group management in production.

------------------------------------------------------------------------

# Viewing Groups

List all groups:

``` bash
getent group
```

View a specific group:

``` bash
getent group developers
```

Check a user's groups:

``` bash
groups hamad
id hamad
```

------------------------------------------------------------------------

# Creating a Group

``` bash
sudo groupadd developers
```

Verify:

``` bash
getent group developers
```

------------------------------------------------------------------------

# Renaming a Group

``` bash
sudo groupmod -n backend developers
```

------------------------------------------------------------------------

# Deleting a Group

``` bash
sudo groupdel backend
```

A primary group of an existing user usually cannot be deleted.

------------------------------------------------------------------------

# Managing Members

Add a user:

``` bash
sudo usermod -aG developers saeed
```

Remove a user:

``` bash
sudo gpasswd -d saeed developers
```

Verify:

``` bash
id saeed
```

------------------------------------------------------------------------

# Production Example

Create a `devops` group, add all DevOps engineers to it, and assign the
deployment directory to that group instead of granting permissions to
each user individually.

------------------------------------------------------------------------

# Hands-on Lab

``` bash
sudo groupadd developers
sudo groupadd qa

sudo usermod -aG developers $USER

id
getent group developers

sudo gpasswd -d $USER developers
```

------------------------------------------------------------------------

# Best Practices

-   Prefer group permissions over individual permissions.
-   Audit group membership regularly.
-   Remove users from groups they no longer need.

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting `-a` with `usermod -G`.
-   Deleting groups that are still in use.
-   Giving unnecessary group memberships.

------------------------------------------------------------------------

# Interview Questions

1.  What is the purpose of a Linux group?
2.  How do you create a group?
3.  How do you rename a group?
4.  How do you remove a user from a group?
5.  Why is `-a` required with `usermod -G`?

------------------------------------------------------------------------

# Quick Reference

  Task           Command
  -------------- ----------------
  Create group   `groupadd`
  Rename group   `groupmod -n`
  Delete group   `groupdel`
  Show groups    `getent group`
  Add user       `usermod -aG`
  Remove user    `gpasswd -d`

------------------------------------------------------------------------

# Summary

You can now create, modify, inspect, and safely manage Linux groups.

------------------------------------------------------------------------

# Next Lesson

**Lesson 10 --- Password Management**
