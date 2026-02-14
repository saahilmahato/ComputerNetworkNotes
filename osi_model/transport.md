# Transport Layer (Layer 4)

---

## **1) Concept Snapshot**

**Definition:**
The Transport Layer provides end-to-end communication services for applications, offering reliable or unreliable delivery of data segments between processes running on different hosts through port-based multiplexing, error detection, flow control, and congestion management.

**Purpose:**
- **Process-to-process delivery:** Identify specific applications using port numbers
- **Segmentation & Reassembly:** Break large data into segments, reconstruct at destination
- **Reliability (optional):** Guarantee delivery with acknowledgments and retransmissions (TCP)
- **Flow control:** Prevent sender from overwhelming receiver
- **Congestion control:** Prevent network overload
- **Error detection:** Verify data integrity with checksums

**Key insight:** This is the layer that makes the **unreliable network appear reliable** (with TCP) or provides a **simple, fast, unreliable service** (with UDP). It's the last layer before data enters the "network wilderness" and the first to see it emerge on the other side.

---

## **2) Mental Model**

**Real-world analogy:**
Think of a postal service for apartment buildings:

- **Network Layer (L3):** Gets package to the right building (IP address = building address)
- **Transport Layer (L4):** Delivers to the right apartment (Port number = apartment number)
  - **TCP = Certified Mail:** Signature required, tracking, guaranteed delivery, can resend if lost
  - **UDP = Regular Mail:** Drop in mailbox, no confirmation, faster, might get lost

**Visual intuition:**
```
APPLICATION LAYER:
┌─────────────────────────────────────┐
│ Web Server (Port 80)                │
│ Email Server (Port 25)              │
│ SSH Server (Port 22)                │
└──────────────┬──────────────────────┘
               ↓
TRANSPORT LAYER (L4):
┌─────────────────────────────────────┐
│ TCP: Reliable, ordered, slow        │
│ UDP: Unreliable, fast               │
│                                     │
│ Ports: Identify applications        │
│ Segments: Data units                │
│ Flow control: Don't overwhelm       │
│ Error detection: Checksums          │
└──────────────┬──────────────────────┘
               ↓
NETWORK LAYER (L3):
┌─────────────────────────────────────┐
│ IP: Routes packets between networks │
└─────────────────────────────────────┘
```

**Simplified story:**
When you stream a video, the Transport Layer breaks the video file into thousands of small segments. TCP ensures every segment arrives in order, resending any that get lost. For a video call (where speed matters more than perfection), UDP sends video frames as fast as possible without waiting for confirmations—if a frame is lost, just move on to the next one.

---

## **3) Layer Context**

**Position in OSI/TCP-IP stack:**
```
OSI Model              TCP/IP Model
─────────────          ────────────
Application (L7)   ┐
Presentation (L6)  ├──→ Application
Session (L5)       ┘
Transport (L4)     ───→ Transport    ← WE ARE HERE
Network (L3)       ───→ Internet
Data Link (L2)     ┐
Physical (L1)      ├──→ Network Access
                   ┘
```

**Who talks to it:**
- **Above:** Application Layer protocols (HTTP, FTP, DNS, etc.)
- **Below:** Network Layer (IP)

**Key protocols:**
```
╔════════════════════════════════════════════════════════╗
║ PRIMARY TRANSPORT PROTOCOLS                            ║
╠════════════════════════════════════════════════════════╣
║ TCP (Transmission Control Protocol)                   ║
║ • Reliable, connection-oriented                        ║
║ • Guarantees delivery and order                        ║
║ • ~95% of internet traffic                             ║
╠════════════════════════════════════════════════════════╣
║ UDP (User Datagram Protocol)                           ║
║ • Unreliable, connectionless                           ║
║ • Fast, low overhead                                   ║
║ • ~5% of internet traffic                              ║
╠════════════════════════════════════════════════════════╣
║ NEWER PROTOCOLS                                        ║
║ • SCTP (Stream Control Transmission Protocol)          ║
║ • DCCP (Datagram Congestion Control Protocol)          ║
║ • QUIC (Quick UDP Internet Connections)                ║
╚════════════════════════════════════════════════════════╝
```

**PDU (Protocol Data Unit):**
- Layer 4 PDU = **Segment** (TCP) or **Datagram** (UDP)

---

## **4) Mechanics (How It Actually Works)**

### **PORT NUMBERS (Process Identification)**

**The fundamental problem:**
- IP address identifies the **host** (computer)
- But one host runs **multiple applications**
- How do we deliver data to the **correct application**?

**Solution: Port numbers**

```
PORT NUMBER STRUCTURE:
16-bit number: 0 - 65,535

PORT RANGES:
┌─────────────────────────────────────────────┐
│ 0 - 1023: Well-Known Ports                  │
│ • Reserved for standard services            │
│ • Require admin/root privileges             │
│ • Examples: 80 (HTTP), 443 (HTTPS),         │
│             22 (SSH), 25 (SMTP)             │
├─────────────────────────────────────────────┤
│ 1024 - 49151: Registered Ports              │
│ • Assigned by IANA for specific services    │
│ • Examples: 3306 (MySQL), 5432 (PostgreSQL),│
│             8080 (HTTP alternate)           │
├─────────────────────────────────────────────┤
│ 49152 - 65535: Dynamic/Ephemeral Ports      │
│ • Used by clients for temporary connections │
│ • OS assigns automatically                  │
│ • Example: Your browser uses random port    │
│             from this range                 │
└─────────────────────────────────────────────┘
```

**Common well-known ports:**
```
20, 21  → FTP (File Transfer Protocol)
22      → SSH (Secure Shell)
23      → Telnet
25      → SMTP (Email sending)
53      → DNS (Domain Name System)
80      → HTTP (Web traffic)
110     → POP3 (Email retrieval)
143     → IMAP (Email retrieval)
443     → HTTPS (Secure web)
3389    → RDP (Remote Desktop Protocol)
```

**Socket = IP Address + Port**
```
Full address for communication:

192.168.1.100:54321  ←→  93.184.216.34:80
   (Client)                  (Server)
     ↓                          ↓
Your computer              example.com
Random port 54321          HTTP port 80

This combination is called a SOCKET PAIR
Uniquely identifies a connection
```

---

## **TCP (Transmission Control Protocol)**

### **TCP Characteristics:**

```
╔════════════════════════════════════════════════════════╗
║ TCP FEATURES                                           ║
╠════════════════════════════════════════════════════════╣
║ ✓ CONNECTION-ORIENTED (3-way handshake)                ║
║ ✓ RELIABLE (acknowledgments + retransmissions)         ║
║ ✓ ORDERED (sequence numbers)                           ║
║ ✓ FLOW CONTROL (sliding window)                        ║
║ ✓ CONGESTION CONTROL (slow start, AIMD)                ║
║ ✓ FULL-DUPLEX (bidirectional simultaneously)           ║
║ ✓ ERROR DETECTION (checksums)                          ║
║                                                        ║
║ ✗ SLOWER (overhead from reliability mechanisms)        ║
║ ✗ MORE COMPLEX (state management)                      ║
║ ✗ HIGHER LATENCY (handshake + acknowledgments)         ║
╚════════════════════════════════════════════════════════╝
```

---

### **TCP SEGMENT STRUCTURE:**

```
TCP HEADER (20-60 bytes minimum):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Key fields explained:**

```
SOURCE PORT (16 bits):
• Port number of sending application
• Example: 54321 (ephemeral port)

DESTINATION PORT (16 bits):
• Port number of receiving application
• Example: 80 (HTTP server)

SEQUENCE NUMBER (32 bits):
• Position of first byte in this segment
• Used for ordering and detecting duplicates
• Example: Seq=1000 means "this segment starts at byte 1000"

ACKNOWLEDGMENT NUMBER (32 bits):
• Next expected sequence number
• "I've received everything up to byte X-1"
• Example: Ack=1500 means "send me byte 1500 next"

FLAGS (6 bits):
• URG: Urgent pointer field valid
• ACK: Acknowledgment field valid
• PSH: Push function (deliver immediately)
• RST: Reset connection
• SYN: Synchronize sequence numbers (connection setup)
• FIN: Finish, no more data (connection teardown)

WINDOW SIZE (16 bits):
• Receiver's buffer space available
• Flow control mechanism
• Example: Window=8192 means "I can accept 8KB more data"

CHECKSUM (16 bits):
• Error detection for header + data
• Computed over pseudo-header + TCP segment

