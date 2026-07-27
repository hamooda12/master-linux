# DevOps Task #13

## P7-L06 — sudo and Least Privilege

**Student:** Hamad Tarawa  
**Stage:** Part 7 — Linux Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Ubuntu

---

## 1. Connection to the Previous Lesson

In the previous lessons, you learned that Linux protects resources using:

- Users and groups
- File permissions
- File ownership

These controls determine what a normal user is allowed to access.

However, some DevOps tasks require administrative permission. For example:

- Installing Nginx
- Updating packages
- Managing system services
- Creating users
- Changing ownership of protected files
- Editing system configuration

This creates an important question:

> How can a normal user perform an administrative task without working as the
> powerful `root` user all the time?

The answer is `sudo`.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain what `sudo` does.
- Distinguish `root`, a normal user, and a sudo-enabled user.
- Explain the Principle of Least Privilege.
- Understand why administrative access should be temporary.
- Use `sudo whoami` and `sudo -l` safely.
- Explain why scripts and applications should not use `sudo` unnecessarily.
- Describe the basic purpose of the `sudoers` configuration.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer regularly manages Linux servers.

Some operations need administrative permission:

```bash
sudo apt update
sudo systemctl restart nginx
sudo chown appuser:appgroup /opt/myapp
```

These commands can affect the entire system. If administrative permission is
used carelessly, a mistake can:

- Break a service.
- Delete or overwrite protected files.
- Change important security settings.
- Affect other users.
- Expose the server.

A DevOps engineer should therefore use administrative access only:

1. When it is required.
2. For the specific command that requires it.
3. For the shortest necessary time.

That is the main connection between `sudo` and the Principle of Least
Privilege.

---

## 4. Simple Explanation

### 4.1 What is `root`?

`root` is the main Linux administrator account.

It normally has permission to:

- Read and modify almost every file.
- Install and remove software.
- Create and delete users.
- Start and stop system services.
- Change file ownership.
- Modify network and firewall settings.
- Shut down or restart the machine.

You can think of `root` as the master administrator of the Linux system.

Because it is extremely powerful, working as `root` all the time is dangerous.
A typing mistake made as a normal user may fail with:

```text
Permission denied
```

The same mistake made as `root` may succeed and damage the system.

---

### 4.2 What is `sudo`?

`sudo` means:

```text
superuser do
```

It allows an approved user to run a command with elevated privileges.

For example:

```bash
apt update
```

may fail for a normal user because updating package information is a system
operation.

The approved user can request administrative permission for that command:

```bash
sudo apt update
```

The important point is:

> `sudo` elevates the command, not every activity performed by the user.

After the command finishes, the user's normal shell still operates under the
normal account.

---

### 4.3 Does `sudo` permanently turn you into root?

No.

Consider:

```bash
whoami
sudo whoami
whoami
```

A possible result is:

```text
hamad
root
hamad
```

Explanation:

1. The first command runs normally, so the result is `hamad`.
2. The second command is elevated with `sudo`, so that command runs as `root`.
3. The final command is normal again, so the result returns to `hamad`.

Therefore, `sudo` gives temporary privilege to a specific command. It does not
permanently change the current user.

---

### 4.4 Why may `sudo` ask for a password?

When you use `sudo`, Linux may ask for your own user password.

Example:

```text
[sudo] password for hamad:
```

This helps verify that the person using the terminal is the approved user.

While entering the password, Linux normally displays:

- No letters
- No dots
- No stars

This is normal. Type the password and press **Enter**.

After successful authentication, `sudo` may temporarily remember it for a
short period. This is why a second `sudo` command may not immediately ask for
the password again.

---

### 4.5 What is the Principle of Least Privilege?

The Principle of Least Privilege, abbreviated as **PoLP**, means:

> A user, application, or process should receive only the permissions required
> to perform its job—and nothing more.

Examples:

