# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 11 --- SSH Fundamentals (Part 2)

------------------------------------------------------------------------

# Connecting with SSH

The basic SSH command is:

``` bash
ssh username@server
```

Example:

``` bash
ssh hamad@192.168.1.20
```

Linux attempts to establish an encrypted connection to the remote SSH
server.

------------------------------------------------------------------------

# Connecting by Hostname

Instead of an IP address, you can use a DNS name.

``` bash
ssh ubuntu@server.example.com
```

If DNS resolves correctly, SSH connects to the server.

------------------------------------------------------------------------

# Using a Different Port

If the SSH server is configured to use another port:

``` bash
ssh -p 2222 hamad@192.168.1.20
```

`-p` specifies the TCP port.

------------------------------------------------------------------------

# First Connection

The first time you connect, SSH displays a message similar to:

``` text
The authenticity of host '192.168.1.20' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

If you trust the server, type:

``` text
yes
```

SSH stores the server's **host key** locally.

------------------------------------------------------------------------

# Host Keys

A host key uniquely identifies an SSH server.

Its purpose is to help detect if you accidentally connect to a different
machine or if someone is attempting a man-in-the-middle attack.

Known host keys are stored in:

``` text
~/.ssh/known_hosts
```

------------------------------------------------------------------------

# Authentication Methods

SSH supports several authentication methods.

## Password Authentication

The server prompts for your password.

``` text
hamad@192.168.1.20's password:
```

------------------------------------------------------------------------

## Public Key Authentication

Instead of a password, SSH verifies a cryptographic key pair.

    Private Key
          │
          ▼
    SSH Client
    ══════════════════════
    SSH Server
          ▲
          │
    Public Key

Public key authentication is the preferred method in production because
it is more secure and easier to automate.

------------------------------------------------------------------------

# Logging Out

To end an SSH session:

``` bash
exit
```

or press:

``` text
Ctrl + D
```

------------------------------------------------------------------------

# Common SSH Options

  Command                   Purpose
  ------------------------- -----------------------------
  `ssh user@host`           Connect to a server
  `ssh -p 2222 user@host`   Connect using a custom port
  `exit`                    Close the session

------------------------------------------------------------------------

# Linux Perspective

Once connected through SSH, you work as though you were sitting at the
remote machine.

You can:

-   View logs
-   Restart services
-   Edit configuration files
-   Monitor processes
-   Deploy applications

SSH is the primary management interface for most Linux servers.

------------------------------------------------------------------------

# Production Example

A cloud virtual machine has no graphical desktop.

Administrators connect via:

``` bash
ssh ubuntu@203.0.113.25
```

After logging in, they perform software updates, deploy applications,
and troubleshoot problems entirely from the terminal.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   SSH command syntax
-   Connecting by IP or hostname
-   Using custom ports
-   Host keys
-   Password authentication
-   Public key authentication
-   Ending an SSH session

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn SSH key pairs, `scp`, `sftp`, and inspect the SSH service
using Linux networking tools.
