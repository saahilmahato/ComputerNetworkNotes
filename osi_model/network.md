# Network Layer (Layer 3)

---

## **1) Concept Snapshot**

**Definition:**
The Network Layer is responsible for logical addressing, routing, and forwarding of packets across multiple networks from source to destination, providing host-to-host communication through internetworking devices (routers) that make path selection decisions based on routing protocols and algorithms.

**Purpose:**
- **Logical addressing:** Assign unique addresses (IP addresses) to identify hosts globally
- **Routing:** Determine the best path from source to destination across multiple networks
- **Forwarding:** Move packets from input interface to appropriate output interface
- **Fragmentation & Reassembly:** Break large packets into smaller pieces if needed
- **Internetworking:** Connect different types of networks (Ethernet, Wi-Fi, etc.)
- **Traffic control:** Manage congestion, QoS, and network performance

**Key insight:** This is the layer that creates the **"internet"** — it connects separate networks into one interconnected network. Without Layer 3, you'd only be able to communicate with devices on your local network (Layer 2). Layer 3 enables global communication.

---

## **2) Mental Model**

**Real-world analogy:**
Think of the postal system delivering mail between cities:

- **MAC Address (Layer 2):** Your apartment number within a building (local, changes at each hop)
- **IP Address (Layer 3):** Your street address including city/country (global, stays the same end-to-end)
- **Router:** Post office that reads the destination city and forwards mail to the next post office
- **Routing table:** Map showing which direction to send mail for each destination city
- **Hop:** Each post office the letter passes through

**Sending a letter from New York to Tokyo:**
1. Local post office (first router)
2. Regional hub (intermediate router)
3. International hub (border router)
4. Tokyo regional hub (intermediate router)
5. Local Tokyo post office (last router)
6. Delivery to building (destination network)

Each post office only needs to know the **next hop**, not the complete path.

**Visual intuition:**
```
HOST A (Source)                                 HOST B (Destination)
192.168.1.10                                    203.0.113.50
     |                                                ^
     | Layer 3: Creates packet with                  |
     | SRC IP: 192.168.1.10                          |
     | DST IP: 203.0.113.50                          |
     ↓                                                |
ROUTER 1 ────────────────────────────────────────────┘
(reads DST IP,        |           |           |
 checks routing table, ↓           ↓           ↓
 forwards to next hop) ROUTER 2  ROUTER 3  ROUTER 4
                           |
                           ↓
                       ROUTER 5 → HOST B

IP PACKET HEADER STAYS THE SAME (end-to-end)
But Layer 2 frame changes at EVERY HOP (hop-by-hop)
```

**Simplified story:**
When you visit a website, your computer creates a packet with your IP address (source) and the website's IP address (destination). This packet travels through dozens of routers, each one reading the destination IP and consulting its routing table: "Which of my neighbors is closest to this destination?" The packet hops from router to router until it reaches the destination network, then the final router delivers it to the destination host.

---

## **3) Layer Context**

**Position in OSI/TCP-IP stack:**
```
OSI Model              TCP/IP Model
─────────────          ────────────
Application (L7)   ┐
Presentation (L6)  ├──→ Application
Session (L5)       ┘
Transport (L4)     ───→ Transport
Network (L3)       ───→ Internet     ← WE ARE HERE
Data Link (L2)     ┐
Physical (L1)      ├──→ Network Access
                   ┘
```

**Who talks to it:**
- **Above:** Transport Layer (TCP/UDP segments)
- **Below:** Data Link Layer (Ethernet, Wi-Fi frames)
- **Peers:** Other routers (routing protocol exchanges)

**Key protocols at this layer:**

```
╔════════════════════════════════════════════════════════╗
║ PRIMARY NETWORK LAYER PROTOCOLS                        ║
╠════════════════════════════════════════════════════════╣
║ IPv4 (Internet Protocol version 4)                     ║
║ • 32-bit addresses (4.3 billion)                       ║
║ • Dominant protocol (~95% of internet)                 ║
║ • Addresses exhausted, uses NAT                        ║
╠════════════════════════════════════════════════════════╣
║ IPv6 (Internet Protocol version 6)                     ║
║ • 128-bit addresses (340 undecillion)                  ║
║ • Slowly growing adoption (~40% of traffic)            ║
║ • Simplified header, built-in security                 ║
╠════════════════════════════════════════════════════════╣
║ ICMP (Internet Control Message Protocol)               ║
║ • Error reporting, diagnostics                         ║
║ • Used by ping, traceroute                             ║
╠════════════════════════════════════════════════════════╣
║ ROUTING PROTOCOLS                                      ║
║ • RIP, OSPF, BGP (how routers learn paths)             ║
║ • Exchange routing information                         ║
╠════════════════════════════════════════════════════════╣
║ ARP (Address Resolution Protocol)                      ║
║ • Maps IP addresses to MAC addresses                   ║
║ • Technically between L2 and L3                        ║
╚════════════════════════════════════════════════════════╝
```

**PDU (Protocol Data Unit):**
- Layer 3 PDU = **Packet** (or **Datagram**)

**Two fundamental functions:**
1. **Control Plane:** Routing (build routing tables, exchange info between routers)
2. **Data Plane:** Forwarding (move packets through router based on routing table)

---

## **4) Mechanics (How It Actually Works)**

### **IP ADDRESSING (IPv4)**

**IPv4 Address Structure:**

```
32-bit address, written in dotted decimal notation:

Binary:    11000000.10101000.00000001.00001010
Decimal:   192     .168     .1       .10

Each octet: 0-255 (8 bits)
Total addresses: 2^32 = 4,294,967,296

ADDRESS CLASSES (Historical, mostly obsolete):

Class A: 0.0.0.0 - 127.255.255.255
• First bit: 0
• Network: 8 bits, Host: 24 bits
• 126 networks, 16 million hosts each
• Example: 10.0.0.0

Class B: 128.0.0.0 - 191.255.255.255
• First bits: 10
• Network: 16 bits, Host: 16 bits
• 16,384 networks, 65,536 hosts each
• Example: 172.16.0.0

Class C: 192.0.0.0 - 223.255.255.255
• First bits: 110
• Network: 24 bits, Host: 8 bits
• 2 million networks, 254 hosts each
• Example: 192.168.1.0

Class D: 224.0.0.0 - 239.255.255.255
• Multicast

Class E: 240.0.0.0 - 255.255.255.255
• Reserved/Experimental
```

---

**CIDR (Classless Inter-Domain Routing):**

Modern addressing uses CIDR notation:

```
192.168.1.0/24

Breaking it down:
192.168.1.0  → Network address
/24          → Subnet mask (24 bits for network, 8 for host)

Subnet mask in binary:
11111111.11111111.11111111.00000000
255     .255     .255     .0

NUMBER OF HOSTS:
/24 → 2^(32-24) = 2^8 = 256 addresses
     - 1 (network address: 192.168.1.0)
     - 1 (broadcast address: 192.168.1.255)
     = 254 usable host addresses

COMMON CIDR BLOCKS:

/32 → 1 address      (host route, single IP)
/31 → 2 addresses    (point-to-point links)
/30 → 4 addresses    (2 usable, point-to-point)
/29 → 8 addresses    (6 usable)
/28 → 16 addresses   (14 usable)
/27 → 32 addresses   (30 usable)
/26 → 64 addresses   (62 usable)
/25 → 128 addresses  (126 usable)
/24 → 256 addresses  (254 usable)  ← Very common
/23 → 512 addresses  (510 usable)
/22 → 1024 addresses (1022 usable)
/21 → 2048 addresses (2046 usable)
/20 → 4096 addresses (4094 usable)
/16 → 65,536 addresses (65,534 usable)  ← Large networks
/8  → 16,777,216 addresses (huge)
```

---

**SPECIAL IP ADDRESS RANGES:**

```
PRIVATE (RFC 1918) - Not routable on internet:
┌─────────────────────────────────────────────┐
│ 10.0.0.0/8                                  │
│ • 10.0.0.0 - 10.255.255.255                 │
│ • 16,777,216 addresses                      │
│ • Large enterprises                         │
├─────────────────────────────────────────────┤
│ 172.16.0.0/12                               │
│ • 172.16.0.0 - 172.31.255.255               │
│ • 1,048,576 addresses                       │
│ • Medium enterprises                        │
├─────────────────────────────────────────────┤
│ 192.168.0.0/16                              │
│ • 192.168.0.0 - 192.168.255.255             │
│ • 65,536 addresses                          │
│ • Home networks, small businesses           │
└─────────────────────────────────────────────┘

LOOPBACK:
127.0.0.0/8
• 127.0.0.1 (localhost)
• Traffic to self, never leaves host

LINK-LOCAL (APIPA):
169.254.0.0/16
• Auto-assigned when DHCP fails
• Only works on local network segment

MULTICAST:
224.0.0.0/4
• One-to-many communication
• 224.0.0.1 = All hosts on subnet
• 224.0.0.2 = All routers on subnet

BROADCAST:
255.255.255.255
• Limited broadcast (local network only)
• x.x.x.255 = Directed broadcast for network x.x.x.0/24

SPECIAL:
0.0.0.0
• "This network" or "any address"
• Used in routing tables as default route
```

