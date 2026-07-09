# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 11 --- SSH Fundamentals (Part 3)

------------------------------------------------------------------------

# SSH Key Pairs

## Why Use SSH Keys?

While password authentication is simple, production environments usually
prefer **SSH key authentication** because it is:

-   More secure
-   Faster
-   Easier to automate
-   Resistant to brute-force password attacks

For this reason, cloud providers often disable password logins and
require SSH keys.

------------------------------------------------------------------------

# Public Key Cryptography

SSH uses two related keys:

    Private Key
          │
          │ (Keep Secret)
          ▼

    Public Key
          │
          │ (Copy to Server)
          ▼

    Remote Linux Server

The two keys work together mathematically.

A message encrypted or verified with one key can only be correctly
validated by the matching key.

------------------------------------------------------------------------

# Private Key

The private key:

-   Stays on your computer.
-   Must never be shared.
-   Proves your identity.

Typical location:

``` text
~/.ssh/id_ed25519

or

~/.ssh/id_rsa
```

------------------------------------------------------------------------

# Public Key

The public key:

-   Can be shared safely.
-   Is copied to the remote server.
-   Allows the server to verify your identity.

Typical location:

``` text
~/.ssh/id_ed25519.pub
```

------------------------------------------------------------------------

# Creating an SSH Key Pair

Modern Linux systems recommend the **Ed25519** algorithm.

Generate a new key pair:

``` bash
ssh-keygen -t ed25519
```

Older systems may still use RSA:

``` bash
ssh-keygen -t rsa -b 4096
```

The command creates both the private and public keys.

------------------------------------------------------------------------

# Installing the Public Key

The public key is placed on the server in:

``` text
~/.ssh/authorized_keys
```

When you connect, the server checks whether your public key matches the
private key used by the client.

If they match, access is granted.

------------------------------------------------------------------------

# Copying Your Public Key

A convenient way to install your key is:

``` bash
ssh-copy-id user@server
```

This copies your public key into the remote user's:

``` text
~/.ssh/authorized_keys
```

Afterwards, passwordless authentication is usually available.

------------------------------------------------------------------------

# Using a Specific Private Key

If you have multiple keys:

``` bash
ssh -i ~/.ssh/work_key user@server
```

The `-i` option specifies which private key to use.

------------------------------------------------------------------------

# Linux Perspective

You can inspect your SSH configuration directory:

``` bash
ls -la ~/.ssh
```

Typical files include:

``` text
id_ed25519
id_ed25519.pub
authorized_keys
known_hosts
config
```

Each file serves a different purpose in SSH authentication and
configuration.

------------------------------------------------------------------------

# Production Example

A CI/CD pipeline automatically deploys an application to a production
server.

The deployment tool authenticates using an SSH private key instead of
storing a password in scripts.

This approach improves both security and automation.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   SSH key pairs
-   Private keys
-   Public keys
-   `ssh-keygen`
-   `ssh-copy-id`
-   `authorized_keys`
-   Using custom private keys

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn file transfers with `scp` and `sftp`, inspect the SSH
service, troubleshoot SSH problems, and complete the SSH lesson with
labs, interview questions, and a cheat sheet.
