# OSI Model: Session Layer (Layer 5)

## Table of Contents
- [Overview](#overview)
- [Core Functions](#core-functions)
- [Session Management](#session-management)
- [Dialog Control](#dialog-control)
- [Synchronization](#synchronization)
- [Protocols & Standards](#protocols--standards)
- [Session Establishment Process](#session-establishment-process)
- [Authentication & Authorization](#authentication--authorization)
- [Practical Examples](#practical-examples)
- [Code Examples](#code-examples)
- [Troubleshooting Guide](#troubleshooting-guide)

---

## Overview

```
┌─────────────────────────────────────────────────────────┐
│            SESSION LAYER (Layer 5)                      │
│                                                         │
│  Primary Functions:                                     │
│  • Session Establishment, Maintenance, Termination      │
│  • Dialog Control (Half-duplex/Full-duplex)            │
│  • Synchronization & Checkpointing                      │
│                                                         │
│  Role: Connection Manager and Dialog Coordinator       │
└─────────────────────────────────────────────────────────┘
```

**Key Purpose:** Manages communication sessions between applications, controlling the dialogue and ensuring orderly data exchange.

---

## Core Functions

```
┌──────────────────────────────────────────────────────────────┐
│                     SESSION LAYER FUNCTIONS                  │
│                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────┐  │
│  │   SESSION       │  │     DIALOG      │  │   SYNC &   │  │
│  │  MANAGEMENT     │  │    CONTROL      │  │ CHECKPOINT │  │
│  │                 │  │                 │  │            │  │
│  │ • Establish     │  │ • Half-duplex   │  │ • Recovery │  │
│  │ • Maintain      │  │ • Full-duplex   │  │ • Restart  │  │
│  │ • Terminate     │  │ • Simplex       │  │ • Tokens   │  │
│  └─────────────────┘  └─────────────────┘  └────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Function Details Table

| Function            | Purpose                           | Examples                        |
|---------------------|-----------------------------------|---------------------------------|
| Session Setup       | Establish connection              | Login, authentication           |
| Session Maintenance | Keep connection alive             | Keepalive messages, heartbeats  |
| Session Termination | Gracefully close connection       | Logout, disconnect              |
| Dialog Control      | Manage communication flow         | Turn-taking, simultaneous comm  |
| Synchronization     | Coordinate data exchange          | Checkpoints, recovery points    |

---

## Session Management

### Session Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│                   SESSION LIFECYCLE                          │
└──────────────────────────────────────────────────────────────┘

1. SESSION ESTABLISHMENT
   ┌─────────────────────────────────────────────┐
   │  Client                         Server      │
   │    │                               │        │
   │    │──── Session Request ─────────→│        │
   │    │     (Credentials, Parameters) │        │
   │    │                               │        │
   │    │←─── Session Accept ───────────│        │
   │    │     (Session ID, Config)      │        │
   │    │                               │        │
   │    ✓ SESSION ESTABLISHED          ✓        │
   └─────────────────────────────────────────────┘

2. SESSION MAINTENANCE (Active Phase)
   ┌─────────────────────────────────────────────┐
   │  Client                         Server      │
   │    │                               │        │
   │    │═══════ Data Exchange ═════════│        │
   │    │                               │        │
   │    │──── Keepalive ───────────────→│        │
   │    │←─── ACK ──────────────────────│        │
   │    │                               │        │
   │    │═══════ More Data ═════════════│        │
   │    │                               │        │
   │    │──── Checkpoint ──────────────→│        │
   │    │←─── Checkpoint ACK ───────────│        │
   └─────────────────────────────────────────────┘

3. SESSION TERMINATION
   ┌─────────────────────────────────────────────┐
   │  Client                         Server      │
   │    │                               │        │
   │    │──── Close Request ───────────→│        │
   │    │                               │        │
   │    │←─── Close Accept ─────────────│        │
   │    │                               │        │
   │    ✓ SESSION CLOSED               ✓        │
   └─────────────────────────────────────────────┘
```

### Session States

```
STATE TRANSITION DIAGRAM:

    ┌──────────┐
    │  IDLE    │  (No session exists)
    └─────┬────┘
          │
          │ Connect Request
          ↓
    ┌──────────┐
    │CONNECTING│  (Negotiating session parameters)
    └─────┬────┘
          │
          │ Session Established
          ↓
    ┌──────────┐
    │  ACTIVE  │  (Data transfer, keepalive)
    └─────┬────┘
          │
          │ Close Request
          ↓
    ┌──────────┐
    │ CLOSING  │  (Graceful shutdown)
    └─────┬────┘
          │
          │ Closed
          ↓
    ┌──────────┐
    │  CLOSED  │  (Session terminated)
    └──────────┘
          │
          │ Timeout/Cleanup
          ↓
    ┌──────────┐
    │  IDLE    │
    └──────────┘
```

### Session Parameters Table

| Parameter         | Description                      | Example Values           |
|-------------------|----------------------------------|--------------------------|
| Session ID        | Unique identifier for session    | UUID, Token              |
| Timeout           | Inactivity timeout period        | 30 min, 1 hour           |
| Authentication    | User credentials verification    | Username/Password, OAuth |
| Communication Mode| Duplex setting                   | Half, Full, Simplex      |
| Checkpoint Interval| Frequency of sync points        | Every 1MB, Every 5 min   |
| Max Session Time  | Maximum session duration         | 8 hours, 24 hours        |

---

## Dialog Control

### Communication Modes

```
┌──────────────────────────────────────────────────────────────┐
│                   COMMUNICATION MODES                        │
└──────────────────────────────────────────────────────────────┘

1. SIMPLEX (One-way communication)
   ┌─────────────────────────────────────────┐
   │  Sender ═════════════════════→ Receiver │
   │                                         │
   │  Example: TV broadcast, Radio           │
   └─────────────────────────────────────────┘

2. HALF-DUPLEX (Two-way, one at a time)
   ┌─────────────────────────────────────────┐
   │  Device A ══════════════════→ Device B  │
   │                                         │
   │  [Turn-taking required]                 │
   │                                         │
   │  Device A ←══════════════════ Device B  │
   │                                         │
   │  Example: Walkie-talkie, CB radio       │
   └─────────────────────────────────────────┘

3. FULL-DUPLEX (Two-way simultaneous)
   ┌─────────────────────────────────────────┐
   │  Device A ══════════════════→ Device B  │
   │           ←══════════════════           │
   │                                         │
   │  Example: Telephone, Video call         │
   └─────────────────────────────────────────┘
```

### Token Management

```
┌──────────────────────────────────────────────────────────────┐
│                    TOKEN-BASED CONTROL                       │
└──────────────────────────────────────────────────────────────┘

SCENARIO: Half-Duplex Communication with Data Token

Time →
  
Client A                    Token Holder                Server B
   │                                                        │
   │──────── Request Token ──────────────────────────────→ │
   │                                                        │
   │ ←─────── Token Granted ────────────────────────────── │
   │         [Client A now has token]                      │
   │                                                        │
   │══════════ Send Data ══════════════════════════════→   │
   │══════════ Send Data ══════════════════════════════→   │
   │                                                        │
   │──────── Release Token ──────────────────────────────→ │
   │         [Token released]                              │
   │                                                        │
   │ ←─────── Token to B ────────────────────────────────  │
   │                        [Server B now has token]       │
   │                                                        │
   │ ←══════════ Receive Data ═══════════════════════════  │
   │ ←══════════ Receive Data ═══════════════════════════  │
   │                                                        │
   │ ←─────── Token Released ────────────────────────────  │
```

### Dialog Control Comparison

| Mode        | Direction      | Simultaneous | Token Required | Example Use Case    |
|-------------|----------------|--------------|----------------|---------------------|
| Simplex     | One-way        | N/A          | No             | Broadcasting        |
| Half-Duplex | Two-way        | No           | Yes            | File transfer       |
| Full-Duplex | Two-way        | Yes          | No             | Video conferencing  |

---

## Synchronization

### Checkpoint Mechanism

```
┌──────────────────────────────────────────────────────────────┐
│              SYNCHRONIZATION & CHECKPOINTS                   │
└──────────────────────────────────────────────────────────────┘

FILE TRANSFER WITH CHECKPOINTS:

Sender                                              Receiver
  │                                                     │
  │═══════ Data Block 1 (1MB) ════════════════════════→│
  │                                                     │
  │──────── Checkpoint 1 ─────────────────────────────→│
  │←──────── ACK CP1 ──────────────────────────────────│
  │         [Checkpoint 1 confirmed]                   │
  │                                                     │
  │═══════ Data Block 2 (1MB) ════════════════════════→│
  │                                                     │
  │──────── Checkpoint 2 ─────────────────────────────→│
  │←──────── ACK CP2 ──────────────────────────────────│
  │         [Checkpoint 2 confirmed]                   │
  │                                                     │
  │═══════ Data Block 3 (1MB) ════════════════════════→│
  │                                                     │
  │         ⚠️  CONNECTION LOST! ⚠️                     │
  │                                                     │
  │         [Reconnect and Resume]                     │
  │                                                     │
  │──────── Resume from CP2 ──────────────────────────→│
  │                                                     │
  │═══════ Resend Block 3 (1MB) ══════════════════════→│
  │                                                     │
  │──────── Checkpoint 3 ─────────────────────────────→│
  │←──────── ACK CP3 ──────────────────────────────────│
  │         ✓ Transfer continues from last checkpoint  │
```

### Major and Minor Synchronization

```
┌──────────────────────────────────────────────────────────────┐
│         MAJOR vs MINOR SYNCHRONIZATION POINTS                │
└──────────────────────────────────────────────────────────────┘

MAJOR SYNC POINT:
  • Bilateral agreement (both parties must acknowledge)
  • Cannot be rolled back unilaterally
  • Used for critical state changes
  • Example: End of transaction, file transfer complete

  Sender                              Receiver
    │                                     │
    │──── Major Sync Request ────────────→│
    │                                     │
    │←─── Major Sync Confirm ─────────────│
    │     [Both parties committed]        │

MINOR SYNC POINT:
  • Unilateral action
  • Can be rolled back if needed
  • Used for intermediate checkpoints
  • Example: Progress markers during transfer

  Sender                              Receiver
    │                                     │
    │──── Minor Sync Mark ───────────────→│
    │     [Sender notes position]         │
    │                                     │
    │     (Can continue or rollback)      │
```

### Synchronization Benefits

| Benefit          | Description                                | Impact                    |
|------------------|--------------------------------------------|---------------------------|
| Fault Recovery   | Resume from last checkpoint                | Saves time and bandwidth  |
| Progress Tracking| Monitor transfer status                    | Better user experience    |
| Data Integrity   | Verify data received correctly             | Ensures accuracy          |
| Resource Mgmt    | Commit/rollback operations                 | Efficient resource use    |

---

## Protocols & Standards

### Common Session Layer Protocols

```
┌──────────────────────────────────────────────────────────────┐
│            SESSION LAYER PROTOCOLS                           │
└──────────────────────────────────────────────────────────────┘

NetBIOS (Network Basic Input/Output System)
├─ Purpose: Session services for Windows networking
├─ Functions: Name registration, session establishment
├─ Modes: Datagram, Broadcast, Session
└─ Port: 137, 138, 139

RPC (Remote Procedure Call)
├─ Purpose: Execute procedures on remote systems
├─ Functions: Request/response handling, marshalling
├─ Implementations: Sun RPC, Microsoft RPC, gRPC
└─ Port: Various (111 for portmapper)

PPTP (Point-to-Point Tunneling Protocol)
├─ Purpose: VPN tunnel creation
├─ Functions: Session establishment, tunnel management
├─ Features: Encapsulation, encryption support
└─ Port: 1723

SQL Session Management
├─ Purpose: Database connection management
├─ Functions: Connection pooling, transaction control
├─ Features: Persistent connections, session state
└─ Port: 1433 (SQL Server), 3306 (MySQL)

SMB (Server Message Block)
├─ Purpose: File/printer sharing
├─ Functions: Session setup, file operations
├─ Versions: SMB 1.0, 2.0, 3.0
└─ Port: 445

SIP (Session Initiation Protocol)
├─ Purpose: VoIP and video call setup
├─ Functions: Session establishment, modification, termination
├─ Features: Call forwarding, transfer
└─ Port: 5060, 5061 (TLS)
```

### Protocol Comparison Table

| Protocol | Primary Use          | Session Type | Connection-Oriented | Port(s)    |
|----------|----------------------|--------------|---------------------|------------|
| NetBIOS  | Windows networking   | Persistent   | Yes                 | 137-139    |
| RPC      | Remote calls         | Per-request  | Yes                 | Various    |
| PPTP     | VPN tunneling        | Persistent   | Yes                 | 1723       |
| SQL      | Database access      | Persistent   | Yes                 | 1433, 3306 |
| SMB      | File sharing         | Persistent   | Yes                 | 445        |
| SIP      | VoIP signaling       | Call-based   | Both                | 5060-5061  |

---

## Session Establishment Process

### Three-Way Session Setup

```
┌──────────────────────────────────────────────────────────────┐
│              SESSION ESTABLISHMENT SEQUENCE                  │
└──────────────────────────────────────────────────────────────┘

CLIENT                                              SERVER
  │                                                     │
  │──────── 1. Session Request ───────────────────────→│
  │         • Protocol version                         │
  │         • Requested parameters                     │
  │         • Client capabilities                      │
  │         • Authentication info                      │
  │                                                     │
  │                                     [Server validates]
  │                                     [Allocates resources]
  │                                                     │
  │←─────── 2. Session Accept ─────────────────────────│
  │         • Session ID assigned                      │
  │         • Agreed parameters                        │
  │         • Server capabilities                      │
  │         • Timeout values                           │
  │                                                     │
  │──────── 3. Acknowledgment ────────────────────────→│
  │         • Confirms receipt                         │
  │         • Ready for data transfer                  │
  │                                                     │
  │                                                     │
  │════════ SESSION ESTABLISHED ═══════════════════════│
  │                                                     │
  │             [Data Exchange Phase]                  │
  │                                                     │
```

### Session Negotiation Parameters

```
PARAMETER NEGOTIATION:

┌────────────────────────────────────────────────────┐
│ Client Proposes:                                   │
│ • Max packet size: 1500 bytes                      │
│ • Timeout: 5 minutes                               │
│ • Compression: gzip                                │
│ • Encryption: TLS 1.3                              │
│ • Keepalive interval: 60 seconds                   │
└────────────────────────────────────────────────────┘
                        ↓
                 [Negotiation]
                        ↓
┌────────────────────────────────────────────────────┐
│ Agreed Parameters:                                 │
│ • Max packet size: 1500 bytes ✓                    │
│ • Timeout: 3 minutes (server preference)           │
│ • Compression: gzip ✓                              │
│ • Encryption: TLS 1.3 ✓                            │
│ • Keepalive interval: 30 seconds (server pref)     │
└────────────────────────────────────────────────────┘
```

### Session Identifiers

| ID Type      | Format                           | Example                              | Persistence |
|--------------|----------------------------------|--------------------------------------|-------------|
| Session ID   | Random string/number             | `a3f5c9e1-4b2d-8f6a-1c3e-9d7b5a2f8c` | Session     |
| Cookie       | Key-value pair                   | `SESSIONID=abc123xyz`                | Browser     |
| Token        | JWT or random string             | `eyJhbGci...`                        | Stateless   |
| Connection ID| Numeric identifier               | `12345`                              | Connection  |

---

## Authentication & Authorization

### Session Authentication Flow

```
┌──────────────────────────────────────────────────────────────┐
│              AUTHENTICATION & SESSION CREATION               │
└──────────────────────────────────────────────────────────────┘

CLIENT                                              SERVER
  │                                                     │
  │──────── 1. Login Request ─────────────────────────→│
  │         Username: alice                            │
  │         Password: ••••••••                         │
  │                                                     │
  │                                     [Verify credentials]
  │                                     [Check user database]
  │                                                     │
  │←─────── 2. Authentication Success ─────────────────│
  │         Session Token: xyz789                      │
  │         Expiry: 1 hour                             │
  │         Permissions: read, write                   │
  │                                                     │
  │         [Client stores token]                      │
  │                                                     │
  │──────── 3. Subsequent Request ────────────────────→│
  │         Token: xyz789                              │
  │         Action: Read file                          │
  │                                                     │
  │                                     [Validate token]
  │                                     [Check permissions]
  │                                     [Execute action]
  │                                                     │
  │←─────── 4. Response ───────────────────────────────│
  │         Data: [file contents]                      │
  │                                                     │
```

### Session Security Methods

```
SECURITY IMPLEMENTATION:

1. SESSION TOKENS
   ┌────────────────────────────────────────────┐
   │ Token: 8f3a9c2e7b1d4f6a8c9e1b3d5f7a       │
   │ Created: 2026-02-12 10:00:00               │
   │ Expires: 2026-02-12 11:00:00               │
   │ User: alice                                │
   │ IP: 192.168.1.100                          │
   │ User-Agent: Mozilla/5.0...                 │
   └────────────────────────────────────────────┘

2. SESSION COOKIES (Web)
   ┌────────────────────────────────────────────┐
   │ Set-Cookie: SESSIONID=abc123;              │
   │             HttpOnly;                      │
   │             Secure;                        │
   │             SameSite=Strict;               │
   │             Max-Age=3600                   │
   └────────────────────────────────────────────┘

3. JWT (JSON Web Token)
   ┌────────────────────────────────────────────┐
   │ Header:                                    │
   │   {"alg": "HS256", "typ": "JWT"}           │
   │                                            │
   │ Payload:                                   │
   │   {"userId": "alice",                      │
   │    "exp": 1676203200,                      │
   │    "role": "admin"}                        │
   │                                            │
   │ Signature:                                 │
   │   HMACSHA256(header + payload, secret)     │
   └────────────────────────────────────────────┘
```

### Session Security Best Practices

| Practice                | Description                          | Security Impact        |
|-------------------------|--------------------------------------|------------------------|
| Token Rotation          | Regenerate tokens periodically       | Prevents token theft   |
| Timeout Management      | Auto-logout after inactivity         | Reduces exposure       |
| IP Binding              | Tie session to source IP             | Prevents hijacking     |
| Secure Storage          | HttpOnly, Secure flags               | Protects cookies       |
| Session Invalidation    | Clear sessions on logout             | Prevents reuse         |
| Multi-factor Auth       | Additional verification              | Stronger authentication|

---

## Practical Examples

### Example 1: Database Session

```
┌──────────────────────────────────────────────────────────────┐
│              DATABASE CONNECTION SESSION                     │
└──────────────────────────────────────────────────────────────┘

APPLICATION                                         DATABASE
     │                                                   │
     │──────── 1. Connection Request ──────────────────→│
     │         Host: db.example.com                     │
     │         Port: 3306                               │
     │         User: app_user                           │
     │         Database: production                     │
     │                                                   │
     │                                   [Authenticate user]
     │                                   [Create session]
     │                                   [Allocate resources]
     │                                                   │
     │←─────── 2. Connection Accepted ──────────────────│
     │         Connection ID: 42                        │
     │         Session ID: sess_8f3a9c2e                │
     │         Timeout: 28800 seconds (8 hours)         │
     │                                                   │
     │──────── 3. Begin Transaction ───────────────────→│
     │         START TRANSACTION;                       │
     │                                                   │
     │←─────── 4. Transaction Started ──────────────────│
     │         Transaction ID: tx_12345                 │
     │                                                   │
     │──────── 5. Execute Query ───────────────────────→│
     │         SELECT * FROM users WHERE id=1;          │
     │                                                   │
     │←─────── 6. Query Results ────────────────────────│
     │         [User data]                              │
     │                                                   │
     │──────── 7. Update Query ────────────────────────→│
     │         UPDATE users SET last_login=NOW()        │
     │         WHERE id=1;                              │
     │                                                   │
     │←─────── 8. Update Confirmed ─────────────────────│
     │         Rows affected: 1                         │
     │                                                   │
     │──────── 9. Commit Transaction ──────────────────→│
     │         COMMIT;                                  │
     │                                                   │
     │←─────── 10. Transaction Committed ───────────────│
     │         [Changes saved]                          │
     │                                                   │
     │         [Session remains open for more queries]  │
     │                                                   │
     │──────── 11. Close Connection ───────────────────→│
     │         QUIT;                                    │
     │                                                   │
     │←─────── 12. Connection Closed ───────────────────│
     │         [Session terminated]                     │
     │         [Resources released]                     │
```

### Example 2: VoIP Call Session (SIP)

```
┌──────────────────────────────────────────────────────────────┐
│                 SIP CALL SESSION FLOW                        │
└──────────────────────────────────────────────────────────────┘

CALLER (Alice)                                   CALLEE (Bob)
     │                                                   │
     │──────── INVITE ─────────────────────────────────→│
     │         To: bob@example.com                      │
     │         From: alice@example.com                  │
     │         Call-ID: call-12345                      │
     │         Session: audio, video                    │
     │                                                   │
     │                                       [Bob's phone rings]
     │                                                   │
     │←─────── 180 Ringing ─────────────────────────────│
     │                                                   │
     │                                       [Bob answers]
     │                                                   │
     │←─────── 200 OK ──────────────────────────────────│
     │         Session Accepted                         │
     │         Codec: G.711                             │
     │         IP: 203.0.113.50:5004                    │
     │                                                   │
     │──────── ACK ─────────────────────────────────────→│
     │                                                   │
     │                                                   │
     │══════════ RTP Media Stream ═══════════════════════│
     │           (Audio/Video data)                     │
     │                                                   │
     │         [Call in progress - 5 minutes]           │
     │                                                   │
     │══════════ RTP Media Stream ═══════════════════════│
     │                                                   │
     │                                       [Alice hangs up]
     │                                                   │
     │──────── BYE ─────────────────────────────────────→│
     │         Call-ID: call-12345                      │
     │                                                   │
     │←─────── 200 OK ──────────────────────────────────│
     │                                                   │
     │         [Session terminated]                     │
     │         [Resources released]                     │
```

### Example 3: Web Session with Shopping Cart

```
┌──────────────────────────────────────────────────────────────┐
│              E-COMMERCE SESSION MANAGEMENT                   │
└──────────────────────────────────────────────────────────────┘

BROWSER                                           WEB SERVER
   │                                                     │
   │──────── 1. Visit Homepage ────────────────────────→│
   │         GET /                                      │
   │                                                     │
   │                                     [Create new session]
   │                                                     │
   │←─────── 2. Response + Session Cookie ──────────────│
   │         Set-Cookie: SESSIONID=xyz789               │
   │         [Homepage HTML]                            │
   │                                                     │
   │         [User browses products]                    │
   │                                                     │
   │──────── 3. Add to Cart ───────────────────────────→│
   │         POST /cart/add                             │
   │         Cookie: SESSIONID=xyz789                   │
   │         Product: Laptop, Qty: 1                    │
   │                                                     │
   │                                     [Validate session]
   │                                     [Update cart in session]
   │                                                     │
   │←─────── 4. Cart Updated ───────────────────────────│
   │         Cart: [Laptop x1]                          │
   │                                                     │
   │──────── 5. Add More Items ────────────────────────→│
   │         POST /cart/add                             │
   │         Cookie: SESSIONID=xyz789                   │
   │         Product: Mouse, Qty: 1                     │
   │                                                     │
   │←─────── 6. Cart Updated ───────────────────────────│
   │         Cart: [Laptop x1, Mouse x1]                │
   │                                                     │
   │         [User continues shopping for 20 minutes]   │
   │                                                     │
   │──────── 7. Proceed to Checkout ───────────────────→│
   │         GET /checkout                              │
   │         Cookie: SESSIONID=xyz789                   │
   │                                                     │
   │                                     [Verify session valid]
   │                                     [Load cart from session]
   │                                                     │
   │←─────── 8. Checkout Page ──────────────────────────│
   │         Cart Items: Laptop, Mouse                  │
   │         Total: $1,299                              │
   │                                                     │
   │──────── 9. Complete Purchase ─────────────────────→│
   │         POST /order/create                         │
   │         Cookie: SESSIONID=xyz789                   │
   │         Payment: [card info]                       │
   │                                                     │
   │                                     [Process payment]
   │                                     [Clear cart]
   │                                     [Create order]
   │                                                     │
   │←─────── 10. Order Confirmation ────────────────────│
   │         Order ID: ORD-98765                        │
   │         [Session cart cleared]                     │
   │                                                     │
   │──────── 11. Logout ───────────────────────────────→│
   │         POST /logout                               │
   │         Cookie: SESSIONID=xyz789                   │
   │                                                     │
   │                                     [Invalidate session]
   │                                     [Clear server data]
   │                                                     │
   │←─────── 12. Logout Confirmed ──────────────────────│
   │         Set-Cookie: SESSIONID=; Max-Age=0          │
   │         [Session destroyed]                        │
```

---

## Code Examples

### Example 1: Session Management (Python - Flask)

```python
#!/usr/bin/env python3
"""
Web Session Management with Flask
Demonstrates Session Layer concepts
"""

from flask import Flask, session, request, jsonify
from datetime import timedelta
import secrets

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)  # Secret key for session encryption
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=1)

# In-memory session store (use Redis/database in production)
active_sessions = {}

@app.route('/login', methods=['POST'])
def login():
    """
    SESSION ESTABLISHMENT
    Authenticate user and create session
    """
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    
    # Simplified authentication (use proper auth in production)
    if username == 'alice' and password == 'secret':
        # Create session
        session_id = secrets.token_urlsafe(32)
        session['user_id'] = username
        session['session_id'] = session_id
        session.permanent = True
        
        # Store session metadata
        active_sessions[session_id] = {
            'user': username,
            'created_at': str(datetime.now()),
            'ip_address': request.remote_addr,
            'user_agent': request.headers.get('User-Agent'),
            'data': {}  # Session data storage
        }
        
        return jsonify({
            'status': 'success',
            'message': 'Session established',
            'session_id': session_id,
            'expires_in': '1 hour'
        }), 200
    
    return jsonify({'status': 'error', 'message': 'Invalid credentials'}), 401

@app.route('/session/data', methods=['GET', 'POST'])
def session_data():
    """
    SESSION MAINTENANCE
    Store and retrieve session data
    """
    if 'session_id' not in session:
        return jsonify({'status': 'error', 'message': 'No active session'}), 401
    
    session_id = session['session_id']
    
    if request.method == 'POST':
        # Store data in session
        data = request.get_json()
        active_sessions[session_id]['data'].update(data)
        
        return jsonify({
            'status': 'success',
            'message': 'Session data updated'
        }), 200
    
    else:
        # Retrieve session data
        session_info = active_sessions.get(session_id, {})
        
        return jsonify({
            'status': 'success',
            'session_id': session_id,
            'user': session_info.get('user'),
            'data': session_info.get('data', {}),
            'created_at': session_info.get('created_at')
        }), 200

@app.route('/logout', methods=['POST'])
def logout():
    """
    SESSION TERMINATION
    Gracefully close session
    """
    if 'session_id' in session:
        session_id = session['session_id']
        
        # Remove from active sessions
        if session_id in active_sessions:
            del active_sessions[session_id]
        
        # Clear session
        session.clear()
        
        return jsonify({
            'status': 'success',
            'message': 'Session terminated'
        }), 200
    
    return jsonify({'status': 'error', 'message': 'No active session'}), 400

@app.route('/session/keepalive', methods=['POST'])
def keepalive():
    """
    SESSION KEEPALIVE
    Extend session timeout
    """
    if 'session_id' not in session:
        return jsonify({'status': 'error', 'message': 'No active session'}), 401
    
    # Update session to extend lifetime
    session.modified = True
    
    return jsonify({
        'status': 'success',
        'message': 'Session extended',
        'expires_in': '1 hour from now'
    }), 200

if __name__ == '__main__':
    app.run(debug=True, port=5000)

# Usage example:
"""
# 1. Login
curl -X POST http://localhost:5000/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}' \
  -c cookies.txt

# 2. Store session data
curl -X POST http://localhost:5000/session/data \
  -H "Content-Type: application/json" \
  -d '{"cart":["laptop","mouse"],"total":1299}' \
  -b cookies.txt

# 3. Retrieve session data
curl http://localhost:5000/session/data -b cookies.txt

# 4. Keepalive
curl -X POST http://localhost:5000/session/keepalive -b cookies.txt

# 5. Logout
curl -X POST http://localhost:5000/logout -b cookies.txt
"""
```

### Example 2: Database Session Management (Python)

```python
#!/usr/bin/env python3
"""
Database Connection Session Management
Demonstrates connection pooling and session lifecycle
"""

import mysql.connector
from mysql.connector import pooling
from contextlib import contextmanager
import time

class DatabaseSessionManager:
    """
    Manages database connection sessions with pooling
    """
    
    def __init__(self, host, user, password, database, pool_size=5):
        """Initialize connection pool"""
        self.connection_pool = pooling.MySQLConnectionPool(
            pool_name="session_pool",
            pool_size=pool_size,
            pool_reset_session=True,
            host=host,
            user=user,
            password=password,
            database=database
        )
        print(f"[Session Manager] Pool created with {pool_size} connections")
    
    @contextmanager
    def get_session(self):
        """
        SESSION ESTABLISHMENT & TERMINATION
        Get a connection from pool (establishes session)
        Automatically returns to pool when done (terminates session)
        """
        connection = None
        try:
            # SESSION ESTABLISHMENT
            connection = self.connection_pool.get_connection()
            session_id = connection.connection_id
            print(f"[Session {session_id}] Established from pool")
            
            yield connection
            
        finally:
            # SESSION TERMINATION
            if connection and connection.is_connected():
                connection.close()  # Returns to pool
                print(f"[Session {session_id}] Returned to pool")
    
    def execute_transaction(self, operations):
        """
        Execute multiple operations within a transaction
        Demonstrates session synchronization
        """
        with self.get_session() as conn:
            cursor = conn.cursor()
            session_id = conn.connection_id
            
            try:
                # Begin transaction (synchronization point)
                conn.start_transaction()
                print(f"[Session {session_id}] Transaction started")
                
                # Execute operations
                for operation in operations:
                    cursor.execute(operation)
                    print(f"[Session {session_id}] Executed: {operation[:50]}...")
                
                # Commit transaction (major synchronization point)
                conn.commit()
                print(f"[Session {session_id}] Transaction committed ✓")
                
                return True
                
            except Exception as e:
                # Rollback on error (synchronization recovery)
                conn.rollback()
                print(f"[Session {session_id}] Transaction rolled back: {e}")
                return False
                
            finally:
                cursor.close()
    
    def session_keepalive(self):
        """
        Maintain active sessions with keepalive
        """
        with self.get_session() as conn:
            session_id = conn.connection_id
            cursor = conn.cursor()
            
            # Send keepalive query
            cursor.execute("SELECT 1")
            cursor.fetchall()
            
            print(f"[Session {session_id}] Keepalive sent")
            cursor.close()

# Usage example
if __name__ == '__main__':
    # Initialize session manager
    db_manager = DatabaseSessionManager(
        host='localhost',
        user='app_user',
        password='password',
        database='testdb',
        pool_size=3
    )
    
    # Example 1: Simple query session
    print("\n=== Example 1: Simple Session ===")
    with db_manager.get_session() as conn:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users LIMIT 5")
        results = cursor.fetchall()
        print(f"Retrieved {len(results)} rows")
        cursor.close()
    
    # Example 2: Transaction session
    print("\n=== Example 2: Transaction Session ===")
    operations = [
        "UPDATE accounts SET balance = balance - 100 WHERE user_id = 1",
        "UPDATE accounts SET balance = balance + 100 WHERE user_id = 2",
        "INSERT INTO transactions (from_user, to_user, amount) VALUES (1, 2, 100)"
    ]
    success = db_manager.execute_transaction(operations)
    print(f"Transaction {'succeeded' if success else 'failed'}")
    
    # Example 3: Multiple concurrent sessions
    print("\n=== Example 3: Concurrent Sessions ===")
    import threading
    
    def worker(session_num):
        time.sleep(0.1 * session_num)  # Stagger requests
        with db_manager.get_session() as conn:
            session_id = conn.connection_id
            print(f"[Worker {session_num}] Using session {session_id}")
            time.sleep(1)  # Simulate work
    
    threads = []
    for i in range(5):
        t = threading.Thread(target=worker, args=(i,))
        threads.append(t)
        t.start()
    
    for t in threads:
        t.join()
    
    print("\n=== All sessions completed ===")
```

### Example 3: Half-Duplex Communication (Python)

```python
#!/usr/bin/env python3
"""
Half-Duplex Communication with Token Passing
Demonstrates dialog control at Session Layer
"""

import socket
import threading
import time
import json

class HalfDuplexSession:
    """
    Implements half-duplex communication with token-based control
    """
    
    def __init__(self, role, peer_address=None):
        self.role = role  # 'server' or 'client'
        self.has_token = (role == 'server')  # Server starts with token
        self.peer_address = peer_address
        self.socket = None
        self.running = False
        
    def start_server(self, host='localhost', port=9000):
        """Start as server (token holder initially)"""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.socket.bind((host, port))
        self.socket.listen(1)
        
        print(f"[Server] Waiting for connection on {host}:{port}")
        self.conn, self.peer_address = self.socket.accept()
        print(f"[Server] Connected to {self.peer_address}")
        
        self.running = True
        
        # Start receive thread
        threading.Thread(target=self._receive_loop, daemon=True).start()
        
    def start_client(self, host='localhost', port=9000):
        """Start as client (waits for token)"""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.connect((host, port))
        self.conn = self.socket
        
        print(f"[Client] Connected to {host}:{port}")
        self.running = True
        
        # Start receive thread
        threading.Thread(target=self._receive_loop, daemon=True).start()
    
    def _receive_loop(self):
        """Receive messages and token passing commands"""
        buffer = b''
        
        while self.running:
            try:
                data = self.conn.recv(1024)
                if not data:
                    break
                
                buffer += data
                
                # Process complete messages (newline-delimited)
                while b'\n' in buffer:
                    message, buffer = buffer.split(b'\n', 1)
                    self._handle_message(message.decode())
                    
            except Exception as e:
                print(f"[{self.role}] Receive error: {e}")
                break
    
    def _handle_message(self, message):
        """Handle received message"""
        try:
            msg = json.loads(message)
            msg_type = msg.get('type')
            
            if msg_type == 'data':
                print(f"[{self.role}] ← Received: {msg['content']}")
                
            elif msg_type == 'token_request':
                print(f"[{self.role}] ← Token requested")
                
            elif msg_type == 'token_grant':
                self.has_token = True
                print(f"[{self.role}] ✓ Token granted - Can send now")
                
            elif msg_type == 'token_release':
                self.has_token = False
                print(f"[{self.role}] Token released")
                
        except json.JSONDecodeError:
            print(f"[{self.role}] Invalid message received")
    
    def send_message(self, content):
        """
        Send message (only if holding token)
        """
        if not self.has_token:
            print(f"[{self.role}] ✗ Cannot send - don't have token")
            return False
        
        message = json.dumps({
            'type': 'data',
            'content': content
        }) + '\n'
        
        self.conn.sendall(message.encode())
        print(f"[{self.role}] → Sent: {content}")
        return True
    
    def request_token(self):
        """Request token from peer"""
        if self.has_token:
            print(f"[{self.role}] Already have token")
            return
        
        message = json.dumps({'type': 'token_request'}) + '\n'
        self.conn.sendall(message.encode())
        print(f"[{self.role}] → Requested token")
    
    def grant_token(self):
        """Grant token to peer"""
        if not self.has_token:
            print(f"[{self.role}] Cannot grant - don't have token")
            return
        
        message = json.dumps({'type': 'token_grant'}) + '\n'
        self.conn.sendall(message.encode())
        self.has_token = False
        print(f"[{self.role}] → Granted token to peer")
    
    def release_token(self):
        """Release token (becomes available)"""
        if not self.has_token:
            print(f"[{self.role}] No token to release")
            return
        
        message = json.dumps({'type': 'token_release'}) + '\n'
        self.conn.sendall(message.encode())
        self.has_token = False
        print(f"[{self.role}] → Released token")
    
    def close(self):
        """Close session"""
        self.running = False
        if self.conn:
            self.conn.close()
        if self.socket:
            self.socket.close()
        print(f"[{self.role}] Session closed")

# Usage Example
if __name__ == '__main__':
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python script.py [server|client]")
        sys.exit(1)
    
    mode = sys.argv[1]
    
    if mode == 'server':
        # Server demonstration
        session = HalfDuplexSession('server')
        session.start_server()
        
        time.sleep(1)
        
        # Server has token initially, send messages
        session.send_message("Hello from server")
        session.send_message("Server message 2")
        
        time.sleep(2)
        
        # Grant token to client
        session.grant_token()
        
        time.sleep(5)
        
        # Request token back
        session.request_token()
        
        time.sleep(10)
        session.close()
        
    elif mode == 'client':
        # Client demonstration
        session = HalfDuplexSession('client')
        session.start_client()
        
        time.sleep(3)
        
        # Client doesn't have token, request it
        session.request_token()
        
        time.sleep(2)
        
        # After receiving token, send messages
        session.send_message("Hello from client")
        session.send_message("Client message 2")
        
        time.sleep(2)
        
        # Release token
        session.release_token()
        
        time.sleep(10)
        session.close()

"""
DEMONSTRATION:

Terminal 1:
$ python half_duplex.py server
[Server] Waiting for connection on localhost:9000
[Server] Connected to ('127.0.0.1', 54321)
[Server] → Sent: Hello from server
[Server] → Sent: Server message 2
[Server] → Granted token to peer
[Server] ← Token requested
[Server] ✓ Token granted - Can send now

Terminal 2:
$ python half_duplex.py client
[Client] Connected to localhost:9000
[Client] ← Received: Hello from server
[Client] ← Received: Server message 2
[Client] ← Token granted - Can send now
[Client] → Sent: Hello from client
[Client] → Sent: Client message 2
[Client] → Released token
"""
```

### Example 4: Session Checkpoint & Recovery

```python
#!/usr/bin/env python3
"""
File Transfer with Checkpoints
Demonstrates synchronization and recovery at Session Layer
"""

import os
import hashlib
import json
import time

class CheckpointedFileTransfer:
    """
    File transfer with checkpoint support for recovery
    """
    
    def __init__(self, chunk_size=1024*1024):  # 1MB chunks
        self.chunk_size = chunk_size
        self.checkpoints = {}
    
    def send_file(self, filepath, checkpoint_interval=5):
        """
        Send file with periodic checkpoints
        
        Args:
            filepath: Path to file to send
            checkpoint_interval: Create checkpoint every N chunks
        """
        filesize = os.path.getsize(filepath)
        total_chunks = (filesize + self.chunk_size - 1) // self.chunk_size
        
        print(f"\n{'='*60}")
        print(f"FILE TRANSFER WITH CHECKPOINTS")
        print(f"{'='*60}")
        print(f"File: {filepath}")
        print(f"Size: {filesize:,} bytes")
        print(f"Chunk size: {self.chunk_size:,} bytes")
        print(f"Total chunks: {total_chunks}")
        print(f"Checkpoint interval: Every {checkpoint_interval} chunks")
        print(f"{'='*60}\n")
        
        with open(filepath, 'rb') as f:
            chunk_num = 0
            bytes_sent = 0
            
            while True:
                chunk_num += 1
                chunk_data = f.read(self.chunk_size)
                
                if not chunk_data:
                    break
                
                # Send chunk
                chunk_hash = hashlib.md5(chunk_data).hexdigest()
                bytes_sent += len(chunk_data)
                progress = (bytes_sent / filesize) * 100
                
                print(f"[Chunk {chunk_num}/{total_chunks}] "
                      f"Sent {len(chunk_data):,} bytes "
                      f"({progress:.1f}% complete)")
                
                # Create checkpoint
                if chunk_num % checkpoint_interval == 0:
                    checkpoint_id = f"CP{chunk_num}"
                    self.checkpoints[checkpoint_id] = {
                        'chunk_num': chunk_num,
                        'bytes_sent': bytes_sent,
                        'timestamp': time.time(),
                        'chunk_hash': chunk_hash
                    }
                    
                    print(f"\n┌{'─'*58}┐")
                    print(f"│ CHECKPOINT {checkpoint_id:44} │")
                    print(f"│ Chunks completed: {chunk_num:38} │")
                    print(f"│ Bytes sent: {bytes_sent:,} bytes{' '*(38-len(f'{bytes_sent:,} bytes'))} │")
                    print(f"│ Progress: {progress:.1f}%{' '*(43-len(f'{progress:.1f}%'))} │")
                    print(f"└{'─'*58}┘\n")
                
                # Simulate network delay
                time.sleep(0.01)
                
                # Simulate connection failure
                if chunk_num == 15:
                    print(f"\n{'⚠'*30}")
                    print("CONNECTION LOST!")
                    print(f"{'⚠'*30}\n")
                    return chunk_num, bytes_sent
        
        print(f"\n✓ Transfer complete: {bytes_sent:,} bytes sent")
        return chunk_num, bytes_sent
    
    def resume_from_checkpoint(self, filepath, checkpoint_id):
        """
        Resume file transfer from last checkpoint
        """
        if checkpoint_id not in self.checkpoints:
            print(f"✗ Checkpoint {checkpoint_id} not found")
            return
        
        cp = self.checkpoints[checkpoint_id]
        
        print(f"\n{'='*60}")
        print(f"RESUMING FROM CHECKPOINT {checkpoint_id}")
        print(f"{'='*60}")
        print(f"Last completed chunk: {cp['chunk_num']}")
        print(f"Bytes already sent: {cp['bytes_sent']:,}")
        print(f"Resuming from chunk: {cp['chunk_num'] + 1}")
        print(f"{'='*60}\n")
        
        filesize = os.path.getsize(filepath)
        
        with open(filepath, 'rb') as f:
            # Seek to checkpoint position
            f.seek(cp['bytes_sent'])
            
            chunk_num = cp['chunk_num']
            bytes_sent = cp['bytes_sent']
            
            while True:
                chunk_num += 1
                chunk_data = f.read(self.chunk_size)
                
                if not chunk_data:
                    break
                
                bytes_sent += len(chunk_data)
                progress = (bytes_sent / filesize) * 100
                
                print(f"[Chunk {chunk_num}] "
                      f"Sent {len(chunk_data):,} bytes "
                      f"({progress:.1f}% complete)")
                
                time.sleep(0.01)
        
        print(f"\n✓ Transfer resumed and completed!")
        print(f"Total bytes sent: {bytes_sent:,}")

# Demonstration
if __name__ == '__main__':
    # Create a test file
    test_file = '/tmp/test_transfer.dat'
    
    # Generate 20MB test file
    print("Creating test file (20MB)...")
    with open(test_file, 'wb') as f:
        f.write(os.urandom(20 * 1024 * 1024))
    
    print(f"✓ Test file created: {test_file}\n")
    
    # Initialize transfer
    transfer = CheckpointedFileTransfer(chunk_size=1024*1024)  # 1MB chunks
    
    # Attempt transfer (will fail at chunk 15)
    last_chunk, bytes_sent = transfer.send_file(test_file, checkpoint_interval=5)
    
    # Wait a bit
    time.sleep(2)
    
    # Resume from last checkpoint
    last_checkpoint = f"CP{(last_chunk // 5) * 5}"  # Round down to last checkpoint
    transfer.resume_from_checkpoint(test_file, last_checkpoint)
    
    # Cleanup
    os.remove(test_file)
    print(f"\n✓ Test file removed")

"""
OUTPUT:

============================================================
FILE TRANSFER WITH CHECKPOINTS
============================================================
File: /tmp/test_transfer.dat
Size: 20,971,520 bytes
Chunk size: 1,048,576 bytes
Total chunks: 20
Checkpoint interval: Every 5 chunks
============================================================

[Chunk 1/20] Sent 1,048,576 bytes (5.0% complete)
[Chunk 2/20] Sent 1,048,576 bytes (10.0% complete)
...
[Chunk 5/20] Sent 1,048,576 bytes (25.0% complete)

┌──────────────────────────────────────────────────────────┐
│ CHECKPOINT CP5                                           │
│ Chunks completed: 5                                      │
│ Bytes sent: 5,242,880 bytes                              │
│ Progress: 25.0%                                          │
└──────────────────────────────────────────────────────────┘

[Chunk 6/20] Sent 1,048,576 bytes (30.0% complete)
...
[Chunk 10/20] Sent 1,048,576 bytes (50.0% complete)

┌──────────────────────────────────────────────────────────┐
│ CHECKPOINT CP10                                          │
│ Chunks completed: 10                                     │
│ Bytes sent: 10,485,760 bytes                             │
│ Progress: 50.0%                                          │
└──────────────────────────────────────────────────────────┘

[Chunk 11/20] Sent 1,048,576 bytes (55.0% complete)
...
[Chunk 15/20] Sent 1,048,576 bytes (75.0% complete)

⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠
CONNECTION LOST!
⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠⚠

============================================================
RESUMING FROM CHECKPOINT CP10
============================================================
Last completed chunk: 10
Bytes already sent: 10,485,760
Resuming from chunk: 11
============================================================

[Chunk 11] Sent 1,048,576 bytes (55.0% complete)
[Chunk 12] Sent 1,048,576 bytes (60.0% complete)
...
[Chunk 20] Sent 1,048,576 bytes (100.0% complete)

✓ Transfer resumed and completed!
Total bytes sent: 20,971,520
"""
```

---

## Troubleshooting Guide

### Common Session Layer Issues

```bash
#!/bin/bash
# Session Layer Troubleshooting Toolkit

echo "SESSION LAYER TROUBLESHOOTING GUIDE"
echo "===================================="

# 1. Session Timeout Issues
echo -e "\n1. DIAGNOSE SESSION TIMEOUTS:"
echo "   Symptoms: Unexpected disconnections, 'session expired' errors"
echo "   "
echo "   Check server timeout settings:"
echo "   $ grep -i timeout /etc/ssh/sshd_config"
echo "   $ mysql -e 'SHOW VARIABLES LIKE \"%timeout%\";'"
echo "   "
echo "   Monitor active sessions:"
echo "   $ netstat -an | grep ESTABLISHED"
echo "   $ ss -tan state established"

# 2. Session Hijacking Detection
echo -e "\n2. DETECT SESSION HIJACKING:"
echo "   Monitor for suspicious session activity:"
echo "   $ tail -f /var/log/auth.log | grep 'session'"
echo "   $ last -a  # Show login history"
echo "   "
echo "   Check for multiple logins:"
echo "   $ who"
echo "   $ w"

# 3. Connection Pool Exhaustion
echo -e "\n3. DATABASE CONNECTION POOL ISSUES:"
echo "   Check active connections:"
echo "   $ mysql -e 'SHOW PROCESSLIST;'"
echo "   $ psql -c 'SELECT * FROM pg_stat_activity;'"
echo "   "
echo "   Monitor pool status:"
echo "   Check application logs for 'pool exhausted' errors"

# 4. Stale Sessions
echo -e "\n4. CLEAN UP STALE SESSIONS:"
echo "   Find idle sessions:"
echo "   $ mysql -e 'SELECT * FROM information_schema.processlist WHERE Time > 3600;'"
echo "   "
echo "   Kill stale sessions (carefully!):"
echo "   $ mysql -e 'KILL <process_id>;'"

# 5. NetBIOS Session Issues
echo -e "\n5. NETBIOS SESSION TROUBLESHOOTING:"
echo "   Check NetBIOS sessions:"
echo "   $ nbstat -S  # Windows"
echo "   $ nmblookup -A <ip_address>  # Linux"
echo "   "
echo "   Test NetBIOS connectivity:"
echo "   $ smbclient -L //<server> -U <user>"

echo -e "\n===================================="
```

### Session State Debugging

```python
#!/usr/bin/env python3
"""
Session State Debugging Tool
"""

import time
from datetime import datetime, timedelta

class SessionDebugger:
    """Debug and monitor session states"""
    
    def __init__(self):
        self.sessions = {}
    
    def add_session(self, session_id, user, metadata=None):
        """Add session for monitoring"""
        self.sessions[session_id] = {
            'user': user,
            'created': datetime.now(),
            'last_activity': datetime.now(),
            'state': 'ACTIVE',
            'requests': 0,
            'data_sent': 0,
            'data_received': 0,
            'metadata': metadata or {}
        }
        print(f"[DEBUG] Session {session_id} created for user '{user}'")
    
    def update_activity(self, session_id, data_sent=0, data_received=0):
        """Update session activity"""
        if session_id in self.sessions:
            session = self.sessions[session_id]
            session['last_activity'] = datetime.now()
            session['requests'] += 1
            session['data_sent'] += data_sent
            session['data_received'] += data_received
    
    def check_timeouts(self, timeout_minutes=30):
        """Check for timed-out sessions"""
        now = datetime.now()
        timeout_delta = timedelta(minutes=timeout_minutes)
        
        print(f"\n{'='*70}")
        print(f"SESSION TIMEOUT CHECK (Timeout: {timeout_minutes} min)")
        print(f"{'='*70}")
        
        timed_out = []
        for session_id, session in self.sessions.items():
            idle_time = now - session['last_activity']
            
            if idle_time > timeout_delta:
                timed_out.append(session_id)
                session['state'] = 'TIMED_OUT'
                print(f"⚠️  Session {session_id}: TIMED OUT")
                print(f"    User: {session['user']}")
                print(f"    Idle: {idle_time}")
            else:
                minutes_left = (timeout_delta - idle_time).seconds // 60
                print(f"✓  Session {session_id}: ACTIVE")
                print(f"    User: {session['user']}")
                print(f"    Time until timeout: {minutes_left} minutes")
        
        return timed_out
    
    def print_session_stats(self):
        """Print detailed session statistics"""
        print(f"\n{'='*70}")
        print(f"SESSION STATISTICS")
        print(f"{'='*70}")
        print(f"Total sessions: {len(self.sessions)}")
        
        active = sum(1 for s in self.sessions.values() if s['state'] == 'ACTIVE')
        print(f"Active sessions: {active}")
        print(f"Inactive sessions: {len(self.sessions) - active}")
        
        print(f"\n{'Session ID':<15} {'User':<12} {'State':<12} {'Requests':<10} {'Data Sent':<12}")
        print(f"{'-'*70}")
        
        for sid, session in self.sessions.items():
            print(f"{sid:<15} "
                  f"{session['user']:<12} "
                  f"{session['state']:<12} "
                  f"{session['requests']:<10} "
                  f"{session['data_sent']:<12}")

# Example usage
if __name__ == '__main__':
    debugger = SessionDebugger()
    
    # Create test sessions
    debugger.add_session('sess_001', 'alice', {'ip': '192.168.1.100'})
    debugger.add_session('sess_002', 'bob', {'ip': '192.168.1.101'})
    debugger.add_session('sess_003', 'charlie', {'ip': '192.168.1.102'})
    
    # Simulate activity
    debugger.update_activity('sess_001', data_sent=1024, data_received=2048)
    debugger.update_activity('sess_001', data_sent=512, data_received=1024)
    debugger.update_activity('sess_002', data_sent=2048, data_received=4096)
    
    # Simulate time passing
    time.sleep(2)
    
    # Update recent activity for one session
    debugger.update_activity('sess_001', data_sent=256)
    
    # Check for timeouts (using very short timeout for demo)
    debugger.check_timeouts(timeout_minutes=0.01)  # 0.6 seconds
    
    # Print statistics
    debugger.print_session_stats()
```

### Session Error Codes

| Error Code | Description                    | Common Cause                  | Solution                        |
|------------|--------------------------------|-------------------------------|---------------------------------|
| 401        | Unauthorized                   | Invalid/expired session       | Re-authenticate                 |
| 408        | Request Timeout                | Session inactive too long     | Extend timeout or reconnect     |
| 419        | Session Expired                | Session lifetime exceeded     | Start new session               |
| 429        | Too Many Requests              | Rate limit exceeded           | Implement backoff               |
| 440        | Login Timeout (IIS)            | Session expired during auth   | Re-login                        |
| 503        | Service Unavailable            | Connection pool exhausted     | Increase pool size              |

---

## Quick Reference

### Session Layer vs Other Layers

```
COMPARISON TABLE:

┌─────────┬──────────────────┬────────────────────────────────────┐
│ Layer   │ Primary Function │ Example                            │
├─────────┼──────────────────┼────────────────────────────────────┤
│ Layer 7 │ User Interface   │ HTTP request: GET /index.html      │
│         │ Application      │                                    │
├─────────┼──────────────────┼────────────────────────────────────┤
│ Layer 6 │ Data Format      │ Encrypt with TLS, Compress GZIP    │
│         │ Presentation     │                                    │
├─────────┼──────────────────┼────────────────────────────────────┤
│ Layer 5 │ Session Mgmt     │ Establish session, Maintain state, │
│         │ Session          │ Dialog control, Checkpoints        │
├─────────┼──────────────────┼────────────────────────────────────┤
│ Layer 4 │ End-to-End       │ TCP connection, Port numbers,      │
│         │ Transport        │ Reliable delivery                  │
├─────────┼──────────────────┼────────────────────────────────────┤
│ Layer 3 │ Routing          │ IP addressing, Packet forwarding   │
│         │ Network          │                                    │
└─────────┴──────────────────┴────────────────────────────────────┘
```

### Session Layer Summary Card

```
┌────────────────────────────────────────────────────────────┐
│         SESSION LAYER (Layer 5) - QUICK REFERENCE          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│ POSITION: Between Presentation (6) and Transport (4)      │
│                                                            │
│ THREE MAIN FUNCTIONS:                                      │
│  1. Session Management (Establish, Maintain, Terminate)    │
│  2. Dialog Control (Simplex, Half-duplex, Full-duplex)    │
│  3. Synchronization (Checkpoints, Recovery)                │
│                                                            │
│ KEY PROTOCOLS:                                             │
│  • NetBIOS  - Windows networking                           │
│  • RPC      - Remote procedure calls                       │
│  • PPTP     - VPN tunneling                                │
│  • SQL      - Database sessions                            │
│  • SIP      - VoIP call setup                              │
│                                                            │
│ COMMON ISSUES:                                             │
│  • Session timeouts                                        │
│  • Connection pool exhaustion                              │
│  • Session hijacking                                       │
│  • Stale sessions                                          │
│                                                            │
│ BEST PRACTICES:                                            │
│  ✓ Implement session timeouts                             │
│  ✓ Use secure session tokens                              │
│  ✓ Monitor active sessions                                │
│  ✓ Implement keepalive mechanisms                         │
│  ✓ Handle reconnection gracefully                         │
│  ✓ Use checkpoints for long operations                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Study Checklist

```
SESSION LAYER MASTERY CHECKLIST:

CORE CONCEPTS:
☐ Understand session lifecycle (establish, maintain, terminate)
☐ Know the three main functions
☐ Understand the layer's position (Layer 5)
☐ Know difference from Transport layer

DIALOG CONTROL:
☐ Understand simplex communication
☐ Understand half-duplex communication
☐ Understand full-duplex communication
☐ Know token-based control mechanisms

SYNCHRONIZATION:
☐ Understand checkpoint concepts
☐ Know major vs minor sync points
☐ Understand recovery mechanisms
☐ Know when to use synchronization

PROTOCOLS:
☐ NetBIOS functionality and use cases
☐ RPC mechanism and implementations
☐ PPTP for VPN tunneling
☐ SQL session management
☐ SIP for VoIP

SESSION MANAGEMENT:
☐ Session establishment process
☐ Session state management
☐ Session termination procedures
☐ Timeout handling

SECURITY:
☐ Session token generation
☐ Authentication methods
☐ Session hijacking prevention
☐ Secure session practices

PRACTICAL SKILLS:
☐ Implement session management in code
☐ Debug session issues
☐ Monitor active sessions
☐ Handle session timeouts
☐ Implement checkpoint recovery

TROUBLESHOOTING:
☐ Diagnose timeout issues
☐ Detect session hijacking
☐ Handle pool exhaustion
☐ Clean up stale sessions
```

---

## Exam Questions

### Multiple Choice

1. What is the primary function of the Session Layer?
   - a) Routing packets
   - b) Managing sessions between applications ✓
   - c) Encrypting data
   - d) Physical transmission

2. Which communication mode allows simultaneous two-way communication?
   - a) Simplex
   - b) Half-duplex
   - c) Full-duplex ✓
   - d) Quarter-duplex

3. What is the purpose of a checkpoint in session management?
   - a) To encrypt data
   - b) To route packets
   - c) To enable recovery from failures ✓
   - d) To compress data

### Short Answer

1. **Explain the difference between session layer and transport layer.**
   - Session Layer: Manages logical connections and dialog between applications
   - Transport Layer: Provides reliable end-to-end data delivery
   - Session Layer works at a higher level of abstraction

2. **What are the three phases of a session lifecycle?**
   - Establishment: Creating and authenticating the session
   - Maintenance: Keeping the session alive and managing data exchange
   - Termination: Gracefully closing the session

3. **Describe token-based dialog control.**
   - A mechanism for half-duplex communication where a "token" represents permission to send data
   - Only the holder of the token can transmit
   - Token must be passed between parties

---

## Summary

The Session Layer (Layer 5) manages communication sessions between applications:

**Key Functions:**
- **Session Management**: Establish, maintain, and terminate connections
- **Dialog Control**: Manage communication flow (simplex/half/full-duplex)
- **Synchronization**: Checkpoints for recovery and data integrity

**Common Uses:**
- Database connections
- VoIP call setup
- File transfer with resume capability
- Remote procedure calls
- VPN tunneling

**Modern Relevance:**
- Web session management (cookies, tokens)
- API session handling
- Database connection pooling
- Real-time communication protocols
- Distributed system coordination