# 🌐 WebSockets

---

## 1) Concept Snapshot

**Definition:** WebSocket is a **full-duplex, persistent, message-oriented communication protocol** operating over a single TCP connection, standardized in **RFC 6455 (2011)**. After an initial HTTP-based handshake that upgrades the connection, the protocol switches to its own lightweight binary framing layer — completely separate from HTTP — allowing either endpoint to send data independently at any time.

**Purpose:** The web was built on HTTP's request-response model, which is fundamentally **client-pull**: the server can only speak when spoken to. As applications demanded real-time behavior — live chat, collaborative editing, financial feeds, multiplayer games — developers hacked around this with polling and long-polling, which were inefficient and latency-heavy. WebSockets solve this at the protocol level by making the TCP connection **bidirectional and persistent**, letting the server push data the moment it's available.

**Key properties:**
- Full-duplex (both sides send simultaneously)
- Single TCP connection for the lifetime of the session
- Low framing overhead (2–10 bytes vs. ~500 bytes for HTTP headers)
- Message-oriented (not a raw byte stream from the application's perspective)
- Works over port 80 (`ws://`) or port 443 (`wss://` with TLS)

---

## 2) Mental Model

### Primary Analogy — The Phone Call

HTTP is like **sending letters**. You write a letter (request), send it, the recipient reads it and writes a reply (response), sends it back. Neither side can say anything until the other initiates. Every exchange requires a new envelope, stamps, and routing — even if you're saying "yes" to the previous letter.

WebSockets is like **a phone call**. You dial (handshake), they pick up (101 response), and from that moment both of you can talk freely, simultaneously, any time, with essentially no overhead per word. The line stays open. Either side can hang up when done (close handshake).

### Secondary Analogy — The Pipe

Think of WebSockets as **installing a pipe between two buildings**. HTTP means sending couriers back and forth. Once you install the pipe (WebSocket connection), you can push anything through it instantly in both directions. The pipe has a small label on each chunk of data (the frame header) so the other side knows what it's receiving.

### Visual: HTTP vs. WebSocket Communication Pattern

```
═══════════════════════════════════════════════════════════════════
  HTTP REQUEST-RESPONSE (traditional)
═══════════════════════════════════════════════════════════════════

  Client                          Server
    │                               │
    │──── GET /data (req #1) ──────►│
    │◄─── 200 OK + data (res #1) ───│
    │  [TCP connection closes]       │
    │                               │
    │──── GET /data (req #2) ──────►│   ← Must re-open TCP
    │◄─── 200 OK + data (res #2) ───│
    │  [TCP connection closes]       │
    │                               │
    ↕  (server cannot initiate)     │
    │                               │


═══════════════════════════════════════════════════════════════════
  HTTP LONG POLLING (hack to simulate push)
═══════════════════════════════════════════════════════════════════

  Client                          Server
    │                               │
    │──── GET /poll ───────────────►│
    │       (server holds request)  │
    │                               │  ← Server waits for event
    │                               │  ← Event occurs
    │◄─── 200 OK + event data ──────│
    │──── GET /poll ───────────────►│  ← Must immediately re-poll
    │                               │


═══════════════════════════════════════════════════════════════════
  WEBSOCKET (correct solution)
═══════════════════════════════════════════════════════════════════

  Client                          Server
    │                               │
    │═══ HTTP Upgrade Handshake ════►│
    │◄══ 101 Switching Protocols ════│
    ║                               ║
    ║  [persistent TCP connection]  ║
    ║                               ║
    ║──── message ─────────────────►║  ← Client sends anytime
    ║◄─── message ──────────────────║  ← Server sends anytime
    ║◄─── message ──────────────────║  ← Server pushes unprompted
    ║──── message ─────────────────►║
    ║◄─── message ──────────────────║
    ║                               ║
    ║──── CLOSE frame ─────────────►║
    ║◄─── CLOSE frame ──────────────║
    │  [TCP connection closes]       │
```

### Simplified Story

1. Your browser is on a webpage that needs live stock prices.
2. It makes what *looks* like a normal HTTP request, but with a secret handshake signal: "I speak WebSocket — want to upgrade?"
3. The server says "Sure, switching protocols."
4. From that moment, the HTTP protocol is abandoned on this connection. A new framing layer takes over.
5. The server now blasts price updates the instant they change. The browser can also send data back (e.g., "subscribe to AAPL too").
6. Neither side has to wait for the other. It's a live two-way pipe.

---

## 3) Layer Context

```
┌─────────────────────────────────────────────────────────┐
│              APPLICATION LAYER (L7)                     │
│                                                         │
│   ┌─────────────┐   ┌──────────────────────────────┐    │
│   │    HTTP/1.1 │   │        WebSocket Protocol    │    │
│   │  (for init  │──►│  (takes over after upgrade)  │    │
│   │  handshake) │   │                              │    │
│   └─────────────┘   └──────────────────────────────┘    │
│                              │                          │
│                    ┌─────────▼──────────┐               │
│                    │  Your App Data     │               │
│                    │  (JSON, binary,    │               │
│                    │   MessagePack...)  │               │
│                    └────────────────────┘               │
└─────────────────────────────┬───────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────┐
│         TRANSPORT LAYER (L4) — TCP                      │
│   Port 80 (ws://) or Port 443 (wss://)                  │
└─────────────────────────────┬───────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────┐
│         NETWORK LAYER (L3) — IP                         │
└─────────────────────────────────────────────────────────┘
```

**What talks above it:** Your application logic — JSON messages, game state packets, chat messages, binary data streams.

**What talks below it:** TCP, which provides ordering, reliability, and flow control. WebSockets inherits all of TCP's guarantees.

**Relationship with HTTP:** WebSockets are not HTTP. They *borrow* HTTP for the handshake only because HTTP is universally understood by browsers, servers, and proxies, and using it means WebSockets work on the same ports without firewall drama. After the 101 response, the connection is no longer HTTP in any meaningful way.

**Relationship with TLS:** `wss://` simply wraps the WebSocket connection in TLS (the same way `https://` wraps HTTP). The TLS handshake happens *before* the WebSocket handshake. From TCP's perspective: TCP → TLS → WebSocket framing → your data.

```
wss:// stack:
┌──────────────────────┐
│  Application Data    │  ← JSON, binary, etc.
├──────────────────────┤
│  WebSocket Framing   │  ← Opcodes, masking, length
├──────────────────────┤
│  TLS (encryption)    │  ← Certificate, symmetric encryption
├──────────────────────┤
│  TCP                 │  ← Reliable ordered byte stream
├──────────────────────┤
│  IP                  │  ← Routing
└──────────────────────┘
```

---

## 4) Mechanics (How It Actually Works)

### Phase 1 — TCP Connection

Before anything WebSocket-specific happens, a normal TCP 3-way handshake establishes the connection:

```
Client                    Server
  │──── SYN ────────────►│
  │◄─── SYN-ACK ──────────│
  │──── ACK ────────────►│
  [TCP connection established]
```

If using `wss://`, TLS handshake follows here before any HTTP bytes flow.

### Phase 2 — HTTP Upgrade Handshake

The client sends a carefully crafted HTTP/1.1 GET request:

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: http://example.com

[Optional headers:]
Sec-WebSocket-Protocol: chat, superchat   ← Subprotocol negotiation
Sec-WebSocket-Extensions: permessage-deflate  ← Extension negotiation
```

**Critical header breakdown:**

| Header | Purpose |
|---|---|
| `Upgrade: websocket` | Signals intent to switch protocols |
| `Connection: Upgrade` | Tells proxies this is an upgrade request |
| `Sec-WebSocket-Key` | Random 16-byte value, base64-encoded. Used for handshake verification |
| `Sec-WebSocket-Version: 13` | Must be 13 (the only finalized version per RFC 6455) |
| `Sec-WebSocket-Protocol` | Optional: application-level subprotocol preference (e.g., STOMP, MQTT over WS) |
| `Sec-WebSocket-Extensions` | Optional: request protocol extensions (e.g., compression) |

The server responds with **101 Switching Protocols**:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

[Optional:]
Sec-WebSocket-Protocol: chat         ← Chosen subprotocol
Sec-WebSocket-Extensions: permessage-deflate  ← Accepted extensions
```

### The Key Derivation — Sec-WebSocket-Accept

```
Input:  Sec-WebSocket-Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"
            (client's key)       (magic GUID hardcoded in RFC 6455)

Step 1: Concatenate them as strings
Step 2: Compute SHA-1 hash of the concatenated string
Step 3: Base64-encode the 20-byte SHA-1 hash
Output: Sec-WebSocket-Accept value
```

```
"dGhlIHNhbXBsZSBub25jZQ==" + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"
                    │
                    ▼
              SHA-1 hash
                    │
                    ▼
              Base64 encode
                    │
                    ▼
     "s3pPLMBiTxaQ9kYGzzhZRbK+xOo="
```

**Why this GUID exists:** The magic GUID `258EAFA5...` is hardcoded in the spec and publicly known. Its purpose is *not* security — it's to **prove the server genuinely implemented WebSockets** rather than blindly echoing back whatever the client sent (which a misconfigured HTTP server might do). A server that doesn't know about WebSockets won't know to append this GUID, so the SHA-1 check will fail.

**Why a random key:** The `Sec-WebSocket-Key` is random per connection. This prevents a caching proxy from caching the 101 response and serving it to future connections — the response will never match a cached version because the key is different every time.

### Phase 3 — WebSocket Framing (The Core Protocol)

After the 101, HTTP is done. Every subsequent byte on this TCP connection is a WebSocket frame.

#### Frame Structure (Detailed)

```
 Bit positions:
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┬─┬─┬─┬───────┬─┬───────────────┬───────────────────────────────┤
│F│R│R│R│Opcode │M│  Payload len  │  Extended payload length      │
│I│S│S│S│(4 bit)│A│   (7 bit)     │  (if payload len == 126: 16b) │
│N│V│V│V│       │S│               │  (if payload len == 127: 64b) │
│ │1│2│3│       │K│               │                               │
├─┴─┴─┴─┴───────┴─┴───────────────┴───────────────────────────────┤
│                 Masking key (32 bits) — only if MASK=1          │
├─────────────────────────────────────────────────────────────────┤
│                 Payload data (variable length)                  │
│                 XOR'd with masking key if MASK=1                │
└─────────────────────────────────────────────────────────────────┘
```

**Field-by-field breakdown:**

| Field | Size | Description |
|---|---|---|
| FIN | 1 bit | Is this the final fragment of the message? 1 = yes, 0 = more fragments follow |
| RSV1, RSV2, RSV3 | 1 bit each | Reserved, must be 0 unless an extension defines them (e.g., RSV1=1 means compressed frame with `permessage-deflate`) |
| Opcode | 4 bits | Type of frame — see opcode table below |
| MASK | 1 bit | Is payload masked? **Must be 1 for client→server, must be 0 for server→client** |
| Payload length | 7 bits | If value ≤125: actual length. If 126: read next 2 bytes as uint16. If 127: read next 8 bytes as uint64 |
| Masking key | 32 bits | Present only if MASK=1. 4 random bytes used to XOR the payload |
| Payload | Variable | The actual data, possibly masked |

#### Opcode Table

```
┌────────┬───────────────────┬───────────────────────────────────────────┐
│ Opcode │ Name              │ Description                               │
├────────┼───────────────────┼───────────────────────────────────────────┤
│ 0x0    │ Continuation      │ Continuation frame for a fragmented msg   │
│ 0x1    │ Text              │ UTF-8 text data                           │
│ 0x2    │ Binary            │ Raw binary data                           │
│ 0x3-7  │ Reserved          │ For future non-control frames             │
│ 0x8    │ Close             │ Initiate closing handshake                │
│ 0x9    │ Ping              │ Heartbeat/keepalive probe                 │
│ 0xA    │ Pong              │ Reply to ping (or unsolicited)            │
│ 0xB-F  │ Reserved          │ For future control frames                 │
└────────┴───────────────────┴───────────────────────────────────────────┘
```

#### Masking Algorithm

```
For each byte i in the payload:
    masked_byte[i] = payload[i] XOR masking_key[i % 4]

Example:
  Payload (ASCII "Hello"):  0x48 0x65 0x6C 0x6C 0x6F
  Masking key (random):     0x37 0xfa 0x21 0x3d

  0x48 XOR 0x37 = 0x7f
  0x65 XOR 0xfa = 0x9f
  0x6C XOR 0x21 = 0x4d
  0x6C XOR 0x3d = 0x51
  0x6F XOR 0x37 = 0x58   ← key wraps around (index 4 % 4 = 0)

  Transmitted: 0x7f 0x9f 0x4d 0x51 0x58
```

Unmasking is identical — XOR again with the same key. XOR is its own inverse.

#### Message Fragmentation

A single **message** can be split into multiple **frames**:

```
Large JSON payload split into 3 fragments:

Frame 1: FIN=0, Opcode=0x1 (text), Payload="{'user': 'ali"
Frame 2: FIN=0, Opcode=0x0 (continuation), Payload="ce', 'msg': 'hel"
Frame 3: FIN=1, Opcode=0x0 (continuation), Payload="lo'}"

Receiver reassembles: {'user': 'alice', 'msg': 'hello'}
```

Control frames (Close, Ping, Pong) **cannot be fragmented** and **can be interleaved** between data fragments.

### Phase 4 — Closing Handshake

```
Client                              Server
  │                                   │
  │──── Close frame ─────────────────►│
  │     (opcode 0x8, status 1000)     │
  │                                   │  ← Server processes remaining
  │                                   │     queued data if any
  │◄─── Close frame ──────────────────│
  │     (opcode 0x8, status 1000)     │
  │                                   │
  │──── TCP FIN ─────────────────────►│
  │◄─── TCP FIN ──────────────────────│
  [TCP connection closed]
```

**Close status codes:**

| Code | Meaning |
|---|---|
| 1000 | Normal closure |
| 1001 | Going away (browser navigating away, server shutting down) |
| 1002 | Protocol error |
| 1003 | Unsupported data type |
| 1006 | Abnormal closure (no close frame received — TCP just died) |
| 1007 | Invalid frame payload data (bad UTF-8 in text frame) |
| 1008 | Policy violation |
| 1009 | Message too large |
| 1011 | Server internal error |

---

## 5) Key Structures & Components

### Browser WebSocket API (JavaScript)

```javascript
// Opening a connection
const ws = new WebSocket('wss://example.com/socket');

// Optional: specify subprotocol
const ws = new WebSocket('wss://example.com/socket', ['chat', 'json']);

// Connection lifecycle
ws.onopen    = (event) => { /* connection established */ };
ws.onmessage = (event) => { console.log(event.data); };
ws.onerror   = (event) => { /* error occurred */ };
ws.onclose   = (event) => { 
    console.log(event.code, event.reason, event.wasClean); 
};

// Sending data
ws.send('Hello, text message');
ws.send(JSON.stringify({ type: 'chat', msg: 'hello' }));
ws.send(new ArrayBuffer(32));    // binary
ws.send(new Blob([data]));       // binary

// State machine
ws.readyState:
  0 = CONNECTING   (handshake in progress)
  1 = OPEN         (ready to send/receive)
  2 = CLOSING      (close handshake in progress)
  3 = CLOSED       (connection terminated)

// Closing
ws.close(1000, 'Normal closure');

// Backpressure indicator
ws.bufferedAmount  // bytes queued for send but not yet transmitted
```

### Server-Side State Per Connection

Each WebSocket connection requires the server to maintain:

```
Connection State:
├── Socket file descriptor (the underlying TCP socket)
├── Current readyState (CONNECTING/OPEN/CLOSING/CLOSED)
├── Receive buffer (for partially received frames/messages)
├── Send buffer (frames queued for transmission)
├── Negotiated subprotocol (if any)
├── Negotiated extensions (if any)
├── Masking key buffer (for current incoming frame)
├── Fragment reassembly buffer (if message is fragmented)
└── Application-level state (user ID, subscriptions, etc.)
```

### The permessage-deflate Extension

The most important WebSocket extension. Compresses message payloads using DEFLATE:

```
Negotiation (in handshake):
  Client: Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
  Server: Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits=15

When active:
  RSV1 bit = 1 in compressed frames
  Payload is DEFLATE-compressed (without the 4-byte tail)
  Shared LZ77 sliding window maintained across messages (context takeover)
```

Context takeover means the compression dictionary persists between messages — excellent compression ratio for similar repeated messages (like JSON with consistent key names), but means the compressor state is per-connection (more memory).

### Subprotocols

WebSocket itself is protocol-agnostic — it just delivers messages. Subprotocols define the *meaning* of those messages:

```
Common subprotocols:
├── STOMP (Simple Text Oriented Messaging Protocol)  — used with RabbitMQ, ActiveMQ
├── MQTT over WebSockets                              — IoT devices in browsers
├── GraphQL over WebSockets (graphql-ws)              — real-time GraphQL subscriptions
├── JSON-RPC 2.0                                      — remote procedure calls
├── Socket.IO protocol (proprietary, but very common)
└── Custom application protocols
```

---

## 6) Performance & Tradeoffs

### Overhead Comparison

```
HTTP/1.1 request headers (typical):
┌──────────────────────────────────────────────────────┐
│ GET /data HTTP/1.1\r\n                              (18)
│ Host: example.com\r\n                               (18)
│ User-Agent: Mozilla/5.0 ...\r\n                    (80+)
│ Accept: application/json\r\n                        (28)
│ Accept-Encoding: gzip, deflate, br\r\n              (36)
│ Cookie: session=abc123...\r\n                      (50+)
│ \r\n                                                 (2)
└──────────────────────────────────────────────────────┘
Total: ~400–800 bytes per request

WebSocket frame header:
┌──────────────────────────────────────────────────────┐
│ 2 bytes minimum (for payload ≤ 125 bytes)            │
│ 4 bytes (for payload 126–65535 bytes)                │
│ 10 bytes (for payload > 65535 bytes)                 │
│ + 4 bytes masking key (client→server only)           │
└──────────────────────────────────────────────────────┘
Total: 2–14 bytes per frame
```

### Latency Profile

```
HTTP polling (every 1 second):
  Best case latency = 0ms  (event happens right after poll starts)
  Worst case latency = 1000ms  (event happens right after poll response)
  Average latency = 500ms

HTTP long-polling (30s timeout):
  Best case latency ≈ RTT (round-trip time, usually <100ms)
  Plus one extra RTT for re-establishing the next poll
  Latency spikes on poll re-establishment

WebSocket:
  Latency ≈ RTT only (data travels in one direction over open connection)
  No re-establishment cost
  Server can push the instant data is available
```

### Concurrency Model Requirements

```
Traditional thread-per-connection (Apache prefork, etc.):
  10,000 connections = 10,000 threads
  Each thread: ~8MB stack = 80GB RAM for stacks alone
  → Completely infeasible for WebSocket servers

Event-loop / async I/O (Node.js, nginx, Go, Netty):
  10,000 connections = handled by a handful of threads
  OS manages socket I/O notifications (epoll/kqueue)
  Server thread wakes only when data arrives
  → WebSocket-capable at scale
```

### The C10K and C10M Problem

WebSockets make you care about connection limits. Linux file descriptors, kernel TCP buffers, and memory all become constraints:

```
Per-connection kernel cost (Linux):
  TCP socket: ~4KB kernel memory (sk_buff, tcp_sock structures)
  Receive buffer: 4KB–6MB (auto-tuned)
  Send buffer: 4KB–6MB (auto-tuned)
  
For 100,000 connections:
  Minimum kernel memory: ~400MB just for socket structures
  Typical: 1–4GB depending on buffer sizes

Mitigation:
  Reduce socket buffer sizes for low-traffic WS connections:
  setsockopt(SO_SNDBUF) / setsockopt(SO_RCVBUF)
```

### Throughput vs. Latency Tradeoff

Nagle's algorithm (TCP default) batches small packets to reduce overhead — terrible for WebSockets where you want low latency, not high throughput. Always disable it:

```
TCP_NODELAY = 1  ← Set this on WebSocket server sockets
```

Without `TCP_NODELAY`, small WebSocket messages may be held up to 40ms (Nagle delay) before being sent.

---

## 7) Failure Modes

### Silent Connection Death

**Problem:** NAT routers, firewalls, and load balancers have idle connection timeouts. A WebSocket that carries no traffic for 5 minutes might be silently dropped by an intermediate device — but neither endpoint's TCP stack will know until it tries to send something.

```
Timeline of silent death:

t=0:    WebSocket established
t=0–5min: No data flows
t=5min: NAT router silently removes the connection table entry
t=5min: Client and server both think connection is OPEN
t=6min: Server tries to push a message
        → TCP sends the packet
        → Router has no entry → drops it (or sends TCP RST)
        → onerror / onclose fires on client (finally!)
        
Worst case: Server never tries to send → zombie connection lives forever
```

**Solution — Heartbeat protocol:**

```
Application-level heartbeat (most common):
  Every 30 seconds, server sends: {"type": "ping"}
  Client must respond: {"type": "pong"}
  If no pong received within 10 seconds → close and reconnect

WebSocket protocol-level ping/pong:
  Server sends: Frame(opcode=0x9, payload=optional)
  Client must respond: Frame(opcode=0xA, same payload)
  This happens at the WebSocket layer, not application layer
  Most browser WebSocket APIs handle pong automatically

Recommended: Use both — WS ping/pong for liveness, app-level heartbeat
             for session management
```

### Proxy Interference

```
Problem landscape:
  
  Client ──► Corporate Proxy ──► Internet ──► Server
  
  Transparent HTTP proxy:
    ✗ May not understand HTTP Upgrade
    ✗ May buffer responses (breaking streaming)
    ✗ May reject non-GET requests midstream
    ✗ May have idle connection timeouts (30s–5min)

  CONNECT tunneling (used by ws:// through explicit proxies):
    Client sends: CONNECT server.example.com:80 HTTP/1.1
    Proxy creates tunnel, WebSocket flows through it
    Proxy can't inspect it → less interference

  wss:// (TLS):
    ✓ Proxy cannot inspect encrypted traffic
    ✓ Less likely to interfere
    ✓ Falls back to CONNECT tunnel naturally
    ✓ Port 443 is almost never blocked
    → Always use wss:// in production
```

### Client Reconnection

The connection will drop. You must handle this. A naive reconnect causes a **thundering herd** if thousands of clients all reconnect simultaneously after a server restart:

```javascript
class ResilientWebSocket {
    constructor(url) {
        this.url = url;
        this.attempts = 0;
        this.connect();
    }

    connect() {
        this.ws = new WebSocket(this.url);
        
        this.ws.onopen = () => {
            this.attempts = 0;  // Reset on success
            console.log('Connected');
        };

        this.ws.onclose = () => this.scheduleReconnect();
        this.ws.onerror = () => this.scheduleReconnect();
    }

    scheduleReconnect() {
        // Exponential backoff with jitter
        const base = Math.min(30000, 1000 * Math.pow(2, this.attempts));
        const jitter = Math.random() * 1000;  // ← prevents thundering herd
        const delay = base + jitter;
        
        this.attempts++;
        setTimeout(() => this.connect(), delay);
        
        // Backoff sequence: ~1s, ~2s, ~4s, ~8s, ~16s, ~30s, ~30s...
    }
}
```

### Head-of-Line Blocking

WebSocket inherits TCP's head-of-line blocking. If a large binary frame (e.g., 10MB image) is being transmitted and a tiny urgent control message needs to go through, the control message waits behind the large frame in the TCP buffer.

```
TCP send buffer:
[──── large frame chunk 1/100 ────][ping frame]

The ping frame cannot be sent until all 100 chunks of the large frame
are transmitted. This is fundamental to TCP, not WebSocket specifically.

Mitigation:
  - Use separate WebSocket connections for different priority streams
  - Use smaller message sizes (fragmentation at application level)
  - WebTransport (future) solves this with QUIC's stream multiplexing
```

### State Loss on Reconnect

```
Problem:
  Client subscribes to rooms: [room_a, room_b, room_c]
  Connection drops
  Client reconnects to load balancer → hits Server #2 (was on Server #1)
  Server #2 has no knowledge of client's subscriptions
  Client misses all messages during reconnect + has no subscriptions

Solutions:
  Option A: Client re-sends subscription list on every connect
            (simple, always correct)
  Option B: Session tokens — server stores state in Redis,
            client sends session ID on reconnect, server restores state
  Option C: Server-side event log — client sends "last message ID received"
            server replays missed messages
```

---

## 8) Real-World Usage

### Chat and Messaging
Slack, Discord, WhatsApp Web, Telegram Web all use WebSockets. Every keypress indicator ("Alice is typing..."), message delivery receipt, and read receipt is a WebSocket push. Discord reportedly maintains millions of concurrent WebSocket connections.

### Collaborative Editing
Figma, Google Docs, Notion use WebSockets to sync document state. Every cursor move, keystroke, and selection change is transmitted in real time. Figma uses a custom binary protocol over WebSockets for performance.

### Financial Platforms
Bloomberg Terminal (web), Robinhood, Binance, and virtually all trading platforms stream order book updates, price ticks, and trade executions via WebSockets. Latency of even 100ms is unacceptable — WebSockets reduce this to network RTT.

### Gaming
Agar.io, Slither.io, and countless browser multiplayer games transmit game state over WebSockets. Position updates, collisions, and score changes are sent many times per second in binary frames.

### DevTools and Monitoring
Browser DevTools itself uses WebSockets internally (Chrome DevTools Protocol). Grafana live dashboards, Datadog, and New Relic use WebSockets to stream metrics. Hot module reloading in webpack dev server is WebSocket-based.

### IoT
MQTT — the IoT standard protocol — is commonly proxied over WebSockets so IoT dashboards in browsers can receive sensor data directly.

---

## 9) Comparison Section

### WebSocket vs. Alternatives

| Feature | WebSockets | Server-Sent Events | HTTP/2 Server Push | HTTP Long Polling | WebTransport |
|---|---|---|---|---|---|
| Direction | Full-duplex | Server→Client only | Server→Client only | Simulated push | Full-duplex |
| Protocol | WS over TCP | HTTP/1.1 or HTTP/2 | HTTP/2 | HTTP | QUIC |
| Connection | Persistent | Persistent | Per-request | Repeated | Persistent |
| Browser auto-reconnect | ✗ Manual | ✓ Built-in | N/A | N/A | ✗ Manual |
| Multiplexing | ✗ Single stream | ✓ (over HTTP/2) | ✓ HTTP/2 streams | ✗ | ✓ Multiple streams |
| HOL blocking | ✓ Affected (TCP) | ✓ Affected | ✓ Affected | ✓ Affected | ✗ QUIC solves it |
| Binary support | ✓ Native | ✗ Base64 workaround | ✓ | ✗ | ✓ Native |
| Proxy friendliness | Medium (`wss://` helps) | ✓ Plain HTTP | ✓ Plain HTTP/2 | ✓ Plain HTTP | Poor (QUIC often blocked) |
| Load balancing | Hard (stateful) | Hard (stateful) | Easy (stateless) | Easy (stateless) | Hard |
| Compression | Optional (extension) | Built-in (HTTP) | Built-in (HTTP/2) | Built-in (HTTP) | Built-in |
| Browser support | Universal | Universal (no IE) | Good | Universal | Limited (Chrome-only 2024) |
| Best for | Chat, games, collab | Feeds, notifications | Asset delivery | Simple infrequent updates | Low-latency gaming, video |

### WebSocket vs. Socket.IO

Socket.IO is a *library* built on top of WebSockets (with fallbacks):

| Feature | Raw WebSocket | Socket.IO |
|---|---|---|
| Protocol | Standard RFC 6455 | Custom protocol on top of WS |
| Reconnection | Manual | Automatic |
| Rooms/namespaces | Manual | Built-in |
| Fallback | None | Long-polling fallback |
| Binary | Native WS binary | Supported |
| Acknowledgements | Manual | Built-in |
| Overhead | Minimal | Extra protocol bytes per message |
| Interoperability | Any WS client | Requires Socket.IO client |
| Use when | You control both ends, need performance | Rapid prototyping, need rooms/ACKs |

### ws:// vs. wss://

| | ws:// | wss:// |
|---|---|---|
| Port | 80 | 443 |
| Encryption | None | TLS |
| Proxy behavior | Transparent proxy can inspect | CONNECT tunnel — proxy can't inspect |
| Corporate firewall | Often blocked | Rarely blocked |
| Use in production | Never | Always |

---

## 10) Packet Walkthrough

Full simulation: A user opens a chat app and sends a message.

```
════════════════════════════════════════════════════════════════
STEP 1: TCP Handshake (port 443, TLS)
════════════════════════════════════════════════════════════════

Client (192.168.1.5:54321)          Server (93.184.216.34:443)
           │                                    │
           │──── TCP SYN, seq=1000 ────────────►│
           │◄─── TCP SYN-ACK, seq=5000 ─────────│
           │──── TCP ACK ──────────────────────►│
           │                                    │
           [TLS handshake follows — omitted for brevity]
           [TLS session established, all bytes below are encrypted]


════════════════════════════════════════════════════════════════
STEP 2: WebSocket Upgrade
════════════════════════════════════════════════════════════════

Client sends (inside TLS):
┌─────────────────────────────────────────────────────────────┐
│ GET /chat HTTP/1.1                                          │
│ Host: chat.example.com                                      │
│ Upgrade: websocket                                          │
│ Connection: Upgrade                                         │
│ Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==                 │
│ Sec-WebSocket-Version: 13                                   │
│ Sec-WebSocket-Protocol: chat-v2                             │
│ Origin: https://example.com                                 │
└─────────────────────────────────────────────────────────────┘

Server computes:
  SHA1("x3JJHMbDL1EzLkh9GBhXDw==" + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11")
  = SHA1("x3JJHMbDL1EzLkh9GBhXDw==258EAFA5-E914-47DA-95CA-C5AB0DC85B11")
  = HqR/DVt+B7K9Ls/HHQIAA== (base64 of SHA1 result)

Server responds:
┌─────────────────────────────────────────────────────────────┐
│ HTTP/1.1 101 Switching Protocols                            │
│ Upgrade: websocket                                          │
│ Connection: Upgrade                                         │
│ Sec-WebSocket-Accept: HqR/DVt+B7K9Ls/HHQIAA=                │
│ Sec-WebSocket-Protocol: chat-v2                             │
└─────────────────────────────────────────────────────────────┘

[HTTP protocol ends. WebSocket framing begins.]


════════════════════════════════════════════════════════════════
STEP 3: Server pushes "user joined" notification (server→client)
════════════════════════════════════════════════════════════════

Payload: {"type":"join","user":"alice","room":"general"}
Payload length: 48 bytes

Frame bytes (hex):
  0x81   = 10000001 → FIN=1, RSV=000, Opcode=0001 (text)
  0x30   = 00110000 → MASK=0 (server→client, no mask), Length=48
  [48 bytes of UTF-8 JSON payload, unmasked]

Full frame: 81 30 7b 22 74 79 70 65 22 3a 22 6a 6f 69 6e 22 ...
            ↑  ↑  └────────── JSON payload ──────────────────┘
            │  └── 48 bytes long
            └── FIN=1, text frame


════════════════════════════════════════════════════════════════
STEP 4: User sends message "Hello everyone!" (client→server)
════════════════════════════════════════════════════════════════

Payload: {"type":"msg","text":"Hello everyone!"}
Payload length: 39 bytes

Client generates random masking key: 0x37 0xfa 0x21 0x3d

Frame bytes:
  0x81   = FIN=1, Opcode=text
  0xA7   = 10100111 → MASK=1, Length=39
  0x37 0xfa 0x21 0x3d   ← masking key
  [39 bytes of masked payload]

Server receives, unmasks using the same key:
  Each byte: payload[i] XOR maskingKey[i % 4]
  Recovers: {"type":"msg","text":"Hello everyone!"}


════════════════════════════════════════════════════════════════
STEP 5: Server heartbeat ping (after 30s idle)
════════════════════════════════════════════════════════════════

Server sends:
  0x89   = 10001001 → FIN=1, Opcode=0x9 (ping)
  0x00   = MASK=0, Length=0 (no payload)

Client automatically responds:
  0x8A   = 10001010 → FIN=1, Opcode=0xA (pong)
  0x80   = MASK=1, Length=0
  0x00 0x00 0x00 0x00   ← masking key (payload empty, so irrelevant)


════════════════════════════════════════════════════════════════
STEP 6: Client closes (navigating away)
════════════════════════════════════════════════════════════════

Client sends close frame:
  0x88   = FIN=1, Opcode=0x8 (close)
  0x82   = MASK=1, Length=2
  [masking key]
  [masked 0x03 0xE8] → status code 1000 (normal closure) after unmasking

Server sends close frame back:
  0x88   = FIN=1, Opcode=0x8 (close)
  0x02   = MASK=0, Length=2
  0x03 0xE8   → status code 1000

[TCP FIN exchange follows. Connection fully closed.]
```

---

## 11) Common Interview / Exam Traps

**"WebSockets are HTTP"** — No. The handshake is HTTP. After `101`, it's a completely separate protocol with different framing, different headers (none!), and different semantics.

**"Masking encrypts the data"** — Masking is XOR with a random key that's sent in plaintext in the same frame. Anyone who can see the frame can see the masking key and trivially decode it. Masking provides zero confidentiality. It exists solely to prevent cache poisoning by intermediate proxies.

**"Server-to-client frames should be masked"** — The spec explicitly states server→client frames **must NOT** be masked. If a client receives a masked frame from the server, it must close the connection. This is enforced.

**"You can use WebSockets with HTTP/2"** — Not straightforwardly. Standard WebSockets require HTTP/1.1 for the Upgrade mechanism. HTTP/2 doesn't support `Connection: Upgrade`. RFC 8441 ("Bootstrapping WebSockets with HTTP/2") defines a separate mechanism using HTTP/2's `CONNECT` method with a `:protocol` pseudo-header, but this is not widely supported or used.

**"WebSocket guarantees message delivery"** — Within an established connection, yes (TCP handles it). But if the connection drops while a message is in flight, there is no retransmission at the WebSocket layer. The message may be lost. You must implement application-level acknowledgment if delivery guarantee across reconnects matters.

**"A WebSocket frame = a WebSocket message"** — Not necessarily. Messages can be fragmented across multiple frames. A single frame with `FIN=1` is a complete message, but `FIN=0` means more fragments follow.

**"WebSockets support multiplexing"** — No. One WebSocket connection is one ordered stream. If you need multiple independent channels, you need multiple WebSocket connections or application-level multiplexing. HTTP/2 and QUIC/WebTransport solve this at the protocol level.

**"Long-polling is basically the same as WebSockets for latency"** — Close, but no. Long-polling always has one extra RTT for re-establishing the next poll request immediately after receiving a response. Under high message rates, this adds significant latency and overhead.

**"You need WebSockets for any real-time feature"** — If the data flow is server→client only (notifications, activity feeds, live scores), Server-Sent Events are simpler, automatically reconnect, and go through proxies better. WebSockets shine when you need bidirectional flow.

---

## 12) Retrieval Prompts

- What is `Sec-WebSocket-Key` and why is it random? What does `Sec-WebSocket-Accept` prove?
- Why must client-to-server frames be masked? What attack does it prevent? Why not server-to-client?
- What happens to an HTTP connection after a 101 response?
- Walk through the WebSocket frame structure. How does the receiver know how long the payload is?
- Why do WebSocket servers need event-loop architectures? What fails with thread-per-connection?
- A WebSocket connection silently dies. How? How do you detect it? How do you recover?
- Your WebSocket app has 100,000 concurrent users. The server restarts. What happens? How do you design around this?
- When would you choose SSE over WebSockets? Long polling over SSE?
- What is `TCP_NODELAY` and why does it matter for WebSocket performance?
- A message is fragmented into 3 frames. Draw the FIN and opcode values of each frame.
- Why can't you use WebSockets directly over HTTP/2?
- What does the `bufferedAmount` property tell you, and why should you check it before sending?
- How does `permessage-deflate` work? What's the cost of "context takeover"?
- A user behind a corporate proxy can't connect via `ws://`. Why? How do you fix it?

---

## 13) TL;DR Compression

- WebSocket starts life as an HTTP/1.1 request with `Upgrade: websocket` — the server responds `101 Switching Protocols`, and from that byte onward, the connection speaks a completely different binary framing protocol, not HTTP.
- Frames have 2–14 byte headers (vs. ~500 bytes for HTTP), carry opcodes for text/binary/close/ping/pong, and client→server frames must be XOR-masked (anti-cache-poisoning, not encryption).
- It's full-duplex — both sides send independently at any time — making it the right tool for chat, collaborative editing, live dashboards, and multiplayer games where the server must push data without being asked.
- The main engineering challenges: persistent connections require event-loop servers (Node.js, Go, nginx), load balancing is hard because connections are stateful, silent dead connections require heartbeat protocols, and reconnection must use exponential backoff with jitter.
- Choose WebSockets when you need bidirectional real-time communication; choose SSE when you only need server push; choose HTTP polling when updates are infrequent; consider WebTransport (QUIC-based) when you need multiple independent streams without head-of-line blocking.