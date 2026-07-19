# DevOps Networking — Professional Roadmap

## The target

This roadmap is designed to make you a DevOps engineer who can:

- explain how a request travels from a user to an application and back;
- design sensible local, container, Kubernetes, and cloud networks;
- distinguish DNS, routing, firewall, TLS, load-balancer, and application failures;
- troubleshoot with evidence instead of trying random commands;
- automate common network infrastructure with Infrastructure as Code;
- communicate clearly with network and security engineers.

It does **not** try to turn you into a Cisco/ISP network engineer. Deep router configuration, carrier networking, and vendor-specific certification material are deliberately deferred.

## Depth guide

- **Master:** calculate, configure, test, break, and troubleshoot it yourself.
- **Working knowledge:** explain it, use it in normal DevOps work, and troubleshoot common failures.
- **Awareness:** understand why it exists and when a specialist is needed.

## Recommended pace

- **Normal pace:** 14–16 weeks, 60–90 minutes a day, 5 days a week.
- **Fast pace:** 8–10 weeks, about 2 hours a day.
- Spend roughly **40% learning and 60% doing labs**.
- Use one simple application throughout: `client → reverse proxy → backend → database`.

---

## Stage 00 — Build the lab environment

**Depth: Master | Time: 2–3 days**

### Learn

- What a network lab is and how to change it safely.
- The difference between your host, a VM, a container, a Kubernetes Pod, and a cloud VM.
- How to record a network diagram and a table of IP addresses, ports, and routes.

### Install

- Linux or a Linux VM
- Docker and Docker Compose
- Kind and `kubectl`
- NGINX or HAProxy
- Wireshark; use `tcpdump` when working in a terminal
- Terraform

### First inspection commands

```bash
ip link
ip address
ip route
cat /etc/resolv.conf
ss -lntup
ping -c 4 1.1.1.1
dig example.com
curl -v https://example.com
```

### Exit check

You can identify your interface, IP address, default route, DNS resolver, listening ports, and the process serving a port.

---

## Stage 01 — How networks move data

**Depth: Master | Time: 1 week**

### Learn in this order

1. Client, server, peer, request, response, and connection.
2. NIC, switch, router, firewall, proxy, load balancer, and gateway.
3. LAN, WAN, internet, intranet, physical network, and virtual network.
4. Frame, packet, TCP segment, UDP datagram, and application data.
5. Encapsulation and decapsulation.
6. TCP/IP model and OSI model, with practical focus on:
   - Layer 2: MAC and Ethernet
   - Layer 3: IP and routing
   - Layer 4: TCP, UDP, and ports
   - Layer 7: DNS, HTTP, SSH, and other application protocols
7. Unicast, broadcast, multicast, and anycast at a conceptual level.
8. Bandwidth, throughput, latency, round-trip time, jitter, packet loss, and availability.

### Lab

Trace one web request and label every known device, protocol, address, and port in the path.

### Exit check

Given `curl https://app.example.com`, you can describe the journey without saying that DNS, a port, an IP address, and a URL are the same thing.

---

## Stage 02 — Ethernet, IP addressing, and subnetting

**Depth: Master | Time: 2 weeks**

### Local delivery

- MAC addresses and network interfaces
- Switch versus router
- ARP for IPv4 and Neighbor Discovery at a basic IPv6 level
- Default gateway
- Ethernet broadcast domain
- VLAN and trunk concepts — working knowledge only
- MTU, MSS, fragmentation, and Path MTU Discovery

### IPv4

- 32-bit IPv4 structure and dotted-decimal notation
- Network part and host part
- Public and private address ranges
- Loopback, link-local, unspecified, and broadcast addresses
- Static versus dynamically assigned addresses
- CIDR notation and prefix length
- Network address, first host, last host, and broadcast address
- Subnet masks
- Subnetting and VLSM
- Address planning and CIDR overlap

### IPv6

