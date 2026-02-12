# Transport Layer — Reliable End-to-End Delivery (OSI Layer 4)

---

## Big Picture

* **Sits between Session and Network layers**

  * Provides **end-to-end communication** between applications/processes across networks.
  * Ensures **data integrity, sequencing, and delivery guarantees**.

* **Abstracts network unreliability**

  * Network may drop, reorder, or duplicate packets.
  * Transport layer fixes this before delivering to the application.

* **Main goal:** Make communication **reliable or fast** depending on protocol.

---

# Core Responsibilities

* **Segmentation & Reassembly**

  * Splits large application data into manageable chunks (segments).
  * Reassembles segments at receiver in correct order.

* **Reliability & Error Recovery**

  * Retransmits lost segments.
  * Detects errors via checksums.

* **Flow Control**

  * Prevents sender from overwhelming receiver.
  * Ensures smooth transmission.

* **Multiplexing/Demultiplexing**

  * Uses **port numbers** to deliver data to correct application.
  * Example: port 80 → HTTP, port 443 → HTTPS.

* **Connection Management**

  * Establishes, maintains, terminates connections (TCP).

---

# Main Protocols

| Protocol                                    | Characteristics                        | Use Case                  |
| ------------------------------------------- | -------------------------------------- | ------------------------- |
| TCP (Transmission Control Protocol)         | Connection-oriented, reliable, ordered | Web, email, file transfer |
| UDP (User Datagram Protocol)                | Connectionless, unreliable, fast       | Streaming, gaming, DNS    |
| SCTP (Stream Control Transmission Protocol) | Reliable, multi-streaming              | Telephony, signaling      |

---

# Key Concepts

---

## 1️⃣ Segmentation

* Splits application data into **segments**.

* Adds **TCP/UDP header**:

  * Source port
  * Destination port
  * Sequence number (TCP)
  * Checksum

* Enables:

  * Reassembly at receiver.
  * Error detection and retransmission.

---

## 2️⃣ Reliable Delivery (TCP)

* **Acknowledgements (ACKs)**

  * Receiver confirms received segments.

* **Retransmission**

  * Lost or corrupted segments resent.

* **Sequence numbers**

  * Maintains order of delivery.

* **Flow control (Sliding Window)**

  * Sender adjusts rate based on receiver’s capacity.

---

## 3️⃣ Connection-Oriented vs Connectionless

| Feature     | TCP                           | UDP                     |
| ----------- | ----------------------------- | ----------------------- |
| Connection  | Established (3-way handshake) | None                    |
| Reliability | Guaranteed                    | Not guaranteed          |
| Ordering    | Ordered                       | May arrive out of order |
| Overhead    | High                          | Low                     |
| Use case    | HTTP, FTP, Email              | DNS, Video, Gaming      |

---

## 4️⃣ Ports & Multiplexing

* **Port numbers** identify applications on a host.

* **Well-known ports**: 80 (HTTP), 443 (HTTPS), 25 (SMTP)

* Enables multiple applications to use **same IP simultaneously**.

* Multiplexing process:

  * Sender tags data with port number → delivered to correct app.
  * Receiver reads port → sends to appropriate socket.

---

## 5️⃣ Connection Lifecycle (TCP)

* **3-Way Handshake (Establish)**

  1. SYN → client requests connection.
  2. SYN-ACK → server acknowledges and responds.
  3. ACK → client confirms.

* **Data Transfer Phase**

  * Segments exchanged reliably.

* **Termination (4-Way Handshake)**

  * FIN from one side, ACK, FIN from other, ACK → connection closed.

---

# Mental Model

* **Transport layer = post office for applications**

  * Splits packages (segmentation)
  * Labels with addresses (port numbers)
  * Ensures safe delivery (ACK/retransmit)
  * Reassembles on arrival (sequence numbers)

---

# Error Detection & Correction

* **Checksum**

  * Validates segment integrity.
  * If fails → segment dropped / retransmitted.

* **Timeouts**

  * Retransmit segment if no ACK received.

* **Duplicate Detection**

  * Sequence numbers prevent duplicate data.

---

# Flow Diagram

```
Application Data
      ↓
  Segmentation
      ↓
  TCP/UDP Header Added
      ↓
  Network Layer (Packet)
      ↓
  Data Link + Physical
```

At receiver:

```
Bits → Frames → Packets → Segments → Application Data
```

---

# Failure Scenarios

* Packet loss → TCP retransmits.
* Network congestion → TCP throttles sending rate.
* Out-of-order delivery → TCP reorders.
* UDP: data may be lost or corrupted → application must handle it.

---

# Performance Considerations

* TCP reliability → **higher latency**.
* UDP → **lower latency**, no reliability.
* Window size impacts throughput (sliding window algorithm).
* Segment size (MTU) impacts efficiency.

---

# Security Implications

* TCP hijacking → attacker spoofs sequence numbers.
* SYN flood → DoS attack exploiting handshake.
* UDP amplification → reflection attacks.

Mitigation:

* Firewalls, rate limiting, TLS (upper layer encryption).

---

# Real-World Example

* Opening [https://example.com](https://example.com):

  * Application → HTTP request.
  * Transport → TCP segments ensure reliable delivery.
  * Network → IP routes packets.
  * Application at server receives complete HTTP message.

* If using streaming video (UDP), lost packets may skip frames → acceptable.

---

# Comparison with OSI Neighbors

| Layer     | Focus                           |
| --------- | ------------------------------- |
| Session   | Maintain conversation context   |
| Transport | Deliver data reliably & orderly |
| Network   | Find path from host to host     |

* Transport guarantees **end-to-end delivery**.
* Network only guarantees **best-effort delivery between nodes**.

---

# Retrieval Prompts

* Why does TCP use a 3-way handshake?
* Why is UDP faster than TCP?
* How does flow control prevent congestion?
* What happens if sequence numbers wrap around?
* Compare TCP and UDP reliability and use cases.
* How does transport layer multiplex multiple applications over the same IP?

---

# Compressed Memory Snapshot

* Ensures **reliable, ordered delivery** (TCP) or **fast, best-effort delivery** (UDP).
* Splits/reassembles data into **segments**.
* Uses **ports** for multiplexing.
* Manages **connections** (TCP) and retransmissions.
* Shields application from network unreliability.
