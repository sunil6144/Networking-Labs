# PAT / NAT Overload – Many Private Hosts, One Public IP (Cisco Packet Tracer)

## Overview

This project demonstrates **PAT (Port Address Translation)**, also called **NAT Overload**, on a Cisco 2911 router in Cisco Packet Tracer — the second lab in the NAT/PAT phase of the networking series.

Where the earlier **Static NAT** lab mapped exactly one private IP to one dedicated public IP, this lab goes further: **three separate internal hosts** (Laptop1, Laptop2, Laptop3) all share a **single public-facing IP** — Router1's own outside interface address — to reach an outside server. PAT distinguishes each host's traffic using a unique **source port number**, which is exactly how most home and office routers provide internet access to many devices using only one public IP from the ISP.

---

## Objectives

- Understand PAT/NAT Overload: many private IPs translating through one public IP, differentiated by port.
- Understand the key difference from Static NAT: Static NAT is 1-to-1 and fixed; PAT is many-to-1 and dynamic.
- Define an access list to identify which internal hosts are eligible for translation.
- Configure PAT using `ip nat inside source list <ACL> interface <outside-interface> overload`.
- Mark the correct interfaces with `ip nat inside` / `ip nat outside`.
- Confirm baseline connectivity from all three hosts before enabling PAT.
- Verify active translations using `show ip nat translations` — specifically seeing multiple inside-local addresses sharing the same inside-global address on different ports.
- Review overall NAT activity using `show ip nat statistics`.

---

## Network Topology

Three laptops share a single LAN behind Router1 (the PAT router), which connects through Router2 to an outside server.

![PAT NAT topology](PAT%20NAT%20topology.png)

| Device | IP Address | Gateway |
|---|---|---|
| Laptop1 | 192.168.10.10/24 | 192.168.10.1 |
| Laptop2 | 192.168.10.11/24 | 192.168.10.1 |
| Laptop3 | 192.168.10.12/24 | 192.168.10.1 |
| Router1 (Gi0/0) | 192.168.10.1 | NAT inside |
| Router1 (Gi0/1) | 10.0.0.1 | NAT outside — this is the shared public-facing address used by all three laptops |
| Router2 (Gi0/1) | 203.0.113.1 | — |
| Server1 | 203.0.113.10/24 | 203.0.113.1 |

---

## Interface Verification

Router1's interfaces were verified as up and correctly addressed before configuring PAT.

![Interface Verification](Interface%20Verification.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | up/up |
| GigabitEthernet0/1 | 10.0.0.1 | up/up |

---

## Initial Connectivity Test (Before PAT)

Before configuring PAT, basic routing was confirmed end-to-end — all three laptops could already reach the outside server via standard routing (PAT only changes the *source address* seen on the outside, not whether the path exists).

![Initial Connectivity Test](Initial%20Connectivity%20Test.png)

Result: 4 of 4 replies received, **0% loss**, TTL = 126 — confirming the two-router path was fully functional before any address translation was introduced.

---

## PAT Configuration

Router1 was configured to translate all hosts matched by access list 1 through its own outside interface, using port-based overload:

```text
access-list 1 permit 192.168.10.0 0.0.0.255

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 10.0.0.1 255.0.0.0
 ip nat outside

ip nat inside source list 1 interface GigabitEthernet0/1 overload
ip route 203.0.113.0 255.255.255.0 10.0.0.2
```

![PAT Configuration](PAT%20Configuration.png)

> **Key difference from Static NAT:** instead of mapping to a separate dedicated public IP, `overload` reuses Router1's own outside interface address (`10.0.0.1`) as the shared public identity for every permitted inside host — this is the standard, realistic way PAT is deployed (identical to how a home router shares one ISP-assigned IP across every device on the network).

---

## NAT Translation Table

`show ip nat translations` captured while all three laptops were actively pinging the server — this is the clearest possible proof of PAT in action.

![NAT Translation Table](NAT%20Translation%20Table.png)

```text
Pro   Inside global   Inside local        Outside local     Outside global
icmp  10.0.0.1:13     192.168.10.11:13    203.0.113.10:13   203.0.113.10:13
icmp  10.0.0.1:14     192.168.10.11:14    203.0.113.10:14   203.0.113.10:14
icmp  10.0.0.1:37     192.168.10.10:37    203.0.113.10:37   203.0.113.10:37
icmp  10.0.0.1:38     192.168.10.10:38    203.0.113.10:38   203.0.113.10:38
icmp  10.0.0.1:5      192.168.10.12:5     203.0.113.10:5    203.0.113.10:5
icmp  10.0.0.1:6      192.168.10.12:6     203.0.113.10:6    203.0.113.10:6
```

All three inside-local addresses — `192.168.10.10`, `.11`, and `.12` — translate through the **same** inside-global address (`10.0.0.1`), each distinguished only by its source port number. This is the defining signature of PAT, and exactly what separates it from Static NAT's fixed 1-to-1 mapping.

---

## NAT Statistics

`show ip nat statistics` summarizes overall PAT activity on Router1.

![NAT Statistics](NAT%20Statistics.png)

| Field | Value |
|---|---|
| Total translations | 12 (0 static, 12 dynamic, 12 extended) |
| Outside interface | GigabitEthernet0/1 |
| Inside interface | GigabitEthernet0/0 |
| Hits | 12 |
| Misses | 12 |
| Expired translations | 0 |

The **0 static** count confirms this router is running pure PAT with no leftover Static NAT entries — all 12 translations were created dynamically from the three laptops' ping traffic.

---

## Post-PAT Connectivity Test

With PAT active, connectivity was re-confirmed — traffic continued to succeed, now translated through the shared outside address rather than passed through untranslated.

![Post-PAT Connectivity Test](Post-PAT%20Connectivity%20Test.png)

Result: 4 of 4 replies received, **0% loss**, TTL = 126, consistent across repeated tests — confirming PAT translation adds no reliability cost over direct routing.

---

## Key Learning Outcomes

- PAT (NAT Overload) lets many internal hosts share one public IP address, distinguished by source port — the same mechanism that lets an entire home network use a single ISP-assigned address.
- `overload` on an `ip nat inside source list ... interface ...` command tells the router to reuse that interface's own IP as the shared public address, rather than requiring a separate address pool.
- The clearest proof of PAT (vs. Static NAT) is a translation table showing **multiple different Inside local addresses mapped to the same Inside global address**, split apart only by port number.
- `show ip nat statistics` with `0 static` and all `dynamic`/`extended` entries confirms a pure PAT setup with no static mappings mixed in.
- PAT is the real-world default for outbound internet access in almost every small network — Static NAT is reserved for the few hosts (like public-facing servers) that need a fixed, dedicated public identity.
