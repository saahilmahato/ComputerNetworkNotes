# IP Addresses: The Internet's Addressing System

## The Problem: Finding Devices on a Global Network

Imagine you're in New York and want to send a letter to someone in Tokyo. You write their name on the envelope:

```
"Takashi Yamamoto"
```

**This won't work.** Why?

- Millions of people might have that name
- Postal workers don't know where every person lives
- Even if they did, tracking billions of people is impossible

**The solution:** Add location information:

```
Takashi Yamamoto
3-2-1 Shibuya
Shibuya-ku
Tokyo 150-0002
Japan
```

Now postal workers can route your letter:
1. Sort to "Japan" (country)
2. Sort to "Tokyo" (city)
3. Sort to "Shibuya-ku" (district)
4. Sort to "3-2-1 Shibuya" (street)
5. Deliver to "Takashi Yamamoto" (person)

**IP addresses work the same way.**

---

## The Internet's Routing Problem

The Internet in 2025:
- **~5 billion devices** connected
- **100,000+ networks** (ISPs, companies, universities)
- **Millions of routers** forwarding packets

**Impossible approach:**

```
Every router stores a route to every device:

Router's table:
  Device 1 (192.168.1.5)     → Gateway A
  Device 2 (10.0.0.25)       → Gateway B
  Device 3 (172.16.5.100)    → Gateway C
  ...
  Device 5,000,000,000       → Gateway Z

Problems:
  ✗ Requires petabytes of memory per router
  ✗ Updates every time a device turns on/off
  ✗ Route lookup takes forever
  ✗ Completely unscalable
```

**Scalable approach: Hierarchical routing**

```
Router only stores routes to networks, not devices:

Router's table:
  Network 192.168.1.0/24     → Gateway A (256 devices)
  Network 10.0.0.0/8         → Gateway B (16 million devices)
  Network 172.16.0.0/16      → Gateway C (65,536 devices)
  ...

Benefits:
  ✓ ~1 million routes instead of billions
  ✓ Manageable memory (~1 GB)
  ✓ Fast lookups (microseconds)
  ✓ Scales indefinitely
```

**This is why IP addresses must encode location, not just identity.**

---

## What is an IP Address?

### The Fundamental Distinction

**MAC Address (Identity):**
```
Who are you?
  MAC: AA:BB:CC:DD:EE:FF
  
This is like your fingerprint—unique but doesn't tell where you are.
```

**IP Address (Location):**
```
Where are you?
  IP: 192.168.1.10
  
This is like your postal address—tells how to reach you.
```

**Key insight:**

> An IP address is a **routing coordinate**, not a name.

It contains two pieces of information:
1. **Network ID** — Which network are you on?
2. **Host ID** — Which device on that network are you?

---

## IPv4 Address Structure

### Human-Readable Format (Dotted-Decimal)

```
192.168.1.10
 │   │   │ │
 │   │   │ └─ 10 (0-255)
 │   │   └─── 1 (0-255)
 │   └─────── 168 (0-255)
 └─────────── 192 (0-255)

4 bytes (octets) separated by dots
Each byte: 0-255 (8 bits)
Total: 32 bits
```

**Total possible addresses:**

```
2^32 = 4,294,967,296 addresses (~4.3 billion)
```

This seemed like a lot in 1981. It's insufficient today (hence IPv6).

---

### The Binary Reality

**What humans see:**
```
192.168.1.10
```

**What computers see:**
```
11000000.10101000.00000001.00001010
│      │ │      │ │      │ │      │
Byte 1   Byte 2   Byte 3   Byte 4
```

**Conversion:**

```
Decimal 192 = 128 + 64 = 2^7 + 2^6
Binary:  11000000

Decimal 168 = 128 + 32 + 8 = 2^7 + 2^5 + 2^3
Binary:  10101000

Decimal 1 = 2^0
Binary:  00000001

Decimal 10 = 8 + 2 = 2^3 + 2^1
Binary:  00001010
```

**Why binary matters:**