| Subject | Required job | Appropriate permission |
|---|---|---|
| Developer | Edit project code | Access to the project directory |
| Spring Boot application | Read its configuration | Read access to its own configuration |
| PostgreSQL | Manage database files | Access to its database directory |
| Nginx | Read website files | Read access to the web content |
| Administrator | Restart Nginx | Permission to run the required administrative command |

Least privilege does not mean:

> Give no permissions.

It means:

> Give enough permission to complete the job, but do not give unnecessary
> permission.

---

### 4.6 How does `sudo` support least privilege?

Without controlled elevation, an administrator might log in directly as
`root` and remain powerful for the entire session.

With `sudo`, the administrator can work normally and elevate only the command
that requires administrative permission.

Example:

```bash
cat app.log
sudo systemctl restart nginx
cat app.log
```

Only the service restart uses administrative privilege.

This reduces the chance that unrelated commands accidentally modify protected
system resources.

---

### 4.7 `sudo` does not mean unrestricted access for everyone

Not every Linux user can necessarily use every `sudo` command.

Linux checks the `sudo` policy to decide:

- Which users may use `sudo`.
- Which commands they may run.
- Which user they may run those commands as.

On Ubuntu, users with general administrative permission commonly belong to
the `sudo` group.

You can inspect your groups using:

```bash
groups
```

or:

```bash
id
```

If `sudo` appears in the group list, the account commonly has administrative
permission under Ubuntu's default configuration.

However, the final decision is determined by the system's `sudo` policy.

---

## 5. Important Terms

| Term | Simple meaning |
|---|---|
| `root` | The most powerful Linux administrator account |
| `sudo` | Runs an approved command with elevated privileges |
| Privilege | Permission to perform an operation |
| Privilege elevation | Temporarily receiving stronger permissions |
| Least Privilege | Giving only the access required for the job |
| `sudoers` | The policy that controls who may use `sudo` and how |
| Administrative command | A command that changes protected system resources |
| `sudo -l` | Lists the current user's allowed sudo operations |
| Audit trail | A record that helps show which administrative commands were used |

---

## 6. Simple Real-World Example

Suppose you manage a Spring Boot application behind Nginx.

Your normal daily work may include:

```bash
cd ~/student-app
ls
git status
cat README.md
```

These commands do not need administrative permission.

After changing the Nginx configuration, you may need to restart Nginx:

```bash
sudo systemctl restart nginx
```

Only that command needs elevated permission.

The correct pattern is:

```text
Normal user
    |
    | ordinary development and inspection commands
    v
Normal permissions

Normal user
    |
    | sudo + approved administrative command
    v
Temporary elevated permission
    |
    | command finishes
    v
Normal permissions again
```

The poor pattern would be logging in as `root` and performing every daily
operation with complete system access.

---

## 7. Commands and Configuration

### Command 1 — Show the current user

```bash
whoami
```

Purpose:

> Display the identity running the current command.

This should normally show your regular username.

---

### Command 2 — Run `whoami` with administrative privilege

```bash
sudo whoami
```

Purpose:

> Demonstrate which identity runs an elevated command.

Expected result:

```text
root
```

This does not permanently change your user. It only runs `whoami` with
elevated privilege.

---

### Command 3 — List your allowed sudo operations

```bash
sudo -l
```

Purpose:

> Display which commands or categories of commands the current user may run
> through `sudo`.

The output depends on the system configuration.

You might see an entry similar to:

```text
User hamad may run the following commands:
    (ALL : ALL) ALL
```

In a simple Ubuntu environment, this commonly means that the user can run
commands as other users, including `root`, after satisfying the configured
authentication rules.

Do not worry about memorizing the full syntax in this lesson. The important
idea is that `sudo -l` shows your permitted sudo access.

---

### Command 4 — Show your groups

```bash
groups
```

Purpose:

> Display the groups containing the current user.

On Ubuntu, the presence of the `sudo` group commonly indicates that the user
has general administrative permission.

---

### Command 5 — Clear the remembered sudo authentication

```bash
sudo -k
```

