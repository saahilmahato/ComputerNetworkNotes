# Packets in Computer Networks

## The Fundamental Question: How Should Data Travel?

Imagine you need to send a large file—say, a 100 MB video—across the Internet. You have two approaches:

**Approach 1: Send it all at once (Circuit Switching)**
```
Your computer ────────────────────────────────────→ Destination
               [100 MB continuous stream]

Problems:
  • Ties up the entire path for minutes
  • One broken link = entire transfer fails
  • Other users blocked while you transmit
  • Inefficient use of network resources
```

**Approach 2: Break into small pieces (Packet Switching)**
```
Your computer → [Packet 1] → [Packet 2] → [Packet 3] → ... → Destination
                    ↓            ↓            ↓
             Different paths can be used
             Other users share the same links
             Failures can be routed around

Benefits:
  ✓ Efficient link utilization
  ✓ Multiple users share network
  ✓ Resilient to failures
  ✓ Flexible routing
```

**This is why the Internet uses packet switching.**

---

## What is a Packet?

### The Definition

A **packet** is a formatted unit of data that contains:

1. **Header** — Control information (addresses, protocol info, metadata)
2. **Payload** — The actual data being transmitted
3. **Trailer** (optional) — Error detection information

**Visual structure:**

```
┌────────────┬──────────────────────────┬────────────┐
│   Header   │        Payload           │  Trailer   │
│ (metadata) │    (actual data)         │ (checksum) │
└────────────┴──────────────────────────┴────────────┘
```

---

### The Postal Mail Analogy

Think of sending a book by mail:

**Without packets (send entire book):**
```
Problem: Book is too heavy for one package

Mail carrier's issues:
  • Can't lift it
  • Too big for mailbox
  • If lost, entire book is gone
```

**With packets (break book into chapters):**
```
Solution: Split into envelopes

Each envelope contains:
  • Return address (source)
  • Destination address
  • Chapter number (sequence)
  • Pages (payload)
  • Seal/stamp (error detection)

Benefits:
  • Each envelope travels independently
  • Can take different routes
  • If one lost, retransmit just that chapter
  • Multiple carriers can work in parallel
```

**Packets work the same way for digital data.**

---

## Why Packet Switching Won

### Circuit Switching (The Old Way)

**How telephone networks worked:**

```
Call setup:
  1. Establish dedicated circuit from caller to receiver
  2. Reserve bandwidth end-to-end
  3. Maintain connection for entire call
  4. Tear down circuit when done

Example: 1960s phone call
  Your phone ──[Switch A]──[Switch B]──[Switch C]── Friend's phone
              └────────────────────────────────────┘
                    Dedicated circuit
```

**Problems:**

```
1. Wasteful:
   • Circuit reserved even during silence
   • Typical phone call: 60% silence
   • 60% of bandwidth wasted

2. Doesn't scale:
   • Need circuit for every simultaneous conversation
   • 1 million users → need 1 million circuits

3. Fragile:
   • One broken link → call drops
   • Can't reroute mid-call

4. Inflexible:
   • Same bandwidth allocated regardless of need
   • Can't share resources
```

---

### Packet Switching (The Internet Way)

**How the Internet works:**

```
Data transmission:
  1. Break data into packets
  2. Each packet routed independently
  3. No dedicated path
  4. Reassemble at destination

Example: Sending email
  Email → [P1] [P2] [P3] [P4]
             ↓     ↓     ↓     ↓
          Different paths possible
             ↓     ↓     ↓     ↓
          Reassemble → Complete email
```

**Advantages:**

```
1. Efficient:
   • Links shared among many users
   • Statistical multiplexing
   • 90%+ utilization possible

2. Scalable:
   • No per-connection state in network core
   • Routers just forward packets
   • Billions of devices supported

3. Resilient:
   • Packets routed around failures
   • Automatic rerouting
   • No single point of failure

4. Flexible:
   • Bandwidth allocated on demand
   • Priority for important traffic
   • Works over any physical medium
```

**Trade-off:**

