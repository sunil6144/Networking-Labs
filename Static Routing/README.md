# Static Routing – Cisco Packet Tracer

## Overview

This project demonstrates **static routing** between two separate LANs connected through two Cisco 2911 routers in Cisco Packet Tracer.

Router0 and Router1 are connected directly over a point-to-point WAN link, each serving as the gateway for its own local network. Since no dynamic routing protocol was used, a **static route** was manually configured on each router, telling it how to reach the remote LAN through the other router's WAN interface.

This lab builds on the earlier VLAN and Inter-VLAN Routing labs, shifting focus from Layer 2 segmentation to Layer 3 routing between fully separate networks.

---

## Objectives

- Understand the concept of static routing and when it is used instead of a dynamic routing protocol.
- Configure IP addresses on router LAN and WAN interfaces.
- Configure static routes on both routers pointing to each other's remote LAN.
- Verify interface status using `show ip interface brief`.
- Verify static route entries in the routing table using `show ip route`.
- Understand the static route notation `S 192.168.x.0/24 [1/0] via <next-hop>`.
- Test and confirm end-to-end connectivity across both routers.
- Relate the TTL value of a successful ping to the number of router hops traversed.

---

## Network Topology

Two independent LANs, each behind its own router, are connected via a direct WAN link between Router0 and Router1.

![Static Routing Topology](Static%20Routing%20Topology.png)

| Device | IP Address | Connected To |
|---|---|---|
| Laptop1 | 192.168.10.10 | Switch0 (LAN 1) |
| Laptop2 | 192.168.10.20 | Switch0 (LAN 1) |
| Laptop3 | 192.168.20.10 | Switch1 (LAN 2) |
| Laptop4 | 192.168.20.20 | Switch1 (LAN 2) |
| Router0 (Gi0/0) | 192.168.10.1 | LAN 1 gateway |
| Router0 (Gi0/1) | 10.0.0.1 | WAN link to Router1 |
| Router1 (Gi0/0) | 10.0.0.2 | WAN link to Router0 |
| Router1 (Gi0/1) | 192.168.20.1 | LAN 2 gateway |

---

## Router0 Interface Verification

Router0's interfaces were verified — Gi0/0 facing LAN 1, Gi0/1 facing the WAN link to Router1.

![R0 Interface Verification](R0%20Interface%20Verification%20.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | up/up |
| GigabitEthernet0/1 | 10.0.0.1 | up/up |

---

## Router1 Interface Verification

Router1's interfaces were verified — Gi0/0 facing the WAN link to Router0, Gi0/1 facing LAN 2.

![R1 Interface Verification](R1%20Interface%20Verification.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 10.0.0.2 | up/up |
| GigabitEthernet0/1 | 192.168.20.1 | up/up |

---

## Router0 Routing Table

Router0's routing table shows its own connected networks plus a static route to reach the remote LAN (192.168.20.0/24) via Router1's WAN interface.

![R0 Routing Table](R0%20Routing%20Table.png)

```text
C    10.0.0.0/30 is directly connected, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
S    192.168.20.0/24 [1/0] via 10.0.0.2
```

---

## Router1 Routing Table

Router1's routing table mirrors this — its own connected networks plus a static route back to LAN 1 (192.168.10.0/24) via Router0's WAN interface.

![R1 Routing Table](R1%20Routing%20Table.png)

```text
C    10.0.0.0/30 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
S    192.168.10.0/24 [1/0] via 10.0.0.1
```

---

## End-to-End Connectivity Test

A host on LAN 1 successfully pinged Laptop3 (`192.168.20.10`) on LAN 2. All 4 packets were delivered with **0% loss**, and the reply came back with **TTL = 126** — two less than the default starting TTL of 128, confirming the packet passed through **two router hops** (Router0 → Router1) using the statically configured routes.

![End to End Connectivity](End%20to%20End%20Connectivity.png)

---

## Key Learning Outcomes

- Static routes must be configured manually on every router in the path, and in both directions, for two-way communication to succeed.
- The `S` code in `show ip route` identifies a statically configured route, with administrative distance and metric shown as `[1/0]`.
- Each router only needs to know how to reach networks it is not directly connected to — via the next-hop IP of the neighboring router.
- TTL decreases by exactly 1 for every router hop a packet passes through, making it a reliable way to count hops from a successful ping.
- Static routing works well for small, predictable topologies but does not scale or adapt automatically like dynamic routing protocols (e.g., OSPF, EIGRP).