URGENT POINTER (16 bits):
• Points to urgent data (rarely used)
```

---

### **TCP CONNECTION ESTABLISHMENT (3-Way Handshake):**

```
════════════════════════════════════════════════════════════
THE THREE-WAY HANDSHAKE
════════════════════════════════════════════════════════════

Client                                              Server
  |                                                    |
  |  Step 1: SYN                                       |
  |  Seq=1000, Ack=0, SYN=1                           |
  |--------------------------------------------------->|
  |  "Hello, let's establish connection"               |
  |  "My starting sequence number is 1000"             |
  |                                                    |
  |                                       Step 2: SYN-ACK|
  |                       Seq=5000, Ack=1001, SYN=1, ACK=1|
  |<---------------------------------------------------|
  |              "OK, my starting sequence is 5000"    |
  |              "I acknowledge your 1000, send 1001 next"|
  |                                                    |
  |  Step 3: ACK                                       |
  |  Seq=1001, Ack=5001, ACK=1                        |
  |--------------------------------------------------->|
  |  "Acknowledged, ready to send data"                |
  |                                                    |
  |         CONNECTION ESTABLISHED                     |
  |                                                    |
  |  Data transfer can now begin...                    |
  |<-------------------------------------------------->|

WHAT GETS ESTABLISHED:
┌─────────────────────────────────────────────────────┐
│ • Sequence numbers synchronized                     │
│ • Buffer sizes exchanged (window size)              │
│ • Maximum segment size (MSS) negotiated             │
│ • Connection state created on both sides            │
└─────────────────────────────────────────────────────┘

WHY 3 STEPS?
• Ensures both sides agree on initial sequence numbers
• Prevents old duplicate SYN from creating false connection
• Allows negotiation of connection parameters
```

**Real packet example:**
```
Step 1 - Client sends SYN:
┌─────────────────────────────────────┐
│ Source Port: 54321                  │
│ Dest Port: 80                       │
│ Seq: 1000                           │
│ Ack: 0                              │
│ Flags: SYN                          │
│ Window: 65535                       │
│ Options: MSS=1460                   │
└─────────────────────────────────────┘

Step 2 - Server responds SYN-ACK:
┌─────────────────────────────────────┐
│ Source Port: 80                     │
│ Dest Port: 54321                    │
│ Seq: 5000                           │
│ Ack: 1001                           │
│ Flags: SYN, ACK                     │
│ Window: 29200                       │
│ Options: MSS=1460                   │
└─────────────────────────────────────┘

Step 3 - Client sends ACK:
┌─────────────────────────────────────┐
│ Source Port: 54321                  │
│ Dest Port: 80                       │
│ Seq: 1001                           │
│ Ack: 5001                           │
│ Flags: ACK                          │
│ Window: 65535                       │
└─────────────────────────────────────┘
```

---

### **TCP DATA TRANSFER:**

**How TCP ensures reliability:**

```
════════════════════════════════════════════════════════════
SEQUENCE NUMBERS & ACKNOWLEDGMENTS
════════════════════════════════════════════════════════════

Sender sends 3 segments:
┌─────────────────────────────────────┐
│ Segment 1: Seq=1000, Len=500        │
│ (bytes 1000-1499)                   │
├─────────────────────────────────────┤
│ Segment 2: Seq=1500, Len=500        │
│ (bytes 1500-1999)                   │
├─────────────────────────────────────┤
│ Segment 3: Seq=2000, Len=500        │
│ (bytes 2000-2499)                   │
└─────────────────────────────────────┘

Receiver acknowledges:
┌─────────────────────────────────────┐
│ ACK=1500  "Got bytes 1000-1499"     │
│ ACK=2000  "Got bytes 1500-1999"     │
│ ACK=2500  "Got bytes 2000-2499"     │
└─────────────────────────────────────┘

CUMULATIVE ACKNOWLEDGMENT:
ACK=2500 means: "I have received ALL bytes up to 2499"
                "Send me byte 2500 next"

════════════════════════════════════════════════════════════
HANDLING PACKET LOSS
════════════════════════════════════════════════════════════

Sender                           Receiver
  |                                 |
  | Seq=1000, Len=500              |
  |------------------------------->|
  |                          ACK=1500|
  |<-------------------------------|
  |                                 |
  | Seq=1500, Len=500              |
  |--------X  LOST                 |
  |                                 |
  | Seq=2000, Len=500              |
  |------------------------------->|
  |                     ACK=1500 (dup)|
  |<-------------------------------| "Still waiting for 1500"
  |                                 |
  | Seq=2500, Len=500              |
  |------------------------------->|
  |                     ACK=1500 (dup)|
  |<-------------------------------| "Still waiting for 1500"
  |                                 |
  | (3 duplicate ACKs received)     |
  | FAST RETRANSMIT TRIGGERED       |
  |                                 |
  | Seq=1500, Len=500 (retransmit) |
  |------------------------------->|
  |                          ACK=3000|
  |<-------------------------------| "Got everything!"

TWO RETRANSMISSION MECHANISMS:

1. TIMEOUT:
   • If no ACK received within timeout period
   • Retransmit the segment
   • Timeout = f(RTT) dynamically adjusted

2. FAST RETRANSMIT:
   • Receive 3 duplicate ACKs
   • Don't wait for timeout
   • Immediately resend missing segment
   • Much faster recovery
```

---

### **TCP FLOW CONTROL (Sliding Window):**

**Purpose:** Prevent sender from overwhelming receiver

```
════════════════════════════════════════════════════════════
RECEIVER BUFFER MANAGEMENT
════════════════════════════════════════════════════════════

Receiver has limited buffer space:
┌─────────────────────────────────────┐
│ Buffer size: 8 KB (8192 bytes)      │
│ Currently used: 2 KB                │
│ Available: 6 KB                     │
└─────────────────────────────────────┘

Receiver advertises window in every ACK:
Window = 6144 (6 KB available)

Sender must not send more than 6 KB unacknowledged data

════════════════════════════════════════════════════════════
SLIDING WINDOW IN ACTION
════════════════════════════════════════════════════════════

Sender's view (Window size = 6 KB):

[Already ACKed] [Sent, awaiting ACK] [Can send] [Future]
                 ←---  Window  -→
                 (max 6 KB)

Example:
Bytes: ...1000  1500  2000  2500  3000  3500  4000...
       ████████░░░░░░░░░░░░▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░
       Already  Sent but    Can send    Future
       ACKed    unACKed     now         (wait)
                (3 KB)      (3 KB)

Receiver sends: ACK=2000, Window=6144
→ Sender can now send bytes 2000-8144

Window slides forward as ACKs arrive:
ACK received for bytes 1500-1999
→ Window slides right by 500 bytes
→ Can now send 500 more bytes

════════════════════════════════════════════════════════════
ZERO WINDOW (Receiver buffer full)
════════════════════════════════════════════════════════════

Receiver: ACK=5000, Window=0
"Stop sending, my buffer is full!"

Sender stops transmission

Receiver processes data, frees buffer space

Receiver: ACK=5000, Window=4096
"I can accept 4 KB now"

Sender resumes transmission
```

---

### **TCP CONGESTION CONTROL:**

**Purpose:** Prevent sender from overwhelming the **network** (not just receiver)

```
╔════════════════════════════════════════════════════════╗
║ CONGESTION CONTROL MECHANISMS                          ║
╠════════════════════════════════════════════════════════╣
║ 1. Slow Start                                          ║
║ 2. Congestion Avoidance                                ║
║ 3. Fast Retransmit                                     ║
║ 4. Fast Recovery                                       ║
╚════════════════════════════════════════════════════════╝

CONGESTION WINDOW (cwnd):
• Sender's estimate of network capacity
• Starts small, grows over time
• Separate from receiver's advertised window
• Actual window = min(cwnd, receiver window)

════════════════════════════════════════════════════════════
PHASE 1: SLOW START (Exponential growth)
════════════════════════════════════════════════════════════

Start: cwnd = 1 MSS (Maximum Segment Size, ~1460 bytes)

RTT 1: Send 1 segment
       ACK received → cwnd = 2 MSS

RTT 2: Send 2 segments
       ACKs received → cwnd = 4 MSS

RTT 3: Send 4 segments
       ACKs received → cwnd = 8 MSS

RTT 4: Send 8 segments
       ACKs received → cwnd = 16 MSS