All routing operations (network matching, subnet calculations) happen on **bits**, not decimal numbers. Understanding binary is essential for understanding IP addressing.

---

## The Network/Host Division

### Why Addresses Must Be Split

**Router's job:**
```
Packet arrives with destination: 192.168.1.10

Router's question: "Which interface should I forward this out of?"

Router does NOT care about:
  • What specific device this is
  • What application will receive it
  • Who owns the device

Router ONLY cares about:
  • Which network is this device on?
  • Which path leads to that network?
```

**This requires splitting the address:**

```
IP Address: 192.168.1.10

Split into:
  Network part:  192.168.1     (shared by all devices on this network)
  Host part:     10             (unique to this device on this network)
```

**Postal analogy:**

```
Address: "Tokyo, Shibuya-ku, 3-2-1, Apartment 10"

Splits into:
  Network part:  Tokyo, Shibuya-ku, 3-2-1  (delivery route)
  Host part:     Apartment 10                (specific destination)

Mail carrier routes to the building (network), then finds apartment (host)
```

---

### The Problem: How to Know Where to Split?

**Given just the IP address `192.168.1.10`, where does the network part end?**

**Option A:**
```
Network:  192.168.1     (24 bits)
Host:     10             (8 bits)
→ Network can have 256 devices
```

**Option B:**
```
Network:  192.168       (16 bits)
Host:     1.10           (16 bits)
→ Network can have 65,536 devices
```

**Option C:**
```
Network:  192           (8 bits)
Host:     168.1.10       (24 bits)
→ Network can have 16,777,216 devices
```

**All are valid splits!** Without additional information, the IP address alone is meaningless.

**This is why subnet masks exist.**

---

## Subnet Masks: Defining the Split

### What a Subnet Mask Is

A **subnet mask** is a 32-bit number that specifies which bits of an IP address represent the network.

**Structure rule:**
- All network bits = `1`
- All host bits = `0`
- Network bits always come first (no mixing allowed)

**Example:**

```
IP Address:     192.168.1.10
Subnet Mask:    255.255.255.0

In binary:
IP:     11000000.10101000.00000001.00001010
Mask:   11111111.11111111.11111111.00000000
        ├─────────────Network─────────┤Host│
        
The mask says: "First 24 bits are network, last 8 bits are host"
```

---

### Common Subnet Masks

| Mask | Binary | Network Bits | Host Bits | Hosts per Network |
|------|--------|--------------|-----------|-------------------|
| 255.0.0.0 | 11111111.00000000.00000000.00000000 | 8 | 24 | 16,777,214 |
| 255.255.0.0 | 11111111.11111111.00000000.00000000 | 16 | 16 | 65,534 |
| 255.255.255.0 | 11111111.11111111.11111111.00000000 | 24 | 8 | 254 |
| 255.255.255.128 | 11111111.11111111.11111111.10000000 | 25 | 7 | 126 |
| 255.255.255.192 | 11111111.11111111.11111111.11000000 | 26 | 6 | 62 |
| 255.255.255.224 | 11111111.11111111.11111111.11100000 | 27 | 5 | 30 |
| 255.255.255.240 | 11111111.11111111.11111111.11110000 | 28 | 4 | 14 |
| 255.255.255.252 | 11111111.11111111.11111111.11111100 | 30 | 2 | 2 |

---

## The Bitwise AND Operation: Computing the Network Address

### How Routers Determine the Network

**The operation every router performs:**

```
Network Address = IP Address AND Subnet Mask
```

**Bitwise AND rules:**
```
1 AND 1 = 1
1 AND 0 = 0
0 AND 1 = 0
0 AND 0 = 0

Summary: Result is 1 only if both bits are 1
```

---

### Example Calculation

**Given:**
```
IP Address:    192.168.1.10
Subnet Mask:   255.255.255.0
```

**Step 1: Convert to binary**
```
IP:     11000000.10101000.00000001.00001010
Mask:   11111111.11111111.11111111.00000000
```

