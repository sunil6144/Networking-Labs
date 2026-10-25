# Dynamic ARP Inspection (DAI) – Cisco Packet Tracer

## Overview

This project configures **Dynamic ARP Inspection (DAI)** on a Cisco 2960 switch, building directly on the DHCP Snooping lab. DAI checks each ARP packet arriving on an untrusted port against the **DHCP snooping binding table** (IP + MAC + VLAN + port). If the ARP information does not match a binding, the switch is meant to drop it. This is the defence against ARP spoofing / ARP poisoning, where an attacker claims "the gateway's IP belongs to my MAC".

> **Result summary:** DAI and its dependencies were configured and the configuration was verified. Actual blocking of spoofed ARP was **not** confirmed in Packet Tracer (see *Simulator Limitation*).

---

## Objectives

- Understand ARP spoofing and how DAI validates ARP against DHCP snooping bindings.
- Confirm the DHCP snooping binding table is populated (DAI's prerequisite).
- Enable DAI on VLAN 1.
- Trust the uplink toward the router and leave client ports untrusted.
- Verify with `show ip arp inspection interfaces` and `show ip arp inspection statistics`.
- Confirm that legitimate traffic still works with DAI enabled.

---

## Network Topology

![DAI Topology](DAI%20Topology.png)

| Device | Address |
|---|---|
| PC1 | 192.168.10.22 |
| PC2 | 192.168.10.23 |
| PC3 | 192.168.10.21 |
| Server0 | static host used as the test machine |
| Router0 | gateway 192.168.10.1 |
| Switch0 | DHCP snooping + DAI on VLAN 1 |

---

## DHCP Snooping Binding Table (Prerequisite)

DAI relies on this table, so it was checked first:

![DHCP Snooping Binding Table](DHCP%20Snooping%20Binding%20Table.png)

| MAC Address | IP Address | Lease (sec) | Type | VLAN | Interface |
|---|---|---|---|---|---|
| 0009.7C72.1EBC | 192.168.10.21 | 86400 | dhcp-snooping | 1 | Fa0/3 |
| 000D.BDC0.AB65 | 192.168.10.22 | 86400 | dhcp-snooping | 1 | Fa0/4 |
| 0001.C9E8.933E | 192.168.10.23 | 86400 | dhcp-snooping | 1 | Fa0/5 |

Total number of bindings: 3.

---

## DAI Configuration

```text
ip arp inspection vlan 1

interface FastEthernet0/1
 ip dhcp snooping trust
 ip arp inspection trust
```

---

## DAI Trusted Interface

`show ip arp inspection interfaces`:

![DAI Trusted Interface](DAI%20Trusted%20Interface.png)

| Interface | Trust State | Rate (pps) | Burst Interval |
|---|---|---|---|
| Fa0/1 | Trusted | 15 | 1 |
| Fa0/2 | Untrusted | 15 | 1 |
| Fa0/3 | Untrusted | 15 | 1 |
| Fa0/4 | Untrusted | 15 | 1 |
| Fa0/5 | Untrusted | 15 | 1 |

Only the router uplink is trusted; all other ports are inspected.

---

## DAI Connectivity Test

With DAI enabled, legitimate traffic still works: a ping to the gateway `192.168.10.1` returned 4/4 replies, 0% loss, TTL = 255.

![DAI Connectivity Test](DAI%20Connectivity%20Test.png)

---

## DAI Statistics

`show ip arp inspection statistics` for VLAN 1:

![DAI Statistics](DAI%20Statistics.png)

All counters were **0**: Forwarded, Dropped, DHCP Drops, ACL Drops, DHCP Permits, ACL Permits, Source MAC Failures, Dest MAC Failures and IP Validation Failures.

---

## Simulator Limitation

DHCP Snooping and DAI were configured and verified in Cisco Packet Tracer. However, the simulator did not enforce DAI against a static-IP test host: that host could still communicate, and the inspection counters stayed at zero. Therefore **successful ARP-spoofing prevention was not confirmed** in this environment. The zero counters should not be read as proof of protection.

This looks like a simulator limitation, but that was not independently confirmed here. To demonstrate real enforcement, the same configuration should be tested on an IOS image that fully supports DAI (for example GNS3 or EVE-NG) or on a physical managed switch.

---

## Key Learning Outcomes

- DAI depends on a populated DHCP snooping binding table: DHCP, then snooping, then bindings, then ARP validation.
- Trust the uplink with `ip arp inspection trust`; leave host-facing ports untrusted.
- `show ip arp inspection interfaces` confirms trust states; `show ip arp inspection statistics` shows forwarded and dropped ARP packets.
- Configuration evidence and enforcement evidence are different things. Documenting the limit of what was verified is part of honest lab reporting.
