
# Chapter 01 — Lesson 18: Final Hands-on Project

## Overview

This project combines everything you have learned throughout Chapter 01 into one realistic hands-on exercise.

Instead of practicing commands individually, you will complete a complete Linux filesystem management project similar to tasks performed by Linux administrators, DevOps engineers, and developers.

---

# Learning Objectives

After completing this project, you will be able to:

- Navigate the Linux filesystem
- Create and organize directories
- Create files
- Copy, move, and rename files
- Delete files safely
- Search for files
- Use wildcards
- Read file contents
- Create hard links
- Create symbolic links
- Change permissions
- Apply special permissions
- Understand ownership

---

# Project Scenario

You have joined a software company as a Junior Linux Administrator.

Your manager asks you to prepare the workspace for a new application.

The final structure should look like this:

```text
company-project/
├── backups/
├── config/
├── docs/
├── logs/
├── scripts/
│   ├── deploy.sh
│   └── cleanup.sh
├── src/
│   ├── app.py
│   ├── utils.py
│   └── config.py
├── shared/
├── public/
├── README.md
└── secret.txt
````

---

# Part 1 — Create the Project Structure

Create every directory and file shown above.

---

# Part 2 — Verify the Structure

Use the appropriate commands to verify:

* Current location
* Directory contents
* Recursive directory structure

Commands you should use:

```bash
pwd
ls
ls -la
ls -R
```

---

# Part 3 — File Management

Complete the following tasks:

* Create a backup of `README.md`
* Move the backup into `backups`
* Rename `deploy.sh` to `deploy-production.sh`
* Rename `cleanup.sh` to `cleanup-old.sh`

Verify your work.

---

# Part 4 — Viewing Files

Practice using:

```bash
cat README.md

head README.md

tail README.md

less README.md
```

---

# Part 5 — Searching

Locate:

* Every Markdown file
* Every Python file
* Every directory
* Every regular file

Suggested commands:

```bash
find . -name "*.md"

find . -name "*.py"

find . -type d

find . -type f
```

---

# Part 6 — Wildcards

Practice wildcard matching:

```bash
ls *.md

ls *.py

ls src/*.py

cp *.md docs/
```

---

# Part 7 — Hard and Soft Links

Create:

```bash
ln README.md README-hard.md

ln -s README.md README-soft.md
```

Compare them:

```bash
ls -li
```

Delete the original README and observe the behavior.

Record your observations.

---

# Part 8 — Permissions

Perform the following:

```bash
chmod 755 scripts/deploy-production.sh

chmod 755 scripts/cleanup-old.sh

chmod 644 README.md

chmod 600 secret.txt
```

Verify:

```bash
ls -l
```

---

# Part 9 — Special Permissions

Enable SetGID on the shared directory.

```bash
chmod 2775 shared
```

Enable the Sticky Bit on the public directory.

```bash
chmod 1777 public
```

Verify:

```bash
ls -ld shared

ls -ld public
```

---

# Part 10 — Ownership (Optional)

If your system has multiple users:

```bash
sudo chown root README.md

sudo chgrp root README.md
```

Verify:

```bash
ls -l
```

---

# Final Verification Checklist

* [ ] I navigated the filesystem.
* [ ] I created directories.
* [ ] I created files.
* [ ] I copied files.
* [ ] I moved files.
* [ ] I renamed files.
* [ ] I deleted files safely.
* [ ] I viewed file contents.
* [ ] I searched using `find`.
* [ ] I used wildcards.
* [ ] I created hard links.
* [ ] I created symbolic links.
* [ ] I changed permissions.
* [ ] I applied special permissions.
* [ ] I inspected ownership.

---

# Self-Assessment

Rate yourself from **1–5**.

| Skill               | Rating |
| ------------------- | ------ |
| Navigation          |        |
| File Management     |        |
| Search              |        |
| Links               |        |
| Permissions         |        |
| Special Permissions |        |
| Overall Confidence  |        |

---

# Challenge

Without looking at previous lessons:

Rebuild the complete project from scratch inside another directory.

Time yourself.

Goal:

* Finish within **30 minutes**
* No documentation
* No internet search

---

# Summary

Congratulations!

You have completed a realistic Linux filesystem project that combines every major concept from Chapter 01.

You are now comfortable with:

* Linux navigation
* Filesystem management
* Viewing files
* Searching
* Wildcards
* Hard links
* Symbolic links
* Permissions
* Special permissions
* Ownership

These are the foundational skills expected from every Linux user before moving into text processing, shell scripting, and system administration.

---

# Lab Assets

Store screenshots in:

```text
assets/chapter-01/
```

| Filename                        | Description                                  |
| ------------------------------- | -------------------------------------------- |
| `72-project-structure.png`      | Final project directory tree                 |
| `73-file-management.png`        | Copying, moving, and renaming files          |
| `74-find-results.png`           | Results of all `find` commands               |
| `75-links-comparison.png`       | Hard link vs. symbolic link                  |
| `76-permissions.png`            | Final permission settings                    |
| `77-special-permissions.png`    | SetGID and Sticky Bit verification           |
| `78-final-project-complete.png` | Final terminal showing the completed project |

