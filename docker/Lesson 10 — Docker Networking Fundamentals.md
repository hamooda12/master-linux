# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 10 — Docker Networking Fundamentals

> **Level:** Beginner  
> **Type:** Guided practical networking lesson  
> **Lab images:** `nginx:alpine` and `alpine:3.22`  
> **Estimated study time:** 130–170 minutes

---

## Overview

In Lesson 09, a client reached Nginx through a published host port:

```text
Client → Docker host port 8080 → container port 80
```

But applications commonly contain several containers:

```text
Frontend container → API container → Database container
```

Those containers need to communicate directly.

In this lesson, you will create a user-defined Docker network, attach two containers, and send an HTTP request using a container name:

```text
client container → http://web:80 → Nginx container
```

You do not need namespaces, virtual Ethernet devices, bridge forwarding tables, or firewall chains yet. First, learn the behavior you will use every day.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain why containers need Docker networks
- List Docker networks
- Recognize bridge, host, and none network modes
- Explain the default bridge at a beginner level
- Create a user-defined bridge network
- Start containers on a selected network
- Inspect network membership and container IP addresses
- Communicate between containers using names
- Explain Docker name resolution on user-defined bridge networks
- Explain why `localhost` cannot reach another container
- Explain why port publishing is unnecessary for same-network communication
- Connect and disconnect a running container
- Use a network alias
- Explain why application configuration should use names instead of hardcoded container IPs
- Remove a custom network safely

---

## Prerequisites

You should understand:

- IP addresses, subnet, gateway, routing, and DNS
- `localhost` and loopback
- TCP ports and listening sockets
- Host port versus container port
- `docker run`, `docker exec`, and `docker inspect`
- Port publishing with `-p`
- Container names

Use your established Docker access model consistently.

---

## Why This Topic Exists

Suppose TechCorp runs:

- `frontend` container
- `product-api` container
- `database` container

The frontend must know where the API is.

A fragile configuration is:

```text
API_URL=http://172.18.0.4:8080
```

That IP may change when the API container is replaced.

A better configuration on a user-defined network is:

```text
API_URL=http://product-api:8080
```

Docker resolves the container/service name to its current network address.

```text
Stable application name → current container IP
```

---

# Part 1 — The Beginner Network Model

## 1. A Container Needs a Network Connection

A normal container can have a network interface, IP address, routes, and DNS configuration inside its network environment.

Beginner view:

```text
Container
├── network interface
├── IP address
├── routing table
└── DNS configuration
```

How Linux creates this isolated network environment is taught later.

## 2. Docker Network as a Shared Segment

Think of a user-defined bridge network as a private virtual network on one Docker host.

```text
Docker host
└── Network: app-net
    ├── web container
    ├── api container
    └── client container
```

Containers attached to the same network can normally communicate directly using their container ports.

## 3. Different Networks Provide Separation

```text
frontend-net
├── proxy
└── frontend

backend-net
├── frontend
├── api
└── database
```

A container connected only to `frontend-net` does not automatically become a member of `backend-net`.

Networks help define which groups of containers can communicate.

This is useful isolation, but it is not a complete security policy by itself.

## 4. Single-host Scope

A bridge network connects containers on the same Docker Engine host.

```text
Host A bridge network ≠ Host B bridge network
```

Multi-host networking uses other designs such as overlay networking or an orchestrator. That comes later.

---

# Part 2 — List Docker Networks

## 5. List Networks

```bash
docker network ls
```

Typical built-in entries on Linux include:

```text
NETWORK ID   NAME      DRIVER   SCOPE
...          bridge    bridge   local
...          host      host     local
...          none      null     local
```

## 6. Read the Columns

| Column | Meaning |
| --- | --- |
| `NETWORK ID` | Docker identifier for the network |
| `NAME` | Human-readable network name |
| `DRIVER` | Networking behavior implementation selected |
| `SCOPE` | Whether network is local to one daemon or has broader scope |

## 7. Built-in Networks

### `bridge`

The default network used by ordinary Linux containers when no network is specified.

### `host`

The container uses the host's network environment rather than normal separate container networking.

### `none`

The container has no normal external network connectivity; loopback remains available.

These modes are introduced later in the lesson without deep internals.

## 8. Do Not Remove Built-in Networks

