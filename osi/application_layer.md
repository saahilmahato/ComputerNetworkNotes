# Application Layer (OSI Layer 7)

## The User-Facing Layer

### What Makes Layer 7 Special

The Application Layer is **closest to the user**. While lower layers handle the mechanics of data transmission (routing, reliability, bits on wire), Layer 7 defines:

> **What services applications can request over the network**

**Key insight:**

```
Layer 7 doesn't care about:
  ✗ How packets are routed (Layer 3)
  ✗ Whether TCP or UDP is used (Layer 4)
  ✗ What the physical medium is (Layer 1)

Layer 7 only cares about:
  ✓ What request is being made
  ✓ What response should be returned
  ✓ What the data means
```

---

### The Boundary Between Application and Network

```
┌────────────────────────────────────────┐
│     User Application                   │
│  (Browser, Email client, Game)         │
└─────────────────┬──────────────────────┘
                  │ Uses
┌─────────────────▼──────────────────────┐
│  Application Layer Protocols           │
│  (HTTP, SMTP, DNS, FTP)                │ ← Layer 7
│  "The language applications speak"     │
└─────────────────┬──────────────────────┘
                  │ Hands data to
┌─────────────────▼──────────────────────┐
│  Transport Layer (TCP/UDP)             │ ← Layer 4
│  "Delivers data reliably to right port"│
└────────────────────────────────────────┘
```

**Example:**

```
You click a link in your browser

Browser (application):
  • Knows you want a web page
  • Doesn't know how to send packets

HTTP (Layer 7 protocol):
  • Defines: "GET /page.html HTTP/1.1"
  • Specifies: Request format, status codes, headers

TCP (Layer 4):
  • Takes HTTP request as payload
  • Delivers it reliably to server port 80

Lower layers:
  • Route packets across Internet
  • Convert to bits on wire
```

---

## Core Responsibilities

### 1. Protocol Specification

**Defines the rules of communication:**

```
HTTP protocol specifies:
  • Request methods: GET, POST, PUT, DELETE
  • Status codes: 200 (OK), 404 (Not Found), 500 (Server Error)
  • Headers: Content-Type, Authorization, Cache-Control
  • Message format: Request line, headers, body

Without this specification:
  • Browsers wouldn't know how to ask for pages
  • Servers wouldn't know how to respond
  • No interoperability between different vendors
```

---

### 2. Service Abstraction

**Provides high-level services to applications:**

```
DNS (Domain Name System):
  Application wants: "Connect to google.com"
  DNS translates: "google.com" → "142.250.80.46"
  Application gets: IP address to connect to
  
  Abstraction: Application doesn't need to know IP addresses
```

---

### 3. Data Representation

**Ensures both sides understand the data format:**

```
REST API using JSON:
  Client sends:
    POST /api/users
    Content-Type: application/json
    {"name": "Alice", "age": 30}
  
  Server receives:
    Parses JSON
    Creates user object
    Returns response in JSON
  
Both sides agree on JSON format
```

---

### 4. User Authentication & Authorization

**Many Layer 7 protocols handle security:**

```
HTTPS (HTTP + TLS):
  • Authenticates server (certificate verification)
  • Encrypts data (confidentiality)
  • Ensures integrity (data not modified)

SMTP AUTH:
  • Requires username/password to send email
  • Prevents spam relay

SSH:
  • Authenticates user (password or key)
  • Encrypts terminal session
```

---

## Major Application Layer Protocols

### HTTP/HTTPS: The Web's Foundation

**HTTP (HyperText Transfer Protocol)** powers the World Wide Web.

#### How HTTP Works

```
Client-Server Model:

Browser (client) ──[HTTP request]──→ Web server
                 ←──[HTTP response]──

Request:
  GET /index.html HTTP/1.1
  Host: example.com
  User-Agent: Mozilla/5.0
  Accept: text/html

Response:
  HTTP/1.1 200 OK
  Content-Type: text/html
  Content-Length: 1234
  
  <html>...</html>
```

---

#### HTTP Methods

