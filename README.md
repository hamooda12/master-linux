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
/usr	Installed software and shared resources	❌ No	Most applications stop working
/opt	Optional third-party software	✅ With caution	Only that application stops working
