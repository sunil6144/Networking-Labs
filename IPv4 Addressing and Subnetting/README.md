# IPv4 Addressing and Subnetting – Cisco Packet Tracer

## Overview

This project demonstrates the practical implementation of **IPv4 addressing and subnetting** using Cisco Packet Tracer.

A `/26` subnet mask was applied to the `192.168.10.0` network, splitting it into smaller logical subnets. Devices were assigned static IP addresses within these subnets, and connectivity was tested both **within the same subnet** and **across different subnets** through a router.

This hands-on exercise was completed as part of a self-learning journey in computer networking and cybersecurity.

---

## Objectives

- Understand IPv4 addressing and CIDR notation.
- Calculate subnets, usable host ranges, and broadcast addresses from a given subnet mask.
- Assign valid static IP addresses to hosts based on their subnet.
- Understand the role of the default gateway in inter-subnet communication.
- Verify connectivity between two hosts in the **same subnet**.
- Verify connectivity between two hosts in **different subnets**, routed through Router0.
- Compare TTL values to distinguish direct (Layer 2) delivery from routed (Layer 3) delivery.
- Analyze packet forwarding behavior using Cisco Packet Tracer.

---

## Network Topology

The topology consists of two switches (Switch1, Switch2) connected through a single router (Router0), with two end devices attached to each switch.

![IPv4 Addressing and Subnetting Topology](IPv4%20Addressing%20and%20Subnetting%20topology.png)

| Device | IP Address | Connected To |
|---|---|---|
| PC1 | 192.168.10.10 | Switch1 |
| PC2 | 192.168.10.20 | Switch1 |
| Laptop3 | 192.168.10.70 | Switch2 |
| Laptop4 | 192.168.10.80 | Switch2 |

---

## Subnetting Breakdown

The base network `192.168.10.0/24` was subnetted using a `/26` (`255.255.255.192`) mask, producing blocks of 64 addresses each.

| Subnet | Network Address | Usable Host Range | Broadcast Address | Devices |
|---|---|---|---|---|
| Subnet 1 | 192.168.10.0/26 | 192.168.10.1 – 192.168.10.62 | 192.168.10.63 | PC1, PC2 |
| Subnet 2 | 192.168.10.64/26 | 192.168.10.65 – 192.168.10.126 | 192.168.10.127 | Laptop3, Laptop4 |

PC1 and PC2 fall in **Subnet 1**, while Laptop3 and Laptop4 fall in **Subnet 2** — making PC↔PC traffic same-subnet, and PC↔Laptop traffic cross-subnet (routed).

---

## Host IP Configuration

PC1 was configured with a static IP address, subnet mask, and default gateway matching Subnet 1.

![PC1 IP Configuration](PC1%20IP%20Configuration.png)

| Setting | Value |
|---|---|
| IPv4 Address | 192.168.10.10 |
| Subnet Mask | 255.255.255.192 |
| Default Gateway | 192.168.10.1 |

---

## Router Interface Verification

Screenshot not yet added — enable once uploaded by removing this comment block:
Router0's interfaces were verified to confirm each was assigned the correct gateway address for its respective subnet and brought up (no shutdown).

![Router Interface Verification](Router%20Interface%20Verification.png)

---

## Same-Subnet Connectivity Test

PC1 (`192.168.10.10`) successfully pinged PC2 (`192.168.10.20`), both residing in **Subnet 1**. All 4 packets were delivered with **0% loss**, and the reply **TTL = 128** — indicating the traffic never left the local subnet (no router hop, pure Layer 2 switching).

![Same Subnet Connectivity](Same%20Subnet%20Connectivity.png)

---

## Different-Subnet Connectivity Test

PC2 (`192.168.10.20`, Subnet 1) pinged Laptop3 (`192.168.10.70`, Subnet 2). The first packet timed out (ARP resolution delay), but the remaining 3 packets succeeded with a reply **TTL = 127** — one less than the same-subnet test, confirming the packet was routed through **one hop (Router0)** to reach the destination subnet.

![Different Subnet Connectivity](Different%20Subnet%20Connectivity.png)

---

## PDU-Level Analysis

Screenshots not yet added — enable once uploaded by removing this comment block:
Cisco Packet Tracer's Simulation Mode was used to inspect the PDU at each stage of both the same-subnet and different-subnet ping tests, showing how the router modifies the Layer 2 (MAC) header while keeping the Layer 3 (IP) header unchanged during forwarding.

<!-- ![Same Subnet PDU Details](Same%20Subnet%20PDU%20Details.png)  -->

![Different Subnet PDU Details](Different%20Subnet%20PDU%20Details.png) 

---

## Key Learning Outcomes

- A `/26` mask divides a `/24` network into 4 subnets of 64 addresses each (62 usable hosts per subnet).
- Devices in the same subnet communicate directly via Layer 2 (switching) — no router involvement, TTL unchanged.
- Devices in different subnets require a default gateway and are routed via Layer 3 — TTL decrements by 1 per hop.
- Correct default gateway configuration is essential for any inter-subnet communication to succeed.