| Method | Purpose | Idempotent? | Safe? | Has Body? |
|--------|---------|-------------|-------|-----------|
| **GET** | Retrieve resource | Yes | Yes | No |
| **POST** | Create resource | No | No | Yes |
| **PUT** | Update/replace | Yes | No | Yes |
| **PATCH** | Partial update | No | No | Yes |
| **DELETE** | Delete resource | Yes | No | Optional |
| **HEAD** | Get headers only | Yes | Yes | No |
| **OPTIONS** | Get allowed methods | Yes | Yes | No |

**Examples:**

```http
GET /api/users/42
  → Retrieve user with ID 42

POST /api/users
  Content-Type: application/json
  {"name": "Alice", "email": "alice@example.com"}
  → Create new user

PUT /api/users/42
  {"name": "Alice", "email": "newemail@example.com"}
  → Replace user 42 entirely

PATCH /api/users/42
  {"email": "newemail@example.com"}
  → Update only email field

DELETE /api/users/42
  → Delete user 42
```

---

#### HTTP Status Codes

**2xx Success:**

```
200 OK
  • Request succeeded
  • Most common status

201 Created
  • Resource created successfully
  • Location header points to new resource

204 No Content
  • Success, but no data to return
  • Common for DELETE operations
```

**3xx Redirection:**

```
301 Moved Permanently
  • Resource permanently moved
  • Example: http://example.com → https://example.com

302 Found (Temporary Redirect)
  • Resource temporarily at different URL
  • Example: After login → redirect to dashboard

304 Not Modified
  • Resource unchanged since last request
  • Use cached version (saves bandwidth)
```

**4xx Client Errors:**

```
400 Bad Request
  • Malformed request
  • Example: Invalid JSON in POST body

401 Unauthorized
  • Authentication required
  • Example: Missing or invalid API token

403 Forbidden
  • Authenticated but not authorized
  • Example: Regular user accessing admin endpoint

404 Not Found
  • Resource doesn't exist
  • Example: /api/users/99999 (no such user)

429 Too Many Requests
  • Rate limit exceeded
  • Example: 1000 requests in 1 minute (limit: 100/min)
```

**5xx Server Errors:**

```
500 Internal Server Error
  • Generic server error
  • Example: Uncaught exception in code

502 Bad Gateway
  • Upstream server invalid response
  • Example: App server crashed, proxy got nothing

503 Service Unavailable
  • Server temporarily unable to handle requests
  • Example: Maintenance mode, overloaded

504 Gateway Timeout
  • Upstream server didn't respond in time
  • Example: Database query took >30s
```

---

#### HTTP Headers

**Request headers:**

```http
Host: api.example.com
  → Which domain (required, enables virtual hosts)

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
  → Client software identifier

Accept: application/json, text/html
  → What content types client understands

Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  → Authentication token

Content-Type: application/json
  → Format of request body

If-None-Match: "686897696a7c876b7e"
  → Conditional request for caching
```

**Response headers:**

```http
Content-Type: application/json
  → Format of response body

Content-Length: 1234
  → Size in bytes

Cache-Control: max-age=3600, public
  → How long to cache (1 hour, can be cached by proxies)

Set-Cookie: session_id=abc123; HttpOnly; Secure
  → Set cookie on client

Location: https://example.com/users/42
  → Where resource was created (201) or moved (3xx)

ETag: "686897696a7c876b7e"
  → Resource version identifier (for caching)
```

---

#### HTTP Versions

**HTTP/1.0 (1996):**
```
• One request per connection
• No persistent connections
• Inefficient (new TCP handshake each request)
```

**HTTP/1.1 (1997):**
```
• Persistent connections (reuse TCP connection)
• Pipelining (send multiple requests without waiting)
• Chunked transfer encoding
• Host header (virtual hosts)

Still widely used today
```

**HTTP/2 (2015):**
```
• Binary protocol (not text)
• Multiplexing (multiple requests on one connection)
• Server push (server sends resources proactively)
• Header compression

Faster, but requires HTTPS
```

