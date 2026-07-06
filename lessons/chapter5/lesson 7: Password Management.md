# Chapter 05 --- Users, Groups & Permissions

# Lesson 7 --- Password Management

## Overview

Passwords are a fundamental part of Linux account security. Besides
setting passwords, Linux allows administrators to enforce password
aging, expiration, account expiration, and password policies.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Set and change user passwords.
-   Lock and unlock password-based logins.
-   Configure password aging.
-   Force password changes.
-   Inspect password policies.
-   Differentiate password expiration from account expiration.

------------------------------------------------------------------------

# 1. Setting or Changing a Password

``` bash
sudo passwd saeed
```

This prompts for a new password.

------------------------------------------------------------------------

# 2. Viewing Password Status

``` bash
sudo passwd -S saeed
```

Example:

``` text
saeed P 2026-07-06 0 99999 7 -1
```

Common status values:

-   `P` = Password set
-   `L` = Password locked
-   `NP` = No password

------------------------------------------------------------------------

# 3. Locking and Unlocking Password Login

Lock:

``` bash
sudo passwd -l saeed
```

Unlock:

``` bash
sudo passwd -u saeed
```

Locking only disables password authentication. It does **not** delete
the account.

------------------------------------------------------------------------

# 4. Password Aging with chage

View current policy:

``` bash
sudo chage -l saeed
```

Force a password change on next login:

``` bash
sudo chage -d 0 saeed
```

Set maximum password age:

``` bash
sudo chage -M 90 saeed
```

Set minimum password age:

``` bash
sudo chage -m 7 saeed
```

Set warning days:

``` bash
sudo chage -W 14 saeed
```

------------------------------------------------------------------------

# 5. Account Expiration

Expire immediately:

``` bash
sudo usermod -e 1 saeed
```

Expire on a future date:

``` bash
sudo usermod -e 2026-12-31 saeed
```

Remove expiration:

``` bash
sudo usermod -e "" saeed
```

Password expiration and account expiration are different:

-   Password expiration → user must change the password.
-   Account expiration → login is no longer allowed.

------------------------------------------------------------------------

# 6. Inspecting /etc/shadow

Example:

``` text
saeed:$y$...:20545:0:90:14:::
```

Important fields:

-   Password hash
-   Last password change
-   Minimum age
-   Maximum age
-   Warning days
-   Account expiration

Never edit this file manually unless absolutely necessary.

------------------------------------------------------------------------

# Production Scenario

Company policy requires:

-   Password changes every 90 days.
-   Warning 14 days before expiration.
-   Immediate password change after a temporary password is assigned.

Commands:

``` bash
sudo chage -M 90 -W 14 developer
sudo chage -d 0 developer
```

------------------------------------------------------------------------

# Hands-on Lab

Create a test user:

``` bash
sudo useradd -m labuser
sudo passwd labuser
```

Inspect:

``` bash
sudo passwd -S labuser
sudo chage -l labuser
```

Force a password change:

``` bash
sudo chage -d 0 labuser
```

Lock and unlock:

``` bash
sudo passwd -l labuser
sudo passwd -u labuser
```

Delete the test user:

``` bash
sudo userdel -r labuser
```

------------------------------------------------------------------------

# Best Practices

-   Use strong passwords.
-   Enforce password aging for human accounts.
-   Lock accounts instead of deleting them immediately.
-   Review password policies regularly.
-   Use SSH keys where appropriate instead of passwords.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing password expiration with account expiration.
-   Unlocking an account that has no password configured.
-   Ignoring password policy requirements.

------------------------------------------------------------------------

# Interview Questions

1.  What does `passwd -S` show?
2.  What is the difference between `passwd -l` and `usermod -e`?
3.  How do you force a password change at the next login?
4.  What does `chage -l` display?
5.  Why is `/etc/shadow` protected?

------------------------------------------------------------------------

# Quick Reference

  Task                    Command
  ----------------------- ------------------------
  Set password            `passwd user`
  Show password status    `passwd -S user`
  Lock password           `passwd -l user`
  Unlock password         `passwd -u user`
  Show aging policy       `chage -l user`
  Force password change   `chage -d 0 user`
  Set max age             `chage -M DAYS user`
  Set warning             `chage -W DAYS user`
  Expire account          `usermod -e DATE user`

------------------------------------------------------------------------

# Summary

You learned how to manage passwords, configure password aging, lock and
unlock password logins, and distinguish password expiration from account
expiration.

------------------------------------------------------------------------

# Next Lesson

**Lesson 11 --- Understanding /etc/passwd, /etc/shadow, /etc/group, and
/etc/gshadow**
