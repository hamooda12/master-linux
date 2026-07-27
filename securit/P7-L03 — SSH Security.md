# DevOps Task #13

## P7-L07 — SSH Security

**Student:** Hamad Tarawa  
**Stage:** Part 7 — Linux Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Ubuntu  

---

## 1. Connection to the Previous Lesson

In **P7-L06 — sudo and Least Privilege**, you learned that:

- Normal work should be performed as a normal user.
- `sudo` should be used only when administrative permission is required.
- Users, applications, and processes should receive only the permissions they
  need.

This lesson applies those ideas to remote server access.

A DevOps engineer often manages Linux servers from another computer. The
common tool used for this connection is **SSH**.

SSH security must answer two important questions:

1. How can the server verify the person connecting?
2. What should that person be allowed to do after connecting?

SSH handles the secure connection and authentication. Linux users,
permissions, groups, and `sudo` continue to control what happens after login.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain the purpose of SSH.
- Explain how SSH protects a remote connection.
- Distinguish password authentication from SSH key authentication.
- Distinguish a public key from a private key.
- Explain why the private key must remain secret.
- Create a safe practice SSH key pair.
- Apply secure permissions to SSH files.
- Explain why direct root login should normally be disabled.
- Explain the purpose of restricting allowed SSH users.
- Understand why changing the SSH port is not a replacement for real security.

---

## 3. Why This Matters for a DevOps Engineer

DevOps engineers use SSH to manage:

- Cloud virtual machines
- Web servers
- Application servers
- Database servers
- CI/CD runners
- On-premises Linux machines

Example:

```bash
ssh hamad@203.0.113.10
```

This command attempts to connect to a remote server as the Linux user
`hamad`.

If SSH is configured poorly, an unauthorized person may attempt to access the
server. If the account also has powerful `sudo` permissions, the possible
impact becomes much greater.

SSH security therefore protects one of the most important administrative
entrances to a Linux server.

---

## 4. Simple Explanation

### 4.1 What is SSH?

SSH means:

```text
Secure Shell
```

SSH is a protocol used to create a secure connection between:

- An SSH client: your computer
- An SSH server: the remote Linux machine

After connecting, an authorized user can:

- Run commands.
- Inspect logs.
- Manage services.
- Transfer files.
- Deploy applications.
- Troubleshoot problems.

SSH protects the communication while it travels across the network.

Without encryption, someone observing the network might be able to read
commands, passwords, or data. SSH encrypts the connection so the information
is not sent as readable plaintext.

---

### 4.2 The basic SSH connection

```text
Your computer
SSH client
     |
     | Encrypted SSH connection
     v
Linux server
SSH server
     |
     | Authentication succeeds
     v
Linux user session
```

A simplified SSH connection includes:

1. Your SSH client contacts the server.
2. The server identifies itself using a host key.
3. The client and server create an encrypted connection.
4. The server authenticates the user.
5. If authentication succeeds, the user receives a Linux session.

You do not need to learn the cryptographic details for this DevOps lesson.

---

### 4.3 SSH does not replace Linux permissions

SSH decides whether a user may establish a remote session.

After login, normal Linux security still applies:

- The user has a UID and groups.
- File permissions control file access.
- Process permissions limit actions.
- `sudo` controls administrative elevation.

Example:

```text
SSH authenticates Hamad
         |
         v
Hamad receives a Linux session
         |
         v
Linux permissions control Hamad's access
         |
         v
sudo is required for approved administrative tasks
```

SSH does not automatically give every connected user root access.

---

## 5. Password Authentication

With password authentication:

1. The user enters a username.
2. The server asks for the account password.
3. The server verifies the password.
4. Access is granted if the password is correct.

Example:

```bash
ssh hamad@server-ip
```

Possible prompt:

```text
hamad@server-ip's password:
```

### Advantages

- Easy for beginners to understand.
- Does not require creating a key pair first.

### Security concerns

- Weak passwords may be guessed.
- Reused passwords may already be exposed elsewhere.
- Public SSH servers may receive repeated login attempts.
- Users may accidentally share or reveal passwords.

If password authentication is used, accounts should use strong, unique
passwords and appropriate server protections.

For administrative server access, SSH key authentication is commonly
preferred.

---

## 6. SSH Key Authentication

SSH key authentication uses a pair of related files:

```text
Private key + Public key
```

The two keys have different jobs.

### Private key

The private key:

- Stays on the client computer.
- Must remain secret.
- Proves that the client possesses the correct credential.
- Should never be sent to the server or another person.

Example filename:

```text
id_ed25519
```

### Public key

The public key:

- Can be copied to the server.
- Does not need to remain secret.
- Allows the server to recognize the matching private key.

Example filename:

```text
id_ed25519.pub
```

### Simple mental model

```text
YOUR COMPUTER                         LINUX SERVER

Private key                           Public key
Keep secret                           Stored for your account
     |                                      |
     +--------- authentication proof -------+
```

The private key itself is not sent across the network during authentication.

---

### 6.1 Where is the public key stored on the server?

For a Linux user, authorized public keys are commonly stored in:

```text
~/.ssh/authorized_keys
```

For example, keys allowed to log in as `hamad` would be stored under the
`hamad` account's home directory.

Possessing a valid private key does not automatically allow login as every
user. The corresponding public key must be authorized for the selected
account.

---

### 6.2 Why is a passphrase useful?

When creating a private key, you may protect it with a passphrase.

The passphrase encrypts the private key file on your computer.

This adds protection if someone copies the key file. They would still need the
passphrase to use it.

A passphrase is not the same as the remote Linux user's password:

- Account password: verified by the remote server during password login.
- Key passphrase: protects the private key stored on your local computer.

---

## 7. Important Terms

| Term | Simple meaning |
|---|---|
| SSH | Secure protocol for remote command-line access |
| SSH client | The computer or program starting the connection |
| SSH server | The remote machine accepting SSH connections |
| Authentication | Verifying who is trying to connect |
| Encryption | Making network data unreadable to unauthorized observers |
| Private key | Secret key that remains on the client |
| Public key | Key that may be copied to the server |
| Key pair | A related private key and public key |
| Passphrase | Secret that protects a private key file |
| `authorized_keys` | File listing public keys permitted for a Linux account |
| `sshd` | The SSH server process |
| `sshd_config` | Main SSH server configuration file |
| Root login | Connecting directly through SSH as the `root` user |

---

## 8. Commands and Configuration

### Command 1 — Create an Ed25519 SSH key pair

The normal command is:

```bash
ssh-keygen -t ed25519
```

Explanation:

- `ssh-keygen`: Creates and manages SSH keys.
- `-t`: Selects the key type.
- `ed25519`: A modern SSH key type.

However, using the normal default path could interact with an existing key.
For the safe lab, we will create a separate practice key with a unique name.

---

### Command 2 — Inspect the SSH directory

```bash
ls -la ~/.ssh
```

Purpose:

> Show SSH files and their permissions.

The `-a` option includes hidden entries, and `-l` displays detailed
permissions.

If `~/.ssh` does not exist yet, the command may show:

```text
No such file or directory
```

That is normal for a user who has not used SSH before.

---

### Command 3 — Protect the SSH directory

```bash
chmod 700 ~/.ssh
```

Meaning:

```text
Owner:  read, write, execute
Group:  no permission
Others: no permission
```

The directory owner can access it, while other users cannot.

---

### Command 4 — Protect a private key

For a normal default private key:

```bash
chmod 600 ~/.ssh/id_ed25519
```

Meaning:

```text
Owner:  read and write
Group:  no permission
Others: no permission
```

SSH may refuse to use a private key if its permissions allow other users to
access it.

In the lab, our practice key will have a different filename to avoid
overwriting an existing key.

---

### Command 5 — Set normal public-key permissions

```bash
chmod 644 ~/.ssh/id_ed25519.pub
```

Meaning:

```text
Owner:  read and write
Group:  read
Others: read
```

The public key does not contain the private secret, so it does not require the
same strict secrecy as the private key.

---

## 9. Basic SSH Server Security Configuration

The main OpenSSH server configuration file is commonly:

```text
/etc/ssh/sshd_config
```

The following settings are important concepts. Do not change them in the lab.

### 9.1 Disable direct root login

```text
PermitRootLogin no
```

Why?

- Attackers already know that a `root` account exists.
- Direct root login immediately targets the most powerful account.
- A named normal user provides clearer accountability.
- The administrator can log in normally and use `sudo` only when needed.

A safer pattern is:

```text
SSH login as hamad
        |
        v
Normal user permissions
        |
        | sudo only when required
        v
Temporary administrative permission
```

---

### 9.2 Restrict which users may connect

An SSH server can restrict login to selected users:

```text
AllowUsers hamad deploy
```

This means only the listed accounts are allowed to connect through SSH under
that rule.

The exact policy should match the organization's real administrative needs.

---

### 9.3 Disable password authentication when keys are ready

```text
PasswordAuthentication no
```

This forces users to use another enabled authentication method, commonly SSH
keys.

Important safety rule:

> Never disable password authentication on a remote server until key-based
> login has been configured and successfully tested in a separate connection.

Otherwise, you may lock yourself out of the server.

---

### 9.4 Changing the SSH port

SSH commonly listens on:

```text
22
```

Some administrators change the port to reduce simple automated connection
noise.

However:

> Changing the SSH port does not replace strong authentication, restricted
> users, firewall rules, updates, or disabled root login.

A network scanner can still discover a service on another port. Therefore,
changing the port is not a complete security control.

---

### 9.5 Keep the SSH server updated

The OpenSSH server is software. Like other software, it should receive
supported security updates.

A DevOps engineer should:

- Use a supported operating-system version.
- Apply appropriate security updates.
- Review configuration changes.
- Remove accounts that no longer need access.
- Review authorized SSH keys.

---

## 10. Safe Practical Lab

### Lab goal

Create a separate practice SSH key pair, inspect it, and apply secure file
permissions.

### WSL note

You do not need to run an SSH server in WSL.

This lab demonstrates:

- Key-pair creation
- Private and public key identification
- Secure file permissions

Actual remote-server access is not required.

### Safety

The lab uses these unique filenames:

```text
devops_security_lab_key
devops_security_lab_key.pub
```

This avoids overwriting a normal key such as `id_ed25519`.

---

### Step 1 — Create the SSH directory if needed

Run:

```bash
mkdir -p ~/.ssh
```

Explanation:

- `mkdir`: Creates a directory.
- `-p`: Creates the directory only if needed and avoids an error if it already
  exists.
- `~/.ssh`: The current user's SSH directory.

---

### Step 2 — Protect the SSH directory

Run:

```bash
chmod 700 ~/.ssh
```

This allows only your user to access the directory.

---

### Step 3 — Check whether the practice key already exists

Run:

```bash
ls -l ~/.ssh/devops_security_lab_key*
```

If the result says:

```text
No such file or directory
```

you can continue.

If files with these names already exist, do not overwrite them. Use another
name such as:

```text
devops_security_lab_key_2
```

---

### Step 4 — Generate the practice key pair

