# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 09 — Port Publishing

> **Level:** Beginner to intermediate  
> **Type:** Networking and troubleshooting lesson  
> **Primary lab image:** `nginx:alpine`  
> **Estimated study time:** 140–180 minutes

---

## Overview

An Nginx process can be running successfully inside a container while this command fails on the Docker host:

```bash
curl http://127.0.0.1:8080
```

Why?

The application may listen on a container port, but Docker has not mapped a host port to it.

In this lesson, you will connect four ideas:

```text
Application listening socket inside container
                    ↓
Container port
                    ↓ Docker publishing rule
Host port and host listening address
                    ↓
Client connection
```

You will publish Nginx with:

```bash
docker run -p 8080:80 nginx:alpine
```

and prove the complete path with Docker inspection, `docker port`, `ss`, and `curl`.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Distinguish a container port from a host port
- Read `HOST_PORT:CONTAINER_PORT` in the correct direction
- Explain what `-p` and `--publish` do
- Explain why a running web container may be unreachable from a host port
- Publish a TCP service on a chosen host port
- Bind a published port to loopback only
- Explain the security difference between `0.0.0.0` and `127.0.0.1`
- Use `docker port` and inspection to verify mappings
- Use `ss` to inspect host-side sockets
- Use `curl` to verify application behavior
- Explain host `localhost` versus container `localhost`
- Explain why `EXPOSE` does not publish a port
- Use `-P` and understand its dynamic host-port behavior
- Diagnose wrong port, wrong protocol, port conflict, and application-listening failures

---

## Prerequisites

You should understand:

- IP address and port
- TCP and UDP
- Listening socket
- `localhost`, `127.0.0.1`, and `0.0.0.0`
- `ss -ltnp`
- `curl`
- Container lifecycle
- `docker inspect`, `docker logs`, and `docker exec`
- Image configuration metadata

Use your established Docker access model.

---

## Why This Topic Exists

A container commonly has its own network environment.

If Nginx listens on TCP port 80 inside that environment, it does not automatically mean that the Docker host listens on TCP port 80 or 8080.

```text
Container:
Nginx listens on 0.0.0.0:80

Host:
No published mapping

Result:
curl host:8080 → connection fails
```

Port publishing creates a controlled path from a host address and port to the container's address and port.

---

# Part 1 — Rebuild the Port Mental Model

## 1. What a Port Identifies

An IP address identifies a network endpoint/interface. A port identifies a service endpoint for a transport protocol on that address.

```text
127.0.0.1:8080/tcp
│         │    └── protocol
│         └────── port
└──────────────── IP address
```

TCP port 80 and UDP port 80 are different transport endpoints.

## 2. Listening Address

A process can listen on:

```text
127.0.0.1:8080 → only loopback on that network environment
192.168.1.31:8080 → one specific interface address
0.0.0.0:8080 → all available IPv4 addresses
[::]:8080 → IPv6 unspecified/all-address binding, with platform-specific dual-stack effects
```

Listening address matters independently from the port number.

## 3. Same Port Number in Different Network Environments

The host and a container can each have a service on port 80 because they normally have separate network environments.

```text
Docker host network environment     → port 80
Container A network environment     → port 80
Container B network environment     → port 80
```

Port conflicts happen when two processes/mappings compete for the same address, port, and protocol in the **same** network environment.

---

# Part 2 — Container Port Versus Host Port

## 4. Container Port

The container port is where the application listens inside the container network environment.

For the Nginx image, the intended service port is commonly:

```text
80/tcp
```

## 5. Host Port

The host port is the port on the Docker host that clients use when accessing a published service.

Example:

```text
Host: 127.0.0.1:8080
```

## 6. Port Mapping

```bash
docker run -p 8080:80 nginx:alpine
```

Read it left to right:

```text
8080 → host port
80   → container port
```

Traffic path:

```text
Client connects to Docker host port 8080
                 ↓
Docker publishing/firewall translation
                 ↓
Container TCP port 80
                 ↓
Nginx
```

