# Introduction to Computer Networks

## The World Before Networks

Imagine it's 1970. You work at a university research lab with three computers:

```
Building A: DEC PDP-11 (for number crunching)
Building B: IBM System/360 (for data processing)  
Building C: Xerox Alto (for document editing)

Problem: Need to share data between them

Solution options:
1. Physically carry magnetic tapes between buildings (slow, error-prone)
2. Duplicate data on each machine (expensive, inconsistent)
3. Connect the machines somehow... (networking!)
```

This is the fundamental problem networks solve: **enabling independent computers to communicate and share resources**.

---

## What is a Computer Network?

### The Formal Definition

A **computer network** is a collection of **autonomous computing devices** interconnected by **communication links** and governed by **protocols** that enable them to exchange data and share resources.

**Breaking this down:**

**Autonomous devices:**
- Each has its own CPU, memory, and operating system
- Makes independent decisions
- No central controller (unlike terminals connected to a mainframe)

**Communication links:**
- Physical media (wires, fiber optics, radio waves)
- Carry signals representing data
- Can be wired or wireless

**Protocols:**
- Agreed-upon rules for communication
- Define message format, timing, error handling
- Enable different devices to understand each other

### The Intuitive Understanding

Think of a network as a **postal system for computers**:

```
Postal System:
  • Houses = Computers (autonomous)
  • Roads = Communication links
  • Addresses = Network addresses
  • Postal rules = Protocols (how to address, stamp, route mail)
  
Network:
  • Computers = Devices
  • Cables/WiFi = Links
  • IP addresses = Addresses
  • TCP/IP = Protocols
```

**Key insight: Autonomy matters**

```
Not a network:
  Mainframe ← Terminal 1
           ← Terminal 2  
           ← Terminal 3
  (Terminals have no CPU, just display—not autonomous)

Is a network:
  Server ↔ Laptop
         ↔ Phone
         ↔ Desktop
  (Each device independent, cooperating)
```

---

## A Real-World Example: Loading a Web Page

Let's trace what happens when you type `www.google.com` and press Enter:

```
Step 1: Your computer
  • Needs Google's IP address
  • Sends DNS query to router

Step 2: Your home router
  • Forwards DNS query to ISP's DNS server
  • Keeps track to route response back

Step 3: ISP's DNS server
  • Looks up google.com → 142.250.80.46
  • Sends response back to your router

Step 4: Your router
  • Forwards DNS response to your computer

Step 5: Your computer
  • Now knows Google's IP: 142.250.80.46
  • Sends HTTP request to that address

Step 6: Request travels through
  • Your router
  • Your ISP's network
  • Internet backbone routers
  • Google's ISP
  • Google's servers

Step 7: Google's server
  • Processes request
  • Sends back HTML, CSS, JavaScript

Step 8: Response travels back
  • Same path in reverse
  • Your browser renders the page

Total: Dozens of networks cooperated
Time: ~100 milliseconds
Distance: Potentially thousands of miles
```

**This is networking:** Multiple independent networks cooperating to deliver a single logical conversation.

---

## Why Do Computer Networks Exist?

Networks solve fundamental problems in computing. Let's explore each goal.

### Goal 1: Resource Sharing

**The problem:**

```
1980s university:
  • Laser printer costs: $100,000
  • Students: 500
  • Budget: Can't buy 500 printers

Solution: Buy 1 printer, connect via network
```

**What can be shared:**

**Hardware resources:**
```
• Printers (expensive peripherals)
• Storage (NAS devices, SAN)
• Compute power (GPU farms, HPC clusters)
• Specialized equipment (oscilloscopes, sensors)
```

**Software resources:**
```
• Database servers (MySQL, PostgreSQL)
• Web servers (Apache, Nginx)
• Application servers
• License servers (expensive software)
```

**Data resources:**
```
• Files (shared drives)
• Databases (customer records)
• APIs (weather data, maps)
• Streaming media
```

**Economic model:**

```
Without networking:
  Cost per user = Full resource cost
  10 users × $10,000 printer = $100,000

With networking:
  Cost per user = (Shared resource cost) / (# users)
  1 × $10,000 printer / 10 users = $1,000 per user
  
Savings: 90%
```

**Modern example: Cloud computing**

```
AWS EC2 instance:
  • Physical server: $10,000
  • 100 VMs share the server via networking
  • You rent 1 VM for $50/month
  
You get $10,000 worth of hardware for $50/month
  → Only possible through resource sharing over networks
```

---

### Goal 2: Communication & Collaboration

**The fundamental insight:**

> Networking is state synchronization across physically separated machines.

**Examples:**