Run:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/devops_security_lab_key -C "devops-security-lab"
```

Explanation:

- `-t ed25519`: Creates an Ed25519 key.
- `-f`: Selects the output filename.
- `-C`: Adds a comment that helps identify the key.

You will see:

```text
Enter passphrase (empty for no passphrase):
```

For better protection, enter a passphrase you can remember.

For a temporary local training key, you may press **Enter** twice to leave it
empty, but a real administrative key should normally be protected
appropriately.

Never send the passphrase to anyone and do not include it in your report.

---

### Step 5 — Inspect the created files

Run:

```bash
ls -l ~/.ssh/devops_security_lab_key*
```

You should see:

```text
devops_security_lab_key
devops_security_lab_key.pub
```

The file without `.pub` is the private key.

The file ending with `.pub` is the public key.

---

### Step 6 — Apply secure permissions

Run:

```bash
chmod 600 ~/.ssh/devops_security_lab_key
chmod 644 ~/.ssh/devops_security_lab_key.pub
```

Then verify:

```bash
ls -l ~/.ssh/devops_security_lab_key*
```

---

### Step 7 — Display only the public key

Run:

```bash
cat ~/.ssh/devops_security_lab_key.pub
```

This displays the public key.

Do **not** run `cat` on the private-key file for screenshots or reports.

The public key output will look similar to:

```text
ssh-ed25519 AAAAC3... devops-security-lab
```

---

## 11. Expected Result

After applying permissions, `ls -l` should show a result similar to:

```text
-rw------- 1 hamad hamad 411 Jul 27 12:00 devops_security_lab_key
-rw-r--r-- 1 hamad hamad 101 Jul 27 12:00 devops_security_lab_key.pub
```

Interpretation:

### Private key

```text
-rw-------
```

- Owner can read and write.
- Group has no access.
- Others have no access.

### Public key

```text
-rw-r--r--
```

- Owner can read and write.
- Group can read.
- Others can read.

The exact sizes and timestamp will differ.

---

## 12. Evidence to Save

Save a screenshot showing:

```bash
ls -l ~/.ssh/devops_security_lab_key*
```

The screenshot should clearly show:

- The private-key filename.
- The public-key filename.
- `600`-style private-key permissions: `-rw-------`
- `644`-style public-key permissions: `-rw-r--r--`

You may also save a screenshot showing:

```bash
cat ~/.ssh/devops_security_lab_key.pub
```

### Never include

- The contents of the private key.
- Your key passphrase.
- A real production private key.
- A real server password.

### Suggested evidence title

```text
SSH Key Pair and Secure Private-Key Permissions in WSL
```

### Suggested evidence explanation

> I generated a separate Ed25519 SSH key pair for the lab. The private key
> remained on my local machine and received `600` permissions, allowing only
> its owner to read or modify it. The public key received `644` permissions
> and can be copied to an authorized server account. The private key must never
> be shared.

---

## 13. Common Mistakes

### Mistake 1 — Sharing the private key

The private key proves your identity. Anyone who obtains it may attempt to act
as you.

Share the public key, not the private key.

---

### Mistake 2 — Using weak private-key permissions

Permissions such as `644` on a private key allow other local users to read it.

Private keys commonly need:

```bash
chmod 600 private-key-file
```

---

### Mistake 3 — Copying the private key to the server

The server needs your public key in `authorized_keys`.

Your private key should remain on the trusted client device.

---

### Mistake 4 — Disabling passwords before testing keys

If key authentication is not working, disabling password authentication may
lock you out.

Test the key in a separate connection first.

---

### Mistake 5 — Allowing direct root login

Users should normally connect through named, limited accounts and use `sudo`
only when required.

---

### Mistake 6 — Believing a different SSH port provides complete security

Changing port `22` may reduce simple automated noise, but it does not replace:

- Strong authentication
- Firewall restrictions
- Updated software
- Limited users
- Disabled root login

---

### Mistake 7 — Keeping unused keys authorized

Keys belonging to former team members or unused automation should be removed
from the authorized access list.

---

## 14. Task Report Notes

You may adapt the following text for your final assignment:

> SSH, or Secure Shell, allows DevOps engineers to manage remote Linux systems
> through an encrypted connection. It protects commands and transmitted data
> from being sent as readable plaintext across the network. After SSH
> authentication succeeds, normal Linux users, groups, file permissions, and
> sudo rules continue to control what the connected user can do.

> SSH may authenticate users with passwords or key pairs. A key pair contains
> a private key and a public key. The private key must remain secret on the
> client computer, while the public key can be installed in the remote user's
> `authorized_keys` file. Secure private-key permissions, such as `600`, help
> prevent other local users from reading the key.

> Recommended SSH protections include using key-based authentication,
> protecting private keys with appropriate permissions, disabling direct root
> login, restricting which accounts may connect, reviewing authorized keys,
> and keeping the SSH server updated. Changing the default SSH port may reduce
> automated connection noise, but it does not replace these security controls.

### Practical demonstration note

> In WSL, I generated a separate Ed25519 key pair for the security lab. I used
> `chmod 600` for the private key and `chmod 644` for the public key. The
> private key was not displayed or shared. This demonstrated the difference
> between public and private SSH keys and the importance of protecting the
> private key.

---

## 15. Five Review Questions

1. What is SSH, and why do DevOps engineers use it?
2. What is the difference between a public key and a private key?
3. Why should a private SSH key commonly have `600` permissions?
4. Why should direct SSH login as `root` normally be disabled?
5. Why does changing the SSH port not replace real security controls?

---

## 16. Lesson Summary

SSH provides encrypted remote access to Linux systems.

```text
SSH protects the connection
           |
           v
Authentication verifies the user
           |
           v
Linux permissions control the session
           |
           v
sudo provides temporary administration
```

The main security lessons are:

- Prefer controlled, secure authentication.
- Keep the private key secret.
- Copy only the public key to the server.
- Protect private keys with restrictive permissions.
- Avoid direct root login.
- Restrict access to users who genuinely need it.
- Keep the SSH server updated.
- Do not treat a changed port as complete protection.

Important lab commands:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/devops_security_lab_key -C "devops-security-lab"
chmod 600 ~/.ssh/devops_security_lab_key
chmod 644 ~/.ssh/devops_security_lab_key.pub
ls -l ~/.ssh/devops_security_lab_key*
```

---

## 17. Progress

### Part 7 — Linux Security

- [x] P7-L01 — Linux Security for DevOps Engineers
- [x] P7-L02 — Linux Users and Groups
- [x] P7-L03 — Linux File Permissions
- [x] P7-L04 — Changing Permissions with chmod
- [x] P7-L05 — File Ownership with chown and chgrp
- [x] P7-L06 — sudo and Least Privilege
- [x] P7-L07 — SSH Security
- [ ] P7-L08 — Linux Firewall Basics
- [ ] P7-L09 — Linux Process Isolation
- [ ] P7-L10 — Part 7 WSL Lab and Report

---

## Exact Next Lesson

**P7-L08 — Linux Firewall Basics**

Do not continue until the student writes:

```text
next
```
