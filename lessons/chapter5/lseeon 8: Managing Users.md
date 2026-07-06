# Chapter 05 --- Users, Groups & Permissions

# Lesson 8 --- Managing Users

## Overview

Creating a user is only the beginning. In real Linux administration, you
also need to modify users, add them to groups, change their shell, lock
accounts, unlock accounts, move home directories, and delete users
safely.

This lesson focuses on managing existing Linux users using `usermod`,
`passwd`, and `userdel`.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson, you will be able to:

-   Modify an existing user account.
-   Change a user's login shell.
-   Add a user to supplementary groups.
-   Lock and unlock a user account.
-   Delete users safely.
-   Understand when to keep or remove a user's home directory.

------------------------------------------------------------------------

# 1. Viewing a User Before Changing It

Before modifying a user, always inspect the current state.

``` bash
id saeed
```

Check the account entry:

``` bash
grep "^saeed:" /etc/passwd
```

Example:

``` text
saeed:x:1002:1002::/home/saeed:/bin/bash
```

Meaning:

  Field           Meaning
  --------------- -------------------------------------
  `saeed`         Username
  `x`             Password is stored in `/etc/shadow`
  `1002`          UID
  `1002`          Primary GID
  empty field     Comment / GECOS
  `/home/saeed`   Home directory
  `/bin/bash`     Login shell

------------------------------------------------------------------------

# 2. Changing a User's Shell

If a user was created with the wrong shell, change it with:

``` bash
sudo usermod -s /bin/bash saeed
```

Verify:

``` bash
grep "^saeed:" /etc/passwd
```

------------------------------------------------------------------------

# 3. Adding a User to a Group

To add a user to a supplementary group:

``` bash
sudo usermod -aG developers saeed
```

Important:

-   `-G` sets supplementary groups.
-   `-a` means append.
-   Always use `-aG` together when adding a user to a group.

Without `-a`, you may accidentally remove the user from other
supplementary groups.

Verify:

``` bash
id saeed
```

------------------------------------------------------------------------

# 4. Changing a User's Home Directory

Change the home directory path:

``` bash
sudo usermod -d /home/newsaeed saeed
```

Move the existing home directory too:

``` bash
sudo usermod -d /home/newsaeed -m saeed
```

Meaning:

-   `-d` sets the new home directory.
-   `-m` moves the content from the old home directory to the new one.

------------------------------------------------------------------------

# 5. Locking a User Account

To lock a user's password:

``` bash
sudo passwd -l saeed
```

This prevents password login.

Verify:

``` bash
sudo passwd -S saeed
```

You can also inspect `/etc/shadow`:

``` bash
sudo grep "^saeed:" /etc/shadow
```

A locked password usually has `!` before the password hash.

------------------------------------------------------------------------

# 6. Unlocking a User Account

To unlock the account:

``` bash
sudo passwd -u saeed
```

Verify:

``` bash
sudo passwd -S saeed
```

------------------------------------------------------------------------

# 7. Expiring a User Account Immediately

Force an account to expire:

``` bash
sudo usermod -e 1 saeed
```

This makes the account expired.

Set a future expiration date:

``` bash
sudo usermod -e 2026-12-31 saeed
```

Remove expiration:

``` bash
sudo usermod -e "" saeed
```

------------------------------------------------------------------------

# 8. Renaming a User

Rename a user:

``` bash
sudo usermod -l newname oldname
```

Example:

``` bash
sudo usermod -l saeed2 saeed
```

Important: this changes the username, but it does not automatically
rename the home directory unless you also move it.

Example:

``` bash
sudo usermod -l saeed2 -d /home/saeed2 -m saeed
```

------------------------------------------------------------------------

# 9. Deleting a User

Delete the user account only:

``` bash
sudo userdel saeed
```

This removes the account but usually keeps the home directory.

Delete the user and home directory:

``` bash
sudo userdel -r saeed
```