**Email (asynchronous communication):**
```
Alice's state:      Bob's state:
  Compose message     Inbox: (empty)
      ↓                    ↓
  Send (network)     Receive
      ↓                    ↓
  Sent folder        Inbox: Alice's message

State synchronized across machines
```

**Video call (synchronous communication):**
```
Alice's webcam:           Bob's screen:
  Capture frame 1     →   Display frame 1
  Capture frame 2     →   Display frame 2
  Capture frame 3     →   Display frame 3
  
30 frames/second synchronized in real-time
```

**Collaborative editing (Google Docs):**
```
Alice types "Hello"
  ↓
Network synchronizes
  ↓
Bob sees "Hello" appear on his screen

Bob types " World"
  ↓
Network synchronizes
  ↓
Alice sees "Hello World"

Both see same document state simultaneously
```

**Categories:**

| Type | Examples | Latency Tolerance |
|------|----------|-------------------|
| **Asynchronous** | Email, forums, social media posts | Seconds to hours |
| **Synchronous** | Video calls, online gaming | <100 milliseconds |
| **Real-time** | Stock trading, VoIP, remote surgery | <10 milliseconds |

---

### Goal 3: Scalability

**The single-machine wall:**

```
One server limits:
  CPU: 64 cores → Can handle ~1,000 concurrent users
  Memory: 256 GB
  Disk: 10 TB
  Network: 10 Gbps

What if you need to serve 1 million users?
```

**Vertical scaling (bigger machine):**

```
Buy a bigger server:
  CPU: 128 cores
  Memory: 1 TB
  
Problems:
  ✗ Expensive (exponential cost increase)
  ✗ Still limited (can't buy infinite CPU)
  ✗ Single point of failure
```

**Horizontal scaling (more machines via network):**

```
Buy 100 smaller servers:
  Each: 8 cores, 32 GB RAM
  Total: 800 cores, 3.2 TB RAM
  
Benefits:
  ✓ Linear cost scaling
  ✓ No practical limit (add more as needed)
  ✓ Fault tolerance (one fails, others continue)
  
Challenge: Need network to coordinate them
```

**Real-world example: Google Search**

```
Query: "What is a computer network?"

Behind the scenes:
  1. Load balancer distributes query to 1 of 1000s of servers
  2. Query server asks index servers (100s of them)
  3. Index servers query document servers (1000s of them)
  4. Results aggregated and ranked
  5. Returned to user
  
Total machines involved: 10,000+
Time: 200 milliseconds
  
This is only possible because of networking
```

---

### Goal 4: Reliability & Fault Tolerance

**The harsh reality:**

> In a large system, something is always broken.

**Failure modes:**

```
Hardware:
  • Cables cut (construction accidents)
  • Routers crash (power failure, bugs)
  • Servers die (disk failure, overheating)
  
Software:
  • Bugs cause crashes
  • Configuration errors
  
External:
  • Power outages
  • Natural disasters
  • Deliberate attacks (DDoS)
```

**Network solution: Redundancy**

```
Single path (fragile):
  Your computer → Router A → Server
  
  If Router A fails → Complete failure
  
Multiple paths (resilient):
  Your computer → Router A → Server
                ↘ Router B ↗
  
  If Router A fails → Automatic reroute via Router B
```

**Internet design principle:**

> "The network is a stupid cloud that just moves packets. Intelligence is at the edges."

This means:
- Network core is simple and resilient
- If a router fails, packets automatically reroute
- End systems (edges) handle error recovery

**Example: Undersea cable cut**

```
Atlantic cable layout:
  TAT-14 (cable 1): US ↔ Europe
  AC-1 (cable 2):   US ↔ Europe  
  AC-2 (cable 3):   US ↔ Europe
  
Scenario: Ship anchor cuts TAT-14
  
What happens:
  • Routers detect failure within seconds
  • Traffic automatically reroutes to AC-1 and AC-2
  • Users may notice slight slowdown (more congestion)
  • No total outage
  
Without redundancy:
  • TAT-14 cut → Europe unreachable from US
  • Catastrophic failure
```

---

## The Building Blocks of a Network

Every network, from a home WiFi to the global Internet, is built from these components.

### 1. Nodes (End Systems)

**Definition:** Devices that generate, process, or consume data.

**Categories:**

**Clients:**
```
• Initiate requests
• Consume resources
• Examples: Laptops, phones, IoT sensors
```

**Servers:**
```
• Respond to requests
• Provide resources
• Examples: Web servers, database servers, file servers
```

**Peers:**
```
• Act as both client and server
• Examples: BitTorrent peers, blockchain nodes
```

**Key characteristic: Intelligence**

```
Nodes run applications:
  • Web browser (client)
  • Database (server)
  • Video chat (peer-to-peer)
  
Network core just forwards packets
  → "Dumb network, smart endpoints"
```