**HTTP/3 (2022):**
```
• Based on QUIC (UDP, not TCP)
• Even faster connection establishment
• Better performance on lossy networks
• Built-in encryption

Gradually being adopted
```

---

#### HTTPS: HTTP + TLS

**HTTP alone is insecure:**

```
HTTP (plain):
  GET /login?password=secret123
  
  ↑ Anyone on network can see this!
```

**HTTPS adds encryption:**

```
HTTPS (encrypted with TLS):
  $#@%^&*(!@#$%^&*(
  
  ↑ Encrypted, unreadable to eavesdroppers
```

**What HTTPS provides:**

```
1. Confidentiality (encryption)
   • Data encrypted in transit
   • Only endpoints can read it

2. Integrity (tampering detection)
   • Data cannot be modified without detection
   • MAC (Message Authentication Code) verifies

3. Authentication (server identity)
   • Certificate proves server identity
   • Prevents man-in-the-middle attacks
```

**TLS handshake (simplified):**

```
1. Client → Server: "Hello, I support TLS 1.3"
2. Server → Client: "Here's my certificate"
3. Client: Verifies certificate (signed by trusted CA)
4. Both: Generate shared encryption key
5. Future communication: Encrypted with that key
```

---

### DNS: The Internet's Phone Book

**DNS (Domain Name System)** translates human-readable names to IP addresses.

#### Why DNS Exists

```
Humans prefer: www.google.com
Computers need: 142.250.80.46

DNS bridges the gap
```

---

#### DNS Query Process

```
You type: https://www.example.com

Step 1: Browser checks cache
  • Have I resolved example.com recently?
  • If yes: Use cached IP (done!)
  • If no: Continue...

Step 2: Query local DNS resolver (usually ISP)
  • "What's the IP for www.example.com?"

Step 3: Recursive resolution
  
  Local resolver → Root server (.)
    "Who handles .com?"
    ← "Ask these .com servers: 1.2.3.4, 5.6.7.8"
  
  Local resolver → .com server
    "Who handles example.com?"
    ← "Ask these servers: 9.10.11.12"
  
  Local resolver → example.com nameserver
    "What's the IP for www.example.com?"
    ← "192.0.2.1"

Step 4: Return to browser
  www.example.com → 192.0.2.1

Step 5: Browser connects to 192.0.2.1
```

**Caching at every step:**

```
Browser cache: 5 minutes
OS cache: 1 hour
ISP resolver cache: 1 day
Authoritative servers: TTL specified

Result: Subsequent queries instant
```

---

#### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | example.com → 192.0.2.1 |
| **AAAA** | IPv6 address | example.com → 2001:db8::1 |
| **CNAME** | Canonical name (alias) | www.example.com → example.com |
| **MX** | Mail server | example.com → mail.example.com |
| **TXT** | Text data | SPF, DKIM, domain verification |
| **NS** | Nameserver | example.com → ns1.example.com |
| **SOA** | Start of authority | Zone metadata |

**Example DNS records:**

```
example.com.        3600  IN  A      192.0.2.1
www.example.com.    3600  IN  CNAME  example.com.
example.com.        3600  IN  MX     10 mail.example.com.
mail.example.com.   3600  IN  A      192.0.2.2
example.com.        3600  IN  TXT    "v=spf1 include:_spf.google.com ~all"
```

---

#### DNS Protocol Details

**Uses UDP on port 53 (usually):**

```
Why UDP?
  • Fast (no connection setup)
  • Small queries/responses fit in one packet
  • If no response, just retry
  
Falls back to TCP:
  • For large responses (> 512 bytes)
  • For zone transfers (AXFR)
```

**Query format:**

```
DNS Query:
  Transaction ID: 0x1234 (matches request to response)
  Flags: Standard query
  Questions: 1
    Name: www.example.com
    Type: A (IPv4 address)
    Class: IN (Internet)

DNS Response:
  Transaction ID: 0x1234
  Flags: Response, authoritative
  Answers: 1
    www.example.com → 192.0.2.1
    TTL: 3600 seconds (cache for 1 hour)
```

