# master-linux

Linux is an operating system inspired by Unix. Unix came first, but it was not free and open for everyone to use or modify. Because of that, Linux was created as a free and open-source system.

Linux is mainly built around the kernel. The kernel is the core part of the operating system. It manages the hardware resources such as CPU, memory, storage, and input/output devices.

Because Linux is open source, people can take the source code, modify it, add tools and software, and build their own Linux distribution. A Linux distribution is a complete operating system that includes the Linux kernel plus other software, utilities, libraries, and package managers. There are many Linux distributions, and one of the most famous is Ubuntu.

The basic Linux architecture includes several layers.

At the bottom, there is the hardware layer. This includes physical resources such as CPU, memory, storage devices, and input/output devices.

Above the hardware, there is the kernel. The kernel manages communication between software and hardware. It receives requests from programs and passes them to the hardware in a way the hardware can understand.

There are also libraries. Libraries help developers use system features more easily without interacting directly with the kernel. They provide ready-made functions that applications can use.

Finally, there are system utilities. These are command-line tools that help users manage, monitor, and configure the Linux system.

So, in simple words, Linux is an open-source Unix-like operating system built around the kernel, and different groups can package it with tools and software to create different Linux distributions.


Linux Filesystem Documentation
Understanding the Most Important Linux Directories

Linux organizes all files and directories under a single root directory (/). Each directory has a specific purpose, making the operating system organized, secure, and easy to maintain.

1. /home
Purpose

The /home directory stores personal files and settings for every regular user on the system.

Each user has their own directory inside /home.

Example:

/home/hamad
/home/alex
/home/john
What type of data is stored?
Documents
Downloads
Pictures
Videos
Desktop files
Projects
Source code
Personal application settings

Example:

/home/hamad/Documents
/home/hamad/Downloads
/home/hamad/Desktop
Why does Linux use /home?

Linux separates user data from system files.

This allows:

Safe operating system updates.
Multiple users on the same computer.
Easier backups.
Easier reinstallation without losing personal files.
What happens if you modify files here?

Usually nothing dangerous.

You can:

Create files
Delete files
Rename folders
Install projects

This is your personal workspace.

What happens if you delete it?

Deleting a user's home directory removes:

Personal documents
Downloads
Photos
Application settings

The operating system itself will continue to work normally.

2. /etc
Purpose

/etc contains system-wide configuration files.

Think of it as the Settings folder of Linux.

What type of data is stored?

Configuration files for:

Network
Users
Passwords
SSH
DNS
Web servers
Databases
Services
Startup configuration

Examples:

/etc/passwd
/etc/hosts
/etc/fstab
/etc/ssh/
Why does Linux use /etc?

Instead of hardcoding settings into programs, Linux stores configuration separately.

This allows administrators to customize the system without changing program code.

What happens if you modify it?

If you edit configuration correctly:

Change hostname
Configure SSH
Add users
Configure networking

If you edit incorrectly:

Network may stop working.
Services may fail to start.
SSH access may break.
Boot problems may occur.
What happens if you delete it?

Very dangerous.

Deleting important configuration files can make Linux partially or completely unusable.

3. /var
Purpose

/var stores data that changes frequently while Linux is running.

The name comes from Variable.

What type of data is stored?

Examples include:

Log files
Cache
Mail queues
Print jobs
Package manager data
Database files

Common directories:

/var/log
/var/cache
/var/lib
Why does Linux use /var?

Applications constantly generate data.

Instead of mixing changing data with system files, Linux stores it inside /var.

What happens if you modify it?

Depends on what you change.

Deleting:

/var/log

removes log history.

Deleting:

/var/cache

usually only removes cached files.

Deleting:

/var/lib

can destroy databases or application data.

What happens if you delete it?

Some applications may stop working.

Databases may lose data.

Package managers may fail.

Logs disappear permanently.

4. /tmp
Purpose

/tmp stores temporary files.

Applications use it while they are running.

What type of data is stored?

Examples:

Temporary downloads
Installation files
Temporary images
Cached processing files
Temporary editor files
Why does Linux use /tmp?

Programs often need temporary storage.

Instead of saving temporary files inside user folders, Linux places them in /tmp.

Many Linux distributions automatically clean this directory after reboot.

What happens if you modify it?

Usually nothing serious.

You can safely create or remove temporary files.

What happens if you delete it?

Normally safe.

However, deleting files currently being used by running applications may cause those applications to crash or behave unexpectedly.

5. /bin
Purpose

/bin contains essential executable programs (binary files) required by Linux.

These commands are available immediately after boot.

What type of data is stored?

Command-line programs such as:

ls
cp
mv
rm
cat
mkdir
echo
pwd
Why does Linux use /bin?

These commands are required for:

File management
Basic troubleshooting
System recovery

Without them, Linux would be difficult to operate.

What happens if you modify it?

Replacing system commands may cause unexpected behavior.

Deleting commands like:

/bin/ls