---

### 2. Communication Links

Links physically connect nodes and carry signals.

#### Wired Media

**Twisted Pair (Ethernet cable):**

```
Structure:
  8 wires twisted in 4 pairs
  
Why twisted?
  • Reduces electromagnetic interference
  • Cheaper than shielding
  
Speeds:
  Cat5e:  1 Gbps  (100 meters max)
  Cat6:   10 Gbps (55 meters max)
  
Use case: Office networks, home networks
```

**Coaxial Cable:**

```
Structure:
  Central conductor + insulation + shield + outer jacket
  
Speeds:
  DOCSIS 3.1: 10 Gbps
  
Use case: Cable TV, cable Internet
```

**Optical Fiber:**

```
Structure:
  Glass core (carries light pulses)
  Cladding (reflects light back into core)
  
Speeds:
  Single-mode fiber: 100+ Gbps
  Multi-mode fiber: 10-40 Gbps
  
Distance:
  100+ kilometers without amplification
  
Advantages:
  ✓ Extremely high bandwidth
  ✓ Low signal loss
  ✓ Immune to electromagnetic interference
  ✓ Difficult to tap (secure)
  
Disadvantages:
  ✗ Expensive to install
  ✗ Fragile (glass breaks)
  
Use case: Long-distance backbones, data center interconnects
```

**Comparison:**

| Medium | Speed | Distance | Cost | Use |
|--------|-------|----------|------|-----|
| Twisted Pair | 1-10 Gbps | 100m | $ | LANs |
| Coax | 1-10 Gbps | 500m | $$ | Cable ISP |
| Fiber | 10-100+ Gbps | 100+ km | $$$ | Backbone, data centers |

---

#### Wireless Media

**WiFi (IEEE 802.11):**

```
Standards evolution:
  802.11b (1999): 11 Mbps
  802.11g (2003): 54 Mbps
  802.11n (2009): 600 Mbps
  802.11ac (2014): 3.5 Gbps
  802.11ax/WiFi 6 (2019): 9.6 Gbps
  
Range: ~50 meters indoors
Frequency: 2.4 GHz and 5 GHz bands
```

**Cellular (Mobile Networks):**

```
Evolution:
  1G (1980s): Analog voice
  2G (1991): Digital voice, SMS (9.6 Kbps)
  3G (2001): Mobile data (2 Mbps)
  4G/LTE (2009): High-speed data (100 Mbps)
  5G (2020): Ultra-high-speed (10 Gbps), low latency
  
Range: ~10 km per tower
Frequency: 600 MHz - 39 GHz
```

**Satellite:**

```
Types:
  GEO (Geostationary): 35,786 km altitude
    • Latency: ~500ms (round trip)
    • Coverage: 1/3 of Earth
    • Use: TV broadcast, rural Internet
    
  LEO (Low Earth Orbit): 500-2000 km altitude  
    • Latency: ~20-40ms
    • Coverage: Requires constellation
    • Use: Starlink, Iridium
```

**Wireless trade-offs:**

```
Advantages:
  ✓ Mobility
  ✓ Easy deployment (no cables)
  ✓ Flexible coverage
  
Disadvantages:
  ✗ Shared medium (interference)
  ✗ Higher error rates
  ✗ Security challenges (broadcast nature)
  ✗ Limited bandwidth
  ✗ Weather-dependent
```

---

### 3. Network Devices (The Intermediaries)

These devices forward data between nodes.

#### Switch (Layer 2 - Data Link)

**What it does:**
```
Connects devices in a Local Area Network (LAN)
Forwards frames based on MAC addresses
```

**How it works:**

```
Learning process:
  1. Switch starts with empty MAC address table
  2. Device A sends frame to Device B
  3. Switch records: "Device A is on Port 1" (learns)
  4. Switch doesn't know where B is → Floods to all ports
  5. Device B responds
  6. Switch records: "Device B is on Port 3" (learns)
  7. Future frames: A ↔ B go directly via learned ports

MAC Address Table:
  Port 1: MAC aa:bb:cc:dd:ee:01 (Device A)
  Port 2: MAC aa:bb:cc:dd:ee:02 (Device C)
  Port 3: MAC aa:bb:cc:dd:ee:03 (Device B)
```

**Benefits:**
- Eliminates collisions (dedicated bandwidth per port)
- Scales to hundreds of devices
- Plug-and-play (no configuration needed)

---

#### Router (Layer 3 - Network)

**What it does:**
```
Connects multiple networks
Forwards packets based on IP addresses
Makes routing decisions
```

**How it works:**

