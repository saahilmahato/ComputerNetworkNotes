# Physical Layer — Raw Bit Transmission (OSI Layer 1)

---

## Big Picture

* **Lowest layer of OSI model**

  * Deals with **transmitting raw bits over a physical medium**.
  * Provides the foundation for all higher-layer communication.

* **Focus:** Electrical, optical, or radio signals.

* **Key problem it solves:** How to send 0s and 1s **over wires, fiber, or air** reliably.

* **No concept of packets, frames, or addresses here**

  * Just raw **bit sequences**.
  * Error detection, addressing, and delivery are handled by higher layers.

---

# Core Responsibilities

* **Bit-level transmission**

  * Converts frames into **electrical, optical, or radio signals**.

* **Encoding / Signaling**

  * Represents 1s and 0s in a medium-specific format.
  * Examples:

    * Voltage levels for copper cables
    * Light pulses for fiber
    * Radio waves for wireless

* **Physical topology management**

  * Defines network layout: bus, star, ring, mesh.

* **Transmission medium definition**

  * Cable type (Ethernet twisted pair, coax, fiber)
  * Wireless spectrum (Wi-Fi, 5G)

* **Bit rate / bandwidth**

  * Determines **speed of data transmission**.
  * Measured in bps, Mbps, Gbps.

* **Connector / interface standardization**

  * RJ45, LC fiber connectors, antenna interfaces.

---

# Protocols & Standards

| Medium   | Standard                     | Notes                |
| -------- | ---------------------------- | -------------------- |
| Copper   | Ethernet (IEEE 802.3), DSL   | Twisted pair         |
| Fiber    | Fiber Channel, 100G Ethernet | Optical pulses       |
| Wireless | IEEE 802.11 (Wi-Fi), LTE, 5G | Radio frequencies    |
| Coax     | DOCSIS, Ethernet             | Cable TV / broadband |

---

# Key Concepts

---

## 1️⃣ Bit Encoding

* Represents digital data using physical signals.

* Examples:

  * **NRZ (Non-Return-to-Zero)** → high voltage = 1, low = 0
  * **Manchester Encoding** → transition in middle of bit period indicates value
  * **PAM, QAM** → multi-level signaling for high-speed links

* Purpose: Reliable detection over physical medium.

---

## 2️⃣ Transmission Mode

* **Simplex** → one direction only
* **Half-duplex** → both directions, but one at a time
* **Full-duplex** → simultaneous two-way communication

---

## 3️⃣ Physical Topology

* **Bus:** All devices share same medium

* **Star:** Central switch/hub

* **Ring:** Devices connected in a loop

* **Mesh:** Every device connected to multiple others

* Topology affects performance, reliability, and fault tolerance.

---

## 4️⃣ Medium Properties

* **Copper wires:** EMI susceptible, limited distance

* **Fiber optics:** High speed, long distance, immune to EMI

* **Wireless:** Flexible, interference-prone, bandwidth-limited

* Signal attenuation and noise limit physical performance.

---

## 5️⃣ Bit Rate & Bandwidth

* **Bit rate:** Number of bits transmitted per second

* **Bandwidth:** Max signal frequency that medium supports

* Shannon-Hartley theorem: Maximum data rate depends on bandwidth and signal-to-noise ratio.

---

# Devices Operating at Physical Layer

| Device                   | Function                                    |
| ------------------------ | ------------------------------------------- |
| Hub                      | Broadcasts bits to all ports (no filtering) |
| Repeater                 | Amplifies and regenerates signals           |
| NIC (Physical interface) | Converts bits to signals and vice versa     |
| Cables / Antennas        | Physical transmission medium                |

---

# Flow Diagram

```
Data Link Layer Frame
        ↓
Physical Layer → Bits → Signals
        ↓
Transmission Medium (copper/fiber/wireless)
```

At receiver:

```
Signals → Bits → Frame → Network Layer
```

---

# Failure Scenarios

* **Signal degradation / attenuation** → bits lost or corrupted
* **Noise / interference** → errors in bit detection
* **Cable breaks / faulty connectors** → communication failure
* **Wrong encoding** → receiver misinterprets bits

---

# Performance Considerations

* **Distance vs speed:** Longer cables → signal degradation → slower max speed
* **Medium choice:** Fiber > Copper > Wireless (speed & reliability)
* **Error rate:** High noise → more retransmissions (handled by higher layers)
* **Physical layout / topology** → collision domains in shared media

---

# Security Implications

* Physical access → total control of network (tapping wires, rogue APs)

* EMI or radio jamming → denial-of-service

* Fiber tapping → possible undetected interception

* Mitigation:

  * Secure physical access
  * Encryption at higher layers (TLS, IPsec)

---

# Debugging Tips

* Check cables, connectors, NICs
* Measure signal strength / SNR (wireless)
* Test bit error rate using tools like `iperf`
* Use loopback tests for local verification

---

# Comparison with Neighbor Layers

| Layer     | Responsibility                                |
| --------- | --------------------------------------------- |
| Data Link | Node-to-node delivery, framing, MAC addresses |
| Physical  | Bit-level transmission over physical medium   |

* Physical layer is the **foundation**; if it fails, nothing above works.

---

# Retrieval Questions

* How does bit encoding affect transmission reliability?
* Why is full-duplex faster than half-duplex?
* What happens if a cable is too long for its Ethernet standard?
* How does signal-to-noise ratio affect maximum data rate?
* Compare copper, fiber, and wireless for speed, reliability, and cost.

---

# Compressed Memory Snapshot

* Handles **raw transmission of 0s and 1s**.
* Converts frames → physical signals over medium.
* Defines **topology, connectors, and encoding**.
* Determines **bit rate and reliability**.
* Devices: hub, repeater, NIC, cables, antennas.
* Essential foundation for all higher-layer communication.
