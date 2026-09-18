# Project 2 — Subnetting + Static Routing

## Overview

This project builds a small routed network in **Cisco Packet Tracer** to practice:

- IPv4 subnetting
- `/24` and `/30` networks
- Default gateways
- Router interfaces
- Routing tables
- Static routes
- ARP troubleshooting
- End-to-end connectivity
- Packet observation in Simulation Mode
- Breaking and fixing a working network

The project follows the learning workflow:

**BUILD → OBSERVE → EXPLAIN → BREAK → FIX → DOCUMENT**

---

## Learning Objectives

By the end of this project, I can:

1. Identify whether two IPv4 addresses are in the same subnet.
2. Understand the purpose of a subnet mask and CIDR notation.
3. Use a `/30` subnet for a point-to-point router link.
4. Configure router interfaces with IPv4 addresses.
5. Configure default gateways on PCs.
6. Read a router routing table.
7. Configure static routes.
8. Explain how a packet travels between two different LANs.
9. Use `ping`, `show ip route`, `show ip interface brief`, and `show arp` to troubleshoot connectivity.
10. Break a routed network intentionally and restore it.

---

# 1. Subnetting Practice

Before building the final routed topology, I practiced how subnetting changes network boundaries.

## `/24`

A `/24` network has:

- 256 total addresses
- 254 usable host addresses
- Mask: `255.255.255.0`

Example:

```text
192.168.1.0/24
```

Network:

```text
192.168.1.0
```

Usable hosts:

```text
192.168.1.1 - 192.168.1.254
```

Broadcast:

```text
192.168.1.255
```

## `/25`

A `/25` divides a `/24` into two subnets.

Mask:

```text
255.255.255.128
```

The two subnets are:

```text
192.168.1.0/25
    Network:   192.168.1.0
    Hosts:     192.168.1.1 - 192.168.1.126
    Broadcast: 192.168.1.127

192.168.1.128/25
    Network:   192.168.1.128
    Hosts:     192.168.1.129 - 192.168.1.254
    Broadcast: 192.168.1.255
```

### Practical test

Two hosts in the same `/25` subnet were able to communicate.

When one host was moved to a different `/25` subnet, the Layer-2 switch could no longer provide communication by itself.

This demonstrated:

> A switch does not perform Layer-3 routing between different IP subnets.

---

# 2. Final Topology

The final project uses two LANs connected through two routers.

```text
PC1 ── Switch1 ── R1 ───────── R2 ── Switch2 ── PC2
                         |
                   R1 ↔ R2 link
```

Detailed topology:

```text
PC1
192.168.10.10/24
     |
     |
  Switch1
     |
     | G0/1
     |
  R1 (2911)
     |
     | G0/1
     |
     | 10.0.0.0/30
     |
     | G0/0
     |
  R2
     |
     | G0/1
     |
  Switch2
     |
     |
PC2
192.168.20.10/24
```

## Physical Connections

| Device A | Interface | Device B | Interface |
|---|---|---|---|
| PC1 | FastEthernet0 | Switch1 | FastEthernet0/1 |
| Switch1 | GigabitEthernet0/1 | R1 | GigabitEthernet0/0 |
| R1 | GigabitEthernet0/1 | R2 | GigabitEthernet0/0 |
| R2 | GigabitEthernet0/1 | Switch2 | GigabitEthernet0/1 |
| Switch2 | FastEthernet0/1 | PC2 | FastEthernet0 |

---

# 3. IP Addressing Plan

The final design uses **three separate IPv4 networks**.

| Network | Purpose | CIDR | Mask |
|---|---|---|---|
| `192.168.10.0/24` | PC1 LAN | /24 | `255.255.255.0` |
| `10.0.0.0/30` | R1 ↔ R2 point-to-point link | /30 | `255.255.255.252` |
| `192.168.20.0/24` | PC2 LAN | /24 | `255.255.255.0` |

## Address Assignments

| Device | Interface | IP Address | Mask | Default Gateway |
|---|---|---|---|---|
| PC1 | FastEthernet0 | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` |
| R1 | G0/0 | `192.168.10.1` | `255.255.255.0` | — |
| R1 | G0/1 | `10.0.0.1` | `255.255.255.252` | — |
| R2 | G0/0 | `10.0.0.2` | `255.255.255.252` | — |
| R2 | G0/1 | `192.168.20.1` | `255.255.255.0` | — |
| PC2 | FastEthernet0 | `192.168.20.10` | `255.255.255.0` | `192.168.20.1` |

---

# 4. Why the R1 ↔ R2 Link Uses `/30`

The router-to-router connection only needs two usable IPv4 addresses.

For:

```text
10.0.0.0/30
```

the addresses are:

```text
10.0.0.0  = Network
10.0.0.1  = R1
10.0.0.2  = R2
10.0.0.3  = Broadcast
```

This gives exactly two usable host addresses for the two router interfaces.

---

# 5. R1 Configuration

R1 is the new Cisco 2911 router.

## R1 G0/0 — PC1 LAN

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

## R1 G0/1 — Router-to-Router Link

```text
exit
interface gigabitEthernet 0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
```

---

# 6. R2 Configuration

R2 is the original router used in the previous topology.

## R2 G0/0 — Router-to-Router Link

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
```

