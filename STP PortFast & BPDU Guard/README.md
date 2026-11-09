# STP Security: PortFast & BPDU Guard (Cisco Packet Tracer)

## Overview

This project demonstrates **PortFast** and **BPDU Guard** on a Cisco 2960 switch in Cisco Packet Tracer. They protect Spanning Tree Protocol (STP) from an unauthorised switch being plugged into a port that should only have an end device on it.

- **PortFast** lets an endpoint-facing access port go straight to forwarding instead of waiting through the normal STP states.
- **BPDU Guard** watches that same port. Endpoint ports should never receive STP messages (BPDUs), so if one arrives, a switch has been connected and the port is shut down (err-disabled).

A fresh topology was built for this lab. Both settings were applied **only to PC-facing ports**, never to a switch-to-switch link.

---

## Objectives

- Understand why an unauthorised switch on an access port is an STP risk.
- Enable PortFast and BPDU Guard on PC-facing access ports.
- Verify PortFast on a port with `show spanning-tree interface ... detail`.
- Confirm normal connectivity before and after the security test.
- Connect a second switch to a protected port and observe the result.
- Recover the port with `shutdown` / `no shutdown` and confirm connectivity returns.

---

## Network Topology

Two PCs connect to Switch0 (the protected switch). Switch1 starts disconnected and is later used as the "unauthorised" switch.

![STP Security Topology](STP%20Security%20Topology.png)

| Device | Address / Role | Switch Port |
|---|---|---|
| PC1 | 192.168.1.10 | Switch0 Fa0/1 |
| PC2 | 192.168.1.20 | Switch0 Fa0/2 |
| Switch0 | Protected main switch | — |
| Switch1 | Test switch, default configuration, disconnected at first | — |

No default gateway is used, since both PCs are in the same subnet. A baseline `ping 192.168.1.20` from PC1 was successful before any STP security was configured.

---

## PortFast & BPDU Guard Configuration

Applied on both PC-facing ports of Switch0:

```text
interface FastEthernet0/1
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable

interface FastEthernet0/2
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
```

| Command | Purpose |
|---|---|
| `switchport mode access` | Makes the port an endpoint-facing access port |
| `spanning-tree portfast` | Moves the port to forwarding quickly |
| `spanning-tree bpduguard enable` | Shuts the port down if a BPDU is received on it |

Verified on `Fa0/1` with `show spanning-tree interface fastethernet 0/1 detail`:

![PortFast BPDU Guard Config](PortFast%20BPDU%20Guard%20Config.png)

| Field | Value |
|---|---|
| Port | FastEthernet0/1, VLAN0001 |
| Port path cost / priority | 19 / 128 (Port Identifier 128.1) |
| Designated root | priority 32769, address 0004.9A3B.2BCC |
| Transitions to forwarding | 1 |
| PortFast | **The port is in the portfast mode** |
| Link type | point-to-point by default |

> The two settings were applied only to endpoint ports. They must not be added to switch-to-switch links without planning, because that can disrupt connectivity.

---

## BPDU Guard Violation

PC1 was disconnected from `Fa0/1`, and Switch1's `Fa0/1` was connected to Switch0's `Fa0/1` with a copper cross-over cable, with Switch1 left on its default configuration. PC2 stayed connected on `Fa0/2`.

![BPDU Guard Violation Topology](BPDU%20Guard%20Violation%20Topology.png)

The link between Switch0 `Fa0/1` and Switch1 `Fa0/1` is shown with **red** link indicators instead of green, meaning the port is no longer forwarding. This is the BPDU Guard response: a switch on a protected port causes the port to be shut down. PC2's link on `Fa0/2` stays green and unaffected.

<!-- Screenshot disabled: the captured image was blank, so it is not shown.
     Re-capture `show interfaces status` or `show logging` from Switch0 and enable it by removing this comment.
![BPDU Guard Violation](BPDU%20Guard%20Violation.png)
-->

The text-output capture of this violation (`show interfaces status` / `show logging`) came out blank, so the port state is shown here through the topology view only.

---

## BPDU Guard Port Recovery

Switch1 was disconnected, PC1 was reconnected to Switch0 `Fa0/1`, and the port was reset manually:

```text
interface FastEthernet0/1
 shutdown
 no shutdown
```

After recovery, PC1 pinged PC2 again: 4/4 replies, 0% loss, TTL = 128.

![BPDU Guard Port Recovery](BPDU%20Guard%20Port%20Recovery.png)

The configuration was then saved with `copy running-config startup-config`, and the lab file was saved as `STP-PortFast-BPDU-Guard.pkt`.

---

## Key Learning Outcomes

- PortFast is meant for **endpoint** ports only; it skips the normal STP wait so PCs get connectivity quickly.
- BPDU Guard turns an unexpected BPDU on a PortFast port into an immediate shutdown, which stops a rogue switch from taking part in STP.
- The red link in the violation topology shows the port stopped forwarding when Switch1 was connected, while the other port (`Fa0/2`) was unaffected.
- A protected port that has been shut down by BPDU Guard does not come back by itself here. It is reset with `shutdown` then `no shutdown`, after the offending switch is removed.
- Never reconnect the unauthorised switch to a protected PC port before the recovery step.
