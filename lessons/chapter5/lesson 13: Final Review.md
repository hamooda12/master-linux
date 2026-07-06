# Chapter 05 --- Final Review

## Objectives

Review every major concept from Chapter 05 before starting the practical
lab and bootcamp.

------------------------------------------------------------------------

# Lesson Checklist

  Topic                     Status
  ------------------------ --------
  Linux Users                 ✅
  Linux Groups                ✅
  Viewing Permissions         ✅
  Creating Users              ✅
  Managing Users              ✅
  Managing Groups             ✅
  Password Management         ✅
  Account Database Files      ✅
  umask                       ✅
  ACL                         ✅
  sudo & visudo               ✅
  File Attributes             ✅

------------------------------------------------------------------------

# Essential Commands

## User Information

``` bash
whoami
id
groups
getent passwd
getent group
```

## User Management

``` bash
useradd
adduser
usermod
userdel
passwd
chage
```

## Group Management

``` bash
groupadd
groupmod
groupdel
gpasswd
```

## Permission Inspection

``` bash
ls -l
stat
namei -l
```

## ACL

``` bash
getfacl
setfacl
```

## Sudo

``` bash
sudo
sudo -l
sudo -k
visudo
```

## File Attributes

``` bash
lsattr
chattr
```

------------------------------------------------------------------------

# Important Files

-   `/etc/passwd`
-   `/etc/shadow`
-   `/etc/group`
-   `/etc/gshadow`
-   `/etc/sudoers`

------------------------------------------------------------------------

# Concepts You Must Know

-   Root vs regular vs system users
-   Primary vs supplementary groups
-   UID and GID
-   Home directory
-   Login shell
-   Password aging
-   Password expiration vs account expiration
-   umask calculations
-   ACL vs traditional permissions
-   Principle of Least Privilege
-   Immutable and append-only attributes

------------------------------------------------------------------------

# Mini Self-Test

1.  Create a user with a home directory and Bash shell.
2.  Add the user to a supplementary group.
3.  Force a password change on next login.
4.  Display the user's UID and groups.
5.  Find the user's entry in `/etc/passwd`.
6.  Show the user's password policy.
7.  Configure `umask` to produce 640/750 permissions.
8.  Grant ACL read access to another user.
9.  Show your sudo permissions.
10. Protect a file with the immutable attribute.

If you can complete these without looking at notes, you're ready for the
practical lab.

------------------------------------------------------------------------

# Next Step

**Chapter 05 Practical Lab --- Build a Multi-User Linux Server**
