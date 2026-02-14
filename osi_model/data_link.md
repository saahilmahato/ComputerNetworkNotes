# Data Link Layer (Layer 2)

---

## **1) Concept Snapshot**

**Definition:**
The Data Link Layer provides node-to-node (hop-to-hop) data transfer between directly connected devices on the same network segment, handling physical addressing (MAC), error detection, frame synchronization, and media access control to ensure reliable transmission over the physical medium.

**Purpose:**
- **Framing:** Package raw bits into structured frames with boundaries
- **Physical addressing:** Use MAC addresses to identify devices on local network
- **Error detection:** Detect (and sometimes correct) transmission errors
- **Media access control:** Manage access to shared physical medium
- **Flow control:** Prevent sender from overwhelming receiver (hop-by-hop)
- **Link management:** Establish, maintain, and terminate data link connections

**Key insight:** This is the layer that makes the **"unreliable wire" appear reliable** for the hop between two directly connected devices. While Layer 3 (Network) handles end-to-end delivery across multiple networks, Layer 2 handles the critical hop-by-hop delivery across a single physical link.

---

## **2) Mental Model**

**Real-world analogy:**
Think of a relay race with multiple runners:

- **Physical Layer (L1):** The track itself (the physical medium - copper, fiber, radio waves)
- **Data Link Layer (L2):** The handoff between runners (ensuring baton passes successfully from one runner to directly connected next runner)
- **Network Layer (L3):** The overall race strategy (getting from start to finish, may involve multiple handoffs)