```
Routing table example:

Destination        Next Hop        Interface
─────────────────────────────────────────────
192.168.1.0/24     Direct          eth0 (Local LAN)
10.0.0.0/8         192.168.1.1     eth1 (Corporate network)
0.0.0.0/0          203.0.113.1     eth2 (Internet - default route)

Packet arrives: Destination = 10.5.6.7
  1. Check routing table
  2. Best match: 10.0.0.0/8
  3. Forward to 192.168.1.1 via eth1
```

**Key responsibilities:**
- IP address lookup
- Best path selection
- TTL (Time To Live) decrement
- Fragmentation if needed
- Network Address Translation (NAT)

---

#### Switch vs Router

| Feature | Switch | Router |
|---------|--------|--------|
| **Layer** | 2 (Data Link) | 3 (Network) |
| **Address** | MAC address | IP address |
| **Scope** | Single network (LAN) | Multiple networks (LAN to WAN) |
| **Intelligence** | Simple forwarding | Path selection, NAT, firewall |
| **Speed** | Very fast (hardware) | Slower (more processing) |
| **Use** | Connect devices in office | Connect home to Internet |

---

#### Modern "Router"

What you buy as a "home router" is actually:

```
┌─────────────────────────────────────┐
│  Consumer "Router" (all-in-one)     │
│                                     │
│  • Modem (for ISP connection)       │
│  • Router (NAT, routing)            │
│  • Switch (4-port Ethernet)         │
│  • WiFi Access Point                │
│  • Firewall                         │
│  • DHCP server                      │
└─────────────────────────────────────┘
```

---

## Network Classification by Scale

Networks are categorized by geographic scope and purpose.

### PAN (Personal Area Network)

**Range:** 1-10 meters  
**Technologies:** Bluetooth, NFC, USB  
**Use cases:**

```
• Phone ↔ Smartwatch
• Laptop ↔ Wireless mouse/keyboard  
• Phone ↔ Wireless earbuds
• Contactless payment (NFC)
```

**Characteristics:**
- Very low power
- No infrastructure needed (ad-hoc)
- Limited devices (<10 typically)

---

### LAN (Local Area Network)

**Range:** Single building or campus  
**Technologies:** Ethernet (wired), WiFi (wireless)  
**Speed:** 1-10 Gbps (wired), 100 Mbps - 1 Gbps (WiFi)

**Topology examples:**

```
Star (most common):
       [Switch]
      /   |   \
    PC1  PC2  PC3

Bus (legacy):
  PC1 ─ PC2 ─ PC3 ─ [Terminator]
  
Ring (legacy):
  PC1 → PC2 → PC3 → PC1
```

**Characteristics:**
- Privately owned and managed
- High speed, low latency (<1 ms)
- High reliability
- Simple management

**Example: Office LAN**

```
┌──────────────────────────────────┐
│         Office Floor 3           │
│  [Switch] ← 10 computers         │
└────────────┬─────────────────────┘
             │ Fiber link
┌────────────▼─────────────────────┐
│         Office Floor 2           │
│  [Switch] ← 10 computers         │
└────────────┬─────────────────────┘
             │
┌────────────▼─────────────────────┐
│         Office Floor 1           │
│  [Router] ← Server room          │
│     ↓                            │
│  Internet                        │
└──────────────────────────────────┘
```

---

### MAN (Metropolitan Area Network)

**Range:** City-scale (10-100 km)  
**Technologies:** Fiber optic, Metro Ethernet  
**Ownership:** Usually ISP or municipality

**Use cases:**
```
• City government connecting multiple offices
• University connecting multiple campuses
• ISP backbone within a city
• Cable TV distribution network
```

**Characteristics:**
- Higher latency than LAN (~5-20 ms)
- Managed by telecom providers
- Expensive infrastructure

---

### WAN (Wide Area Network)

**Range:** Country to global  
**Technologies:** Leased lines, MPLS, Internet  
**Speed:** Varies (1 Mbps - 100 Gbps)

**The Internet is a WAN** (the biggest one).

**Characteristics:**
- Latency becomes dominant factor
- Multiple hops (10-30 routers)
- Crosses ISP boundaries
- Expensive (distance-based pricing)

**Latency examples:**

```
Same city (LAN):        <1 ms
Cross-country (US):     ~50 ms
US ↔ Europe:            ~100 ms
US ↔ Asia:              ~150 ms
US ↔ Australia:         ~200 ms
Satellite (GEO):        ~500 ms
```

**Why latency matters:**

```
Web page load (100 resources):
  • LAN (1 ms): 100 × 1 ms = 0.1 second
  • WAN (100 ms): 100 × 100 ms = 10 seconds
  
This is why CDNs (Content Delivery Networks) exist:
  → Cache content geographically close to users
  → Reduce latency
```

---

## The Internet: Network of Networks

### What Makes the Internet Special

