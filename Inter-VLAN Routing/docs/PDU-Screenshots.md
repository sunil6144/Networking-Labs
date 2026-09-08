# PDU Screenshots — Inter-VLAN Routing Packet Walkthrough (PC1 → Laptop3)

The following screenshots trace a single ICMP echo request from **PC1 (192.168.10.10, VLAN 10)** to **Laptop3 (192.168.20.10, VLAN 20)**, hop by hop through Simulation Mode — showing exactly how the 802.1Q VLAN tag is added and removed as the packet crosses from one VLAN to another via Router-on-a-Stick.

---

## 1. PC1 — Creating the Request

PC1 builds the ICMP Echo Request. Since the destination (`192.168.20.10`) is not in its own subnet, PC1 sends the frame to its **default gateway** (`192.168.10.1`) instead of the destination directly.

![PC1 PDU Details](PC1%20PDU%20Details.png)

## 2. Switch0 — Tagging the Frame for the Trunk

The frame arrives untagged on the access port connected to PC1 (Fa0/2, VLAN 10). Switch0 adds an **802.1Q tag** for VLAN 10 before forwarding it out the trunk port (Fa0/1) toward Router0.

![Switch0-PDU-Details](Switch0-PDU-Details.png)

## 3. Router0 — Routing Between Subinterfaces

Router0 receives the tagged frame on its trunk interface (Gi0/0, subinterface .10), routes the packet at Layer 3 from the 192.168.10.0/24 network to the 192.168.20.0/24 network, and sends it back out the same physical interface — now re-tagged for VLAN 20 (subinterface .20).

![Router0 PDU details](Router0%20PDU%20details.png)

## 4. Switch0 — Untagging for the Destination VLAN

Switch0 receives the VLAN 20–tagged frame back from Router0 on the trunk port (Fa0/1) and **removes the tag** before forwarding it out the access port connected to Laptop3 (Fa0/4, VLAN 20) as a normal Ethernet II frame.

![Switch0 PDU Details](Switch0%20PDU%20Details.png)

## 5. Laptop3 — Final Delivery

The ICMP Echo Request arrives at Laptop3, completing the inter-VLAN journey. Laptop3 replies, and the same path is followed in reverse.

![Laptop3 PDU Details](Laptop3%20PDU%20Details.png)