## 7. Memory Rule

```text
-p OUTSIDE:INSIDE
-p HOST:CONTAINER
```

Do not memorize only the colon. Always say:

> Host 8080 maps to container 80.

## 8. The Ports Can Match

```bash
docker run -p 80:80 nginx:alpine
```

This maps host port 80 to container port 80.

Matching numbers are optional, not required.

## 9. Why Use Different Numbers?

```bash
docker run -p 8080:80 nginx:alpine
docker run -p 8081:80 nginx:alpine
```

Two containers can both listen on port 80 internally while different host ports select them.

```text
Host 8080 → Container A port 80
Host 8081 → Container B port 80
```

---

# Part 3 — Publish Nginx

## 10. Run With a Loopback-only Mapping First

For a safe learning default:

```bash
docker run -d \
  --name published-web \
  -p 127.0.0.1:8080:80 \
  nginx:alpine
```

Breakdown:

| Part | Meaning |
| --- | --- |
| `-d` | Run detached |
| `--name published-web` | Assign container name |
| `-p` | Publish a port |
| `127.0.0.1` | Bind on host IPv4 loopback |
| `8080` | Host port |
| `80` | Container port |
| `nginx:alpine` | Image |

## 11. Verify Container State

```bash
docker ps --filter name=published-web
```

Typical `PORTS` shape:

```text
127.0.0.1:8080->80/tcp
```

Read:

```text
host 127.0.0.1:8080 → container port 80/TCP
```

## 12. Test With Curl

```bash
curl http://127.0.0.1:8080
```

Expected: Nginx HTML response.

Headers only:

```bash
curl -I http://127.0.0.1:8080
```

Verbose connection evidence:

```bash
curl -v http://127.0.0.1:8080
```

## 13. What Curl Proves

A successful HTTP response proves several layers together:

- Host name/address resolved or was explicit
- TCP connection to host port succeeded
- Docker forwarding path worked
- Container service accepted the connection
- Nginx returned an HTTP response

It does not prove that every application route, dependency, or future request will succeed.

---

# Part 4 — Verify the Mapping

## 14. `docker port`

```bash
docker port published-web
```

Typical output:

```text
80/tcp -> 127.0.0.1:8080
```

This command presents the mapping from the container-port point of view.

## 15. Query One Container Port

```bash
docker port published-web 80/tcp
```

Expected:

```text
127.0.0.1:8080
```

## 16. Inspect Port Bindings

Creation configuration:

```bash
docker inspect --format \
  '{{json .HostConfig.PortBindings}}' \
  published-web
```

Runtime network view:

```bash
docker inspect --format \
  '{{json .NetworkSettings.Ports}}' \
  published-web
```

These answer related but different questions:

```text
HostConfig.PortBindings → requested host binding configuration
NetworkSettings.Ports   → runtime port mapping information
```

## 17. Inspect Host Socket With `ss`

```bash
sudo ss -ltnp | grep ':8080'
```

Flags:

```text
-l → listening
-t → TCP
-n → numeric address/port
-p → process information when permitted
```

Depending on Docker networking mode and implementation, process attribution may show `docker-proxy`, another component, or no conventional owning userspace process. Docker may implement forwarding primarily through firewall/NAT rules.

The key evidence is the listening/published address and successful traffic path, not memorizing one process name.

## 18. Cross-check

Use all four:

```bash
docker ps --filter name=published-web
docker port published-web
sudo ss -ltnp | grep ':8080'
curl -v http://127.0.0.1:8080
```

Each provides a different view:

| Command | Evidence |
| --- | --- |
| `docker ps` | Container state and summarized mapping |
| `docker port` | Docker port mapping |
| `ss` | Host-side socket/listening view |
| `curl` | End-to-end service behavior |

---

# Part 5 — Host Binding Address and Security

## 19. Omitted Host Address

```bash
docker run -d -p 8080:80 nginx:alpine
```

By default, Docker publishes to all host addresses, commonly shown as:

```text
0.0.0.0:8080->80/tcp
[::]:8080->80/tcp
```

This can make the service reachable through the host's LAN or public interfaces, subject to routing, firewall behavior, cloud controls, and Docker configuration.

## 20. Loopback-only IPv4

```bash
docker run -d -p 127.0.0.1:8080:80 nginx:alpine
```

Only clients on the Docker host can normally reach this mapping through IPv4 loopback.

## 21. Loopback-only IPv6

```bash
docker run -d -p '[::1]:8080:80' nginx:alpine
```

The brackets prevent ambiguity between IPv6 colons and mapping separators.

To publish on both loopback families, specify both mappings if supported and required:

```bash
docker run -d \
  -p 127.0.0.1:8080:80 \
  -p '[::1]:8080:80' \
  nginx:alpine
```

## 22. Specific Host Interface

If the host owns `192.168.1.31`:

```bash
docker run -d -p 192.168.1.31:8080:80 nginx:alpine
```

This requests a binding on that host address.

The address must belong to the host and must be available for binding.

## 23. Secure Default for Local Development

Prefer:

```bash
-p 127.0.0.1:HOST_PORT:CONTAINER_PORT
```

when only the same host needs access.

Use an all-interface or external-interface binding only when remote access is intended and protected.

## 24. Docker and Host Firewalls

Docker creates firewall/NAT rules for published ports. On Linux, Docker's published-port rules can interact with or bypass traffic paths you expected UFW/firewalld to control.

Do not assume:

```text
UFW says deny → Docker-published port must be unreachable
```

Verify from the actual client network and design rules using Docker's documented firewall integration, including the `DOCKER-USER` chain where appropriate.

Deep firewall integration is covered later.

---

# Part 6 — `localhost` Means “This Network Environment”

## 25. Host Localhost

On the Docker host:

```text
127.0.0.1 → Docker host itself
```

Therefore:

```bash
curl http://127.0.0.1:8080
```

connects to host loopback port 8080, then Docker forwards to the container mapping.

## 26. Container Localhost

Inside a normal container:

```text
127.0.0.1 → that container's own network environment
```

It does not refer to the Docker host or another container.

## 27. Demonstrate

With `published-web` running:

```bash
docker exec published-web sh -c \
  'wget -qO- http://127.0.0.1:80 | head'
```

This reaches Nginx inside its container on port 80.

Inside that container:

```text
127.0.0.1:80 → same container's Nginx
```

On the host:

```text
127.0.0.1:8080 → host mapping → container:80
```

## 28. Common Wrong Assumption

Inside an application container:

```text
localhost:5432
```

means PostgreSQL must be running in the same container network environment.

It does not mean “find my database container.” Container-to-container networking and DNS are taught next.

---

# Part 7 — `EXPOSE`

## 29. What `EXPOSE` Is

Dockerfile example:

```dockerfile
EXPOSE 80
```

It records that the image is intended to listen on port 80, with TCP as the default protocol.

## 30. What `EXPOSE` Does Not Do

`EXPOSE` does not publish a host port.

```text
EXPOSE 80 → image documentation/metadata
-p 8080:80 → runtime host-to-container publication
```

## 31. Inspect Exposed-port Metadata

```bash
docker image inspect --format \
  '{{json .Config.ExposedPorts}}' \
  nginx:alpine
```

Typical shape:

```text
{"80/tcp":{}}
```

## 32. An Application Can Listen Without EXPOSE

If an application listens on port 5000 but the image does not declare `EXPOSE 5000`, you can still publish it explicitly:

```bash
docker run -p 8080:5000 IMAGE
```

provided the application really listens on container port 5000.

## 33. EXPOSE Cannot Make an Application Listen

Declaring:

```dockerfile
EXPOSE 9000
```

does not start a server on port 9000.

The application must bind a socket itself.

---

# Part 8 — Publish All Exposed Ports With `-P`

## 34. Uppercase `-P`

```bash
docker run -d --name auto-published -P nginx:alpine
```

`-P` is short for:

```text
--publish-all
```

