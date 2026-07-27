# DevOps Task #13

## P7-L08 — Linux Firewall Basics

**Student:** Hamad Tarawa  
**Stage:** Part 7 — Linux Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Ubuntu

---

## 1. Connection to the Previous Lesson

In **P7-L07 — SSH Security**, you learned how SSH protects remote
administrative connections.

SSH commonly uses TCP port `22`. However, even a securely configured SSH
service should not automatically be reachable by everyone.

This introduces another Linux security layer: the **firewall**.

SSH protects a connection after it reaches the SSH service. A firewall decides
whether that network traffic is allowed to reach the service in the first
place.

```text
Remote administrator
        |
        | SSH traffic to port 22
        v
     Firewall
   Allow or deny?
        |
        | Allowed
        v
    SSH server
```

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain what a firewall does.
- Distinguish incoming traffic from outgoing traffic.
- Explain the relationship between services and ports.
- Distinguish an allow rule from a deny rule.
- Explain the default-deny approach.
- Explain why only required ports should be open.
- Understand the basic purpose of UFW.
- Inspect UFW status safely.
- Understand why firewall behavior in WSL differs from a normal Linux server.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer deploys services such as:

- SSH
- Nginx
- Spring Boot
- PostgreSQL
- Monitoring systems
- CI/CD runners

Each network service may listen on a port.

If a service is unnecessarily exposed, an unauthorized person may attempt to
connect to it.

For example:

```text
Port 443  -> Nginx HTTPS       -> public access may be required
Port 8080 -> Spring Boot API   -> may need to remain internal
Port 5432 -> PostgreSQL        -> should normally remain private
Port 22   -> SSH               -> should be restricted to administrators
```

A DevOps engineer uses firewall rules to make the implemented network access
match the intended architecture.

---

## 4. Simple Explanation

### 4.1 What is a firewall?

A firewall is a security control that filters network traffic.

It examines traffic and applies rules such as:

```text
Allow this traffic.
Deny this traffic.
```

A firewall rule may consider information such as:

- Direction of traffic
- Network protocol
- Source IP address
- Destination IP address
- Destination port

For this beginner lesson, the most important ideas are:

```text
Direction + Protocol + Port + Allow/Deny
```

---

### 4.2 A firewall is like a security guard

Imagine a server as a building.

- Applications are rooms.
- Ports are doors.
- Network requests are visitors.
- The firewall is the security guard.

The guard checks the rules before allowing a visitor through a door.

```text
Visitor asks for door 443
            |
            v
Firewall checks its rules
            |
       +----+----+
       |         |
     Allow      Deny
       |         |
       v         v
   HTTPS      Connection
   service     blocked
```

The firewall does not create the application service. It controls network
access to it.

---

## 5. Incoming and Outgoing Traffic

### 5.1 Incoming traffic

Incoming traffic starts outside the machine and attempts to reach a service
inside it.

Examples:

- A user opens your website.
- An administrator connects through SSH.
- A monitoring system contacts an application endpoint.

```text
External computer -> Linux server
```

Common incoming firewall questions:

- Should the internet reach port `443`?
- Should only the company network reach port `22`?
- Should anyone reach PostgreSQL on port `5432`?

---

### 5.2 Outgoing traffic

Outgoing traffic starts from the machine and travels to another system.

Examples:

- A server downloads package updates.
- Spring Boot calls an external API.
- A CI/CD runner downloads a dependency.
- A monitoring agent sends data to a monitoring service.

```text
Linux server -> External system
```

Many simple host-firewall configurations focus first on incoming traffic.
However, outgoing traffic can also be controlled when the environment
requires it.

---

### 5.3 Why direction matters

An application server may need:

- Incoming HTTPS connections from users.
- Outgoing connections to an approved external API.

These are separate traffic flows and may need different rules.

```text
User
  |
  | Incoming HTTPS
  v
Application server
  |
  | Outgoing API request
  v
External service
```

---

## 6. Ports and Services

A network port is a numbered connection point used by a service.

Common examples:

| Port | Protocol | Common service |
|---:|---|---|
| `22` | TCP | SSH |
| `53` | TCP/UDP | DNS |
| `80` | TCP | HTTP |
| `443` | TCP | HTTPS |
| `8080` | TCP | Spring Boot application |
| `5432` | TCP | PostgreSQL |

### Important distinction

A port number and a running service are related, but they are not the same
thing.

- A service may listen on a port.
- The firewall may allow or block access to that port.
- Allowing a port does not start the service.
- Stopping the service does not automatically remove a firewall rule.

Example:

```text
Nginx is not running
Firewall allows port 443
Result: No Nginx service is available
```

Another example:

```text
Nginx is running on port 443
Firewall blocks port 443
Result: The service runs, but blocked clients cannot reach it
```

