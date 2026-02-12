# Network Layer — Host-to-Host Delivery Across Networks (OSI Layer 3)

---

## Big Picture

* **Sits between Transport and Data Link layers**

  * Responsible for **delivering packets from source host to destination host**, possibly across multiple networks.
  * Abstracts away the details of local networks (LANs, WANs).

* **Key problem it solves**

  * How to move data **between devices not on the same network segment**.
  * Routing, addressing, fragmentation, and inter-network delivery.

* **Focuses on logical addressing, not physical delivery**

  * IP addresses identify devices across networks.
  * Routing protocols determine best path.

---

# Core Responsibilities

* **Logical Addressing**

  * Assigns unique addresses to hosts (IP addresses).
  * Ensures each device can be located on a network.

* **Routing**

  * Determines the path packets take through the network.
  * May traverse multiple routers and networks.

* **Packet Forwarding**

  * Sends packets from one network segment to another.
  * Devices like routers operate here.

* **Fragmentation & Reassembly**

  * Splits large packets into smaller ones to fit MTU of underlying networks.
  * Reassembles them at destination.

* **Error handling & congestion notification (optional)**

  * Some protocols (e.g., ICMP) provide feedback on delivery issues.

---

# Protocols at This Layer

| Protocol | Purpose                              | Example Use                 |
| -------- | ------------------------------------ | --------------------------- |
| IPv4     | Standard host addressing & routing   | Internet traffic            |
| IPv6     | Next-generation addressing           | Modern networks             |
| ICMP     | Network diagnostics / error messages | Ping, traceroute            |
| IGMP     | Multicast group management           | Streaming to multiple hosts |
| IPsec    | Security & encryption                | VPN tunnels                 |

---

# Key Concepts

---

## 1️⃣ Logical vs Physical Addressing

* **Logical address** = IP (host identification)
* **Physical address** = MAC (network interface identification)

Flow:

```
Packet: IP header + Payload
→ Encapsulated in Frame (Data Link layer) for local delivery
```

---

## 2️⃣ Routing

* **Purpose:** Deliver packets across networks.

* **Devices:** Routers.

* **Techniques:**

  * Static routing → manually configured.
  * Dynamic routing → uses algorithms (OSPF, BGP) to determine paths.

* **Routing tables** store paths for different IP ranges.

---

## 3️⃣ Packet Fragmentation

* Network layers have **MTU limits**.

* Large packets may be split into smaller fragments.

* Each fragment has:

  * Fragment ID
  * Offset
  * More-fragments flag

* Reassembly happens at the destination host.

---

## 4️⃣ Connectionless Communication

* Network layer is generally **connectionless**:

  * Each packet treated independently.
  * Packets may arrive out of order, be duplicated, or lost.
  * Transport layer handles reliability (TCP).

* Example: IP packets may take different routes to same destination.

---

# Mental Model

* **Think of the network layer as GPS + post office for packets:**

  * Assigns addresses (GPS coordinates) → IP addresses.
  * Decides best path → routing algorithms.
  * Delivers packets across multiple networks.

* Network layer **doesn’t care what’s inside the packet**.

  * Could be TCP segment, UDP datagram, or even ICMP message.

---

# Flow Diagram (Sender → Receiver)

```
Application Data
      ↓
Transport Segment (TCP/UDP)
      ↓
Network Layer → Packet with IP header
      ↓
Data Link → Frame with MAC addresses
      ↓
Physical → Bits on wire
```

At receiver:

* Reverse order → Frame → Packet → Segment → Application data.

---

# Devices Operating at Network Layer

| Device         | Function                               |
| -------------- | -------------------------------------- |
| Router         | Routes packets between networks        |
| Layer 3 Switch | Switch + basic routing                 |
| Firewall       | Filters packets based on IP / protocol |

---

# Common Network Layer Problems

* **Routing loops** → packets circulate endlessly
* **IP conflicts** → duplicate IP addresses cause delivery failure
* **Fragmentation issues** → MTU mismatch
* **Packet loss** → dropped by intermediate router
* **TTL expiration** → packet discarded after exceeding hop count

---

# Security Considerations

* **IP spoofing** → attacker fakes source IP
* **DDoS attacks** → flood network with packets
* **Packet sniffing** → capture unencrypted data
* Mitigation:

  * Firewalls
  * Access control lists (ACLs)
  * IPsec for encryption

---

# Example: Sending a Packet Across Networks

1. Host A wants to send TCP segment to Host B.
2. Transport layer hands segment to Network layer.
3. Network layer adds **source & destination IP**.
4. Packet forwarded to router → router checks IP → chooses next hop.
5. Repeated until packet reaches destination network.
6. Destination host receives packet → passes to transport layer.

---

# Comparison with Neighbor Layers

| Layer     | Responsibility                            |
| --------- | ----------------------------------------- |
| Transport | End-to-end delivery (reliable/unreliable) |
| Network   | Host-to-host delivery across networks     |
| Data Link | Node-to-node delivery in same network     |

* Network ensures packet reaches **correct device**, transport ensures it reaches **correct application reliably**.

---

# Retrieval Questions

* Why does IP not guarantee delivery?
* How does TTL prevent infinite routing loops?
* What happens if two devices have the same IP?
* How do routers decide the best path?
* Compare IPv4 vs IPv6 in addressing & routing.
* Why is fragmentation necessary and what issues can it cause?

---

# Compressed Memory Snapshot

* Delivers packets **host-to-host** across networks.
* Uses **IP addresses** (logical addressing).
* Routes via **routers**, may fragment packets.
* Connectionless → reliability handled by transport layer.
* Protocols: IPv4, IPv6, ICMP, IGMP.