Docker publishes all exposed image ports to dynamically selected high host ports.

## 35. Discover the Selected Port

```bash
docker port auto-published
```

Example shape:

```text
80/tcp -> 0.0.0.0:49153
```

The actual host port is chosen dynamically.

## 36. Test the Selected Port

If Docker selected `49153`:

```bash
curl http://127.0.0.1:49153
```

Use your actual output, not the example number.

## 37. `-p` Versus `-P`

| `-p` | `-P` |
| --- | --- |
| Explicit mapping | Automatically maps all exposed ports |
| You choose host port/address | Docker selects host ports |
| Predictable endpoint | Must be discovered |
| Preferred for normal defined services | Useful for tests or collision avoidance |

Uppercase and lowercase matter.

---

# Part 9 — TCP and UDP

## 38. Default Protocol

If no protocol is written:

```bash
-p 8080:80
```

Docker assumes TCP:

```bash
-p 8080:80/tcp
```

## 39. UDP Mapping

```bash
docker run -p 5353:53/udp IMAGE
```

This publishes host UDP 5353 to container UDP 53.

It does not publish TCP.

## 40. Publish Both Protocols

```bash
docker run \
  -p 5353:53/tcp \
  -p 5353:53/udp \
  IMAGE
```

TCP and UDP use separate transport namespaces, so the same numeric host port can be mapped for both protocols.

## 41. Verification Commands Differ

TCP listeners:

```bash
sudo ss -ltnp
```

UDP sockets:

```bash
sudo ss -lunp
```

`curl` normally tests HTTP over TCP, not arbitrary UDP services.

Choose a protocol-appropriate client.

---

# Part 10 — Common Failures

## 42. Wrong Host Port

Container mapping:

```text
127.0.0.1:8080 → container:80
```

Wrong test:

```bash
curl http://127.0.0.1:80
```

Correct test:

```bash
curl http://127.0.0.1:8080
```

The client uses the host port.

## 43. Reversed Mapping

Mistake:

```bash
docker run -p 80:8080 nginx:alpine
```

This maps host 80 to container 8080. Standard Nginx is normally listening on container 80, not 8080.

Docker can create a mapping to a port where no application listens. Publication does not verify application availability.

## 44. Host Port Already Allocated

Symptom shape:

```text
bind: address already in use
port is already allocated
```

Investigate:

```bash
sudo ss -ltnp | grep ':8080'
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Correction options:

- Stop/remove the conflicting service if authorized and appropriate
- Select a different host port
- Bind to a different host address when design permits

Do not kill an unknown process merely to free the number.

## 45. Container Not Running

```bash
docker ps -a --filter name=CONTAINER
docker logs --tail 100 CONTAINER
docker inspect --format \
  'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}}' \
  CONTAINER
```

Port diagnosis begins only after process state is understood.

## 46. Application Listens on Wrong Container Port

Mapping:

```text
host 8080 → container 80
```

Application actually listens on:

```text
container 8080
```

Result: mapping reaches an unused container port.

Inspect inside when authorized:

```bash
docker exec CONTAINER ss -ltn
```

Minimal images may not include `ss`. Use image documentation, application logs, or a suitable diagnostic container later rather than installing tools into production containers blindly.

## 47. Application Binds Only to Container Loopback

An application binding only to:

```text
127.0.0.1:5000 inside container
```

may not accept traffic arriving through the container's non-loopback interface.

For ordinary bridge-network access, applications commonly listen on:

```text
0.0.0.0:5000 inside container
```

Security exposure is then controlled through Docker networks and host publishing, not by incorrectly binding only to container loopback when external container traffic is required.

## 48. Host Firewall or Cloud Firewall

Local curl may succeed while a remote client fails.

Possible layers:

- Host bind address is loopback only
- Host firewall
- Docker firewall rules
- Cloud security group/network ACL
- Router/NAT
- Remote route
- Client DNS

Test from both the host and intended remote network.

---

## Big Picture

```text
Remote/local client
        │ connects to HOST_IP:HOST_PORT
        ▼
