```text
That last field immediately stands out.

ethan:x:1005:1005:Ethan Miller,,,:/home/ethan:/usr/sbin/nologin
                                               ↑
                                         Login Shell
```

# /usr/sbin/nologin 

is a special shell that ```text intentionally prevents interactive logins```. It's commonly used for service accounts such as web servers or databases that should never allow a user to log in.
