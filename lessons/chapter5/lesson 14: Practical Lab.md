# Chapter 05 --- Practical Lab

# Build a Multi-User Linux Server

## Scenario

You are the new Linux System Administrator for **TechCorp**.

A fresh Ubuntu server has been installed. Your job is to prepare it for
a development team while following Linux security best practices.

------------------------------------------------------------------------

# Objectives

By the end of this lab you will be able to:

-   Create users and groups.
-   Configure passwords.
-   Manage supplementary groups.
-   Verify account information.
-   Configure `umask`.
-   Apply ACLs.
-   Use `sudo`.
-   Protect files with file attributes.

------------------------------------------------------------------------

# Lab Topology

    TechCorp

    Users
    ├── hamad
    ├── saeed
    ├── sara
    └── admin

    Groups
    ├── developers
    ├── qa
    └── devops

------------------------------------------------------------------------

# Tasks

## Task 1 --- Create Groups

``` bash
sudo groupadd developers
sudo groupadd qa
sudo groupadd devops
```

Verify:

``` bash
getent group
```

------------------------------------------------------------------------

## Task 2 --- Create Users

Create:

-   hamad
-   saeed
-   sara
-   admin

Requirements:

-   Home directory
-   Bash shell

Example:

``` bash
sudo useradd -m -s /bin/bash hamad
```

Set passwords for all users.

------------------------------------------------------------------------

## Task 3 --- Configure Membership

-   hamad → developers
-   saeed → developers
-   sara → qa
-   admin → developers + devops + sudo

Verify with:

``` bash
id username
```

------------------------------------------------------------------------

## Task 4 --- Password Policy

Configure for every human user:

-   Maximum age: 90 days
-   Warning: 14 days
-   Force password change at first login

------------------------------------------------------------------------

## Task 5 --- Shared Project

Create:

``` text
/srv/project
```

Requirements:

-   Owner: root
-   Group: developers
-   Group writable

------------------------------------------------------------------------

## Task 6 --- ACL

Give `sara` read-only access to the project directory without changing
ownership.

Verify using:

``` bash
getfacl
```

------------------------------------------------------------------------

## Task 7 --- umask

Temporarily set:

``` bash
umask 027
```

Create a file and verify the resulting permissions.

------------------------------------------------------------------------

## Task 8 --- sudo

Verify:

``` bash
sudo -l
```

Confirm the `admin` account belongs to the `sudo` group.

------------------------------------------------------------------------

## Task 9 --- Immutable File

Create:

``` text
/etc/company.conf
```

Protect it with:

``` bash
sudo chattr +i /etc/company.conf
```

Verify:

``` bash
lsattr /etc/company.conf
```

Try editing the file, then remove the attribute and edit it
successfully.

------------------------------------------------------------------------

# Verification Checklist

-   [ ] Users created
-   [ ] Groups created
-   [ ] Passwords configured
-   [ ] Group membership verified
-   [ ] Password aging configured
-   [ ] Shared directory created
-   [ ] ACL verified
-   [ ] umask tested
-   [ ] sudo verified
-   [ ] Immutable attribute tested

------------------------------------------------------------------------

# Challenge

Without looking at previous lessons:

1.  Create a new `intern` user.
2.  Add them to `developers`.
3.  Force a password change on first login.
4.  Give them read-only ACL access to `/srv/project`.
5.  Verify every step.

------------------------------------------------------------------------

# Completion

When you complete this lab successfully, you are ready for the **Chapter
05 Users & Permissions Bootcamp (30 Production Incidents).**
