# Switch Port Security – Sticky MAC and Shutdown Violation Mode (Cisco Packet Tracer)

## Overview

This project demonstrates **Switch Port Security** on a Cisco 2960 switch in Cisco Packet Tracer. Port Security protects a switch access port from unauthorised devices by controlling which MAC addresses are allowed to use it.

Port `Fa0/1` was configured to learn the MAC address of the first device that connects (**sticky MAC**), allow **only one** MAC address, and **shut down** if any other device appears. An unauthorised PC was then connected to the secured port to confirm the switch reacts as expected, and the port was recovered afterwards.

A fresh topology was built for this lab so earlier configurations could not affect the results.

---

## Objectives

- Understand how MAC-based port security limits who can use a switch port.
- Configure sticky MAC learning on an access port.
- Limit the port to a maximum of one MAC address.
- Set the violation mode to `shutdown`.
- Verify learned MACs, port status and the violation counter.
- Connect an unauthorised device and confirm the port is shut down.
- Recover the port and confirm the original device is still the only authorised MAC.

---

## Network Topology

PC1 and PC2 connect to Switch0. PC3 starts disconnected and is used later as the unauthorised device.

![Switch Port Security Topology](Switch%20Port%20Security%20Topology.png)

| Device | IP Address | Switch Port |
|---|---|---|
| PC1 | 192.168.1.10 | Fa0/1 (secured) |
| PC2 | 192.168.1.20 | Fa0/2 |
| PC3 | 192.168.1.30 | Not connected at first (unauthorised test device) |

No default gateway is needed, since all devices are in the same subnet.

---

## Port Security Configuration

Applied on `Fa0/1`:

```text
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

| Command | Purpose |
|---|---|
| `switchport mode access` | Makes the port an access port (required for port security) |
| `switchport port-security` | Enables port security |
| `maximum 1` | Allows only one MAC address on the port |
| `mac-address sticky` | Learns the connected device's MAC and keeps it in the configuration |
| `violation shutdown` | Shuts the port down if an unauthorised MAC appears |

A baseline `ping 192.168.1.20` from PC1 to PC2 was successful before security was applied, and again afterwards so the switch could learn PC1's MAC.

---

## Sticky MAC Verification

`show port-security interface fastethernet 0/1` after PC1 sent traffic:

![Sticky MAC Verification](Sticky%20MAC%20Verification.png)

| Field | Value |
|---|---|
| Port Security | Enabled |
| Port Status | Secure-up |
| Violation Mode | Shutdown |
| Maximum MAC Addresses | 1 |
| Total MAC Addresses | 1 |
| Sticky MAC Addresses | 1 |
| Last Source Address:Vlan | 0001.9661.3B94 : 1 |
| Security Violation Count | 0 |

PC1's MAC (`0001.9661.3B94`) was learned automatically and the port is healthy with no violations.

---

## Port Security Violation

PC1 was disconnected from `Fa0/1` and PC3 (`192.168.1.30`, static IP) was connected to the same port with a copper straight-through cable. PC3 has a different MAC address from PC1, so the switch treated it as unauthorised. PC3's `ping 192.168.1.20` failed, and the switch responded by shutting the port down.

![Port Security Violation](Port%20Security%20Violation.png)

| Field | Value |
|---|---|
| Port Status | **Secure-shutdown** |
| Violation Mode | Shutdown |
| Sticky MAC Addresses | 1 |
| Last Source Address:Vlan | 0002.1728.47B8 : 1 (the unauthorised device) |
| Security Violation Count | **1** |

The port moved from `Secure-up` to `Secure-shutdown`, and the offending MAC (`0002.1728.47B8`) was recorded. This is the expected security response.

---

## Port Recovery Verification

PC3 was disconnected, PC1 was reconnected to `Fa0/1`, and the port was reset manually:

```text
interface FastEthernet0/1
 shutdown
 no shutdown
```

After a few seconds the port came back up as `Secure-up`, and PC1's `ping 192.168.1.20` to PC2 succeeded again, because PC1's MAC was still the sticky secure address. The earlier violation count can remain after recovery, which is normal. The configuration was then saved with `copy running-config startup-config`, and the lab file was saved as `Switch-Port-Security.pkt`.

`show port-security address` after recovery:

![Port Recovery Verification](Port%20Recovery%20Verification.png)

| Vlan | MAC Address | Type | Port |
|---|---|---|---|
| 1 | 0001.9661.3B94 | SecureSticky | Fa0/1 |

PC1's sticky MAC (`0001.9661.3B94`) is still the only secure address on `Fa0/1`, so after recovery only the original, authorised device is bound to the port and normal connectivity is restored.

---

## Key Learning Outcomes

- **Sticky MAC** learns the connected device's MAC automatically, so addresses do not have to be typed in by hand.
- **Maximum 1** means a second, different MAC on the same port is a violation.
- **Shutdown** mode disables the port immediately (`Secure-shutdown`) and counts the violation.
- A shut-down port is not recovered automatically in this setup: it needs `shutdown` then `no shutdown`.
- The sticky MAC stays bound to the port after recovery, so a different device still cannot use it, while the original device (PC1) regains connectivity.
- Port Security is a Layer 2 defence against someone plugging an unknown device into a wall port.