Purpose:

> Invalidate the current user's cached sudo authentication.

This command does not remove your permission to use `sudo`. It only causes a
future `sudo` command to require authentication again according to the
system's policy.

This command may produce no output. That is normal.

---

### What is the `sudoers` configuration?

The `sudoers` configuration defines the sudo policy.

It controls:

- Which users can use `sudo`.
- Which commands they can run.
- Which target users they can act as.
- Whether authentication is required.

The main configuration is commonly associated with:

```text
/etc/sudoers
```

Additional rules may also exist under:

```text
/etc/sudoers.d/
```

You should not directly edit `/etc/sudoers` with an ordinary text editor.
A syntax mistake could break sudo access.

Administrators use:

```bash
sudo visudo
```

`visudo` checks the configuration syntax before accepting changes.

In this lesson, we will **not modify** the sudoers configuration. You only need
to understand its purpose.

---

## 8. Safe Practical Lab

### Lab goal

Compare a normal command with an elevated command and inspect your sudo
permissions.

### Safety

This lab is safe for your own WSL environment.

The commands only:

- Display identities.
- Display sudo permissions.
- Display groups.
- Clear cached sudo authentication.

They do not install, delete, or modify application files.

### Step 1 — Confirm your normal user

Run:

```bash
whoami
```

Record the result.

Expected example:

```text
hamad
```

### Step 2 — Compare it with an elevated command

Run:

```bash
sudo whoami
```

Enter your password if requested.

Expected result:

```text
root
```

### Step 3 — Confirm that you returned to your normal identity

Run:

```bash
whoami
```

Expected example:

```text
hamad
```

This proves that only the command after `sudo` was elevated.

### Step 4 — Inspect your sudo permissions

Run:

```bash
sudo -l
```

Read the output and identify the line explaining which sudo commands you may
run.

### Step 5 — Inspect your group membership

Run:

```bash
groups
```

Check whether `sudo` appears.

### Step 6 — Clear cached sudo authentication

Run:

```bash
sudo -k
```

This may return no visible output.

The next sudo command may ask for authentication again:

```bash
sudo whoami
```

You do not need to run the final command if you have already collected the
required evidence.

---

## 9. Expected Result

A possible terminal session is:

```text
$ whoami
hamad

$ sudo whoami
[sudo] password for hamad:
root

$ whoami
hamad

$ groups
hamad sudo

$ sudo -l
User hamad may run the following commands:
    (ALL : ALL) ALL

$ sudo -k
```

The important result is:

```text
Normal command       -> normal user
sudo command         -> elevated user
Next normal command  -> normal user again
```

This demonstrates temporary privilege elevation.

Your exact `groups` and `sudo -l` results may be different.

---

## 10. Evidence to Save

Save a screenshot containing:

```bash
whoami
sudo whoami
whoami
sudo -l
```

You may also include:

```bash
groups
```

Do not include your password in a screenshot. Linux normally does not display
it anyway.

### Suggested screenshot title

```text
WSL Demonstration — Temporary Administrative Privilege with sudo
```

### Suggested evidence explanation

> The first `whoami` command showed my normal Linux user. The
> `sudo whoami` command returned `root`, proving that this individual command
> received administrative privilege. The following `whoami` command returned
> my normal username again, proving that sudo elevation was temporary.
> The `sudo -l` command displayed the administrative operations permitted for
> my account.

### Part 7 demonstration status

This lab can be used as the required:

> Administrative privilege demonstration using `sudo`.

---

## 11. Common Mistakes

### Mistake 1 — Adding `sudo` whenever a command fails

A command may fail because:

- The path is wrong.
- The file does not exist.
- The command syntax is incorrect.
- The service is not installed.
- The user genuinely lacks permission.

Do not automatically add `sudo` to every failed command. Read the error first.

---

### Mistake 2 — Running applications as root

An application usually does not need complete control over the server.

Use a dedicated service account with limited access.

---

### Mistake 3 — Running an entire script with sudo unnecessarily

