# Web Servers: The Internet's Request Handlers

## The Fundamental Problem: Connecting Browsers to Applications

Imagine you type `www.example.com` into your browser and press Enter. Within milliseconds:

- Your browser displays a complete web page
- Images load
- Styles apply
- JavaScript runs

**But how does this actually happen?**

```
You (browser) ────────[Internet]────────→ ??? ────→ Application/Files

What's the "???" that connects these?
```

**Answer: A web server.**

---

## What is a Web Server?

### The Simple Definition

A **web server** is software that:

1. **Listens** for HTTP/HTTPS requests from clients (browsers, apps, other servers)
2. **Processes** those requests (parse, validate, route)
3. **Responds** with the requested resource (HTML, JSON, images) or an error

**The machine running this software is also called a "web server"** (though technically it's just a computer running web server software).

---

### The Critical Boundary

A web server sits at the **boundary between two worlds:**

```
┌─────────────────────────────────────────┐
│         The Internet                    │
│  • Unreliable (packets lost)            │
│  • Slow (variable latency)              │
│  • Untrusted (anyone can send requests) │
│  • Messy (malformed requests, attacks)  │
└────────────────┬────────────────────────┘
                 │
            [WEB SERVER]  ← The gatekeeper
                 │
┌────────────────▼────────────────────────┐
│      Application & Data                 │
│  • Reliable (controlled environment)    │
│  • Fast (local access)                  │
│  • Trusted (authenticated)              │
│  • Structured (clean data)              │
└─────────────────────────────────────────┘
```

**The web server's job:**
- Translate chaotic network requests into clean application calls
- Translate application results into proper HTTP responses
- Protect the application from the Internet's chaos

---

### Real-World Example

**What you see:**

```
Browser: "I want example.com/products"
         ↓
         [Magic happens]
         ↓
Browser: Displays page with product listings
```

**What actually happens:**

```
1. Browser resolves example.com → 93.184.216.34
2. Browser connects to 93.184.216.34:443 (HTTPS port)
3. TLS handshake (encrypted connection established)
4. Browser sends:
   GET /products HTTP/1.1
   Host: example.com
   User-Agent: Mozilla/5.0...
   Accept: text/html...

5. Web server (Nginx) receives request
6. Nginx forwards to application server
7. Application queries database
8. Application generates HTML
9. Web server sends response:
   HTTP/1.1 200 OK
   Content-Type: text/html
   Content-Length: 5432
   
   <html>...</html>

10. Browser renders page
```

**The web server orchestrates steps 5-9.**

---

## Web Server vs Application Server

These terms are often confused. Let's clarify:

### Web Server (HTTP Handler)

**Responsibilities:**
```
• Accept TCP connections
• Perform TLS handshake (HTTPS)
• Parse HTTP requests
• Serve static files efficiently
• Route requests to application code
• Send HTTP responses
• Handle connection pooling
```

**Examples:**
- **Nginx** — Event-driven, high performance
- **Apache** — Process/thread-based, mature
- **Caddy** — Automatic HTTPS, modern

**Optimized for:**
- Network I/O
- Concurrent connections
- Static file serving

---

### Application Server (Business Logic Runner)

**Responsibilities:**
```
• Execute application code
• Process business logic
• Query databases
• Call external APIs
• Generate dynamic content
• Handle application state
```

**Examples:**
- **Go Fiber, Gin** — Go frameworks
- **Express.js** — Node.js framework
- **Spring Boot** — Java framework
- **Django, Flask** — Python frameworks

**Optimized for:**
- Code execution
- Framework features
- Developer productivity

---

### How They Work Together

**Modern architecture:**

```
┌──────────┐
│  Client  │
└────┬─────┘
     │ HTTP request
     ↓
┌────────────────┐
│  Web Server    │ ← Nginx, Caddy
│  (Nginx)       │   • TLS termination
└────┬───────────┘   • Load balancing
     │ Forward       • Static files
     ↓               • Caching
┌────────────────┐
│ App Server 1   │ ← Go Fiber, Express
│ (Go Fiber)     │   • Business logic
├────────────────┤   • Database queries
│ App Server 2   │   • API calls
└────────────────┘
```

**Common patterns:**

**Pattern 1: Separate (traditional)**
```
Nginx (web server) + Go Fiber (app server)
  • Nginx handles TLS, static files
  • Forwards dynamic requests to Go app
  • Best for complex deployments
```

**Pattern 2: Integrated (modern)**
```
Go net/http (both web + app server in one)
  • Single binary
  • Simpler deployment
  • Good for microservices
```

**Pattern 3: Serverless**
```
API Gateway (web server) + Lambda (app server)
  • Managed web server layer
  • Your code runs on-demand
  • Pay per request
```

---

## The Request-Response Cycle

Let's trace a complete HTTP request through the system.

### Step-by-Step: Loading a Product Page

**Scenario:** User visits `https://shop.example.com/products/42`

```
Step 1: DNS Resolution
────────────────────────────────────────────
Browser: "What's the IP of shop.example.com?"
DNS Server: "93.184.216.34"

Step 2: TCP Connection
────────────────────────────────────────────
Browser → 93.184.216.34:443 (HTTPS port)
  SYN →
  ← SYN-ACK
  ACK →
  
TCP connection established

Step 3: TLS Handshake
────────────────────────────────────────────
Browser: "Hello, I support TLS 1.3, here are my cipher suites"
Server: "Let's use TLS 1.3 with AES-256-GCM, here's my certificate"
Browser: (verifies certificate)
Browser: "Here's my encrypted session key"
Server: "Ready for encrypted communication"

Encrypted tunnel established

Step 4: HTTP Request
────────────────────────────────────────────
Browser sends (encrypted):
┌─────────────────────────────────────────┐
│ GET /products/42 HTTP/1.1               │
│ Host: shop.example.com                  │
│ User-Agent: Mozilla/5.0 (Mac...)        │
│ Accept: text/html,application/json      │
│ Cookie: session_id=abc123               │
│ Accept-Encoding: gzip, deflate          │
│                                         │
│ (no body for GET request)               │
└─────────────────────────────────────────┘

Step 5: Server Processing
────────────────────────────────────────────
Web server (Nginx):
  1. Decrypt TLS
  2. Parse HTTP request
  3. Check: Is /products/42 a static file? No
  4. Forward to application server

Application server (Go Fiber):
  1. Route: /products/:id → getProduct handler
  2. Extract id = 42
  3. Validate: Is user authenticated? (check cookie)
  4. Query database: SELECT * FROM products WHERE id=42
  5. Generate JSON response

Step 6: HTTP Response
────────────────────────────────────────────
Server sends (encrypted):
┌─────────────────────────────────────────┐
│ HTTP/1.1 200 OK                         │
│ Content-Type: application/json          │
│ Content-Length: 245                     │
│ Cache-Control: private, max-age=300     │
│ Set-Cookie: session_id=abc123; HttpOnly │
│                                         │
│ {                                       │
│   "id": 42,                             │
│   "name": "Laptop",                     │
│   "price": 999.99,                      │
│   "stock": 15                           │
│ }                                       │
└─────────────────────────────────────────┘

Step 7: Browser Rendering
────────────────────────────────────────────
Browser:
  1. Receives response
  2. Parses JSON
  3. Updates DOM
  4. Displays product info

Total time: ~100ms
```

**This cycle happens millions of times per second on large systems.**

---

## Anatomy of an HTTP Request

### Request Structure

```
┌────────────────────────────────────────────────┐
│              Request Line                      │
│  GET /api/users HTTP/1.1                       │
│  └┬┘ └────┬───┘ └────┬───┘                    │
│   │       │          └─ Protocol version       │
│   │       └──────────── Path                   │
│   └──────────────────── Method                 │
├────────────────────────────────────────────────┤
│              Headers                           │
│  Host: api.example.com                         │
│  User-Agent: curl/7.68.0                       │
│  Accept: application/json                      │
│  Authorization: Bearer eyJ0eXAi...             │
│  Content-Type: application/json                │
│  Content-Length: 57                            │
├────────────────────────────────────────────────┤
│              Blank Line                        │
│                                                │
├────────────────────────────────────────────────┤
│              Body (optional)                   │
│  {                                             │
│    "name": "Alice",                            │
│    "email": "alice@example.com"                │
│  }                                             │
└────────────────────────────────────────────────┘
```

### HTTP Methods (Verbs)

| Method | Purpose | Has Body? | Idempotent? | Safe? |
|--------|---------|-----------|-------------|-------|
| **GET** | Retrieve resource | No | Yes | Yes |
| **POST** | Create resource | Yes | No | No |
| **PUT** | Update/replace resource | Yes | Yes | No |
| **PATCH** | Partial update | Yes | No | No |
| **DELETE** | Delete resource | Optional | Yes | No |
| **HEAD** | Get headers only | No | Yes | Yes |
| **OPTIONS** | Get allowed methods | No | Yes | Yes |

**Idempotent:** Same request repeated multiple times has same effect as once  
**Safe:** Read-only, doesn't modify server state

---

### Important Headers

**Request headers:**

```
Host: example.com
  • Which domain (required for virtual hosts)

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
  • Client software identifier

Accept: text/html,application/json
  • What content types client understands

Accept-Encoding: gzip, deflate, br
  • What compression algorithms supported

Authorization: Bearer <token>
  • Authentication credentials

Cookie: session_id=abc123; user_pref=dark_mode
  • Session and preference data

Content-Type: application/json
  • Format of request body

Content-Length: 245
  • Size of request body in bytes

If-None-Match: "686897696a7c876b7e"
  • Conditional request (for caching)
```

**Web server responsibilities:**

```
✓ Parse all headers correctly
✓ Enforce size limits (prevent DoS)
✓ Validate required headers
✓ Reject malformed requests
✓ Handle special headers (Host, Authorization)
```

---

## Static vs Dynamic Content

### Static Content (Pre-existing Files)

**What it is:**
```
Files that exist on disk and don't change per-request:
  • HTML files (landing pages, documentation)
  • CSS stylesheets
  • JavaScript files
  • Images (PNG, JPEG, SVG)
  • Videos
  • PDFs
  • Fonts
```

**How web servers serve it:**

```
Request: GET /images/logo.png

Web server (optimized path):
  1. Check if file exists: /var/www/html/images/logo.png
  2. Memory-map file (kernel optimization)
  3. Set Content-Type: image/png
  4. Use sendfile() syscall (zero-copy)
  5. Return file directly from kernel buffer

No application code runs!
Extremely fast: 50,000+ requests/second on single server
```

**Nginx static file serving:**

```nginx
server {
    listen 80;
    server_name static.example.com;
    
    location /images/ {
        root /var/www/html;
        expires 30d;  # Cache for 30 days
        add_header Cache-Control "public, immutable";
    }
}
```

**Performance optimizations:**

```
1. Kernel zero-copy (sendfile):
   • Avoid copying data to user space
   • Send directly from page cache to network
   
2. Memory-mapped files:
   • File contents in RAM
   • No read() syscalls needed
   
3. Aggressive caching:
   • Browser caches for days/months
   • CDN caches globally
   • Reduces server load by 90%+
```

---

### Dynamic Content (Generated Per-Request)

**What it is:**
```
Content generated by application code:
  • User profiles (different for each user)
  • Search results (different for each query)
  • API responses (real-time data)
  • Personalized recommendations
```

**How web servers handle it:**

```
Request: GET /api/search?q=laptop

Web server (request dispatcher):
  1. Parse request
  2. Forward to application server
  3. Wait for application response
  4. Add HTTP headers
  5. Send response to client

Application server:
  1. Parse query parameter: q=laptop
  2. Query database: SELECT * FROM products WHERE name LIKE '%laptop%'
  3. Format results as JSON
  4. Return to web server

Much slower: 100-10,000 requests/second
Depends on application complexity
```

**Example: Express.js handling dynamic request:**

```javascript
app.get('/api/search', async (req, res) => {
    const query = req.query.q;
    
    // Application logic
    const results = await database.query(
        'SELECT * FROM products WHERE name LIKE ?',
        [`%${query}%`]
    );
    
    // Generate response
    res.json({
        results: results,
        count: results.length
    });
});
```

---

### Hybrid: Dynamic Generation + Caching

**The best of both worlds:**

```
First request: GET /blog/post-123
  → Application generates HTML (slow)
  → Cache the result
  → Serve from cache for next 1000 requests (fast)

Caching strategies:
  • In-memory (Redis)
  • Reverse proxy cache (Nginx, Varnish)
  • CDN edge cache (Cloudflare, CloudFront)
```

---

## Concurrency: Handling Many Clients Simultaneously

### The Challenge

```
Real-world scenario:
  • 10,000 concurrent users
  • Each connection waits for database query (100ms)
  
Question: How can one server handle this?
```

**Bad solution:** Create 10,000 threads

```
Memory usage: 10,000 threads × 1 MB stack = 10 GB RAM
Context switching: Constant CPU overhead
Result: Server crashes or becomes unresponsive
```

---

### Concurrency Models

#### Model 1: Process-per-Request (Ancient)

```
Apache (old MPM prefork mode):

For each request:
  1. Fork new process
  2. Process handles request
  3. Process exits

Problems:
  ✗ fork() is expensive (~1ms)
  ✗ Each process uses 5-10 MB RAM
  ✗ Doesn't scale beyond ~1000 concurrent requests
  
When used: Early 1990s-2000s
```

---

#### Model 2: Thread-per-Request (Better)

```
Apache (MPM worker mode):

Thread pool:
  • Pre-create 100 threads
  • Each thread handles one request at a time
  • Reuse threads

Improvement:
  ✓ Cheaper than processes
  ✗ Still limited (each thread ~1 MB stack)
  ✗ Context switching overhead
  
Max concurrency: ~5,000 connections
```

---

#### Model 3: Event-Driven (Modern)

```
Nginx, Node.js, Go:

Single-threaded event loop (or small thread pool):
  1. Wait for events (epoll/kqueue)
  2. When connection ready: Process non-blocking
  3. If I/O needed: Register callback, move to next event
  4. When I/O completes: Resume processing

Advantages:
  ✓ Handle 100,000+ concurrent connections
  ✓ Low memory (few KB per connection)
  ✓ No context switching overhead
  
Modern standard for high-performance servers
```

**Event loop visualization:**

```
Event Queue:
┌────────────────────────────────────────┐
│ Connection 1: New request arrived      │
│ Connection 2: Database query complete  │
│ Connection 3: File read complete       │
│ Connection 4: Timeout expired          │
└────────────────────────────────────────┘
         ↓
  Single thread processes
  one event at a time
         ↓
For each event:
  • If non-blocking: Complete immediately
  • If blocking I/O needed: Register callback, continue
         ↓
Never blocks! Always moving forward
```

**Why it works:**

```
Most time is spent waiting:
  • Database query: 50ms
  • File read: 10ms
  • Network I/O: variable
  
CPU processing: <1ms

Event-driven approach:
  • While waiting for Connection 1's database query
  • Process Connection 2's request
  • Process Connection 3's request
  • ...
  • By the time we loop back, Connection 1's query is ready
  
Result: 10,000 connections on 1 thread!
```

---

## Ports: The Network's Addressing System

### What is a Port?

```
IP address identifies a computer: 192.168.1.10
Port identifies a process on that computer: :80

Full address: 192.168.1.10:80
              └─────┬──────┘ └┬┘
              IP address    Port
```

**Analogy:**

```
IP address = Street address (123 Main St)
Port = Apartment number (Apt 5B)

Mail carrier (network) delivers to:
  • 123 Main St (IP) → The building
  • Apt 5B (port) → Specific apartment/person
```

---

### Well-Known Ports

| Port | Protocol | Use |
|------|----------|-----|
| **20, 21** | FTP | File transfer |
| **22** | SSH | Secure shell |
| **23** | Telnet | Unsecure remote access |
| **25** | SMTP | Email sending |
| **53** | DNS | Domain name resolution |
| **80** | HTTP | Web traffic (unencrypted) |
| **110** | POP3 | Email retrieval |
| **143** | IMAP | Email access |
| **443** | HTTPS | Web traffic (encrypted) |
| **3306** | MySQL | Database |
| **5432** | PostgreSQL | Database |
| **6379** | Redis | Cache/database |
| **8080** | HTTP Alt | Development web server |

**Port ranges:**

```
0-1023:    Well-known ports (requires root/admin)
1024-49151: Registered ports (assigned by IANA)
49152-65535: Dynamic/private ports (ephemeral)
```

---

### Multiple Servers on One Machine

```
Same machine can run multiple web servers:

Nginx:     0.0.0.0:80   (HTTP)
Nginx:     0.0.0.0:443  (HTTPS)
App Server: 127.0.0.1:8080 (Internal)
Dev Server: 127.0.0.1:3000 (Development)

OS routes incoming packets to correct process based on port
```

**How binding works:**

```javascript
// Go example
http.ListenAndServe(":8080", handler)

What happens:
  1. Application calls bind() syscall
  2. Kernel reserves port 8080 for this process
  3. Incoming TCP connections to port 8080 delivered to this process
  4. Other processes cannot bind to 8080 (already in use)
```

---

## HTTPS and TLS: Security Layer

### HTTP vs HTTPS

**HTTP (Insecure):**
```
Browser ────────[Plaintext]────────→ Server
       "GET /login?password=secret123"
                  ↑
          Anyone can read this!
          (ISP, wifi router, hackers)
```

**HTTPS (Secure):**
```
Browser ────────[Encrypted]────────→ Server
       "$#@%^&*(!@#$%^&*()"
                  ↑
          Unreadable to eavesdroppers
```

---

### What TLS Provides

**1. Confidentiality (Encryption):**
```
Data encrypted with symmetric key (AES-256)
  • Only sender and receiver can decrypt
  • Eavesdroppers see random noise
```

**2. Integrity (Tampering Detection):**
```
Message Authentication Code (MAC) added to each message
  • If data modified in transit → MAC check fails
  • Receiver rejects corrupted data
```

**3. Authentication (Server Identity):**
```
Server proves identity with certificate:
  • Certificate signed by trusted Certificate Authority (Let's Encrypt, DigiCert)
  • Browser verifies signature chain
  • Prevents man-in-the-middle attacks
```

---

### TLS Handshake Process

```
Step 1: ClientHello
────────────────────────────────────────────
Browser → Server:
  • TLS version supported (1.3)
  • Cipher suites (AES-256-GCM, ChaCha20-Poly1305)
  • Random number (for key generation)

Step 2: ServerHello
────────────────────────────────────────────
Server → Browser:
  • Chosen TLS version (1.3)
  • Chosen cipher suite (AES-256-GCM)
  • Server certificate (contains public key)
  • Random number

Step 3: Certificate Verification
────────────────────────────────────────────
Browser:
  • Verifies certificate signature chain
  • Checks certificate not expired
  • Ensures domain matches (example.com)
  • Checks certificate not revoked

Step 4: Key Exchange
────────────────────────────────────────────
Browser:
  • Generates session key using Diffie-Hellman
  • Encrypts with server's public key
  • Sends to server

Server:
  • Decrypts with private key
  • Both sides now have shared secret

Step 5: Encrypted Communication
────────────────────────────────────────────
All future messages encrypted with session key
```

**Performance cost:**

```
TLS handshake: ~50-100ms overhead
  • Only on initial connection
  • Connections reused (HTTP/2, HTTP/3)
  • Amortized over many requests

Modern servers handle TLS efficiently:
  • Hardware acceleration (AES-NI)
  • Session resumption
  • TLS 1.3 (faster handshake)
```

---

### Certificate Management

**Web server configuration (Nginx):**

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
}
```

**Automatic certificates (Caddy):**

```
# Caddyfile
example.com {
    root * /var/www/html
    file_server
}