The built-in `bridge`, `host`, and `none` networks are Docker-managed defaults. Create and remove your own lab networks instead.

---

# Part 3 — Default Bridge

## 9. Automatic Attachment

When you run a Linux container without `--network`, Docker normally attaches it to the default `bridge` network.

```bash
docker run -d --name default-web nginx:alpine
```

Inspect:

```bash
docker inspect --format \
  '{{json .NetworkSettings.Networks}}' \
  default-web
```

## 10. Default Bridge Communication

Containers on the default bridge can communicate using container IP addresses under normal configuration.

However, automatic container-name resolution is not provided there in the same convenient way as on user-defined bridge networks.

Legacy `--link` can provide old-style behavior, but it should not be used for new designs.

## 11. Why Prefer User-defined Bridges

User-defined bridges provide:

- Automatic name resolution between attached containers
- Better separation between application groups
- Ability to connect and disconnect running containers
- Per-network configuration
- Clear ownership by project or application

Therefore, this course uses user-defined networks for multi-container applications.

---

# Part 4 — Create a User-defined Bridge

## 12. Create the Network

```bash
docker network create techcorp-net
```

Expected output:

```text
<network-id>
```

Docker uses the bridge driver by default for a local network unless another driver is selected.

Explicit form:

```bash
docker network create --driver bridge techcorp-net
```

## 13. Verify

```bash
docker network ls --filter name=techcorp-net
```

## 14. Inspect

```bash
docker network inspect techcorp-net
```

Important beginner fields include:

- Name
- ID
- Driver
- Scope
- IPAM configuration
- Containers attached
- Options
- Labels

## 15. Inspect Selected Fields

```bash
docker network inspect --format \
  'name={{.Name}} driver={{.Driver}} scope={{.Scope}}' \
  techcorp-net
```

Subnet and gateway:

```bash
docker network inspect --format \
  '{{json .IPAM.Config}}' \
  techcorp-net
```

Docker normally selects a non-overlapping subnet automatically.

## 16. Network Exists Before Containers

A custom network is its own Docker object.

```text
Network can exist with zero containers
Container can later join network
```

---

# Part 5 — Start Containers on the Network

## 17. Start Nginx as `web`

```bash
docker run -d \
  --name web \
  --network techcorp-net \
  nginx:alpine
```

Notice that no host port is published.

## 18. Start an Alpine Client

```bash
docker run -dit \
  --name client \
  --network techcorp-net \
  alpine:3.22 sh
```

## 19. Verify Membership

```bash
docker network inspect techcorp-net
```

The `Containers` section should include both `web` and `client`.

Formatted view:

```bash
docker network inspect --format \
  '{{range $id, $c := .Containers}}{{$c.Name}} {{$c.IPv4Address}}{{println}}{{end}}' \
  techcorp-net
```

You are not expected to memorize this template. It prints each attached container name and IPv4 address.

## 20. Inspect One Container Address

```bash
docker inspect --format \
  '{{(index .NetworkSettings.Networks "techcorp-net").IPAddress}}' \
  web
```

The exact IP is assigned by Docker and may be different on another host or after recreation.

---

# Part 6 — Communicate by Container Name

## 21. Request Nginx From the Client

```bash
docker exec client wget -qO- http://web:80
```

Expected: Nginx HTML page.

## 22. What Happened

```text
client application asks for name web
              ↓
Docker-provided name resolution on techcorp-net
              ↓
Current web container IP
              ↓
TCP connection to container port 80
              ↓
Nginx response
```

## 23. No `-p` Was Needed

The request stayed between containers on the same Docker network.

```text
Same-network container communication
→ use destination name + container port

Host/external client communication
→ usually use published host port
```

## 24. Verify DNS Resolution

Inside the client:

```bash
docker exec client getent hosts web
```

Alpine's available tools can vary. If `getent` is not present, the successful `wget http://web:80` already proves both name resolution and connectivity.

Do not install packages into the container merely to satisfy one diagnostic command unless the lab requires it.

## 25. Names Are Scoped to the Network

The name `web` is resolvable for containers attached to the same user-defined network.

A container outside `techcorp-net` should not be expected to resolve or reach it through that network.

---

# Part 7 — Why `localhost` Is Wrong

## 26. Test From Client

```bash
docker exec client wget -qO- -T 3 http://127.0.0.1:80
```