**Not one network, but many interconnected networks:**

```
┌────────────────┐    ┌────────────────┐
│ Home Network   │────│ ISP Network    │
└────────────────┘    └────────┬───────┘
                               │
                      ┌────────▼────────┐
                      │ Internet        │
                      │ Backbone        │
                      │ (Tier 1 ISPs)   │
                      └────────┬────────┘
                               │
                      ┌────────▼───────┐
                      │ Google's       │
                      │ Network        │
                      └────────────────┘
```

**Key properties:**

**Decentralized:**
- No single owner
- No central control point
- Resilient to attacks and failures

**Standardized:**
- TCP/IP protocol suite (common language)
- Anyone can connect if they follow standards

**Layered:**
- Each network autonomous within itself
- Interconnects via peering agreements

---

### Autonomous Systems (AS)

**Definition:** A collection of IP networks under a single administrative domain.

```
AS Example:
  Google (AS15169):
    • Owns thousands of servers worldwide
    • Has own routing policies
    • Peers with other ASes
    
  Your ISP (e.g., AS7922 Comcast):
    • Manages residential customers
    • Routes traffic between customers and rest of Internet
    • Peers with other ISPs
```

**Peering:**

```
Two ASes agree to exchange traffic:

AS 1 (ISP A) ↔ AS 2 (ISP B)
  
Types:
  1. Settlement-free peering (mutual benefit)
  2. Paid transit (smaller AS pays larger AS)
```

**Routing between ASes: BGP (Border Gateway Protocol)**

```
AS announces to neighbors:
  "I can reach 192.0.2.0/24"
  
Neighbors propagate:
  "I can reach 192.0.2.0/24 via AS X"
  
Global routing table emerges from local announcements
  → Decentralized path discovery
```

---

## Data Communication Fundamentals

### How Data Becomes Signals

**Digital data representation:**

```
Text "Hi" in ASCII:
  H → 01001000 (binary)
  i → 01101001 (binary)
  
Image (1 red pixel):
  Red: 255 → 11111111
  Green: 0 → 00000000
  Blue: 0 → 00000000
  
Video (sequence of images):
  Frame 1 (image) + Frame 2 (image) + ...
```

**Signal encoding:**

```
Digital signal (Ethernet):
  High voltage (+2.5V) = 1
  Low voltage (0V) = 0
  
  Data: 1 0 1 1 0
  Signal:
    ─┐   ┌─┐ ┌─
     │   │ │ │
     └───┘ └─┘

Analog signal (over phone line):
  High frequency = 1
  Low frequency = 0
```

---

### Performance Metrics

#### Bandwidth (Capacity)

**Definition:** Maximum data rate of a link.

**Units:**
```
bps  = bits per second
Kbps = 1,000 bps
Mbps = 1,000,000 bps
Gbps = 1,000,000,000 bps
```

**Real-world examples:**

```
Dial-up modem:    56 Kbps
DSL:              10-100 Mbps
Cable:            100-1000 Mbps
Fiber (home):     1-10 Gbps
Data center:      40-400 Gbps
```

**Bandwidth ≠ Speed:**

```
Bandwidth = How much water flows through a pipe
Latency = How long water takes to reach the end

Wide pipe (high bandwidth):
  • Can send large file quickly
  
Long pipe (high latency):
  • Still takes time for first bit to arrive
```

---

#### Latency (Delay)

**Definition:** Time for data to travel from source to destination.

**Components:**

```
Total Latency = Propagation + Transmission + Processing + Queuing

Propagation delay:
  • Speed of signal through medium
  • ~2/3 speed of light in fiber (~200,000 km/s)
  • Distance / Speed
  • Example: 3000 km / 200,000 km/s = 15 ms

Transmission delay:
  • Time to push all bits onto wire
  • Packet size / Bandwidth
  • Example: 1500 bytes / 1 Gbps = 12 microseconds

Processing delay:
  • Router lookup, checksum verification
  • ~1-10 microseconds per router

Queuing delay:
  • Waiting in router buffer
  • Variable (depends on congestion)
  • Can be 0 ms (empty queue) to 100+ ms (congested)
```

**Why latency matters more than bandwidth for some apps:**

```
Video streaming:
  • Bandwidth: Need 5 Mbps sustained
  • Latency: Can tolerate 500 ms (buffering)
  
Online gaming:
  • Bandwidth: Need only 100 Kbps
  • Latency: Must be <50 ms (real-time)
```

---

#### Jitter (Latency Variation)

**Definition:** Variation in packet arrival times.