# That's it! Caddy automatically:
# - Obtains certificate from Let's Encrypt
# - Renews before expiration
# - Configures HTTPS
```

---

## Reverse Proxies and Load Balancers

### What is a Reverse Proxy?

**Forward proxy (VPN):**
```
You → Proxy → Internet
     (hides your IP)
```

**Reverse proxy:**
```
Internet → Proxy → Your servers
          (hides server details)
```

---

### Reverse Proxy Responsibilities

```
┌──────────┐
│  Client  │
└────┬─────┘
     │ HTTPS request
     ↓
┌────────────────────────┐
│   Reverse Proxy        │ ← Nginx, HAProxy, AWS ALB
│   (Nginx)              │
│                        │
│ Responsibilities:      │
│ • TLS termination      │
│ • Load balancing       │
│ • Caching             │
│ • Compression (gzip)   │
│ • Rate limiting        │
│ • WAF (firewall)       │
│ • Header manipulation  │
└────┬───────────────────┘
     │ HTTP (internal network)
     ↓
┌────────────────────────┐
│  Backend Servers       │
│  ┌──────┐  ┌──────┐   │
│  │ App 1│  │ App 2│   │
│  └──────┘  └──────┘   │
└────────────────────────┘
```

---

### Load Balancing Algorithms

**1. Round Robin:**
```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1 (cycle repeats)