**Step 2: Perform bitwise AND**
```
        11000000.10101000.00000001.00001010  (IP)
AND     11111111.11111111.11111111.00000000  (Mask)
        ─────────────────────────────────────
Result: 11000000.10101000.00000001.00000000
```

**Step 3: Convert result back to decimal**
```
Network Address: 192.168.1.0
```

**What this means:**

All devices with IPs from `192.168.1.0` to `192.168.1.255` belong to the **same network** (when using mask 255.255.255.0).

---

### Why This Operation Matters

**Routers use this to make forwarding decisions:**

```
Routing table entry:
  Network: 192.168.1.0/24 → Forward to Interface eth0

Packet arrives for: 192.168.1.50

Router computes:
  192.168.1.50 AND 255.255.255.0 = 192.168.1.0
  
Router checks table:
  192.168.1.0 matches entry!
  → Forward to eth0
```

This operation happens **billions of times per second** across the Internet.

---

## How Hosts Use Subnet Masks: Local vs. Remote

### The Decision Algorithm

When your computer wants to send a packet, it asks:

**"Is the destination on my local network, or on a remote network?"**

**Algorithm:**

```
1. Compute my network address:
   My IP AND My Subnet Mask = My Network

2. Compute destination's network address:
   Destination IP AND My Subnet Mask = Destination Network

3. Compare:
   If My Network == Destination Network:
     → Send directly (use ARP to find MAC address)
   Else:
     → Send to default gateway (router)
```

---

### Example 1: Local Destination

**Your computer:**
```
IP:   192.168.1.10
Mask: 255.255.255.0
```

**Destination:**
```
IP: 192.168.1.25
```

**Calculation:**

```
Your network:
  192.168.1.10 AND 255.255.255.0 = 192.168.1.0

Destination network:
  192.168.1.25 AND 255.255.255.0 = 192.168.1.0

Compare: 192.168.1.0 == 192.168.1.0 ✓

Decision: Local! Send directly via Ethernet
```

---

### Example 2: Remote Destination

**Your computer:**
```
IP:   192.168.1.10
Mask: 255.255.255.0
Gateway: 192.168.1.1
```

**Destination:**
```
IP: 8.8.8.8 (Google DNS)
```

**Calculation:**

```
Your network:
  192.168.1.10 AND 255.255.255.0 = 192.168.1.0

Destination network:
  8.8.8.8 AND 255.255.255.0 = 8.8.8.0

Compare: 192.168.1.0 ≠ 8.8.8.0 ✗

Decision: Remote! Send to gateway (192.168.1.1)
```

The gateway (router) then forwards the packet toward the Internet.

---

## CIDR Notation: A Better Way to Write Subnet Masks

### The Problem with Dotted-Decimal

```
Which is clearer?

Option A: 192.168.1.0 with subnet mask 255.255.255.0
Option B: 192.168.1.0/24
```

Option B is **CIDR notation** (Classless Inter-Domain Routing).

---

### How CIDR Works

**Format:**
```
IP_Address/Prefix_Length

192.168.1.0/24
           ││
           │└─ Prefix length (number of network bits)
           └── Network address
```

**Prefix length directly tells you the mask:**

```
/24 means:
  • First 24 bits = Network
  • Last 8 bits = Host
  • Subnet mask = 255.255.255.0

/26 means:
  • First 26 bits = Network
  • Last 6 bits = Host
  • Subnet mask = 255.255.255.192
```

---

### Common CIDR Prefixes

| CIDR | Subnet Mask | Network Bits | Host Bits | Usable Hosts |
|------|-------------|--------------|-----------|--------------|
| /8 | 255.0.0.0 | 8 | 24 | 16,777,214 |
| /16 | 255.255.0.0 | 16 | 16 | 65,534 |
| /24 | 255.255.255.0 | 24 | 8 | 254 |
| /25 | 255.255.255.128 | 25 | 7 | 126 |
| /26 | 255.255.255.192 | 26 | 6 | 62 |
| /27 | 255.255.255.224 | 27 | 5 | 30 |
| /28 | 255.255.255.240 | 28 | 4 | 14 |
| /29 | 255.255.255.248 | 29 | 3 | 6 |
| /30 | 255.255.255.252 | 30 | 2 | 2 |