**Each handoff (L2) must be perfect:**
- Identify which runner gets the baton (MAC address = runner's number)
- Ensure clean transfer (error detection)
- Synchronize the handoff (frame boundaries)
- Only one handoff at a time on the track (media access control)

**Visual intuition:**
```
END-TO-END DELIVERY (Layer 3 - IP):
Host A ═══════════════════════════════════════════> Host B
  |                                                    ^
  | Layer 3: One logical path, same IP addresses       |
  |                                                    |

HOP-BY-HOP DELIVERY (Layer 2 - MAC):
Host A ──> Switch ──> Router ──> Switch ──> Router ──> Host B
  └─Hop 1─┘  └─Hop 2─┘  └─Hop 3─┘  └─Hop 4─┘  └─Hop 5─┘

Each hop:
• Different MAC addresses (source and destination change)
• New frame created at each device
• Error detection per hop
• Independent from other hops
```

**Simplified story:**
When you send an email, Layer 3 (IP) knows the email must go from New York to Tokyo. But it gets there through many hops. Layer 2 handles each individual hop: your computer to your home router (hop 1), then router to router (hop 2), then router to router (hop 3), etc. Each hop uses MAC addresses to identify the sender and receiver for **that specific link**, creates a new frame, and ensures error-free delivery to the next device.

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
Network (L3)       ───→ Internet
Data Link (L2)     ┐   ← WE ARE HERE
Physical (L1)      ├──→ Network Access
                   ┘
```

**Who talks to it:**
- **Above:** Network Layer (IP packets)
- **Below:** Physical Layer (bits on wire/radio)
- **Peers:** Other devices on same network segment

**Two sublayers of Data Link:**

```
╔════════════════════════════════════════════════════════╗
║ DATA LINK LAYER SUBLAYERS (IEEE 802 model)            ║
╠════════════════════════════════════════════════════════╣
║ LLC (Logical Link Control) - IEEE 802.2               ║
║ • Interface to Network Layer                           ║
║ • Protocol multiplexing                                ║
║ • Flow control and error control (optional)            ║
║ • Less important in modern networks                    ║
╠════════════════════════════════════════════════════════╣
║ MAC (Media Access Control)                             ║
║ • Physical addressing (MAC addresses)                  ║
║ • Media access control (who can transmit when)         ║
║ • Frame formatting                                     ║
║ • Error detection (CRC)                                ║
║ • This is where most action happens                    ║
╚════════════════════════════════════════════════════════╝
```

**Common Data Link protocols:**

```
WIRED NETWORKS:
• Ethernet (IEEE 802.3) - Dominant LAN technology
• PPP (Point-to-Point Protocol) - Serial links, DSL
• HDLC (High-Level Data Link Control) - Cisco routers
• Frame Relay - Legacy WAN (mostly obsolete)
• ATM (Asynchronous Transfer Mode) - Legacy (obsolete)

WIRELESS NETWORKS:
• Wi-Fi (IEEE 802.11) - Wireless LAN
• Bluetooth (IEEE 802.15.1) - Personal area network
• LTE/5G - Cellular networks
• WiMAX (IEEE 802.16) - Wireless broadband

OTHERS:
• Token Ring (IEEE 802.5) - Legacy (obsolete)
• FDDI (Fiber Distributed Data Interface) - Legacy
```

**PDU (Protocol Data Unit):**
- Layer 2 PDU = **Frame**

---

## **4) Mechanics (How It Actually Works)**

### **MAC ADDRESSES (Physical Addressing)**

**MAC Address Structure:**

```
48-bit address (6 bytes), written in hexadecimal:

Format 1: AA:BB:CC:DD:EE:FF (colon-separated)
Format 2: AA-BB-CC-DD-EE-FF (hyphen-separated)
Format 3: AABBCC.DDEEFF (Cisco format)

Binary: 10101010.10111011.11001100.11011101.11101110.11111111

STRUCTURE:
┌─────────────────────┬─────────────────────┐
│  OUI (24 bits)      │  Device ID (24 bits)│
│  Organizationally   │  Manufacturer       │
│  Unique Identifier  │  assigned           │
└─────────────────────┴─────────────────────┘
    AA:BB:CC              DD:EE:FF

OUI EXAMPLES:
• 00:1A:A0:xx:xx:xx → Dell
• 00:50:56:xx:xx:xx → VMware
• 3C:D9:2B:xx:xx:xx → HP
• B8:27:EB:xx:xx:xx → Raspberry Pi Foundation
• FC:FB:FB:xx:xx:xx → Apple (some devices)

FIRST BYTE SPECIAL BITS:
┌──────────────────────────────────┐
│ Bit 0 (LSB): I/G (Individual/Group)│
│   0 = Unicast                    │
│   1 = Multicast/Broadcast        │
├──────────────────────────────────┤
│ Bit 1: U/L (Universal/Local)     │
│   0 = Globally unique (OUI)      │
│   1 = Locally administered       │
└──────────────────────────────────┘

SPECIAL MAC ADDRESSES:

Broadcast:
FF:FF:FF:FF:FF:FF
• All bits set to 1
• Sent to all devices on network
• Used by ARP, DHCP discover

Multicast:
01:00:5E:xx:xx:xx (IPv4 multicast)
33:33:xx:xx:xx:xx (IPv6 multicast)
• First bit = 1 (multicast)
• Sent to subset of devices

Zero Address:
00:00:00:00:00:00
• Not used for actual communication
• Sometimes used as placeholder
```

---

**MAC Address Scope:**

```
CRITICAL DIFFERENCE FROM IP:

IP ADDRESS (Layer 3):
• Global significance
• Stays same end-to-end
• Identifies device anywhere on internet

MAC ADDRESS (Layer 2):
• Local significance only
• Changes at each hop (each router)
• Identifies device on current network segment

EXAMPLE:

Packet from Host A to Host B through 2 routers:

┌────────────────────────────────────────────────┐
│ HOP 1: Host A → Router 1                       │
│ SRC MAC: Host A (AA:AA:AA:AA:AA:AA)            │
│ DST MAC: Router 1 (BB:BB:BB:BB:BB:BB)          │
│ SRC IP: 192.168.1.100 (unchanged)              │
│ DST IP: 203.0.113.50 (unchanged)               │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│ HOP 2: Router 1 → Router 2                     │
│ SRC MAC: Router 1 (CC:CC:CC:CC:CC:CC)          │
│ DST MAC: Router 2 (DD:DD:DD:DD:DD:DD)          │
│ SRC IP: 192.168.1.100 (unchanged)              │
│ DST IP: 203.0.113.50 (unchanged)               │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│ HOP 3: Router 2 → Host B                       │
│ SRC MAC: Router 2 (EE:EE:EE:EE:EE:EE)          │
│ DST MAC: Host B (FF:FF:FF:FF:FF:FF)            │
│ SRC IP: 192.168.1.100 (unchanged)              │
│ DST IP: 203.0.113.50 (unchanged)               │
└────────────────────────────────────────────────┘

MAC addresses: Changed at every hop
IP addresses: Unchanged end-to-end (unless NAT)
```

---

### **ETHERNET (IEEE 802.3)**

**Ethernet Frame Structure (Ethernet II):**

```
ETHERNET II FRAME (Most common):

┌──────────────────────────────────────────────────────────┐
│ PREAMBLE (7 bytes): 10101010 repeated                    │
│ • Clock synchronization                                  │
│ • Not considered part of frame                           │
├──────────────────────────────────────────────────────────┤
│ SFD - Start Frame Delimiter (1 byte): 10101011           │
│ • Marks beginning of frame                               │
│ • Last two bits = 11 signal "frame starts now"           │
├──────────────────────────────────────────────────────────┤
│ DESTINATION MAC (6 bytes)                                │
│ • Who should receive this frame                          │
│ • Can be unicast, multicast, or broadcast                │
├──────────────────────────────────────────────────────────┤
│ SOURCE MAC (6 bytes)                                     │
│ • Who sent this frame                                    │
│ • Always unicast (never broadcast/multicast)             │
├──────────────────────────────────────────────────────────┤
│ TYPE/LENGTH (2 bytes)                                    │
│ • If ≥ 1536 (0x0600): EtherType                         │
│   - 0x0800 = IPv4                                        │
│   - 0x0806 = ARP                                         │
│   - 0x86DD = IPv6                                        │
│ • If ≤ 1500: Length of payload                          │
├──────────────────────────────────────────────────────────┤
│ PAYLOAD (46-1500 bytes)                                  │
│ • Data from upper layer (IP packet, ARP, etc.)           │
│ • Minimum: 46 bytes (padded if necessary)                │
│ • Maximum: 1500 bytes (MTU)                              │
├──────────────────────────────────────────────────────────┤
│ FCS - Frame Check Sequence (4 bytes)                     │
│ • CRC-32 checksum                                        │
│ • Detects transmission errors                            │
│ • Calculated over: Dest MAC + Src MAC + Type + Payload   │
└──────────────────────────────────────────────────────────┘

TOTAL FRAME SIZE:
• Minimum: 64 bytes (header + 46 byte payload + FCS)
• Maximum: 1518 bytes (header + 1500 byte payload + FCS)
• With 802.1Q VLAN tag: 1522 bytes
• Jumbo frames: Up to 9000 bytes (non-standard)

EXAMPLE FRAME (ARP request):

Preamble:     AA AA AA AA AA AA AA
SFD:          AB
Dest MAC:     FF FF FF FF FF FF (broadcast)
Src MAC:      00 1A A0 12 34 56 (Dell device)
Type:         08 06 (ARP)
Payload:      [28 bytes of ARP data + 18 bytes padding = 46]
FCS:          A3 2F 1C 9D (CRC-32)
```

---

**802.1Q VLAN Tagging:**

```
VLAN TAG (4 bytes inserted after Source MAC):

┌──────────────────────────────────────────┐
│ TPID (Tag Protocol ID): 0x8100 (2 bytes) │
│ • Indicates VLAN-tagged frame            │
├──────────────────────────────────────────┤
│ TCI (Tag Control Information): 2 bytes   │
│ ┌─────────────────────────────────────┐  │
│ │ PCP (Priority): 3 bits              │  │
│ │ • QoS priority (0-7)                │  │
│ ├─────────────────────────────────────┤  │
│ │ DEI (Drop Eligible): 1 bit          │  │
│ │ • Can frame be dropped under load?  │  │
│ ├─────────────────────────────────────┤  │
│ │ VID (VLAN ID): 12 bits              │  │
│ │ • VLAN number (1-4094)              │  │
│ │ • 0 = priority tagged only          │  │
│ │ • 4095 = reserved                   │  │
│ └─────────────────────────────────────┘  │
└──────────────────────────────────────────┘

FRAME WITH VLAN TAG:

Dest MAC (6) | Src MAC (6) | 802.1Q Tag (4) | Type (2) | Payload | FCS (4)

VLAN TYPES:

Access Port:
• Untagged frames
• Belongs to one VLAN
• End devices connected here
• Switch adds/removes VLAN tag

Trunk Port:
• Tagged frames (802.1Q)
• Carries multiple VLANs
• Switch-to-switch or switch-to-router
• Tags preserved across link

EXAMPLE:
PC in VLAN 10 sends frame:

1. PC → Switch (Access Port):
   [Dest][Src][Type][Data][FCS]  ← Untagged

2. Switch → Switch (Trunk Port):
   [Dest][Src][0x8100][VLAN=10][Type][Data][FCS]  ← Tagged

3. Switch → PC (Access Port):
   [Dest][Src][Type][Data][FCS]  ← Untagged

PC never sees VLAN tag; switch handles it
```

---

### **ERROR DETECTION**

**CRC (Cyclic Redundancy Check):**

```
PURPOSE:
Detect errors in transmitted frames
• Bit flips (0→1 or 1→0)
• Burst errors (multiple consecutive bits)
• Most transmission errors

ALGORITHM (Simplified):

1. SENDER:
   Data: 11010011101100
   Generator polynomial: 1101 (CRC-4 example)
   
   Append zeros: 11010011101100 000
   Divide by generator: 11010011101100 000 ÷ 1101
   Remainder: 001 (this is the CRC)
   
   Send: 11010011101100 001
         └─── Data ────┘ └CRC┘

2. RECEIVER:
   Receive: 11010011101100 001
   Divide by same generator: ÷ 1101
   
   If remainder = 0: ✓ No errors detected
   If remainder ≠ 0: ✗ Errors detected, discard frame

ETHERNET USES CRC-32:
• 32-bit checksum
• Generator polynomial: x^32 + ... (complex)
• Detects:
  - All single-bit errors
  - All double-bit errors
  - All burst errors ≤ 32 bits
  - 99.99% of longer burst errors

LIMITATIONS:
✓ Very good at detecting errors
✗ Cannot correct errors (only detect)
✗ Can miss errors if multiple bits flip in specific patterns
✗ Not cryptographically secure (not for tamper detection)

WHEN ERROR DETECTED:
Ethernet: Silently discard frame
• No retransmission at Layer 2
• Upper layers (TCP) handle retransmission
• Assumes errors are rare on modern networks
```

---

**Error Detection vs Correction:**

```
ERROR DETECTION (Layer 2):
• CRC detects errors
• Frame discarded if error found
• No attempt to fix
• Upper layers must retransmit
• Used: Ethernet, Wi-Fi, most Layer 2 protocols

ERROR CORRECTION (Some specialized links):
• FEC (Forward Error Correction)
• Can fix limited errors without retransmission
• Higher overhead (extra bits)
• Used: Satellite links, deep space, some wireless
• Example: Reed-Solomon codes

TRADE-OFF:
Detection: Low overhead, simple, works for most cases
Correction: High overhead, complex, needed for high-error environments
```

---

### **MEDIA ACCESS CONTROL (MAC)**

**The Problem:**

```
SHARED MEDIUM:
Multiple devices connected to same physical medium (wire, radio)

┌────────┐
│ Device │
│   A    │──┐
└────────┘  │
            │
┌────────┐  │  ┌──────────────┐
│ Device ├──┼──┤ Shared Medium│ (e.g., Ethernet hub, Wi-Fi)
│   B    │  │  └──────────────┘
└────────┘  │
            │
┌────────┐  │
│ Device ├──┘
│   C    │
└────────┘

QUESTION: How do we prevent collisions?
• If A and B transmit simultaneously → signals overlap → garbage

SOLUTION: Media Access Control protocols
```

---

**CSMA/CD (Carrier Sense Multiple Access with Collision Detection):**
*Used in: Ethernet (legacy, half-duplex)*

```
ALGORITHM:

1. CARRIER SENSE:
   Before transmitting, listen to medium
   If busy (someone else transmitting):
     → Wait until idle
   If idle:
     → Proceed to step 2

2. MULTIPLE ACCESS:
   Start transmitting frame
   All devices can potentially transmit

3. COLLISION DETECTION:
   While transmitting:
     Monitor medium for collision
     
   If collision detected:
     → Both senders detect voltage spike/signal distortion
     → Stop transmitting immediately
     → Send JAM signal (48 bits) to alert all devices
     → Wait random backoff time
     → Try again
     
   If no collision:
     → Frame transmitted successfully

BINARY EXPONENTIAL BACKOFF:
After collision:
• Attempt 1: Wait 0 or 1 time slots (random)
• Attempt 2: Wait 0, 1, 2, or 3 time slots
• Attempt 3: Wait 0-7 time slots
• Attempt n: Wait 0 to (2^n - 1) time slots
• Max attempts: 16, then give up

Time slot = 51.2 microseconds (for 10 Mbps Ethernet)

EXAMPLE:
Device A and B both start transmitting at same time

Time 0:     Both transmit
Time 5μs:   Collision occurs
Time 5.1μs: Both detect collision
Time 5.2μs: Both send JAM signal
Time 5.3μs: A picks 1 slot wait, B picks 3 slots
Time 56μs:  A transmits (1 slot elapsed)
Time 157μs: A finishes successfully
Time 158μs: B transmits (3 slots elapsed)

COLLISION DOMAIN:
• All devices that can collide with each other
• Hub: All ports = 1 collision domain (collisions possible)
• Switch: Each port = separate collision domain (no collisions)

MODERN ETHERNET:
• Full-duplex (switches)
• No collisions possible (separate TX/RX)
• CSMA/CD disabled
• Much simpler and faster!
```

---

**CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance):**
*Used in: Wi-Fi (802.11)*

```
PROBLEM WITH WIRELESS:
• Can't detect collisions while transmitting (radio limitations)
• Hidden node problem (A and C can't hear each other, both transmit to B)

SOLUTION: Avoid collisions instead of detecting them

ALGORITHM:

1. CARRIER SENSE:
   Listen before transmitting
   If busy:
     → Wait
   If idle:
     → Proceed

2. RANDOM BACKOFF (Even if idle):
   Wait random time before transmitting
   Reduces probability of collision

3. RTS/CTS (Optional, for large frames):
   
   Sender → RTS (Request to Send) → Receiver
            ← CTS (Clear to Send) ←
   
   All other devices hear RTS or CTS:
     → Set NAV (Network Allocation Vector) timer
     → Don't transmit until NAV expires
   
   Sender → DATA →
            ← ACK ←

4. ACKNOWLEDGMENT:
   Receiver sends ACK for every frame
   If no ACK received:
     → Assume collision/loss
     → Retransmit after backoff

DISTRIBUTED COORDINATION FUNCTION (DCF):

Interframe spaces (wait times):
• SIFS (Short IFS): 10μs - Highest priority (ACKs, CTS)
• PIFS (PCF IFS): 30μs - Medium priority
• DIFS (DCF IFS): 50μs - Lowest priority (data frames)

Timeline:
Medium idle for DIFS
  ↓
Random backoff (contention window)
  ↓
Transmit frame
  ↓
Wait SIFS
  ↓
Receive ACK

HIDDEN NODE PROBLEM:

    A ←——— Cannot hear ———→ C
     \                     /
      \                   /
       \                 /
        ↘               ↙
           B (in range of both)

A transmits to B
C transmits to B (can't hear A is transmitting)
→ Collision at B

RTS/CTS solves this:
A → RTS → B
B → CTS → C (C hears CTS, sets NAV)
A → DATA → B (C stays quiet)
```

---

**Token Passing (Legacy):**
*Used in: Token Ring, FDDI (mostly obsolete)*

```
CONCEPT:
Special "token" frame circulates the ring
Only device holding token can transmit

PROCESS:
1. Token circulates: A → B → C → D → A → ...
2. Device B wants to transmit:
   • Captures token
   • Converts to data frame
   • Transmits data
   • Removes data from ring when it comes back around
   • Releases new token

3. Next device can now capture token

ADVANTAGES:
✓ No collisions
✓ Predictable access time (bounded latency)
✓ Fair access (everyone gets turn)

DISADVANTAGES:
✗ Overhead (token circulation)
✗ Single point of failure (lost token)
✗ Complex recovery procedures
✗ Doesn't scale well

OBSOLETE:
• Replaced by switched Ethernet
• Simpler, faster, cheaper
```

---

### **SWITCHING**

**Switch Operation:**

```
SWITCH vs HUB:

HUB (Layer 1 device):
┌────────────────────────────────┐
│  Physical Layer Repeater       │
│                                │
│  Receives bits on one port     │
│  Repeats on ALL other ports    │
│                                │
│  • One collision domain        │
│  • Half-duplex                 │
│  • Shared bandwidth            │
│  • Obsolete                    │
└────────────────────────────────┘

SWITCH (Layer 2 device):
┌────────────────────────────────┐
│  Data Link Layer Device        │
│                                │
│  • Reads MAC addresses         │
│  • Forwards to specific port   │
│  • Each port = collision domain│
│  • Full-duplex                 │
│  • Dedicated bandwidth per port│
│  • Dominant technology         │
└────────────────────────────────┘

SWITCH FUNCTIONS:

1. LEARNING:
   • Reads source MAC of incoming frames
   • Builds MAC address table
   • Maps MAC → Port

2. FORWARDING:
   • Reads destination MAC
   • Looks up in MAC table
   • Forwards to correct port

3. FLOODING:
   • If destination MAC unknown
   • Sends to all ports except source

4. FILTERING:
   • If destination on same port as source
   • Don't forward (already there)

5. AGING:
   • Remove stale entries (default: 5 minutes)
   • Keeps table current
```

---

**MAC Address Table:**

```
INITIAL STATE (Empty table):
Port | MAC Address | Age
──────────────────────────
(empty)

FRAME ARRIVES:
Port 1: Frame from AA:AA:AA:AA:AA:AA to BB:BB:BB:BB:BB:BB

LEARNING:
Port | MAC Address        | Age
──────────────────────────────────
1    | AA:AA:AA:AA:AA:AA | 0

FORWARDING DECISION:
Destination BB:BB:BB:BB:BB:BB not in table
→ FLOOD: Send to all ports except Port 1

PORT 3 RESPONDS:
Port 3: Frame from BB:BB:BB:BB:BB:BB to AA:AA:AA:AA:AA:AA

LEARNING:
Port | MAC Address        | Age
──────────────────────────────────
1    | AA:AA:AA:AA:AA:AA | 0
3    | BB:BB:BB:BB:BB:BB | 0

FORWARDING DECISION:
Destination AA:AA:AA:AA:AA:AA in table (Port 1)
→ FORWARD: Send to Port 1 only

ONGOING:
Future frames between AA and BB:
• Directly forwarded to correct ports
• No flooding needed
• Efficient!

VIEW MAC TABLE (Cisco):
Switch# show mac address-table

Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  1     aaaa.aaaa.aaaa    DYNAMIC     Gi0/1
  1     bbbb.bbbb.bbbb    DYNAMIC     Gi0/3
  1     cccc.cccc.cccc    DYNAMIC     Gi0/5

AGING:
After 300 seconds (5 minutes) of no traffic:
• Entry removed from table
• Next frame from that MAC will re-learn
```

---

**Switching Methods:**

```
STORE-AND-FORWARD:
┌─────────────────────────────────┐
│ 1. Receive entire frame         │
│ 2. Check FCS (error detection)  │
│ 3. If valid, forward            │
│ 4. If error, discard            │
└─────────────────────────────────┘

✓ Error checking (reliable)
✓ Can buffer frames (handle speed mismatch)
✗ Higher latency (must receive full frame)

Latency: Frame transmission time + processing

CUT-THROUGH:
┌─────────────────────────────────┐
│ 1. Read destination MAC (6 bytes)│
│ 2. Start forwarding immediately │
│ 3. Don't wait for rest of frame │
└─────────────────────────────────┘

✓ Lower latency (fixed ~10μs delay)
✗ No error checking (forwards bad frames)
✗ Can't handle speed mismatches

Latency: Time to read dest MAC + processing (~10μs)

FRAGMENT-FREE:
┌─────────────────────────────────┐
│ 1. Read first 64 bytes          │
│ 2. If collision, would occur here│
│ 3. Assume rest is valid         │
│ 4. Forward                      │
└─────────────────────────────────┘

✓ Better than cut-through (catches collision fragments)
✓ Faster than store-and-forward
✗ Still no full error checking

ADAPTIVE:
Switch automatically changes method based on error rate
High errors → Store-and-forward
Low errors → Cut-through
```

---

**Spanning Tree Protocol (STP):**

```
PROBLEM: LOOPS IN SWITCHED NETWORKS

Topology:
     ┌────────┐
  ┌──┤Switch A├──┐
  │  └────────┘  │
  │              │
┌─┴──┐        ┌──┴─┐
│Sw B│────────│Sw C│
└────┘        └────┘

Without loop prevention:
1. Broadcast from PC
2. Switch A forwards to B and C
3. B forwards to C
4. C forwards to B
5. B forwards back to A
6. A forwards to C
→ Infinite loop! (Broadcast storm)

SOLUTION: SPANNING TREE PROTOCOL (IEEE 802.1D)

GOAL:
• Prevent loops by blocking redundant paths
• Maintain redundancy (activate backup if primary fails)

PROCESS:

1. ELECT ROOT BRIDGE:
   • Bridge with lowest Bridge ID becomes root
   • Bridge ID = Priority (2 bytes) + MAC (6 bytes)
   • Default priority: 32768
   • Lowest MAC wins if priorities equal

2. CALCULATE PATH COST:
   • Cost to reach root bridge
   • Based on link speed:
     - 10 Gbps: Cost 2
     - 1 Gbps: Cost 4
     - 100 Mbps: Cost 19
     - 10 Mbps: Cost 100

3. SELECT PORT ROLES:
   
   ROOT PORT (RP):
   • Port with lowest cost to root
   • One per non-root switch
   
   DESIGNATED PORT (DP):
   • Port that forwards on this segment
   • Sends BPDUs
   
   BLOCKED PORT:
   • Prevents loops
   • Doesn't forward data
   • Listens to BPDUs
   • Can activate if topology changes

EXAMPLE:
     ┌────────┐
  ┌──┤Switch A├──┐  ROOT BRIDGE
  │  │(Root)  │  │
  │  └────────┘  │
  │ DP        DP │
  │              │
┌─┴──┐        ┌──┴─┐
│Sw B│────────│Sw C│
│ RP │   BP   │ RP │
└────┘  ××××  └────┘
         ↑
    Blocked port
    (prevents loop)

PORT STATES:
┌──────────┬──────────┬──────────────┐
│ State    │ Duration │ Forwards?    │
├──────────┼──────────┼──────────────┤
│ Disabled │ -        │ No (admin)   │
│ Blocking │ 20s      │ No (STP)     │
│ Listening│ 15s      │ No (learning)│
│ Learning │ 15s      │ No (building)│
│ Forwarding│ -       │ Yes          │
└──────────┴──────────┴──────────────┘

Total convergence: ~50 seconds (too slow!)

RAPID STP (RSTP - 802.1w):
• Faster convergence (~1 second)
• New port states: Discarding, Learning, Forwarding
• Proposal/Agreement mechanism
• Modern default

MULTIPLE SPANNING TREE (MST - 802.1s):
• Multiple spanning trees per VLAN groups
• Better load balancing
• Reduced overhead
```

---

### **Wi-Fi (IEEE 802.11)**

**Wi-Fi Frame Structure:**

```
802.11 MAC FRAME:

┌────────────────────────────────────────────────┐
│ FRAME CONTROL (2 bytes)                        │
│ • Protocol version, Type, Subtype              │
│ • To DS, From DS (distribution system)         │
│ • Retry, Power management, More data flags     │
├────────────────────────────────────────────────┤
│ DURATION/ID (2 bytes)                          │
│ • NAV (Network Allocation Vector) timer        │
│ • How long channel will be busy                │
├────────────────────────────────────────────────┤
│ ADDRESS 1 (6 bytes) - Receiver MAC             │
├────────────────────────────────────────────────┤
│ ADDRESS 2 (6 bytes) - Transmitter MAC          │
├────────────────────────────────────────────────┤
│ ADDRESS 3 (6 bytes) - Filtering/Routing        │
├────────────────────────────────────────────────┤
│ SEQUENCE CONTROL (2 bytes)                     │
│ • Fragment number, Sequence number             │
├────────────────────────────────────────────────┤
│ ADDRESS 4 (6 bytes) - Optional (WDS mode)      │
├────────────────────────────────────────────────┤
│ PAYLOAD (0-2304 bytes)                         │
│ • Data from upper layers                       │
├────────────────────────────────────────────────┤
│ FCS (4 bytes) - CRC-32 checksum                │
└────────────────────────────────────────────────┘

WHY 4 MAC ADDRESSES?

Different modes use addresses differently:

MODE 1 - CLIENT TO AP:
Address 1: AP MAC (destination)
Address 2: Client MAC (source)
Address 3: Final destination MAC (e.g., internet gateway)

MODE 2 - AP TO CLIENT:
Address 1: Client MAC (destination)
Address 2: AP MAC (source)
Address 3: Original source MAC

MODE 3 - AP TO AP (WDS):
All 4 addresses used for wireless bridging

FRAME TYPES:

MANAGEMENT (Type 0):
• Beacon (subtype 8) - AP announces presence
• Probe request/response - Client discovers APs
• Authentication
• Association request/response
• Deauthentication

CONTROL (Type 1):
• RTS (Request to Send)
• CTS (Clear to Send)
• ACK (Acknowledgment)

DATA (Type 2):
• Standard data frames
• QoS data frames (with priority)
```

---

**Wi-Fi Association Process:**

```
CONNECTING TO Wi-Fi NETWORK:

1. SCANNING (Passive or Active):
   
   PASSIVE:
   • Client listens for Beacon frames
   • APs broadcast beacons every 100ms
   • Contains: SSID, supported rates, encryption
   
   ACTIVE:
   • Client sends Probe Request
   • APs respond with Probe Response
   • Faster than passive

2. AUTHENTICATION:
   
   OPEN AUTHENTICATION (No WPA):
   Client → Authentication Request → AP
          ← Authentication Response ←
   (Trivial, always succeeds)
   
   WPA/WPA2:
   4-Way Handshake (happens after association)

3. ASSOCIATION:
   Client → Association Request → AP
          ← Association Response ←
   
   Request contains:
   • Supported rates
   • Capabilities
   • SSID
   
   Response contains:
   • Association ID (AID)
   • Status code (success/failure)

4. FOUR-WAY HANDSHAKE (WPA2):
   
   Message 1: AP → Client (ANonce)
   Message 2: Client → AP (SNonce, MIC)
   Message 3: AP → Client (GTK, MIC)
   Message 4: Client → AP (ACK)
   
   Result:
   • PTK (Pairwise Transient Key) derived
   • GTK (Group Temporal Key) installed
   • Encryption enabled

5. CONNECTED:
   Client can now send/receive data
   Frames encrypted with derived keys

STATES:
┌──────────────────────────────────┐
│ State 1: Unauthenticated         │
│          Unassociated            │
├──────────────────────────────────┤
│ State 2: Authenticated           │
│          Unassociated            │
├──────────────────────────────────┤
│ State 3: Authenticated           │
│          Associated              │
└──────────────────────────────────┘
```

---

**Wi-Fi Channels and Frequency:**

```
2.4 GHz BAND (802.11b/g/n):

Channels 1-14 (US uses 1-11):
• Each channel: 20 MHz wide
• Center frequencies: 2.412 GHz - 2.484 GHz
• Spacing: 5 MHz between centers
• Overlap: Channels overlap!

Non-overlapping channels (US):
• Channel 1 (2.412 GHz)
• Channel 6 (2.437 GHz)
• Channel 11 (2.462 GHz)

     1    2    3    4    5    6    7    8    9   10   11
  |────|────|────|────|────|────|────|────|────|────|────|
     └──── Channel 1 (20 MHz) ────┘
                    └──── Channel 6 (20 MHz) ────┘
                                   └──── Channel 11 (20 MHz) ────┘

BEST PRACTICE: Use channels 1, 6, 11 only (no overlap)

5 GHz BAND (802.11a/n/ac/ax):

Many more channels:
• 36, 40, 44, 48 (lower)
• 52, 56, 60, 64 (middle)
• 100-144 (upper, DFS)
• 149, 153, 157, 161, 165 (UNII-3)

Channel width:
• 20 MHz (basic)
• 40 MHz (bonding 2 channels)
• 80 MHz (802.11ac)
• 160 MHz (802.11ac Wave 2)

Advantages:
✓ Less congested than 2.4 GHz
✓ More channels available
✓ Higher speeds (wider channels)
✗ Shorter range
✗ Worse penetration through walls

6 GHz BAND (802.11ax/Wi-Fi 6E):
• Even more spectrum
• Very clean (new allocation)
• Lower latency
• Wi-Fi 6E devices only
```

---

## **5) Key Structures & Components**

### **Network Topologies:**

```
PHYSICAL TOPOLOGIES (How devices are connected):

BUS:
───┬───┬───┬───┬───
   │   │   │   │
  PC  PC  PC  PC

✓ Simple, cheap
✗ Single point of failure
✗ Collision domain shared
✗ Obsolete (10Base2, 10Base5)

STAR:
      ┌────────┐
   ┌──┤ Switch ├──┐
   │  └────────┘  │
  PC      PC      PC

✓ Centralized management
✓ Easy to add/remove devices
✓ Failure isolated to one cable
✗ Central device failure = network down
✓ Modern standard

RING:
   PC ── PC
   │      │
   PC ── PC

✓ Predictable latency (token)
✗ Failure breaks ring (needs dual ring)
✗ Obsolete (Token Ring, FDDI)

MESH:
   PC ──── PC
   │╲    ╱│
   │ ╲  ╱ │
   │  ╲╱  │
   PC ──── PC

✓ Redundant paths
✓ High availability
✗ Expensive (many cables/links)
✗ Complex
• Used: Enterprise core, wireless mesh

HIERARCHICAL (Tree/Hybrid):
        ┌────────┐
        │  Core  │
        └───┬────┘
      ┌─────┴─────┐
   ┌──┴──┐     ┌──┴──┐
   │Dist │     │Dist │
   └──┬──┘     └──┬──┘
   ┌──┴──┬──┐  ┌──┴──┬──┐
   │Acc  │Acc│ │Acc  │Acc│
   └──┬──┴─┬─┘ └─┬──┴─┬─┘
     PCs  PCs   PCs  PCs

• Three-tier: Core, Distribution, Access
• Modern enterprise standard
• Scalable, organized
```

---

### **Cable Types:**

```
TWISTED PAIR (Ethernet):

CATEGORIES:
┌────────┬──────────┬──────────┬────────────┐
│ Cat    │ Speed    │ Distance │ Use        │
├────────┼──────────┼──────────┼────────────┤
│ Cat 5  │ 100 Mbps │ 100m     │ Obsolete   │
│ Cat 5e │ 1 Gbps   │ 100m     │ Common     │
│ Cat 6  │ 1 Gbps   │ 100m     │ Common     │
│        │ 10 Gbps  │ 55m      │            │
│ Cat 6a │ 10 Gbps  │ 100m     │ Modern     │
│ Cat 7  │ 10 Gbps  │ 100m     │ Shielded   │
│ Cat 8  │ 25/40 Gbps│ 30m     │ Data center│
└────────┴──────────┴──────────┴────────────┘

TYPES:
• UTP (Unshielded Twisted Pair) - Most common
• STP (Shielded Twisted Pair) - EMI protection
• FTP (Foiled Twisted Pair) - Foil shield

WIRING STANDARDS:
T568A and T568B (pin configurations)
• Straight-through: Both ends same (A-A or B-B)
• Crossover: Different ends (A-B)
• Modern devices: Auto-MDIX (auto-detect)

FIBER OPTIC:

SINGLE-MODE (SMF):
• Small core (9 microns)
• Laser light source
• Long distance (up to 100 km+)
• Higher cost
• Use: Long-haul, data center interconnect

MULTI-MODE (MMF):
• Larger core (50 or 62.5 microns)
• LED light source
• Shorter distance (up to 550m typically)
• Lower cost
• Use: Campus networks, buildings

ADVANTAGES:
✓ Immune to EMI
✓ Very high bandwidth
✓ Long distances
✓ Secure (hard to tap)
✗ More expensive
✗ Fragile
✗ Requires specialized equipment

CONNECTORS:
• LC (Lucent Connector) - Common, small
• SC (Subscriber Connector) - Common
• ST (Straight Tip) - Legacy
• MTP/MPO - High-density (data centers)
```

---

### **Ethernet Standards:**

```
IEEE 802.3 VARIANTS:

┌────────────┬──────────┬──────────┬────────────┬──────────┐
│ Standard   │ Speed    │ Medium   │ Distance   │ Notes    │
├────────────┼──────────┼──────────┼────────────┼──────────┤
│ 10Base-T   │ 10 Mbps  │ Cat 3 UTP│ 100m       │ Legacy   │
│ 100Base-TX │ 100 Mbps │ Cat 5 UTP│ 100m       │ Common   │
│ 1000Base-T │ 1 Gbps   │ Cat 5e   │ 100m       │ Dominant │
│ 10GBase-T  │ 10 Gbps  │ Cat 6a   │ 100m       │ Growing  │
│ 25GBase-T  │ 25 Gbps  │ Cat 8    │ 30m        │ DC only  │
│ 40GBase-T  │ 40 Gbps  │ Cat 8    │ 30m        │ DC only  │
│            │          │          │            │          │
│ 1000Base-SX│ 1 Gbps   │ MMF      │ 550m       │ Common   │
│ 1000Base-LX│ 1 Gbps   │ SMF      │ 5km        │ Common   │
│ 10GBase-SR │ 10 Gbps  │ MMF      │ 400m       │ Common   │
│ 10GBase-LR │ 10 Gbps  │ SMF      │ 10km       │ Common   │
│ 40GBase-SR4│ 40 Gbps  │ MMF      │ 150m       │ DC       │
│ 100GBase-SR│ 100 Gbps │ MMF      │ 100m       │ DC       │
└────────────┴──────────┴──────────┴────────────┴──────────┘

NOTATION EXPLAINED:
• Number: Speed in Mbps (10, 100, 1000) or Gbps
• "Base": Baseband signaling (whole cable used)
• T: Twisted pair copper
• S: Short wavelength (multi-mode fiber)
• L: Long wavelength (single-mode fiber)
• Number: Max distance in 100m units (or specified)

POWER OVER ETHERNET (PoE):

┌────────────┬──────────┬──────────┬────────────┐
│ Standard   │ Power    │ Voltage  │ Use        │
├────────────┼──────────┼──────────┼────────────┤
│ 802.3af    │ 15.4W    │ 44-57V   │ Phones, APs│
│ 802.3at    │ 30W      │ 50-57V   │ APs, cameras│
│ 802.3bt    │ 60/100W  │ 50-57V   │ Displays   │
└────────────┴──────────┴──────────┴────────────┘

Delivers power + data over same Ethernet cable
Eliminates need for separate power cables
```

---

## **6) Performance & Tradeoffs**

### **Half-Duplex vs Full-Duplex:**

```
HALF-DUPLEX:
┌─────────────────────────────────────┐
│ Bidirectional, but not simultaneous │
│                                     │
│ A ──────→ B    OR    A ←────── B   │
│                                     │
│ Cannot send and receive at same time│
└─────────────────────────────────────┘

Characteristics:
• CSMA/CD required (collision detection)
• Maximum 50% utilization (best case)
• Used: Legacy hubs, some wireless
• Collision domain shared

FULL-DUPLEX:
┌─────────────────────────────────────┐
│ Bidirectional and simultaneous      │
│                                     │
│ A ──────→ B    AND    A ←────── B  │
│                                     │
│ Send and receive at same time       │
└─────────────────────────────────────┘

Characteristics:
• No collisions possible (separate TX/RX)
• No CSMA/CD needed
• Maximum 100% utilization (both directions)
• Used: Modern switches
• Each port = separate collision domain

COMPARISON:
                Half-Duplex    Full-Duplex
Throughput:     50% max        200% (both directions)
Collisions:     Possible       Impossible
CSMA/CD:        Required       Disabled
Distance:       Limited        Standard
Use:            Legacy         Modern default

EXAMPLE:
100 Mbps Ethernet:

Half-duplex:
• Send: up to 100 Mbps
• Receive: up to 100 Mbps
• But NOT at same time
• Effective: ~50 Mbps aggregate

Full-duplex:
• Send: 100 Mbps
• Receive: 100 Mbps
• Simultaneously
• Effective: 200 Mbps aggregate
```

---

### **Bandwidth vs Latency:**

```
BANDWIDTH (Throughput):
• Amount of data per unit time
• Measured in: bps, Mbps, Gbps
• "Width of the pipe"

LATENCY (Delay):
• Time for signal to travel
• Measured in: milliseconds (ms)
• "Length of the pipe"

ANALOGY:
Bandwidth = highway lanes (more lanes = more cars)
Latency = distance to destination (closer = faster)

DATA LINK LAYER LATENCY COMPONENTS:

1. TRANSMISSION DELAY:
   Time to put frame on wire
   = Frame size / Bandwidth
   
   Example:
   1500 byte frame on 1 Gbps link:
   (1500 × 8 bits) / (1,000,000,000 bps) = 12 μs

2. PROPAGATION DELAY:
   Time for signal to travel medium
   = Distance / Signal speed
   
   Example:
   100m Cat6 cable:
   100m / (2×10^8 m/s) = 0.5 μs
   (Speed of light in copper ~0.67c)

3. QUEUING DELAY:
   Time waiting in buffer
   Variable, depends on congestion

4. PROCESSING DELAY:
   Switch processing time
   Modern: 1-10 μs (store-forward)
   Modern: <1 μs (cut-through)

TOTAL LATENCY:
Transmission + Propagation + Queuing + Processing

COMPARISON:
┌──────────┬────────────┬────────────┐
│ Medium   │ Bandwidth  │ Latency    │
├──────────┼────────────┼────────────┤
│ Ethernet │ High       │ Low (<1ms) │
│ Wi-Fi    │ Medium     │ Low (~5ms) │
│ 4G/LTE   │ Medium     │ Med (~50ms)│
│ Satellite│ High       │ High (600ms)│
└──────────┴────────────┴────────────┘

TRADE-OFFS:
• High bandwidth doesn't mean low latency
• Satellite: High bandwidth, terrible latency
• Local Ethernet: High bandwidth, excellent latency
```

---

### **Error Rates:**

```
BIT ERROR RATE (BER):
Probability of bit flip during transmission

TYPICAL BER:

Wired:
• Fiber optic: 10^-12 (1 error per trillion bits)
• Cat6 Ethernet: 10^-10 to 10^-12
• Very reliable

Wireless:
• Wi-Fi (good signal): 10^-5 to 10^-6
• Wi-Fi (weak signal): 10^-3 to 10^-4
• 4G/LTE: 10^-6 to 10^-8
• Much higher than wired

IMPACT OF ERRORS:

Example: 10^-6 BER (1 error per million bits)

1500 byte frame = 12,000 bits
Probability of error in frame:
P(error) = 1 - (1 - 10^-6)^12000 ≈ 1.2%

At 1 Gbps:
• 83,000 frames/second
• ~996 errored frames/second
• CRC detects and discards these
• Upper layers must retransmit

FRAME LOSS RATE:
Wired Ethernet: <0.01% (very low)
Wi-Fi: 0.1-5% (higher, varies with conditions)

MITIGATION:
• CRC detection (Layer 2)
• TCP retransmission (Layer 4)
• FEC in some specialized links
```

---

## **7) Failure Modes**

### **What breaks at Data Link Layer:**

```
╔════════════════════════════════════════════════════════╗
║ 1. FRAME ERRORS / CORRUPTION                           ║
╚════════════════════════════════════════════════════════╝

Symptoms:
• High error counters on interface
• Packet loss
• Slow performance
• Retransmissions

Causes:
• Bad cable (damaged, loose)
• EMI (Electromagnetic Interference)
• Duplex mismatch
• Bad NIC or switch port

Detection:
$ ethtool -S eth0 | grep error
rx_errors: 1523
rx_crc_errors: 1489
tx_errors: 0

$ show interfaces GigabitEthernet0/1
  5 minute input rate 1000 bits/sec, 2 packets/sec
  5 minute output rate 2000 bits/sec, 4 packets/sec
  12345 packets input, 123456 bytes
  0 input errors, 234 CRC, 0 frame, 0 overrun

CRC errors indicate:
• Bad cable (most common)
• Electrical interference
• Faulty hardware

╔════════════════════════════════════════════════════════╗
║ 2. DUPLEX MISMATCH                                     ║
╚════════════════════════════════════════════════════════╝

Problem: One end full-duplex, other end half-duplex

Scenario:
Port A: Auto-negotiate → Full-duplex
Port B: Forced 100 Mbps Half-duplex

Result:
• Port A thinks no collisions possible
• Port B detects "collisions" (both transmitting)
• Intermittent connectivity
• Very slow speeds
• High error rates

Symptoms:
• Works, but slow
• High late collisions
• Input errors on full-duplex side
• Collisions on half-duplex side

Diagnosis:
Switch A# show interfaces status
Port      Duplex  Speed
Gi0/1     Full    1000

Switch B# show interfaces status
Port      Duplex  Speed
Gi0/1     Half    100   ← MISMATCH!

Fix:
• Set both to auto-negotiate, OR
• Set both to same fixed settings
• Never mix auto + fixed

╔════════════════════════════════════════════════════════╗
║ 3. BROADCAST STORM                                     ║
╚════════════════════════════════════════════════════════╝

Cause: Loop in Layer 2 topology without STP

Flow:
1. PC sends broadcast (e.g., ARP)
2. Switch A forwards to all ports
3. Reaches Switch B via 2 paths
4. Switch B forwards to all ports
5. Back to Switch A
6. Switch A forwards again
7. Exponential growth
8. Network saturated in seconds

Symptoms:
• Network completely unusable
• Link LEDs constantly flashing
• Switches CPU at 100%
• Timeouts everywhere

Detection:
$ show interfaces GigabitEthernet0/1
  Input queue: 75/75/0 (size/max/drops)  ← FULL
  5 minute input rate 950000000 bits/sec ← SATURATED

Prevention:
• Enable STP/RSTP on all switches
• Monitor for STP topology changes

Recovery:
• Unplug cables to break loop
• STP will eventually converge (if enabled)
• Find and fix loop source

╔════════════════════════════════════════════════════════╗
║ 4. MAC TABLE OVERFLOW (CAM Table Exhaustion)          ║
╚════════════════════════════════════════════════════════╝

Attack: Flood switch with frames from random MAC addresses

Process:
1. Attacker sends frames with unique source MACs
2. Switch learns each MAC → fills table
3. Table full (typically 8K-16K entries)
4. Switch fails open → acts like hub
5. Floods all frames to all ports
6. Attacker can sniff all traffic

Symptoms:
• Switch flooding all traffic
• Performance degradation
• Security breach (traffic visible to all)

Detection:
$ show mac address-table count
Dynamic Address Count: 15998
Total Mac Addresses: 16000  ← Nearly full!

Mitigation:
• Port security (limit MACs per port)
• 802.1X authentication
• MAC address aging
• VLAN isolation

Port Security:
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 2
Switch(config-if)# switchport port-security violation shutdown

╔════════════════════════════════════════════════════════╗
║ 5. VLAN HOPPING                                        ║
╚════════════════════════════════════════════════════════╝

Attack: Access VLANs you shouldn't have access to

METHOD 1 - Switch Spoofing:
1. Attacker port configured as access (VLAN 10)
2. Attacker sends DTP (Dynamic Trunking Protocol) frames
3. Switch thinks "Oh, this is a switch, make it trunk"
4. Port becomes trunk
5. Attacker can now send/receive all VLANs

METHOD 2 - Double Tagging:
1. Attacker in VLAN 10 (native VLAN on trunk)
2. Sends frame with 2 VLAN tags:
   [Outer: VLAN 10][Inner: VLAN 20][Data]
3. First switch strips outer tag (VLAN 10, native)
4. Forwards frame with inner tag (VLAN 20)
5. Frame delivered to VLAN 20
6. Bypasses VLAN isolation!

Prevention:
• Disable DTP on access ports
• Change native VLAN to unused VLAN
• Explicitly configure trunk/access modes

Switch(config-if)# switchport mode access
Switch(config-if)# switchport nonegotiate

╔════════════════════════════════════════════════════════╗
║ 6. Wi-Fi SPECIFIC FAILURES                             ║
╚════════════════════════════════════════════════════════╝

HIDDEN NODE PROBLEM:
A and C can't hear each other, both transmit to B
→ Collision at B
→ High retry rate
→ Poor performance

Solution: RTS/CTS (overhead, reduces throughput)

INTERFERENCE:
• 2.4 GHz: Microwaves, Bluetooth, cordless phones
• 5 GHz: Radar (DFS channels), weather radar
• Physical: Metal, water (fish tanks, humans)

Symptoms:
• Intermittent disconnections
• Low signal despite being close
• High retry rates

Detection:
• Wi-Fi analyzer (shows channel utilization)
• Packet captures (retransmissions)

CHANNEL CONTENTION:
Too many devices, too few channels
→ Everyone waiting for airtime
→ High latency
→ Low throughput

Mitigation:
• Use 5 GHz (more channels)
• Reduce AP power (smaller cells)
• More APs with lower power (dense deployment)
```

---

## **8) Real-World Usage**

### **1. Home Network (Typical Setup):**

```
HOME NETWORK TOPOLOGY:

Internet (Cable/DSL/Fiber)
   │
   │
┌──┴──────────────┐
│ ISP Modem/ONT   │
└──┬──────────────┘
   │ Ethernet
┌──┴──────────────┐
│ Wi-Fi Router    │ ← Layer 2 switch + Wi-Fi AP + Router
│ (Combo device)  │
└──┬─┬─┬─┬────────┘
   │ │ │ │
  Wi-Fi │ │ └─ Ethernet ports (built-in switch)
   │    │ └──── Smart TV (wired)
   │    └────── NAS (wired)
   │
   └── Laptops, phones, tablets (wireless)

LAYER 2 FUNCTIONS:

SWITCH PORTION (4-port built-in switch):
• Each Ethernet port = separate collision domain
• MAC address learning per port
• Store-and-forward switching
• Full-duplex 1 Gbps per port

MAC TABLE:
Port  | MAC Address        | VLAN
──────┼───────────────────┼─────
LAN1  | 00:11:22:33:44:55 | 1    (Smart TV)
LAN2  | AA:BB:CC:DD:EE:FF | 1    (NAS)
LAN3  | -                 | -    (Empty)
LAN4  | -                 | -    (Empty)
WLAN  | (multiple)        | 1    (Wi-Fi clients)

Wi-Fi ACCESS POINT:
• SSID: "MyHomeWifi"
• Channel: 6 (2.4 GHz), 36 (5 GHz)
• Security: WPA2-PSK (AES)
• CSMA/CA for media access

FRAME FLOW (Laptop browsing web):

1. Laptop → Router (Wi-Fi):
   SRC MAC: Laptop MAC
   DST MAC: Router Wi-Fi MAC
   Payload: IP packet

2. Router → Internet (Ethernet to modem):
   SRC MAC: Router WAN MAC
   DST MAC: Modem MAC
   Payload: Same IP packet

MAC addresses change at router (Layer 2 boundary)
IP addresses stay same (Layer 3 end-to-end)
```

---

### **2. Enterprise Office (VLAN Segmentation):**

```
ENTERPRISE TOPOLOGY:

          ┌──────────────┐
          │ Core Switch  │
          │ Layer 3      │
          └──────┬───────┘
           ┌─────┴─────┐
           │           │
      ┌────┴───┐   ┌───┴────┐
      │ Dist   │   │  Dist  │
      │ Switch │   │ Switch │
      └────┬───┘   └───┬────┘
    ┌──────┴───┬───────┴──┬─────────┐
    │          │          │         │
┌───┴────┐ ┌──┴─────┐ ┌──┴────┐ ┌──┴────┐
│ Access │ │ Access │ │Access │ │Access │
│Switch 1│ │Switch 2│ │Switch3│ │Switch4│
└───┬────┘ └───┬────┘ └───┬───┘ └───┬───┘
    │          │          │         │
  Users     Servers      VoIP      WiFi APs

VLAN ASSIGNMENT:

VLAN 10 - Users/Workstations:
• Ports: Access switches 1-4, various ports
• Subnet: 10.10.10.0/24
• QoS: Normal priority

VLAN 20 - Servers:
• Ports: Access switch 2, ports 1-12
• Subnet: 10.20.20.0/24
• QoS: Normal priority

VLAN 30 - VoIP Phones:
• Ports: Access switch 3 (+ user desks via voice VLAN)
• Subnet: 10.30.30.0/24
• QoS: HIGH priority (EF/DSCP 46)

VLAN 40 - Guest Wi-Fi:
• Ports: Wi-Fi controllers
• Subnet: 192.168.100.0/24
• QoS: Low priority
• Isolated from corporate VLANs

VLAN 99 - Management:
• Ports: All switch management interfaces
• Subnet: 10.99.99.0/24
• Access: Restricted

TRUNK LINKS:
All inter-switch links carry multiple VLANs

Access Switch 1 → Distribution Switch:
┌────────────────────────────────────┐
│ 802.1Q Trunk                       │
│ Allowed VLANs: 10,30,99            │
│ Native VLAN: 999 (unused)          │
│ Speed: 10 Gbps                     │
└────────────────────────────────────┘

FRAME EXAMPLE (User PC to Server):

User PC (VLAN 10, Port 5, Access Switch 1):
→ Sends untagged frame to switch

Access Switch 1:
→ Receives on access port (VLAN 10)
→ Adds 802.1Q tag (VLAN 10)
→ Forwards on trunk to distribution

Distribution Switch:
→ Receives tagged frame (VLAN 10)
→ Routes to VLAN 20 (Layer 3 function)
→ Tags as VLAN 20
→ Forwards on trunk to Access Switch 2

Access Switch 2:
→ Receives tagged frame (VLAN 20)
→ Removes tag (access port)
→ Delivers untagged to server

End devices never see VLAN tags!
```

---

### **3. Data Center (Leaf-Spine with VXLANs):**

```
MODERN DATA CENTER (Layer 2 overlay on Layer 3 fabric):

          ┌───────┐  ┌───────┐
  SPINE   │Spine 1│  │Spine 2│
          └───┬───┘  └───┬───┘
              │ ╲    ╱   │
              │  ╲  ╱    │
              │   ╲╱     │
              │   ╱╲     │
              │  ╱  ╲    │
          ┌───┴───┐  ┌───┴───┐
  LEAF    │Leaf 1 │  │Leaf 2 │
          └───┬───┘  └───┬───┘
              │          │
          Servers    Servers

UNDERLAY (Layer 3 - IP routing):
• All links routed (no Layer 2 loops)
• BGP or OSPF between leafs/spines
• ECMP for load balancing
• Fast convergence

OVERLAY (VXLAN - Layer 2 over Layer 3):
• Virtual networks (VNIs) spanning leaf switches
• MAC-in-UDP encapsulation
• Allows Layer 2 adjacency across Layer 3 fabric

VXLAN FRAME STRUCTURE:

Original Ethernet Frame (Inner):
[DST MAC][SRC MAC][Type][Data][FCS]
            ↓
VXLAN Encapsulation:
┌────────────────────────────────────────┐
│ Outer Ethernet Header                  │
│ • Outer DST MAC: Next hop switch       │
│ • Outer SRC MAC: Current switch        │
├────────────────────────────────────────┤
│ Outer IP Header                        │
│ • Outer DST IP: Remote VTEP            │
│ • Outer SRC IP: Local VTEP             │
├────────────────────────────────────────┤
│ Outer UDP Header                       │
│ • Dest Port: 4789 (VXLAN)              │
├────────────────────────────────────────┤
│ VXLAN Header (8 bytes)                 │
│ • VNI: 24-bit (16 million networks!)   │
│ • Flags                                │
├────────────────────────────────────────┤
│ Original Frame                         │
│ [Inner DST][Inner SRC][Data]           │
└────────────────────────────────────────┘

EXAMPLE:
VM1 (Leaf 1) → VM2 (Leaf 2), same VNI 5000

1. VM1 sends Ethernet frame
2. Leaf 1 (VTEP) encapsulates:
   • Outer DST IP: Leaf 2 VTEP IP
   • VNI: 5000
   • Inner frame: Original

3. Routed through spine (IP routing)
4. Leaf 2 (VTEP) decapsulates:
   • Checks VNI: 5000
   • Extracts inner frame
   • Delivers to VM2

Layer 2 network (VNI 5000) spans data center!
But transported over Layer 3 fabric
```

---

### **4. Wi-Fi Coffee Shop:**

```
PUBLIC Wi-Fi DEPLOYMENT:

Internet
   │
┌──┴───────────┐
│ Edge Router  │
└──┬───────────┘
   │
┌──┴────────────┐
│ Wi-Fi         │
│ Controller    │
└──┬────────────┘
   │
   ├──┬──┬──┬──
   │  │  │  │
  AP1 AP2 AP3 AP4
   (Coverage area)

ACCESS POINT CONFIGURATION:

SSID: "CoffeeShop-WiFi"
Security: WPA2-Personal (password on wall)
Channel: Auto (controller manages)
Power: Auto (reduces interference)

MULTIPLE SSIDs (Virtual APs):

SSID 1: "CoffeeShop-WiFi" (Customer)
• VLAN 10
• Internet access only
• Bandwidth limit: 5 Mbps per device
• Client isolation enabled

SSID 2: "CoffeeShop-Staff" (Employees)
• VLAN 20
• Access to POS system + Internet
• Higher bandwidth
• WPA2-Enterprise (802.1X)

SSID 3: "CoffeeShop-POS" (Hidden)
• VLAN 30
• POS terminals only
• MAC filtering
• No internet access

CLIENT ISOLATION:
Prevents customers from seeing each other

Customer A ─┐
Customer B  ├─ AP ─→ Internet
Customer C ─┘

A cannot communicate with B or C
All traffic goes through gateway
Security + privacy

CAPTIVE PORTAL:
1. Client connects to Wi-Fi
2. Redirected to splash page
3. Must accept terms/enter code
4. Granted internet access
5. Session tracked by MAC

ROAMING:
Customer walks around:
• AP1 signal weakens
• AP2 signal stronger
• Client reassociates to AP2
• Controller handles handoff
• Seamless (ideally <50ms)

FAST ROAMING (802.11r):
• Pre-authentication with neighboring APs
• Handoff <10ms
• Important for VoIP
```

---

### **5. Industrial Network (Deterministic Ethernet):**

```
FACTORY AUTOMATION:

Control Room
   │
┌──┴─────────────┐
│ SCADA / HMI    │
│ (Monitoring)   │
└──┬─────────────┘
   │
┌──┴─────────────┐
│ Industrial     │
│ Switch (TSN)   │
│ Time-Sensitive │
│ Networking     │
└──┬──┬──┬───────┘
   │  │  │
  PLC │  │ Robot
      │  └─ CNC Machine
      └──── Safety System

TIME-SENSITIVE NETWORKING (TSN):

Traditional Ethernet: Best-effort
• Variable latency (queuing delays)
• No guarantees
• OK for office, bad for robots

TSN Ethernet: Deterministic
• Guaranteed latency (<1ms)
• Time-synchronized (IEEE 1588 PTP)
• Reserved bandwidth per stream
• Scheduled traffic

TRAFFIC CLASSES:

Class 1: Safety-critical (highest priority)
• Emergency stop signals
• Latency: <1ms guaranteed
• Jitter: <50μs
• Reserved bandwidth

Class 2: Control traffic
• PLC commands to robots
• Latency: <5ms guaranteed
• Scheduled transmission

Class 3: Monitoring
• Sensor data collection
• Latency: <100ms
• Lower priority

Class 4: Best-effort
• Firmware updates, diagnostics
• No guarantees

TIME SYNCHRONIZATION:
All devices synchronized to <1μs accuracy
• Know exact time slots for transmission
• No collisions on scheduled traffic
• Deterministic behavior

FRAME EXAMPLE:
Safety stop signal:

┌────────────────────────────────────┐
│ Priority: 7 (highest)              │
│ VLAN: 100 (safety)                 │
│ Scheduled: Time slot 1234          │
│ Payload: EMERGENCY_STOP            │
└────────────────────────────────────┘

Switch: Recognizes priority + schedule
→ Transmits in exact time slot
→ Guaranteed delivery within 500μs
→ Robot responds immediately

REDUNDANCY:
Parallel redundant protocol (PRP):
• Two independent networks
• Send same frame on both
• Receiver accepts first, discards duplicate
• Zero failover time
```

---

## **9) Comparison Section**

### **Ethernet vs Wi-Fi:**

| Feature | Ethernet (802.3) | Wi-Fi (802.11) |
|---------|------------------|----------------|
| **Medium** | Wired (copper/fiber) | Wireless (radio) |
| **Max speed** | 400 Gbps (theoretical) | 9.6 Gbps (Wi-Fi 6) |
| **Typical speed** | 1-10 Gbps | 100-600 Mbps |
| **Range** | 100m (copper), km (fiber) | 50m indoor, 100m outdoor |
| **Reliability** | Very high (BER 10^-12) | Variable (BER 10^-5) |
| **Latency** | <1ms | 5-10ms |
| **Security** | Physical access needed | Encryption required |
| **Installation** | Requires cabling | Easy deployment |
| **Mobility** | No | Yes |
| **Interference** | Minimal (EMI only) | Common (neighbors, devices) |
| **Collisions** | None (full-duplex) | Managed (CSMA/CA) |
| **Cost per port** | $5-20 | $50-200 |
| **Use case** | Servers, desktops, infrastructure | Laptops, phones, IoT |

---

### **Switch Types:**

| Type | Layer | Function | Use Case | Example |
|------|-------|----------|----------|---------|
| **Unmanaged** | 2 | Basic switching, plug-and-play | Home, small office | Netgear GS108 |
| **Managed** | 2 | VLANs, QoS, monitoring | Enterprise access | Cisco Catalyst 2960 |
| **L3 Switch** | 2+3 | Switching + routing | Enterprise distribution/core | Cisco Catalyst 3850 |
| **PoE Switch** | 2 | Power + data | IP phones, cameras, APs | Cisco Catalyst 9300 |
| **Stackable** | 2/3 | Multiple switches as one | Scalable deployments | Arista 7050 |
| **Modular** | 2/3 | Customizable line cards | Large campus/data center | Cisco Catalyst 9600 |

---

### **Wi-Fi Standards:**

| Standard | Name | Year | Frequency | Max Speed | Range | Notes |
|----------|------|------|-----------|-----------|-------|-------|
| **802.11b** | - | 1999 | 2.4 GHz | 11 Mbps | 35m | Legacy |
| **802.11a** | - | 1999 | 5 GHz | 54 Mbps | 35m | Legacy |
| **802.11g** | - | 2003 | 2.4 GHz | 54 Mbps | 38m | Legacy |
| **802.11n** | Wi-Fi 4 | 2009 | 2.4/5 GHz | 600 Mbps | 70m | MIMO |
| **802.11ac** | Wi-Fi 5 | 2014 | 5 GHz | 3.5 Gbps | 35m | MU-MIMO |
| **802.11ax** | Wi-Fi 6 | 2019 | 2.4/5 GHz | 9.6 Gbps | 30m | OFDMA, efficient |
| **802.11ax** | Wi-Fi 6E | 2020 | 6 GHz | 9.6 Gbps | 30m | New spectrum |
| **802.11be** | Wi-Fi 7 | 2024 | 2.4/5/6 GHz | 46 Gbps | TBD | Future |

---

## **10) Packet Walkthrough**

**Scenario:** PC sends ping to server on same switched network

```
════════════════════════════════════════════════════════════
SETUP:
════════════════════════════════════════════════════════════

Topology:
PC A (192.168.1.100) ─── Switch ─── PC B (192.168.1.200)
 MAC: AA:AA:AA:AA:AA:AA            MAC: BB:BB:BB:BB:BB:BB

Switch MAC table initially empty

════════════════════════════════════════════════════════════
STEP 1: ARP REQUEST (PC A needs PC B's MAC)
════════════════════════════════════════════════════════════

PC A creates ARP request:
┌────────────────────────────────────────┐
│ ETHERNET FRAME:                        │
│   Preamble: AA AA AA AA AA AA AA       │
│   SFD: AB                              │
│   DST MAC: FF:FF:FF:FF:FF:FF (broadcast)│
│   SRC MAC: AA:AA:AA:AA:AA:AA (PC A)    │
│   Type: 0x0806 (ARP)                   │
│                                        │
│ ARP PAYLOAD:                           │
│   Operation: Request (1)               │
│   Sender MAC: AA:AA:AA:AA:AA:AA        │
│   Sender IP: 192.168.1.100             │
│   Target MAC: 00:00:00:00:00:00        │
│   Target IP: 192.168.1.200             │
│   Padding: 18 bytes (reach 46 min)     │
│                                        │
│   FCS: [CRC-32 checksum]               │
└────────────────────────────────────────┘

PC A transmits frame to switch

════════════════════════════════════════════════════════════
STEP 2: SWITCH PROCESSES FRAME
════════════════════════════════════════════════════════════

Switch receives on Port 1:
1. Check FCS: Valid ✓
2. Read source MAC: AA:AA:AA:AA:AA:AA
3. LEARNING: Update MAC table
   
   MAC Table:
   Port | MAC Address        | VLAN | Age
   ─────┼───────────────────┼──────┼────
   1    | AA:AA:AA:AA:AA:AA | 1    | 0

4. Read destination MAC: FF:FF:FF:FF:FF:FF (broadcast)
5. FORWARDING DECISION: FLOOD
   → Send to all ports except Port 1

6. Transmit on Port 2, Port 3, Port 4, etc.

════════════════════════════════════════════════════════════
STEP 3: PC B RECEIVES ARP REQUEST
════════════════════════════════════════════════════════════

PC B (Port 2) receives frame:
1. NIC checks destination MAC: FF:FF:FF:FF:FF:FF (broadcast)
   → Accept frame
2. Check FCS: Valid ✓
3. Remove Ethernet header
4. Check EtherType: 0x0806 (ARP)
5. Pass to ARP module

ARP module processes:
• Target IP: 192.168.1.200
• This is MY IP!
• Respond with my MAC

════════════════════════════════════════════════════════════
STEP 4: ARP REPLY (PC B responds)
════════════════════════════════════════════════════════════

PC B creates ARP reply:
┌────────────────────────────────────────┐
│ ETHERNET FRAME:                        │
│   DST MAC: AA:AA:AA:AA:AA:AA (PC A)    │
│   SRC MAC: BB:BB:BB:BB:BB:BB (PC B)    │
│   Type: 0x0806 (ARP)                   │
│                                        │
│ ARP PAYLOAD:                           │
│   Operation: Reply (2)                 │
│   Sender MAC: BB:BB:BB:BB:BB:BB        │
│   Sender IP: 192.168.1.200             │
│   Target MAC: AA:AA:AA:AA:AA:AA        │
│   Target IP: 192.168.1.100             │
│                                        │
│   FCS: [CRC-32 checksum]               │
└────────────────────────────────────────┘

PC B transmits to switch (Port 2)

════════════════════════════════════════════════════════════
STEP 5: SWITCH PROCESSES ARP REPLY
════════════════════════════════════════════════════════════

Switch receives on Port 2:
1. Check FCS: Valid ✓
2. Read source MAC: BB:BB:BB:BB:BB:BB
3. LEARNING: Update MAC table

   MAC Table:
   Port | MAC Address        | VLAN | Age
   ─────┼───────────────────┼──────┼────
   1    | AA:AA:AA:AA:AA:AA | 1    | 5
   2    | BB:BB:BB:BB:BB:BB | 1    | 0

4. Read destination MAC: AA:AA:AA:AA:AA:AA
5. LOOKUP: Found in MAC table → Port 1
6. FORWARDING DECISION: Forward to Port 1 only

7. Transmit on Port 1 (to PC A)

════════════════════════════════════════════════════════════
STEP 6: PC A RECEIVES ARP REPLY
════════════════════════════════════════════════════════════

PC A receives frame:
1. Destination MAC matches: AA:AA:AA:AA:AA:AA ✓
2. Check FCS: Valid ✓
3. EtherType: 0x0806 (ARP)
4. Process ARP reply

ARP cache updated:
192.168.1.200 → BB:BB:BB:BB:BB:BB

Now PC A knows PC B's MAC address!

════════════════════════════════════════════════════════════
STEP 7: ICMP ECHO REQUEST (Ping)
════════════════════════════════════════════════════════════

PC A creates ping packet:
┌────────────────────────────────────────┐
│ ETHERNET FRAME:                        │
│   DST MAC: BB:BB:BB:BB:BB:BB (PC B)    │
│   SRC MAC: AA:AA:AA:AA:AA:AA (PC A)    │
│   Type: 0x0800 (IPv4)                  │
│                                        │
│ IP PACKET:                             │
│   SRC IP: 192.168.1.100                │
│   DST IP: 192.168.1.200                │
│   Protocol: 1 (ICMP)                   │
│   TTL: 64                              │
│                                        │
│ ICMP PAYLOAD:                          │
│   Type: 8 (Echo Request)               │
│   Code: 0                              │
│   Sequence: 1                          │
│   Data: [32 bytes]                     │
│                                        │
│   FCS: [CRC-32 checksum]               │
└────────────────────────────────────────┘

PC A transmits to switch

════════════════════════════════════════════════════════════
STEP 8: SWITCH FORWARDS PING
════════════════════════════════════════════════════════════

Switch receives on Port 1:
1. Check FCS: Valid ✓
2. Source MAC: AA:AA:AA:AA:AA:AA
   → Already in table (Port 1), refresh age
3. Destination MAC: BB:BB:BB:BB:BB:BB
   → Lookup: Port 2
4. Forward to Port 2 only (unicast, not flood)

MAC Table:
Port | MAC Address        | VLAN | Age
─────┼───────────────────┼──────┼────
1    | AA:AA:AA:AA:AA:AA | 1    | 0  ← Refreshed
2    | BB:BB:BB:BB:BB:BB | 1    | 10

════════════════════════════════════════════════════════════
STEP 9: PC B RECEIVES PING, SENDS REPLY
════════════════════════════════════════════════════════════

PC B receives:
1. Destination MAC matches ✓
2. FCS valid ✓
3. Extract IP packet
4. Destination IP matches (192.168.1.200) ✓
5. Protocol: ICMP
6. Type: Echo Request → Respond with Echo Reply

PC B creates ICMP Echo Reply:
┌────────────────────────────────────────┐
│ ETHERNET FRAME:                        │
│   DST MAC: AA:AA:AA:AA:AA:AA (PC A)    │
│   SRC MAC: BB:BB:BB:BB:BB:BB (PC B)    │
│   Type: 0x0800 (IPv4)                  │
│                                        │
│ IP PACKET:                             │
│   SRC IP: 192.168.1.200                │
│   DST IP: 192.168.1.100                │
│   Protocol: 1 (ICMP)                   │
│   TTL: 64                              │
│                                        │
│ ICMP PAYLOAD:                          │
│   Type: 0 (Echo Reply)                 │
│   Code: 0                              │
│   Sequence: 1                          │
│   Data: [same 32 bytes echoed back]    │
│                                        │
│   FCS: [CRC-32 checksum]               │
└────────────────────────────────────────┘

════════════════════════════════════════════════════════════
STEP 10: SWITCH FORWARDS REPLY TO PC A
════════════════════════════════════════════════════════════

Switch receives on Port 2:
1. Source MAC: BB:BB:BB:BB:BB:BB → Refresh age
2. Destination MAC: AA:AA:AA:AA:AA:AA → Port 1
3. Forward to Port 1

════════════════════════════════════════════════════════════
STEP 11: PC A RECEIVES REPLY
════════════════════════════════════════════════════════════

PC A receives:
1. Destination MAC matches ✓
2. FCS valid ✓
3. Extract IP packet → ICMP Echo Reply
4. Display: "Reply from 192.168.1.200: time<1ms"

Ping successful!

════════════════════════════════════════════════════════════
SUMMARY
════════════════════════════════════════════════════════════

Frames exchanged:
1. ARP Request (broadcast) - PC A → Switch → All ports
2. ARP Reply (unicast) - PC B → Switch → PC A
3. ICMP Echo Request - PC A → Switch → PC B
4. ICMP Echo Reply - PC B → Switch → PC A

Switch learned:
• AA:AA:AA:AA:AA:AA on Port 1
• BB:BB:BB:BB:BB:BB on Port 2

Key Layer 2 operations:
✓ MAC learning (source MAC → port mapping)
✓ Forwarding decision (destination MAC lookup)
✓ Flooding (broadcast to all ports)
✓ Unicast forwarding (to specific port)
✓ Error detection (FCS/CRC-32)
✓ Frame creation/removal at each hop

MAC addresses: Changed at each device boundary
IP addresses: Unchanged end-to-end
```

---

## **11) Common Interview / Exam Traps**

### **Misconception 1: "Switch forwards based on IP addresses"**
**Wrong:** Switches use IP addresses for forwarding decisions  
**Right:** **Switches (Layer 2) use MAC addresses.** Routers (Layer 3) use IP addresses. A Layer 3 switch can do both, but its switching function uses MACs.

### **Misconception 2: "MAC addresses are globally unique forever"**
**Wrong:** Every MAC address is unique and never reused  
**Right:** MAC addresses *should* be globally unique (assigned by IEEE OUI), but:
- Can be duplicated (manufacturing errors, virtualization cloning)
- Can be changed (MAC spoofing)
- Locally administered addresses (bit 1 set) are not globally unique
- Uniqueness only matters within a single broadcast domain

### **Misconception 3: "Hubs and switches are the same"**
**Wrong:** Hubs and switches both connect devices, so they're equivalent  
**Right:**
- **Hub (Layer 1):** Repeats signals to all ports, one collision domain, half-duplex, obsolete
- **Switch (Layer 2):** Forwards to specific port based on MAC, each port is separate collision domain, full-duplex, modern

### **Misconception 4: "Ethernet frame has variable header size like IP"**
**Wrong:** Ethernet header size varies based on options  
**Right:** **Ethernet header is fixed at 18 bytes** (excluding preamble/SFD):
- Dest MAC: 6 bytes
- Src MAC: 6 bytes
- Type/Length: 2 bytes
- FCS: 4 bytes
- (802.1Q VLAN tag adds 4 bytes if present)

### **Misconception 5: "CRC can correct errors"**
**Wrong:** CRC both detects and corrects bit errors  
**Right:** **CRC only detects errors, cannot correct them.** When error detected, frame is simply discarded. Upper layers (TCP) handle retransmission.

### **Misconception 6: "Broadcast domain = Collision domain"**
**Wrong:** These terms are interchangeable  
**Right:**
- **Collision domain:** Devices that can collide (conflict on media). Separated by switches.
- **Broadcast domain:** Devices that receive broadcasts. Separated by routers (or VLANs).
- One broadcast domain can contain many collision domains

### **Misconception 7: "Wi-Fi and Ethernet use same frame format"**
**Wrong:** 802.11 and 802.3 frames are identical  
**Right:** Different frame formats:
- **Ethernet:** 2 MAC addresses (src, dst)
- **Wi-Fi:** 3-4 MAC addresses (receiver, transmitter, filtering, optional 4th for WDS)
- Different headers, different error handling

### **Misconception 8: "VLAN tags are visible to end devices"**
**Wrong:** PCs see VLAN tags in frames they receive  
**Right:** **VLAN tags are added/removed by switches on trunk ports.** Access ports (where PCs connect) send/receive untagged frames. End devices typically don't know about VLANs.

---

### **Frequently Asked:**

**Q: What's the difference between a MAC address and an IP address?**  
A:
- **MAC (Layer 2):** Physical address, 48-bit, hex notation, local significance (changes each hop), burned into NIC (but can be changed)
- **IP (Layer 3):** Logical address, 32-bit (IPv4) or 128-bit (IPv6), dotted decimal or hex, global significance (stays same end-to-end), configured by software

**Q: Why does Ethernet have a minimum frame size (64 bytes)?**  
A: **Collision detection in half-duplex.** Device must still be transmitting when collision occurs to detect it. At 10 Mbps over 100m: minimum transmission time must exceed round-trip time (slot time). 64 bytes ensures this. Modern full-duplex doesn't need this, but standard preserved for compatibility.

**Q: What happens if two devices have the same MAC address on a network?**  
A: **Unpredictable behavior:**
- Switch MAC table confused (flaps between ports)
- Frames delivered to wrong device
- Intermittent connectivity
- Difficult to troubleshoot
Rare in practice, but can happen with VM cloning or manual MAC assignment.

**Q: Can a switch forward a frame to the port it was received on?**  
A: **Rare, but yes:**
- **Hairpin turn** (allowed on some switches)
- Useful for Wi-Fi repeaters, certain NAT scenarios
- Most switches filter by default (don't forward back to source port)

**Q: What's the purpose of the preamble in an Ethernet frame?**  
A: **Clock synchronization.** Alternating 1s and 0s (10101010...) allow receiver to:
- Lock onto sender's clock rate
- Know when frame starts (SFD = 10101011)
- Prepare to receive actual data
Not considered part of frame itself.

**Q: How does a switch handle unknown unicast (MAC not in table)?**  
A: **Floods the frame** to all ports except the source port, similar to broadcast. When destination replies, switch learns that MAC address. This is why initial communication on a network may be slower.

**Q: What's the difference between store-and-forward and cut-through switching?**  
A:
- **Store-and-forward:** Receive entire frame, check FCS, then forward. Slower but detects errors.
- **Cut-through:** Start forwarding after reading destination MAC. Faster but no error checking.
- Most modern switches use store-and-forward (error checking worth the minimal delay).

---

## **12) Retrieval Prompts**

### **Fundamentals:**
1. What are the two sublayers of the Data Link Layer and what does each do?
2. Explain the structure of a MAC address and its components
3. What's the difference between unicast, multicast, and broadcast MAC addresses?
4. Why are MAC addresses local (hop-by-hop) while IP addresses are global (end-to-end)?

### **Ethernet:**
5. Walk through an Ethernet II frame structure field by field
6. What's the purpose of the FCS and how does CRC work?
7. Explain the minimum and maximum frame sizes (64 and 1518 bytes)
8. What's 802.1Q VLAN tagging and where is it inserted in the frame?

### **Switching:**
9. How does a switch build its MAC address table?
10. Explain the three forwarding decisions: forward, flood, filter
11. What's the difference between a hub, a switch, and a router?
12. Compare store-and-forward, cut-through, and fragment-free switching

### **Media Access Control:**
13. Explain CSMA/CD and why it's needed in half-duplex Ethernet
14. How does CSMA/CA differ from CSMA/CD and why is it used in Wi-Fi?
15. What's binary exponential backoff?
16. Why don't modern switched networks need CSMA/CD?

### **VLANs:**
17. What problem do VLANs solve?
18. Explain the difference between access and trunk ports
19. How do VLAN tags work (where added, where removed)?
20. What's a native VLAN and why is it used?

### **STP:**
21. What problem does Spanning Tree Protocol solve?
22. Explain the three port states/roles: root port, designated port, blocked port
23. Why is convergence time important in STP?
24. What's the difference between STP and RSTP?

### **Wi-Fi:**
25. Walk through the Wi-Fi association process step by step
26. Explain the hidden node problem and how RTS/CTS solves it
27. Why does Wi-Fi use 3-4 MAC addresses instead of 2?
28. What's the difference between 2.4 GHz and 5 GHz bands?

### **Troubleshooting:**
29. How would you diagnose high CRC errors on an interface?
30. What causes a duplex mismatch and how do you detect it?
31. Explain what a broadcast storm is and how to prevent it
32. How would you detect a MAC flooding attack?

### **Performance:**
33. What's the difference between half-duplex and full-duplex?
34. Calculate transmission delay for a 1500-byte frame on 1 Gbps link
35. Why is Wi-Fi inherently slower than Ethernet for the same bandwidth?
36. What's the bandwidth-delay product for a 100m Ethernet cable?

---

## **13) TL;DR Compression**

**5-bullet summary:**

1. **Data Link Layer = Hop-by-hop reliable delivery** — Provides framing, physical addressing (MAC addresses), error detection (CRC), and media access control; operates locally on each network segment; MAC addresses change at every hop while IP addresses stay constant end-to-end; ensures error-free transmission over potentially unreliable physical medium

2. **MAC addresses and framing:**
   - **MAC:** 48-bit physical address (AA:BB:CC:DD:EE:FF), first 24 bits = OUI (vendor), last 24 bits = device ID
   - **Frame:** Preamble + Dest MAC + Src MAC + Type/Length + Payload (46-1500 bytes) + FCS (CRC-32)
   - **Special MACs:** FF:FF:FF:FF:FF:FF (broadcast), 01:xx:xx:xx:xx:xx (multicast)
   - **Error detection:** CRC detects errors, frame discarded if corrupted, no correction at Layer 2

3. **Switching operation:**
   - **Learning:** Read source MAC, map to incoming port, build MAC address table
   - **Forwarding:** Read destination MAC, lookup in table, forward to specific port
   - **Flooding:** Unknown unicast or broadcast → send to all ports except source
   - **Each port = separate collision domain (full-duplex), all ports = one broadcast domain (until router/VLAN)**

4. **Media access control:**
   - **CSMA/CD (Ethernet half-duplex):** Listen before transmit, detect collisions, backoff and retry; obsolete in modern full-duplex switching
   - **CSMA/CA (Wi-Fi):** Avoid collisions via random backoff, RTS/CTS for hidden nodes, ACK required for reliability
   - **Modern Ethernet:** Full-duplex eliminates collisions, dedicated bandwidth per port, no contention

5. **VLANs and STP:**
   - **VLANs:** Logical network segmentation; 802.1Q tags (4 bytes) carry VLAN ID (1-4094); access ports untagged, trunk ports tagged; separates broadcast domains without physical isolation
   - **STP:** Prevents Layer 2 loops by blocking redundant paths; elects root bridge, calculates shortest paths, designates port roles; RSTP converges faster (~1s vs ~50s)
   - **Spanning Tree needed:** Without it, loops cause broadcast storms that saturate network in seconds

**One-sentence essence:**
The Data Link Layer ensures reliable hop-by-hop delivery between directly connected devices using MAC addresses for physical addressing, switches that learn and forward based on these addresses, error detection via CRC, media access protocols (CSMA/CD for Ethernet, CSMA/CA for Wi-Fi) to manage shared medium contention, VLANs for logical network segmentation within physical infrastructure, and Spanning Tree Protocol to prevent catastrophic loops while maintaining redundant paths for fault tolerance — all operating locally on each network segment while Layer 3 handles the end-to-end routing across multiple segments.