This should fail because Nginx is not running inside the `client` container.

## 27. Loopback Meaning

```text
Inside client:
127.0.0.1 → client container

Inside web:
127.0.0.1 → web container

On host:
127.0.0.1 → Docker host
```

Each network environment owns its own loopback.

## 28. Correct Destination

From client to web:

```bash
docker exec client wget -qO- http://web:80
```

Use the other container's network name and its container port.

## 29. Production Configuration Example

Wrong inside API container:

```text
DATABASE_URL=postgresql://localhost:5432/products
```

unless PostgreSQL truly runs inside the same container.

Correct multi-container pattern:

```text
DATABASE_URL=postgresql://database:5432/products
```

where `database` is the other container/service's network name.

---

# Part 8 — Why Not Hardcode Container IPs?

## 30. Container Replacement

Record the current web IP:

```bash
docker inspect --format \
  '{{(index .NetworkSettings.Networks "techcorp-net").IPAddress}}' \
  web
```

Remove and recreate the container:

```bash
docker rm -f web
docker run -d \
  --name web \
  --network techcorp-net \
  nginx:alpine
```

Inspect the IP again.

It may be the same in a small lab due to address reuse, or it may differ. Docker does not promise that a recreated container keeps the same dynamically assigned IP.

## 31. Name Remains the Application Contract

The client still uses:

```bash
docker exec client wget -qO- http://web:80
```

Docker resolves `web` to its current address.

## 32. Stable Name, Replaceable Instance

```text
Application configuration → web:80
Current instance          → IP chosen by Docker
Replacement instance      → possibly another IP
```

This idea prepares you for Kubernetes Services later:

> Applications should depend on stable service discovery names, not individual workload IPs.

## 33. When Static IPs Exist

Docker can assign a chosen IP on a custom subnet, but static container IPs create management responsibilities and are rarely the best first solution for application discovery.

Use names unless a justified network requirement demands static addressing.

---

# Part 9 — Connect a Running Container

## 34. Start Outside the Network

```bash
docker run -dit --name late-client alpine:3.22 sh
```

It joins the default bridge because no network was specified.

## 35. Test Before Connection

```bash
docker exec late-client wget -qO- -T 3 http://web:80
```

It should not resolve/reach `web` through `techcorp-net` because `late-client` is not a member.

## 36. Connect It

```bash
docker network connect techcorp-net late-client
```

## 37. Verify

```bash
docker network inspect techcorp-net
docker inspect --format '{{json .NetworkSettings.Networks}}' late-client
```

The container can belong to multiple networks at the same time.

## 38. Test Again

```bash
docker exec late-client wget -qO- http://web:80
```

It should now succeed.

## 39. Multiple Networks Mental Model

```text
late-client
├── default bridge connection
└── techcorp-net connection
```

Like a server with more than one network interface, the container can have a connection and address on each Docker network.

---

# Part 10 — Disconnect a Container

## 40. Disconnect

```bash
docker network disconnect techcorp-net late-client
```

The container must normally be running for a standard disconnect operation.

## 41. Verify Removal From Network

```bash
docker network inspect techcorp-net
docker inspect --format '{{json .NetworkSettings.Networks}}' late-client
```

## 42. Test Again

```bash
docker exec late-client wget -qO- -T 3 http://web:80
```

It should fail again through the removed network relationship.

## 43. Disconnect Can Break Applications Immediately

Removing a network connection from a running container interrupts new communication paths and may break active services.

Do not use connect/disconnect casually on production containers. Treat them as state-changing network operations.

---

# Part 11 — Network Aliases

## 44. What an Alias Is

A network alias gives a container another name on a specific network.

Example:

```text
Container name: web
Network alias: frontend
```

Both may resolve to the container on that network.

## 45. Create With Alias

Recreate the lab web container if needed:

```bash
docker rm -f web
docker run -d \
  --name web \
  --network techcorp-net \
  --network-alias frontend \
  nginx:alpine
```

Test:

```bash
docker exec client wget -qO- http://frontend:80
```

## 46. Connect With Alias

For a running container:

```bash
docker network connect \
  --alias web-service \
  techcorp-net \
  late-client
```

The alias belongs to that network attachment, not necessarily every network the container uses.

## 47. Good Alias Use