- Why IPv6 exists
- 128-bit addresses and shortened notation
- Prefixes and `/64`
- Global unicast, link-local, loopback, and multicast
- Neighbor Discovery and SLAAC at a working level
- Dual stack and IPv6-only awareness
- The important difference: IPv6 should not be treated as “IPv4 with bigger addresses”

### Required practice

Calculate these without a subnet calculator first, and then verify with one:

- `/24`, `/25`, `/26`, `/27`, `/28`, and `/30`
- number of addresses and normally usable hosts;
- whether two addresses belong to the same subnet;
- whether two planned cloud or Kubernetes CIDRs overlap.

### Lab

Create an address plan for development, staging, and production. Give each environment non-overlapping application, database, Pod, and Service CIDRs.

### Exit check

You can explain exactly what `10.20.4.0/24` means and divide it into smaller networks for a real design.

---

## Stage 03 — Routing, forwarding, and NAT

**Depth: Master | Time: 1–1.5 weeks**

### Routing

- Routing table: destination, prefix, next hop, interface, and metric
- Directly connected, local, static, and default routes
- Longest-prefix match
- Default route `0.0.0.0/0` and IPv6 default `::/0`
- Packet forwarding and TTL/hop limit
- Symmetric versus asymmetric paths
- Route propagation and route conflicts
- Policy routing awareness

### NAT and connection state

- Why NAT exists and what it does not do
- Source NAT, destination NAT, PAT, and masquerading
- Port forwarding
- Five-tuple: source IP, source port, destination IP, destination port, protocol
- Stateful connection tracking
- Ephemeral ports
- NAT/SNAT port exhaustion

### Network connection patterns

- Public, private, and isolated subnets
- Network peering
- Hub-and-spoke and transit routing
- Point-to-site and site-to-site VPN concepts
- Private dedicated links
- BGP: ASN, prefix advertisement, and route exchange — working knowledge
- OSPF and other dynamic-routing protocols — awareness only

### Lab

Use Linux network namespaces and `veth` pairs to build `client → router → server`. Add routes, enable forwarding, add NAT, deliberately break each part, and repair it.

### Exit check

For any source and destination IP, you can inspect the route table and predict the next hop.

---

## Stage 04 — Transport and core application protocols

**Depth: Master | Time: 2 weeks**

### Ports and sockets

- Port numbers, listening sockets, established connections, and processes
- Well-known, registered, dynamic, and ephemeral ports
- Binding to `127.0.0.1`, a specific interface, or `0.0.0.0`
- “Connection refused” versus timeout

### TCP

- Three-way handshake
- Sequence numbers and acknowledgements
- Reliability, retransmission, ordering, and duplicate handling
- Receive window, flow control, and congestion control at a practical level
- Connection teardown, FIN, RST, and common TCP states
- `TIME_WAIT`, keepalive, and connection reuse

### UDP and ICMP

- UDP characteristics and common use cases
- When UDP is preferable to TCP
- ICMP error and diagnostic messages
- What `ping` proves and what it does not prove
- How traceroute discovers a path

### DNS

- Stub resolver, recursive resolver, root, TLD, and authoritative server
- Forward and reverse lookup
- Records: `A`, `AAAA`, `CNAME`, `NS`, `MX`, `TXT`, `PTR`, `SRV`, and `CAA`
- Recursive versus iterative queries
- TTL, positive cache, and negative cache
- Delegation and zones
- Private DNS and split-horizon DNS
- Search domains and `ndots`
- DNS over UDP and TCP
- DNSSEC awareness

### DHCP and time

- DHCP discover, offer, request, and acknowledge
- Lease, address, gateway, DNS, and other supplied options
- NTP and why correct time matters to logs, distributed systems, and certificates

### HTTP and HTTPS

- URL components
- Request line, methods, headers, and body
- Response status, headers, and body
- Common status codes and redirect behavior
- Idempotency, cookies, sessions, and caching basics
- Persistent connections
- HTTP/1.1, HTTP/2 multiplexing, and HTTP/3 over QUIC at the correct DevOps depth
- TLS handshake and encryption purpose
- Certificate, public/private key, CA, certificate chain, SAN, and hostname validation
- SNI and ALPN
- TLS termination, re-encryption, and passthrough
- Mutual TLS awareness

