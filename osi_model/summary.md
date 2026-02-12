The ultimate **OSI Model Cheat Sheet** — all 7 layers in **one visual + table**, detailed yet concise, ready for brain absorption.

---

# OSI Model Cheat Sheet — 7 Layers

| Layer            | Layer # | Main Focus                   | Key Responsibilities                                                    | Protocols / Standards                            | Devices                            | Example                                 |
| ---------------- | ------- | ---------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------ | ---------------------------------- | --------------------------------------- |
| **Application**  | 7       | Interface for apps           | Provides network services to apps; defines message semantics            | HTTP, FTP, SMTP, DNS, POP3                       | None (app level)                   | Web browser, Email client               |
| **Presentation** | 6       | Data format & transformation | Encoding, serialization, encryption, compression                        | TLS, SSL, JPEG, MPEG, JSON, XML                  | None (software)                    | JSON.parse, gzip, HTTPS                 |
| **Session**      | 5       | Conversation management      | Establish, maintain, terminate sessions; checkpoints; token management  | NetBIOS, RPC, PPTP, SIP                          | None (software/middleware)         | Web login session, WebSocket            |
| **Transport**    | 4       | End-to-end delivery          | Segmentation/reassembly, reliability, flow control, ports, multiplexing | TCP, UDP, SCTP                                   | Host NIC, OS transport stack       | HTTP over TCP, Video streaming over UDP |
| **Network**      | 3       | Host-to-host delivery        | Logical addressing, routing, packet forwarding, fragmentation           | IPv4, IPv6, ICMP, IPsec, IGMP                    | Router, Layer 3 Switch             | Sending IP packet across Internet       |
| **Data Link**    | 2       | Node-to-node delivery        | Framing, MAC addressing, error detection, flow control, access control  | Ethernet (802.3), Wi-Fi (802.11), ARP, PPP, VLAN | Switch, Bridge, NIC, AP            | Ethernet frame delivery on LAN          |
| **Physical**     | 1       | Raw bit transmission         | Signals, bit encoding, topology, media, connectors, bit rate            | Ethernet physical, Wi-Fi PHY, Fiber optics, DSL  | Hub, Repeater, NIC, Cable, Antenna | Sending bits over copper/fiber/wireless |

---

# Quick Flow of Data (Sender → Receiver)

```
[Application Data]
      ↓
Application Layer (HTTP/FTP)
      ↓
Presentation Layer (Encoding, Encryption, Compression)
      ↓
Session Layer (Session ID, Tokens, Handshake)
      ↓
Transport Layer (TCP/UDP Segment, Port Numbers)
      ↓
Network Layer (IP Packet, Routing)
      ↓
Data Link Layer (Frame, MAC addresses, CRC)
      ↓
Physical Layer (Bits → Signals on Cable/Fiber/Air)
```

Receiver reverses the process step by step.

---

# Visual Mnemonic

```
7. Application      → User interface
6. Presentation     → Translator & protector
5. Session          → Conversation manager
4. Transport        → Post office & delivery guarantee
3. Network          → GPS & router
2. Data Link        → MAC addresses & error checking
1. Physical         → Bits on wire / signals
```

Think **“All People Seem To Need Data Processing”** — classic mnemonic.

---

# Quick Comparison: Responsibility Scope

| Layer        | Scope             | Reliability                         |
| ------------ | ----------------- | ----------------------------------- |
| Physical     | Bit-level         | None                                |
| Data Link    | Node-to-node      | Optional                            |
| Network      | Host-to-host      | Best effort                         |
| Transport    | End-to-end        | Reliable (TCP) or best-effort (UDP) |
| Session      | Conversation      | Stateful/session management         |
| Presentation | Format & security | Ensures compatibility               |
| Application  | User interface    | Depends on underlying layers        |

---

# Devices / Tools Cheat

| Layer        | Devices / Tools                    | Function                                |
| ------------ | ---------------------------------- | --------------------------------------- |
| Physical     | NIC, Hub, Repeater, Cable, Antenna | Raw bit transmission                    |
| Data Link    | Switch, Bridge, NIC, AP            | Node-to-node delivery                   |
| Network      | Router, Layer 3 switch             | Routing, host addressing                |
| Transport    | OS transport stack                 | Segmentation, ports, flow control       |
| Session      | Middleware, Web servers            | Session management, tokens              |
| Presentation | Software libraries                 | Encoding, encryption, compression       |
| Application  | Apps                               | User-facing services (HTTP, FTP, Email) |

---

# Quick Protocol Map

```
Application Layer: HTTP, FTP, DNS, SMTP
Presentation Layer: TLS, SSL, JSON, XML
Session Layer: NetBIOS, RPC, PPTP, SIP
Transport Layer: TCP, UDP, SCTP
Network Layer: IPv4, IPv6, ICMP, IPsec
Data Link Layer: Ethernet, Wi-Fi, ARP, VLAN
Physical Layer: Ethernet PHY, Fiber optics, Wi-Fi signals
```

---

# High-Level Mental Model

* **Physical:** Transmit raw bits → think “cable & signals”
* **Data Link:** Package bits → “local delivery & error check”
* **Network:** Address hosts → “GPS & routing”
* **Transport:** Deliver reliably → “post office & sequencing”
* **Session:** Maintain conversation → “call manager”
* **Presentation:** Make data usable → “translator & protector”
* **Application:** Interface for humans → “apps & services”

---

# Pro Tips for Engineers

* Always know **which layer a problem belongs to**: e.g.,

  * Cannot ping → Physical/Data Link
  * Wrong IP → Network
  * TCP handshake fails → Transport
  * HTTPS fails → Presentation
  * Session timeout → Session
  * App error → Application

* Remember **flow vs scope vs reliability** for each layer.