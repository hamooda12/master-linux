# Chapter 05 --- Users, Groups & Permissions

# Lesson 2 --- Understanding Linux Groups

## Overview

Groups allow Linux administrators to manage permissions for multiple
users efficiently. Instead of assigning permissions user by user, you
assign permissions to a group and add users to that group.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson you will be able to:

-   Explain what a Linux group is.
-   Distinguish between a primary group and supplementary groups.
-   Understand why groups are essential in production environments.
-   Read group membership using `groups`, `id`, and `getent group`.

------------------------------------------------------------------------

# What Is a Group?

A **group** is a collection of users that share permissions.

Instead of giving the same permission to every individual user, Linux
allows you to assign permissions to a group.

Example:

    developers
    ├── alice
    ├── bob
    └── hamad

If the `developers` group owns a project directory, every member can
access it according to the group's permissions.

------------------------------------------------------------------------

# Why Do Groups Exist?

Imagine a development team with ten engineers.

Without groups:

-   You must assign permissions to each user individually.
-   Adding a new developer requires updating many files.
-   Removing a developer becomes time-consuming.

With groups:

-   Create one group.
-   Grant permissions once.
-   Add or remove users from the group.

------------------------------------------------------------------------

# Primary Group

Every Linux user has one **primary group**.

Example:

    User: hamad
    Primary Group: developers

Files created by the user normally inherit this primary group.

------------------------------------------------------------------------

# Supplementary Groups

A user may belong to multiple additional groups.

Example:

    hamad

    Primary:
    developers

    Supplementary:
    docker
    sudo
    git

This allows one user to collaborate across multiple teams.

------------------------------------------------------------------------

# Production Example

A company may organize users like this:

    developers
    qa
    designers
    devops
    database

Permissions are granted to groups instead of individuals.

------------------------------------------------------------------------

# Commands

## groups

Display the groups of the current user.

``` bash
groups
```

------------------------------------------------------------------------

## id

Display UID, GID, primary group, and supplementary groups.

``` bash
id
```

------------------------------------------------------------------------

## getent group

Display information about groups stored on the system.

``` bash
getent group
```

Example:

``` bash
getent group sudo
```

------------------------------------------------------------------------

# Best Practices

-   Assign permissions to groups instead of individual users.
-   Follow the Principle of Least Privilege.
-   Use meaningful group names.
-   Regularly review group memberships.

------------------------------------------------------------------------

# Hands-on Lab

1.  Run:

``` bash
groups
```

2.  Run:

``` bash
id
```

3.  Run:

``` bash
getent group
```

4.  Identify:

-   Your primary group.
-   Your supplementary groups.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing users with groups.
-   Granting permissions to every user individually.
-   Ignoring unnecessary group memberships.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between a user and a group?
2.  What is a primary group?
3.  What are supplementary groups?
4.  Why are groups preferred over individual permissions?

------------------------------------------------------------------------

# Summary

You learned:

-   What Linux groups are.
-   Why groups exist.
-   Primary vs. supplementary groups.
-   How groups simplify permission management.
-   How to inspect groups with `groups`, `id`, and `getent group`.

------------------------------------------------------------------------

# Next Lesson

**Lesson 4 --- Viewing Permissions Deeply**
