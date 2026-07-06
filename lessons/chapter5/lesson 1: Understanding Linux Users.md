# Chapter 05 --- Users, Groups & Permissions

# Lesson 1 --- Understanding Linux Users

## Learning Objectives

By the end of this lesson, you will be able to:

-   Explain what a Linux user is.
-   Distinguish between different types of users.
-   Understand why Linux is called a multi-user operating system.
-   Interpret the output of `whoami`, `id`, `who`, `w`, `users`, and
    `logname`.
-   Understand the concepts of **UID**, **GID**, **home directory**, and
    **login shell**.

------------------------------------------------------------------------

# What Is a Linux User?

A **Linux user** is an identity that the operating system uses to
determine:

-   Who is running a command.
-   Which files they can access.
-   Which programs they are allowed to execute.
-   What resources they can use.

Think of a user as a **digital identity**. Linux doesn't care whether
the identity belongs to a human or a program---it treats both as users.

------------------------------------------------------------------------

# Why Does Linux Use Users?

On a production server there may be many people and services running
simultaneously. Linux isolates them using separate user accounts so each
has only the permissions it needs.

------------------------------------------------------------------------

# Linux Is a Multi-User Operating System

At the same time, a server may have:

-   Alice editing code
-   Bob deploying applications
-   Charlie checking logs
-   Nginx serving requests
-   MySQL handling the database
-   Spring Boot processing APIs

Each runs under a separate identity.

------------------------------------------------------------------------

# Types of Linux Users

## 1. Root User

The superuser.

Capabilities:

-   Full system access
-   Install/remove software
-   Manage users
-   Manage services
-   Change ownership and permissions

Use carefully.

------------------------------------------------------------------------

## 2. Regular Users

Examples:

-   hamad
-   alice
-   john

Characteristics:

-   Have a home directory
-   Own personal files
-   Use `sudo` when administrative privileges are required

------------------------------------------------------------------------

## 3. System Users (Service Accounts)

Examples:

-   www-data
-   nginx
-   mysql
-   postgres
-   systemd-network
-   nobody

These accounts usually cannot log in interactively and exist only to run
services securely.

------------------------------------------------------------------------

# Key Concepts

## Username

Human-readable account name.

Example:

    hamad

## UID (User ID)

Linux internally identifies users by numbers.

Example:

    hamad -> UID 1000

## GID (Group ID)

Every user belongs to at least one group.

## Home Directory

Examples:

    /home/hamad
    /home/alice

## Login Shell

Examples:

    /bin/bash
    /bin/zsh
    /bin/sh

------------------------------------------------------------------------

# Principle of Least Privilege

Services should run with only the permissions they actually need. For
example, a Spring Boot application should run as its own service account
instead of `root`.

------------------------------------------------------------------------

# Commands Introduced

  Command     Description
  ----------- -----------------------------------------
  `whoami`    Show current username
  `id`        Show UID, GID, and groups
  `who`       Show logged-in users
  `w`         Show logged-in users and their activity
  `users`     List logged-in usernames
  `logname`   Show original login name

------------------------------------------------------------------------

# Summary

You learned:

-   What a Linux user is
-   Why Linux uses users
-   Types of Linux users
-   UID and GID
-   Home directories
-   Login shells
-   The six basic user information commands

------------------------------------------------------------------------

# Next Lesson

**Lesson 2 --- Understanding Linux Groups**