---

### Why /30 for Point-to-Point Links

```
Network: 10.0.0.0/30

Available addresses: 2^2 = 4
  10.0.0.0   → Network address (unusable)
  10.0.0.1   → Router A
  10.0.0.2   → Router B
  10.0.0.3   → Broadcast address (unusable)

Perfect for connecting two routers!
```

---

## Calculating Network Information

### The Essential Formulas

Given a network with prefix length `n`:

**Number of host bits:**
```
h = 32 - n
```

**Total addresses in network:**
```
Total = 2^h
```

**Usable host addresses:**
```
Usable = 2^h - 2
```

**Why subtract 2?**

Two addresses are reserved:
1. **Network address** (all host bits = 0)
2. **Broadcast address** (all host bits = 1)

---

### Example: Calculate Network Details

**Given:** `192.168.10.0/26`

**Step 1: Find host bits**
```
n = 26
h = 32 - 26 = 6 bits
```

**Step 2: Total addresses**
```
2^6 = 64 addresses
```

**Step 3: Usable addresses**
```
64 - 2 = 62 usable host addresses
```

**Step 4: Determine address range**

```
Network address:       192.168.10.0     (all host bits = 000000)
First usable address:  192.168.10.1     (host bits = 000001)
Last usable address:   192.168.10.62    (host bits = 111110)
Broadcast address:     192.168.10.63    (all host bits = 111111)
```

**Step 5: Write subnet mask**

```
26 network bits means:
  11111111.11111111.11111111.11000000
  
In decimal:
  255.255.255.192
```

---

### Quick Method: Block Size

**Block size** = Size of the subnet in the relevant octet

**Formula:**
```
Block size = 256 - (value in relevant octet of mask)
```

**Example: 255.255.255.192**

```
Relevant octet: 192
Block size: 256 - 192 = 64

Networks increment by 64:
  192.168.10.0/26   (0 - 63)
  192.168.10.64/26  (64 - 127)
  192.168.10.128/26 (128 - 191)
  192.168.10.192/26 (192 - 255)
```

---

## Subnetting: Dividing Networks

### Why Subnet?

**Scenario:** You have network `192.168.1.0/24` (254 hosts) but need to create 4 separate networks.

**Reasons to subnet:**

```
1. Limit broadcast domains
   • Broadcasts (ARP, DHCP) only within subnet
   • Reduces network noise

2. Security isolation
   • Separate HR from Engineering
   • Apply firewall rules between subnets

3. Efficient address usage
   • Don't waste a /24 (254 hosts) on a network with 10 devices
   • Subnet into smaller blocks

4. Geographic organization
   • Floor 1: 192.168.1.0/26
   • Floor 2: 192.168.1.64/26
   • Floor 3: 192.168.1.128/26
```

---

### Subnetting Process

**Goal:** Divide `192.168.1.0/24` into 4 equal subnets.

**Step 1: Determine how many subnet bits needed**

```
Need 4 subnets
2^x ≥ 4
x = 2 bits

We'll borrow 2 bits from the host portion
```

**Step 2: Calculate new prefix length**

```
Original: /24 (24 network bits, 8 host bits)
Borrow: 2 bits
New prefix: /26 (26 network bits, 6 host bits)
```

**Step 3: Calculate subnet details**

```
Each subnet:
  Host bits: 6
  Addresses per subnet: 2^6 = 64
  Usable hosts: 62
```

**Step 4: List the subnets**

```
Subnet 1: 192.168.1.0/26     (192.168.1.0 - 192.168.1.63)
Subnet 2: 192.168.1.64/26    (192.168.1.64 - 192.168.1.127)
Subnet 3: 192.168.1.128/26   (192.168.1.128 - 192.168.1.191)
Subnet 4: 192.168.1.192/26   (192.168.1.192 - 192.168.1.255)
```

---