- Provide a service-oriented name
- Support controlled transition between names
- Give one container a role-specific name on one network

Do not use many unclear aliases that make service ownership difficult to understand.

---

# Part 12 — Host and None Modes

## 48. Host Network Mode

On Docker Engine for Linux:

```bash
docker run --network host IMAGE
```

The container shares the host's network environment rather than receiving ordinary separate container networking.

Consequences:

- Container and host use the same network interfaces/ports
- Port conflicts are host port conflicts
- `-p` publication is not meaningful in the normal way
- Network isolation is reduced

Use it only for justified workloads and platform-aware designs.

We will not use it in the main beginner lab.

## 49. None Network Mode

```bash
docker run --rm --network none alpine:3.22 ip addr
```

The container has loopback but no normal Docker network connection.

Useful for:

- Network-isolated batch processing
- Security testing
- Workloads that do not require networking

## 50. Driver Overview

| Driver/mode | Beginner purpose |
| --- | --- |
| `bridge` | Container networking on one Docker host |
| `host` | Share host network environment |
| `none` | No external container networking |
| `overlay` | Multi-host Docker/Swarm networking; later |
| `macvlan` / `ipvlan` | Specialized integration with physical networks; later |

Do not learn advanced drivers before you need them.

---

# Part 13 — Remove the Network

## 51. Removal Requirement

Docker refuses to remove a network that still has connected containers.

```bash
docker network rm techcorp-net
```

If containers remain, read the error and inspect membership.

## 52. Find Connected Containers

```bash
docker network inspect techcorp-net
```

## 53. Safe Cleanup Order

```text
Stop/remove lab containers
          ↓
Verify network has no connected endpoints
          ↓
Remove custom network
```

Commands:

```bash
docker rm -f web client late-client
docker network rm techcorp-net
```

Verify those names belong to your disposable lab containers before using `-f`. In normal operations, prefer graceful stop then remove.

## 54. Avoid Broad Network Prune

```bash
docker network prune
```

removes all unused custom networks after confirmation.

On a shared or important host, inspect and remove the specific intended network instead.

---

## Big Picture

```text
Docker host
│
├── techcorp-net (user-defined bridge)
│   ├── web
│   │    └── Nginx listens on 80
│   └── client
│        └── wget http://web:80
│
└── default bridge
    └── unrelated container
```

Communication rule:

```text
Same user-defined network
→ use container/service name + container port

Host or external client
→ use host address + published host port
```

---

## Commands — Detailed Reference

## 55. List Networks

```bash
docker network ls
docker network ls --filter name=techcorp-net
```

## 56. Create a Network

```bash
docker network create techcorp-net
docker network create --driver bridge techcorp-net
```

## 57. Inspect a Network

```bash
docker network inspect techcorp-net
```

## 58. Start on a Network

```bash
docker run --network techcorp-net IMAGE
```

## 59. Connect a Running Container

```bash
docker network connect techcorp-net CONTAINER
docker network connect --alias NAME techcorp-net CONTAINER
```

## 60. Disconnect

```bash
docker network disconnect techcorp-net CONTAINER
```

## 61. Remove

```bash
docker network rm techcorp-net
```

The network must have no connected containers.

---

## Guided Hands-on Lab — Communicate by Name

### Scenario

TechCorp needs an Nginx service and a diagnostic client on one private application network. The client must reach Nginx using the name `web`, without any published host port.

### Phase A — Baseline

```bash
docker network ls
docker network ls --filter name=techcorp-net
docker ps -a --filter name=web
docker ps -a --filter name=client
```

If names already belong to unrelated containers, choose unique lab names.

### Phase B — Create Network

```bash
docker network create techcorp-net
docker network inspect --format \
  'name={{.Name}} driver={{.Driver}} scope={{.Scope}} ipam={{json .IPAM.Config}}' \
  techcorp-net
```

### Phase C — Run Containers

```bash
docker run -d \
  --name web \
  --network techcorp-net \
  nginx:alpine

docker run -dit \
  --name client \
  --network techcorp-net \
  alpine:3.22 sh
```

### Phase D — Verify Membership

```bash
docker network inspect techcorp-net
docker inspect --format '{{json .NetworkSettings.Networks}}' web
docker inspect --format '{{json .NetworkSettings.Networks}}' client
```

### Phase E — Request by Name