### Other useful protocols

- SSH, keys, host verification, port forwarding, and jump hosts
- SMTP, IMAP, SNMP, and FTP/SFTP: recognize their roles and common ports; do not study deeply yet

### Lab

For one HTTPS request, prove DNS resolution, TCP establishment, TLS certificate validation, HTTP request, and HTTP response separately.

### Exit check

You can distinguish `NXDOMAIN`, DNS timeout, TCP timeout, connection refused, TLS certificate error, and HTTP `502`.

---

## Stage 05 — Linux networking and evidence-first troubleshooting

**Depth: Master | Time: 1.5–2 weeks**

### Linux concepts

- Interfaces, addresses, routes, neighbors, and sockets
- `/etc/hosts`, `/etc/resolv.conf`, and the system resolver
- NetworkManager and `systemd-resolved` at a working level
- Network namespaces, bridges, and `veth` pairs
- TUN/TAP awareness
- IP forwarding and relevant `sysctl` settings
- Netfilter, nftables, iptables compatibility, chains, rules, filtering, and NAT
- Host firewall basics

### Commands to master

| Question | Main tools |
| --- | --- |
| What interfaces and addresses exist? | `ip link`, `ip address` |
| Which next hop will Linux choose? | `ip route`, `ip route get` |
| What MAC/neighbor information is known? | `ip neigh` |
| Which process owns a port? | `ss`, `lsof` |
| Can an IP answer? | `ping` |
| Where does the path fail? | `tracepath`, `traceroute`, `mtr` |
| Does DNS work correctly? | `dig`, `resolvectl` |
| Can I open this TCP/UDP port? | `nc` |
| What exactly happened in HTTP? | `curl -v` |
| Is the TLS certificate/handshake valid? | `openssl s_client` |
| What packets crossed the interface? | `tcpdump`, Wireshark |
| What filtering/NAT rules apply? | `nft`, and `iptables` where still used |
| What are the NIC/link characteristics? | `ethtool` |

### The troubleshooting order

1. Define source, destination, protocol, port, and expected behavior.
2. Reproduce the failure and record the exact error.
3. Check the application process and listening socket.
4. Check DNS, but also test the destination IP directly when meaningful.
5. Check source address, route, next hop, and return route.
6. Check TCP/UDP connectivity.
7. Check firewall, security policy, NAT, and load-balancer health.
8. Check TLS and then HTTP/application behavior.
9. Capture packets when the earlier evidence is insufficient.
10. Fix one cause, retest, and preserve evidence.

### Lab incidents

- Service bound only to localhost
- Wrong subnet mask
- Missing default route
- Broken DNS server or record
- Closed port versus dropped packets
- Expired or hostname-mismatched certificate
- MTU black-hole symptom
- Wrong NAT or forwarding rule

### Exit check

You can explain what each command proves, what it does not prove, and which command should come next.

---

## Stage 06 — Proxies, load balancing, and reliable application delivery

**Depth: Master | Time: 1 week**

### Learn

- Forward proxy versus reverse proxy
- Layer 4 versus Layer 7 load balancing
- Public versus internal load balancer
- Algorithms: round robin, weighted, least connections, hash-based
- Active and passive health checks
- Readiness versus liveness from a traffic perspective
- TLS termination, passthrough, and re-encryption
- Host-based, path-based, header-based, and TCP routing
- Session affinity and why stateless applications are preferable
- Connection draining and graceful shutdown
- Connect, read, write, idle, and request timeouts
- Retries, exponential backoff, jitter, circuit breaking, and retry storms
- Rate limiting
- CDN, edge caching, origin, cache control, and cache invalidation basics
- DNS-based traffic steering and failover
- API gateway versus reverse proxy versus load balancer
- Service discovery

### Lab