Suppose a deployment script contains 20 operations, but only one needs
administrative access.

Running:

```bash
sudo ./deploy.sh
```

gives every command inside the script elevated privilege.

It is safer to design the workflow so that only the required operation is
elevated.

---

### Mistake 4 — Editing `/etc/sudoers` directly

A syntax mistake can break sudo access.

Use `visudo` when an authorized configuration change is actually required.

---

### Mistake 5 — Giving every user unrestricted sudo access

Only users who need administrative operations should receive them.

In production, permissions may be restricted to particular commands rather
than granting complete root access.

---

### Mistake 6 — Assuming sudo makes a dangerous command safe

`sudo` does not check whether your decision is correct. It only grants the
permission required to execute the command.

Always understand a command before elevating it.

---

## 12. Best Practices

1. Work as a normal user by default.
2. Use `sudo` only when the operation requires administrative permission.
3. Understand the command before running it with `sudo`.
4. Do not run applications as `root` without a real requirement.
5. Give administrative access only to approved users.
6. Prefer narrowly scoped permissions in production.
7. Use `sudo -l` to inspect permitted operations.
8. Use `visudo` for authorized sudo-policy changes.
9. Do not place `sudo` in scripts unnecessarily.
10. Review administrative access when a team member changes roles or leaves.

---

## 13. Task Report Notes

You may adapt the following text for your final assignment:

> The `sudo` command allows an approved Linux user to run a specific command
> with elevated privileges. It provides a safer approach than performing all
> daily work while logged in as the root user. After the elevated command
> finishes, normal commands continue to run under the regular user account.

> This supports the Principle of Least Privilege, which requires users,
> applications, and processes to receive only the permissions needed to
> perform their responsibilities. In DevOps, administrative access should be
> used only for operations such as package management, service management, or
> protected system configuration.

> The `sudo -l` command displays the sudo operations permitted for the current
> account. The sudo policy is controlled through the sudoers configuration.
> Authorized administrators should use `visudo` instead of directly editing
> the main sudoers file because `visudo` validates the configuration syntax.

### Practical demonstration note

> In WSL, I compared `whoami`, `sudo whoami`, and `whoami` again. The normal
> commands returned my regular username, while the elevated command returned
> `root`. This demonstrated that sudo provided administrative privilege only
> to the selected command and did not permanently change my user identity.

---

## 14. Five Review Questions

1. What does `sudo` do?
2. Why is working as `root` all the time dangerous?
3. What does the Principle of Least Privilege mean?
4. What information does `sudo -l` display?
5. Why should an administrator use `visudo` instead of directly editing
   `/etc/sudoers`?

---

## 15. Lesson Summary

The `root` user has extremely powerful access to a Linux system.

Instead of working as `root` all the time, an approved user can use `sudo` to
elevate only the command that requires administrative privilege.

```text
Normal user
    |
    | sudo + approved command
    v
Temporary administrative privilege
    |
    | command finishes
    v
Normal user
```

This supports the Principle of Least Privilege:

> Give each user, application, and process only the access required to perform
> its job.

Important commands from this lesson:

```bash
whoami
sudo whoami
sudo -l
groups
sudo -k
```

---

## 16. Progress

### Part 7 — Linux Security

- [x] P7-L01 — Linux Security for DevOps Engineers
- [x] P7-L02 — Linux Users and Groups
- [x] P7-L03 — Linux File Permissions
- [x] P7-L04 — Changing Permissions with chmod
- [x] P7-L05 — File Ownership with chown and chgrp
- [x] P7-L06 — sudo and Least Privilege
- [ ] P7-L07 — SSH Security
- [ ] P7-L08 — Linux Firewall Basics
- [ ] P7-L09 — Linux Process Isolation
- [ ] P7-L10 — Part 7 WSL Lab and Report

---

## 17. Exact Next Lesson

**P7-L07 — SSH Security**

Do not continue until the student writes:

```text
next
```