---

#### Common DNS Tools

```bash
# Query DNS
$ nslookup example.com
Server:  192.168.1.1
Address: 192.168.1.1#53

Non-authoritative answer:
Name:    example.com
Address: 192.0.2.1

# More detailed query
$ dig example.com

; <<>> DiG 9.10.6 <<>> example.com
;; ANSWER SECTION:
example.com.    3600  IN  A  192.0.2.1

# Query specific record type
$ dig example.com MX
example.com.    3600  IN  MX  10 mail.example.com.

# Trace full resolution path
$ dig +trace example.com
. → .com → example.com → 192.0.2.1
```

---

### SMTP/IMAP/POP3: Email Protocols

#### The Email System Architecture

```
┌─────────────┐                    ┌─────────────┐
│   Sender    │                    │  Recipient  │
│   (Alice)   │                    │    (Bob)    │
└──────┬──────┘                    └──────▲──────┘
       │                                  │
       │ Compose email                    │ Read email
       │ (Email client)                   │ (Email client)
       ↓                                  │
┌──────────────────┐                      │
│  SMTP            │                      │
│  (Sending)       │                      │
└──────┬───────────┘                      │
       │                                  │
       ↓                                  │
┌──────────────────┐    ┌──────────────────────┐
│ Alice's Mail     │    │ Bob's Mail           │
│ Server           │───→│ Server               │
│ (smtp.alice.com) │SMTP│ (smtp.bob.com)       │
└──────────────────┘    └──────────┬───────────┘
                                   │
                          ┌────────▼─────────┐
                          │ IMAP/POP3        │
                          │ (Retrieving)     │
                          └──────────────────┘
```

---

#### SMTP: Simple Mail Transfer Protocol

**Purpose:** Send email from client to server, and between servers.

**Port:** 25 (server-to-server), 587 (client-to-server with authentication)

**How SMTP works:**

```
Client → Server conversation:

Client: HELO alice.com
Server: 250 Hello alice.com

Client: MAIL FROM:<alice@alice.com>
Server: 250 OK

Client: RCPT TO:<bob@bob.com>
Server: 250 OK

Client: DATA
Server: 354 Start mail input; end with <CRLF>.<CRLF>

Client: From: alice@alice.com
Client: To: bob@bob.com
Client: Subject: Meeting tomorrow
Client: 
Client: Hi Bob, can we meet tomorrow at 2pm?
Client: .
Server: 250 OK: Message accepted

Client: QUIT
Server: 221 Bye
```

**SMTP is text-based and simple** — which is why email is so reliable!

---

#### IMAP: Internet Message Access Protocol

**Purpose:** Retrieve and manage email on server.

**Port:** 143 (plain), 993 (with TLS)

**Key features:**

```
• Emails stored on server
• Access from multiple devices
• Folder management (Inbox, Sent, Trash)
• Search on server
• Partial message retrieval (headers only)
```

**IMAP workflow:**

```
1. Connect to mail server
2. Authenticate (username/password)
3. Select mailbox (INBOX)
4. Fetch message list
5. Download specific messages
6. Mark as read/unread
7. Delete/move messages
8. Logout

Messages remain on server (synced across devices)
```

---

#### POP3: Post Office Protocol v3

**Purpose:** Download email from server to client.

**Port:** 110 (plain), 995 (with TLS)

**Key features:**

```
• Download-and-delete model
• Emails moved to client
• Simple protocol
• No server-side management
```

**POP3 vs IMAP:**

| Feature | POP3 | IMAP |
|---------|------|------|
| **Storage** | Client (local) | Server |
| **Multi-device** | No (emails on one device) | Yes (sync across devices) |
| **Folder management** | No | Yes |
| **Server search** | No | Yes |
| **Bandwidth** | Downloads entire messages | Can fetch headers only |
| **Use case** | Single device, offline access | Multiple devices, always online |

**Modern trend:** IMAP (with Gmail, Outlook, etc.)

---

### FTP/SFTP: File Transfer

#### FTP: File Transfer Protocol

