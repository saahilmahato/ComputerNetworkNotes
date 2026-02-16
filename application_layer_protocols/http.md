# 🌐 HTTP Protocol

---

## **1) Concept Snapshot**

**Definition:**
HTTP (HyperText Transfer Protocol) is an application-layer, stateless, request-response protocol that defines how messages are formatted and transmitted between web clients (browsers) and servers, and how they should respond to various commands.

**Purpose:**
- Enables retrieval and manipulation of resources (HTML, images, videos, APIs) on the World Wide Web
- Provides standardized communication between web browsers and servers
- Foundation of data exchange on the internet

**Quick recall:** *Stateless, text-based protocol where clients request resources and servers respond with status codes and data.*

---

## **2) Mental Model**

**Real-world analogy:**
Think of HTTP like ordering at a restaurant:
- You (client) give your order (HTTP request) to the waiter
- Kitchen (server) prepares your food (processes request)
- Waiter brings back your dish with a note (HTTP response with status code)
- **Stateless part:** Each time you order, the waiter doesn't remember your previous orders unless you explicitly remind them (cookies/sessions)

**Visual intuition:**
```
[Browser] ---"GET /index.html"---> [Web Server]
          <---"200 OK + HTML"----
```

**Simplified story:**
You type a URL → Browser sends "Hey, can I get this page?" → Server says "Sure, here's the HTML (200 OK)" or "Nope, not found (404)" → Browser renders what it receives.

---

## **3) Layer Context**

**Which layer?**
- **OSI Model:** Layer 7 (Application Layer)
- **TCP/IP Model:** Application Layer

**Who talks to it (above)?**
- Web browsers (Chrome, Firefox)
- Mobile apps
- API clients (curl, Postman)
- Web crawlers/bots

**What's below it?**
- **Transport Layer:** TCP (typically port 80 for HTTP, 443 for HTTPS)
- HTTP relies on TCP's reliable, ordered byte stream
- HTTP/3 uses UDP + QUIC instead of TCP

---

## **4) Mechanics (How It Actually Works)**

### **Step-by-step flow:**

1. **DNS Resolution:** Browser resolves domain to IP address
2. **TCP Connection:** Three-way handshake (SYN, SYN-ACK, ACK)
3. **HTTP Request:** Client sends request message
4. **Server Processing:** Server interprets request, fetches resource
5. **HTTP Response:** Server sends status code + headers + body
6. **Connection Handling:** Connection closed or kept alive
7. **Rendering:** Browser parses and displays content

### **HTTP Request Structure:**
```
GET /index.html HTTP/1.1          ← Request line (Method, URI, Version)
Host: www.example.com              ← Headers
User-Agent: Mozilla/5.0
Accept: text/html
Connection: keep-alive
                                   ← Blank line
[Optional request body]            ← Body (for POST/PUT)
```

### **HTTP Response Structure:**
```
HTTP/1.1 200 OK                    ← Status line
Content-Type: text/html            ← Headers
Content-Length: 1024
Date: Mon, 16 Feb 2026 10:30:00 GMT
Connection: keep-alive
                                   ← Blank line
<html>...</html>                   ← Body
```

### **Important Fields:**

**Request Headers:**
- `Host`: Target server (mandatory in HTTP/1.1)
- `User-Agent`: Client software identifier
- `Accept`: Media types client can handle
- `Cookie`: Session/state data
- `Authorization`: Credentials

**Response Headers:**
- `Content-Type`: MIME type of body
- `Content-Length`: Body size in bytes
- `Set-Cookie`: Store state on client
- `Cache-Control`: Caching directives
- `Location`: Redirect URL (with 3xx codes)

---

## **5) Key Structures & Components**

### **HTTP Methods (Verbs):**
- **GET:** Retrieve resource (idempotent, safe)
- **POST:** Submit data, create resource (non-idempotent)
- **PUT:** Replace/create resource (idempotent)
- **DELETE:** Remove resource (idempotent)
- **HEAD:** Get headers only (like GET without body)
- **PATCH:** Partial modification
- **OPTIONS:** Describe communication options

### **Status Code Categories:**
- **1xx (Informational):** 100 Continue, 101 Switching Protocols
- **2xx (Success):** 200 OK, 201 Created, 204 No Content
- **3xx (Redirection):** 301 Moved Permanently, 302 Found, 304 Not Modified
- **4xx (Client Error):** 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
- **5xx (Server Error):** 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable

### **Connection Management:**
- **HTTP/1.0:** Connection closed after each request
- **HTTP/1.1:** Persistent connections (Connection: keep-alive)
- **Pipelining:** Multiple requests without waiting (rarely used due to issues)

---

## **6) Performance & Tradeoffs**