Simple, fair distribution
```

**2. Least Connections:**
```
Send request to server with fewest active connections

Server 1: 10 connections
Server 2: 5 connections  ← Send here
Server 3: 8 connections

Better for long-lived connections
```

**3. IP Hash:**
```
Hash(client IP) mod (number of servers) = server

Same client always → same server
Useful for sticky sessions
```

**4. Weighted Round Robin:**
```
Server 1 (powerful): Weight 3
Server 2 (medium):   Weight 2
Server 3 (weak):     Weight 1

Distribution: S1, S1, S1, S2, S2, S3, repeat

Accounts for different server capacities
```

---

### Nginx Reverse Proxy Configuration

```nginx
upstream backend {
    least_conn;  # Load balancing algorithm
    
    server 192.168.1.101:8080 weight=3;
    server 192.168.1.102:8080 weight=2;
    server 192.168.1.103:8080 weight=1 backup;
}

server {
    listen 443 ssl;
    server_name api.example.com;
    
    ssl_certificate /etc/ssl/certs/api.example.com.crt;
    ssl_certificate_key /etc/ssl/private/api.example.com.key;
    
    location / {
        proxy_pass http://backend;
        
        # Forward client info
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        
        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 30s;
    }
    
    location /static/ {
        root /var/www;
        expires 30d;
    }
}
```

---

## Virtual Hosts: One Server, Many Websites

### The Problem

```
Old way (one IP per website):
  website1.com → 1.2.3.4
  website2.com → 1.2.3.5
  website3.com → 1.2.3.6
  
