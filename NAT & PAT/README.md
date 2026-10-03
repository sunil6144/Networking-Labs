# Static NAT – Private-to-Public Address Translation (Cisco Packet Tracer)

## Overview

This project demonstrates **Static NAT (Network Address Translation)** on a Cisco 2911 router in Cisco Packet Tracer — the first lab in the NAT/PAT phase of the networking series.

A private internal host (`PC1 – 192.168.10.10`) was mapped one-to-one to a dedicated public address (`198.51.100.10`) on Router1, so it could communicate with an outside server (`203.0.113.10`) without ever exposing its real private IP on the public-facing side of the network.

> **Important distinction:** NAT is an *addressing* mechanism, not a security mechanism on its own. It hides internal addressing structure, but traffic filtering and access control are handled separately by ACLs/firewalls (see the earlier ACL labs in this series).

---

## Objectives

- Understand the purpose of Static NAT: one-to-one mapping of a private IP to a public IP.
- Distinguish NAT **inside** and **outside** interfaces.
- Configure a Static NAT translation using `ip nat inside source static`.
- Mark the correct interfaces with `ip nat inside` / `ip nat outside`.
- Verify interface addressing using `show ip interface brief`.
- Confirm end-to-end connectivity after NAT is applied.
- Read and interpret a live NAT translation table using `show ip nat translations`.
- Understand `Inside local`, `Inside global`, `Outside local`, and `Outside global` address roles.

---

## Network Topology

PC1 sits behind Router1 (the NAT router), which connects through Router2 to an outside server.

![Static NAT topology](Static%20NAT%20topology.png)

| Device | Interface | IP Address |
|---|---|---|
| PC1 | NIC | 192.168.10.10/24 |
| Router1 (Gi0/0) | NAT inside | 192.168.10.1/24 |
| Router1 (Gi0/1) | NAT outside | 10.0.0.1/30 |
| Router2 (Gi0/0) | — | 10.0.0.2/30 |
| Router2 (Gi0/1) | — | 203.0.113.1/24 |
| Server1 | NIC | 203.0.113.10/24 |

**Static NAT mapping used:**

```text
Inside local (private)        Inside global (public)
192.168.10.10          ↔      198.51.100.10
```

---

## Interface Verification

Router1's interfaces were verified as up and correctly addressed on both the inside (LAN) and outside (WAN) sides.

![Interface Verification](Interface%20Verification.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | up/up |
| GigabitEthernet0/1 | 10.0.0.1 | up/up |

---

## Static NAT Configuration

Router1 was configured with the NAT roles on each interface and the static translation rule:

```text
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 ip nat outside

ip nat inside source static 192.168.10.10 198.51.100.10
ip route 203.0.113.0 255.255.255.0 10.0.0.2
```

![Static NAT Configuration](Static%20NAT%20Configuration.png)

> **Note on the running-config:** an earlier, incorrect static mapping (`192.168.10.10 203.0.133.10`) from a first configuration attempt is still visible alongside the corrected one (`192.168.10.10 198.51.100.10`). It was left in place as a harmless leftover — only the correct mapping is actually used for translation, as confirmed in the NAT translation table below.

---

## NAT Translation Table

`show ip nat translations` on Router1 confirms the static mapping is active and in use.

![NAT Translation Table](NAT%20Translation%20Table.png)

```text
Pro   Inside global        Inside local          Outside local    Outside global
icmp  198.51.100.10:16     192.168.10.10:16      10.0.0.2:16      10.0.0.2:16
...
icmp  198.51.100.10:43     192.168.10.10:43      203.0.113.10:43  203.0.113.10:43
...
---   198.51.100.10        192.168.10.10         ---              ---
```

Two distinct patterns are visible here, which line up with the real troubleshooting done in this lab:

- **Earlier translations (ports 16–25):** `Outside local`/`Outside global` show only `10.0.0.2` (Router2) — these were captured while Server1's default gateway was misconfigured, so ICMP traffic only reached as far as Router2 and never completed the round trip to the server.
- **Later translations (ports 43–52):** `Outside local`/`Outside global` correctly show `203.0.113.10` — captured after fixing Server1's default gateway, confirming traffic now reaches and returns from the actual server.
- The final `---` entry is the static mapping itself, always present regardless of active traffic.

---

## Post-NAT Connectivity Test

With the corrected server gateway and active NAT translation, PC1 successfully pinged the server at `203.0.113.10`.

![Post-NAT Connectivity Test](Post-NAT%20Connectivity%20Test.png)

Result: 4 of 4 replies received, **0% loss**, TTL = 126 — confirming the packet was translated by Router1 and successfully routed through Router2 to reach the server, two hops away.

---

## Key Learning Outcomes

- Static NAT creates a fixed, one-to-one mapping between a private (`inside local`) and public (`inside global`) address — ideal for internal servers that need a consistent public identity.
- `ip nat inside` and `ip nat outside` must be applied to the correct interfaces for NAT to activate at all; the direction of traffic flow determines which is which.
- `show ip nat translations` is the definitive way to confirm NAT is actually translating live traffic, not just configured.
- A misconfigured default gateway on the *destination* device can look like a NAT problem — the translation table itself (`Outside local`/`Outside global` stuck at the next-hop router instead of the real destination) was the key clue that pointed to the real issue here.
- NAT changes addressing, not permissions — it works alongside, not instead of, ACL-based traffic filtering.