**Purpose:** Transfer files between client and server.

**Ports:** 21 (control), 20 (data)

**How FTP works:**

```
Two connections:

1. Control connection (port 21):
   • Commands (LIST, RETR, STOR)
   • Persistent during session

2. Data connection (port 20):
   • Actual file transfer
   • Created for each file transfer
   • Closed after transfer
```

**FTP commands:**

```
USER alice          → Login username
PASS secret123      → Login password
PWD                 → Print working directory
CWD /documents      → Change directory
LIST                → List files
RETR file.txt       → Download file
STOR newfile.txt    → Upload file
DELE oldfile.txt    → Delete file
QUIT                → Disconnect
```

**Problem: FTP is insecure**

```
• Passwords sent in plaintext
• Data transferred unencrypted
• Vulnerable to eavesdropping
```

---

#### SFTP: SSH File Transfer Protocol

**Purpose:** Secure file transfer over SSH.

**Port:** 22 (same as SSH)

**Advantages over FTP:**

```
✓ Encrypted (all data and commands)
✓ Single port (firewall-friendly)
✓ SSH authentication (keys, not passwords)
✓ Integrity checking
```

**SFTP usage:**

```bash
# Connect to server
$ sftp user@example.com

sftp> ls                    # List files
sftp> cd /home/user/docs    # Change directory
sftp> get file.txt          # Download file
sftp> put newfile.txt       # Upload file
sftp> rm oldfile.txt        # Delete file
sftp> exit
```

---

### SSH: Secure Shell

**Purpose:** Secure remote access to servers.

**Port:** 22

**What SSH provides:**

```
1. Remote terminal access
   • Execute commands on remote machine
   • Full shell access

2. Secure file transfer (SFTP, SCP)
   • Upload/download files securely

3. Port forwarding (tunneling)
   • Encrypt other protocols through SSH
   • Access internal services securely
```

---

#### SSH Authentication Methods

**1. Password:**

```bash
$ ssh user@server.com
user@server.com's password: ********
```

**2. Public key (preferred):**

```bash
# Generate key pair
$ ssh-keygen -t ed25519
  → Creates: ~/.ssh/id_ed25519 (private key)
             ~/.ssh/id_ed25519.pub (public key)

# Copy public key to server
$ ssh-copy-id user@server.com

# Now login without password
$ ssh user@server.com
  → Automatic authentication with private key
```

---

#### SSH Use Cases

**Remote server administration:**

```bash
# Login to server
$ ssh admin@server.example.com

# Execute single command
$ ssh admin@server.example.com 'uptime'
 10:30:42 up 42 days,  3:15,  1 user,  load average: 0.15, 0.10, 0.08

# Copy files (SCP)
$ scp file.txt user@server.com:/home/user/
$ scp user@server.com:/var/log/app.log ./
```

**Port forwarding (SSH tunnel):**

```bash
# Local port forwarding
$ ssh -L 8080:localhost:80 user@server.com
  → Access server's port 80 via localhost:8080
  → Encrypts traffic through SSH tunnel

# Use case: Access remote database securely
$ ssh -L 5432:localhost:5432 user@dbserver.com
$ psql -h localhost -p 5432  # Connects to remote DB securely
```

**Reverse tunnel:**

```bash
# Allow remote server to access your local service
$ ssh -R 8080:localhost:3000 user@server.com
  → Server's port 8080 → Your localhost:3000
  → Useful for demos, debugging
```

---

### WebSockets: Real-Time Communication

**Purpose:** Full-duplex communication between browser and server.

#### HTTP vs WebSocket

**HTTP (request-response):**

```
Browser → Server: GET /messages
Server → Browser: {messages: [...]}

Browser → Server: GET /messages (poll again)
Server → Browser: {messages: [...]}

Inefficient for real-time updates
```

**WebSocket (bidirectional):**

```
Browser ⇄ Server: Persistent connection

Server → Browser: New message!
Browser → Server: User typed "Hello"
Server → Browser: Another user joined

Efficient, real-time, low latency
```

---

#### WebSocket Handshake