Growth: DOUBLES every RTT (exponential)
        1 → 2 → 4 → 8 → 16 → 32 → 64...

Graph:
cwnd
 ^
 |                    ╱
 |                  ╱
 |                ╱
 |              ╱
 |            ╱
 |          ╱
 |        ╱
 |      ╱
 |    ╱
 |  ╱
 |╱
 +-----------------> Time (RTTs)

Continue until:
• Reach threshold (ssthresh)
• Packet loss detected

════════════════════════════════════════════════════════════
PHASE 2: CONGESTION AVOIDANCE (Linear growth)
════════════════════════════════════════════════════════════

When cwnd reaches ssthresh:
Switch to ADDITIVE INCREASE

RTT 1: cwnd = 32 MSS
       ACKs received → cwnd = 33 MSS (+1)

RTT 2: cwnd = 33 MSS
       ACKs received → cwnd = 34 MSS (+1)

RTT 3: cwnd = 34 MSS
       ACKs received → cwnd = 35 MSS (+1)

Growth: Increases by 1 MSS per RTT (linear)

Graph:
cwnd
 ^
 |              ╱────────────
 |            ╱    Congestion
 |          ╱      Avoidance
 |        ╱
 |      ╱   Slow
 |    ╱    Start
 |  ╱
 |╱
 +-----------------> Time (RTTs)
       ^
     ssthresh

════════════════════════════════════════════════════════════
PACKET LOSS DETECTED
════════════════════════════════════════════════════════════

TWO SCENARIOS:

1. TIMEOUT (severe congestion):
   • ssthresh = cwnd / 2 (half current window)
   • cwnd = 1 MSS (back to start)
   • Enter Slow Start again
   • MULTIPLICATIVE DECREASE

2. 3 DUPLICATE ACKs (mild congestion):
   • Fast Retransmit: Resend lost packet immediately
   • Fast Recovery: Don't go back to cwnd=1
   • ssthresh = cwnd / 2
   • cwnd = ssthresh + 3
   • Enter Congestion Avoidance
   • Less drastic reduction

════════════════════════════════════════════════════════════
AIMD (Additive Increase Multiplicative Decrease)
════════════════════════════════════════════════════════════

ADDITIVE INCREASE:
• No loss → cwnd += 1 MSS per RTT
• Gradually probe for more bandwidth

MULTIPLICATIVE DECREASE:
• Loss detected → cwnd = cwnd / 2
• Quickly back off to prevent collapse

Graph (Sawtooth pattern):
cwnd
 ^
 |     ╱\      ╱\      ╱\
 |    ╱  \    ╱  \    ╱  \
 |   ╱    \  ╱    \  ╱    \
 |  ╱      \╱      \╱      \
 | ╱
 |╱
 +-------------------------> Time
   ^       ^       ^
  Loss   Loss   Loss
```

**Real-world example:**
```
Download 10 MB file over TCP:

0.0s: Connection established (3-way handshake)
0.1s: Slow Start begins, cwnd=1 (1.4 KB/RTT)
0.2s: cwnd=2 (2.8 KB/RTT)
0.3s: cwnd=4 (5.6 KB/RTT)
0.4s: cwnd=8 (11.2 KB/RTT)
...
1.0s: Reach ssthresh=64, switch to Congestion Avoidance
1.1s: cwnd=65
1.2s: cwnd=66
...
2.5s: Packet loss (3 dup ACKs), cwnd drops to 33
2.6s: Fast Recovery, resume at cwnd=36
...
5.0s: Transfer complete

Throughput graph shows sawtooth pattern
```

---

### **TCP CONNECTION TERMINATION (4-Way Handshake):**

```
════════════════════════════════════════════════════════════
GRACEFUL CONNECTION CLOSE
════════════════════════════════════════════════════════════

Client                                              Server
  |                                                    |
  |  Step 1: FIN                                       |
  |  Seq=3000, FIN=1                                  |
  |--------------------------------------------------->|
  |  "I'm done sending data"                           |
  |                                                    |
  |                                          Step 2: ACK|
  |                              Seq=5000, Ack=3001, ACK=1|
  |<---------------------------------------------------|
  |  "Acknowledged your FIN"                           |
  |                                                    |
  |  (Server can still send data)                      |
  |                                                    |
  |                                          Step 3: FIN|
  |                        Seq=5000, Ack=3001, FIN=1, ACK=1|
  |<---------------------------------------------------|
  |  "I'm also done sending"                           |
  |                                                    |
  |  Step 4: ACK                                       |
  |  Seq=3001, Ack=5001, ACK=1                        |
  |--------------------------------------------------->|
  |  "Acknowledged your FIN"                           |
  |                                                    |
  |  (Wait 2*MSL before fully closing)                 |
  |                                                    |
  |         CONNECTION CLOSED                          |

WHY 4 STEPS?
• TCP is FULL-DUPLEX (two independent data streams)
• Each direction must be closed separately
• Allows one side to finish sending while other is done

TIME_WAIT STATE:
After sending final ACK, client waits 2*MSL (Maximum Segment Lifetime)
• Ensures final ACK arrives
• Prevents old segments from next connection
• Typically 1-4 minutes
```

**Abrupt close (RST):**
```
RESET (RST) - Forceful termination:

Sent when:
• Connection refused (no server listening)
• Invalid segment received
• Application crashes
• Firewall blocks connection

Client                                    Server
  |                                          |
  | SYN                                      |
  |----------------------------------------->|
  |                                     RST  |
  |<-----------------------------------------|
  | (Port not listening)                     |
  