Docker host network
        │ published mapping
        ▼
Container_IP:CONTAINER_PORT
        │ application listening socket
        ▼
Application process
```

For the lab:

```text
curl 127.0.0.1:8080
          ↓
host loopback TCP 8080
          ↓ Docker -p mapping
container TCP 80
          ↓
Nginx
```

---

## Commands — Detailed Reference

## 49. Publish One Port

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

Example:

```bash
docker run -d -p 8080:80 nginx:alpine
```

## 50. Publish on a Specific Host Address

```bash
docker run -d \
  -p HOST_IP:HOST_PORT:CONTAINER_PORT \
  IMAGE
```

Example:

```bash
docker run -d -p 127.0.0.1:8080:80 nginx:alpine
```

## 51. Specify Protocol

```bash
docker run -p 8080:80/tcp IMAGE
docker run -p 5353:53/udp IMAGE
```

## 52. Publish All Exposed Ports

```bash
docker run -d -P IMAGE
```

Maps image-exposed ports to dynamically selected host ports.

## 53. Show Mappings

```bash
docker port CONTAINER
docker port CONTAINER 80/tcp
```

## 54. Verify Host Socket

```bash
sudo ss -ltnp
sudo ss -ltnp | grep ':8080'
```

## 55. Verify HTTP

```bash
curl http://127.0.0.1:8080
curl -I http://127.0.0.1:8080
curl -v http://127.0.0.1:8080
```

---

## Guided Hands-on Lab — Publish and Prove Nginx

### Scenario

TechCorp needs a local-only Nginx test endpoint on host TCP port 8080. You must prove the application, mapping, host socket, and request path.

### Phase A — Baseline

```bash
docker ps -a --filter name=published-web
sudo ss -ltnp | grep ':8080' || true
```

If port 8080 is already used, identify the owner and choose another lab port rather than disrupting it.

### Phase B — Run Without Publishing

```bash
docker run -d --name unpublished-web nginx:alpine
docker ps --filter name=unpublished-web
docker port unpublished-web
curl -v --max-time 3 http://127.0.0.1:8080
```

The last result depends on whether another host service uses 8080. The key point: `unpublished-web` itself has no host mapping.

Verify Nginx internally:

```bash
docker exec unpublished-web sh -c \
  'wget -qO- http://127.0.0.1:80 | head'
```

### Phase C — Create Published Container

```bash
docker run -d \
  --name published-web \
  -p 127.0.0.1:8080:80 \
  nginx:alpine
```

### Phase D — Verify Every Layer

```bash
docker ps --filter name=published-web
docker port published-web
docker inspect --format '{{json .NetworkSettings.Ports}}' published-web
sudo ss -ltnp | grep ':8080'
curl -v http://127.0.0.1:8080
docker logs --tail 20 published-web
```

The Nginx access log should normally record the request.

### Phase E — Intentional Wrong Port

```bash
curl -v --max-time 3 http://127.0.0.1:8081
```

Explain:

- Container remains running
- Correct mapping remains 8080 → 80
- No mapping was configured on host 8081
- Client failure at 8081 does not prove Nginx failed

### Phase F — Compare Localhost Contexts

Host:

```bash
curl -I http://127.0.0.1:8080
```

Container:

```bash
docker exec published-web sh -c \
  'wget -S -O- http://127.0.0.1:80 2>&1 | head'
```

Explain the different ports and different meanings of loopback.

### Phase G — Automatic Publishing

```bash
docker run -d --name auto-published -P nginx:alpine
docker port auto-published
```

Use the actual selected port in curl.

### Phase H — Second Explicit Mapping

```bash
docker run -d \
  --name published-web-2 \
  -p 127.0.0.1:8081:80 \
  nginx:alpine