Run two backend instances behind NGINX or HAProxy. Add health checks, TLS termination, path routing, and graceful removal of one backend.

### Exit check

You can explain a `502`, `503`, and `504` from the proxy’s point of view and collect evidence from both proxy and backend.

---

## Stage 07 — Docker and container networking

**Depth: Master | Time: 1 week**

Docker’s official networking documentation describes bridge, host, overlay, IPvlan, Macvlan, and none drivers. For normal DevOps work, master bridge networking and port publishing; keep the others at working or awareness depth.

### Learn

- Container network namespace
- `veth` pair and Linux bridge mental model
- Default bridge versus user-defined bridge
- Container IP addresses and embedded DNS
- Service-name communication in Docker Compose
- Port publishing: host port versus container port
- DNAT/SNAT involved in published traffic
- `EXPOSE` versus actually publishing a port
- Host and none modes
- Overlay networking concept
- IPvlan and Macvlan awareness
- Container-to-container, container-to-host, host-to-container, and internet paths
- How Docker interacts with host firewall rules

### Lab

Create frontend, backend, and database containers. Publish only the frontend, place components on intentional networks, prove DNS by service name, inspect the namespaces and routes, and troubleshoot a broken connection.

### Exit check

You can draw and prove the full path from `localhost:8080` to a container process.

---

## Stage 08 — Kubernetes networking

**Depth: Master | Time: 2 weeks**

Kubernetes relies on a cluster network implementation, commonly provided through CNI plugins. Its Service layer gives stable discovery and traffic distribution over changing Pods. Current Kubernetes documentation recommends Gateway API for new capabilities while Ingress remains common and its API is frozen, so a DevOps engineer should learn both.

### Cluster and Pod networking

- Node IP, Pod IP, Service IP, and external IP
- Node, Pod, and Service CIDRs; preventing overlap
- Kubernetes network model
- Pod-to-Pod traffic on the same node and across nodes
- Container Network Interface (CNI) role
- Overlay versus routed Pod networks
- Encapsulation awareness: VXLAN and similar technologies
- eBPF data-plane awareness

### Services and discovery

- Why Pod IPs are not stable application endpoints
- Labels, selectors, EndpointSlices, and ready endpoints
- `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`, and headless Services
- `port`, `targetPort`, and `nodePort`
- CoreDNS and Service DNS names
- Service data plane and kube-proxy concepts
- `externalTrafficPolicy` and source-IP awareness

### North-south traffic

- Ingress resource versus Ingress controller
- Host and path routing
- Gateway API: GatewayClass, Gateway, HTTPRoute, attachment, and controller
- TLS secrets and certificate automation awareness

### East-west controls

- NetworkPolicy selectors
- Ingress and egress rules
- Default deny and explicit allow
- The requirement for a CNI implementation that enforces NetworkPolicy
- Egress control and external destinations

### Troubleshooting

- Pod → Pod
- Pod → Service
- Service → ready EndpointSlice
- Pod → DNS
- External client → load balancer/controller → Service → Pod
- Service mesh and multi-cluster networking — awareness only

### Lab

On Kind, deploy a frontend and backend, expose them with Services, prove CoreDNS, add an Ingress or Gateway API implementation, apply default-deny NetworkPolicies, allow only required flows, then repair deliberately broken DNS, selector, port, and policy failures.

### Exit check

You can inspect every hop from an external request to the selected Pod and identify the failing object or layer.

---

## Stage 09 — Cloud networking

**Depth: Master for one cloud; working knowledge across the others | Time: 2 weeks**

Learn the common model first, then map it to AWS, Azure, or Google Cloud. Exact scope and defaults differ across providers, so do not assume that identically named resources behave identically.

### Core virtual-network design

- VPC/VNet and provider scope
- CIDR planning
- Subnets, regions, and availability zones
- Virtual NICs and private/public IP addresses
- System, local, default, and custom routes
- Public, private, and isolated subnet design
- Internet gateway or equivalent internet path
- Managed NAT for controlled outbound access
- Bastion and private administration patterns

