# Extended ACL – Port-Level Traffic Filtering (Cisco Packet Tracer)

## Overview

This project extends the earlier **Standard ACL** lab by demonstrating an **Extended Access Control List** on a Cisco 2911 router in Cisco Packet Tracer.

Where a Standard ACL can only filter by source IP address, an Extended ACL adds **destination IP, protocol, and port number** to the decision — allowing much more precise, service-level traffic control. In this lab, PC1 was denied **HTTP (TCP/80)** access to Server1, while **ICMP (ping)** between the same two hosts remained fully allowed.

This lab continues the security-focused phase of the networking series, moving from broad host-level blocking to precise, protocol-aware filtering.

---

## Objectives

- Understand what an Extended ACL can match that a Standard ACL cannot: destination IP, protocol, and port.
- Configure an Extended ACL that denies a specific TCP port (HTTP/80) between two specific hosts, while permitting everything else.
- Apply the ACL to the correct router interface, closest to the traffic source.
- Verify ACL application using `show ip interface`.
- Confirm both ping and HTTP work **before** the ACL is applied (baseline proof).
- Confirm ping still works but HTTP is blocked **after** the ACL is applied — proving protocol-level (not host-level) filtering.
- Use `show access-lists` match counters to prove real-time enforcement per protocol.

---

## Network Topology

A single router connects PC1 to a web server (Server1) hosting an HTTP service.

![Extended ACL topology](Extended%20ACL%20topology.png)

| Device | Interface | IP Address |
|---|---|---|
| PC1 | NIC | 192.168.10.10/24 |
| R1 | Gi0/0 | 192.168.10.1/24 |
| R1 | Gi0/1 | 192.168.20.1/24 |
| Server1 | NIC | 192.168.20.10/24 |

**Target policy:**

```text
PC1 → Server1   ICMP (ping)      ✅ ALLOW
PC1 → Server1   HTTP (TCP/80)    ❌ DENY
```

---

## Initial Ping Test (Before ACL)

Before any ACL was configured, PC1 successfully pinged Server1, confirming basic routing was working.

![Initial Ping Test](Initial%20Ping%20Test.png)

Result: 3 of 4 replies received (first packet lost to normal ARP delay), TTL = 127.

---

## Initial HTTP Test (Before ACL)

PC1's web browser successfully loaded Server1's page at `http://192.168.20.10`, confirming HTTP service was reachable before any filtering was applied.

![Initial HTTP Test](Initial%20HTTP%20Test.png)

---

## Extended ACL Configuration

The following Extended ACL was configured on R1 to deny only HTTP (TCP port 80) traffic from PC1 to Server1, while permitting all other traffic:

```text
access-list 100 deny tcp host 192.168.10.10 host 192.168.20.10 eq 80
access-list 100 permit ip any any
```

| Part | Meaning |
|---|---|
| `deny tcp` | Block TCP traffic specifically (not ICMP, not UDP) |
| `host 192.168.10.10` | Only from this exact source — PC1 |
| `host 192.168.20.10` | Only to this exact destination — Server1 |
| `eq 80` | Only on port 80 — HTTP |
| `permit ip any any` | Everything else — including ICMP — stays allowed |

It was then applied **inbound** on Gi0/0, the interface facing PC1's subnet:

```text
interface GigabitEthernet0/0
ip access-group 100 in
```

---

## ACL Applied to Interface

`show ip interface gigabitEthernet 0/0` was used to confirm the ACL is actively bound to the interface.

![ACL Applied to Interface](ACL%20Applied%20to%20Interface.png)

Key line confirming the ACL is active:

```text
Inbound access list is 100
```

---

## Ping Allowed Test (After ACL)

With the Extended ACL active, PC1 pinged Server1 again — this time with **full success**, confirming ICMP was never targeted by the ACL.

![Ping Allowed Test](Ping%20Allowed%20Test.png)

Result: 4 of 4 replies received, **0% loss**, TTL = 127.

---

## Blocked HTTP Test (After ACL)

PC1's browser attempted to load `http://192.168.20.10` again. This time the page failed to load (`Host Name Unresolved`), confirming HTTP traffic was blocked by the ACL while the underlying network path remained intact.

![Blocked HTTP Test](Blocked%20HTTP%20Test.png)

---

## Extended ACL Hit Count — Proof of Enforcement

`show access-lists` confirms the ACL is actively matching traffic on a **per-protocol basis** — this single output also captures the exact configuration, since the command and its result are identical to what was applied.

![Extended ACL Hit Count](Extended%20ACL%20Hit%20Count.png)

```text
Extended IP access list 100
    10 deny tcp host 192.168.10.10 host 192.168.20.10 eq www (27 match(es))
    20 permit ip any any (7 match(es))
```

- **27 matches** on the `deny tcp ... eq www` line — every failed HTTP attempt (each browser page load generates multiple TCP segments: SYN, retransmits, etc.), all blocked.
- **7 matches** on `permit ip any any` — this covers the ICMP ping traffic (and any other non-HTTP traffic), confirming it correctly bypassed the deny rule and was explicitly permitted.

This single screenshot doubles as both the enforcement proof and the configuration record, since the command output reproduces the exact ACL statements entered.

---

## Key Learning Outcomes

- Extended ACLs can match on **source IP, destination IP, protocol, and port** — Standard ACLs can only match source IP.
- `eq 80` (or `eq www`) restricts a rule to a single port, allowing precise, service-level filtering instead of blocking a host entirely.
- Because the deny rule was scoped to `tcp ... eq 80`, ICMP traffic was never evaluated against it and fell through to the `permit ip any any` line.
- `show access-lists` hit counters that increase per-protocol are strong evidence that filtering is happening exactly as intended — not just for the "blocked" service, but confirming the "allowed" one is truly unaffected.
- Extended ACLs are the foundation of real-world firewall-style rules, where different services need different access policies between the same two hosts.