```
Circuit switching:
  ✓ Predictable latency
  ✓ Guaranteed bandwidth
  ✗ Wasteful
  ✗ Doesn't scale

Packet switching:
  ✓ Efficient
  ✓ Scalable
  ✗ Variable latency
  ✗ No guarantees (best-effort)

Internet chose scalability over guarantees
```

---

## Terminology: Message vs Packet vs Frame vs Segment

**These terms refer to the same data at different layers:**

```
┌─────────────────────────────────────────────────┐
│          Application Layer                      │
│          Creates: MESSAGE                       │
│          (e.g., HTTP request, email)            │
└──────────────────────┬──────────────────────────┘
                       ↓ Passed down
┌─────────────────────────────────────────────────┐
│          Transport Layer (TCP/UDP)              │
│          Creates: SEGMENT (TCP) or DATAGRAM (UDP)│
│          Adds: Port numbers, sequence numbers   │
└──────────────────────┬──────────────────────────┘
                       ↓ Passed down
┌─────────────────────────────────────────────────┐
│          Network Layer (IP)                     │
│          Creates: PACKET                        │
│          Adds: IP addresses, TTL                │
└──────────────────────┬──────────────────────────┘
                       ↓ Passed down
┌─────────────────────────────────────────────────┐
│          Data Link Layer (Ethernet)             │
│          Creates: FRAME                         │
│          Adds: MAC addresses, CRC               │
└─────────────────────────────────────────────────┘
                       ↓
                Physical transmission
```

**Summary:**

| Term | Layer | What it is |
|------|-------|------------|
| **Message** | Application | Raw data (email, web page) |
| **Segment** | Transport (TCP) | Message + TCP header |
| **Datagram** | Transport (UDP) | Message + UDP header |
| **Packet** | Network (IP) | Segment/Datagram + IP header |
| **Frame** | Data Link | Packet + Ethernet header + trailer |

**Common usage:** "Packet" often refers to any of these generically.

---

## Encapsulation: Wrapping Data Layer by Layer

As data moves down the protocol stack, each layer adds its own header:

**Example: Sending "Hello" via HTTP**

```
Application Layer:
┌───────────────────┐
│  "Hello" (5 bytes)│
└───────────────────┘

Transport Layer (TCP adds 20-byte header):
┌──────────┬───────────────────┐
│TCP Header│  "Hello"          │
│ 20 bytes │  5 bytes          │
└──────────┴───────────────────┘
Total: 25 bytes

Network Layer (IP adds 20-byte header):
┌──────────┬──────────┬───────────────────┐
│IP Header │TCP Header│  "Hello"          │
│ 20 bytes │ 20 bytes │  5 bytes          │
└──────────┴──────────┴───────────────────┘
Total: 45 bytes

Data Link Layer (Ethernet adds 14-byte header + 4-byte trailer):
┌─────────┬──────────┬──────────┬───────────┬─────────┐
│Ethernet │IP Header │TCP Header│  "Hello"  │Ethernet │
│ Header  │ 20 bytes │ 20 bytes │  5 bytes  │ Trailer │
│14 bytes │          │          │           │ 4 bytes │
└─────────┴──────────┴──────────┴───────────┴─────────┘
Total: 63 bytes

Overhead: 58 bytes of headers for 5 bytes of data (92% overhead!)
```

**At the receiver, headers are stripped off (de-encapsulation) as data moves up the stack.**

---

## IP Packet Structure (IPv4)

### Complete Packet Anatomy

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───────┬───────┬───────────────┬───────────────────────────────┐
│Version│  IHL  │   Type of     │        Total Length           │
│ (4)   │ (5)   │   Service     │      (in bytes)               │
├───────────────────────────────┼─────────────────┬─────────────┤
│       Identification          │Flags│  Fragment Offset        │
│                               │     │                         │
├───────────────┬───────────────┼─────────────────────────────┬─┤
│  Time to Live │   Protocol    │    Header Checksum          │
│     (TTL)     │  (TCP/UDP/...)│                             │
├───────────────┴───────────────┴─────────────────────────────┴─┤
│                    Source IP Address                          │
│                    (32 bits)                                  │
├───────────────────────────────────────────────────────────────┤
│                    Destination IP Address                     │
│                    (32 bits)                                  │
├───────────────────────────────────────────────────────────────┤
│                    Options (if any)                           │
├───────────────────────────────────────────────────────────────┤
│                    Payload (Data)                             │
│                    ...                                        │
└───────────────────────────────────────────────────────────────┘