### Traffic control and delivery

- Stateful security groups/NSGs/firewall rules
- Subnet ACL concepts and stateful versus stateless behavior
- Inbound versus outbound rules
- Cloud Layer 4 and Layer 7 load balancers
- Public versus internal load balancers
- Target groups/backend services, listeners, rules, and health checks
- Managed DNS and private DNS zones
- Private endpoints/private service access
- CDN, WAF, and DDoS protection awareness; deeper treatment belongs in the security roadmap

### Connectivity

- VPC/VNet peering and non-transitive routing
- Transit gateway, hub-and-spoke, and shared-network designs
- Client VPN and site-to-site VPN
- Dedicated private connectivity
- Hybrid DNS and route propagation
- BGP’s role in VPN and dedicated connectivity
- Multi-account/subscription/project networking
- Cross-region and multi-region patterns

### Operations

- Flow logs
- Load-balancer access logs
- DNS query logs
- Reachability/path analysis tools
- NAT connection/port capacity
- IPv6 and dual-stack planning
- Data-transfer and NAT cost awareness

### Cloud-neutral mapping

| Concept | AWS | Azure | Google Cloud |
| --- | --- | --- | --- |
| Private virtual network | VPC | Virtual Network (VNet) | VPC network |
| Instance traffic control | Security group | Network Security Group | VPC firewall rule/policy |
| Managed outbound NAT | NAT Gateway | NAT Gateway | Cloud NAT |
| Private managed-service access | PrivateLink/VPC endpoints | Private Link/private endpoint | Private Service Connect |
| Hub routing | Transit Gateway | Virtual WAN/Virtual Network Manager patterns | Network Connectivity Center |
| Dedicated hybrid link | Direct Connect | ExpressRoute | Cloud Interconnect |

### Design lab

Design a two-availability-zone production network containing:

- public load balancer;
- private application nodes;
- isolated database tier;
- controlled outbound access;
- private access to managed services;
- least-privilege traffic rules;
- DNS, flow logs, and failure paths.

Implement it with Terraform only when you have a safe sandbox. A diagram and reviewed Terraform configuration are sufficient for learning without creating paid resources.

### Exit check

You can justify every subnet, route, public IP, NAT path, load balancer, and traffic rule in the design.

---

## Stage 10 — Networking as Code, observability, and reliability

**Depth: Master | Time: 1–1.5 weeks**

### Infrastructure as Code

- Model networks with Terraform resources, variables, outputs, and modules
- Dependencies and the resource graph
- Separate environments without copying uncontrolled configuration
- IP address management (IPAM) concepts
- Naming, tagging, ownership, and network inventory
- Formatting, validation, plan review, policy checks, and drift detection
- Safe changes, blast radius, rollback thinking, and peer review
- Automated connectivity tests after deployment

### Observability

- Interface, host, firewall, NAT, DNS, proxy, load-balancer, and application evidence
- Throughput, latency, jitter, packet loss, errors, retransmissions, drops, and saturation
- Active connections, connection rate, conntrack use, ephemeral/SNAT ports, and queue depth
- DNS latency and failure rate
- TCP connect, TLS handshake, and HTTP timing
- Target health and endpoint readiness
- Flow logs versus packet captures versus application logs
- Synthetic probes and black-box monitoring

### Reliability patterns

- Redundant paths and eliminating single points of failure
- Multi-zone design
- Sensible health checks
- Timeouts before retries
- Exponential backoff with jitter
- Connection pooling and keepalive
- Graceful shutdown and connection draining
- DNS TTL and failover trade-offs
- Capacity planning for load balancers, NAT, ports, and connection tracking
- Understanding that the network may be delayed, lossy, duplicated, reordered, or partitioned

### Lab

Create a small network module, validate changes in CI, expose useful outputs, run connectivity tests, collect metrics/logs, and write a short incident runbook.

### Exit check

You can review a network change before deployment and define the evidence that proves it worked afterward.

---