---

**SUBNETTING EXAMPLE:**

```
Given: 192.168.1.0/24
Task: Divide into 4 equal subnets

Step 1: Determine new mask
4 subnets = 2^2, need 2 more bits
/24 + 2 = /26

Step 2: Calculate subnet size
/26 → 2^(32-26) = 64 addresses per subnet

Step 3: List subnets
Subnet 1: 192.168.1.0/26    (0-63)
  Network:   192.168.1.0
  First host: 192.168.1.1
  Last host:  192.168.1.62
  Broadcast: 192.168.1.63

Subnet 2: 192.168.1.64/26   (64-127)
  Network:   192.168.1.64
  First host: 192.168.1.65
  Last host:  192.168.1.126
  Broadcast: 192.168.1.127

Subnet 3: 192.168.1.128/26  (128-191)
  Network:   192.168.1.128
  First host: 192.168.1.129
  Last host:  192.168.1.190
  Broadcast: 192.168.1.191

Subnet 4: 192.168.1.192/26  (192-255)
  Network:   192.168.1.192
  First host: 192.168.1.193
  Last host:  192.168.1.254
  Broadcast: 192.168.1.255

Each subnet has 62 usable host addresses
```

---

### **IPv4 PACKET STRUCTURE:**

```
IPv4 HEADER (20-60 bytes):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Key fields explained:**

```
VERSION (4 bits):
• IP version (4 for IPv4, 6 for IPv6)

IHL - INTERNET HEADER LENGTH (4 bits):
• Header length in 32-bit words
• Minimum: 5 (20 bytes), Maximum: 15 (60 bytes)
• Needed because Options field is variable length

TYPE OF SERVICE / DSCP (8 bits):
• Priority/QoS markings
• Modern: DiffServ Code Point (DSCP)
• Routers can prioritize packets (VoIP > email)

TOTAL LENGTH (16 bits):
• Entire packet size (header + data)
• Maximum: 65,535 bytes
• Typical: ~1500 bytes (Ethernet MTU)

IDENTIFICATION (16 bits):
• Unique ID for each packet
• Used for reassembling fragments
• All fragments of same packet share ID

FLAGS (3 bits):
• Bit 0: Reserved (always 0)
• Bit 1: Don't Fragment (DF)
  - If set and packet too large → ICMP error
  - Used for Path MTU Discovery
• Bit 2: More Fragments (MF)
  - 1 = more fragments coming
  - 0 = last fragment (or not fragmented)

FRAGMENT OFFSET (13 bits):
• Position of this fragment in original packet
• In units of 8 bytes
• Allows reassembly in correct order

TIME TO LIVE - TTL (8 bits):
• Maximum number of hops (routers)
• Decremented by 1 at each router
• When TTL = 0, packet discarded
• Prevents infinite loops
• Typical initial value: 64 or 128

PROTOCOL (8 bits):
• Identifies upper-layer protocol
• 1 = ICMP, 6 = TCP, 17 = UDP
• Tells receiver how to process payload

HEADER CHECKSUM (16 bits):
• Error detection for header only (not data)
• Recalculated at each hop (because TTL changes)

SOURCE ADDRESS (32 bits):
• IP address of sender
• Stays the same end-to-end (unless NAT)

DESTINATION ADDRESS (32 bits):
• IP address of receiver
• Stays the same end-to-end (unless NAT)

OPTIONS (variable, 0-40 bytes):
• Rarely used
• Examples: Source routing, timestamp
• Padding added to make header multiple of 32 bits
```

---

### **ROUTING FUNDAMENTALS:**

**Routing vs Forwarding:**

```
ROUTING (Control Plane):
• Building routing tables
• Exchange information with other routers
• Use routing protocols (RIP, OSPF, BGP)
• Done by routing software/protocols
• Happens BEFORE packet arrives

FORWARDING (Data Plane):
• Moving packet from input to output interface
• Lookup destination IP in routing table
• Send packet to next hop
• Done by router hardware
• Happens WHEN packet arrives
```

---

**ROUTING TABLE:**

```
EXAMPLE ROUTING TABLE:

Destination      Netmask         Gateway         Interface   Metric
───────────────────────────────────────────────────────────────────
0.0.0.0          0.0.0.0         192.168.1.1     eth0        100
192.168.1.0      255.255.255.0   0.0.0.0         eth0        0
10.0.0.0         255.255.255.0   0.0.0.0         eth1        0
172.16.0.0       255.255.0.0     192.168.1.254   eth0        50

READING THE TABLE:

Entry 1 (Default Route):
Destination: 0.0.0.0/0 (matches everything)
Gateway: 192.168.1.1 (send here for unknown destinations)
Use: Catch-all for internet traffic

Entry 2 (Direct Connection):
Destination: 192.168.1.0/24
Gateway: 0.0.0.0 (directly connected, no gateway needed)
Interface: eth0
Use: Local network traffic

Entry 3 (Direct Connection):
Destination: 10.0.0.0/24
Gateway: 0.0.0.0
Interface: eth1
Use: Another directly connected network

Entry 4 (Remote Network):
Destination: 172.16.0.0/16
Gateway: 192.168.1.254 (next hop router)
Use: Reach remote network via this gateway

LOOKUP ALGORITHM:
1. Find most specific match (longest prefix match)
2. If multiple matches, use highest priority/lowest metric
3. Forward to gateway (or directly if gateway = 0.0.0.0)
4. If no match, use default route (0.0.0.0/0)
5. If no default route, drop packet (send ICMP unreachable)
```

---

**LONGEST PREFIX MATCH:**

```
Packet destination: 192.168.1.50

Routing table:
1. 0.0.0.0/0          → Matches (but least specific)
2. 192.168.0.0/16     → Matches
3. 192.168.1.0/24     → Matches (MOST SPECIFIC) ← USE THIS
4. 192.168.1.0/25     → Doesn't match

Router uses entry #3 (longest prefix match)

WHY?
More specific routes take precedence
Allows exception routes within larger blocks

Example:
- Send most of 192.168.0.0/16 to Router A
- But send 192.168.1.0/24 to Router B (exception)
```

---

### **ROUTING PROTOCOLS:**

**Classification:**

```
╔════════════════════════════════════════════════════════╗
║ INTERIOR GATEWAY PROTOCOLS (IGP)                       ║
║ Within a single organization/AS                        ║
╠════════════════════════════════════════════════════════╣
║ DISTANCE VECTOR                                        ║
║ • RIP (Routing Information Protocol)                   ║
║   - Uses hop count (max 15)                            ║
║   - Simple, slow convergence                           ║
║   - Sends entire routing table periodically            ║
║                                                        ║
║ LINK STATE                                             ║
║ • OSPF (Open Shortest Path First)                      ║
║   - Uses Dijkstra's algorithm                          ║
║   - Fast convergence, scalable                         ║
║   - Maintains topology database                        ║
║ • IS-IS (Intermediate System to Intermediate System)   ║
║   - Similar to OSPF                                    ║
║   - Used by ISPs                                       ║
╚════════════════════════════════════════════════════════╝

╔════════════════════════════════════════════════════════╗
║ EXTERIOR GATEWAY PROTOCOLS (EGP)                       ║
║ Between different organizations/AS                     ║
╠════════════════════════════════════════════════════════╣
║ PATH VECTOR                                            ║
║ • BGP (Border Gateway Protocol)                        ║
║   - De facto internet routing protocol                 ║
║   - Policy-based routing                               ║
║   - Prevents loops with AS path                        ║
║   - Connects ISPs, major networks                      ║
╚════════════════════════════════════════════════════════╝
```

---

**RIP (Routing Information Protocol):**

```
CHARACTERISTICS:
• Distance vector protocol
• Metric: Hop count (number of routers to destination)
• Maximum: 15 hops (16 = unreachable/infinity)
• Updates: Every 30 seconds
• Algorithm: Bellman-Ford

HOW IT WORKS:

Router A's table:
Network      Next Hop    Hops
10.0.1.0/24  Direct      0
10.0.2.0/24  Router B    1
10.0.3.0/24  Router B    2

Router A sends to neighbors: "I can reach these networks"

Router B receives, adds 1 to all hop counts:
"Router A can reach 10.0.1.0 in 0 hops"
→ "I can reach 10.0.1.0 via Router A in 1 hop"

PROBLEMS:
• Count to infinity (routing loops)
• Slow convergence (minutes)
• Not suitable for large networks (15 hop limit)
• Wastes bandwidth (periodic full table updates)