### Subnet Assignment Example

**Company network: `192.168.1.0/24`**

**Requirements:**
- Engineering: 50 hosts
- Sales: 25 hosts
- HR: 10 hosts
- Guest WiFi: 20 hosts

**Solution:**

```
Engineering: 192.168.1.0/26
  • Usable: 62 hosts ✓ (50 needed)
  • Range: 192.168.1.1 - 192.168.1.62
  • Gateway: 192.168.1.1

Sales: 192.168.1.64/26
  • Usable: 62 hosts ✓ (25 needed)
  • Range: 192.168.1.65 - 192.168.1.126
  • Gateway: 192.168.1.65

HR: 192.168.1.128/27
  • Usable: 30 hosts ✓ (10 needed)
  • Range: 192.168.1.129 - 192.168.1.158
  • Gateway: 192.168.1.129

Guest WiFi: 192.168.1.160/27
  • Usable: 30 hosts ✓ (20 needed)
  • Range: 192.168.1.161 - 192.168.1.190
  • Gateway: 192.168.1.161

Reserved for future: 192.168.1.192/26
```

---

## Special IP Addresses

### Reserved Address Ranges (Private Networks)

**RFC 1918 private addresses** (not routed on public Internet):

```
10.0.0.0/8
  • Range: 10.0.0.0 - 10.255.255.255
  • Addresses: 16,777,216
  • Use: Large enterprises, cloud providers

172.16.0.0/12
  • Range: 172.16.0.0 - 172.31.255.255
  • Addresses: 1,048,576
  • Use: Medium networks

192.168.0.0/16
  • Range: 192.168.0.0 - 192.168.255.255
  • Addresses: 65,536
  • Use: Home networks, small businesses
```

**These can be used freely inside your network** but require NAT to access the Internet.

---

### Special Purpose Addresses

```
0.0.0.0
  • "This network" or "any address"
  • Used in routing to mean "default route"

127.0.0.0/8 (Loopback)
  • 127.0.0.1 is "localhost"
  • Packets sent here stay on the same machine
  • Used for testing

169.254.0.0/16 (Link-Local)
  • Auto-assigned when DHCP fails
  • Only works on local network segment
  • "APIPA" addresses on Windows

224.0.0.0/4 (Multicast)
  • Used for one-to-many communication
  • Example: 224.0.0.1 = "all hosts on this subnet"

255.255.255.255 (Limited Broadcast)
  • Broadcast to all devices on local network
  • Not forwarded by routers
```

---

## Complete Worked Example

### Problem: Design a Network

**Given:**
- Network: `172.16.0.0/16`
- Requirements:
  - 3 departments (Sales, Engineering, HR)
  - Sales needs 500 hosts
  - Engineering needs 1000 hosts
  - HR needs 100 hosts

**Constraints:**
- Use CIDR
- Minimize waste
- Leave room for growth

---

### Solution

**Step 1: Determine subnet sizes**

```
Engineering: 1000 hosts needed
  • Find n where 2^n - 2 ≥ 1000
  • 2^10 = 1024, 1024 - 2 = 1022 ✓
  • Need /22 (32 - 10 = 22)

Sales: 500 hosts needed
  • 2^9 = 512, 512 - 2 = 510 ✓
  • Need /23

HR: 100 hosts needed
  • 2^7 = 128, 128 - 2 = 126 ✓
  • Need /25
```

**Step 2: Assign networks (largest first)**

```
Engineering: 172.16.0.0/22
  • Network: 172.16.0.0
  • First usable: 172.16.0.1
  • Last usable: 172.16.3.254
  • Broadcast: 172.16.3.255
  • Mask: 255.255.252.0

Sales: 172.16.4.0/23
  • Network: 172.16.4.0
  • First usable: 172.16.4.1
  • Last usable: 172.16.5.254
  • Broadcast: 172.16.5.255
  • Mask: 255.255.254.0

HR: 172.16.6.0/25
  • Network: 172.16.6.0
  • First usable: 172.16.6.1
  • Last usable: 172.16.6.126
  • Broadcast: 172.16.6.127
  • Mask: 255.255.255.128
```

