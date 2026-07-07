# why is /etc/sudoers.d/ preferred?

Imagine your company has:

10 junior administrators

5 DevOps engineers

3 database administrators

If everyone edits /etc/sudoers, the file becomes:

difficult to read
difficult to audit
prone to merge conflicts and accidental mistakes

Instead, Linux allows you to keep the main file clean and place separate policies in individual files, for example:

``` text
/etc/sudoers.d/
├── noah
├── devops
├── database-admins
├── monitoring
└── backup
```

Each file contains only the rules for that user or role, making management and auditing much easier.

-------------------------------------------------------------------------------------------------------------------------------

# What would we do in a real production environment?

Rather than adding Noah's rules directly to /etc/sudoers, we would create a dedicated file such as:

```text 
/etc/sudoers.d/noah
```

and edit it safely with:

``` text 
sudo visudo -f /etc/sudoers.d/noah
```

This gives us the same syntax checking and file locking benefits while keeping the configuration modular.