**Upgrade from HTTP to WebSocket:**

```http
Client request:
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server response:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

→ Now using WebSocket protocol (not HTTP)
```

---

#### WebSocket Use Cases

```
• Chat applications (real-time messaging)
• Live notifications (alerts, updates)
• Collaborative editing (Google Docs style)
• Online gaming (multiplayer state sync)
• Financial tickers (stock prices)
• Live sports scores
• IoT device communication
```

**Example (JavaScript):**

```javascript
// Client-side
const socket = new WebSocket('wss://example.com/chat');

socket.onopen = () => {
  console.log('Connected');
  socket.send('Hello, server!');
};

socket.onmessage = (event) => {
  console.log('Received:', event.data);
};

socket.onclose = () => {
  console.log('Disconnected');
};

// Server-side (Node.js with ws library)
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    console.log('Received:', message);
    ws.send('Echo: ' + message);
  });
  
  ws.send('Welcome!');
});
```

---

### DHCP: Dynamic Host Configuration

**Purpose:** Automatically assign IP addresses to devices on a network.

**Port:** 67 (server), 68 (client), UDP

#### Why DHCP Exists

```
Without DHCP:
  • Manually configure IP on every device
  • Ensure no IP conflicts
  • Update when network changes
  • Time-consuming, error-prone

With DHCP:
  • Device connects to network
  • Automatically gets IP, subnet mask, gateway, DNS
  • No manual configuration needed
```

---

#### DHCP Process (DORA)

```
1. Discover
   Client → Broadcast: "I need an IP address!"
   (to 255.255.255.255, because client has no IP yet)

2. Offer
   DHCP Server → Client: "You can have 192.168.1.100"
   (includes: IP, subnet mask, lease time, gateway, DNS)

3. Request
   Client → Broadcast: "I accept 192.168.1.100"
   (broadcast in case multiple DHCP servers offered)

4. Acknowledge
   DHCP Server → Client: "Confirmed. It's yours for 24 hours"

Client now has:
  IP: 192.168.1.100
  Subnet: 255.255.255.0
  Gateway: 192.168.1.1
  DNS: 8.8.8.8, 8.8.4.4
  Lease: 24 hours
```

**After lease expires:**

```
Client renews lease (typically at 50% of lease time)
If renewal fails: Starts DORA process again
```

---

## Application Layer in Practice

### Example: Loading a Web Page

**Complete application-layer flow:**

```
1. User types: https://www.example.com

2. DNS Resolution (Application Layer)
   • Query: "What's the IP for www.example.com?"
   • Response: "93.184.216.34"
   
3. TCP Connection (Transport Layer handles, but triggered by app)
   • Connect to 93.184.216.34:443
   
4. TLS Handshake (Presentation Layer, but initiated by app)
   • Verify server certificate
   • Establish encryption
   
5. HTTP Request (Application Layer)
   GET / HTTP/1.1
   Host: www.example.com
   User-Agent: Mozilla/5.0
   Accept: text/html
   
6. HTTP Response (Application Layer)
   HTTP/1.1 200 OK
   Content-Type: text/html
   
   <html>...</html>
   
7. Browser Renders (Application processes the response)
   • Parse HTML
   • Request CSS, JS, images (more HTTP requests)
   • Execute JavaScript
   • Display page
```

**All application-layer protocols:**
- DNS (step 2)
- HTTP/HTTPS (steps 5-6)
- TLS is technically presentation, but tightly coupled with HTTPS

---

### Example: Sending Email

```
1. User composes email in client (Outlook, Gmail)

2. Client connects to SMTP server
   SMTP: smtp.gmail.com:587

3. SMTP Authentication
   Client: EHLO gmail.com
   Client: AUTH LOGIN
   Server: Authenticate with username/password

4. Send email
   Client: MAIL FROM:<alice@gmail.com>
   Client: RCPT TO:<bob@example.com>
   Client: DATA
   Client: [email content]
   Client: .
   Server: 250 OK

5. Gmail's server delivers to recipient's server
   Gmail SMTP → example.com SMTP (server-to-server SMTP)

6. Recipient retrieves email
   Bob's client → IMAP → example.com mail server
   Downloads email, displays in inbox
```

