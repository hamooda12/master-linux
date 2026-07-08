# -exec is special for find.

It means:
``` text
find ... -exec COMMAND {} \;
```

For every file/directory that find discovers, run this command on it.

Example:
``` text
find /srv/techcorp/project-atlas -type d
```

This only shows directories.
-------------------------------------------------------------------------------------------------------
But:
``` text
find /srv/techcorp/project-atlas -type d -exec chmod g+s {} \;
```

Means:

find all directories
for each directory, run:
chmod g+s THAT_DIRECTORY
``` text
{} means:
```
put the found directory here

So if find finds:

/srv/techcorp/project-atlas

/srv/techcorp/project-atlas/docs

/srv/techcorp/project-atlas/releases

It runs:

chmod g+s /srv/techcorp/project-atlas

chmod g+s /srv/techcorp/project-atlas/docs

chmod g+s /srv/techcorp/project-atlas/releases

And \; means:

the -exec command ends here

Think of it like this:

``` text
find [where] [what] -exec [do this] [on each result] [end]
```

Your command:
``` text
sudo find /srv/techcorp/project-atlas -type d -exec chmod g+s {} \;
```

Means:

Inside /srv/techcorp/project-atlas,

find directories only,

then run chmod g+s on every directory found.

# Important: -exec is not a normal Linux command. It is an option/action inside find.
