# Project 1 — First LAN: ARP, Ethernet Frames, MAC Learning & ICMP

## 1. Objective

Build a basic Local Area Network (LAN) using Cisco Packet Tracer and understand how devices communicate using IPv4, ARP, MAC addresses, Ethernet frames, a network switch, and ICMP.

The original lab started with two PCs and was later expanded to three PCs to observe how a switch learns multiple MAC addresses and forwards traffic.

---

## 2. Topology

```
             ┌─────────┐
PC1 ─────────│         │──────── PC2
             │  Switch │
PC3 ─────────│         │
             └─────────┘
```

**Devices**
- 3 × PCs
- 1 × Cisco 2960 switch
- Copper straight-through Ethernet cables

---

## 3. IP Configuration

| Device | IPv4 Address | Subnet Mask | Default Gateway |
|--------|--------------|-------------|------------------|
| PC1 | 192.168.1.1 | 255.255.255.0 | Not configured |
| PC2 | 192.168.1.2 | 255.255.255.0 | Not configured |
| PC3 | 192.168.1.3 | 255.255.255.0 | Not configured |

All three PCs were configured within the same IPv4 subnet.

---

## 4. ICMP Ping Test

PC1 was used to ping PC2.

The ping was successful and PC2 returned ICMP Echo Replies.

This demonstrated that PC1 and PC2 could successfully communicate across the local network.

The basic ICMP communication was:

```
PC1
  |
  | ICMP Echo Request
  ↓
PC2
  |
  | ICMP Echo Reply
  ↓
PC1
```

---

## 5. ARP Investigation

ARP (Address Resolution Protocol) was observed using Cisco Packet Tracer Simulation Mode.

ARP is used to discover the MAC address associated with an IPv4 address on the local network.

The basic relationship is:

```
IPv4 address → MAC address
```

For example, when PC1 wanted to communicate with PC2:

```
PC1 wants:
192.168.1.2

PC1 needs to discover:
PC2's MAC address
```

PC1 sends an ARP Request as a broadcast:

> "Who has 192.168.1.2?"

The device that owns that IPv4 address responds with its MAC address.

**ARP process**

```
PC1
 |
 | ARP Request (Broadcast)
 | "Who has 192.168.1.2?"
 ↓
Switch
 |
 | Broadcast forwarded within the LAN
 ↓
PC2
 |
 | ARP Reply (Unicast)
 | "192.168.1.2 is at my MAC address"
 ↓
PC1
```

This allows PC1 to learn the MAC address needed for local communication.

---

## 6. Ethernet Frames

An Ethernet frame is a Layer 2 container used to carry data across an Ethernet network.

A simplified Ethernet frame contains:

```
+----------------------------------+
| Destination MAC Address          |
| Source MAC Address               |
|                                   |
|          Data / Payload          |
|                                   |
+----------------------------------+
```

The destination MAC address is important because the switch uses it to determine where the frame should be forwarded.

The basic concept is:

```
ARP
IPv4 address → MAC address

        ↓

Ethernet Frame
Carries the Layer 2 information and data

        ↓

Switch
MAC address → Switch port

        ↓

Destination device
```

---

## 7. Switch MAC Address Learning

The switch was inspected using its MAC address table.

After network traffic was generated, the switch learned the MAC addresses of connected devices.

The switch maintains a table that associates MAC addresses with switch ports.

Conceptually:

```
MAC address → Switch port
```

For example:

```
PC1 MAC → Fa0/1
PC2 MAC → Fa0/2
PC3 MAC → Fa0/3
```

The exact port depends on how the devices were connected in Packet Tracer.

When an Ethernet frame arrives, the switch examines the destination MAC address and uses its MAC address table to determine the appropriate outgoing port.

---

## 8. Broadcast vs Unicast

This lab demonstrated both broadcast and unicast communication.

**Broadcast**

ARP Requests are broadcast on the local network.

Conceptually:

```
One device → All devices in the local broadcast domain
```

Example:

> "Who has 192.168.1.2?"

**Unicast**

The ARP Reply is sent back to the requesting device.

Normal communication between two known devices can also be unicast.

Conceptually:

```
One device → One destination device
```

---

## 9. Important Networking Relationships

This project helped demonstrate the relationship between several networking concepts.

- **ARP** — IPv4 address → MAC address
- **Ethernet** — Carries Layer 2 frames
- **Switch** — MAC address → Switch port
- **ICMP** — Used by tools such as ping to test IP connectivity

---

## 10. Layer Understanding

This project connected several concepts from the OSI model.

**Layer 2 — Data Link**

Examples observed:
- MAC addresses
- Ethernet frames
- Switches
- ARP-related local network communication

**Layer 3 — Network**

Examples observed:
- IPv4 addresses
- IP communication
- ICMP

The important idea is that different networking layers perform different jobs.

---

## 11. Security Relevance

Understanding normal network communication is important for a SOC analyst.

A SOC analyst needs to understand normal ARP and Ethernet behavior before investigating suspicious network activity.

The concepts learned in this project provide a foundation for investigating issues such as:

- ARP spoofing
- ARP poisoning
- Unexpected MAC address changes
- Network scanning
- Unusual broadcast activity
- Suspicious network traffic

These security topics will be investigated in later projects using more appropriate tools and lab environments.

---

## 12. What I Learned

The main concepts I learned from this project were:

- An IPv4 address identifies a device/interface at Layer 3.
- ARP can be used to discover the MAC address associated with an IPv4 address on the local network.
- ARP Requests are broadcast within the local network.
- Ethernet frames carry Layer 2 information and data.
- Ethernet frames contain source and destination MAC addresses.
- A switch learns MAC addresses and associates them with switch ports.
- A switch uses the destination MAC address to make forwarding decisions.
- ICMP Echo Requests and Echo Replies are used by ping to test IP connectivity.
- Broadcast and unicast traffic behave differently.
- Layer 2 and Layer 3 perform different functions during network communication.

---

## 13. Tools Used

- Cisco Packet Tracer

---

## 14. Lab File

The Packet Tracer topology is included in this directory:

```
first-lan.pkt
```

---

## 15. Future Improvements

This project is the foundation for more advanced networking and security labs.

Future projects will investigate:

- IPv4 subnetting
- CIDR
- VLANs
- 802.1Q trunking
- Inter-VLAN routing
- Static routing
- Dynamic routing
- DHCP
- NAT
- TCP and UDP
- DNS
- HTTP/HTTPS
- Wireshark packet analysis
- Network security monitoring
- Network attacks and indicators