Problem: Running out of IPv4 addresses!
```

### The Solution: Virtual Hosts

```
All websites → Same IP (1.2.3.4)

How does server know which website?
  → Host header in HTTP request!
```

**How it works:**

```
Request 1:
  GET / HTTP/1.1
  Host: website1.com
  
  → Server serves website1's content

Request 2:
  GET / HTTP/1.1
  Host: website2.com
  
  → Server serves website2's content

Same IP, same server, different content based on Host header
```

---

### Nginx Virtual Host Configuration

```nginx
# website1.com
server {
    listen 80;
    server_name website1.com www.website1.com;
    root /var/www/website1;
    index index.html;
}

# website2.com
server {
    listen 80;
    server_name website2.com www.website2.com;
    root /var/www/website2;
    index index.html;
}

# Default catch-all
server {
    listen 80 default_server;
    return 444;  # Close connection (no Host header or unrecognized)
}
```

**This is how shared hosting works:**

```
Shared hosting provider:
  • One physical server
  • 100 customer websites
  • All on same IP
  • Virtual hosts separate them
  
Customer 1: myshop.com     → /var/www/customer1/
Customer 2: myblog.com     → /var/www/customer2/
Customer 3: portfolio.com  → /var/www/customer3/
...
```

---

## HTTP Status Codes: Communication via Numbers

### Status Code Categories

| Range | Category | Meaning |
|-------|----------|---------|
| **1xx** | Informational | Request received, continuing |
| **2xx** | Success | Request successful |
| **3xx** | Redirection | Further action needed |
| **4xx** | Client Error | Request contains error |
| **5xx** | Server Error | Server failed to fulfill valid request |

---

### Common Status Codes

**2xx Success:**

```
200 OK
  • Request succeeded
  • Most common status
  • Example: GET /index.html → Returns page

