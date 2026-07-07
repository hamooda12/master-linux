# A sudoers rule has this general format:

``` text
user HOSTS=(RUN_AS:GROUP) COMMANDS
```

Let's break it down.
``` text 
noah    ALL=(root)    /usr/bin/df -h
│       │    │              │
│       │    │              └── Command allowed
│       │    └──────────────── Run as root
│       └───────────────────── Any host
└───────────────────────────── User

```
---------------------------------------------------------------------------------------------------------------------------------

In our case:
``` text
User: noah
Host: ALL (the rule applies on this server)
Run as: root
Commands: only the approved commands
```
Why use full paths?

We don't write:
``` text
df -h
```
We write:

``` text
/usr/bin/df -h
```

This prevents ambiguity and improves security. sudo compares the exact executable path.

You can discover command paths with:
``` text
which df

which free

which ps

which journalctl

which systemctl
```

On a typical Ubuntu system they are:
``` text

/usr/bin/df

/usr/bin/free

/usr/bin/ps

/usr/bin/journalctl

/usr/bin/systemctl

Noah's policy
```
---------------------------------------------------------------------------------------------------------

# The rule we would place in /etc/sudoers.d/noah is:

``` text 
noah ALL=(root) /usr/bin/df -h
noah ALL=(root) /usr/bin/free -h
noah ALL=(root) /usr/bin/ps aux
noah ALL=(root) /usr/bin/journalctl -u *
noah ALL=(root) /usr/bin/systemctl status *
noah ALL=(root) /usr/bin/systemctl restart backend.service
```