Minimum header size: 20 bytes (without options)
Maximum header size: 60 bytes (with options)
```

---

### Key Fields Explained

**Version (4 bits):**
```
Value: 4 (for IPv4) or 6 (for IPv6)
Purpose: Tells router which protocol version
```

**Total Length (16 bits):**
```
Range: 0 - 65,535 bytes
Includes: Header + Payload
Example: 1500-byte Ethernet packet → Total Length = 1500
```

**Identification (16 bits):**
```
Purpose: Unique ID for each packet
Used for: Reassembling fragments
Example: Original packet ID = 12345
         All fragments have ID = 12345
```

**Flags (3 bits):**
```
Bit 0: Reserved (must be 0)
Bit 1: Don't Fragment (DF)
  • If set: Router must not fragment
  • If packet too large: Router drops it, sends ICMP error
Bit 2: More Fragments (MF)
  • If set: More fragments follow
  • If clear: This is the last (or only) fragment
```

**Fragment Offset (13 bits):**
```
Purpose: Position of this fragment in original packet
Unit: 8-byte blocks
Example: Fragment 1: Offset = 0 (bytes 0-1479)
         Fragment 2: Offset = 185 (bytes 1480-2959)
                              ↑ (1480 / 8 = 185)
```

**Time To Live (TTL) (8 bits):**
```
Purpose: Prevent infinite routing loops
Initial value: Typically 64 or 128
Action: Decremented by 1 at each router
When TTL = 0: Packet dropped, ICMP "Time Exceeded" sent back

Example trace:
  Source (TTL=64) → Router A (TTL=63) → Router B (TTL=62) → ... → Destination

This is how 'traceroute' works!
```

**Protocol (8 bits):**
```
Indicates what's inside the payload:
  1 = ICMP (ping, errors)
  6 = TCP (web, email, SSH)
  17 = UDP (DNS, video streaming)
  41 = IPv6 (IPv6-in-IPv4 tunnel)
```

**Header Checksum (16 bits):**
```
Purpose: Detect corruption in the IP header
Calculated: Sum of all 16-bit words in header
Note: Only protects header, not payload
      (Payload protected by higher layers: TCP, Ethernet CRC)

Why it exists: Detect errors in routing info
If checksum fails: Packet dropped
```

**Source/Destination IP (32 bits each):**
```
The core addressing information
Source: 192.168.1.10 → 11000000.10101000.00000001.00001010
Destination: 8.8.8.8 → 00001000.00001000.00001000.00001000
```

---

## Fragmentation: When Packets Are Too Large

### The Problem: Maximum Transmission Unit (MTU)

**Every link has an MTU—the maximum packet size it can carry:**

```
Common MTUs:
  Ethernet: 1500 bytes
  PPPoE (DSL): 1492 bytes
  IPv4 minimum: 576 bytes
  IPv6 minimum: 1280 bytes
  Jumbo frames: 9000 bytes (data centers)
```

**What if a packet is larger than the link's MTU?**

```
Scenario:
  Router receives 3000-byte packet
  Outgoing link MTU = 1500 bytes
  
Problem: Packet doesn't fit!

Solutions:
  1. Fragment the packet (IPv4)
  2. Drop packet, send error (IPv6, or IPv4 with DF flag)
```

---

### Fragmentation Process (IPv4)

**Original packet:**
```
┌──────────────────────────────────────────────────────┐
│ IP Header │         Payload (2960 bytes)            │
│ 20 bytes  │                                         │
└──────────────────────────────────────────────────────┘
Total size: 2980 bytes