No handshake, immediate closure
No TIME_WAIT state
```

---

## **UDP (User Datagram Protocol)**

### **UDP Characteristics:**

```
╔════════════════════════════════════════════════════════╗
║ UDP FEATURES                                           ║
╠════════════════════════════════════════════════════════╣
║ ✓ CONNECTIONLESS (no handshake)                        ║
║ ✓ FAST (minimal overhead)                              ║
║ ✓ LOW LATENCY (no waiting for ACKs)                    ║
║ ✓ SIMPLE (stateless)                                   ║
║ ✓ MULTICAST/BROADCAST support                          ║
║ ✓ SMALL HEADER (8 bytes vs TCP's 20+)                  ║
║                                                        ║
║ ✗ UNRELIABLE (no delivery guarantee)                   ║
║ ✗ UNORDERED (packets may arrive out of order)          ║
║ ✗ NO FLOW CONTROL                                      ║
║ ✗ NO CONGESTION CONTROL                                ║
║ ✗ NO CONNECTION STATE                                  ║
╚════════════════════════════════════════════════════════╝
```

---

### **UDP DATAGRAM STRUCTURE:**

```
UDP HEADER (8 bytes - very simple!):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

KEY FIELDS:

SOURCE PORT (16 bits):
• Sending application's port
• Optional (can be 0 if no reply needed)

DESTINATION PORT (16 bits):
• Receiving application's port

LENGTH (16 bits):
• Total length of UDP header + data
• Minimum: 8 bytes (header only)
• Maximum: 65,535 bytes

CHECKSUM (16 bits):
• Error detection (optional in IPv4, mandatory in IPv6)
• Can be 0 if not used

That's it! No sequence numbers, no ACKs, no window size.
```

---

### **UDP COMMUNICATION:**

```
════════════════════════════════════════════════════════════
SEND AND FORGET
════════════════════════════════════════════════════════════

Sender                                         Receiver
  |                                               |
  | UDP Datagram #1                              |
  |--------------------------------------------->|
  | (No acknowledgment)                          |
  |                                              |
  | UDP Datagram #2                              |
  |--------------------------------------------->|
  | (No acknowledgment)                          |
  |                                              |
  | UDP Datagram #3                              |
  |---X  LOST IN NETWORK                         |
  |                                              |
  | UDP Datagram #4                              |
  |--------------------------------------------->|
  | (Sender doesn't know #3 was lost)            |
  |                                              |

Receiver gets: #1, #2, #4
Missing: #3
Result: Application must handle missing data

NO RETRANSMISSION:
• Sender doesn't wait for ACKs
• Sender doesn't know if data arrived
• Receiver can't request retransmission
• If reliability needed, application must implement it
```

---

## **5) Key Structures & Components**

### **Transport Layer in Action:**

```
╔════════════════════════════════════════════════════════╗
║ MULTIPLEXING & DEMULTIPLEXING                          ║
╚════════════════════════════════════════════════════════╝

MULTIPLEXING (Sending side):
Multiple applications share one network connection

Application Layer:
┌────────┐ ┌────────┐ ┌────────┐
│Browser │ │Email   │ │SSH     │
│Port    │ │Port    │ │Port    │
│54321   │ │54322   │ │54323   │
└────┬───┘ └────┬───┘ └────┬───┘
     │          │          │
     └──────────┼──────────┘
                ↓
         Transport Layer
         (adds port numbers)
                ↓
         Network Layer
         (single IP address)

DEMULTIPLEXING (Receiving side):
One network connection serves multiple applications

         Network Layer
         (receives packet)
                ↓
         Transport Layer
         (reads port number)
                ↓
     ┌──────────┼──────────┐
     │          │          │
┌────┴───┐ ┌────┴───┐ ┌────┴───┐
│Port 80 │ │Port 25 │ │Port 22 │
│Web     │ │Email   │ │SSH     │
│Server  │ │Server  │ │Server  │
└────────┘ └────────┘ └────────┘

Port number determines which application receives data
```

---

### **Segment vs Packet vs Frame:**

```
DATA ENCAPSULATION AT EACH LAYER:

Application Layer:
┌─────────────────────────────┐
│ HTTP Request                │
│ "GET /index.html HTTP/1.1"  │
└─────────────────────────────┘
              ↓
Transport Layer (adds TCP/UDP header):
┌──────────┬──────────────────┐
│TCP Header│ HTTP Request     │
│(20 bytes)│                  │
└──────────┴──────────────────┘
       SEGMENT
              ↓
Network Layer (adds IP header):
┌──────────┬──────────┬─────────┐
│IP Header │TCP Header│HTTP Req │
│(20 bytes)│(20 bytes)│         │
└──────────┴──────────┴─────────┘
           PACKET
              ↓
Data Link Layer (adds Ethernet header + trailer):
┌────────┬─────┬─────┬────┬────┐
│Eth Hdr │IP   │TCP  │Data│CRC │
│(14 B)  │(20) │(20) │    │(4) │
└────────┴─────┴─────┴────┴────┘
              FRAME
              ↓
Physical Layer:
010110101001... (bits on wire)
```

---

### **TCP State Machine:**

```
TCP CONNECTION STATES:

CLIENT SIDE:                    SERVER SIDE:
                                
CLOSED                          CLOSED
  ↓                               ↓
(active open)                   (passive open)
  ↓                               ↓
SYN_SENT  ------------------>  LISTEN
  ↓                               ↓
  |  <------- SYN-ACK --------  SYN_RCVD
  ↓                               ↓
ESTABLISHED <---- ACK ------> ESTABLISHED
  |                               |
(active close)                (passive close)
  |                               |
FIN_WAIT_1 ----- FIN -------> CLOSE_WAIT
  ↓                               ↓
  |  <------- ACK -----------     |
  ↓                               ↓
FIN_WAIT_2                     (close)
  ↓                               ↓
  |  <------- FIN ------------ LAST_ACK
  ↓                               ↓
TIME_WAIT -------- ACK ------> CLOSED
  ↓
(2*MSL wait)
  ↓
CLOSED

COMMON STATES EXPLAINED:

LISTEN:
• Server waiting for connection requests
• Port is open and ready

SYN_SENT:
• Client sent SYN, waiting for SYN-ACK
• Connection in progress

ESTABLISHED:
• Connection is fully operational
• Data can be transferred

FIN_WAIT_1:
• Sent FIN, waiting for ACK
• Closing connection

TIME_WAIT:
• Waiting for potential retransmissions
• Ensures clean close
• Typically 1-4 minutes
```

---

### **Maximum Segment Size (MSS):**

```
MSS = Maximum amount of DATA in one TCP segment

Calculation:
MTU (Maximum Transmission Unit)
 - IP header (20 bytes)
 - TCP header (20 bytes)
 = MSS

Common MTU values:
• Ethernet: 1500 bytes
  → MSS = 1500 - 20 - 20 = 1460 bytes

• Jumbo frames: 9000 bytes
  → MSS = 9000 - 20 - 20 = 8960 bytes

• PPPoE (DSL): 1492 bytes
  → MSS = 1492 - 20 - 20 = 1452 bytes

MSS NEGOTIATION (during handshake):
Client: "My MSS is 1460"
Server: "My MSS is 1460"
Result: Both use min(1460, 1460) = 1460 bytes

WHY IT MATTERS:
• Larger MSS = fewer segments needed
• Smaller MSS = less risk of fragmentation
• Fragmentation at IP layer is bad (extra overhead, can't retransmit fragments individually)
```

---

## **6) Performance & Tradeoffs**

### **TCP vs UDP Comparison:**

```
╔════════════════════════════════════════════════════════╗
║ WHEN TO USE TCP                                        ║
╠════════════════════════════════════════════════════════╣
║ ✓ Data integrity critical (file transfers, web pages) ║
║ ✓ Order matters (streaming video playback)            ║
║ ✓ Can tolerate latency (downloads, emails)            ║
║ ✓ Need flow/congestion control                        ║
║                                                        ║
║ Examples:                                              ║
║ • HTTP/HTTPS (web browsing)                            ║
║ • FTP/SFTP (file transfer)                             ║
║ • SMTP/IMAP (email)                                    ║
║ • SSH (remote shell)                                   ║
║ • Database connections                                 ║
╚════════════════════════════════════════════════════════╝

╔════════════════════════════════════════════════════════╗
║ WHEN TO USE UDP                                        ║
╠════════════════════════════════════════════════════════╣
║ ✓ Speed more important than reliability               ║
║ ✓ Real-time applications (delay unacceptable)         ║
║ ✓ Can tolerate some data loss                         ║
║ ✓ Broadcast/multicast needed                          ║
║ ✓ Simple request-response (implement own reliability) ║
║                                                        ║
║ Examples:                                              ║
║ • DNS lookups (single query/response)                  ║
║ • Video streaming (live, not buffered)                 ║
║ • Online gaming (position updates)                     ║
║ • VoIP (voice calls)                                   ║
║ • DHCP (address assignment)                            ║
║ • SNMP (network monitoring)                            ║
║ • IoT sensors (frequent small updates)                 ║
╚════════════════════════════════════════════════════════╝
```

---

### **Performance Metrics:**

```
LATENCY COMPARISON:

TCP (first request):
┌──────────────────────────────────────┐
│ 1. SYN       (1 RTT)                 │
│ 2. SYN-ACK                           │
│ 3. ACK + Data                        │
│ 4. Response                          │
│                                      │
│ Total: ~1.5 RTT before data          │
└──────────────────────────────────────┘

UDP:
┌──────────────────────────────────────┐
│ 1. Request                           │
│ 2. Response                          │
│                                      │
│ Total: 1 RTT                         │
└──────────────────────────────────────┘

Savings: 0.5 RTT (~25ms on 50ms RTT network)

OVERHEAD COMPARISON:

TCP:
• Header: 20 bytes minimum (can be 60 with options)
• Handshake: 3 packets before data
• ACKs: ~1 ACK per 2 data segments
• Retransmissions: ~0.1-5% depending on network

UDP:
• Header: 8 bytes (fixed)
• No handshake
• No ACKs
• No retransmissions

For small messages:
• UDP: 8 byte overhead
• TCP: 20 byte header + 3 handshake packets + ACKs
• TCP overhead can exceed data size!
```

---

### **Bandwidth Efficiency:**

```
THROUGHPUT CALCULATION:

TCP Maximum Throughput:
Throughput = (Window Size) / (RTT)

Example 1 - Fast network:
• Window size: 64 KB
• RTT: 10 ms
• Max throughput: 64 KB / 0.01s = 6.4 MB/s = 51 Mbps

Example 2 - Slow network:
• Window size: 64 KB
• RTT: 200 ms
• Max throughput: 64 KB / 0.2s = 320 KB/s = 2.5 Mbps

Even with 1 Gbps link, high RTT limits throughput!

WINDOW SCALING:
Standard window field: 16 bits → max 64 KB
Window scale option: Multiply by 2^n (up to 2^14)
→ Max window: 64 KB × 16384 = 1 GB
Enables high-throughput on high-latency links

BANDWIDTH-DELAY PRODUCT:
BDP = Bandwidth × RTT

Example:
• 100 Mbps link
• 100 ms RTT
• BDP = 100 Mbps × 0.1s = 10 Mb = 1.25 MB

Need window size ≥ 1.25 MB to fully utilize link
```

---

### **Head-of-Line Blocking (TCP Problem):**

```
PROBLEM:

Segment order: 1, 2, 3, 4, 5

Sender ───→ 1 ───→ Receiver ✓
Sender ───→ 2 ──X  Lost
Sender ───→ 3 ───→ Receiver (buffered, can't deliver)
Sender ───→ 4 ───→ Receiver (buffered, can't deliver)
Sender ───→ 5 ───→ Receiver (buffered, can't deliver)

Application receives: 1, [BLOCKED]

Segments 3, 4, 5 arrived but can't be delivered
because TCP guarantees ORDER
Must wait for retransmission of segment 2

IMPACT:
• Latency spike (wait for retransmission)
• All data blocked by one lost packet
• Problem for multiplexed streams

SOLUTIONS:
• HTTP/2: Still suffers (single TCP connection)
• HTTP/3 + QUIC: Independent streams over UDP
• SCTP: Multiple streams in one association
```

---

## **7) Failure Modes**

### **What breaks at Transport Layer:**

```
╔════════════════════════════════════════════════════════╗
║ 1. CONNECTION TIMEOUT                                  ║
╚════════════════════════════════════════════════════════╝

Symptom: "Connection timed out"

Causes:
• Server not responding (down/overloaded)
• Firewall blocking connection
• Wrong IP/port
• SYN packet lost (client retries ~6 times over 127 seconds)

Debug:
$ telnet example.com 80
Trying 93.184.216.34...
telnet: Unable to connect to remote host: Connection timed out

$ nc -zv example.com 80
nc: connect to example.com port 80 (tcp) failed: Connection timed out

╔════════════════════════════════════════════════════════╗
║ 2. CONNECTION REFUSED                                  ║
╚════════════════════════════════════════════════════════╝

Symptom: "Connection refused" (immediate)

Causes:
• No service listening on port
• Service crashed
• Port blocked by firewall (local)

Server sends: RST (reset)

Debug:
$ telnet example.com 12345
Trying 93.184.216.34...
telnet: Unable to connect to remote host: Connection refused

Difference from timeout:
• Refused: Immediate (RST received)
• Timeout: Delayed (no response at all)

╔════════════════════════════════════════════════════════╗
║ 3. PACKET LOSS & RETRANSMISSIONS                       ║
╚════════════════════════════════════════════════════════╝

Symptoms:
• Slow downloads
• High latency
• Stuttering video/audio

TCP automatically handles:
• Detects loss (missing ACKs)
• Retransmits lost segments
• Reduces sending rate (congestion control)

But you see:
• Increased latency (waiting for retransmissions)
• Reduced throughput (smaller congestion window)

Debug:
$ netstat -s | grep retransmit
    45123 segments retransmitted
    
$ ss -ti
...
  cubic rto:204 rtt:3.5/1.5 ato:40 mss:1460 rcvmss:1460
  retrans:0/3 ← 3 retransmissions so far

╔════════════════════════════════════════════════════════╗
║ 4. OUT OF EPHEMERAL PORTS                              ║
╚════════════════════════════════════════════════════════╝

Problem: Too many outgoing connections

Ephemeral port range: 49152-65535 = ~16,000 ports

If you make 16,000 connections:
→ No more ports available
→ New connections fail

Symptoms:
• "Cannot assign requested address"
• "Address already in use"

Common causes:
• Load testing without connection pooling
• TIME_WAIT sockets accumulating
• Connection leaks (not closing)

Debug:
$ ss -tan | grep TIME_WAIT | wc -l
15847  ← Almost out of ports!

$ sysctl net.ipv4.ip_local_port_range
net.ipv4.ip_local_port_range = 32768 65535

Fix:
• Close connections properly
• Use connection pooling
• Reduce TIME_WAIT duration (risky)
• Increase ephemeral port range

╔════════════════════════════════════════════════════════╗
║ 5. SYN FLOOD ATTACK (DoS)                              ║
╚════════════════════════════════════════════════════════╝

Attack: Send thousands of SYN packets with spoofed IPs

Server:
1. Receives SYN
2. Allocates resources (SYN queue entry)
3. Sends SYN-ACK
4. Waits for ACK... (never comes)
5. Repeat thousands of times
6. SYN queue fills up
7. Legitimate connections rejected

Defense: SYN Cookies
• Don't allocate resources on SYN
• Encode state in sequence number
• Verify with returning ACK
• Stateless defense

╔════════════════════════════════════════════════════════╗
║ 6. TCP RESET (RST) INJECTION                           ║
╚════════════════════════════════════════════════════════╝

Attack: Attacker sends forged RST packet

If attacker knows:
• Source IP, Port
• Dest IP, Port  
• Valid sequence number

Can inject RST → connection terminated

Used by:
• Firewalls (Great Firewall of China)
• ISPs (BitTorrent blocking)
• Attackers (man-in-the-middle)

Defense:
• Encryption (TLS) prevents sequence number guessing
• TCP MD5 signature option (rarely used)

╔════════════════════════════════════════════════════════╗
║ 7. UDP - NO ERROR FEEDBACK                             ║
╚════════════════════════════════════════════════════════╝

Problem: UDP provides no reliability

Packet lost → Application doesn't know
Packet duplicated → Application receives twice
Packets reordered → Application gets wrong order

Application must handle:
• Timeouts (no response)
• Sequence numbers (detect loss/duplicates)
• Retransmissions (if needed)

Example - DNS:
1. Send query via UDP
2. Wait 2 seconds
3. No response? Retry
4. Still no response? Try different server
5. All fail? Give up

UDP good for: Cheap to retry (DNS, DHCP)
UDP bad for: Expensive operations (databases)
```

---

## **8) Real-World Usage**

### **1. Web Browsing (HTTP/HTTPS over TCP):**

```
LOADING A WEBPAGE:

User types: https://example.com

STEP 1: DNS Lookup (UDP)
┌─────────────────────────────────────┐
│ DNS Query (UDP port 53)             │
│ "What's the IP of example.com?"     │
│ Response: 93.184.216.34             │
│ Time: ~20ms                         │
└─────────────────────────────────────┘

STEP 2: TCP Connection (3-way handshake)
┌─────────────────────────────────────┐
│ SYN → SYN-ACK → ACK                 │
│ Establish connection to port 443    │
│ Time: ~50ms (1 RTT)                 │
└─────────────────────────────────────┘

STEP 3: TLS Handshake
┌─────────────────────────────────────┐
│ ClientHello → ServerHello           │
│ Certificate exchange                │
│ Key establishment                   │
│ Time: ~100ms (2 RTT)                │
└─────────────────────────────────────┘

STEP 4: HTTP Request (over TCP)
┌─────────────────────────────────────┐
│ GET / HTTP/1.1                      │
│ Sent as TCP segments                │
│ Sequence numbers track bytes        │
│ Time: ~25ms (0.5 RTT)               │
└─────────────────────────────────────┘

STEP 5: HTTP Response
┌─────────────────────────────────────┐
│ HTTP/1.1 200 OK                     │
│ HTML content (may be many segments) │
│ TCP ensures all arrive in order     │
│ Time: ~50ms + transfer time         │
└─────────────────────────────────────┘

STEP 6: Additional Resources
┌─────────────────────────────────────┐
│ Browser parses HTML                 │
│ Finds: 5 CSS, 10 JS, 20 images      │
│ Makes 35 more HTTP requests         │
│ Over SAME TCP connection (HTTP/1.1) │
│ Or multiple parallel (HTTP/2)       │
└─────────────────────────────────────┘

Total time to interactive: ~500ms - 2s
TCP ensures: Every byte arrives correctly
```

---

### **2. Video Streaming (Adaptive over TCP):**

```
NETFLIX / YOUTUBE STREAMING:

Why TCP despite latency sensitivity?
• Buffering hides retransmission delays
• Can't tolerate missing video chunks
• HTTP-based (infrastructure compatibility)

ADAPTIVE BITRATE STREAMING:

Client measures:
┌─────────────────────────────────────┐
│ TCP throughput: 5 Mbps              │
│ Buffer level: 30 seconds            │
│ Congestion window: growing          │
└─────────────────────────────────────┘

Decision: Request 1080p chunks (4 Mbps)

Network degrades:
┌─────────────────────────────────────┐
│ TCP throughput: drops to 2 Mbps     │
│ Packet loss detected (retransmits)  │
│ Buffer level: dropping              │
│ Congestion window: reduced          │
└─────────────────────────────────────┘

Decision: Switch to 720p chunks (2 Mbps)

TCP's congestion control naturally signals quality:
• Fast network → high cwnd → request high quality
• Slow network → low cwnd → request low quality
• Seamless quality adaptation
```

---

### **3. Online Gaming (UDP):**

```
REAL-TIME MULTIPLAYER GAME:

WHY UDP?
• Player position updates every 50ms
• Lost packet = skip one frame (acceptable)
• Retransmitting old position = useless
• Low latency critical

PROTOCOL DESIGN:

Player sends (every 50ms):
┌─────────────────────────────────────┐
│ UDP Datagram:                       │
│ • Sequence: 1234                    │
│ • Position: (100, 250)              │
│ • Velocity: (5, -2)                 │
│ • Timestamp: 10:35:22.450           │
└─────────────────────────────────────┘

Server receives:
Seq 1233 ✓
Seq 1234 ✓
Seq 1235 ✗ LOST
Seq 1236 ✓ (gap detected, interpolate 1235)

RELIABILITY WHERE NEEDED:
• Player movement: UDP (frequent, ok to lose)
• Chat messages: TCP (must arrive)
• Item pickup: UDP + application ACK (critical, retransmit if lost)

Hybrid approach:
• UDP for 99% of traffic (game state)
• TCP for rare events (chat, authentication)
• Application-level reliability for critical events
```

---

### **4. DNS Lookup (UDP, falls back to TCP):**

```
DOMAIN NAME RESOLUTION:

PRIMARY: UDP (Port 53)
┌─────────────────────────────────────┐
│ Query: "www.example.com" A record   │
│ Packet size: ~50 bytes              │
│ Fits in single UDP datagram         │
│ Response: <512 bytes (typically)    │
│ Time: ~20ms (no handshake overhead) │
└─────────────────────────────────────┘

Benefits:
• Fast (no TCP handshake)
• Stateless (no connection state)
• Cheap for server (no state per query)

FALLBACK: TCP (Port 53)
When:
• Response > 512 bytes (DNSSEC, many records)
• TC (truncated) bit set in UDP response
• Zone transfers (AXFR)

Process:
1. Try UDP
2. Response has TC bit → response truncated
3. Retry with TCP
4. Get full response

Drawback:
• TCP handshake adds latency (~50ms extra)
• More server resources (connection state)

Modern: EDNS0 allows UDP responses > 512 bytes (up to ~4 KB)
```

---

### **5. File Transfer (TCP - FTP/SFTP):**

```
FTP (File Transfer Protocol):

DUAL CONNECTION MODEL:

Control Connection (TCP port 21):
┌─────────────────────────────────────┐
│ Persistent TCP connection           │
│ Commands: USER, PASS, LIST, RETR    │
│ Responses: 200 OK, 550 Error        │
│ Stays open entire session           │
└─────────────────────────────────────┘

Data Connection (TCP port 20 or random):
┌─────────────────────────────────────┐
│ Separate TCP connection             │
│ Created for each file transfer      │
│ Closes after transfer complete      │
│ Carries actual file data            │
└─────────────────────────────────────┘

DOWNLOAD 1 GB FILE:

1. Control: "RETR largefile.zip"
2. Server opens data connection
3. TCP segments:
   • File broken into ~700,000 segments (1460 bytes each)
   • Each segment has sequence number
   • Client ACKs received segments
   • Lost segments retransmitted
4. Transfer complete: Data connection closes
5. Control: "226 Transfer complete"

TCP ensures:
✓ Every byte arrives
✓ Correct order
✓ Automatic retransmission
✓ Flow control (don't overflow receiver)
✓ Congestion control (don't flood network)

PERFORMANCE:
Initial: Slow Start (cwnd grows exponentially)
Middle: Full speed (cwnd at maximum)
End: Potential slowdown if packet loss

Transfer time:
• Perfect network: ~80 seconds (100 Mbps)
• With 1% loss: ~120 seconds (retransmissions)
• With 5% loss: ~300 seconds (congestion control backs off)
```

---

### **6. VoIP (Voice over IP - UDP):**

```
PHONE CALL OVER INTERNET:

PROTOCOL: RTP (Real-time Transport Protocol) over UDP

Voice encoding:
• Sample voice at 8 kHz
• Encode with codec (G.711, Opus)
• Create RTP packets every 20ms
• Each packet: ~160 bytes

Transmission (UDP):
┌─────────────────────────────────────┐
│ RTP packet every 20ms               │
│ Sequence number: track order        │
│ Timestamp: playback timing          │
│ Payload: encoded voice              │
└─────────────────────────────────────┘

LOSS TOLERANCE:

Perfect delivery (0% loss):
"Hello, how are you today?"

1% packet loss:
"Hello, h*w are you today?"  (* = minor glitch)

5% packet loss:
"H*llo, ** a*e yo* to**y?"  (noticeable degradation)

10% packet loss:
"** **o, * * ** **day?"  (barely comprehensible)

WHY NOT TCP?

TCP with retransmission:
Time 0ms: Packet lost
Time 20ms: Next packet (waiting for retransmit)
Time 40ms: Next packet
Time 100ms: Retransmission arrives

By time retransmission arrives (100ms):
• Already played next 5 packets
• Old packet is USELESS
• Creates jarring audio gaps

UDP approach:
• Lost packet = tiny glitch
• Keep playing newer packets
• Smooths over gaps with interpolation
• Overall better experience

JITTER BUFFER:
Buffer 40-80ms of packets
• Absorbs network timing variations
• Reorders out-of-sequence packets
• Plays back at constant rate
• Discards packets that arrive too late
```

---

## **9) Comparison Section**

### **TCP vs UDP Detailed:**

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (handshake) | Connectionless (no handshake) |
| **Reliability** | Guaranteed delivery (ACK + retransmit) | Best effort (no guarantees) |
| **Ordering** | Strict order (sequence numbers) | No ordering (may arrive out of order) |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Header size** | 20-60 bytes | 8 bytes |
| **Flow control** | Yes (sliding window) | No |
| **Congestion control** | Yes (slow start, AIMD) | No |
| **Error detection** | Yes (checksum) | Optional (checksum can be 0) |
| **State** | Stateful (connection tracking) | Stateless |
| **Broadcast/Multicast** | No | Yes |
| **Usage** | ~95% of internet traffic | ~5% of internet traffic |
| **Best for** | Reliability critical (web, email, files) | Speed critical (gaming, streaming, DNS) |

---

### **Transport Protocol Comparison:**

| Protocol | Type | Reliability | Use Case | Status |
|----------|------|-------------|----------|--------|
| **TCP** | Stream | Reliable, ordered | General purpose | Standard |
| **UDP** | Datagram | Unreliable | Real-time, low overhead | Standard |
| **SCTP** | Stream | Reliable, multi-stream | Telephony signaling | Niche |
| **DCCP** | Datagram | Unreliable + congestion control | Streaming with congestion awareness | Rare |
| **QUIC** | Stream | Reliable, multiplexed | HTTP/3, modern web | Growing |

---

### **QUIC (Modern TCP Alternative):**

```
QUIC = Quick UDP Internet Connections

Built on UDP, adds:
✓ Reliability (like TCP)
✓ Encryption (built-in TLS 1.3)
✓ Multiplexing (multiple streams, no head-of-line blocking)
✓ 0-RTT connection resumption
✓ Connection migration (survive IP changes)

Comparison:

TCP + TLS:
┌──────────────────────────────────────┐
│ RTT 1: TCP SYN handshake             │
│ RTT 2: TLS handshake                 │
│ RTT 3: HTTP request                  │
│ Total: 3 RTT before data             │
└──────────────────────────────────────┘

QUIC:
┌──────────────────────────────────────┐
│ RTT 1: Connection + TLS + Request    │
│ (or 0-RTT on resumption)             │
│ Total: 1 RTT (or 0 RTT)              │
└──────────────────────────────────────┘

Adoption:
• HTTP/3 uses QUIC
• Google services
• Facebook, Cloudflare
• ~25% of internet traffic (2024)
```

---

## **10) Packet Walkthrough**

**Scenario:** User downloads a 10 KB file over TCP

```
════════════════════════════════════════════════════════════
SETUP: TCP CONNECTION ESTABLISHMENT
════════════════════════════════════════════════════════════

Client: 192.168.1.100:54321
Server: 93.184.216.34:80

PACKET 1 - SYN:
┌─────────────────────────────────────┐
│ Source: 192.168.1.100:54321         │
│ Dest: 93.184.216.34:80              │
│ Seq: 1000                           │
│ Ack: 0                              │
│ Flags: SYN                          │
│ Window: 65535                       │
│ Options: MSS=1460                   │
└─────────────────────────────────────┘

PACKET 2 - SYN-ACK:
┌─────────────────────────────────────┐
│ Source: 93.184.216.34:80            │
│ Dest: 192.168.1.100:54321           │
│ Seq: 5000                           │
│ Ack: 1001                           │
│ Flags: SYN, ACK                     │
│ Window: 29200                       │
│ Options: MSS=1460                   │
└─────────────────────────────────────┘

PACKET 3 - ACK:
┌─────────────────────────────────────┐
│ Source: 192.168.1.100:54321         │
│ Dest: 93.184.216.34:80              │
│ Seq: 1001                           │
│ Ack: 5001                           │
│ Flags: ACK                          │
│ Window: 65535                       │
└─────────────────────────────────────┘

CONNECTION ESTABLISHED
Both sides know:
• MSS = 1460 bytes
• Initial sequence numbers
• Window sizes

════════════════════════════════════════════════════════════
DATA TRANSFER: HTTP GET REQUEST
════════════════════════════════════════════════════════════

PACKET 4 - Client sends HTTP request:
┌─────────────────────────────────────┐
│ TCP Header:                         │
│   Source: 54321, Dest: 80           │
│   Seq: 1001                         │
│   Ack: 5001                         │
│   Flags: PSH, ACK                   │
│   Len: 100 bytes                    │
├─────────────────────────────────────┤
│ Data:                               │
│ GET /file.txt HTTP/1.1              │
│ Host: example.com                   │
│ ...                                 │
│ (100 bytes total)                   │
└─────────────────────────────────────┘

PACKET 5 - Server ACKs request:
┌─────────────────────────────────────┐
│ Source: 80, Dest: 54321             │
│ Seq: 5001                           │
│ Ack: 1101  (1001 + 100)             │
│ Flags: ACK                          │
│ "Got your request"                  │
└─────────────────────────────────────┘

════════════════════════════════════════════════════════════
DATA TRANSFER: FILE DOWNLOAD (10 KB)
════════════════════════════════════════════════════════════

Server breaks 10 KB into segments:
10,240 bytes ÷ 1460 bytes/segment = 8 segments

PACKET 6 - Server sends segment 1:
┌─────────────────────────────────────┐
│ TCP Header:                         │
│   Seq: 5001                         │
│   Ack: 1101                         │
│   Flags: ACK                        │
│   Len: 1460 bytes                   │
├─────────────────────────────────────┤
│ Data: [bytes 5001-6460]             │
│ (first 1460 bytes of file)          │
└─────────────────────────────────────┘

PACKET 7 - Server sends segment 2:
┌─────────────────────────────────────┐
│ Seq: 6461                           │
│ Len: 1460 bytes                     │
│ Data: [bytes 6461-7920]             │
└─────────────────────────────────────┘

PACKET 8 - Server sends segment 3:
┌─────────────────────────────────────┐
│ Seq: 7921                           │
│ Len: 1460 bytes                     │
│ Data: [bytes 7921-9380]             │
└─────────────────────────────────────┘

PACKET 9 - Client ACKs segments 1-3:
┌─────────────────────────────────────┐
│ Source: 54321, Dest: 80             │
│ Seq: 1101                           │
│ Ack: 9381  (cumulative)             │
│ Flags: ACK                          │
│ "Got bytes 5001-9380"               │
└─────────────────────────────────────┘

PACKET 10 - Server sends segment 4:
┌─────────────────────────────────────┐
│ Seq: 9381                           │
│ Len: 1460 bytes                     │
│ Data: [bytes 9381-10840]            │
└─────────────────────────────────────┘

PACKET 11 - Server sends segment 5:
┌─────────────────────────────────────┐
│ Seq: 10841                          │
│ Len: 1460 bytes                     │
│ Data: [bytes 10841-12300]           │
└─────────────────────────────────────┘

═══ PACKET LOSS SCENARIO ═══

PACKET 12 - Server sends segment 6:
┌─────────────────────────────────────┐
│ Seq: 12301                          │
│ Len: 1460 bytes                     │
└─────────────────────────────────────┘
       ↓
    *** LOST IN NETWORK ***

PACKET 13 - Server sends segment 7:
┌─────────────────────────────────────┐
│ Seq: 13761                          │
│ Len: 1460 bytes                     │
│ Arrives at client                   │
└─────────────────────────────────────┘

Client sees GAP (missing 12301-13760)

PACKET 14 - Client sends duplicate ACK:
┌─────────────────────────────────────┐
│ Ack: 12301  (still waiting)         │
│ Flags: ACK                          │
│ "Still need byte 12301"             │
└─────────────────────────────────────┘

PACKET 15 - Server sends segment 8:
┌─────────────────────────────────────┐
│ Seq: 15221                          │
│ Len: 19 bytes (last segment)        │
└─────────────────────────────────────┘

PACKET 16 - Client sends duplicate ACK:
┌─────────────────────────────────────┐
│ Ack: 12301  (dup #2)                │
│ "STILL need byte 12301"             │
└─────────────────────────────────────┘

PACKET 17 - Client sends duplicate ACK:
┌─────────────────────────────────────┐
│ Ack: 12301  (dup #3)                │
│ "STILL need byte 12301!"            │
└─────────────────────────────────────┘

SERVER RECEIVES 3 DUPLICATE ACKs
→ FAST RETRANSMIT TRIGGERED

PACKET 18 - Server retransmits segment 6:
┌─────────────────────────────────────┐
│ Seq: 12301                          │
│ Len: 1460 bytes                     │
│ Data: [bytes 12301-13760]           │
│ (RETRANSMISSION)                    │
└─────────────────────────────────────┘

PACKET 19 - Client ACKs everything:
┌─────────────────────────────────────┐
│ Ack: 15240  (5001 + 10,239)         │
│ Flags: ACK                          │
│ "Got complete file!"                │
└─────────────────────────────────────┘

════════════════════════════════════════════════════════════
CONNECTION TEARDOWN: 4-WAY HANDSHAKE
════════════════════════════════════════════════════════════

PACKET 20 - Client initiates close:
┌─────────────────────────────────────┐
│ Seq: 1101                           │
│ Ack: 15240                          │
│ Flags: FIN, ACK                     │
│ "I'm done sending"                  │
└─────────────────────────────────────┘

PACKET 21 - Server ACKs FIN:
┌─────────────────────────────────────┐
│ Seq: 15240                          │
│ Ack: 1102  (1101 + 1 for FIN)       │
│ Flags: ACK                          │
│ "Acknowledged your FIN"             │
└─────────────────────────────────────┘

PACKET 22 - Server sends FIN:
┌─────────────────────────────────────┐
│ Seq: 15240                          │
│ Ack: 1102                           │
│ Flags: FIN, ACK                     │
│ "I'm also done"                     │
└─────────────────────────────────────┘

PACKET 23 - Client ACKs server FIN:
┌─────────────────────────────────────┐
│ Seq: 1102                           │
│ Ack: 15241  (15240 + 1 for FIN)     │
│ Flags: ACK                          │
│ "Acknowledged"                      │
└─────────────────────────────────────┘

CONNECTION CLOSED

════════════════════════════════════════════════════════════
SUMMARY
════════════════════════════════════════════════════════════

Total packets: 23
• 3 packets: Connection setup (SYN, SYN-ACK, ACK)
• 1 packet: HTTP request
• 1 packet: ACK for request
• 8 packets: File data (segments)
• 4 packets: ACKs for data
• 1 packet: Retransmission (segment 6)
• 1 packet: Final ACK for retransmitted data
• 4 packets: Connection teardown (FIN, ACK, FIN, ACK)

Key observations:
✓ Sequence numbers ensure ordering
✓ ACKs confirm delivery
✓ Lost packet detected via duplicate ACKs
✓ Fast retransmit (didn't wait for timeout)
✓ Cumulative ACKs acknowledge all previous data
✓ Connection properly closed (graceful shutdown)
```

---

## **11) Common Interview / Exam Traps**

### **Misconception 1: "Port numbers are Layer 3"**
**Wrong:** IP addresses and ports are both network addresses  
**Right:**
- **IP address (Layer 3):** Identifies the host/device
- **Port number (Layer 4):** Identifies the application/process
- Socket = IP + Port (spans L3 and L4)

### **Misconception 2: "TCP guarantees delivery"**
**Wrong:** TCP always delivers packets  
**Right:** TCP guarantees delivery **OR** reports failure. If network is completely down, TCP will eventually timeout and return error to application. "Guarantee" means "best effort with confirmation."

### **Misconception 3: "UDP is faster than TCP"**
**Wrong:** UDP protocol is inherently faster  
**Right:** UDP has **less overhead** (smaller header, no handshake, no ACKs), which can result in lower latency. But for bulk data transfer on a good network, TCP can achieve similar throughput. UDP is faster for **small messages** and **real-time data** where retransmission would add unacceptable delay.

### **Misconception 4: "Sequence numbers count packets"**
**Wrong:** TCP sequence number increments by 1 for each packet  
**Right:** Sequence numbers count **BYTES**, not packets.
- Packet 1: Seq=1000, Len=500 → bytes 1000-1499
- Packet 2: Seq=1500, Len=500 → bytes 1500-1999
- Packet 3: Seq=2000, Len=500 → bytes 2000-2499

### **Misconception 5: "Window size is fixed"**
**Wrong:** TCP window size never changes  
**Right:** Window size is **dynamic**, advertised by receiver in every ACK. Changes based on buffer availability. Can also use window scaling for high-bandwidth networks.

### **Misconception 6: "FIN terminates connection immediately"**
**Wrong:** FIN packet closes the connection  
**Right:** FIN closes **one direction** only (half-close). TCP is full-duplex, so each side must send FIN. After sending FIN, that side can't send more data but can still receive.

### **Misconception 7: "Three-way handshake sends data"**
**Wrong:** Data can be sent during handshake  
**Right:** In traditional TCP, data is sent **after** handshake completes (in the ACK or subsequent packets). TCP Fast Open (TFO) allows data in SYN, but this is a newer extension, not standard behavior.

### **Misconception 8: "UDP has no error detection"**
**Wrong:** UDP provides no reliability features at all  
**Right:** UDP includes a **checksum** for error detection (optional in IPv4, mandatory in IPv6). It detects corrupted packets but doesn't correct them or request retransmission.

---

### **Frequently Asked:**

**Q: What happens if both sides send FIN simultaneously?**  
A: **Simultaneous close.** Both sides go through FIN_WAIT_1 → CLOSING → TIME_WAIT → CLOSED. Results in 4 packets total (2 FINs, 2 ACKs), same as normal close.

**Q: Why does TCP use a three-way handshake instead of two-way?**  
A: **Prevent old duplicate SYNs from creating false connections.** With 2-way, an old delayed SYN could arrive and server would think it's a new connection. Three-way ensures both sides agree on current connection.

**Q: Can TCP send data before handshake completes?**  
A: **Not in standard TCP.** TCP Fast Open (RFC 7413) allows data in SYN packet for repeat connections, but this is an extension. Standard TCP: handshake first, then data.

**Q: What's the maximum number of TCP connections between two hosts?**  
A: **Theoretical:** 2^32 × 2^16 × 2^16 (source IP × source port × dest port) = enormous  
**Practical:** Limited by:
- Ephemeral ports (~16,000-64,000)
- File descriptors (often ~1,000-100,000 per process)
- Memory (each connection uses ~4 KB)
- Realistic limit: ~65,000 connections from one client IP to one server IP:port

**Q: Why does DNS use UDP instead of TCP?**  
A: **Speed.** DNS queries are typically small (<512 bytes), fit in one packet, and cheap to retry if lost. TCP handshake would double the latency. DNS falls back to TCP for large responses or zone transfers.

**Q: What's the difference between flow control and congestion control?**  
A:
- **Flow control:** Prevent overwhelming the **receiver** (receiver's buffer full)
- **Congestion control:** Prevent overwhelming the **network** (routers dropping packets)
- Flow control uses receiver's advertised window
- Congestion control uses sender's congestion window (cwnd)

**Q: How does TCP detect network congestion?**  
A: **Packet loss.** Assumes packet loss indicates congestion (routers dropping packets because queues are full). Reduces sending rate in response. This isn't always accurate (wireless networks have loss without congestion), but it's the primary signal.

---

## **12) Retrieval Prompts**

### **Core Concepts:**
1. What are the main responsibilities of the Transport Layer?
2. Explain the difference between TCP and UDP in one sentence each
3. What is a port number and why do we need it?
4. What is a socket? How is it different from a port?

### **TCP Deep Dive:**
5. Walk through the TCP three-way handshake step-by-step
6. Explain how TCP sequence numbers and acknowledgments work
7. What happens when a TCP packet is lost? (Two detection mechanisms)
8. How does TCP flow control work? (Sliding window)
9. Explain TCP congestion control: Slow Start, Congestion Avoidance, Fast Retransmit
10. Why does TCP connection close require 4 packets instead of 2?

### **UDP Deep Dive:**
11. When should you use UDP instead of TCP?
12. How does a UDP-based application achieve reliability if needed?
13. Why is DNS primarily UDP but sometimes uses TCP?

### **Performance:**
14. Why is TCP slower than UDP for small messages?
15. What is head-of-line blocking in TCP?
16. Calculate TCP maximum throughput given window size and RTT
17. Why does packet loss cause TCP throughput to drop?

### **Troubleshooting:**
18. "Connection timeout" vs "Connection refused" — what's the difference?
19. How would you diagnose slow download speeds? What TCP metrics matter?
20. What causes "too many open files" error?
21. How do you detect if you're experiencing packet loss?

### **Security:**
22. What is a SYN flood attack and how does it work?
23. How do SYN cookies defend against SYN floods?
24. What is TCP RST injection?

### **Real-World:**
25. Trace all Transport Layer operations when loading a webpage
26. Why does Netflix use TCP for video streaming instead of UDP?
27. How do online games use UDP effectively despite unreliability?
28. Explain QUIC and how it improves on TCP

### **Advanced:**
29. What is the bandwidth-delay product and why does it matter?
30. How does TCP handle out-of-order packet arrival?
31. What happens during the TIME_WAIT state and why is it needed?
32. How can you have multiple connections to the same IP:port pair?

---

## **13) TL;DR Compression**

**5-bullet summary:**

1. **Transport Layer = End-to-end communication between processes** — Uses port numbers (0-65535) to identify applications on hosts; delivers data to correct application; Layer 3 (IP) gets to the right computer, Layer 4 gets to the right program

2. **TCP = Reliable, connection-oriented protocol:**
   - Three-way handshake (SYN, SYN-ACK, ACK) establishes connection
   - Sequence numbers + ACKs ensure all data arrives in order
   - Flow control (sliding window) prevents overwhelming receiver
   - Congestion control (slow start, AIMD) prevents overwhelming network
   - Used for: web (HTTP), email (SMTP), file transfer (FTP), SSH

3. **UDP = Unreliable, connectionless protocol:**
   - No handshake, no ACKs, no guarantees (send and forget)
   - 8-byte header vs TCP's 20+ bytes (minimal overhead)
   - Fast, low latency, stateless
   - Used for: DNS lookups, video streaming, online gaming, VoIP
   - Application must implement reliability if needed

4. **Performance tradeoffs:**
   - **TCP:** Overhead from handshake + ACKs + retransmissions; guarantees delivery but adds latency; ~95% of internet traffic
   - **UDP:** Fast for small messages and real-time apps; no retransmission delays; packet loss = data loss; ~5% of traffic
   - **Modern:** QUIC (HTTP/3) combines TCP-like reliability with UDP-like speed

5. **Key mechanisms:**
   - **Reliability:** Sequence numbers, ACKs, retransmissions (timeout or 3 dup ACKs trigger fast retransmit)
   - **Flow control:** Receiver advertises window size, sender limits unACKed data
   - **Congestion control:** cwnd grows (slow start → congestion avoidance), shrinks on loss (multiplicative decrease)
   - **Connection management:** 3-way handshake to open, 4-way handshake to close gracefully

**One-sentence essence:**
The Transport Layer bridges the gap between unreliable network packets and reliable application communication by providing process-to-process delivery via ports, with TCP adding reliability through handshakes, sequence numbers, acknowledgments, and retransmissions for applications that need guarantees, while UDP offers a lightweight alternative for speed-critical applications willing to handle their own reliability or tolerate loss.