means you can no longer use:

ls
What happens if you delete it?

Extremely dangerous.

Linux may become unusable because many basic commands will disappear.

6. /usr
Purpose

/usr stores most installed software, libraries, documentation, and shared resources.

Although the name originally meant "User System Resources," today it mainly contains software provided by the operating system and installed packages.

What type of data is stored?

Examples include:

Programs
Libraries
Documentation
Icons
Fonts
Manuals

Common directories:

/usr/bin
/usr/lib
/usr/share
Why does Linux use /usr?

It keeps software organized and shared between all users.

Applications installed through package managers are usually stored here.

What happens if you modify it?

Deleting program files may prevent applications from running.

Removing shared libraries may break multiple applications simultaneously.

What happens if you delete it?

Linux will lose most installed software.

Many commands and graphical applications will stop working.

The system may become impossible to use.

7. /opt
Purpose

/opt stores optional third-party software that is not part of the core operating system.

What type of data is stored?

Examples:

Google Chrome
JetBrains IDEs
Oracle software
Commercial applications
Custom-installed software

Example:

/opt/google/
/opt/idea/
/opt/myapp/
Why does Linux use /opt?

It keeps manually installed software separate from the operating system.

This makes installation and removal easier.

What happens if you modify it?

Only the software inside that directory is affected.

The operating system itself is usually not impacted.

What happens if you delete it?

The corresponding application will stop working.

Linux itself will continue running normally.

Summary Table
Directory	Stores	Safe to Modify?	Risk if Deleted
/home	User files and personal settings	✅ Yes	Lose personal data
/etc	System configuration files	⚠️ Carefully	System configuration may break
/var	Logs, cache, databases, variable data	⚠️ Carefully	Services or databases may fail
/tmp	Temporary files	✅ Usually	Running applications may be affected
/bin	Essential system commands	❌ No	Basic Linux commands stop working

Linux Permissions and Ownership
Complete Documentation
Linux Permissions and Ownership
Introduction

Linux is a multi-user operating system. Unlike Windows, Linux was designed from the beginning to allow many users to work on the same system securely at the same time.

To prevent unauthorized access, every file and directory has ownership and permissions that determine:

Who owns the file
Who can read it
Who can modify it
Who can execute it
Who is completely denied access

Without permissions, any user could modify or delete important system files, making Linux insecure and unstable.

Ownership in Linux

Every file and directory has two owners:

User (Owner)
Group

Example:

ls -l

Output:

-rwxr-x--- 1 hamad developers 542 Jun 30 script.sh

Breakdown:

-rwxr-x---
 ^
 Permissions

hamad
^^^^^
Owner

developers
^^^^^^^^^^
Group

This means:

Owner = hamad
Group = developers
The Three Permission Classes

Linux divides permissions into three categories.

1. User (u)

The owner of the file.

Example:

hamad

Only this user is considered the owner.

2. Group (g)

Users who belong to the assigned group.

Example:

developers

Every member of the developers group receives these permissions.

3. Others (o)

Everyone else on the system.

If a user is neither:

the owner
nor in the group

then Linux uses the Others permissions.

Understanding Permission Letters

Each permission consists of three possible letters.

Letter	Meaning	Description
r	Read	View file contents
w	Write	Modify the file
x	Execute	Run the file as a program
Reading Permissions (r)
For Files

Allows viewing the contents.

Example:

cat notes.txt

Without read permission:

Permission denied
For Directories

Allows listing the directory contents.

Example:

ls Documents

Without read permission:

Permission denied
Writing Permissions (w)
For Files

Allows:

Editing
Saving
Overwriting
Truncating

Example:

nano report.txt

Without write permission:

You can open the file but cannot save changes.

For Directories

Allows:

Creating files
Deleting files
Renaming files

Example:

touch newfile.txt
rm oldfile.txt
mv file1 file2

Without write permission, none of these operations are allowed.

Execute Permission (x)

This permission is often misunderstood.

For Files

Allows Linux to run the file as a program.

Example:

./script.sh

Without execute permission:

Permission denied
For Directories

Execute means:

You are allowed to enter the directory.

Example:

cd Documents

Without execute permission:

Permission denied

Even if read permission exists, you cannot enter the directory.

Viewing Permissions

Use:

ls -l

Example:

-rwxr-xr--

Let's split it.

- rwx r-x r--
  |   |   |
  |   |   Others
  |   Group
  User
First Character
-

means regular file.

Other possibilities:

Symbol	Type
-	File
d	Directory
l	Symbolic Link
Understanding rwxr-xr--

Breakdown:

rwx

Owner can:

Read
Write
Execute
r-x

Group can:

Read
Execute

Cannot write.

r--

Others can:

Read only
Numeric Permissions

Linux also represents permissions with numbers.

Each permission has a value.

Permission	Value
Read	4
Write	2
Execute	1

Simply add them together.