MTU = 1500 bytes → Must fragment
```

**After fragmentation:**
```
Fragment 1:
┌──────────────────────────────────────────────────┐
│ IP Header │     Payload (1480 bytes)             │
│ ID=100    │     Data: bytes 0-1479               │
│ Offset=0  │                                      │
│ MF=1      │                                      │
└──────────────────────────────────────────────────┘
Size: 1500 bytes (fits MTU)

Fragment 2:
┌──────────────────────────────────────────────────┐
│ IP Header │     Payload (1480 bytes)             │
│ ID=100    │     Data: bytes 1480-2959            │
│ Offset=185│                                      │
│ MF=0      │                                      │
└──────────────────────────────────────────────────┘
Size: 1500 bytes (fits MTU)

All fragments have same ID (100) for reassembly
Offset tells where each fragment goes
MF flag indicates more fragments (except last)
```

**Reassembly at destination:**
```
Receiver collects all fragments with ID=100
Sorts by offset
Reconstructs original packet
If any fragment missing: Discard entire packet
```

---

### Problems with Fragmentation

**1. Amplification of packet loss:**
```
If any fragment is lost → Entire packet lost

Example:
  3 fragments
  Each has 1% loss probability
  Total packet success = 0.99³ ≈ 97%
  Effective loss rate: 3%

More fragments = Higher loss probability
```

**2. Reassembly overhead:**
```
Receiver must:
  • Buffer fragments
  • Track partial packets
  • Timeout incomplete packets
  • Consume memory
```

**3. Router overhead:**
```
Fragmenting router must:
  • Copy packet multiple times
  • Recompute checksums
  • Process each fragment
  
Slows down forwarding
```

**4. Security issues:**
```
Fragmentation used in attacks:
  • Tiny fragments (header split across fragments)
  • Overlapping fragments
  • Reassembly timeout attacks
```

---

### Path MTU Discovery (PMTUD): Avoiding Fragmentation

**The modern solution:**

```
Goal: Find the smallest MTU on the path, never fragment