```

Verify:

```bash
curl -I http://127.0.0.1:8080
curl -I http://127.0.0.1:8081
```

Prove both containers use internal port 80 but different host ports.

### Phase I — Cleanup

```bash
docker stop unpublished-web published-web auto-published published-web-2
docker rm unpublished-web published-web auto-published published-web-2
```

Verify the names belong to your lab resources first.

### Acceptance Criteria

You can prove:

- Nginx runs internally without host publishing
- Host 8080 maps to container 80
- Loopback-only publishing is visible in Docker output
- Host socket/publishing evidence exists
- Curl receives an HTTP response
- Nginx logs the request
- Host 8081 initially fails for the correct reason
- `-P` selects a dynamic host port
- Two containers can both use internal port 80
- Host and container loopback refer to different environments

---

## Discovery Questions

### Question 1

If `docker ps` shows `80/tcp` but no arrow mapping, can a remote client use host port 80 automatically?

<details>
<summary>Explanation</summary>

No. The value can represent exposed/intended container-port metadata. A host publication is shown with a host address/port mapping such as `127.0.0.1:8080->80/tcp`.

</details>

### Question 2

Why can two containers both listen on container port 80?

<details>
<summary>Explanation</summary>

They normally have separate network environments. Conflict occurs only if they try to publish the same host address, host port, and protocol.

</details>

### Question 3

Does `EXPOSE 80` create a host listener?

<details>
<summary>Explanation</summary>

No. It records intended port metadata. `-p` or `-P` creates runtime publication.

</details>

### Question 4

What does `localhost` mean inside a container?

<details>
<summary>Explanation</summary>

It refers to that container's own loopback network environment, not automatically the Docker host or another container.

</details>

### Question 5

If `curl 127.0.0.1:8081` fails while mapping is `8080:80`, is Nginx proven broken?

<details>
<summary>Explanation</summary>

No. The client tested an unpublished host port. Test the configured mapping and inspect Nginx separately.

</details>

---

## Production Scenario — Accidental Public Exposure

An engineer deploys an internal admin interface:

```bash
docker run -d -p 9000:9000 company-admin:1.0
```

They expected access only from the Docker host. Because the host address was omitted, Docker published on all host addresses by default. The endpoint became reachable from a broader network.

Safer local-only mapping:

```bash
docker run -d \
  -p 127.0.0.1:9000:9000 \
  company-admin:1.0
