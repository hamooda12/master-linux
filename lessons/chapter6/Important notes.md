1 - What information did we gain?
---
The key part is:

# Could not resolve host

This is not the same as:
``` text
Connection refused
Connection timed out
No route to host
```
----
Instead, it tells us that the hostname could not be translated into an IP address.

--------------------

# 2 - What did the output reveal?

The important lines are:
``` text
communications error to 127.0.0.53#53: timed out
```

no servers could be reached
``` text

127.0.0.53 is Ubuntu’s local DNS stub resolver. The PC sent a DNS request to it, but it did not receive an answer.
```

This is different from NXDOMAIN:

``` text
NXDOMAIN
```

means the DNS server answered, but said the hostname does not exist.

Here:
``` text
no servers could be reached
```
---
# 3 - ignoe DNS

why ignore-auto-dns was necessary: 

``` text
sudo nmcli connection modify "Office-LAN" ipv4.ignore-auto-dns yes
```


Without it, NetworkManager could continue accepting 192.168.10.99 from DHCP and use it alongside—or ahead of—the manually configured server.

means DNS resolution itself is failing.