SOLUTIONS:
• Split horizon: Don't advertise route back to source
• Poison reverse: Advertise unreachable routes as 16 hops
• Hold-down timers: Wait before accepting worse routes

VERSIONS:
• RIPv1: Classful, no authentication
• RIPv2: CIDR support, authentication, multicast updates
• RIPng: For IPv6
```

---

**OSPF (Open Shortest Path First):**

```
CHARACTERISTICS:
• Link-state protocol
• Metric: Cost (based on bandwidth)
• Algorithm: Dijkstra's shortest path
• Fast convergence (seconds)
• Scalable (hierarchical with areas)

HOW IT WORKS:

Phase 1 - NEIGHBOR DISCOVERY:
Routers send Hello packets (multicast 224.0.0.5)
Establish adjacencies with neighbors

Phase 2 - LINK STATE DATABASE:
Each router builds map of entire network topology
Exchange Link State Advertisements (LSAs)
All routers have identical database

Phase 3 - SPF CALCULATION:
Run Dijkstra's algorithm
Calculate shortest path to each destination
Build routing table

Phase 4 - INCREMENTAL UPDATES:
Only send updates when topology changes
Convergence in seconds (not minutes like RIP)

AREAS:
┌─────────────────────────────────────┐
│          AREA 0 (Backbone)          │
│         ┌──────────────┐            │
│         │   ABR        │            │
│         └──────┬───────┘            │
└────────────────┼────────────────────┘
                 │
       ┌─────────┴─────────┐
       │                   │
┌──────┴─────┐      ┌──────┴─────┐
│  AREA 1    │      │  AREA 2    │
│  (Stub)    │      │ (Standard) │
└────────────┘      └────────────┘

ABR = Area Border Router
Reduces LSA flooding
Improves scalability

COST CALCULATION:
Cost = Reference Bandwidth / Interface Bandwidth
Reference (default) = 100 Mbps

Examples:
• 100 Mbps (Fast Ethernet): 100/100 = 1
• 10 Mbps (Ethernet): 100/10 = 10
• 1 Gbps (Gigabit): 100/1000 = 1 (minimum)

Lower cost = preferred path
```

---

**BGP (Border Gateway Protocol):**

```
CHARACTERISTICS:
• Path vector protocol
• Used between Autonomous Systems (AS)
• Policy-based (not just shortest path)
• Extremely scalable (~900,000 routes in global table)
• TCP-based (port 179)

AUTONOMOUS SYSTEM (AS):
• Collection of networks under single administration
• AS Number (ASN): 16-bit (1-65535) or 32-bit
• Examples:
  - AS15169: Google
  - AS32934: Facebook
  - AS7018: AT&T

HOW IT WORKS:

BGP Speaker announces:
"I can reach 8.8.8.0/24 via AS-PATH: 15169"

Neighbor receives, adds own AS:
"I can reach 8.8.8.0/24 via AS-PATH: 7018 15169"

Next neighbor:
"I can reach 8.8.8.0/24 via AS-PATH: 3356 7018 15169"