---

## 7. Allow and Deny Rules

### Allow rule

An allow rule permits matching traffic.

Conceptual example:

```text
Allow incoming TCP traffic to port 443
```

This may allow users to reach an HTTPS service.

### Deny rule

A deny rule blocks matching traffic.

Conceptual example:

```text
Deny incoming TCP traffic to port 5432
```

This helps prevent direct public access to PostgreSQL.

### Rules should match the architecture

Suppose your architecture is:

```text
Internet
   |
   | HTTPS :443
   v
Nginx reverse proxy
   |
   | Internal :8080
   v
Spring Boot
   |
   | Private :5432
   v
PostgreSQL
```

The intended access is:

| Port | Intended access |
|---:|---|
| `443` | Public users |
| `22` | Approved administrators only |
| `8080` | Reverse proxy or internal network only |
| `5432` | Spring Boot or private network only |

The firewall should help enforce this design.

---

## 8. The Default-Deny Approach

The default-deny approach means:

> Block incoming traffic unless a rule explicitly allows it.

The mental model is:

```text
Deny incoming traffic by default
            |
            v
Allow only required exceptions
```

Example required exceptions:

```text
Allow SSH from an approved administration network.
Allow HTTPS from users.
Keep application and database ports private.
```

### Why is default deny useful?

Suppose a developer accidentally starts a test service on port `9000`.

With a permissive network policy, that service might become reachable.

With default-deny incoming rules, the service remains blocked unless someone
deliberately adds an allow rule.

Default deny reduces accidental exposure.

It does not mean that the server blocks all useful traffic forever. Required
traffic is added as explicit exceptions.

---

## 9. Defense in Depth

A firewall is one security layer. It should not be the only layer.

For example, an SSH server should still use:

- Secure authentication
- Restricted users
- Disabled direct root login
- Updated software
- Linux permissions

Even when the firewall restricts port `22`.

Likewise, PostgreSQL should still use:

- Database authentication
- Limited database users
- Secure configuration
- Restricted Linux permissions

Even when a firewall blocks public access to port `5432`.

Using several connected protections is called **Defense in Depth**.

---

## 10. Important Terms

| Term | Simple meaning |
|---|---|
| Firewall | A control that filters network traffic |
| Incoming traffic | Traffic entering the machine |
| Outgoing traffic | Traffic leaving the machine |
| Port | A numbered network connection point |
| Protocol | Rules used for network communication, such as TCP or UDP |
| Allow rule | Permits matching traffic |
| Deny rule | Blocks matching traffic |
| Default deny | Blocks traffic unless it is explicitly allowed |
| UFW | Uncomplicated Firewall, a firewall-management tool for Linux |
| Listening port | A port on which a service waits for connections |
| Exposure | Making a service reachable from another system |
| Defense in Depth | Using multiple security layers together |

---

## 11. Introduction to UFW

UFW means:

```text
Uncomplicated Firewall
```

UFW provides simpler commands for managing Linux firewall rules.

It is commonly available on Ubuntu systems.

UFW is a management tool. The Linux kernel performs the actual packet
filtering through the underlying firewall system.

For this lesson, you only need to understand these common UFW operations:

```text
Check status
View rules
Allow required traffic
Deny unwanted traffic
Enable or disable the firewall
```

We will not enable or disable UFW in the practical lab.

---

## 12. Commands and Configuration

### Command 1 — Inspect listening ports

```bash
ss -tuln
```

Purpose:

> Show TCP and UDP ports on which local services are listening.

Important options:

- `t`: TCP
- `u`: UDP
- `l`: Listening
- `n`: Numeric addresses and port numbers

This command shows listening services. It does not tell you by itself whether
those ports are reachable through every firewall or network layer.

---

### Command 2 — Check UFW status

```bash
sudo ufw status
```

Possible result:

```text
Status: inactive
```

This means UFW is not currently enforcing its configured rules.

Another possible result:

```text
Status: active
```

If UFW is active, the output may also show rules.

---

### Command 3 — Show detailed UFW status

```bash
sudo ufw status verbose
```

Purpose:

> Display whether UFW is active and show additional information such as
> default policies and configured rules.

This is a read-only inspection command.

---

### Command 4 — Show numbered rules

```bash
sudo ufw status numbered
```

Purpose:

> Display active UFW rules with numbers.

Rule numbers are useful when an administrator needs to identify a specific
rule. We will not delete or change rules in this lab.

---

### Example allow rules

The following are real configuration commands:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Meaning:

- Allow TCP port `22` for SSH.
- Allow TCP port `80` for HTTP.
- Allow TCP port `443` for HTTPS.

Do not run these commands just because they appear in the lesson.

A firewall rule should only be added when:

1. The service is actually required.
2. The intended source is understood.
3. The current rules have been reviewed.
4. The change will not expose a private service.

---

### A safer SSH rule concept

Instead of allowing SSH from everywhere:

```bash
sudo ufw allow 22/tcp
```

a production server may restrict SSH to an approved source network.

Conceptual example:

```bash
sudo ufw allow from 192.0.2.0/24 to any port 22 proto tcp
```

This is only an example using a documentation-only address range.

Do not copy it into a real environment. The real source network must match the
organization's approved administrator network.

---

### UFW dry-run

UFW supports a dry-run option that displays what a command would do without
applying the firewall change.

Example:

```bash
sudo ufw --dry-run allow 8080/tcp
```

This is useful for learning command behavior safely.

The output may display underlying rule information that looks complicated.
You do not need to understand every generated line in this beginner lesson.

---

## 13. Important WSL Note

UFW behavior inside WSL may differ from a normal Ubuntu server.

Why?

```text
Windows host
     |
     | Windows networking and Windows Firewall
     v
WSL networking layer
     |
     | Linux environment
     v
Linux services and possible UFW rules
```

Windows and WSL use connected but separate networking layers.

As a result:

- Windows Firewall may affect traffic before it reaches WSL.
- WSL networking behavior depends on the installed WSL version and
  configuration.
- A UFW rule inside WSL may not behave exactly like the same rule on a normal
  Ubuntu cloud server.
- Testing only inside WSL does not prove how a production firewall will
  behave.

For this task, you may:

- Explain the firewall concepts.
- Inspect UFW status.
- Document example commands.
- Use a dry-run command.
- State clearly that WSL networking differs from a standard Linux server.

Do not depend on WSL UFW behavior as your only proof of real server firewall
security.

---

## 14. Safe Practical Lab

### Lab goal

Inspect listening ports and UFW status without changing active firewall
behavior.

### Safety rules

In this lab, do not run:

```bash
sudo ufw enable
sudo ufw disable
sudo ufw reset
```

Also do not add or delete active rules.

Enabling or changing a firewall without understanding the environment could
interrupt network access.

---

### Step 1 — Inspect listening services

Run:

```bash
ss -tuln
```

Look for entries containing:

```text
LISTEN
```

Record one listening port if any appear.

If no ports appear, record:

> No listening TCP or UDP services were shown in my current WSL environment.

---

### Step 2 — Check whether UFW is available

Run:

```bash
ufw --version
```

If UFW is installed, you will see version information.

If the result is:

```text
ufw: command not found
```

do not install it only for this lesson. Record that UFW was not available and
complete the conceptual report using the lesson commands.

---

### Step 3 — Inspect UFW status

If UFW is available, run:

```bash
sudo ufw status verbose
```

Possible result:

```text
Status: inactive
```

An inactive result is acceptable in WSL.

Do not enable it for this lab.

---

### Step 4 — Inspect numbered rules

If UFW is available, run:

```bash
sudo ufw status numbered
```

If UFW is inactive or has no rules, there may be very little output. That is
normal.

---

### Step 5 — Perform a safe dry run

If your installed UFW version accepts `--dry-run`, run:

```bash
sudo ufw --dry-run allow 8080/tcp
```

This previews a rule that would allow TCP port `8080`.

It does not apply the rule.

If the command is not supported in your environment, record that and continue.

---

### Step 6 — Explain the intended production design

For this architecture:

```text
Internet -> Nginx :443 -> Spring Boot :8080 -> PostgreSQL :5432
```

Write:

```text
Port 443 should be publicly reachable for HTTPS.
Port 8080 should normally be reachable only by the reverse proxy or internal network.
Port 5432 should normally be reachable only by the application or private network.
Port 22 should be limited to approved administrators.
```

---

## 15. Expected Result

Your output may look similar to:

```text
$ ss -tuln
Netid  State   Local Address:Port
tcp    LISTEN  127.0.0.1:8080
```

This means a TCP service is listening locally on port `8080`.

Your UFW result may be:

```text
$ sudo ufw status verbose
Status: inactive
```

This means UFW is not active in that WSL environment.

It does not prove that the Windows host has no firewall. Windows Firewall is a
separate layer.

If UFW is unavailable:

```text
$ ufw --version
ufw: command not found
```

you may still complete the lesson by documenting:

- The firewall concept.
- The example commands.
- The intended rules for the example architecture.
- The WSL limitation.

---

## 16. Evidence to Save

Save one screenshot showing:

```bash
ss -tuln
```

If UFW is available, save another screenshot showing:

```bash
sudo ufw status verbose
sudo ufw status numbered
```

Optionally save the dry-run result:

```bash
sudo ufw --dry-run allow 8080/tcp
```

### Suggested evidence title