```
Ideal (no jitter):
  Packet 1 arrives at t=100ms
  Packet 2 arrives at t=200ms (exactly 100ms later)
  Packet 3 arrives at t=300ms (exactly 100ms later)
  
Reality (with jitter):
  Packet 1 arrives at t=100ms
  Packet 2 arrives at t=215ms (115ms later)
  Packet 3 arrives at t=285ms (70ms later)
  
Jitter = variation (45ms in this example)
```

**Impact:**

```
VoIP call:
  High jitter → choppy audio, gaps
  
Video conferencing:
  High jitter → frozen frames, desynced audio
  
Solution: Jitter buffer (delay playback to smooth out variation)
```

---

#### Packet Loss

**Definition:** Percentage of packets that don't arrive.

**Causes:**
```
• Buffer overflow (router queue full)
• Transmission errors (wireless interference)
• Route changes (packets take wrong path)
```

**Impact:**

```
TCP (reliable):
  • Detects loss
  • Retransmits missing packets
  • Reduces throughput
  
UDP (unreliable):
  • No retransmission
  • Application must handle loss
  • Examples: Live video (skip lost frames), VoIP (accept glitch)
```

**Acceptable loss rates:**

```
Web browsing: <1% (TCP handles it)
Video streaming: <2% (minor quality degradation)
VoIP: <5% (noticeable but usable)
```

---

## Protocols: The Language of Networks

### What is a Protocol?

A **protocol** is a set of rules governing communication between network entities.

**Analogy: Human conversation**

```
Protocol: English language + social norms

Rules:
  • Format: Words, grammar
  • Ordering: Greeting → conversation → goodbye
  • Error handling: "Pardon?" if didn't hear
  • State: Taking turns speaking
  
Network protocol similar but more rigid
```

**What a protocol defines:**

```
1. Message format:
   ┌──────────┬─────────┬─────────────┐
   │  Header  │ Payload │  Checksum   │
   └──────────┴─────────┴─────────────┘
   
2. Message sequence:
   Client → Server: "GET /index.html"
   Server → Client: "200 OK [file contents]"
   
3. Error handling:
   Packet corrupted → Discard, request retransmission
   
4. State transitions:
   CLOSED → LISTEN → ESTABLISHED → CLOSED
```

---

### Protocol Examples

**HTTP (HyperText Transfer Protocol):**

```
Client request:
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0

Server response:
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<html>...</html>
```

**TCP (Transmission Control Protocol):**

```
Establishes connection:
  Client → Server: SYN
  Server → Client: SYN-ACK
  Client → Server: ACK
  (3-way handshake)

Transfers data reliably:
  • Sequence numbers (detect loss/reordering)
  • Acknowledgments (confirm receipt)
  • Retransmission (recover from loss)
```

**UDP (User Datagram Protocol):**

```
Simple, connectionless:
  • Just send packet (no handshake)
  • No reliability (no ACKs, no retransmission)
  • Low overhead
  
Use when:
  • Speed > reliability (gaming, VoIP, DNS)
  • Application handles errors itself
```

**IP (Internet Protocol):**

```
Packet header:
┌──────────────────────────────────┐
│ Source IP: 192.168.1.5           │
│ Destination IP: 142.250.80.46    │
│ Protocol: TCP (6)                │
│ TTL: 64                          │
│ [Payload]                        │
└──────────────────────────────────┘

Routing:
  Each router forwards based on destination IP
  Decrements TTL (prevents infinite loops)
```

---

### Why Standards Matter

**Without standards:**

```
Vendor A's device speaks Protocol X
Vendor B's device speaks Protocol Y
  → Cannot communicate
  → Must buy all equipment from same vendor
```

**With standards:**

```
Everyone implements TCP/IP
  → Any device can talk to any other device
  → Competition improves quality and lowers prices
  → Innovation flourishes
```

**Standard bodies:**

```
IETF (Internet Engineering Task Force):
  • Defines Internet protocols (TCP, IP, HTTP)
  • Open process (anyone can participate)
  • RFCs (Request for Comments) document standards
  
IEEE (Institute of Electrical and Electronics Engineers):
  • Defines Ethernet (802.3), WiFi (802.11)
  
ITU (International Telecommunication Union):
  • Defines telecom standards
```

---

## Layered Architecture: Managing Complexity

### The Problem: Networking is Complex

```
To send a message across the Internet:
  • Application formats data (HTTP)
  • Reliable delivery needed (TCP)
  • Addressing and routing (IP)
  • Physical transmission (Ethernet/WiFi)
  • Error detection (CRC)
  • Signal encoding (voltages)
  
Too complex to design as one monolithic system!
```

### The Solution: Layers

**Each layer solves one problem, hiding details from layers above.**

The Internet uses the **TCP/IP model** with **4 layers**:

```
┌─────────────────────────────────────────────────┐
│  Layer 4: Application Layer                     │
│  Protocols: HTTP, SMTP, FTP, DNS, SSH          │
│  Purpose: "What application service do I need?" │
│  Example: Web browsing, email, file transfer   │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  Layer 3: Transport Layer                       │
│  Protocols: TCP, UDP                            │
│  Purpose: "How do I deliver data reliably?"     │
│  Example: Connection management, flow control   │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  Layer 2: Internet Layer                        │
│  Protocols: IP (IPv4, IPv6), ICMP, ARP          │
│  Purpose: "How do I route across networks?"     │
│  Example: Logical addressing, packet forwarding │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  Layer 1: Network Access Layer                  │
│  (Also called Link Layer or Network Interface)  │
│  Protocols: Ethernet, WiFi, PPP                 │
│  Purpose: "How do I transmit on this medium?"   │
│  Example: Physical addressing, frame creation   │
└─────────────────────────────────────────────────┘
                      ↓
              Physical Hardware
           (Cables, NICs, Radio)
```

**Note:** Some references describe TCP/IP as 5 layers by splitting the bottom layer into:
- **Data Link Layer** (Ethernet, WiFi framing)
- **Physical Layer** (actual bits on wire)

But the **original TCP/IP model has 4 layers**. We'll use the 4-layer model.

---

### How Layering Works: An Example

**Scenario:** You send an email from your laptop to a friend across the Internet.

```
Your Laptop:
┌─────────────────────────────────────────────────────────────┐
│ Application Layer (SMTP):                                   │
│   "Send email to friend@example.com"                        │
│   Creates: Email message with headers                       │
└──────────────────────────┬──────────────────────────────────┘
                           ↓ (hands down to Transport)
┌─────────────────────────────────────────────────────────────┐
│ Transport Layer (TCP):                                      │
│   "Break email into segments, ensure reliable delivery"     │
│   Adds: Port numbers (src: 54321, dst: 25), sequence #s    │
└──────────────────────────┬──────────────────────────────────┘
                           ↓ (hands down to Internet)
┌─────────────────────────────────────────────────────────────┐
│ Internet Layer (IP):                                        │
│   "Route to destination network"                            │
│   Adds: IP addresses (src: 192.168.1.5, dst: 203.0.113.10) │
└──────────────────────────┬──────────────────────────────────┘
                           ↓ (hands down to Network Access)
┌─────────────────────────────────────────────────────────────┐
│ Network Access Layer (Ethernet):                            │
│   "Transmit to next hop (router)"                           │
│   Adds: MAC addresses (src: AA:BB:CC:DD:EE:FF, dst: router) │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
                    [Physical wire]
                           ↓
Router:
┌─────────────────────────────────────────────────────────────┐
│ Network Access Layer: Receive frame, check MAC              │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Internet Layer: Check destination IP, forward to next hop   │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Network Access Layer: Add new MAC address, send to next hop │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
                    (Repeat through Internet...)
                           ↓
Friend's Mail Server:
┌─────────────────────────────────────────────────────────────┐
│ Network Access Layer: Receive frame                         │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Internet Layer: "This packet is for me! (203.0.113.10)"    │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Transport Layer: "Reassemble segments, verify delivery"     │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Application Layer: "Process email via SMTP, store in inbox" │
└─────────────────────────────────────────────────────────────┘
```

**Key insight:** Each layer only talks to its corresponding layer on the other end (logical communication), but physically data flows down and up the stack.

---

### Data Encapsulation: Headers All the Way Down

As data moves down the stack, each layer adds its own header:

```
Application Layer:
┌──────────────────────────┐
│   Email Message          │
└──────────────────────────┘

Transport Layer (TCP adds header):
┌──────────┬──────────────────────────┐
│TCP Header│   Email Message          │
└──────────┴──────────────────────────┘

Internet Layer (IP adds header):
┌──────────┬──────────┬──────────────────────────┐
│IP Header │TCP Header│   Email Message          │
└──────────┴──────────┴──────────────────────────┘

Network Access Layer (Ethernet adds header and trailer):
┌─────────┬──────────┬──────────┬──────────────────────────┬─────────┐
│Ethernet │IP Header │TCP Header│   Email Message          │Ethernet │
│ Header  │          │          │                          │ Trailer │
└─────────┴──────────┴──────────┴──────────────────────────┴─────────┘
                                ↓
                    Transmitted as bits on wire
```

At the receiver, headers are stripped off (de-encapsulation) as data moves up the stack.

**Each header contains:**

```
Ethernet Header:
  • Source MAC address
  • Destination MAC address
  • Type (e.g., "this contains IP")

IP Header:
  • Source IP address
  • Destination IP address
  • Protocol (e.g., "this contains TCP")
  • TTL (Time To Live)

TCP Header:
  • Source port
  • Destination port
  • Sequence number
  • Acknowledgment number
  • Flags (SYN, ACK, FIN)

Application Data:
  • The actual message (email content)
```