LOOP PREVENTION:
If router sees its own AS in path → Reject
(Can't loop because path shows all ASes traversed)

ATTRIBUTES (Path selection):
1. Weight (Cisco-specific, local preference)
2. Local Preference (within AS)
3. Locally originated routes
4. AS-PATH length (shorter = better)
5. Origin type
6. MED (Multi-Exit Discriminator)
7. eBGP over iBGP
8. Lowest IGP cost to next hop
9. Oldest route
10. Lowest Router ID

POLICY EXAMPLES:
• Prefer customer routes over peer routes
• Avoid certain countries/ASes
• Load balance across multiple links
• Implement traffic engineering

BGP TYPES:
• eBGP (External): Between different ASes
• iBGP (Internal): Within same AS
```

---

### **FRAGMENTATION:**

```
WHY FRAGMENTATION?
Different networks have different MTU (Maximum Transmission Unit)
• Ethernet: 1500 bytes
• Wi-Fi: 1500 bytes
• PPPoE: 1492 bytes
• VPN tunnels: Less (overhead from encryption)

PROBLEM:
Packet = 3000 bytes
Network MTU = 1500 bytes
→ Packet too large!

SOLUTION 1: FRAGMENTATION (IPv4)

Original packet (3000 bytes):
┌────────────────────────────────────────┐
│ IP Header (20) │ Data (2980)           │
└────────────────────────────────────────┘

Fragment into 2 packets:

Fragment 1:
┌────────────────────────────────────────┐
│ IP Header (20) │ Data (1480)           │
│ ID=12345, MF=1, Offset=0               │
└────────────────────────────────────────┘

Fragment 2:
┌────────────────────────────────────────┐
│ IP Header (20) │ Data (1500)           │
│ ID=12345, MF=0, Offset=185             │
└────────────────────────────────────────┘

Both share same ID (12345)
Offset in units of 8 bytes: 1480/8 = 185
MF (More Fragments): 1 = more coming, 0 = last

REASSEMBLY:
Destination host:
1. Receives fragments
2. Groups by ID
3. Sorts by offset
4. Waits for MF=0 (last fragment)
5. Reconstructs original packet

PROBLEMS WITH FRAGMENTATION:
• Performance: Extra processing
• Lost fragment = entire packet lost (must retransmit all)
• Security: Firewall evasion attacks
• NAT issues: Can't track connections in fragments

SOLUTION 2: PATH MTU DISCOVERY (Modern)

1. Send packet with DF (Don't Fragment) flag set
2. If too large, router sends ICMP "Fragmentation Needed"
3. Sender reduces packet size
4. Eventually finds working MTU
5. No fragmentation needed!

Most modern systems use Path MTU Discovery
Fragmentation is rare on modern internet
```

---

### **NAT (Network Address Translation):**

```
PURPOSE:
• Conserve IPv4 addresses (address exhaustion problem)
• Allow many private IPs to share one public IP
• Security through obscurity (hide internal network)

HOW IT WORKS:

Inside network: 192.168.1.0/24 (private)
Outside: Internet (public IPs)
NAT router: Public IP = 203.0.113.10

OUTBOUND PACKET:

Before NAT:
SRC: 192.168.1.50:54321
DST: 93.184.216.34:80

NAT Router translates:
SRC: 203.0.113.10:12345 ← Changed!
DST: 93.184.216.34:80

NAT table entry:
Internal             External           
192.168.1.50:54321 ↔ 203.0.113.10:12345

INBOUND RESPONSE:

Server responds to: 203.0.113.10:12345

NAT Router looks up in table:
External 203.0.113.10:12345 → Internal 192.168.1.50:54321

After NAT:
SRC: 93.184.216.34:80
DST: 192.168.1.50:54321 ← Translated back!

NAT TYPES:

╔════════════════════════════════════════════════════════╗
║ STATIC NAT (One-to-one)                                ║
║ 192.168.1.10 always maps to 203.0.113.20               ║
║ Used for servers                                       ║
╠════════════════════════════════════════════════════════╣
║ DYNAMIC NAT (Many-to-many)                             ║
║ Pool of public IPs, assigned as needed                 ║
║ 192.168.1.x maps to any available 203.0.113.x          ║
╠════════════════════════════════════════════════════════╣
║ PAT (Port Address Translation) / NAT Overload          ║
║ Many private IPs → One public IP (different ports)     ║
║ Most common for home/office                            ║
║ 192.168.1.10:1234 → 203.0.113.10:5001                  ║
║ 192.168.1.20:1234 → 203.0.113.10:5002                  ║
║ Uses port numbers to multiplex                         ║
╚════════════════════════════════════════════════════════╝

LIMITATIONS:
✗ Breaks end-to-end connectivity
✗ Some protocols don't work (FTP active mode, SIP, P2P)
✗ Can't host servers without port forwarding
✗ Complicates troubleshooting
✗ Not a security feature (not a firewall!)

PORT FORWARDING (workaround for servers):
External: 203.0.113.10:80 → Internal: 192.168.1.100:80
Allows hosting web server behind NAT
```

---

### **ICMP (Internet Control Message Protocol):**

```
PURPOSE:
• Error reporting
• Network diagnostics
• Communication between network devices

ICMP MESSAGE STRUCTURE:
┌────────────────────────────────────────┐
│ Type (8 bits) │ Code (8 bits)          │
├────────────────────────────────────────┤
│ Checksum (16 bits)                     │
├────────────────────────────────────────┤
│ Rest of Header (varies)                │
├────────────────────────────────────────┤
│ Data (original packet that caused msg) │
└────────────────────────────────────────┘

COMMON ICMP TYPES:

Type 0 - Echo Reply (ping response):
"I'm here!"

Type 3 - Destination Unreachable:
Code 0: Network unreachable (no route)
Code 1: Host unreachable (ARP failed)
Code 2: Protocol unreachable (service not running)
Code 3: Port unreachable (no one listening)
Code 4: Fragmentation needed but DF set (Path MTU Discovery)

Type 5 - Redirect:
"Use a different router for this destination"

Type 8 - Echo Request (ping):
"Are you there?"

Type 11 - Time Exceeded:
Code 0: TTL expired in transit (traceroute uses this)
Code 1: Fragment reassembly timeout

PING (DIAGNOSTIC TOOL):

How it works:
1. Send ICMP Echo Request (Type 8)
2. Destination sends Echo Reply (Type 0)
3. Measure round-trip time

$ ping example.com
PING example.com (93.184.216.34): 56 data bytes
64 bytes from 93.184.216.34: icmp_seq=0 ttl=56 time=15.2 ms
64 bytes from 93.184.216.34: icmp_seq=1 ttl=56 time=14.8 ms
64 bytes from 93.184.216.34: icmp_seq=2 ttl=56 time=15.1 ms

Information gathered:
• Host is reachable
• RTT (round-trip time): ~15 ms
• TTL: 56 (started at 64, crossed 8 routers)
• No packet loss

TRACEROUTE (PATH DISCOVERY):

How it works:
1. Send packet with TTL=1
   → First router decrements to 0, sends ICMP Time Exceeded
2. Send packet with TTL=2
   → Second router decrements to 0, sends ICMP Time Exceeded
3. Continue until destination reached

$ traceroute example.com
1  192.168.1.1 (192.168.1.1)  1.234 ms  (home router)
2  10.0.0.1 (10.0.0.1)  5.678 ms  (ISP gateway)
3  203.0.113.1 (203.0.113.1)  15.2 ms  (ISP backbone)
...
10  93.184.216.34 (93.184.216.34)  45.6 ms  (destination)

Shows complete path and latency at each hop

SECURITY IMPLICATIONS:
• Many firewalls block ICMP (breaks ping/traceroute)
• ICMP can be used for attacks:
  - Ping flood (DDoS)
  - Smurf attack (amplification)
  - ICMP tunneling (data exfiltration)
• Best practice: Rate-limit, don't block entirely
```

---

### **ARP (Address Resolution Protocol):**

```
PURPOSE:
Map IP addresses (Layer 3) to MAC addresses (Layer 2)

PROBLEM:
• Router knows destination IP: 192.168.1.50
• But Ethernet needs MAC address to deliver frame
• How to find MAC address for 192.168.1.50?

SOLUTION: ARP

ARP REQUEST (Broadcast):
┌────────────────────────────────────────┐
│ Ethernet Destination: FF:FF:FF:FF:FF:FF│ (broadcast)
│ Ethernet Source: AA:BB:CC:DD:EE:FF     │
│                                        │
│ ARP Opcode: Request (1)                │
│ Sender MAC: AA:BB:CC:DD:EE:FF          │
│ Sender IP: 192.168.1.1                 │
│ Target MAC: 00:00:00:00:00:00 (unknown)│
│ Target IP: 192.168.1.50                │
└────────────────────────────────────────┘

Broadcast to everyone on network:
"Who has IP 192.168.1.50? Tell 192.168.1.1"

ARP REPLY (Unicast):
┌────────────────────────────────────────┐
│ Ethernet Destination: AA:BB:CC:DD:EE:FF│ (unicast)
│ Ethernet Source: 11:22:33:44:55:66     │
│                                        │
│ ARP Opcode: Reply (2)                  │
│ Sender MAC: 11:22:33:44:55:66          │
│ Sender IP: 192.168.1.50                │
│ Target MAC: AA:BB:CC:DD:EE:FF          │
│ Target IP: 192.168.1.1                 │
└────────────────────────────────────────┘

Host 192.168.1.50 responds:
"I have IP 192.168.1.50, my MAC is 11:22:33:44:55:66"

ARP CACHE:
Router stores mapping:
IP: 192.168.1.50 → MAC: 11:22:33:44:55:66
Cached for ~5 minutes (OS-dependent)

Next time: Use cached MAC, no ARP needed

VIEW ARP CACHE:
$ arp -a
? (192.168.1.1) at aa:bb:cc:dd:ee:ff on en0 ifscope [ethernet]
? (192.168.1.50) at 11:22:33:44:55:66 on en0 ifscope [ethernet]
? (192.168.1.100) at 22:33:44:55:66:77 on en0 ifscope [ethernet]

GRATUITOUS ARP:
Host announces own IP-to-MAC mapping
Purpose:
• Detect IP conflicts
• Update caches when MAC changes
• Announce presence on network

Format: ARP request where sender IP = target IP

PROXY ARP:
Router answers ARP requests on behalf of other hosts
Used when hosts on different subnets but no routing configured
Generally discouraged (causes confusion)

SECURITY ISSUES:
• ARP spoofing: Attacker sends fake ARP replies
  - Claim to be gateway (man-in-the-middle)
  - Redirect traffic to attacker
• No authentication in ARP
• Mitigations:
  - Static ARP entries (manual, doesn't scale)
  - Dynamic ARP Inspection (switch feature)
  - Encrypted network (VPN, 802.1X)
```

---

## **5) Key Structures & Components**

### **Router Components:**

```
╔════════════════════════════════════════════════════════╗
║ ROUTER ARCHITECTURE                                    ║
╠════════════════════════════════════════════════════════╣
║ INPUT PORTS                                            ║
║ • Physical layer (receive bits)                        ║
║ • Data link layer (process frames)                     ║
║ • Network layer (lookup, queue)                        ║
╠════════════════════════════════════════════════════════╣
║ SWITCHING FABRIC                                       ║
║ • Moves packets from input to output                   ║
║ • High-speed backplane                                 ║
║ • Crossbar, bus, or shared memory                      ║
╠════════════════════════════════════════════════════════╣
║ OUTPUT PORTS                                           ║
║ • Buffer packets in queues                             ║
║ • Data link layer (create frames)                      ║
║ • Physical layer (transmit bits)                       ║
╠════════════════════════════════════════════════════════╣
║ ROUTING PROCESSOR (Control Plane)                      ║
║ • Run routing protocols                                ║
║ • Build routing tables                                 ║
║ • Handle management (SNMP, CLI)                        ║
╚════════════════════════════════════════════════════════╝

PACKET FORWARDING PROCESS:

1. INPUT PORT:
   ┌─────────────────────────────┐
   │ Receive packet              │
   │ Extract destination IP      │
   │ Lookup in forwarding table  │
   │ Determine output port       │
   └─────────────────────────────┘
              ↓
2. SWITCHING FABRIC:
   ┌─────────────────────────────┐
   │ Transfer packet from input  │
   │ port to output port         │
   └─────────────────────────────┘
              ↓
3. OUTPUT PORT:
   ┌─────────────────────────────┐
   │ Queue packet if busy        │
   │ Decrement TTL               │
   │ Recalculate checksum        │
   │ Encapsulate in new frame    │
   │ Transmit                    │
   └─────────────────────────────┘
```

---

### **Forwarding Table vs Routing Table:**

```
ROUTING TABLE (Control Plane):
• Maintained by routing protocols
• Contains all known routes
• May have multiple paths to same destination
• Administrative distance, metrics, next-hop info
• Updated by routing protocols (OSPF, BGP, etc.)

Example:
Network        Gateway      Metric  Protocol
10.0.0.0/8     10.1.1.1     10      OSPF
10.0.0.0/8     10.2.2.1     20      RIP
172.16.0.0/16  10.1.1.2     5       OSPF
0.0.0.0/0      192.168.1.1  1       Static

FORWARDING TABLE (Data Plane):
• Optimized subset of routing table
• One best route per destination
• Fast lookup (hardware/ASIC)
• Actually used for packet forwarding

Example:
Prefix         Next Hop     Interface
10.0.0.0/8     10.1.1.1     eth0
172.16.0.0/16  10.1.1.2     eth0
0.0.0.0/0      192.168.1.1  eth1

Forwarding table built FROM routing table
Routing protocols update routing table
Routing table updates forwarding table
```

---

### **Administrative Distance:**

```
When multiple routing protocols provide route to same destination,
which one to trust?

ADMINISTRATIVE DISTANCE (AD):
Trustworthiness metric (lower = more trusted)

Protocol           AD
────────────────────────
Connected          0   (directly connected network)
Static             1   (manually configured)
eBGP              20   (external BGP)
EIGRP Internal    90   (Cisco proprietary)
OSPF             110   (most common IGP)
RIP              120   (distance vector)
EIGRP External   170
iBGP             200   (internal BGP)
Unknown          255   (never use)

EXAMPLE:
Route to 10.0.0.0/8 learned via:
• OSPF: Metric 100, AD 110
• RIP: Metric 5, AD 120

Router chooses OSPF (lower AD)
Even though RIP has better metric!

AD determines WHICH protocol
Metric determines WHICH route within that protocol
```

---

## **6) Performance & Tradeoffs**

### **IPv4 vs IPv6:**

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address size** | 32 bits | 128 bits |
| **Address space** | 4.3 billion | 340 undecillion |
| **Notation** | Dotted decimal (192.168.1.1) | Hexadecimal (2001:db8::1) |
| **Header size** | 20-60 bytes (variable) | 40 bytes (fixed) |
| **Header complexity** | 13 fields, options | 8 fields, simpler |
| **Fragmentation** | Routers can fragment | Only source can fragment |
| **Checksum** | Header checksum | No checksum (rely on L2/L4) |
| **Broadcast** | Yes | No (uses multicast) |
| **NAT** | Common (required) | Rare (not needed) |
| **Configuration** | DHCP or manual | SLAAC, DHCPv6, or manual |
| **Security** | IPsec optional | IPsec required (originally, now optional) |
| **QoS** | ToS field | Flow label, traffic class |
| **Adoption** | ~100% | ~40% (growing) |

**IPv6 Address Example:**
```
Full: 2001:0db8:0000:0000:0000:0000:0000:0001
Compressed: 2001:db8::1

Rules:
• Leading zeros in each block can be omitted
• Consecutive blocks of zeros can be replaced with ::
• :: can only appear once

Special addresses:
::1/128        → Loopback (like 127.0.0.1)
::/128         → Unspecified (like 0.0.0.0)
fe80::/10      → Link-local (like 169.254.0.0/16)
2000::/3       → Global unicast (internet)
ff00::/8       → Multicast
```

---

### **Routing Protocol Comparison:**

| Feature | RIP | OSPF | BGP |
|---------|-----|------|-----|
| **Type** | Distance vector | Link state | Path vector |
| **Metric** | Hop count | Cost (bandwidth) | Path attributes |
| **Max hops** | 15 | Unlimited | Unlimited |
| **Convergence** | Slow (minutes) | Fast (seconds) | Slow (minutes) |
| **Scalability** | Small (< 15 hops) | Large (hierarchical) | Internet-scale |
| **Updates** | Periodic (30s) | Triggered | Triggered |
| **Algorithm** | Bellman-Ford | Dijkstra | Best path selection |
| **CPU/Memory** | Low | Medium-High | Very High |
| **Complexity** | Simple | Medium | Complex |
| **Use case** | Small networks | Enterprise | Internet backbone |

---

### **Routing Overhead:**

```
BANDWIDTH CONSUMPTION:

RIP:
• Full routing table every 30 seconds
• 100 routes × 20 bytes/route = 2 KB every 30s
• Small for modern networks, but wasteful

OSPF:
• Hello packets every 10 seconds (small)
• LSAs only when topology changes
• Initial sync: Large database exchange
• Steady state: Minimal traffic

BGP:
• TCP keepalives every 60 seconds (tiny)
• Full table on session start (~900K routes = ~30 MB)
• Incremental updates only (efficient)
• Memory intensive (stores full internet routing table)

CPU USAGE:

RIP:
• Low (simple algorithm)
• Process received updates
• No complex calculations

OSPF:
• Medium-High
• SPF calculation (Dijkstra) on topology change
• Scales with number of routers in area
• Can use significant CPU on large networks

BGP:
• High
• Complex path selection algorithm
• Thousands of BGP peers
• Hundreds of thousands of routes
```

---

## **7) Failure Modes**

### **What breaks at Network Layer:**

```
╔════════════════════════════════════════════════════════╗
║ 1. NO ROUTE TO HOST                                    ║
╚════════════════════════════════════════════════════════╝

Symptom: "No route to host" or ICMP Destination Unreachable

Causes:
• No route in routing table for destination
• Destination network down/disconnected
• Firewall dropping packets
• Wrong subnet mask (thinks destination is local)

Example:
$ ping 10.5.5.5
PING 10.5.5.5: 56 data bytes
ping: sendto: No route to host

Debug:
$ ip route get 10.5.5.5
RTNETLINK answers: Network is unreachable

$ ip route show
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link

→ No route to 10.5.5.0/24, add route or fix gateway

╔════════════════════════════════════════════════════════╗
║ 2. ROUTING LOOPS                                       ║
╚════════════════════════════════════════════════════════╝

Problem: Packet circulates between routers forever

Scenario:
Router A: "Send 10.0.0.0/8 to Router B"
Router B: "Send 10.0.0.0/8 to Router C"
Router C: "Send 10.0.0.0/8 to Router A"

Packet: A → B → C → A → B → C → A → ...

Detection: TTL expires

$ traceroute 10.0.0.1
1  10.1.1.1  1 ms
2  10.2.2.1  2 ms
3  10.3.3.1  3 ms
4  10.1.1.1  4 ms  ← Back to router 1!
5  10.2.2.1  5 ms
6  10.3.3.1  6 ms
...
30 10.3.3.1  TTL exceeded

Causes:
• Misconfigured static routes
• Routing protocol misconfiguration
• Temporary during convergence

Prevention:
• Split horizon (RIP)
• TTL (limits damage)
• Proper routing protocol design (BGP AS-path, OSPF areas)

╔════════════════════════════════════════════════════════╗
║ 3. ASYMMETRIC ROUTING                                  ║
╚════════════════════════════════════════════════════════╝

Problem: Outbound and inbound paths differ

Flow:
A → B: via Route 1 (fast, low latency)
B → A: via Route 2 (slow, high latency)

Issues:
• Stateful firewalls may drop return traffic
• Makes troubleshooting difficult
• Performance issues (one direction slow)
• Monitoring shows different paths

Example:
$ traceroute example.com
1  router1  1 ms
2  router2  5 ms
3  router3  10 ms

$ traceroute example.com --reverse  (conceptual)
1  router5  15 ms  ← Different path!
2  router4  8 ms
3  router1  2 ms

Not always a problem, but complicates analysis

╔════════════════════════════════════════════════════════╗
║ 4. BLACK HOLE ROUTING                                  ║
╚════════════════════════════════════════════════════════╝

Problem: Route exists but packets disappear silently

Causes:
• Route points to down interface
• Router drops packets without ICMP
• MTU black hole (Path MTU Discovery failure)
• Firewall silently dropping

Symptoms:
• Ping fails with NO error message
• Connection hangs (TCP SYN never ACKed)
• Some packets work, others don't (MTU-dependent)

Example:
$ ping 10.0.0.1
PING 10.0.0.1: 56 data bytes
[silence... no response, no error]

$ traceroute 10.0.0.1
1  192.168.1.1  1 ms
2  10.1.1.1  5 ms
3  * * *  (packets disappear here)
4  * * *
...

MTU Black Hole:
• Small packets work (ping -s 100 succeeds)
• Large packets fail (ping -s 1400 fails)
• Path MTU Discovery broken
• DF bit set, router can't fragment, doesn't send ICMP

╔════════════════════════════════════════════════════════╗
║ 5. TTL EXPIRATION                                      ║
╚════════════════════════════════════════════════════════╝

Symptom: "TTL exceeded in transit"

Legitimate: Traceroute (intentional)
Problem: Path too long, default TTL insufficient

Packet path: 65 hops
Default TTL: 64
Result: Packet dies 1 hop before destination

Solution:
• Increase TTL (OS setting)
• Fix routing (shouldn't need 65 hops)
• Check for routing loops

Modern networks: TTL 64 or 128 sufficient
If legitimately need more: Network design problem

╔════════════════════════════════════════════════════════╗
║ 6. IP ADDRESS CONFLICTS                                ║
╚════════════════════════════════════════════════════════╝

Problem: Two hosts with same IP address

Symptoms:
• Intermittent connectivity
• "IP address conflict" message
• ARP cache confusion
• Connection to wrong host

Detection:
$ arping 192.168.1.50
ARPING 192.168.1.50
60 bytes from aa:bb:cc:dd:ee:ff (192.168.1.50): index=0 time=1.234 ms
60 bytes from 11:22:33:44:55:66 (192.168.1.50): index=1 time=2.345 ms
→ Two different MACs responding!

Causes:
• Static IP assigned to DHCP range
• Rogue DHCP server
• Manual misconfiguration

╔════════════════════════════════════════════════════════╗
║ 7. NAT TRAVERSAL FAILURES                              ║
╚════════════════════════════════════════════════════════╝

Problem: Protocols don't work through NAT

Affected:
• FTP (active mode - server initiates data connection)
• SIP (VoIP - embedded IP addresses in payload)
• IPsec (ESP mode - encrypts ports NAT needs)
• P2P (BitTorrent, games - need incoming connections)

Solutions:
• ALG (Application Layer Gateway) - NAT understands protocol
• UPnP (Universal Plug and Play) - auto port forwarding
• STUN/TURN (NAT traversal for VoIP)
• Port forwarding (manual configuration)
• IPv6 (no NAT needed)
```

---

## **8) Real-World Usage**

### **1. Home Network (Typical Setup):**

```
NETWORK TOPOLOGY:

Internet
   │
   │ Public IP: 203.0.113.45 (from ISP)
   ↓
┌──────────────────┐
│  Home Router     │
│  (NAT Gateway)   │
└────────┬─────────┘
         │ Private network: 192.168.1.0/24
         │ Router IP: 192.168.1.1
         ↓
    ┌────┴────┬────────┬────────┐
    │         │        │        │
  Laptop    Phone    TV    Printer
  .100      .101     .102   .103

DHCP CONFIGURATION:
Router assigns IPs automatically:
• Range: 192.168.1.100 - 192.168.1.200
• Gateway: 192.168.1.1
• DNS: 8.8.8.8, 8.8.4.4 (Google)
• Lease: 24 hours

NAT OPERATION:
Laptop (192.168.1.100) browsing web:

Outbound:
SRC: 192.168.1.100:54321 → 203.0.113.45:12345
DST: 93.184.216.34:80 (unchanged)

Inbound:
SRC: 93.184.216.34:80 (unchanged)
DST: 203.0.113.45:12345 → 192.168.1.100:54321

NAT Table:
Internal           External
192.168.1.100:54321 ↔ 203.0.113.45:12345
192.168.1.101:44444 ↔ 203.0.113.45:12346
192.168.1.102:33333 ↔ 203.0.113.45:12347

All devices share ONE public IP!
```

---

### **2. Corporate Network (Multi-site):**

```
ENTERPRISE TOPOLOGY:

Headquarters (New York)
Network: 10.1.0.0/16
├─ Servers: 10.1.1.0/24
├─ WiFi: 10.1.10.0/24
└─ Wired: 10.1.20.0/24

Branch Office (London)
Network: 10.2.0.0/16
├─ WiFi: 10.2.10.0/24
└─ Wired: 10.2.20.0/24

Branch Office (Tokyo)
Network: 10.3.0.0/16
├─ WiFi: 10.3.10.0/24
└─ Wired: 10.3.20.0/24

CONNECTIVITY:
• Sites connected via VPN (IPsec)
• Each site has router running OSPF
• All sites can communicate

ROUTING TABLE (NY Router):
Network       Next Hop     Interface  Protocol
10.1.0.0/16   Direct       eth0       Connected
10.2.0.0/16   VPN-London   tun0       OSPF
10.3.0.0/16   VPN-Tokyo    tun1       OSPF
0.0.0.0/0     ISP-Gateway  eth1       Static

Employee in NY (10.1.20.50) accesses server in Tokyo (10.3.1.10):
1. NY router: Lookup 10.3.1.10 → Route via VPN-Tokyo
2. Encrypt packet (IPsec)
3. Send to Tokyo router over internet
4. Tokyo router: Decrypt, forward to 10.3.1.10
5. Server responds, reverse path

Transparent to users!
```

---

### **3. ISP Core Network:**

```
INTERNET SERVICE PROVIDER TOPOLOGY:

┌─────────────────────────────────────────┐
│         BGP ROUTERS (Border)            │
│  AS 65000 (Our ISP)                     │
│                                         │
│  Peer with:                             │
│  • AS 65001 (Tier 1 Provider)           │
│  • AS 65002 (Peer ISP)                  │
│  • AS 15169 (Google)                    │
│  • AS 32934 (Facebook)                  │
└─────────────┬───────────────────────────┘
              │
     ┌────────┴────────┐
     │                 │
┌────┴──────┐   ┌──────┴─────┐
│  OSPF     │   │   OSPF     │
│  Area 0   │   │   Area 1   │
│ (Core)    │   │ (Regional) │
└───────────┘   └──┬──────┬──┘
                   │      │
              ┌────┴──┐ ┌─┴────┐
              │ POP 1 │ │ POP 2│
              │       │ │      │
              └───┬───┘ └──┬───┘
                  │        │
             Customers  Customers

ROUTING:
• BGP between ISPs (interdomain)
• OSPF within ISP (intradomain)
• MPLS for traffic engineering

BGP POLICY EXAMPLE:
Prefer routes:
1. Customers (they pay us)
2. Peers (free exchange)
3. Upstream (we pay them)

Announce:
• Customer routes to everyone
• Our routes to everyone
• Don't announce peer routes to other peers (no transit)

TRAFFIC FLOW:
Customer wants to reach Google (8.8.8.8):

1. Customer router → ISP edge router
2. ISP OSPF routing within network
3. ISP BGP lookup: Best path to AS 15169
4. Forward to Google via peering point
5. Direct connection (no transit cost)

ISP maintains:
• Full internet routing table (~900K routes)
• OSPF for internal routing (~1000 routes)
• BGP policies for path selection
```

---

### **4. Content Delivery Network (CDN):**

```
CDN ARCHITECTURE (e.g., Cloudflare, Akamai):

Global presence:
• 200+ POPs worldwide
• Anycast IP addresses
• Closest server responds

ANYCAST ROUTING:

Traditional (Unicast):
Server A: 203.0.113.10 (New York)
Server B: 198.51.100.20 (London)
→ Different IPs, DNS selects

Anycast:
Server A: 203.0.113.10 (New York)
Server B: 203.0.113.10 (London)  ← SAME IP!
Server C: 203.0.113.10 (Tokyo)   ← SAME IP!

How it works:
1. All servers announce SAME IP via BGP
2. Routers see multiple paths to 203.0.113.10
3. Choose closest path (shortest AS-PATH)
4. User automatically routed to nearest server

User in USA:
→ Routes to New York server (2 hops)

User in Europe:
→ Routes to London server (3 hops)

User in Asia:
→ Routes to Tokyo server (4 hops)

SAME IP, different servers!

BGP ANNOUNCEMENT:
New York announces: "I can reach 203.0.113.10 via AS-PATH: 65000"
London announces: "I can reach 203.0.113.10 via AS-PATH: 65001"
Tokyo announces: "I can reach 203.0.113.10 via AS-PATH: 65002"

Router selects path based on AS-PATH + local policy
```

---

### **5. Data Center Networking:**

```
DATA CENTER TOPOLOGY (Leaf-Spine):

          ┌──────┐  ┌──────┐  ┌──────┐
  SPINE   │  S1  │  │  S2  │  │  S3  │
          └──┬───┘  └──┬───┘  └──┬───┘
             │  ╲   ╱  │  ╲   ╱  │
             │   ╲ ╱   │   ╲ ╱   │
             │   ╱ ╲   │   ╱ ╲   │
             │  ╱   ╲  │  ╱   ╲  │
          ┌──┴───┐  ┌─┴────┐  ┌─┴────┐
  LEAF    │  L1  │  │  L2  │  │  L3  │
          └──┬───┘  └──┬───┘  └──┬───┘
             │         │         │
         Servers   Servers   Servers

ADVANTAGES:
• Any leaf can reach any other leaf in 2 hops
• Equal-cost multipath (ECMP)
• Easy to scale (add more leafs)
• Predictable latency

ROUTING:
• BGP Unnumbered (no IPs on p2p links)
• ECMP for load balancing
• Fast failover

IP ADDRESSING:
• Servers: 10.0.0.0/8
• Loopbacks: 192.168.0.0/24
• Point-to-point links: /31 subnets

TRAFFIC FLOW (Server A to Server B):
Server A (10.1.1.10) → Leaf1
Leaf1 → Spine1, Spine2, or Spine3 (ECMP)
Spine → Leaf2
Leaf2 → Server B (10.2.2.20)

Consistent 2-hop latency, load balanced
```

---

## **9) Comparison Section**

### **Routing Protocol Detailed Comparison:**

| Characteristic | RIP | EIGRP | OSPF | IS-IS | BGP |
|----------------|-----|-------|------|-------|-----|
| **Algorithm** | Distance vector | Hybrid (advanced DV) | Link state | Link state | Path vector |
| **Metric** | Hop count | Bandwidth + delay | Cost | Cost | Path attributes |
| **Convergence** | Minutes | Seconds | Seconds | Seconds | Minutes |
| **Scalability** | Poor (15 hops) | Good | Excellent | Excellent | Internet-scale |
| **Standards** | RFC (open) | Cisco proprietary | RFC (open) | RFC (open) | RFC (open) |
| **Complexity** | Low | Medium | Medium | Medium | High |
| **Use case** | Small legacy | Cisco shops | Enterprise | ISP/carrier | Internet |
| **Loop prevention** | Count-to-infinity issue | DUAL algorithm | Topology database | Topology database | AS-PATH |
| **Support VLSM** | v2 yes | Yes | Yes | Yes | Yes |
| **Support IPv6** | RIPng | EIGRPv6 | OSPFv3 | IS-IS for IPv6 | MP-BGP |

---

### **IPv4 vs IPv6 Migration Strategies:**

| Strategy | Description | Pros | Cons | Use Case |
|----------|-------------|------|------|----------|
| **Dual Stack** | Run both IPv4 and IPv6 simultaneously | Simple, no translation | Double management | Most common |
| **Tunneling (6to4, 6in4)** | Encapsulate IPv6 in IPv4 | Works with IPv4 infrastructure | Overhead, complexity | Transition phase |
| **NAT64** | Translate IPv6 to IPv4 | IPv6-only clients can reach IPv4 | Stateful, breaks end-to-end | Mobile networks |
| **DNS64** | Return IPv6 address for IPv4-only sites | Transparent to clients | Requires NAT64 | Mobile networks |
| **464XLAT** | Combination NAT64 + CLAT | Works for all apps | Complex | Mobile carriers |

---

## **10) Packet Walkthrough**

**Scenario:** User in New York (192.168.1.100) accesses web server in Tokyo (203.0.113.50)

```
════════════════════════════════════════════════════════════
STEP 1: PACKET CREATION (Host A - New York)
════════════════════════════════════════════════════════════

Application: Browser requests http://tokyo-server.com
DNS lookup: tokyo-server.com → 203.0.113.50

Transport Layer (TCP):
SRC Port: 54321 (ephemeral)
DST Port: 80 (HTTP)

Network Layer (IPv4) creates packet:
┌─────────────────────────────────────┐
│ IP HEADER:                          │
│   Version: 4                        │
│   IHL: 5 (20 bytes)                 │
│   Total Length: 60 bytes            │
│   TTL: 64                           │
│   Protocol: 6 (TCP)                 │
│   SRC IP: 192.168.1.100             │
│   DST IP: 203.0.113.50              │
├─────────────────────────────────────┤
│ TCP SEGMENT (40 bytes)              │
└─────────────────────────────────────┘

════════════════════════════════════════════════════════════
STEP 2: ROUTING DECISION (Host A)
════════════════════════════════════════════════════════════

Host checks routing table:
Destination     Gateway        Interface
0.0.0.0/0       192.168.1.1    eth0  ← Default route

203.0.113.50 doesn't match any specific route
→ Use default route
→ Send to gateway 192.168.1.1

════════════════════════════════════════════════════════════
STEP 3: ARP RESOLUTION (Host A)
════════════════════════════════════════════════════════════

Need MAC address of gateway (192.168.1.1)

Check ARP cache:
$ arp -a
192.168.1.1 → AA:BB:CC:DD:EE:FF

If not cached:
1. Send ARP Request: "Who has 192.168.1.1?"
2. Gateway replies: "I'm AA:BB:CC:DD:EE:FF"
3. Cache entry created

════════════════════════════════════════════════════════════
STEP 4: FRAME CREATION (Host A - Data Link Layer)
════════════════════════════════════════════════════════════

Ethernet frame:
┌─────────────────────────────────────┐
│ ETHERNET HEADER:                    │
│   DST MAC: AA:BB:CC:DD:EE:FF        │ (gateway)
│   SRC MAC: 11:22:33:44:55:66        │ (host A)
│   Type: 0x0800 (IPv4)               │
├─────────────────────────────────────┤
│ IP PACKET (60 bytes)                │
│   SRC IP: 192.168.1.100             │
│   DST IP: 203.0.113.50              │
│   TTL: 64                           │
└─────────────────────────────────────┘

Send on wire

════════════════════════════════════════════════════════════
STEP 5: HOME ROUTER / NAT GATEWAY (Hop 1)
════════════════════════════════════════════════════════════

Router receives frame:
1. Remove Ethernet header
2. Extract IP packet

Process IP packet:
• Decrement TTL: 64 → 63
• Recalculate IP checksum
• Check destination: 203.0.113.50
• Lookup routing table:
  0.0.0.0/0 → ISP Gateway (10.0.0.1)

NAT Translation:
┌─────────────────────────────────────┐
│ BEFORE NAT:                         │
│   SRC: 192.168.1.100:54321          │
│   DST: 203.0.113.50:80              │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ AFTER NAT:                          │
│   SRC: 203.0.113.10:12345           │
│   DST: 203.0.113.50:80              │
└─────────────────────────────────────┘

NAT Table Entry:
Internal             External
192.168.1.100:54321 ↔ 203.0.113.10:12345

Modified packet:
┌─────────────────────────────────────┐
│ IP HEADER:                          │
│   SRC IP: 203.0.113.10 (CHANGED)    │
│   DST IP: 203.0.113.50              │
│   TTL: 63 (DECREMENTED)             │
│   Checksum: RECALCULATED            │
└─────────────────────────────────────┘

ARP for next hop (10.0.0.1):
New Ethernet frame:
DST MAC: ISP router MAC
SRC MAC: Our WAN interface MAC

Send to ISP

════════════════════════════════════════════════════════════
STEP 6: ISP ROUTER (Hop 2)
════════════════════════════════════════════════════════════

ISP Router receives:
• Decrement TTL: 63 → 62
• Destination: 203.0.113.50
• Lookup routing table:

BGP Table:
203.0.113.0/24 via AS-PATH: 3356 2914 (next hop: 10.1.1.1)

Forward to next hop

════════════════════════════════════════════════════════════
STEP 7: TRANSIT ROUTERS (Hops 3-15)
════════════════════════════════════════════════════════════

Each router:
1. Receive frame
2. Extract IP packet
3. Decrement TTL (62 → 61 → 60 → ... → 49)
4. Check destination
5. Lookup in routing/forwarding table
6. Forward to next hop
7. Create new Ethernet frame (different MACs each hop)

NOTE: IP header stays mostly the same!
• SRC IP: 203.0.113.10 (unchanged)
• DST IP: 203.0.113.50 (unchanged)
• Only TTL changes (decrements)
• Checksum recalculated each hop

But Ethernet frame changes completely each hop:
• Different source/dest MAC addresses
• Rebuilt at each router

════════════════════════════════════════════════════════════
STEP 8: DESTINATION NETWORK ROUTER (Hop 16 - Tokyo)
════════════════════════════════════════════════════════════

Border router in Tokyo:
• Decrement TTL: 49 → 48
• Destination: 203.0.113.50
• Routing table: 203.0.113.50/32 → Directly connected

This is our network!

ARP for 203.0.113.50:
"Who has 203.0.113.50?"
Web server replies: "I'm FF:FF:FF:FF:FF:FF"

Final Ethernet frame:
┌─────────────────────────────────────┐
│ DST MAC: FF:FF:FF:FF:FF:FF (server) │
│ SRC MAC: Router's LAN interface     │
├─────────────────────────────────────┤
│ IP PACKET:                          │
│   SRC: 203.0.113.10 (NAT IP)        │
│   DST: 203.0.113.50 (web server)    │
│   TTL: 48                           │
└─────────────────────────────────────┘

═══════════════════════════════════════════════════════════
STEP 9: WEB SERVER RECEIVES (Tokyo)
════════════════════════════════════════════════════════════

Server extracts packet:
• SRC: 203.0.113.10:12345
• DST: 203.0.113.50:80
• TTL: 48 (started at 64, crossed 16 routers)

Server processes HTTP request
Generates response

════════════════════════════════════════════════════════════
STEP 10: RETURN PATH (Tokyo → New York)
════════════════════════════════════════════════════════════

Response packet:
┌─────────────────────────────────────┐
│ SRC IP: 203.0.113.50 (server)       │
│ DST IP: 203.0.113.10 (NAT address)  │
│ TTL: 64                             │
└─────────────────────────────────────┘

Travels back through internet (possibly different path!)
Asymmetric routing is common

════════════════════════════════════════════════════════════
STEP 11: HOME ROUTER RECEIVES RESPONSE
════════════════════════════════════════════════════════════

Router checks NAT table:
External: 203.0.113.10:12345
→ Internal: 192.168.1.100:54321

Reverse NAT:
┌─────────────────────────────────────┐
│ BEFORE:                             │
│   SRC: 203.0.113.50:80              │
│   DST: 203.0.113.10:12345           │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ AFTER:                              │
│   SRC: 203.0.113.50:80              │
│   DST: 192.168.1.100:54321          │
└─────────────────────────────────────┘

Forward to 192.168.1.100 on LAN

════════════════════════════════════════════════════════════
STEP 12: HOST A RECEIVES RESPONSE
════════════════════════════════════════════════════════════

Host receives:
• IP layer: Extract TCP segment
• TCP layer: Reassemble data, ACK
• HTTP layer: Parse web page
• Browser: Render page

═══════════════════════════════════════════════════════════
SUMMARY
════════════════════════════════════════════════════════════

Journey:
• 16 hops from New York to Tokyo
• TTL: 64 → 48 (decremented 16 times)
• IP addresses: UNCHANGED end-to-end (except NAT)
• MAC addresses: CHANGED at every hop
• Routing: Each router independently decided next hop
• NAT: Translated private IP to public IP

Layer 3 (Network) provided:
✓ End-to-end addressing (IP addresses)
✓ Routing across multiple networks
✓ Hop-by-hop forwarding
✓ NAT for address conservation

Layer 2 (Data Link) provided:
✓ Hop-by-hop delivery (MAC addresses)
✓ Changed at every router

This is the essence of internetworking!
```

---

## **11) Common Interview / Exam Traps**

### **Misconception 1: "MAC address is Layer 3"**
**Wrong:** MAC addresses and IP addresses are both network addresses  
**Right:**
- **MAC address (Layer 2):** Physical address, local significance, changes at each hop
- **IP address (Layer 3):** Logical address, global significance, stays same end-to-end

### **Misconception 2: "Router forwards packets based on destination MAC"**
**Wrong:** Routers use MAC addresses to route  
**Right:** Routers use **IP addresses** to make routing decisions. MAC addresses only matter for the current hop (Layer 2 delivery to next router).

### **Misconception 3: "Subnet mask is part of the IP address"**
**Wrong:** An IP address includes its subnet mask  
**Right:** IP address and subnet mask are separate. The mask defines which portion is network vs host. Same IP can be in different subnets with different masks.

### **Misconception 4: "All packets take the same path"**
**Wrong:** Packets between two hosts always follow the same route  
**Right:** Routes can change due to:
- Load balancing (ECMP)
- Link failures
- Routing protocol updates
- Asymmetric routing (A→B path ≠ B→A path)

### **Misconception 5: "TTL measures time"**
**Wrong:** TTL (Time To Live) is a duration in seconds  
**Right:** TTL is a **hop count** (number of routers). Decremented by 1 at each router, not based on time. Name is historical misnomer.

### **Misconception 6: "Private IPs can't be routed"**
**Wrong:** Private IP addresses (10.x.x.x, 192.168.x.x) are completely unroutable  
**Right:** Private IPs:
- CAN be routed within your network
- CAN'T be routed on the public internet
- ISPs filter/drop private IPs at border routers
- NAT translates private → public for internet access

### **Misconception 7: "Routers regenerate the entire packet"**
**Wrong:** Routers create a completely new IP packet  
**Right:** Routers modify the IP packet slightly:
- Decrement TTL
- Recalculate IP checksum
- **SRC and DST IPs unchanged** (unless NAT)
- Payload completely untouched
- New Layer 2 frame created (different MACs)

### **Misconception 8: "Default gateway is always needed"**
**Wrong:** Every network device must have a default gateway configured  
**Right:** Default gateway only needed for **communication outside local network**. If all communication is local, no gateway needed. But in practice, almost always configured.

---

### **Frequently Asked:**

**Q: What's the difference between a router and a switch?**  
A:
- **Switch (Layer 2):** Forwards based on MAC addresses, within a single network/VLAN
- **Router (Layer 3):** Forwards based on IP addresses, between different networks
- **Layer 3 switch:** Combines both (can route between VLANs)

**Q: How does a router know which interface to send a packet out?**  
A: **Routing table lookup.** Router looks up destination IP, finds matching route (longest prefix match), uses the specified outgoing interface.

**Q: Why does IP checksum only cover the header, not data?**  
A: **Performance.** Upper layers (TCP/UDP) have their own checksums for data. IP checksum only protects critical routing info (addresses, TTL). Recalculating checksum on large data at every hop would be slow.

**Q: Can two networks have the same IP address space?**  
A: **Yes, if isolated** (e.g., two companies both using 192.168.1.0/24 internally). But they can't communicate without NAT or renumbering. On the internet, IP spaces must be unique (allocated by RIRs).

**Q: What happens when TTL reaches 0?**  
A:
1. Router discards packet (doesn't forward)
2. Sends ICMP Time Exceeded message back to source
3. Source can retry or report error
This prevents infinite routing loops

**Q: How do routers handle broadcasts?**  
A: **They don't!** Routers **do not forward broadcasts**. Broadcasts are Layer 2 (limited to local network segment). This is a key difference from switches (which do forward broadcasts).

**Q: What's the difference between routing and forwarding?**  
A:
- **Routing (control plane):** Building the routing table using protocols (OSPF, BGP)
- **Forwarding (data plane):** Using that table to move individual packets through the router

**Q: Why use multiple routing protocols in one network?**  
A: Different protocols for different purposes:
- IGP (OSPF) within organization
- BGP to connect to internet/other organizations
- Static routes for simple/specific cases
- Redistribute routes between protocols

---

## **12) Retrieval Prompts**

### **Addressing:**
1. Explain the difference between a network address, host address, and broadcast address
2. How do you calculate the number of hosts in a /26 subnet?
3. What's the difference between a subnet mask and a network mask?
4. Walk through subnetting 192.168.1.0/24 into 8 equal subnets
5. What are private IP ranges and why do we need them?
6. Explain CIDR notation and why it replaced classful addressing

### **Routing:**
7. What's the difference between a routing table and a forwarding table?
8. Explain longest prefix matching with an example
9. Walk through the routing decision process for a packet
10. What's a default route and when would you use it?
11. Compare distance vector vs link-state routing protocols
12. How does BGP prevent routing loops?

### **Protocols:**
13. Explain the IPv4 packet header fields and their purposes
14. How does TTL prevent infinite loops?
15. Walk through the ARP process step-by-step
16. What's the difference between ICMP types and codes?
17. How does Path MTU Discovery work?
18. Explain how fragmentation and reassembly work

### **NAT:**
19. How does NAT work? Walk through an outbound and inbound packet
20. What's the difference between static NAT, dynamic NAT, and PAT?
21. Why does NAT break some protocols? Give examples
22. How does port forwarding work?

### **Troubleshooting:**
23. User can't reach a website — what Network Layer issues would you check?
24. How do you diagnose a routing loop?
25. What's the difference between "no route to host" and "connection refused"?
26. How would you use ping and traceroute to diagnose network issues?
27. Explain MTU black holes and how to detect them

### **Performance:**
28. What causes asymmetric routing and why does it matter?
29. Compare OSPF vs BGP — when would you use each?
30. What's ECMP and how does it improve performance?
31. Why is IPv6 considered better than IPv4 beyond just address space?

### **Real-World:**
32. Trace a packet from your home computer to a website, listing all hops
33. Explain how a CDN uses anycast routing
34. How does NAT conservation work in a typical home network?
35. What routing protocols would a large enterprise use and why?

---

## **13) TL;DR Compression**

**5-bullet summary:**

1. **Network Layer = Host-to-host communication across networks** — Uses logical addressing (IP addresses) to identify devices globally; routers make path selection decisions based on destination IP; provides internetworking by connecting different Layer 2 networks; Layer 2 (MAC) changes at every hop, but Layer 3 (IP) stays constant end-to-end

2. **IPv4 addressing:**
   - 32-bit addresses (4.3 billion), written dotted decimal (192.168.1.1)
   - CIDR notation: 192.168.1.0/24 (24 network bits, 8 host bits = 254 usable IPs)
   - Private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 (not routable on internet)
   - NAT maps many private IPs to one/few public IPs (conserves address space)

3. **Routing fundamentals:**
   - **Routing table:** Contains all known routes, built by routing protocols
   - **Forwarding table:** Optimized subset used for packet forwarding decisions
   - **Lookup:** Longest prefix match (most specific route wins)
   - **Protocols:** RIP (simple, slow), OSPF (enterprise, link-state), BGP (internet, policy-based)

4. **IP packet structure:**
   - Header: 20-60 bytes with SRC IP, DST IP, TTL, Protocol, Checksum
   - **TTL (Time To Live):** Hop count decremented at each router; prevents loops; packet dies at TTL=0
   - **Fragmentation:** Breaks large packets to fit MTU; modern networks use Path MTU Discovery instead
   - **ICMP:** Error reporting (ping, traceroute, destination unreachable)

5. **Key mechanisms:**
   - **ARP:** Maps IP addresses to MAC addresses on local network (broadcast "who has X?" → unicast reply)
   - **NAT:** Translates private IPs to public IPs; enables address reuse but breaks end-to-end connectivity
   - **Routing:** Control plane builds tables (OSPF, BGP); data plane forwards packets (lookup, decrement TTL, forward)
   - **Administrative Distance:** When multiple protocols provide routes, lower AD wins (Connected=0, Static=1, OSPF=110, RIP=120)

**One-sentence essence:**
The Network Layer enables global communication by assigning logical IP addresses to hosts, using routers to make intelligent path selection decisions based on routing tables built by protocols like OSPF and BGP, forwarding packets hop-by-hop across diverse networks while decrementing TTL to prevent loops, with NAT translating between private and public address spaces to conserve the limited IPv4 address pool until IPv6 adoption completes.