Be careful with `-r` because it deletes the user's files.

------------------------------------------------------------------------

# Production Scenario

A developer named `saeed` left the company.

Good process:

1.  Lock the account immediately.
2.  Check files owned by the user.
3.  Transfer important files if needed.
4.  Remove the user from groups.
5.  Delete the account only after verification.

Example:

``` bash
sudo passwd -l saeed
```

Find files owned by the user:

``` bash
sudo find / -user saeed 2>/dev/null
```

Delete safely later:

``` bash
sudo userdel saeed
```

------------------------------------------------------------------------

# Best Practices

-   Always inspect the user before modifying it.
-   Use `usermod -aG`, not only `-G`, when adding groups.
-   Lock accounts before deleting them in production.
-   Do not delete home directories until you verify important data.
-   Use `passwd -S` to check account password status.
-   Document why an account was locked or removed.

------------------------------------------------------------------------

# Hands-on Lab

Create a test user:

``` bash
sudo useradd -m -s /bin/bash testuser
sudo passwd testuser
```

Create a group:

``` bash
sudo groupadd developers
```

Add the user to the group:

``` bash
sudo usermod -aG developers testuser
```

Verify:

``` bash
id testuser
```

Lock the user:

``` bash
sudo passwd -l testuser
sudo passwd -S testuser
```

Unlock the user:

``` bash
sudo passwd -u testuser
sudo passwd -S testuser
```

Delete the test user:

``` bash
sudo userdel -r testuser
```

------------------------------------------------------------------------

# Common Mistakes

## Mistake 1 --- Forgetting `-a` with `usermod -G`

Wrong:

``` bash
sudo usermod -G docker saeed
```

This may remove the user from other supplementary groups.

Correct:

``` bash
sudo usermod -aG docker saeed
```

------------------------------------------------------------------------

## Mistake 2 --- Deleting a User Too Quickly

Wrong:

``` bash
sudo userdel -r saeed
```

without checking files first.

Better:

``` bash
sudo passwd -l saeed
sudo find / -user saeed 2>/dev/null
sudo userdel saeed
```

------------------------------------------------------------------------

## Mistake 3 --- Changing the Home Directory Without Moving Files

Wrong:

``` bash
sudo usermod -d /home/newsaeed saeed
```

Better:

``` bash
sudo usermod -d /home/newsaeed -m saeed
```

------------------------------------------------------------------------

# Interview Questions

1.  What does `usermod -aG` do?
2.  Why is `-a` important when using `-G`?
3.  How do you lock a user account?
4.  What is the difference between `userdel saeed` and
    `userdel -r saeed`?
5.  How do you change a user's shell?
6.  How do you move a user's home directory safely?
7.  What should you check before deleting a user in production?

------------------------------------------------------------------------

# Quick Command Reference

  Task                    Command
  ----------------------- -----------------------------------------
  View user info          `id username`
  View passwd entry       `grep "^username:" /etc/passwd`
  Change shell            `sudo usermod -s /bin/bash username`
  Add to group            `sudo usermod -aG group username`
  Change home directory   `sudo usermod -d /new/home username`
  Move home directory     `sudo usermod -d /new/home -m username`
  Lock user               `sudo passwd -l username`
  Unlock user             `sudo passwd -u username`
  Check password status   `sudo passwd -S username`
  Expire account          `sudo usermod -e YYYY-MM-DD username`
  Rename user             `sudo usermod -l newname oldname`
  Delete user only        `sudo userdel username`
  Delete user and home    `sudo userdel -r username`

------------------------------------------------------------------------

# Summary

You learned how to:

-   Modify user accounts.
-   Change login shells.
-   Add users to groups safely.
-   Move home directories.
-   Lock and unlock accounts.
-   Expire accounts.
-   Rename users.
-   Delete users safely.
-   Avoid dangerous production mistakes.

------------------------------------------------------------------------

# Next Lesson

**Lesson 9 --- Managing Groups**