```bash
docker exec client wget -qO- http://web:80
docker logs --tail 20 web
```

Explain why no `-p` mapping was needed.

### Phase F — Prove Localhost Difference

```bash
docker exec client wget -qO- -T 3 http://127.0.0.1:80
```

Then:

```bash
docker exec client wget -qO- http://web:80
```

Explain why one fails and the other succeeds.

### Phase G — Late Connection

```bash
docker run -dit --name late-client alpine:3.22 sh
docker exec late-client wget -qO- -T 3 http://web:80
docker network connect techcorp-net late-client
docker exec late-client wget -qO- http://web:80
```

### Phase H — Disconnect

```bash
docker network disconnect techcorp-net late-client
docker exec late-client wget -qO- -T 3 http://web:80
```

### Phase I — Alias

```bash
docker network connect \
  --alias diagnostic-client \
  techcorp-net \
  late-client
```

Test Nginx again from `late-client`, then inspect its networks.

### Phase J — Container Replacement

```bash
docker inspect --format \
  '{{(index .NetworkSettings.Networks "techcorp-net").IPAddress}}' \
  web

docker rm -f web

docker run -d \
  --name web \
  --network techcorp-net \
  nginx:alpine

docker inspect --format \
  '{{(index .NetworkSettings.Networks "techcorp-net").IPAddress}}' \
  web

docker exec client wget -qO- http://web:80
```

The IP may or may not change in this small lab. The name-based request must remain the intended design.

### Phase K — Safe Cleanup

```bash
docker stop web client late-client
docker rm web client late-client
docker network rm techcorp-net
```

### Acceptance Criteria

You can prove:

- Custom bridge network exists
- Both main containers belong to it
- Docker assigned network addresses
- `client` resolves and reaches `web`
- `localhost` in `client` does not mean `web`
- No host port was required for same-network access
- A running container can be connected and disconnected
- Network alias belongs to a network attachment
- Name-based communication survives the container-replacement design
- Network removal requires disconnected/removed containers

---

## Discovery Questions

### Question 1

Why can `client` access `http://web:80` without `-p`?

<details>
<summary>Explanation</summary>

Both containers are attached to the same user-defined network. They communicate directly using the destination container name and container port.

</details>

### Question 2

Why does `http://localhost:80` inside `client` not reach Nginx?

<details>
<summary>Explanation</summary>

Loopback refers to the client container itself. Nginx runs in the separate `web` container.

</details>

### Question 3

Why should an API use `database:5432` instead of a container IP?

<details>
<summary>Explanation</summary>

The name is the stable discovery identity, while Docker can assign a different IP when the database container is replaced.

</details>

### Question 4

Can one container join more than one Docker network?

<details>
<summary>Explanation</summary>

Yes. It receives a network attachment on each connected network and can communicate according to those memberships.

</details>

### Question 5

Why does Docker refuse to remove a network with attached containers?

<details>
<summary>Explanation</summary>

The network still has active endpoint dependencies. Disconnect or remove those containers first.

</details>

---

## Production Scenario — Hardcoded Database IP

TechCorp configures:

```text
DATABASE_HOST=172.20.0.5
```

The database container is replaced during maintenance and receives `172.20.0.7`. The API continues contacting the old address and fails.

Evidence:

```bash
docker network inspect product-net
docker inspect --format '{{json .NetworkSettings.Networks}}' database
docker inspect --format '{{json .Config.Env}}' product-api
```

Root cause:

```text
Application configuration depended on an ephemeral container IP instead of the stable network name.
```

Correction:

```text
DATABASE_HOST=database
```

Then recreate the API container through the deployment source so it receives the corrected configuration.

Verify from the API network environment using an application-level database check, not only ping.

---

## Troubleshooting Workflow

## 62. Name Does Not Resolve

Check both containers' network memberships:

```bash
docker inspect --format '{{json .NetworkSettings.Networks}}' SOURCE
docker inspect --format '{{json .NetworkSettings.Networks}}' DESTINATION
docker network inspect NETWORK
```

Common cause: containers are not on the same user-defined network.

## 63. Name Resolves but Connection Is Refused

Likely layers:

- Wrong container port
- Application not listening
- Application stopped/crashed
- Application bound only to loopback

Check:

```bash
docker ps -a --filter name=DESTINATION
docker logs --tail 100 DESTINATION
```

## 64. Connection Times Out

Check:

- Network membership
- Correct destination name/IP
- Container state
- Application behavior
- Firewall/network rules

Do not assume DNS merely because the command used a name.

## 65. Network Removal Fails

```bash
docker network inspect NETWORK
```

Identify connected containers and remove/disconnect only intended resources.

## 66. Container Has Multiple Networks

```bash
docker inspect --format '{{json .NetworkSettings.Networks}}' CONTAINER
```

Verify which network supplies the destination name and route. Do not assume the first displayed IP is always the correct path.

---

## Common Mistakes

### 1. Using localhost for another container

Loopback always refers to the current network environment.

### 2. Publishing ports for all container-to-container traffic

Containers on the same network normally use container ports directly.

### 3. Hardcoding container IPs

Dynamic addresses can change when containers are replaced.

### 4. Using the default bridge for a multi-container application

User-defined networks provide better name resolution and separation.

### 5. Using legacy `--link`

Use user-defined networks and DNS-based names.

### 6. Assuming containers with different networks can resolve each other

They need shared network membership or another deliberately exposed path.

### 7. Removing a network before containers

Docker refuses while endpoints are attached.

### 8. Installing troubleshooting tools into production containers

Use existing evidence or dedicated diagnostic techniques rather than changing the workload casually.

### 9. Treating bridge networks as a complete firewall policy

They provide useful separation but need broader security design.

### 10. Jumping into bridge internals too early

First master network membership, names, ports, and verification.

---

## Best Practices

- Create a user-defined network per application boundary
- Use service-oriented names
- Use container ports for same-network communication
- Publish only ports needed by host/external clients
- Avoid hardcoded container IPs
- Separate frontend and backend networks when architecture requires it
- Connect each container only to networks it needs
- Inspect membership before changing it
- Use application-level connection tests
- Remove application containers before removing their network
- Avoid legacy links
- Document network ownership and intended communication paths

---

## Interview Questions

### Junior

1. What is a Docker network?
2. What is the default bridge network?
3. Why create a user-defined bridge?
4. Can containers on the same custom network communicate by name?
5. Is `-p` required for containers on the same network?
6. What does localhost mean inside a container?
7. Why should container IPs not be hardcoded?
8. Can one container join multiple networks?
9. What does `docker network connect` do?
10. Why can a network not be removed while containers use it?

### Intermediate

1. How does a user-defined bridge improve service discovery?
2. How do you prove two containers share a network?
3. Why might a name resolve but the connection still fail?
4. How do network aliases differ from container names?
5. What is the operational impact of disconnecting a running container?
6. When is host networking appropriate?
7. What does none networking provide?
8. How would you design frontend and backend network membership?

### Senior Preview

Network namespaces, veth pairs, Linux bridges, Docker DNS internals, iptables/nftables isolation, conntrack, IPAM, overlay networks, and Kubernetes CNI are intentionally deferred.

---

## Lesson Reflection

Answer in your own words:

1. What problem does a Docker network solve?
2. Explain default bridge versus user-defined bridge.
3. Why can `client` use `web:80`?
4. Why is publishing unnecessary in that case?
5. Explain three meanings of localhost: host, web, and client.
6. Why is a name better than a container IP?
7. What happens when a container joins two networks?
8. What is a network alias?
9. Explain bridge, host, and none in one sentence each.
10. Write an evidence order for a failed container-to-container request.

---

## Summary

- Docker networks connect groups of containers
- Bridge networks operate on one Docker host
- Containers use the default bridge when no network is selected
- User-defined bridges provide automatic name resolution
- Containers on the same custom network communicate by name or IP
- Name plus container port is the normal same-network endpoint
- Host port publishing is not required for same-network traffic
- Localhost refers to the current container, not another container
- Container IP addresses may change after replacement
- Names provide stable discovery
- Running containers can connect to and disconnect from custom networks
- One container can join multiple networks
- Network aliases provide additional names on a network
- Host mode shares host networking; none mode removes normal connectivity
- A custom network cannot be removed while containers remain attached

---

## Cheat Sheet

### Create and inspect

```bash
docker network ls
docker network create techcorp-net
docker network inspect techcorp-net
```

### Run on a network

