# Data Link Layer — Reliable Node-to-Node Delivery (OSI Layer 2)

---

## Big Picture

* **Sits between Network and Physical layers**

  * Responsible for **moving data between devices on the same local network (LAN or WAN segment)**.
  * Ensures that packets from the Network layer are **correctly packaged, addressed, and error-checked** for delivery on the local link.

* **Focus:** Node-to-node reliability, framing, and addressing.

* Provides foundation for **higher-level end-to-end communication**.

---

# Core Responsibilities

* **Framing**

  * Converts packets from Network layer into **frames** for transmission.
  * Adds headers and trailers containing addressing and error-checking info.

* **Physical Addressing (MAC)**

  * Assigns **hardware addresses** (MAC) to devices on the same network.
  * Ensures correct delivery to the intended device on LAN.

* **Error Detection & Handling**

  * Checks for transmission errors using checksums, CRC, parity bits.
  * May request retransmission in reliable links.

* **Flow Control (optional)**

  * Prevents a fast sender from overwhelming a slow receiver on the same link.

* **Access Control**

  * Determines **who can transmit and when** on shared media (Ethernet, Wi-Fi).

---

# Protocols & Standards

| Protocol / Standard               | Purpose              | Example               |
| --------------------------------- | -------------------- | --------------------- |
| Ethernet (IEEE 802.3)             | Wired LAN framing    | Most office networks  |
| Wi-Fi (IEEE 802.11)               | Wireless LAN framing | Home/enterprise Wi-Fi |
| PPP (Point-to-Point Protocol)     | Direct links         | DSL, VPN tunnels      |
| ARP (Address Resolution Protocol) | Maps IP → MAC        | Ethernet networks     |
| VLAN tagging (IEEE 802.1Q)        | Network segmentation | Enterprise networks   |

---

# Key Concepts

---

## 1️⃣ Frames

* Basic unit at Data Link layer.
* Structure:

```
[Header][Payload][Trailer]
```

* Header:

  * Source MAC
  * Destination MAC
  * VLAN tag (optional)

* Payload:

  * Network layer packet (IP)

* Trailer:

  * CRC / checksum

* **Purpose:** Encapsulation and error detection.

---

## 2️⃣ MAC Addressing

* **Media Access Control (MAC)** → unique hardware address.
* Example: `00:1A:2B:3C:4D:5E`
* Ensures device-level delivery **within same network segment**.

---

## 3️⃣ Error Detection

* **Cyclic Redundancy Check (CRC)**

  * Detects bit errors in transmission.
  * Receiver recalculates CRC → if mismatch, frame discarded.

* **Parity / Checksum**

  * Simpler error detection in older protocols.

* **Optional retransmission**

  * Some protocols (PPP) request retransmission.
  * Ethernet typically relies on higher layers (Transport) for recovery.

---

## 4️⃣ Flow & Access Control

* **Flow Control:** Stops fast sender from overwhelming receiver.
* **Media Access Control (MAC) protocols):**

  * Ethernet → CSMA/CD (Carrier Sense Multiple Access with Collision Detection)
  * Wi-Fi → CSMA/CA (Collision Avoidance)

---

## 5️⃣ Node-to-Node vs End-to-End

* **Data Link layer:** Ensures **node-to-node** delivery.
* **Transport layer:** Ensures **end-to-end** reliability.
* Example:

  * Packet arrives at next router → Data Link layer of that hop handles delivery.
  * Transport layer ensures data ultimately reaches correct application.

---

# Devices Operating at Data Link Layer

| Device                       | Function                               |
| ---------------------------- | -------------------------------------- |
| Switch                       | Forwards frames using MAC addresses    |
| Bridge                       | Connects LAN segments, filters frames  |
| NIC (Network Interface Card) | Sends/receives frames for host         |
| Wireless Access Point        | Wireless frame delivery and management |

---

# Flow Diagram (Node-to-Node)

```
Network Packet (IP)
        ↓
Data Link Layer → Frame (Ethernet/Wi-Fi)
        ↓
Physical Layer → Bits on wire
```

At receiver:

* Bits → Frames → Packet → Transport

---

# Failure Scenarios

* **Frame collision** → Ethernet handles via backoff algorithms.
* **CRC mismatch** → corrupted frame discarded.
* **Wrong MAC address** → frame ignored.
* **Buffer overflow in switch** → dropped frames.

---

# Performance Considerations

* Frame size affects efficiency (MTU).
* Collision domains limit throughput in shared media.
* VLANs can improve segmentation and reduce broadcast traffic.
* Wi-Fi: channel interference reduces effective throughput.

---

# Security Implications

* MAC spoofing → attacker impersonates a device.
* ARP spoofing → redirect traffic maliciously.
* VLAN hopping → bypass network segmentation.
* Mitigation:

  * Port security on switches
  * Dynamic ARP inspection
  * Network segmentation

---

# Debugging Tips

* Ping works but network services fail → check MAC layer or switch configuration.
* Packet capture → verify correct source/destination MAC.
* CRC errors → physical layer issues.
* Collisions / retransmissions → congestion or faulty NIC.

---

# Comparison with Neighbor Layers

| Layer     | Responsibility                                            |
| --------- | --------------------------------------------------------- |
| Network   | Host-to-host routing and addressing (IP)                  |
| Data Link | Node-to-node delivery using MAC, error detection, framing |
| Physical  | Raw transmission of bits                                  |

* Data Link layer **bridges Network and Physical layers**, encapsulating IP packets into frames for local delivery.

---

# Retrieval Questions

* What is the difference between MAC and IP addresses?
* Why are CRCs important at this layer?
* How does Ethernet handle collisions?
* What happens if a switch receives a frame with an unknown destination MAC?
* How do VLANs improve security and traffic segmentation?

---

# Compressed Memory Snapshot

* Moves packets **between nodes on the same network segment**.
* Uses **frames** with MAC addresses.
* Detects errors via **CRC/checksum**.
* Controls **who can transmit** on shared media.
* Key protocols: Ethernet, Wi-Fi, PPP, ARP, VLAN tagging.
* Devices: switches, bridges, NICs, wireless APs.
