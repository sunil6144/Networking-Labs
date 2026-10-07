# DHCP Server – Automatic IP Addressing (Cisco Packet Tracer)

## Overview

This project demonstrates configuring a **Cisco router as a DHCP server** in Cisco Packet Tracer — the first lab in the services phase of the networking series, moving from manually assigning every IP address to **automatic, pool-based addressing**.

Three PCs were set to obtain their IP configuration automatically, and Router1 was configured to lease addresses from a defined pool — handing out IP address, subnet mask, default gateway, and DNS server with no manual input on the client side.

---

## Objectives

- Understand why DHCP is used instead of manual IP configuration in real networks.
- Configure a router interface to act as the LAN gateway.
- Create a DHCP pool with a network range, default gateway, and DNS server.
- Exclude a reserved range of addresses from the DHCP pool.
- Set client PCs to obtain addressing via DHCP instead of static configuration.
- Verify automatic addressing using `ipconfig` on the client side.
- Verify DHCP server state using `show ip dhcp pool` and `show ip dhcp binding`.
- Observe DHCP lease behavior directly using `ipconfig /release` and `ipconfig /renew`.

---

## Network Topology

Three PCs connect to Router1 (the DHCP server) through a single switch.

![DHCP topology](DHCP%20topology.png)

| Device | Addressing | Assigned IP |
|---|---|---|
| PC1 | DHCP | 192.168.10.21 |
| PC2 | DHCP | 192.168.10.22 |
| PC3 | DHCP | 192.168.10.23 |
| Router1 (Gi0/0) | Static (gateway) | 192.168.10.1 |

---

## Router Interface Verification

Router1's LAN interface was verified as up and addressed before configuring the DHCP pool.

![Router Interface Verification](Router%20Interface%20Verification.png)

| Interface | IP Address | Status |
|---|---|---|
| GigabitEthernet0/0 | 192.168.10.1 | up/up |

---

## DHCP Pool Configuration

Router1 was configured as the DHCP server for the `192.168.10.0/24` network:

```text
ip dhcp excluded-address 192.168.10.1 192.168.10.20

ip dhcp pool LAN-USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

| Setting | Value |
|---|---|
| Pool name | LAN-USERS |
| Network | 192.168.10.0/24 |
| Default gateway handed to clients | 192.168.10.1 |
| DNS server handed to clients | 8.8.8.8 |
| Excluded range | 192.168.10.1 – 192.168.10.20 (reserved for static use, e.g. the gateway itself) |

Verified using `show ip dhcp pool`:

![DHCP Pool Configuration](DHCP%20Pool%20Configuration.png)

```text
Pool LAN-USERS :
  Total addresses       : 254
  Leased addresses      : 3
  Excluded addresses    : 1
  1 subnet is currently in the pool
```

> Only **1** address shows as excluded here rather than the full 20-address range — Packet Tracer's `show ip dhcp pool` counts the excluded *range* as a single pool-level deduction rather than listing each address, which is normal simulator behavior.

---

## PC IP Configuration (Client Side)

With PC1 set to DHCP, `ipconfig` confirms it received a full configuration automatically — no manual entry was made on the PC at all.

![PC IP Configuration](PC%20IP%20Configuration.png)

```text
IPv4 Address:     192.168.10.21
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.10.1
```

---

## DHCP Connectivity Test

PC1 successfully pinged its gateway (`192.168.10.1`) using the address it received entirely via DHCP.

![DHCP Connectivity Test](DHCP%20Connectivity%20Test.png)

Result: 4 of 4 replies received, **0% loss**, TTL = 255 — the maximum starting TTL, confirming this reply came directly from a locally attached router interface with no intermediate hop.

---

## DHCP Binding Table

`show ip dhcp binding` on Router1 confirms exactly which MAC address was leased which IP — the definitive server-side proof of DHCP activity.

![DHCP Binding Table](DHCP%20Binding%20Table.png)

| IP Address | Client MAC Address | Type |
|---|---|---|
| 192.168.10.21 | 0060.472D.39D4 | Automatic |
| 192.168.10.22 | 00E0.F942.7C74 | Automatic |
| 192.168.10.23 | 000C.8537.1D59 | Automatic |

All three leases are marked **Automatic**, confirming no static DHCP reservations were used — every address was dynamically assigned from the pool.

---

## DHCP Release / Renew

To directly observe the DHCP lease process, PC1's address was manually released and then renewed.

![DHCP release/renew](DHCP%20release%20renew.png)

```text
C:\>ipconfig /release
IP Address:      0.0.0.0
Subnet Mask:     0.0.0.0
Default Gateway: 0.0.0.0
DNS Server:      0.0.0.0

C:\>ipconfig /renew
IP Address:      192.168.10.21
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
DNS Server:      8.8.8.8
```

This confirms the full DHCP cycle: releasing a lease drops the PC back to an unconfigured state (`0.0.0.0` across the board), and renewing it triggers a fresh DHCP request that is answered with the exact same configuration — IP, mask, gateway, and DNS — straight from Router1's pool.

---

## Key Learning Outcomes

- A DHCP pool needs, at minimum, a network range, default gateway, and DNS server to hand out a complete client configuration.
- `ip dhcp excluded-address` keeps the pool from handing out addresses already reserved for static use (gateways, servers, infrastructure).
- `show ip dhcp binding` is the clearest server-side proof of DHCP activity — it ties specific MAC addresses to specific leased IPs.
- `ipconfig /release` followed by `/renew` is a hands-on way to directly observe the DHCP request/response cycle on the client, rather than only seeing the end result.
- DHCP removes manual IP configuration entirely from the client side, which is both convenient and — from a security standpoint — the reason DHCP Snooping exists: an unauthorized ("rogue") DHCP server could just as easily hand out malicious gateway or DNS settings to unsuspecting clients.