## R2 G0/1 — PC2 LAN

```text
exit
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
```

---

# 7. PC1 Configuration

PC1 was configured in Packet Tracer using:

**PC1 → Desktop → IP Configuration**

```text
IP Address:       192.168.10.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.10.1
```

### Local gateway test

From PC1:

```text
ping 192.168.10.1
```

Result:

```text
Successful
0% packet loss
```

This proved:

```text
PC1 → Switch1 → R1
```

was working.

---

# 8. PC2 Configuration

PC2 was configured using:

**PC2 → Desktop → IP Configuration**

```text
IP Address:       192.168.20.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.20.1
```

### Local gateway test

From PC2:

```text
ping 192.168.20.1
```

Result:

```text
Successful
```

This proved:

```text
PC2 → Switch2 → R2
```

was working.

---

# 9. Static Routing

At first, each router only knew about networks directly connected to its interfaces.

## R1 needed a route to the PC2 LAN

On R1:

```text
enable
configure terminal
ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
```

Meaning:

> To reach `192.168.20.0/24`, send the traffic to the next hop `10.0.0.2`.

## R2 needed a return route to the PC1 LAN

On R2:

```text
enable
configure terminal
ip route 192.168.10.0 255.255.255.0 10.0.0.1
end
```

Meaning:

> To reach `192.168.10.0/24`, send the traffic to the next hop `10.0.0.1`.

---

# 10. Routing Table Verification

On R1:

```text
show ip route
```

The routing table should contain a static route similar to:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

On R2:

```text
show ip route
```

The routing table should contain:

```text
S 192.168.10.0/24 [1/0] via 10.0.0.1
```

`S` means the route was configured as a **static route**.

The routers also show their directly connected networks with `C`.

---

# 11. Verification Commands

## Check Interface Status

On both routers:

```text
show ip interface brief
```

Expected final state:

### R1

```text
GigabitEthernet0/0   192.168.10.1   up   up
GigabitEthernet0/1   10.0.0.1       up   up
```

### R2

```text
GigabitEthernet0/0   10.0.0.2        up   up
GigabitEthernet0/1   192.168.20.1    up   up
```

`up / up` means the interface and its line protocol are operational.

---

# 12. End-to-End Test

From PC1:

```text
ping 192.168.20.10
```

The working path is:

```text
PC1
 ↓
Switch1
 ↓
R1
 ↓
R2
 ↓
Switch2
 ↓
PC2
```

Final result:

```text
Successful
0% packet loss
```

This confirmed end-to-end Layer-3 routing between two different LANs.

---

# 13. Observing the Packet in Simulation Mode

Packet Tracer Simulation Mode was used to observe the ICMP traffic.

The packet path was observed as:

```text
PC1 → Switch1 → R1 → R2 → Switch2 → PC2
```

The important router behavior is:

```text
Packet arrives at R1
        ↓
R1 looks at destination IP
        ↓
R1 checks its routing table
        ↓
Finds 192.168.20.0/24 via 10.0.0.2
        ↓
R1 forwards the packet toward R2
```

R2 then sees that:

```text
192.168.20.0/24
```

is directly connected to its G0/1 interface, so it forwards the traffic toward PC2.

---

# 14. BREAK + FIX Exercise

## Break 1 — Incorrect Default Gateway

PC2's default gateway was deliberately changed from:

```text
192.168.20.1
```

to another address.

An initial test using `192.168.1.1` did not produce the expected failure in Packet Tracer, so a second deliberate break was used.

The gateway was changed to:

```text
192.168.1.200
```

Then PC1 tested:

```text
ping 192.168.20.10
```

Result:

```text
Request timed out
```

The gateway was restored to:

```text
192.168.20.1
```

and connectivity was restored.

### Lesson

The default gateway must provide a path out of the local subnet. A wrong or unreachable gateway can prevent successful return traffic.

---

# 15. BREAK 2 — Remove the Static Route from R1

The R1 static route was removed:

```text
enable
configure terminal
no ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
```

Then:

```text
PC1> ping 192.168.20.10
```

The ping failed.

### Why?

The physical topology was still connected:

```text
PC1 → SW1 → R1 → R2 → SW2 → PC2
```

But R1 no longer had a route telling it where to send traffic for:

```text
192.168.20.0/24
```

The key lesson was:

```text
A working cable does not automatically create a route.
```

The route was restored with:

```text
enable
configure terminal
ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
```

Connectivity then returned.

---

# 16. ARP Troubleshooting

During troubleshooting, the R2 ARP table was checked:

```text
show arp
```

At one point the table showed:

```text
192.168.20.130   0002.17D4.1BC6   ARPA   GigabitEthernet0/1
```

while the test was being performed against:

```text
192.168.20.10
```