Process:
  1. Sender sets DF (Don't Fragment) flag
  2. Sends packet at assumed MTU (e.g., 1500 bytes)
  3. If too large:
     Router drops packet
     Router sends ICMP "Fragmentation Needed" (Type 3, Code 4)
     Message includes: "MTU on this link is X"
  4. Sender reduces packet size
  5. Repeat until successful

Example:
  Try 1500 → Router says "MTU is 1400"
  Try 1400 → Router says "MTU is 1300"
  Try 1300 → Success!
  
  Use 1300-byte packets for this connection
```

**IPv6 difference:**
- Routers **never** fragment
- DF flag doesn't exist (always implied)
- PMTUD is mandatory

---

## Maximum Segment Size (MSS): TCP's MTU

### What is MSS?

**MSS** is the maximum amount of data TCP can put in a single segment.

**Calculation:**
```
MSS = MTU - IP Header - TCP Header

Standard Ethernet:
  MTU:        1500 bytes
  IP header:  20 bytes (minimum)
  TCP header: 20 bytes (minimum)
  ────────────────────────────
  MSS:        1460 bytes

This is why you often see 1460-byte TCP payloads
```

### MSS Negotiation

**During TCP handshake:**
```
Client → Server: SYN, MSS=1460
Server → Client: SYN-ACK, MSS=1460
Client → Server: ACK

Both sides agree on MSS
Use the smaller of the two values
```

**Why negotiate?**
- Different networks have different MTUs
- Avoid fragmentation
- Optimize for the path

---

## Packet Size Trade-offs

### Small Packets

**Advantages:**
```
✓ Lower serialization delay
✓ Better for real-time traffic
✓ Less chance of bit errors
✓ Faster retransmission if lost
```

**Disadvantages:**
```
✗ High header overhead
✗ More packets per second (CPU load)
✗ More interrupts to process
✗ Lower throughput
```

**Example:**

```
Sending 1 MB of data:

With 100-byte packets:
  Payload per packet: 100 bytes
  Header per packet: 54 bytes (Ethernet + IP + TCP)
  Total overhead: 10,000 packets × 54 bytes = 540 KB
  Efficiency: 1 MB / 1.54 MB = 65%

With 1460-byte packets:
  Payload per packet: 1460 bytes
  Header per packet: 54 bytes
  Total overhead: 685 packets × 54 bytes = 37 KB
  Efficiency: 1 MB / 1.037 MB = 96%
```

---

### Large Packets

**Advantages:**
```
✓ Lower header overhead
✓ Fewer packets per second
✓ Higher throughput
✓ Lower CPU usage
```

**Disadvantages:**
```
✗ Higher serialization delay
✗ Higher loss probability
✗ Entire packet lost if any bit corrupted
✗ May require fragmentation
```

**Serialization delay example:**

```
Link speed: 1 Gbps

100-byte packet:
  Transmission time = 100 bytes × 8 bits/byte / 1 Gbps
                    = 800 nanoseconds

1500-byte packet:
  Transmission time = 1500 × 8 / 1 Gbps
                    = 12 microseconds

9000-byte packet (jumbo frame):
  Transmission time = 9000 × 8 / 1 Gbps
                    = 72 microseconds

Larger packets block the link longer
```

---

### Loss Probability

**Bit error rate (BER) analysis:**

```
If BER = 10⁻⁶ (1 error per million bits)

Small packet (100 bytes = 800 bits):
  Success probability = (1 - 10⁻⁶)⁸⁰⁰ ≈ 99.92%
  Loss rate ≈ 0.08%

Large packet (1500 bytes = 12,000 bits):
  Success probability = (1 - 10⁻⁶)¹²⁰⁰⁰ ≈ 98.81%
  Loss rate ≈ 1.19%

Jumbo frame (9000 bytes = 72,000 bits):
  Success probability = (1 - 10⁻⁶)⁷²⁰⁰⁰ ≈ 93.04%
  Loss rate ≈ 6.96%

Larger packets → More likely to have bit errors
```

---

## Jumbo Frames: When Bigger is Better

### What Are Jumbo Frames?

**Definition:** Ethernet frames with MTU > 1500 bytes

**Common jumbo MTU:** 9000 bytes

**Structure:**
```
Standard Ethernet frame:
┌─────────┬──────────┬────────────┬─────────┐
│ Header  │ IP/TCP   │  Payload   │ Trailer │
│ 14 bytes│ 40 bytes │ 1460 bytes │ 4 bytes │
└─────────┴──────────┴────────────┴─────────┘
Total: 1518 bytes

Jumbo Ethernet frame:
┌─────────┬──────────┬──────────────────────┬─────────┐
│ Header  │ IP/TCP   │      Payload         │ Trailer │
│ 14 bytes│ 40 bytes │   8946 bytes         │ 4 bytes │
└─────────┴──────────┴──────────────────────┴─────────┘
Total: 9004 bytes
```

---

### Why Use Jumbo Frames?

**Throughput improvement:**

```
Transferring 1 GB:

Standard frames (1460-byte payload):
  Packets needed: 1 GB / 1460 bytes ≈ 731,700 packets
  Headers: 731,700 × 54 bytes ≈ 39.5 MB overhead
  Efficiency: 96.2%

Jumbo frames (8946-byte payload):
  Packets needed: 1 GB / 8946 bytes ≈ 119,400 packets
  Headers: 119,400 × 54 bytes ≈ 6.4 MB overhead
  Efficiency: 99.4%

Result: ~3% throughput improvement
```

**CPU load reduction:**

```
Packets per second:

1 Gbps with standard frames:
  1,000,000,000 bits/sec / (1518 bytes × 8 bits/byte)
  ≈ 82,300 packets/second

1 Gbps with jumbo frames:
  1,000,000,000 / (9004 × 8)
  ≈ 13,900 packets/second

6x fewer packets → 6x fewer interrupts → Lower CPU usage
```

---

### Where Jumbo Frames Are Used

**Data centers:**
```
Use case: Server-to-server communication
  • Controlled environment
  • All equipment supports jumbo frames
  • High throughput critical (storage, databases)
  • Low latency less important

Example: iSCSI storage (block-level over Ethernet)
  • Large data transfers
  • Sequential reads/writes
  • Jumbo frames reduce overhead
```

**High-Performance Computing (HPC):**
```
Use case: Supercomputer clusters
  • Message Passing Interface (MPI)
  • Large data sets
  • All nodes homogeneous
```

---

### Why Jumbo Frames Aren't Universal

**Problem 1: Must be supported end-to-end**
```
If any device doesn't support jumbo frames:
  • Packet dropped silently
  • Or fragmented (performance hit)
  • Hard to troubleshoot

Path: PC → Switch A → Switch B → Router → Server
       9000    9000     1500      9000    9000
                         ↑
                    Breaks here!
```

**Problem 2: Internet is heterogeneous**
```
Your data center: 9000 MTU
Internet backbone: Various MTUs
Destination network: 1500 MTU

Can't use jumbo frames for Internet traffic
```

**Problem 3: Higher latency for small packets**
```
9000-byte frame on 1 Gbps link = 72 µs

If you send a small urgent packet behind it:
  Must wait 72 µs for jumbo frame to finish
  
Head-of-line blocking
Bad for latency-sensitive traffic
```

**Rule of thumb:**
```
Jumbo frames:
  ✓ Use in isolated, controlled networks
  ✓ Use for bulk data transfers
  ✓ Ensure all devices support them
  
Don't use:
  ✗ Over the Internet
  ✗ In mixed-MTU environments
  ✗ For latency-sensitive traffic
```

---

## Packet Delay Components

### Total End-to-End Delay

```
Total Delay = Processing + Queuing + Transmission + Propagation
```

**For a packet traveling through N routers:**

```
Total = Σ(Processing_i + Queuing_i + Transmission_i + Propagation_i)
        i=1 to N
```

Let's break down each component:

---

### 1. Processing Delay

**What it is:** Time to examine packet header and determine output link

**Typical value:** 1-10 microseconds per router

**Factors:**
```
• Routing table lookup
• Header checksum verification
• TTL decrement
• Forwarding decision

Modern routers:
  • Hardware-accelerated lookups
  • ~1 µs per packet in fast path
  • Higher for complex decisions (ACLs, NAT)
```

---

### 2. Queuing Delay

**What it is:** Time waiting in router's output buffer

**Typical value:** 0 (empty queue) to 100+ ms (congested)

**Most variable component!**

**Formula:**
```
Average queuing delay = (λ × L) / (2 × (μ - λ))

Where:
  λ = Arrival rate (packets/sec)
  L = Average packet size (bytes)
  μ = Service rate (link capacity in bytes/sec)
```

**Example:**

```
Link: 10 Mbps = 1,250,000 bytes/sec
Packet size: 1500 bytes
Arrival rate: 700 packets/sec

Utilization:
  ρ = (700 × 1500) / 1,250,000 = 0.84 (84%)

Average queuing delay:
  ≈ (0.84 × 1500 bytes) / (2 × 1,250,000 × (1 - 0.84))
  ≈ 3.15 ms

As utilization → 100%, queuing delay → ∞
```

**Bufferbloat:**
```
Problem: Routers with huge buffers
  • Packets wait in queue for 100+ ms
  • Latency spikes
  • Poor interactive performance

Solution: Active Queue Management (AQM)
  • CoDel (Controlled Delay)
  • PIE (Proportional Integral controller Enhanced)
  • Drop packets before buffers fill completely
```

---

### 3. Transmission Delay

**What it is:** Time to push all bits onto the link

**Formula:**
```
Transmission Delay = Packet Size / Link Bandwidth
```

**Examples:**

```
100-byte packet on 1 Mbps link:
  (100 bytes × 8 bits/byte) / 1,000,000 bps = 0.8 ms

1500-byte packet on 1 Gbps link:
  (1500 × 8) / 1,000,000,000 = 12 µs

9000-byte packet on 10 Gbps link:
  (9000 × 8) / 10,000,000,000 = 7.2 µs
```

**Key insight:** Transmission delay inversely proportional to link speed.

---

### 4. Propagation Delay

**What it is:** Time for signal to travel through the medium

**Formula:**
```
Propagation Delay = Distance / Signal Speed
```

**Signal speeds:**
```
Fiber optic: ~200,000 km/s (2/3 speed of light in vacuum)
Copper: ~200,000 km/s
Vacuum (satellite): 300,000 km/s
```

**Examples:**

```
New York to Los Angeles (4000 km):
  4,000,000 m / 200,000,000 m/s = 20 ms

Transatlantic cable (5600 km):
  5,600 km / 200,000 km/s = 28 ms

Geostationary satellite (35,786 km up + down):
  71,572 km / 300,000 km/s = 239 ms round-trip
```

**Cannot be reduced** (physics limit!)

---

### Complete Example

**Sending a 1500-byte packet from New York to London:**

```
Path: [NYC PC] → [Router 1] → [Router 2] → ... → [London Server]
Link: 100 Mbps
Distance: 5600 km
3 routers in path

Per-hop:
  Processing: 5 µs
  Queuing: 10 ms (average, congested)
  Transmission: (1500 × 8) / 100,000,000 = 120 µs
  Propagation: 5,600,000 m / 200,000,000 m/s = 28 ms

Total for 3 hops:
  Processing: 3 × 5 µs = 15 µs
  Queuing: 3 × 10 ms = 30 ms
  Transmission: 3 × 120 µs = 360 µs
  Propagation: 28 ms (one-time, distance-based)
  
Total: ~58.4 ms

Dominated by queuing delay and propagation delay
```

---

## Best-Effort Delivery: IP's Promises (or Lack Thereof)

### What IP Guarantees

```
Nothing.
```

**IP provides:**
- ❌ No guarantee of delivery
- ❌ No guarantee of order
- ❌ No guarantee against duplication
- ❌ No guarantee of timing
- ❌ No guarantee of bandwidth

**This is called "best-effort" delivery.**

---

### Why So Unreliable?

**Design philosophy:**

> Keep the network core simple and fast.  
> Let the endpoints handle complexity.

**Benefits:**
```
1. Routers can be fast
   • No per-flow state
   • No acknowledgments
   • Just look up and forward

2. Network is resilient
   • No connection setup
   • Automatically routes around failures
   • Scales to billions of devices

3. Flexibility
   • Can run any application
   • Application chooses reliability level
   • TCP for reliability, UDP for speed
```

**End-to-end principle:**

> Intelligence at the edges, simplicity in the core.

---

### How Reliability is Added

**Transport layer (TCP) adds:**
```
✓ Reliable delivery (retransmission)
✓ Ordered delivery (sequence numbers)
✓ Duplicate detection
✓ Flow control (don't overwhelm receiver)
✓ Congestion control (don't overwhelm network)
```

**Application layer can add more:**
```
• Forward error correction (video streaming)
• Application-level retries (HTTP)
• Acknowledgments (email delivery)
```

**Layering in action:**

```
IP: "I'll try to deliver your packet, but no promises"
TCP: "I guarantee it gets there, in order, exactly once"
HTTP: "I'll retry if the request fails"

Each layer adds guarantees as needed
```

---

## Packet Loss: Expected, Not Exceptional

### Causes of Packet Loss

**1. Buffer overflow (congestion):**
```
Router's output queue:
┌───┬───┬───┬───┬───┐
│ P1│ P2│ P3│ P4│ P5│ (Queue full)
└───┴───┴───┴───┴───┘

New packet arrives → Dropped (queue full)

Most common cause of loss on the Internet
```

**2. Bit errors (corruption):**
```
Wireless links:
  • Interference
  • Fading
  • Multipath

Copper cables:
  • Electromagnetic interference
  • Cross-talk

Fiber:
  • Rare, but possible

If checksum fails → Packet dropped
```

**3. Routing loops (TTL expires):**
```
Misconfigured routing:
  Packet: A → B → C → A → B → C → ...
  
TTL decrements each hop:
  64 → 63 → 62 → ... → 1 → 0
  
When TTL = 0: Packet dropped
```

**4. Policy/filtering:**
```
Firewall rules:
  • Block certain ports
  • Block certain IPs
  • Rate limiting

Packet matches drop rule → Silently discarded
```

---

### Handling Packet Loss

**TCP's approach:**
```
1. Sender transmits packet
2. Starts retransmission timer
3. Receiver acknowledges receipt
4. If timer expires before ACK:
   → Assume packet lost
   → Retransmit

Automatic recovery, transparent to application
```

**UDP's approach:**
```
No retransmission
Application must handle loss

Use cases:
  • DNS: Retry query if no response
  • VoIP: Accept occasional glitch
  • Video streaming: Skip lost frame
  • Gaming: Ignore old position updates
```

**Acceptable loss rates:**

| Application | TCP/UDP | Acceptable Loss |
|-------------|---------|-----------------|
| Web browsing | TCP | 0% (retransmit) |
| File transfer | TCP | 0% (retransmit) |
| Email | TCP | 0% (retransmit) |
| Video streaming | UDP/TCP | <2% (minor quality loss) |
| VoIP | UDP | <5% (noticeable but usable) |
| Online gaming | UDP | <10% (playable) |
| DNS | UDP | ~0% (retry) |

---

## Security and Packets

### Packet Inspection

**Firewalls examine packet headers:**

```
Rule: Block all packets to port 22 (SSH)

Packet arrives:
  Destination IP: 192.168.1.10
  Destination Port: 22
  
Firewall checks:
  Port 22? → Match rule → DROP

User sees: Connection refused
```

**Deep Packet Inspection (DPI):**

```
Examines payload, not just headers
  
Use cases:
  • Malware detection
  • Content filtering
  • Traffic shaping
  • Surveillance
  
Privacy concerns!
```

---

### Encryption: Hiding the Payload

**TLS/SSL encryption:**

```
Without encryption:
┌──────────┬──────────┬─────────────────────────┐
│IP Header │TCP Header│ "password: secret123"   │
│ (visible)│ (visible)│ (visible)               │
└──────────┴──────────┴─────────────────────────┘
         ↑
    Anyone can read payload!

With encryption (HTTPS):
┌──────────┬──────────┬─────────────────────────┐
│IP Header │TCP Header│ "$#@%^&*(!@#$%^&*()"    │
│ (visible)│ (visible)│ (encrypted)             │
└──────────┴──────────┴─────────────────────────┘
         ↑
    Headers still visible, but payload encrypted
```

**What's still visible:**
- Source/destination IP addresses
- Source/destination ports
- Packet sizes
- Timing

**Metadata leaks information!**

---

### VPNs: Hiding Metadata

**VPN tunneling:**

```
Original packet:
┌──────────────┬────────────────────────────┐
│ IP: You → Bank │ Payload: Login request  │
└──────────────┴────────────────────────────┘

Encrypted and wrapped:
┌──────────────────┬─────────────────────────────────────┐
│ IP: You → VPN    │ Encrypted original packet           │
│                  │ (Including original IP header!)     │
└──────────────────┴─────────────────────────────────────┘

ISP sees:
  • You're talking to VPN server
  • Cannot see final destination (Bank)
  • Cannot see contents

VPN server decrypts and forwards to actual destination
```

---

## Summary: The Big Picture

**Packets are the fundamental unit of Internet communication:**

```
Why packets?
  • Efficient resource sharing
  • Resilient to failures
  • Scalable architecture

How they work:
  • Break data into chunks
  • Add headers (encapsulation)
  • Route independently
  • Reassemble at destination

Key concepts:
  • MTU limits packet size
  • Fragmentation (avoid it!)
  • Path MTU Discovery
  • MSS for TCP optimization
  • Header overhead vs efficiency
  • Delay components
  • Best-effort delivery
  • Loss is normal

Trade-offs everywhere:
  • Small packets: Low latency, high overhead
  • Large packets: High throughput, higher loss probability
  • Jumbo frames: Data centers only
  • Reliability: Added by higher layers (TCP)
```

**The Internet's genius:**

> Simple, unreliable packet forwarding in the core  
> + Complex, reliable protocols at the edges  
> = Scalable, resilient global network