**Step 3: Configure devices**

```
Engineering Department:
  Router:      172.16.0.1/22
  Server:      172.16.0.10/22
  Workstation: 172.16.0.100/22
  Default GW:  172.16.0.1

Sales Department:
  Router:      172.16.4.1/23
  Server:      172.16.4.10/23
  Workstation: 172.16.4.100/23
  Default GW:  172.16.4.1

HR Department:
  Router:      172.16.6.1/25
  Workstation: 172.16.6.10/25
  Default GW:  172.16.6.1
```

**Step 4: Configure routing**

```
Router needs routes to all subnets:
  172.16.0.0/22   → Interface eth0 (Engineering)
  172.16.4.0/23   → Interface eth1 (Sales)
  172.16.6.0/25   → Interface eth2 (HR)
  0.0.0.0/0       → Interface eth3 (Internet default route)
```

---

## Why This Design Scales

### Hierarchical Aggregation

**Without hierarchy:**

```
Internet routing table:
  Device 1: 192.168.1.1
  Device 2: 192.168.1.2
  Device 3: 192.168.1.3
  ...
  (Billions of entries)
```

**With hierarchy:**

```
Internet routing table:
  192.168.0.0/16 → ISP A
  10.0.0.0/8     → ISP B
  172.16.0.0/12  → ISP C
  ...
  (Hundreds of thousands of entries)

ISP A's routing table:
  192.168.1.0/24 → Customer X
  192.168.2.0/24 → Customer Y
  ...
```

**Route aggregation example:**

```
Before:
  192.168.0.0/24   → Gateway 1
  192.168.1.0/24   → Gateway 1
  192.168.2.0/24   → Gateway 1
  192.168.3.0/24   → Gateway 1
  (4 routes)

After aggregation:
  192.168.0.0/22   → Gateway 1
  (1 route covering all 4 networks)
```

This is called **supernetting** or **route summarization**.

---

## Mental Model: The Tree of Networks

Think of IP addressing as a tree:

```
                    0.0.0.0/0 (Internet)
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    10.0.0.0/8      172.16.0.0/12    192.168.0.0/16
        │                │                │
   ┌────┴────┐      ┌────┴────┐      ┌────┴────┐
10.0.0.0/16  10.1.0.0/16  ...    192.168.0.0/24  192.168.1.0/24
   │              │                   │              │
   └──────┐       └──────┐        ┌───┴───┐      ┌───┴───┐
      10.0.0.0/24    10.1.0.0/24  .0.0  .0.128  .1.0  .1.128
          │              │        /25    /25    /25    /25
       Hosts          Hosts     Hosts  Hosts  Hosts  Hosts

Each level narrows down location
Routers only store the branches they need
```

---

## Summary: Key Concepts

**1. IP addresses encode location (not identity)**
```
Network part + Host part = Routing coordinate
```

**2. Subnet masks define the boundary**
```
Mask = Where network ends and host begins
```

**3. Bitwise AND is the core operation**
```
IP AND Mask = Network address
Used by every router, every host, billions of times per second
```

**4. CIDR notation is standard**
```
192.168.1.0/24 > 192.168.1.0 with mask 255.255.255.0
```

**5. Subnetting creates hierarchy**
```
Borrow host bits → Create smaller networks
Enables organization, security, efficiency
```

**6. Special addresses serve specific purposes**
```
Private networks (RFC 1918), loopback, broadcast, multicast
```

**7. Hierarchical routing scales the Internet*
```
Aggregation reduces routing table size
Makes global routing possible
```

---

## The Critical Understanding

> Without subnet masks, IP routing cannot exist.

Every device and every router on the Internet uses subnet masks to determine:
- "Is this destination local or remote?"
- "Which interface should I forward this packet to?"
- "Should I send this to my default gateway?"

Understanding IP addressing at the binary level—particularly the bitwise AND operation—is fundamental to understanding how the Internet actually works.

