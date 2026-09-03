# VLAN Configuration and Segmentation – Cisco Packet Tracer

## Overview

This project demonstrates **VLAN (Virtual LAN) configuration and network segmentation** on a single Cisco 2960 switch using Cisco Packet Tracer.

Four end devices were connected to one physical switch and divided into two separate VLANs — **VLAN 10 (EMPLOYEES)** and **VLAN 20 (GUESTS)**. The lab demonstrates how VLANs create independent broadcast domains on the same physical switch, allowing devices within a VLAN to communicate freely while blocking communication between different VLANs — without any router or Layer 3 device involved.

This hands-on exercise was completed as part of a self-learning journey in computer networking and cybersecurity, building on the earlier IPv4 Addressing and Subnetting lab.

---

## Objectives

- Understand the concept of VLANs and Layer 2 network segmentation.
- Create and name VLANs on a Cisco switch.
- Assign access ports to specific VLANs.
- Understand the difference between access ports and the default VLAN.
- Verify VLAN membership using `show vlan brief`.
- Test and confirm connectivity **within the same VLAN**.
- Test and confirm connectivity failure **between different VLANs** on a switch-only topology.
- Analyze ICMP request/reply PDU details for same-VLAN traffic.
- Relate VLAN segmentation to real-world network security architecture.

---

## Network Topology

Four devices were connected to a single switch (Switch0), split evenly between two VLANs.

![VLAN and Network Segmentation Topology](VLAN%20%26%20Network%20Segmentation%20Topology.png)

| Device | IP Address | Switch Port | VLAN |
|---|---|---|---|
| PC1 | 192.168.10.10 | Fa0/1 | 10 (EMPLOYEES) |
| PC2 | 192.168.10.20 | Fa0/2 | 10 (EMPLOYEES) |
| Laptop3 | 192.168.20.10 | Fa0/3 | 20 (GUESTS) |
| Laptop4 | 192.168.20.20 | Fa0/4 | 20 (GUESTS) |

---

## Host IP Configuration

Each device was configured with a static IP address matching its VLAN's subnet.

![PC IP Configuration](PC%20IP%20Configuration.png)

| Setting | Value |
|---|---|
| IPv4 Address | 192.168.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 0.0.0.0 *(not required — no inter-VLAN routing in this lab)* |

---

## VLAN Configuration on the Switch

VLAN 10 (EMPLOYEES) and VLAN 20 (GUESTS) were created on the switch, and access ports were assigned accordingly. Verified using `show vlan brief`:

![VLAN Configuration](VLAN%20Configuration.png)

| VLAN ID | Name | Status | Assigned Ports |
|---|---|---|---|
| 1 | default | active | Fa0/5–Fa0/24, Gig0/1–Gig0/2 *(unused ports)* |
| 10 | EMPLOYEES | active | Fa0/1, Fa0/2 |
| 20 | GUESTS | active | Fa0/3, Fa0/4 |

All ports not explicitly assigned remain in the default **VLAN 1**, confirming that VLAN membership must be manually configured per port.

---

## Same-VLAN Connectivity Test

PC2 (`192.168.10.20`, VLAN 10) successfully pinged PC1 (`192.168.10.10`, VLAN 10). All 4 packets were delivered with **0% loss**, TTL = 128, confirming that devices within the same VLAN communicate normally over the shared switch.

![Same VLAN Connectivity](Same%20VLAN%20Connectivity.png)

---

## Inter-VLAN Connectivity Failure

PC2 (VLAN 10) attempted to ping Laptop3 (`192.168.20.10`, VLAN 20). All 4 requests **timed out — 100% loss**. Since VLANs create separate broadcast domains at Layer 2, and no router or Layer 3 switch was present to route between them, traffic between VLAN 10 and VLAN 20 could not reach its destination.

![Inter VLAN Connectivity Failure](Inter%20VLAN%20Connectivity%20Failure.png)

This confirms that VLAN segmentation alone is sufficient to isolate traffic between groups, even without any subnetting-based restriction.

---

## PDU-Level Analysis

Cisco Packet Tracer's Simulation Mode was used to inspect the ICMP echo request/reply between PC2 and PC1 (same VLAN). The OSI Model view shows the Layer 3 IP header (ICMP Message Type 8 → Type 0) and the Layer 2 Ethernet II header carrying the frame directly between the two hosts via the switch — with no routing involved.

![VLAN and Network Segmentation PDU Details](VLAN%20%26%20Network%20Segmentation%20PDU%20Details.png)

---

## Key Learning Outcomes

- VLANs segment a single physical switch into multiple independent broadcast domains at Layer 2.
- Devices in the same VLAN communicate normally, regardless of physical port location.
- Devices in different VLANs **cannot** communicate without a router or Layer 3 switch performing inter-VLAN routing — confirmed by 100% packet loss.
- `show vlan brief` is the primary command to verify VLAN creation and port membership.
- VLAN segmentation is a foundational concept in network security architecture, limiting the blast radius of broadcast traffic and unauthorized access between device groups.