## Stage 11 — Capstone incidents

**Depth: Master | Time: 1–2 weeks**

### Capstone 1 — Linux routed network

Build a client, router, and server from network namespaces. Configure addressing, routes, forwarding, NAT, DNS, and HTTP. Repair at least five injected failures.

### Capstone 2 — Docker three-tier application

Deploy reverse proxy, frontend/backend, and database components on intentional Docker networks. Publish only the required entry point. Prove every flow with commands and a packet capture.

### Capstone 3 — Kubernetes application on Kind

Deploy frontend and backend workloads, ClusterIP Services, CoreDNS discovery, Gateway API or Ingress, TLS, and NetworkPolicies. Troubleshoot broken selectors, target ports, DNS, policies, and controller routing.

### Capstone 4 — Cloud production design

Produce a diagram and Terraform for a two-zone network with public entry, private compute, isolated data, controlled egress, private service access, flow logs, and a failure/recovery plan. It can remain plan-only to avoid cost.

### Required incident report format

1. User-visible symptom
2. Expected path
3. Evidence collected
4. Failed layer or component
5. Root cause
6. Fix
7. Verification
8. Prevention

---

## What to master, understand, and defer

### Must master now

- IPv4, CIDR, and subnetting
- Routing tables and longest-prefix match
- NAT, connection tracking, ports, and sockets
- TCP, UDP, ICMP, DNS, HTTP, and TLS
- Linux inspection and troubleshooting commands
- Reverse proxies and Layer 4/Layer 7 load balancing
- Docker networking and published ports
- Kubernetes Services, DNS, Ingress/Gateway, and NetworkPolicy
- One cloud provider’s VPC/VNet networking
- Terraform-based network changes
- Logs, metrics, flow records, and packet capture

### Working knowledge is enough initially

- IPv6
- VLANs
- VPNs and hybrid connectivity
- BGP concepts
- CDN and DNS traffic steering
- Overlay networks and VXLAN
- eBPF networking
- Service meshes and multi-cluster networking

### Defer until a job specifically needs them

- Cisco/Juniper command-line configuration
- Deep STP, LACP, switch trunk, and campus design
- Deep OSPF/BGP policy and route-reflector design
- MPLS, EVPN, and carrier networks
- Wireless RF design
- Advanced QoS and traffic engineering
- Kernel-level TCP tuning without evidence

---

## Professional mastery checklist

You are ready when you can complete all of these without guessing:

- [ ] Calculate and design non-overlapping CIDR ranges.
- [ ] Predict a packet’s next hop from a routing table.
- [ ] Explain ARP, NAT, TCP handshake, DNS resolution, TLS handshake, and HTTP exchange.
- [ ] Find which process owns a port and why a client cannot connect.
- [ ] Separate a DNS failure from a route, firewall, TLS, proxy, or application failure.
- [ ] Use `tcpdump` or Wireshark to prove whether packets and replies exist.
- [ ] Configure and troubleshoot a reverse proxy and load balancer.
- [ ] Explain Docker bridge networking and published-port traffic.
- [ ] Trace Kubernetes traffic from client to controller, Service, EndpointSlice, and Pod.
- [ ] Create and troubleshoot a basic Kubernetes NetworkPolicy.
- [ ] Design a two-zone cloud network with safe ingress and egress.
- [ ] Explain stateful security rules versus stateless ACL rules.
- [ ] Represent the cloud network with reviewed Infrastructure as Code.
- [ ] Use metrics, logs, flow records, and packet captures appropriately.
- [ ] Write a clear root-cause report and verification steps.

---

## The correct study sequence

Do not jump directly from subnetting to Kubernetes. Follow this dependency order:

```mermaid
flowchart TD
    A["Packets and layers"] --> B["IP and subnetting"]
    B --> C["Routing and NAT"]
    C --> D["DNS, TCP, HTTP, and TLS"]
    D --> E["Linux troubleshooting"]
    E --> F["Proxies and load balancers"]
    F --> G["Docker netw