Examples
Read + Write
4 + 2 = 6
Read + Execute
4 + 1 = 5
Read + Write + Execute
4 + 2 + 1 = 7
Execute only
1
No permissions
0
Common Numeric Values
Number	Permission
7	rwx
6	rw-
5	r-x
4	r--
3	-wx
2	-w-
1	--x
0	---
chmod Command

chmod changes permissions.

Syntax:

chmod permissions file
Numeric Method

Example:

chmod 755 script.sh

Meaning:

7 = rwx
5 = r-x
5 = r-x

Result:

Owner   rwx
Group   r-x
Others  r-x

Example:

chmod 644 notes.txt

Result:

Owner   rw-
Group   r--
Others  r--

Perfect for documents.

Example:

chmod 600 secret.txt

Only owner has access.

Owner   rw-
Group   ---
Others  ---
Symbolic Method

Instead of numbers, Linux allows symbolic notation.

Users:

Symbol	Meaning
u	User
g	Group
o	Others
a	All

Operations:

Symbol	Meaning
+	Add permission
-	Remove permission
=	Set exactly
Examples

Give execute permission:

chmod +x script.sh

Remove write permission:

chmod -w report.txt

Add write permission to owner:

chmod u+w file.txt

Remove execute from group:

chmod g-x script.sh

Give everyone read permission:

chmod a+r file.txt

Set owner permissions exactly:

chmod u=rwx file.sh
Numeric vs Symbolic chmod
Numeric	Symbolic
Fast	More readable
Sets all permissions at once	Changes only specific permissions
Great for scripts	Great for daily administration
Uses numbers (755, 644)	Uses letters (u+x, g-w)

Example:

Numeric:

chmod 755 script.sh

Symbolic:

chmod u=rwx,g=rx,o=rx script.sh

Both produce exactly the same result.

chown Command

Ownership is different from permissions.

Permissions answer:

What can users do?

Ownership answers:

Who owns the file?

Syntax:

chown owner file

Example:

sudo chown hamad report.txt

Owner becomes:

hamad

Change owner and group together:

sudo chown hamad:developers report.txt

Change only group:

sudo chown :developers report.txt
chgrp Command

Another command exists for changing only the group.

sudo chgrp developers report.txt

Equivalent to:

sudo chown :developers report.txt
Access Restrictions

Linux enforces permissions every time a user attempts to access a file or directory.

The permission check follows this order:

Owner – If the user owns the file, Linux uses the owner's permissions.
Group – If not the owner but a member of the file's group, Linux applies the group's permissions.
Others – If neither condition applies, Linux uses the permissions assigned to others.

If the required permission is not present, the operation is blocked with a Permission denied error.

Example

Suppose a file has these permissions:

-rw-r-----
Owner: Read and write
Group: Read only
Others: No access

Results:

The owner can read and edit the file.
Group members can only read it.
Everyone else cannot even open the file.
Real-Life Permission Examples
Private Documents

A personal diary or financial report should be readable only by its owner.

chmod 600 diary.txt
Shell Scripts

Scripts must be executable before they can run.

chmod +x backup.sh
Shared Project Folder

A development team shares a directory where everyone in the group can collaborate.

sudo chown :developers project
chmod 775 project
Owner: Full access
Developers group: Full access
Others: Read and execute only
Public Website Files

HTML files are typically readable by everyone but editable only by administrators.

chmod 644 index.html
Why Permissions Are Important in Real Life

Linux permissions are a fundamental security mechanism used on personal computers, servers, cloud platforms, and enterprise systems.

They help to:

Protect sensitive information from unauthorized users.
Prevent accidental modification or deletion of important files.
Limit access based on user roles and responsibilities.
Enable secure collaboration by granting different permissions to owners, groups, and others.
Reduce the impact of malware or compromised accounts by restricting what files can be accessed or changed.
Maintain the stability of the operating system by preventing ordinary users from modifying critical system files.

Without proper permissions, any user could overwrite system configuration files, delete application data, or execute malicious programs, making the system unreliable and vulnerable. Correct ownership and permission settings are therefore essential for both security and system administration.

Summary
Concept	Purpose
Owner	The user who owns the file or directory.
Group	A collection of users who can share permissions.
Others	Everyone else on the system.
Read (r)	View file contents or list directory contents.
Write (w)	Modify files or create, delete, and rename items within a directory.
Execute (x)	Run a file as a program or access (enter) a directory.
chmod	Changes file or directory permissions.
chown	Changes the owner and/or group of a file or directory.
Numeric mode	Uses numbers (e.g., 755, 644) to define permissions.
Symbolic mode	Uses letters (e.g., u+x, g-w) to add, remove, or set permissions.
Access restrictions	Linux checks permissions in the order: Owner → Group → Others before granting access.
Importance	Protects data, enforces security, supports collaboration, and maintains system integrity.
/usr	Installed software and shared resources	❌ No	Most applications stop working
/opt	Optional third-party software	✅ With caution	Only that application stops working