```

Incident evidence:

```bash
docker port CONTAINER
docker inspect --format '{{json .NetworkSettings.Ports}}' CONTAINER
sudo ss -ltnp | grep ':9000'
sudo iptables -S DOCKER-USER
sudo nft list ruleset
```

The full correction may also require:

- Authentication
- TLS
- Reverse proxy/access control
- Host/cloud firewall rules
- Network segmentation
- Secret rotation if exposure was exploited

Loopback binding reduces reachability but does not replace application security.

---

## Troubleshooting Workflow

## 56. Evidence Order

```text
1. Container exists and runs?
2. Application listens on expected container port?
3. Docker mapping matches expected host/container ports?
4. Host address and socket match intended exposure?
5. Local request succeeds?
6. Remote route/firewall allows intended client?
7. Application returns correct response?
```

## 57. Commands

```bash
docker ps -a --filter name=CONTAINER
docker logs --tail 100 CONTAINER
docker port CONTAINER
docker inspect --format '{{json .NetworkSettings.Ports}}' CONTAINER
sudo ss -ltnp
curl -v http://HOST:PORT
```

## 58. Connection Refused

Usually means the TCP connection reached an endpoint that rejected it because no listener/mapping exists there, though firewall rejection can also produce it.

Check exact address and port.

## 59. Connection Timed Out

Often points toward dropped traffic, unreachable route, firewall filtering, or an application path that does not respond.

Do not treat timeout and refusal as identical.

## 60. HTTP Error Response

An HTTP `404`, `401`, `403`, or `500` proves the TCP/HTTP path reached an HTTP service.

The problem is now likely at application routing, authentication, authorization, or application-error layers—not basic TCP reachability.

---

## Common Mistakes

### 1. Reversing host and container ports

Read `-p` as host first, container second.

### 2. Thinking EXPOSE publishes

It is metadata/documentation.

### 3. Thinking publishing starts a listener inside the container

The application must listen itself.

### 4. Testing the container port on the host

Clients use the published host port.

### 5. Using `localhost` to reach another container

Container loopback refers to the same container.

### 6. Omitting host address unintentionally

This normally exposes the mapping on all host interfaces.

### 7. Assuming UFW alone protects Docker ports

Docker firewall rules require explicit integration and verification.

### 8. Killing a host process to free a port without identification

Inspect ownership and service impact first.

### 9. Using curl for UDP diagnosis

Choose a protocol-appropriate tool.

### 10. Changing ports before proving the listening configuration

Inspect container state, application logs, and mapping first.

---

## Best Practices

- Bind development-only services to loopback
- Publish only ports that clients genuinely need
- Use explicit host addresses, ports, and protocols
- Document host-to-container mappings
- Avoid relying blindly on `latest` image tags in deployments
- Verify application listening behavior before publishing
- Test locally and from the intended remote network
- Protect published services with authentication and TLS where required
- Integrate Docker with host/cloud firewall policy deliberately
- Avoid exposing databases directly unless architecture explicitly requires it
- Use reverse proxies or load balancers for production HTTP entry points
- Monitor and inventory published ports continuously

---

## Interview Questions

### Junior

1. What is a container port?
2. What is a host port?
3. How do you read `-p 8080:80`?
4. What does `docker port` show?
5. What is the difference between `-p` and `-P`?
6. Does `EXPOSE` publish a port?
7. What does `localhost` mean inside a container?
8. Can two containers listen on internal port 80?
9. What protocol does `-p 8080:80` use by default?
10. How do you bind a published port only to host loopback?

### Intermediate

1. Why can a mapping exist while the service is still unreachable?
2. Why might `ss -p` not always show a simple docker-proxy owner?
3. What is the security effect of omitting the host IP?
4. How can Docker port rules interact with UFW?
5. How do you distinguish connection refused, timeout, and HTTP 500?
6. Why can container-loopback binding block bridge traffic?
7. How would you diagnose a host-port conflict?
8. What evidence proves the complete request path?

### Senior Preview

Bridge internals, veth pairs, NAT chains, conntrack, direct routing, gateway modes, userland proxy behavior, and Kubernetes Services are deferred until networking internals and orchestration lessons.

---

## Lesson Reflection

Answer in your own words:

1. Draw the path from host port to application socket.
2. Explain host port versus container port.
3. Why can several containers use port 80 internally?
4. Explain host localhost versus container localhost.
5. Why does EXPOSE not create reachability?
6. Explain `-p` versus `-P`.
7. What security change occurs between `-p 8080:80` and `-p 127.0.0.1:8080:80`?
8. Why can Docker publish a port where no service listens?
9. What does each of `docker port`, `ss`, and `curl` prove?
10. Write your evidence order for a failed published endpoint.

---

## Summary

- Applications listen on container sockets
- Host and container ports belong to different network environments
- `-p HOST:CONTAINER` creates a published mapping
- Host and container port numbers do not need to match
- Several containers can use the same internal port
- Explicit host ports must be unique per host address and protocol
- Omitting host IP normally publishes on all host addresses
- Loopback binding restricts normal access to the host
- `docker port` shows Docker mappings
- `ss` shows host socket/listener evidence
- `curl` tests end-to-end HTTP behavior
- Host localhost and container localhost are different
- `EXPOSE` documents intended ports but does not publish
- `-P` maps exposed ports to dynamically selected host ports
- TCP and UDP mappings are separate

---

## Cheat Sheet

### Mapping syntax

```text
-p HOST_PORT:CONTAINER_PORT
-p HOST_IP:HOST_PORT:CONTAINER_PORT
-p HOST_PORT:CONTAINER_PORT/PROTOCOL
```

### Safe local example

```bash
docker run -d \
  --name published-web \
  -p 127.0.0.1:8080:80 \
  nginx:alpine
