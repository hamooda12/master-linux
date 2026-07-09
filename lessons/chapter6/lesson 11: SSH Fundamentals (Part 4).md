# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 11 --- SSH Fundamentals (Part 4)

------------------------------------------------------------------------

# File Transfer with SCP

SSH is not only used for remote login.

It also provides a secure way to transfer files between computers.

The most common tool is **SCP (Secure Copy Protocol)**.

Copy a file from your computer to a remote server:

``` bash
scp backup.tar.gz hamad@192.168.1.20:/home/hamad/
```

Copy a file from a remote server to your local computer:

``` bash
scp hamad@192.168.1.20:/var/log/syslog .
```

SCP encrypts all transferred data using SSH.

------------------------------------------------------------------------

# File Transfer with SFTP

**SFTP (SSH File Transfer Protocol)** is another secure file transfer
method built on SSH.

Start an SFTP session:

``` bash
sftp hamad@192.168.1.20
```

Common commands:

``` text
ls
pwd
cd
put file.txt
get report.log
exit
```

SFTP provides an interactive shell for transferring files securely.

------------------------------------------------------------------------

# Verifying the SSH Service

To verify that the SSH server is listening:

``` bash
ss -tulpn | grep :22
```

Example output:

``` text
tcp LISTEN 0 128 0.0.0.0:22 users:(("sshd",pid=821))
```

This confirms that the SSH daemon is accepting connections on TCP port
22.

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Connection Refused

``` bash
ssh hamad@192.168.1.20
```

Output:

``` text
ssh: connect to host 192.168.1.20 port 22: Connection refused
```

Possible causes:

-   SSH service is not running.
-   SSH is listening on a different port.
-   Firewall blocks the connection.

------------------------------------------------------------------------

## Scenario 2 --- Permission Denied

``` text
Permission denied (publickey,password)
```

Possible causes:

-   Wrong username.
-   Incorrect password.
-   Public key not installed.
-   Incorrect file permissions on `.ssh` files.

------------------------------------------------------------------------

## Scenario 3 --- Connection Timeout

``` text
Connection timed out
```

Possible causes:

-   Server unreachable.
-   Network routing issue.
-   Firewall blocking SSH.
-   Wrong IP address.

------------------------------------------------------------------------

# Troubleshooting Workflow

``` text
Verify network connectivity
        │
        ▼
Check SSH port with ss
        │
        ▼
Verify SSH service is running
        │
        ▼
Confirm username and authentication
        │
        ▼
Review server logs if needed
```

------------------------------------------------------------------------

# Best Practices

-   Use SSH key authentication whenever possible.
-   Protect private keys with a passphrase.
-   Never share private keys.
-   Disable password authentication in production when appropriate.
-   Verify the server fingerprint before trusting a new host.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is SSH?
2.  What port does SSH use by default?
3.  What is the difference between SSH and Telnet?

## Intermediate

1.  Explain the difference between public and private keys.
2.  What is the purpose of `authorized_keys`?
3.  When would you use `scp` instead of `sftp`?

## Advanced

1.  A server refuses SSH connections. How would you investigate?
2.  Why is SSH key authentication preferred over passwords in
    production?

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ssh localhost
```

Inspect your SSH configuration:

``` bash
ls -la ~/.ssh
```

Verify the SSH listening socket:

``` bash
ss -tulpn | grep :22
```

Answer:

1.  Is SSH listening on port 22?
2.  Which SSH-related files exist in `~/.ssh`?
3.  Which process owns port 22?

------------------------------------------------------------------------

# Challenge Exercise

A cloud virtual machine is reachable by `ping`, but SSH connections fail
with:

``` text
Connection refused
```

Describe an investigation using the Linux networking concepts learned in
this chapter to identify the root cause.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 11 you understand:

-   SSH fundamentals
-   Secure remote login
-   Password and key authentication
-   Host keys
-   `ssh`
-   `scp`
-   `sftp`
-   `ssh-keygen`
-   `ssh-copy-id`
-   SSH troubleshooting methodology

SSH is the primary remote administration tool for Linux servers and is
used daily by system administrators, DevOps engineers, and cloud
engineers.

------------------------------------------------------------------------

# Cheat Sheet

  Command                    Purpose
  -------------------------- ----------------------------------
  `ssh user@host`            Connect to a remote server
  `ssh -p PORT user@host`    Connect using a custom port
  `ssh-keygen -t ed25519`    Generate an SSH key pair
  `ssh-copy-id user@host`    Install a public key on a server
  `scp source destination`   Securely copy files
  `sftp user@host`           Start an SFTP session
  `ss -tulpn \| grep :22`    Verify SSH is listening

------------------------------------------------------------------------

# What's Next

Lesson 12 --- Linux Firewall Fundamentals

You'll learn how Linux firewalls control network traffic, understand
inbound and outbound rules, explore UFW, and prepare for production
firewall management.