---

### Benefits of Layering

**1. Modularity:**
```
Want to upgrade from IPv4 to IPv6?
  → Only change Internet Layer
  → Transport and Application layers unchanged
  → Applications don't even know!
```

**2. Abstraction:**
```
Application developer writes HTTP code
  → Doesn't care if network uses Ethernet or WiFi
  → Doesn't care if using TCP or UDP (well, usually TCP)
  → Just calls socket API
```

**3. Interoperability:**
```
Web browser (Chrome) ↔ Web server (Apache)
  • Different applications
  • Different operating systems
  • But both speak HTTP over TCP over IP
  → They can communicate!
```

**4. Debugging:**
```
Website not loading?
  • Check Application Layer: Is HTTP request correct?
  • Check Transport Layer: Is TCP connection established?
  • Check Internet Layer: Can we ping the server?
  • Check Network Access Layer: Is Ethernet cable plugged in?
  
Each layer can be tested independently
```

**5. Innovation:**
```
New application protocol?
  → Build on top of existing Transport Layer
  → No need to reinvent reliability (TCP already does it)

Example: HTTP/3 uses QUIC (new transport protocol)
  → Built on UDP instead of TCP
  → Application layer (HTTP) barely changed
```

---

### TCP/IP vs OSI Model

**OSI (Open Systems Interconnection) - 7 Layers:**

```
┌──────────────────────────┐
│ 7. Application           │ ─┐
├──────────────────────────┤  │
│ 6. Presentation          │  │ Combined in TCP/IP
├──────────────────────────┤  │ Application Layer
│ 5. Session               │ ─┘
├──────────────────────────┤
│ 4. Transport             │ ← TCP/IP Transport Layer
├──────────────────────────┤
│ 3. Network               │ ← TCP/IP Internet Layer
├──────────────────────────┤
│ 2. Data Link             │ ─┐ TCP/IP Network Access
├──────────────────────────┤  │ Layer (combined)
│ 1. Physical              │ ─┘
└──────────────────────────┘
```

**Why two models?**

```
OSI (1970s-1980s):
  • Designed by committee (ISO)
  • Comprehensive, theoretical
  • Never widely implemented

TCP/IP (1970s-present):
  • Designed by DARPA for ARPANET
  • Pragmatic, battle-tested
  • What the Internet actually uses
```

**Which to learn?**

```
In practice:
  • Internet uses TCP/IP (4 layers)
  • But people reference OSI concepts
  • Example: "Layer 2 switch" (OSI Data Link)
  • Example: "Layer 3 routing" (OSI Network)

Our approach:
  • Focus on TCP/IP (how things actually work)
  • Reference OSI when helpful for terminology
```

---

### Layer Responsibilities Summary

| Layer | Also Called | Protocols | Addressing | Device | Function |
|-------|-------------|-----------|------------|--------|----------|
| **Application** | Layer 7 (OSI) | HTTP, SMTP, DNS, FTP | URLs, email addresses | End systems | User services |
| **Transport** | Layer 4 (OSI) | TCP, UDP | Port numbers (80, 443) | End systems | Reliable delivery |
| **Internet** | Network Layer, Layer 3 | IP, ICMP, ARP | IP addresses (192.168.1.1) | Routers | Routing between networks |
| **Network Access** | Link/Physical, Layer 1-2 | Ethernet, WiFi | MAC addresses (AA:BB:CC...) | Switches, NICs | Local network transmission |

---

## Summary: The Foundation

**Key concepts:**

1. **Networks connect autonomous computers** to share resources and communicate
2. **Goals:** Resource sharing, communication, scalability, reliability
3. **Components:** Nodes (end systems), links (wired/wireless), network devices (switches/routers)
4. **Scale:** PAN → LAN → MAN → WAN (Internet)
5. **Performance:** Bandwidth, latency, jitter, packet loss all matter
6. **Protocols** define the rules of communication
7. **Layering** manages complexity

**The big picture:**

```
Your computer
     ↓ (application creates data)
  Protocols package data
     ↓ (network delivers data)
  Remote computer
     ↓ (application receives data)
  
All enabled by:
  • Physical infrastructure (cables, routers)
  • Agreed-upon rules (protocols)
  • Layered architecture (managing complexity)
```

**What's next:**

- OSI and TCP/IP models in detail
- Physical and data link layers
- Network layer and routing
- Transport layer and reliability
- Application layer protocols

Networking is the invisible infrastructure that connects our digital world. Understanding it means understanding how the Internet—and all modern computing—actually works.