---

## Security Considerations

### Application-Layer Attacks

**1. SQL Injection:**

```
Vulnerable code:
  query = "SELECT * FROM users WHERE username = '" + input + "'"

Attacker input: admin' OR '1'='1
Result: 
  SELECT * FROM users WHERE username = 'admin' OR '1'='1'
  → Returns all users (authentication bypass!)

Defense: Prepared statements, input validation
```

**2. Cross-Site Scripting (XSS):**

```
Vulnerable code:
  <div>Welcome, <?php echo $_GET['name']; ?>!</div>

Attacker URL: ?name=<script>alert('XSS')</script>
Result:
  <div>Welcome, <script>alert('XSS')</script>!</div>
  → Executes malicious JavaScript

Defense: Escape output, Content-Security-Policy header
```

**3. Cross-Site Request Forgery (CSRF):**

```
User logged into bank.com

Attacker sends email with:
  <img src="https://bank.com/transfer?to=attacker&amount=1000">

Browser automatically sends cookies to bank.com
Bank thinks it's a legitimate request!

Defense: CSRF tokens, SameSite cookies
```

**4. DNS Spoofing:**

```
Attacker compromises DNS server or poisons cache

User requests: bank.com
Fake DNS response: bank.com → 6.6.6.6 (attacker's IP)
User connects to attacker's site thinking it's bank.com

Defense: DNSSEC, HTTPS (certificate mismatch will alert user)
```

---

### Best Practices

```
1. Always use HTTPS (not HTTP)
   • Encrypts data in transit
   • Authenticates server
   
2. Validate all input
   • Never trust user data
   • Sanitize, escape, validate
   
3. Use authentication tokens (not passwords in URLs)
   • JWT, OAuth tokens
   • Expire tokens
   
4. Implement rate limiting
   • Prevent brute force
   • Mitigate DoS
   
5. Keep software updated
   • Security patches
   • Dependency updates
   
6. Use security headers
   • Content-Security-Policy
   • X-Frame-Options
   • Strict-Transport-Security
```

---

## Summary

### Key Takeaways

**1. Application Layer defines services:**
```
Not how data travels, but what services are available
  • HTTP: Web browsing
  • SMTP: Email sending
  • DNS: Name resolution
  • SSH: Secure remote access
```

**2. Protocols enable interoperability:**
```
Standardized protocols mean:
  • Any browser works with any web server
  • Any email client works with any mail server
  • Different vendors' products work together
```

**3. Layer 7 is where software engineers live:**
```
You design:
  • REST APIs (HTTP)
  • Real-time apps (WebSockets)
  • Email systems (SMTP/IMAP)
  • File transfer (SFTP)
  
Understanding protocols = Better system design
```

**4. Security starts at Layer 7:**
```
Application-layer security:
  • HTTPS (encryption)
  • Authentication (OAuth, JWT)
  • Input validation (prevent injection)
  • Rate limiting (prevent abuse)
```

---

### Protocol Quick Reference

| Protocol | Port | Purpose | Encrypted |
|----------|------|---------|-----------|
| **HTTP** | 80 | Web browsing | No |
| **HTTPS** | 443 | Secure web browsing | Yes (TLS) |
| **DNS** | 53 | Name resolution | No* |
| **SMTP** | 25, 587 | Send email | Optional |
| **IMAP** | 143, 993 | Retrieve email | Optional |
| **POP3** | 110, 995 | Download email | Optional |
| **FTP** | 20, 21 | File transfer | No |
| **SFTP** | 22 | Secure file transfer | Yes (SSH) |
| **SSH** | 22 | Secure remote access | Yes |
| **DHCP** | 67, 68 | IP address assignment | No |

*DNS-over-HTTPS (DoH) and DNS-over-TLS (DoT) provide encrypted DNS

---

**Application Layer is where the Internet becomes useful to humans.**

Understanding these protocols means understanding how the modern web works.