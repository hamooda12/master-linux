# Chapter 05 --- Users, Groups & Permissions

# Lesson 4 --- Creating Users

## Overview

Linux administrators regularly create user accounts for developers,
system administrators, interns, service accounts, and automation. This
lesson introduces the standard tools used to create and initialize user
accounts.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Create user accounts.
-   Set passwords for new users.
-   Understand the difference between `useradd` and `adduser`.
-   Verify that a user was created successfully.
-   Understand the default home directory and login shell.

------------------------------------------------------------------------

# 1. Creating a User with useradd

Basic syntax:

``` bash
sudo useradd hamad
```

This creates the account, but on many distributions it **does not**
automatically create a home directory.

Create a home directory explicitly:

``` bash
sudo useradd -m hamad
```

Useful options:

``` bash
sudo useradd -m -s /bin/bash hamad
```

-   `-m` → Create the home directory.
-   `-s` → Set the login shell.

------------------------------------------------------------------------

# 2. Creating a User with adduser

On Debian and Ubuntu, `adduser` is a friendly interactive tool.

``` bash
sudo adduser hamad
```

It usually:

-   Creates the home directory.
-   Creates the user's primary group.
-   Prompts for a password.
-   Copies default files from `/etc/skel`.

------------------------------------------------------------------------

# 3. Setting or Changing a Password

``` bash
sudo passwd hamad
```

Follow the prompts to set the password.

------------------------------------------------------------------------

# 4. Verifying the User

Check the account:

``` bash
id hamad
```

View the home directory:

``` bash
ls -ld /home/hamad
```

Check the passwd database:

``` bash
grep "^hamad:" /etc/passwd
```

Example entry:

``` text
hamad:x:1001:1001:Hamad:/home/hamad:/bin/bash
```

Fields:

1.  Username
2.  Password placeholder
3.  UID
4.  GID
5.  Comment (GECOS)
6.  Home directory
7.  Login shell

------------------------------------------------------------------------

# Production Scenario

A new backend developer joins the company.

Tasks:

-   Create the account.
-   Create the home directory.
-   Set the password.
-   Verify the account.
-   Confirm the correct shell.

------------------------------------------------------------------------

# Best Practices

-   Create a home directory for human users.
-   Use a modern shell such as `/bin/bash`.
-   Verify every new account.
-   Avoid using the root account for daily work.

------------------------------------------------------------------------

# Hands-on Lab

Create a test user:

``` bash
sudo useradd -m linuxstudent
```

Set a password:

``` bash
sudo passwd linuxstudent
```

Verify:

``` bash
id linuxstudent
ls -ld /home/linuxstudent
grep "^linuxstudent:" /etc/passwd
```

Delete the account after the lab if desired:

``` bash
sudo userdel -r linuxstudent
```

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting `-m` with `useradd`.
-   Forgetting to set a password.
-   Assuming `useradd` and `adduser` behave identically.

------------------------------------------------------------------------

# Interview Questions

1.  What is the difference between `useradd` and `adduser`?
2.  What does the `-m` option do?
3.  Which file stores basic user account information?
4.  How do you verify that a user exists?

------------------------------------------------------------------------

# Summary

You learned how to:

-   Create users.
-   Create home directories.
-   Set passwords.
-   Verify user accounts.
-   Understand the fields in `/etc/passwd`.

------------------------------------------------------------------------

# Next Lesson

**Lesson 8 --- Managing Users**