```

### Verify

```bash
docker ps --filter name=published-web
docker port published-web
docker port published-web 80/tcp
docker inspect --format '{{json .NetworkSettings.Ports}}' published-web
sudo ss -ltnp | grep ':8080'
curl -v http://127.0.0.1:8080
docker logs --tail 20 published-web
```

### Publish all exposed ports

```bash
docker run -d --name auto-published -P nginx:alpine
docker port auto-published
```

### Protocol

```bash
-p 8080:80/tcp
-p 5353:53/udp
```

### Critical distinctions

```text
Host port ≠ container port
Host localhost ≠ container localhost
EXPOSE ≠ publish
-p ≠ -P
TCP port 53 ≠ UDP port 53
Running container ≠ reachable service
```

---

## Challenge — Find the Wrong Port

### Incident

A junior runs:

```bash
docker run -d \
  --name port-challenge \
  -p 127.0.0.1:9090:80 \
  nginx:alpine
```

Then tests:

```bash
curl http://127.0.0.1:80
```

and reports that Nginx is broken.

### Mission

Without recreating the container initially:

1. Prove its state
2. Read its mapping from `docker ps`
3. Confirm with `docker port`
4. Inspect runtime port data
5. Inspect host socket evidence
6. Test the wrong port and record result
7. Test the correct port
8. Confirm Nginx request logs
9. Explain the root cause
10. State whether any Docker change was required
11. Clean up gracefully

### Rules

- Evidence before change
- No force removal
- Do not alter host firewall
- Do not blame Nginx without an application test
- Explain host and container ports explicitly

### Report Template

```text
Container state:
Configured mapping:
Host bind address:
Host port:
Container port:
Wrong test:
Wrong-test result:
Correct test:
Correct-test result:
Relevant logs:
Root cause:
Correction:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-09/
├── 01-host-port-baseline.txt
├── 02-unpublished-container.txt
├── 03-internal-nginx-test.txt
├── 04-published-port-summary.txt
├── 05-docker-port-output.txt
├── 06-host-socket.txt
├── 07-curl-success.txt
├── 08-wrong-port-test.txt
├── 09-nginx-access-log.txt
├── 10-auto-published-port.txt
└── port-incident-report.md
```

Suggested captures:

```bash
docker port published-web > 05-docker-port-output.txt
sudo ss -ltnp | grep ':8080' > 06-host-socket.txt
curl -v http://127.0.0.1:8080 > 07-curl-success.txt 2>&1
docker logs --tail 20 published-web > 09-nginx-access-log.txt 2>&1
```

Do not expose unrelated host sockets or sensitive URLs in shared lab evidence.

---

## Official References

- [Docker port publishing and mapping](https://docs.docker.com/engine/network/port-publishing/)
- [Docker container run](https://docs.docker.com/reference/cli/docker/container/run/)
- [Dockerfile `EXPOSE`](https://docs.docker.com/reference/dockerfile/#expose)
- [Docker container port](https://docs.docker.com/reference/cli/docker/container/port/)

---

## Next Lesson

**Lesson 10 — Docker Networking Fundamentals**

Port publishing connects clients through the host. Next, you will connect containers directly to one another.

You will learn:

- Default bridge
- User-defined bridge
- Container IP addresses
- Docker's embedded DNS on user-defined networks
- Communication using container names
- Why container IPs should not be hardcoded
- Why `localhost` cannot reach another container
- Host and none network modes

Commands will include:

```bash
docker network ls
docker network inspect
docker network create
docker network connect
docker network disconnect
docker network rm
docker run --network
```

---

## Completion Check

Before moving forward, explain without reading:

```text
Container port versus host port
How to read -p 8080:80
Host binding address
0.0.0.0 versus 127.0.0.1
Host localhost versus container localhost
EXPOSE versus publish
-p versus -P
TCP versus UDP publication
What docker port, ss, and curl each prove
Your wrong-port troubleshooting workflow
```

If you can prove every layer of the Nginx mapping, you are ready for container-to-container networking.