201 Created
  • Resource created successfully
  • Example: POST /api/users → New user created
  • Response often includes Location header

204 No Content
  • Success, but no data to return
  • Example: DELETE /api/users/123 → User deleted
```

**3xx Redirection:**

```
301 Moved Permanently
  • Resource permanently moved
  • Example: http://example.com → https://example.com
  • Browsers update bookmarks

302 Found (Temporary Redirect)
  • Resource temporarily at different URL
  • Example: /login redirects to /dashboard after auth
  • Don't update bookmarks

304 Not Modified
  • Resource unchanged (caching)
  • Example: GET /style.css with If-None-Match header
  • Browser uses cached version
```

**4xx Client Errors:**

```
400 Bad Request
  • Malformed request
  • Example: Invalid JSON in POST body

401 Unauthorized
  • Authentication required
  • Example: No or invalid API token

403 Forbidden
  • Authenticated but not authorized
  • Example: Admin endpoint accessed by regular user

404 Not Found
  • Resource doesn't exist
  • Example: GET /api/users/99999 (no such user)

429 Too Many Requests
  • Rate limit exceeded
  • Example: 1000 requests in 1 minute (limit: 100/min)
```

**5xx Server Errors:**

```
500 Internal Server Error
  • Generic server error
  • Example: Uncaught exception in application code

