# DHCP Snooping Lab (Cisco Packet Tracer)

A hands-on security lab demonstrating how **DHCP Snooping** protects a LAN from rogue (unauthorized) DHCP servers. A legitimate DHCP server on Router R1 is placed on a trusted switch port, while all client-facing ports and a simulated rogue DHCP server remain untrusted.

---

## Table of Contents

1. [Objective](#objective)
2. [Topology](#topology)
3. [Devices](#devices)
4. [IP Addressing](#ip-addressing)
5. [Configuration](#configuration)
6. [Verification and Evidence](#verification-and-evidence)
7. [Troubleshooting](#troubleshooting)
8. [Repository Structure](#repository-structure)
9. [Key Takeaways](#key-takeaways)

---

## Objective

- Enable DHCP Snooping on the switch for VLAN 1.
- Mark the port connected to the legitimate DHCP server (R1) as **trusted**.
- Keep PC-facing and rogue-server ports **untrusted**.
- Verify normal DHCP operation, snooping status, and DHCP renewal.
- Demonstrate that an unauthorized DHCP server on an untrusted port is not accepted.

> **Note:** This lab is intended for an isolated Packet Tracer environment for educational purposes only.

---

## Topology

![DHCP Snooping Topology](DHCP%20Snooping%20Topology.png)

*File: `DHCP Snooping Topology.png`*

```
PC1 ──┐
PC2 ──┼── Switch1 ── R1 (Legitimate DHCP Server)
PC3 ──┤
      └── Server1 (Rogue DHCP Server)
```

| Switch Port | Connected Device | DHCP Snooping State |
|-------------|------------------|---------------------|
| Fa0/1       | R1 G0/0 (legitimate DHCP) | **Trusted** |
| Fa0/2       | PC1              | Untrusted (default) |
| Fa0/3       | PC2              | Untrusted (default) |
| Fa0/4       | PC3              | Untrusted (default) |
| Fa0/5       | Rogue DHCP Server | Untrusted (default) |

---

## Devices

| Qty | Device |
|-----|--------|
| 1 | Cisco 2911 Router (Router1) |
| 1 | Cisco 2960-24TT Switch (Switch1) |
| 3 | PC-PT (PC1, PC2, PC3) |
| 1 | Server-PT (Server1 – Rogue DHCP) |

---

## IP Addressing

| Item | Value |
|------|-------|
| Network | 192.168.10.0/24 |
| R1 G0/0 (gateway and DHCP server) | 192.168.10.1/24 |
| DHCP pool start | 192.168.10.21 |
| DNS server | 8.8.8.8 |
| Rogue server (static) | 192.168.10.250/24 |

Leases observed: PC1 = 192.168.10.21, PC2 = 192.168.10.22, PC3 = 192.168.10.23.

---

## Configuration

### 1. Enable DHCP Snooping

```
Switch> enable
Switch# configure terminal
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 1
Switch(config)# end
```

### 2. Trust the legitimate DHCP server port

```
Switch# configure terminal
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# ip dhcp snooping trust
Switch(config-if)# end
```

### 3. Leave client and rogue ports untrusted

No configuration is required. Ports are untrusted by default.

---

## Verification and Evidence

### 1. Baseline connectivity before snooping

PC1 obtains an address via DHCP and successfully pings the gateway 192.168.10.1.

![Initial DHCP Test](Initial%20DHCP%20Test.png)

*File: `Initial DHCP Test.png`*

### 2. DHCP Snooping status

Command: `show ip dhcp snooping`

Confirms snooping is enabled on VLAN 1, Fa0/1 is trusted, and Fa0/3 is untrusted.

![DHCP Snooping Configuration](DHCP%20Snooping%20Configuration.png)

*File: `DHCP Snooping Configuration.png`*

### 3. Trusted port in running-config

Command: `show running-config`

Shows `ip dhcp snooping vlan 1`, `ip dhcp snooping`, and `ip dhcp snooping trust` under `interface FastEthernet0/1`.

![Trusted Port Configuration](Trusted%20Port%20Configuration.png)

*File: `Trusted Port Configuration.png`*

### 4. DHCP Snooping binding table

Command: `show ip dhcp snooping binding`

Lists the MAC address, IP address, lease, type, VLAN, and interface learned by the switch from DHCP exchanges.

![DHCP Snooping Binding Table](DHCP%20Snooping%20Binding%20Table.png)

*File: `DHCP Snooping Binding Table.png`*

### 5. DHCP renewal test

`ipconfig /release` followed by `ipconfig /renew` on PC1. The client receives a valid lease from the legitimate server (R1).

![DHCP Renewal Test](DHCP%20Renewal%20Test.png)

*File: `DHCP Renewal Test.png`*

### 6. Rogue DHCP server test

Server1 (Rogue DHCP) is connected to an untrusted port. Evidence of the test from the rogue server is shown below.

![Rogue DHCP Test](Rogue%20DHCP%20Test.png)

*File: `Rogue DHCP Test.png`*

---

## Troubleshooting

### Problem: `Total number of bindings: 0`

**Symptoms**

- `show ip dhcp snooping` showed:
  - DHCP snooping enabled
  - VLAN 1 enabled
  - Fa0/1 trusted, Fa0/3 untrusted
- PCs had valid leases (R1 showed 192.168.10.21, .22, .23), yet `show ip dhcp snooping binding` reported **0 bindings**.

**Observation on a mistaken command**

The line `Switch>DHCP snooping is enable` is **output text**, not a command. Typing it at the prompt is invalid and produces an error.

**Likely cause: Option 82 (Packet Tracer behavior)**

The output showed:

```
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
```

In Packet Tracer, when a router acts as the DHCP server, the switch inserting Option 82 can prevent the snooping binding from being created. Option 82 is not required for this lab.

**Fix steps**

1. Disable Option 82 insertion on the switch:

   ```
   Switch> enable
   Switch# configure terminal
   Switch(config)# no ip dhcp snooping information option
   Switch(config)# end
   ```

2. Verify the change. The output should now show `Insertion of option 82 is disabled`:

   ```
   Switch# show ip dhcp snooping
   ```

   Keep Fa0/1 trusted and Fa0/3 untrusted.

3. Request a fresh lease on the PC:

   ```
   C:\> ipconfig /release
   C:\> ipconfig /renew
   ```

   The PC should receive a `192.168.10.x` address (not `169.254.x.x`).

4. Re-check the snooping binding table on the switch:

   ```
   Switch# show ip dhcp snooping binding
   ```

   Expected entry format:

   ```
   MacAddress        IpAddress       Lease(sec)  Type           VLAN  Interface
   xxxx.xxxx.xxxx    192.168.10.21   ...         dhcp-snooping  1     Fa0/x
   ```

   Repeat the release/renew on PC2 and PC3 so their entries are added too.

5. Cross-check on R1:

   ```
   R1# show ip dhcp binding
   ```

**Important distinction**

| Command | Device | Meaning |
|---------|--------|---------|
| `show ip dhcp binding` | R1 | Which IPs the DHCP **server** leased to clients |
| `show ip dhcp snooping binding` | Switch | Which IP ↔ MAC ↔ port mappings the **switch observed** from DHCP exchanges |

Three leases on R1 do not automatically mean the switch learned three snooping bindings.

**If bindings are still 0**

Use Packet Tracer **Simulation Mode** (filter: DHCP) to trace the Discover, Offer, Request, and ACK packets and identify the stage where traffic is dropped.

---

## Repository Structure

```
DHCP-Snooping/
├── DHCP Snooping.pkt
├── DHCP Snooping Topology.png
├── Initial DHCP Test.png
├── DHCP Snooping Configuration.png
├── Trusted Port Configuration.png
├── DHCP Snooping Binding Table.png
├── DHCP Renewal Test.png
├── Rogue DHCP Test.png
└── README.md
```

---

## Key Takeaways

```
DHCP Snooping
     ↓
Identify the trusted DHCP path
     ↓
Trust the legitimate DHCP server port
     ↓
Keep client-facing ports untrusted
     ↓
Block unauthorized DHCP replies
```

- Only trusted ports may send DHCP server messages (Offer/ACK).
- Untrusted ports are limited to client messages.
- The snooping binding table (IP ↔ MAC ↔ port ↔ VLAN) is built from observed DHCP exchanges and is reused by other Layer 2 security features.

---