```text
Linux Listening Ports and UFW Status in WSL
```

### Suggested evidence explanation

> I used `ss -tuln` to inspect services listening on TCP and UDP ports. I used
> `ufw status verbose` to inspect the Linux firewall without modifying it.
> UFW was [active/inactive/unavailable] in my WSL environment. WSL and Windows
> have separate networking and firewall layers, so this local result may
> differ from a standard Ubuntu server.

### Important evidence rule

Do not claim that a port is protected only because it did not appear in one
command.

A complete network-access decision may involve:

- The running service
- The Linux host firewall
- The Windows host firewall
- Cloud security groups
- Router or network firewall rules

---

## 17. Common Mistakes

### Mistake 1 — Opening every port

Only services required by the architecture should be reachable.

---

### Mistake 2 — Exposing the database publicly

PostgreSQL should normally be reachable only from approved application or
administration networks.

---

### Mistake 3 — Enabling a firewall remotely before allowing SSH

On a remote server, enabling a default-deny firewall without a correct SSH
rule may block your own administration connection.

Firewall changes require a planned and recoverable procedure.

---

### Mistake 4 — Assuming an allowed port starts the service

A firewall rule only permits traffic. It does not start Nginx, Spring Boot, or
PostgreSQL.

---

### Mistake 5 — Assuming a listening port is public

`ss -tuln` shows local listening sockets. Other network and firewall layers
still determine external reachability.

---

### Mistake 6 — Using only the firewall for security

Services still need authentication, updates, limited users, and secure
configuration.

---

### Mistake 7 — Treating WSL exactly like a production server

WSL networking interacts with the Windows host and may behave differently
from a normal Ubuntu server.

---

## 18. Task Report Notes

You may adapt the following text for your final assignment:

> A firewall protects a Linux system by filtering incoming and outgoing
> network traffic according to security rules. Rules may allow or deny traffic
> based on properties such as protocol, source address, destination address,
> and port number. DevOps engineers use firewalls to expose only the services
> required by the application architecture.

> A default-deny approach blocks incoming traffic unless it is explicitly
> allowed. For a typical web application, HTTPS port 443 may be public, while
> the Spring Boot application port and PostgreSQL database port remain
> internal. SSH access should be limited to approved administrators or trusted
> networks.

> UFW, or Uncomplicated Firewall, provides simplified commands for managing
> Linux firewall rules on Ubuntu. Commands such as `ufw status verbose` can
> inspect the current policy. Firewall testing in WSL may differ from a normal
> Ubuntu server because Windows and WSL use separate but connected networking
> layers.

### Practical demonstration note

> I used `ss -tuln` to inspect listening network ports and
> `ufw status verbose` to inspect UFW without changing the firewall. UFW was
> [active/inactive/unavailable] in my WSL environment. I also documented that
> only required application ports should be exposed.

---

## 19. Five Review Questions

1. What does a firewall do?
2. What is the difference between incoming and outgoing traffic?
3. What does the default-deny approach mean?
4. Why should Spring Boot port `8080` and PostgreSQL port `5432` normally
   remain private behind Nginx?
5. Why may UFW behavior in WSL differ from a normal Ubuntu server?

---

## 20. Lesson Summary

A firewall filters network traffic using allow and deny rules.

```text
Network traffic
      |
      v
Firewall rules
      |
  +---+---+
  |       |
Allow   Deny
  |       |
  v       v
Service  Blocked
```

The main DevOps lessons are:

- Expose only required ports.
- Use default deny and add deliberate exceptions.
- Restrict SSH to approved administrators.
- Keep application and database ports private when possible.
- Remember that allowing a port does not start a service.
- Remember that a listening port is not automatically public.
- Use firewalls together with authentication, permissions, updates, and
  secure service configuration.
- Document WSL networking limitations when performing a local demonstration.

Important inspection commands:

```bash
ss -tuln
ufw --version
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered
```

Safe preview example:

```bash
sudo ufw --dry-run allow 8080/tcp
```

---

## 21. Progress

### Part 7 — Linux Security

- [x] P7-L01 — Linux Security for DevOps Engineers
- [x] P7-L02 — Linux Users and Groups
- [x] P7-L03 — Linux File Permissions
- [x] P7-L04 — Changing Permissions with chmod
- [x] P7-L05 — File Ownership with chown and chgrp
- [x] P7-L06 — sudo and Least Privilege
- [x] P7-L07 — SSH Security
- [x] P7-L08 — Linux Firewall Basics
- [ ] P7-L09 — Linux Process Isolation
- [ ] P7-L10 — Part 7 WSL Lab and Report

---

## Exact Next Lesson

**P7-L09 — Linux Process Isolation**

Do not continue until the student writes:

```text
next
```