```bash
docker run -d \
  --name web \
  --network techcorp-net \
  nginx:alpine

docker run -dit \
  --name client \
  --network techcorp-net \
  alpine:3.22 sh
```

### Test by name

```bash
docker exec client wget -qO- http://web:80
```

### Connect and disconnect

```bash
docker network connect techcorp-net CONTAINER
docker network connect --alias ALIAS techcorp-net CONTAINER
docker network disconnect techcorp-net CONTAINER
```

### Remove

```bash
docker network rm techcorp-net
```

### Modes

```bash
docker run --network bridge IMAGE
docker run --network host IMAGE
docker run --network none IMAGE
```

### Critical distinctions

```text
Container name ≠ hostname/IP guarantee
localhost ≠ another container
container port ≠ published host port
default bridge ≠ user-defined bridge
bridge network ≠ multi-host network
network separation ≠ complete security policy
```

---

## Challenge — The API Cannot Find the Database

### Incident Setup

Create the network and server:

```bash
docker network create challenge-net
docker run -d \
  --name database \
  --network challenge-net \
  nginx:alpine
```

Create the client outside the network:

```bash
docker run -dit --name api alpine:3.22 sh
```

### Reported Symptom

```bash
docker exec api wget -qO- -T 3 http://database:80
```

fails.

### Mission

1. Prove both containers run
2. Inspect both network memberships
3. Inspect `challenge-net`
4. Explain why the name is unavailable to `api`
5. Apply the smallest correction without recreating the API
6. Verify name resolution through an HTTP request
7. Prove localhost still does not mean database
8. Disconnect the API and prove failure returns
9. Reconnect with alias `product-db`
10. Verify `http://product-db:80`
11. Clean up containers and network safely

### Rules

- No host port publishing
- No hardcoded IP
- No legacy `--link`
- No network prune
- Evidence before connect/disconnect

### Report Template

```text
API networks before:
Database networks:
Shared network before correction:
Root cause:
Correction command:
Verification by database name:
Localhost test:
Disconnect result:
Alias result:
Cleanup proof:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-10/
├── 01-network-baseline.txt
├── 02-techcorp-network.txt
├── 03-container-membership.txt
├── 04-name-resolution-request.txt
├── 05-localhost-failure.txt
├── 06-late-client-before.txt
├── 07-late-client-connected.txt
├── 08-late-client-disconnected.txt
├── 09-web-replacement.txt
├── 10-network-cleanup.txt
└── network-incident-report.md
```

Suggested captures:

```bash
docker network inspect techcorp-net > 02-techcorp-network.txt
docker exec client wget -qO- http://web:80 \
  > 04-name-resolution-request.txt
docker inspect --format '{{json .NetworkSettings.Networks}}' late-client \
  > 07-late-client-connected.txt
```

Do not store secrets or unrelated network information.

---

## Official References

- [Docker networking overview](https://docs.docker.com/engine/network/)
- [Docker bridge network driver](https://docs.docker.com/engine/network/drivers/bridge/)
- [Docker network create](https://docs.docker.com/reference/cli/docker/network/create/)
- [Docker network inspect](https://docs.docker.com/reference/cli/docker/network/inspect/)
- [Docker network connect](https://docs.docker.com/reference/cli/docker/network/connect/)
- [Docker network disconnect](https://docs.docker.com/reference/cli/docker/network/disconnect/)
- [Docker network remove](https://docs.docker.com/reference/cli/docker/network/rm/)

---

## Next Lesson

**Lesson 11 — Container Writable Layer and Data Loss**

You can now connect application containers. Next, you will discover what happens to files created inside a container.

The lab will:

1. Create a file inside a container
2. Stop and start the same container
3. Prove the file remains
4. Remove the container
5. Create a new container from the same image
6. Prove the file is gone

This experience will prepare you to understand why Docker volumes and bind mounts exist.

---

## Completion Check

Before moving forward, explain without reading:

```text
Default bridge versus user-defined bridge
Why user-defined networks provide better discovery
Name plus container port
Why -p is unnecessary for same-network traffic
Why localhost cannot reach another container
Why container IPs should not be hardcoded
Connect, disconnect, and alias
Multiple network membership
Bridge, host, and none modes
Safe network removal order
```

If you can make `client` reach `web:80` and explain every step, you are ready for container data behavior.
