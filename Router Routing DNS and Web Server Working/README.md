# Router Routing, DNS and Web Server Working

## Overview

This project demonstrates the practical working of a Cisco 2911 router in a multi-network environment using Cisco Packet Tracer.

The lab was designed to understand how a router connects different IPv4 networks and enables communication between devices located on separate networks.

In addition to basic routing, the lab demonstrates DNS name resolution and web server communication. A DNS record was configured for `www.hello.com`, which resolves to the remote web server at `20.0.0.1`.

The client was then able to access the web server using the hostname:

`http://www.hello.com`

The communication was further analyzed using Cisco Packet Tracer Simulation Mode and PDU/OSI inspection to understand how application data is transported through TCP, IP, and Ethernet layers.

---

## Objectives

The main objectives of this practical lab were:

- Understand the role of a router in connecting different IP networks.
- Configure router interfaces for different IPv4 networks.
- Verify router interface status and addressing.
- Understand Layer 3 packet forwarding.
- Inspect and understand the routing table.
- Configure a DNS server and hostname record.
- Configure a web server.
- Understand the relationship between DNS, routing, TCP and HTTP.
- Verify communication using a real browser request.
- Analyze packets using Cisco Packet Tracer Simulation Mode.
- Relate practical packet behavior to the OSI and TCP/IP models.
- Understand how Layer 2 and Layer 3 addressing work together during routed communication.

---

## Network Topology

The network was created and tested using Cisco Packet Tracer. The topology consists of two different IPv4 networks connected through a Cisco 2911 router.

### Network 1 — `10.1.1.0/24`

This network contains:

- PC
- Laptops
- DNS Server
- Switch0
- Router interface connected to the local network

### Network 2 — `20.0.0.0/24`

This network contains:

- Switch1
- Web Server
- Router interface connected to the remote network

### Logical Communication Flow

```text
                 10.1.1.0/24
                      |
       +--------------+--------------+
       |              |              |
      PC            Laptop        DNS Server
   10.1.1.1       10.1.1.2        10.1.1.60
       |              |              |
       +--------------+--------------+
                      |
                   Switch0
                      |
                Cisco 2911
                  Router0
                      |
                   Switch1
                      |
                 Web Server
                  20.0.0.1
                      |
                 20.0.0.0/24
```

The following topology screenshot was captured directly from Packet Tracer for visual reference.

![Router Topology](Router%20topology.png)

---

## Router Configuration and Interface Verification

Router interfaces were configured with appropriate IP addresses and enabled to establish communication between different networks.

![Router Configuration and Interface Verification](Router%20Configuration%20/Interface%20Verification.png)

---

## Routing Table

The routing table was inspected to verify that the router had the required network information for forwarding packets between networks.

<!-- This screenshot is optional since the routing table can also be shown as CLI text output.
     Enable it later by removing this comment block if you want the visual proof included:
![Routing Table](Routing-Table.png)
-->

---

## DNS Configuration

A DNS service was configured to resolve the custom domain:

`www.hello.com`

![Router DNS Configuration](Router%20DNS%20Configuration.png)

---

## HTTP and HTTPS Services

HTTP/HTTPS services were configured on the web server to simulate a basic real-world web service.

<!-- Currently disabled — remove this comment block to enable:
![HTTP and HTTPS Configuration](HTTP-and-HTTPS-Configuration.png)
-->

---

## Connectivity and Application Testing

Connectivity was verified by accessing the configured web service through the domain name:

`http://www.hello.com`

The successful webpage response demonstrates that DNS resolution, routing, and HTTP communication were functioning correctly.

![Router Connectivity Test](Router%20Connectivity%20Test.png)

---

## Packet Analysis – OSI Model

Cisco Packet Tracer's Simulation Mode was used to inspect how application traffic travels through the network. The PDU information was examined across the OSI layers, including TCP, IP, and Ethernet information.

![Router PDU OSI Details](Router%20PDU%20OSI%20details.png)