This revealed that PC2 was still configured with the old address:

```text
192.168.20.130
```

PC2 was corrected to:

```text
192.168.20.10
```

After that, R2 could reach PC2:

```text
ping 192.168.20.10
```

The first test showed:

```text
Success rate is 80 percent (4/5)
```

with the later packets succeeding. This was observed during the ARP resolution stage in Packet Tracer.

The full PC1 → PC2 test then worked.

### Lesson

When troubleshooting:

1. Verify the destination IP.
2. Check the endpoint configuration.
3. Check ARP.
4. Check interfaces.
5. Check the routing table.
6. Test each hop separately.
7. Only then change configuration.

---

# 17. Troubleshooting Path Used

The troubleshooting process for the failed end-to-end ping was:

```text
PC1 → PC2
   ↓
Failed
   ↓
PC1 → R2 G0/0 (10.0.0.2)
   ↓
Successful
   ↓
PC1 → R2 G0/1 (192.168.20.1)
   ↓
Successful
   ↓
R2 → PC2
   ↓
Failed
   ↓
show ip interface brief
   ↓
R2 interfaces = up/up
   ↓
show arp
   ↓
Found PC2 had 192.168.20.130
   ↓
Corrected PC2 to 192.168.20.10
   ↓
R2 → PC2 successful
   ↓
PC1 → PC2 successful
```

This was an important practical troubleshooting exercise because the problem was isolated by testing one segment at a time.

---

# 18. Key Concepts Learned

## Subnet

A subnet is a defined IP network determined by an IP address and subnet mask/prefix.

## Default Gateway

A default gateway is the router interface a host uses to reach destinations outside its local subnet.

Example:

```text
PC1 gateway = 192.168.10.1
PC2 gateway = 192.168.20.1
```

## Routing

Routing is the process of deciding where an IP packet should be forwarded based on its destination IP and routing table.

## Static Route

A static route is manually configured by an administrator.

Example:

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## Directly Connected Route

A router automatically knows about networks attached directly to its active interfaces.

## ARP

ARP maps an IPv4 address to a MAC address on the local network.

Example:

```text
IPv4 address → MAC address
```

## Switching vs Routing

A simplified distinction:

```text
Switch:
MAC address → local switch port

Router:
Destination IP/network → next hop/interface
```

---

# 19. Final Configuration Summary

## PC1

```text
IP:       192.168.10.10
Mask:     255.255.255.0
Gateway:  192.168.10.1
```

## R1

```text
G0/0 = 192.168.10.1/24
G0/1 = 10.0.0.1/30
```

Static route:

```text
192.168.20.0/24 via 10.0.0.2
```

## R2

```text
G0/0 = 10.0.0.2/30
G0/1 = 192.168.20.1/24
```

Static route:

```text
192.168.10.0/24 via 10.0.0.1
```

## PC2

```text
IP:       192.168.20.10
Mask:     255.255.255.0
Gateway:  192.168.20.1
```

---

# 20. Evidence / Screenshots

Recommended screenshots for this project:

```text
screenshots/
├── topology.png
├── pc1-ip-config.png
├── pc2-ip-config.png
├── r1-interface-status.png
├── r2-interface-status.png
├── r1-routing-table.png
├── r2-routing-table.png
├── pc1-to-pc2-ping-success.png
├── simulation-icmp-path.png
├── broken-static-route.png
└── arp-troubleshooting.png
```

Only include screenshots that clearly demonstrate the configuration or troubleshooting result.

---

# 21. Files

Recommended project folder:

```text
02-subnetting/
├── subnetting-routing.pkt
├── screenshots/
└── README.md
```

The `.pkt` file contains the Packet Tracer topology and configuration.

Do not place credentials, passwords, private keys, or other secrets in the repository.

---

# 22. Project Result

The final lab successfully demonstrated:

```text
Two LANs
   ↓
Different IP networks
   ↓
Two routers
   ↓
A /30 point-to-point link
   ↓
Static routing
   ↓
Successful end-to-end ICMP communication
```

Final connectivity:

```text
192.168.10.10
      |
      v
192.168.10.1
      |
10.0.0.1
      |
10.0.0.2
      |
192.168.20.1
      |
      v
192.168.20.10

✅ End-to-end ping successful
✅ 0% packet loss
```

---

# 23. What This Project Demonstrates for a SOC Learning Path

This project is useful as a networking foundation because it provides practical experience with:

- IPv4 addressing
- Subnet boundaries
- Gateways
- Routing tables
- Static routes
- ARP
- ICMP
- Packet paths
- Network troubleshooting
- Simulation and packet observation

The main practical lesson is:

> **When traffic fails, isolate the path one hop at a time instead of changing configuration randomly.**

---

## Status

**Project 2 — Subnetting + Static Routing**

**Status: Completed ✅**

Learning workflow completed:

```text
BUILD ✅
OBSERVE ✅
EXPLAIN ✅
BREAK ✅
FIX ✅
DOCUMENT ✅
```