### **Latency vs Throughput:**
- **HTTP/1.1:** Head-of-line blocking (must wait for response before next request)
- **HTTP/2:** Multiplexing allows parallel requests on single connection
- **HTTP/3:** Uses QUIC (UDP) to avoid TCP head-of-line blocking

### **Reliability vs Speed:**
- Uses TCP (reliable but slower handshake overhead)
- HTTP/3 trades some ordering guarantees for speed

### **Stateless Design:**
- **Pro:** Scalable (servers don't store client state), simpler architecture
- **Con:** Every request must contain full context (cookies grow large), overhead

### **Text-based vs Binary:**
- **HTTP/1.x:** Text-based (human-readable, easier debugging)
- **HTTP/2:** Binary framing (efficient parsing, compression)

### **Scalability Issues:**
- Connection overhead in HTTP/1.1 (multiple TCP connections needed)
- Cookie bloat (sent with every request)
- Solved partially by HTTP/2 multiplexing and header compression (HPACK)

---

## **7) Failure Modes**

### **What breaks:**

1. **TCP Connection Failures:**
   - Network partition → Request never arrives
   - Recovery: Timeout, retry with exponential backoff

2. **DNS Failures:**
   - Can't resolve hostname → No connection
   - Recovery: Fallback DNS servers, local caching

3. **Server Overload:**
   - Too many requests → 503 Service Unavailable
   - Recovery: Load balancing, rate limiting, queueing

4. **Timeouts:**
   - Client timeout: No response within threshold
   - Server timeout: Slow backend processing
   - Recovery: Configurable timeout values, circuit breakers

5. **Redirect Loops:**
   - 301/302 chains that never end
   - Recovery: Browsers limit redirect count (typically 20)

6. **Malformed Requests/Responses:**
   - Invalid headers, wrong Content-Length
   - Recovery: 400 Bad Request, connection close

### **Recovery Mechanisms:**
- **Retries:** Idempotent methods (GET, PUT, DELETE) safe to retry
- **Caching:** Reduce server load, provide stale content on failure
- **Fallback Content:** 503 pages, degraded mode
- **Connection Pooling:** Reuse TCP connections

---

## **8) Real-World Usage**

### **Where you see it:**

1. **Web Browsing:**
   - Every website visit uses HTTP/HTTPS
   - Browser loads HTML, then makes additional HTTP requests for CSS, JS, images

2. **RESTful APIs:**
   - Mobile apps communicating with backends
   - Microservices talking to each other
   - Example: `GET /api/users/123` returns user data as JSON

3. **Webhooks:**
   - GitHub sends POST request when code is pushed
   - Payment processors notify your server of transactions

4. **Content Delivery:**
   - Video streaming (HLS uses HTTP)
   - Software updates, file downloads

5. **IoT Devices:**
   - Smart home devices reporting status
   - Sensors pushing data to cloud

### **Practical Examples:**
- **E-commerce:** Product page loads use GET, checkout uses POST
- **Social Media:** Infinite scroll uses AJAX (XMLHttpRequest/Fetch API over HTTP)
- **Authentication:** Login form POSTs credentials, server returns Set-Cookie

---

## **9) Comparison Section

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|---------|---------|
| **Transport** | TCP | TCP | UDP (QUIC) |
| **Format** | Text | Binary frames | Binary |
| **Multiplexing** | No (1 req/resp per connection) | Yes (multiple streams) | Yes |
| **Header Compression** | None | HPACK | QPACK |
| **Head-of-line Blocking** | Yes (app + TCP) | TCP layer only | No |
| **Connection Setup** | TCP 3-way handshake | Same + negotiation | Combined with TLS (0-RTT) |
| **Push** | No | Server Push | Server Push |
| **Browser Support** | Universal | ~98% | ~75% (growing) |

### **HTTP vs HTTPS:**

| Feature | HTTP | HTTPS |
|---------|------|-------|
| **Security** | Plaintext (visible to attackers) | Encrypted (TLS/SSL) |
| **Port** | 80 | 443 |
| **SEO** | Lower ranking | Higher ranking (Google prefers) |
| **Performance** | Slightly faster (no encryption) | Negligible difference (TLS 1.3 fast) |
| **Trust** | No verification | Certificate-based authentication |

---

## **10) Packet Walkthrough**

### **Scenario:** User types `http://example.com/page.html` in browser

```
1. DNS Query:
   Browser → "What's the IP for example.com?" → DNS Server
   DNS Server → "It's 93.184.216.34" → Browser

2. TCP Handshake:
   Browser → SYN → Server (93.184.216.34:80)
   Server → SYN-ACK → Browser
   Browser → ACK → Server
   [Connection established]

3. HTTP Request:
   Browser → Server:
   ┌─────────────────────────────────────┐
   │ GET /page.html HTTP/1.1             │
   │ Host: example.com                   │
   │ User-Agent: Mozilla/5.0             │
   │ Accept: text/html                   │
   │ Connection: keep-alive              │
   │                                     │
   └─────────────────────────────────────┘

4. Server Processing:
   - Parses request
   - Finds /page.html on disk
   - Prepares response

5. HTTP Response:
   Server → Browser:
   ┌─────────────────────────────────────┐
   │ HTTP/1.1 200 OK                     │
   │ Content-Type: text/html             │
   │ Content-Length: 512                 │
   │ Date: Mon, 16 Feb 2026 10:30:00 GMT │
   │                                     │
   │ <html><body>Hello!</body></html>    │
   └─────────────────────────────────────┘

6. Browser Rendering:
   - Parses HTML
   - Discovers <link> and <script> tags
   - Makes additional HTTP requests for CSS/JS/images
   - Renders page

7. Connection:
   - Kept alive (HTTP/1.1 persistent connection)
   - Can be reused for subsequent requests
   - Eventually times out or closed by FIN handshake
```

---

## **11) Common Interview / Exam Traps**

### **Misconceptions:**

❌ **"HTTP is always insecure"**
→ HTTP itself is unencrypted, but HTTPS (HTTP over TLS) is secure

❌ **"GET requests can't have a body"**
→ Technically allowed, but semantically undefined and often ignored by servers

❌ **"POST is for sending data, GET is for receiving"**
→ Both can send/receive; difference is GET is safe/idempotent (no side effects)

❌ **"PUT and POST are the same"**
→ PUT is idempotent (multiple identical requests = same result), POST is not

❌ **"HTTP/2 is always faster"**
→ Only beneficial with multiple resources; single-file download sees little benefit

❌ **"401 means Unauthorized"**
→ 401 = **Unauthenticated** (missing/invalid credentials), 403 = **Unauthorized** (no permission)

### **Frequently Confused Points:**

1. **Idempotency:**
   - GET, PUT, DELETE are idempotent
   - POST is NOT idempotent (creates new resource each time)

2. **Status Codes:**
   - 301 vs 302: Permanent vs Temporary redirect
   - 401 vs 403: Authentication vs Authorization
   - 502 vs 504: Bad Gateway vs Gateway Timeout

3. **Headers:**
   - `Content-Type` (in response): What format is the body?
   - `Accept` (in request): What formats can client handle?

4. **Stateless vs Connection:**
   - HTTP is stateless (no memory between requests)
   - But connection can be persistent (keep-alive)

---

## **12) Retrieval Prompts**

**Test yourself:**

1. Why is HTTP stateless? What are the benefits and drawbacks?
2. Walk through what happens when you type a URL and press Enter.
3. What's the difference between 301 and 302 redirects? When would you use each?
4. Why does HTTP use TCP instead of UDP? What changed in HTTP/3?
5. How does HTTP/2 solve head-of-line blocking compared to HTTP/1.1?
6. What headers are mandatory in HTTP/1.1 requests?
7. If a server returns 503, should the client retry? How about 404?
8. How do cookies work with HTTP's stateless design?
9. What's the security risk of using HTTP instead of HTTPS?
10. How does Connection: keep-alive improve performance?

**Scenario questions:**
- Your API returns 200 but the client sees an error. Where could the problem be?
- A page loads slowly. How can HTTP headers help diagnose (Cache-Control, Content-Length)?
- How would you implement API rate limiting using HTTP?

---

## **13) TL;DR Compression

**🎯 5-Bullet Summary:**

1. **Stateless request-response protocol** at Layer 7; clients request resources, servers respond with status codes + data
2. **Uses TCP (port 80)** for reliability; HTTP/3 uses UDP+QUIC for speed; HTTPS adds TLS encryption (port 443)
3. **Methods matter:** GET (safe, idempotent), POST (creates), PUT (idempotent update), DELETE (idempotent remove)
4. **Status codes:** 2xx success, 3xx redirect, 4xx client error, 5xx server error
5. **Evolution:** HTTP/1.1 (persistent connections) → HTTP/2 (multiplexing, binary) → HTTP/3 (QUIC, no HOL blocking)

**One-liner:** *HTTP is the stateless, text/binary protocol that powers the web by letting clients request resources from servers using methods (GET/POST) and receiving responses with status codes (200/404).*

---

**🔑 Key Takeaway for Interviews:**
Understand the *stateless nature* (why cookies/sessions exist), *method semantics* (idempotency), *status code meanings*, and *HTTP/2 vs HTTP/3 improvements* (multiplexing, QUIC). Be able to trace a full request-response cycle including TCP handshake.