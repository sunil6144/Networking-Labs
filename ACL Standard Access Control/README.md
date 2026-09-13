# Standard ACL – Block a Single Host (Cisco Packet Tracer)

## Overview

This project demonstrates a **Standard Access Control List (ACL)** on a Cisco 2911 router in Cisco Packet Tracer — the first hands-on step from pure routing into **network security / traffic filtering**.

Where routing decides *where* a packet goes, an ACL decides *whether it's allowed to go there at all*. In this lab, a specific host (`PC1 – 192.168.10.10`) was explicitly denied access to the remote network (`192.168.20.0/24`), while all other traffic through the router continued to be permitted.

This lab marks the beginning of the security-focused phase of the networking series, building directly on the earlier Static and OSPF routing labs.

---

## Objectives

- Understand the difference between routing ("can the packet reach its destination?") and ACLs ("is the packet allowed to reach its destination?").
- Configure a Standard ACL that filters traffic based on **source IP address only**.
- Understand and correctly order `deny` and `permit` statements, including the ACL's implicit `deny any` at the end.
- Apply the ACL to the correct router interface, in the correct direction (`in` vs `out`).
- Verify router interface configuration using `show ip interface brief` and `show ip interface`.
- Confirm connectivity **before** applying the ACL (baseline proof).
- Confirm the same connectivity **fails** after the ACL is applied (enforcement proof).
- Use `show access-list` to view real-time match counters as proof the ACL is actively filtering traffic.

---

## Network Topology

A single router (R1) connects two LANs, each with one PC.

![ACL Topology](ACL%20Topology.png)

| Device | Interface | IP Address |
|---|---|---|
| PC1 | NIC | 192.168.10.10/24 |
| R1 | Gi0/0 | 192.168.10.1/24 |
| R1 | Gi0/1 | 192.168.20.1/24 |
| PC2 | NIC | 192.168.20.10/24 |

---

## R1 Interface Verification

Router interfaces were verified as up and correctly addressed before any ACL was applied.

![R1 Interface Verification](R1%20Interface%20Verification.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | up/up |
| GigabitEthernet0/1 | 192.168.20.1 | up/up |

---

## Initial Connectivity Test (Before ACL)

Before configuring any ACL, PC1 successfully pinged PC2, confirming routing between the two networks was working correctly. This baseline is essential — an ACL should only be blamed for a failure once "before" connectivity is proven.

![Initial Connectivity Test](Initial%20Connectivity%20Test.png)

Result: 3 of 4 replies received (first packet lost to normal ARP delay), TTL = 127 — one router hop, as expected.

---

## ACL Configuration

The following Standard ACL was configured on R1 to deny PC1 specifically while permitting all other traffic:

```text
access-list 10 deny host 192.168.10.10
access-list 10 permit any
```

It was then applied **inbound** on Gi0/0 — the interface facing PC1's subnet — so the deny takes effect as close to the traffic source as possible:

```text
interface GigabitEthernet0/0
ip access-group 10 in
```

Verified using `show ip interface gigabitEthernet 0/0`:

![ACL Configuration](ACL%20Configuration.png)

Key line confirming the ACL is active on the interface:

```text
Inbound access list is 10
```

> **Why `permit any` matters:** every ACL ends with an implicit `deny any` that cannot be seen in the configuration. Without an explicit `permit any` as the second line, *all* traffic through Gi0/0 — not just PC1's — would have been silently blocked.

---

## Blocked Connectivity Test (After ACL)

With the ACL active, PC1 attempted the exact same ping to PC2 again.

![Blocked Connectivity Test](Blocked%20Connectivity%20Test.png)

Result: **100% packet loss** — all 4 requests returned `Destination host unreachable` from `192.168.10.1` (R1's own interface), confirming the router itself is rejecting the traffic at the ACL, rather than the packet failing to route.

---

## ACL Hit Count — Proof of Enforcement

`show access-list` was used to confirm the ACL is actively matching and blocking traffic, not just present in the configuration.

![ACL Hit Count](ACL%20Hit%20Count.png)

```text
Standard IP access list 10
    10 deny host 192.168.10.10 (4 match(es))
    20 permit any
```

The **4 match(es)** directly correspond to the 4 ping packets sent during the blocked connectivity test — clear, measurable evidence the ACL is enforcing the policy in real time.

---

## Key Learning Outcomes

- Standard ACLs filter traffic based on **source IP address only** — they cannot match destination, protocol, or port (that requires an Extended ACL).
- Every ACL has an implicit `deny any` at the end; forgetting `permit any` can silently block all traffic, not just the intended target.
- ACL placement matters: applying `deny host 192.168.10.10` inbound on Gi0/0 blocks the traffic as close to its source as Cisco best practice recommends for standard ACLs.
- `show ip interface <interface>` confirms exactly which ACL is applied to an interface and in which direction.
- `show access-list` match counters are the strongest evidence that an ACL is actively enforcing policy, not just configured.
- Routing determines the path; ACLs determine permission — together they form the foundation of basic network access control.
