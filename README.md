# Enterprise VPN-MPLS network with a redundant LAN, AAA and monitoring

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=flat-square&logo=cisco&logoColor=white)
![MPLS](https://img.shields.io/badge/MPLS-L3VPN-005073?style=flat-square)
![GNS3](https://img.shields.io/badge/GNS3-simulation-00B5B8?style=flat-square)
![Routing](https://img.shields.io/badge/OSPF%20%C2%B7%20LDP%20%C2%B7%20MP--BGP-F46800?style=flat-square)

Design and deployment of a complete service-provider **VPN-MPLS** network that interconnects
several enterprise sites with isolated routing per customer, a redundant campus LAN, and the
administration and monitoring that keep it secure and observable.

> Engineering project, TEK-UP University. Built and validated in **GNS3 / VMware** with Cisco
> IOS. With **Zied Mellouli**, supervised by Mr. Tarek Hdiji.

---

## Architecture

```mermaid
flowchart LR
    subgraph C1["VPN Customer 1"]
        CE11["CE11"]
        CE12["CE12"]
    end
    subgraph C2["VPN Customer 2"]
        CE21["CE21"]
        CE22["CE22"]
    end
    subgraph MPLS["MPLS backbone (OSPF + LDP, AS 100)"]
        PE1["PE1 (Provider Edge)<br/>VRFs + MP-BGP"]
        P1["P1 (core)"]
        P2["P2 (core)"]
        PE2["PE2 (Provider Edge)<br/>VRFs + MP-BGP"]
    end
    CE11 --- PE1
    CE21 --- PE1
    CE12 --- PE2
    CE22 --- PE2
    PE1 --- P1
    PE1 --- P2
    P1 --- P2
    P2 --- PE2
    P1 --- PE2
    PE1 == MP-BGP VPNv4 == PE2
```

**Backbone:** 2 core routers (P) and 2 edge routers (PE). **Customers:** 4 sites in 2 isolated
VPNs. Traffic for Customer 1 can never reach Customer 2, and vice versa.

## 1. MPLS L3VPN backbone

| Element | Choice | Why |
|---|---|---|
| Core routing | **OSPF** (area 0 + per-site areas) | Fast convergence, smaller routing tables, traffic-engineering support |
| Label distribution | **LDP** | Builds the label-switched paths across the backbone |
| Customer isolation | **VRF** (Virtual Routing and Forwarding) | A separate virtual routing table per customer on the shared PE routers |
| VPN route exchange | **MP-BGP** (VPNv4) | Carries each customer's routes between PE routers with route distinguishers / route targets |

Isolation was proven: CE11 reaches CE12 (same VPN) but not CE21 or CE22 (other VPN).

## 2. Redundant campus LAN

A three-tier design (core / distribution / access) built for high availability:

- **HSRP** - a virtual gateway shared between two distribution switches; the standby takes over in under a second if the active one fails.
- **EtherChannel** - link aggregation between the distribution switches for bandwidth and resilience.
- **VLAN segmentation** - separate DATA and Management VLANs, trunked with 802.1Q and inter-VLAN routing.
- **DHCP** served from the Layer-3 switches, with reserved address ranges.
- Failover tested by shutting a VLAN on the primary switch and confirming the standby took over.

## 3. Administration, AAA and monitoring (chapter 3)

- **AAA with RADIUS**, falling back to local authentication if the RADIUS server is unreachable - authentication, authorisation and session accounting for every device.
- **VTY access-control lists** restricting remote management to the administrators' IP addresses only.
- Local console / VTY protection with privilege levels on every router and switch.
- **SNMP, Syslog, NTP and IP SLA** for centralised monitoring, time synchronisation and reachability checks.

## What this project shows

- Service-provider networking end to end: MPLS L3VPN, VRF isolation, OSPF, LDP and MP-BGP.
- High-availability campus design with HSRP, EtherChannel and VLAN segmentation.
- Network security and operations: AAA/RADIUS, management-plane hardening (ACLs, privilege levels) and monitoring.

> Configuration is described at the design level. Device credentials from the simulation are
> deliberately kept out of this README.

---

**Rayen Gader** - Cyber Security Engineer, L2 SOC analyst ·
[Portfolio](https://rayengader.github.io/) · [LinkedIn](https://www.linkedin.com/in/rayengader/) · rayengader9@gmail.com
