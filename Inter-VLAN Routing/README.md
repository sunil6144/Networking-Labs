# Inter-VLAN Routing (Router-on-a-Stick) – Cisco Packet Tracer

## Overview

This project demonstrates **Inter-VLAN Routing** using the **Router-on-a-Stick** technique in Cisco Packet Tracer.

A single physical link connects Router0 to Switch0, carrying traffic for both VLAN 10 (USERS) and VLAN 20 (GUESTS) using **802.1Q trunking**. Instead of using separate physical interfaces, the router's single interface was divided into two **logical subinterfaces**, each configured with 802.1Q encapsulation matching its VLAN and acting as the default gateway for that VLAN — allowing devices in different VLANs to communicate through the router.

This lab builds directly on the earlier **VLAN Configuration** and **VLAN Trunking** labs, completing the picture of how VLANs are isolated at Layer 2 and then selectively connected at Layer 3.

---

## Objectives

- Understand the Router-on-a-Stick concept for inter-VLAN communication.
- Configure router subinterfaces with 802.1Q encapsulation matching each VLAN ID.
- Assign each subinterface an IP address to act as the default gateway for its VLAN.
- Verify subinterface status using `show ip interface brief`.
- Verify the routing table using `show ip route`.
- Verify the trunk link on the switch side using `show interfaces trunk`.
- Test and confirm connectivity **within the same VLAN** (Layer 2 only, no routing).
- Test and confirm connectivity **between different VLANs** (routed through Router0).
- Compare TTL values to distinguish same-VLAN from inter-VLAN traffic.
- Trace the full packet path hop-by-hop using PDU analysis.

---

## Network Topology

Router0 connects to Switch0 over a single trunk link. Devices are split across VLAN 10 and VLAN 20 on the same switch.

![Inter VLAN Routing Topology](Inter%20VLAN%20Routing%20Topology.png)

| Device | IP Address | VLAN |
|---|---|---|
| PC1 | 192.168.10.10 | 10 (USERS) |
| PC2 | 192.168.10.20 | 10 (USERS) |
| Laptop3 | 192.168.20.10 | 20 (GUESTS) |
| Laptop4 | 192.168.20.20 | 20 (GUESTS) |
| Router0 (Gi0/0.10) | 192.168.10.1 | Gateway for VLAN 10 |
| Router0 (Gi0/0.20) | 192.168.20.1 | Gateway for VLAN 20 |

---

## Switch VLAN Configuration

VLAN membership was verified on Switch0 using `show vlan brief`.

![Switch VLAN Details](Switch%20VLAN%20Details.png)

| VLAN ID | Name | Status | Ports |
|---|---|---|---|
| 1 | default | active | Fa0/6–Fa0/24, Gig0/2 *(unused)* |
| 10 | USERS | active | Fa0/2, Fa0/3 |
| 20 | GUESTS | active | Fa0/4, Fa0/5 |

---

## Trunk Link to Router

The link between Switch0 and Router0 was configured and verified as an 802.1Q trunk carrying both VLANs.

![Trunk Interfaces Status](Trunk%20Interfaces%20Status.png)

| Parameter | Value |
|---|---|
| Port | Fa0/1 |
| Mode | on (trunking) |
| Encapsulation | 802.1Q |
| Native VLAN | 1 |
| VLANs allowed on trunk | 1–1005 |
| VLANs active in management domain | 1, 10, 20 |

---

## Router Subinterface Configuration

Router0's single Gigabit interface was split into two 802.1Q subinterfaces, each acting as the gateway for one VLAN.

![IP Interfaces](IP%20Interfaces.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | unassigned | up/up *(parent — no IP, carries the trunk)* |
| GigabitEthernet0/0.10 | 192.168.10.1 | up/up |
| GigabitEthernet0/0.20 | 192.168.20.1 | up/up |

---

## Routing Table

`show ip route` confirms both VLAN subnets are directly connected via their respective subinterfaces, with no additional routing protocol required.

![IP Routes](IP%20Routes.png)

```text
192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
```

---

## Same-VLAN Connectivity Test

Laptop3 (`192.168.20.10`, VLAN 20) successfully pinged Laptop4 (`192.168.20.20`, VLAN 20). All 4 packets were delivered with **0% loss**, TTL = 128 — confirming that same-VLAN traffic stays within the switch and never reaches the router.

![Same VLAN Connectivity](Same%20VLAN%20Connectivity.png)

---

## Inter-VLAN Connectivity Test

PC1 (`192.168.10.10`, VLAN 10) successfully pinged Laptop3 (`192.168.20.10`, VLAN 20). The reply came back with **TTL = 127** — one less than the same-VLAN test, confirming the packet was routed through **one hop (Router0)** via its subinterfaces before reaching the other VLAN.

![Inter VLAN Connectivity](Inter%20VLAN%20Connectivity.png)

---

## PDU-Level Analysis

A detailed hop-by-hop walkthrough of this exact ping (PC1 → Switch0 → Router0 → Switch0 → Laptop3), showing how the 802.1Q tag is added and removed at each stage, is available here:

📄 [View PDU Screenshots](docs/PDU-Screenshots.md)

---

## Key Learning Outcomes

- Router-on-a-Stick allows a single physical router interface to serve as the gateway for multiple VLANs using subinterfaces.
- Each subinterface is configured with 802.1Q encapsulation matching its VLAN ID and holds the default gateway IP for that VLAN.
- Same-VLAN traffic never leaves the switch — TTL stays unchanged (128).
- Inter-VLAN traffic is routed through the router — TTL decrements by 1 (127) per hop.
- The switch tags frames destined for the router's trunk link and untags them again before delivering to the destination VLAN's access port.