502 Bad Gateway
  • Upstream server returned invalid response
  • Example: App server crashed, proxy got no response

503 Service Unavailable
  • Server temporarily unable to handle requests
  • Example: Maintenance mode, overloaded

504 Gateway Timeout
  • Upstream server didn't respond in time
  • Example: Database query took >30s, proxy timed out
```

---

### Why Correct Status Codes Matter

**Caching behavior:**
```
200 OK → Cacheable (with headers)
404 Not Found → Cacheable (don't keep asking for non-existent resources)
500 Internal Server Error → Not cacheable (might be temporary)
```

**Browser behavior:**
```
301 Moved Permanently → Update bookmarks, permanent redirect
302 Found → Temporary, don't update bookmarks
304 Not Modified → Use cached copy
```

**API clients:**
```
if status == 200:
    process_data(response)
elif status == 404:
    handle_not_found()
elif status >= 500:
    retry_with_backoff()
```

---

## Performance Metrics

### Latency (Time to First Byte)

```
What it measures:
  Time from request sent to first byte of response received

Components:
  • Network latency (client → server)
  • TLS handshake (if HTTPS)
  • Request processing
  • Application logic
  • Response start

Target: <200ms (good), <100ms (excellent)
```

**Measuring with curl:**

```bash
$ curl -w "Total: %{time_total}s\nTTFB: %{time_starttransfer}s\n" https://example.com

Total: 0.245s
TTFB: 0.187s
```

---

### Throughput (Requests Per Second)

```
What it measures:
  How many requests server can handle per second

Factors:
  • Static content: 50,000+ RPS (single server)
  • Dynamic content: 100-10,000 RPS (depends on complexity)
  • Database-heavy: 10-1,000 RPS

Benchmarking with Apache Bench:
```bash
$ ab -n 10000 -c 100 https://example.com/
# 10,000 requests, 100 concurrent
```

---

### Concurrency (Simultaneous Connections)

```
What it measures:
  How many clients can be served at once

Server models:
  • Process/thread-based: 1,000-10,000 concurrent
  • Event-driven (Nginx): 100,000+ concurrent

Resource consumption:
  • Per-thread: ~1 MB RAM
  • Event-driven: ~5 KB RAM per connection
```

---

### Error Rate

```
What it measures:
  Percentage of requests that fail

Calculation:
  Error Rate = (4xx + 5xx responses) / (total requests)

Targets:
  • <0.1% (excellent)
  • <1% (acceptable)
  • >5% (problematic)

Monitoring:
  • 4xx errors → Client issues (bad requests, auth failures)
  • 5xx errors → Server issues (crashes, timeouts, bugs)
```

---

## What Happens When You Deploy: From Code to Production

Let's demystify what actually happens when you `git push` or `deploy` your backend.

### Step 1: From Source Code to Executable

**Go Fiber example:**

```go
// main.go
package main

import "github.com/gofiber/fiber/v2"

func main() {
    app := fiber.New()
    
    app.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })
    
    app.Listen(":8080")
}
```

**Build process:**

```bash
$ go build -o myapp main.go

Output: myapp (native binary, ~10 MB)
  • Contains all dependencies
  • Statically linked (no runtime needed)
  • Just a file on disk (not running yet)
```

**Spring Boot example:**

```bash
$ mvn package

Output: myapp.jar (fat JAR, ~50 MB)
  • Contains all dependencies
  • Requires JVM to run
  • Just a file on disk
```

---

### Step 2: Platform Provisions Compute

**What the platform does (AWS, Render, Fly.io):**

```
1. Allocate compute resource:
   • EC2 instance (virtual machine)
   • ECS container (Docker)
   • Lambda function (serverless)

2. Install runtime if needed:
   • Go binary: No runtime needed
   • Java: Install JVM
   • Node.js: Install Node runtime

3. Copy your application:
   • Upload binary/jar to server
   • Set file permissions (chmod +x)

4. Configure environment:
   • Set environment variables (DATABASE_URL, API_KEYS)
   • Configure network (security groups, firewalls)
```

---

### Step 3: Starting the Application

**The platform runs your app as a process:**

```bash
# Go
$ ./myapp

# Java
$ java -jar myapp.jar

# Node.js
$ node index.js
```

**What happens internally:**

```
1. Operating system creates process (PID assigned)
2. Process requests to bind to port 8080
3. Kernel reserves port 8080 for this process
4. Application starts event loop / thread pool
5. Process becomes ready to accept connections

$ ps aux | grep myapp
USER   PID   %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
app    1234  0.5  2.0  500000 204800 ?   Sl   10:30   0:05 ./myapp
```

**Your application is now just an OS process:**

- Has PID (Process ID): 1234
- Uses memory (RSS): 200 MB
- Uses CPU: 0.5%
- Listens on port: 8080
- Runs as user: app
- Can be killed with: `kill 1234`

---

### Step 4: Port Binding and Networking

**Application binds to port:**

```go
// Inside your code
app.Listen(":8080")

// What actually happens:
1. Application calls bind() syscall
   socket = bind(0.0.0.0:8080)

2. Kernel checks:
   • Is port 8080 available? (not already in use)
   • Does process have permission? (ports <1024 need root)

3. Kernel reserves port 8080
   • Incoming TCP connections to :8080 delivered to this process

4. Application calls listen() syscall
   • Starts accepting connections

5. Application calls accept() in loop
   • Blocks until connection arrives
   • Wakes up when client connects
```

**From network perspective:**

```
Before:
  Port 8080 → (nothing listening)
  
After your app starts:
  Port 8080 → Your application (PID 1234)
  
Network packets:
  Destination: <server-ip>:8080
       ↓
  Kernel routes to your process
       ↓
  Your application receives connection
```

---

### Step 5: The Real Request Path in Production

**Your app is never directly exposed to the Internet.**

**Typical architecture:**

```
Internet
    ↓
┌───────────────────────┐
│  DNS                  │
│  example.com          │
│  → 52.1.2.3 (Load Balancer IP)
└─────────┬─────────────┘
          ↓
┌───────────────────────┐
│  Load Balancer        │ ← AWS ALB, Nginx
│  (52.1.2.3:443)       │   • Public IP
└─────────┬─────────────┘   • TLS termination
          │                 • Health checks
          ↓
┌───────────────────────┐
│  Reverse Proxy        │ ← Nginx, Envoy
│  (10.0.1.5:80)        │   • Private IP
└─────────┬─────────────┘   • Request routing
          │                 • Rate limiting
          ↓
┌───────────────────────┐
│  Your Application     │ ← Your Go/Java/Node app
│  Instance 1           │   • Private IP
│  (10.0.2.10:8080)     │   • Only accessible internally
├───────────────────────┤   • Receives already-parsed requests
│  Instance 2           │
│  (10.0.2.11:8080)     │
└───────────────────────┘
```

**What your application sees:**

```
Incoming request appears to come from:
  Source IP: 10.0.1.5 (the proxy, not the real client!)
  
To get real client IP:
  X-Forwarded-For: 203.0.113.45 (header added by proxy)
  X-Real-IP: 203.0.113.45
  X-Forwarded-Proto: https (original protocol)
  
Your app code:
  clientIP := c.Get("X-Forwarded-For")
  // "203.0.113.45"
```

---

### Step 6: Request Lifecycle Inside Your Application

**Single request trace:**

```
1. TCP connection arrives at load balancer
   Time: 0ms

2. Load balancer performs TLS handshake
   Time: 0-50ms (cached session: <1ms)

3. Load balancer terminates TLS, gets plaintext HTTP
   GET /api/users/123 HTTP/1.1
   Time: 50ms

4. Load balancer forwards to reverse proxy
   (internal network, no TLS needed)
   Time: 51ms

5. Reverse proxy routes to your app instance
   Selects: Instance 1 (least connections)
   Time: 52ms

6. Your application receives connection
   Event loop wakes up
   Time: 52ms

7. Your framework parses request
   • Extract path: /api/users/123
   • Extract params: id=123
   • Find route handler
   Time: 53ms

8. Your handler executes
   func getUser(c *fiber.Ctx) error {
       id := c.Params("id")  // "123"
       user := db.Query("SELECT * FROM users WHERE id = ?", id)
       return c.JSON(user)
   }
   Time: 53-103ms (50ms database query)

9. Response sent back through chain
   Your app → Proxy → Load Balancer → Client
   Time: 103-120ms

Total: 120ms (from client's perspective)
```

**Your code only runs for ~50ms** (step 8). The rest is networking and infrastructure!

---

### Step 7: Concurrency and Scaling

**Horizontal scaling:**

```
One instance can't handle load:
  • 1 instance: 1,000 RPS
  • Need: 10,000 RPS
  
Solution: Scale to 10 instances
  • Load balancer distributes evenly
  • 10 × 1,000 = 10,000 RPS
  
This requires stateless design!
```

**Stateless vs Stateful:**

```
Bad (stateful):
  var sessions = {}  // In-memory session storage
  
  Problem: User logs in on Instance 1
           Next request routed to Instance 2
           Instance 2 doesn't have session → User appears logged out

Good (stateless):
  // Store sessions in Redis (external)
  session = redis.get(sessionId)
  
  Benefit: Any instance can handle any request
```

---

### Step 8: Health Checks and Failure Handling

**Platforms continuously probe your app:**

```
Every 10 seconds:
  Load Balancer → GET /health → Your app
  
  Expected response: 200 OK
  
  If 200 OK:
    • Continue sending traffic
  
  If 500 or timeout (3 consecutive):
    • Mark instance unhealthy
    • Stop sending traffic
    • Restart instance
```

**Your health endpoint:**

```go
app.Get("/health", func(c *fiber.Ctx) error {
    // Check database
    if !db.Ping() {
        return c.Status(503).SendString("Database unavailable")
    }
    
    // Check critical dependencies
    if !cache.Ping() {
        return c.Status(503).SendString("Cache unavailable")
    }
    
    return c.SendString("OK")
})
```

**Graceful shutdown:**

```go
// Trap SIGTERM (sent by platform when stopping instance)
c := make(chan os.Signal, 1)
signal.Notify(c, os.Interrupt, syscall.SIGTERM)

go func() {
    <-c
    fmt.Println("Gracefully shutting down...")
    
    // Stop accepting new requests
    app.Shutdown()
    
    // Wait for in-flight requests to complete (up to 30s)
    // Then exit
}()
```

**Why this matters:**

```
Without graceful shutdown:
  1. Platform sends SIGTERM
  2. Process immediately exits
  3. In-flight requests → error
  
With graceful shutdown:
  1. Platform sends SIGTERM
  2. Process stops accepting new requests
  3. Waits for active requests to finish
  4. Then exits cleanly
  5. No errors for users
```

---

### Step 9: Logging and Observability

**Your application logs:**

```go
log.Printf("User %d logged in", userId)
```

**Where do logs go?**

```
Your app writes to: stdout (standard output)
      ↓
Container/VM captures stdout
      ↓
Logs shipped to centralized system:
  • CloudWatch (AWS)
  • Stackdriver (Google Cloud)
  • Papertrail, Datadog, etc.
      ↓
You query logs via web UI/API
```

**Structured logging:**

```go
// Bad (unstructured)
log.Printf("User login: id=%d name=%s", userId, userName)

// Good (structured, JSON)
log.Info().
    Int("user_id", userId).
    Str("user_name", userName).
    Msg("User logged in")

Output: {"level":"info","user_id":123,"user_name":"alice","msg":"User logged in"}

Why? Can query: user_id=123 AND msg contains "login"
```

---

## Summary: The Web Server's Role

**Web servers are the critical interface between:**

```
Chaotic Internet          Clean Application
     ↓                           ↑
     └──[Web Server]─────────────┘
```

**What they handle for you:**

```
✓ TCP connection management
✓ TLS encryption/decryption
✓ HTTP protocol parsing
✓ Concurrent request handling
✓ Static file serving (optimized)
✓ Request routing
✓ Load balancing
✓ Error handling
✓ Security (rate limiting, filtering)
✓ Compression
✓ Caching
```

**What your application focuses on:**

```
✓ Business logic
✓ Database queries
✓ API integration
✓ Data processing
✓ Authentication/authorization
```

**Modern reality:**

Most backend frameworks (Go Fiber, Express, Spring Boot) include a **basic web server** built-in. For simple deployments, this is sufficient. For production at scale, add:

- **Reverse proxy** (Nginx, Caddy) for TLS, caching, static files
- **Load balancer** (AWS ALB, HAProxy) for distribution
- **CDN** (Cloudflare, CloudFront) for global edge caching

**The takeaway:**

Understanding web servers removes the "magic" from deployment. Your application is just a process listening on a port. The infrastructure around it—load balancers, proxies, health checks, logging—makes it production-ready.

Every time you visit a website, this entire stack runs invisibly in milliseconds. That's the power of modern web infrastructure.