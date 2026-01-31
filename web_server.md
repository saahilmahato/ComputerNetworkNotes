# Web Servers

## 1. What is a Web Server?

A **web server** is a software system (and often the machine running it) responsible for **accepting HTTP/HTTPS requests from clients and sending back HTTP responses**. The client is usually a web browser, but it can also be a mobile app, another server, a command-line tool (like `curl`), or any program that speaks HTTP.

Conceptually, a web server sits at the boundary between:

* **The Internet (untrusted, slow, unreliable)**
* **Application logic and data (trusted, structured, controlled)**

Its job is to translate network requests into meaningful application actions and then translate results back into network responses.

---

## 2. Web Server vs Application Server

Although often used interchangeably, these are **logically distinct roles**.

A **web server**:

* Speaks HTTP at a low level
* Handles TCP connections, TLS, request parsing
* Serves static content efficiently
* Routes requests to application code

An **application server**:

* Runs business logic
* Executes application code (Java, Go, Node, Python, etc.)
* Talks to databases, caches, message queues

In modern systems:

* **Nginx / Apache** → web server
* **Go Fiber, Express, Spring Boot** → application server

Often, both run together or are merged into a single binary, but the separation is still conceptually important.

---

## 3. Client–Server Interaction Model

### Request–Response Cycle

1. Client resolves domain name using DNS
2. Client opens a TCP connection to server IP (port 80 or 443)
3. (If HTTPS) TLS handshake occurs
4. Client sends HTTP request
5. Web server parses request
6. Server generates or fetches response
7. Server sends HTTP response
8. Connection is reused or closed

This cycle may happen **millions of times per second** on large systems.

---

## 4. Anatomy of an HTTP Request

An HTTP request consists of:

1. **Request line**

   * Method (GET, POST, PUT, DELETE, etc.)
   * Path (/api/users)
   * HTTP version

2. **Headers**

   * Metadata (Host, User-Agent, Content-Type, Authorization)

3. **Body (optional)**

   * Data sent to server (JSON, form data, binary)

The web server must:

* Parse all of this correctly
* Enforce size limits
* Reject malformed or malicious requests

---

## 5. Static vs Dynamic Content

### Static Content

Static content is **pre-existing data**:

* HTML files
* CSS
* JavaScript
* Images

Key properties:

* Same response for every request
* Can be cached aggressively
* Very fast to serve

Web servers are heavily optimized for static content using:

* Memory-mapped files
* Kernel zero-copy (`sendfile`)
* CDN integration

### Dynamic Content

Dynamic content is **generated per request**:

* Database queries
* API responses
* User-specific pages

Here, the web server acts as a **dispatcher**, forwarding requests to application logic.

---

## 6. Concurrency Model (Critical Section)

A web server must handle **many clients at the same time**.

### Common Models

1. **Process-per-request** (old Apache)

   * Simple but expensive
   * High memory usage

2. **Thread-per-request**

   * Better than processes
   * Still limited by thread overhead

3. **Event-driven (non-blocking I/O)**

   * Single or few threads
   * Uses epoll/kqueue
   * Extremely scalable

Modern high-performance servers (Nginx, Node.js, Go servers) use **event-driven architectures**.

---

## 7. Ports and Well-Known Defaults

* HTTP → port **80**
* HTTPS → port **443**

The port tells the OS which process should receive incoming packets. Multiple web servers can run on the same machine as long as they bind to different ports.

---

## 8. HTTPS and TLS (Why Encryption Matters)

HTTPS is HTTP layered over **TLS (Transport Layer Security)**.

TLS provides:

* **Confidentiality** – data is encrypted
* **Integrity** – data cannot be modified in transit
* **Authentication** – server identity is verified

Web servers:

* Store private keys securely
* Perform TLS handshakes
* Encrypt/decrypt data efficiently

TLS termination is often handled by the web server itself or a reverse proxy.

---

## 9. Reverse Proxies and Load Balancers

A **reverse proxy** is a web server that sits in front of other servers.

Responsibilities:

* TLS termination
* Load balancing
* Rate limiting
* Caching
* Security filtering

Example:

```
Client → Nginx → App Server 1
                 → App Server 2
```

This architecture improves:

* Scalability
* Fault tolerance
* Security isolation

---

## 10. Virtual Hosts (One Server, Many Websites)

A single web server can host **multiple domains** using virtual hosts.

The `Host` header in HTTP tells the server:

* Which domain the client is requesting
* Which configuration to use

This is how shared hosting works.

---

## 11. Error Handling and Status Codes

Web servers must generate correct HTTP status codes:

* 2xx → Success
* 3xx → Redirection
* 4xx → Client error
* 5xx → Server error

Correct status codes are critical for:

* Browser behavior
* Caching correctness
* API clients

---

## 12. Performance Metrics

Key metrics for web servers:

* **Latency** – time to first byte, total response time
* **Throughput** – requests per second
* **Concurrency** – simultaneous connections
* **Error rate** – failed requests

Optimizations focus on:

* Reducing context switches
* Minimizing memory copies
* Efficient connection reuse

---

## 13. Popular Web Servers

* **Apache** – mature, flexible, process-based origins
* **Nginx** – event-driven, high performance
* **Caddy** – automatic HTTPS
* **Go built-in net/http** – simple, powerful, production-grade

---

## 14. What Happens Under the Hood When You Deploy a Backend Service

This section connects theory to practice by explaining what actually happens when you deploy a backend built with frameworks like **Go Fiber** or **Spring Boot** on platforms such as **AWS, Render, Fly.io, or Railway**.

### 14.1 From Source Code to a Running Process

When you build your backend application, the output is simply an executable artifact:

* **Go Fiber** → a statically linked native binary
* **Spring Boot** → a fat JAR file executed by the JVM

At this stage, your application is just a file on disk. It is not yet a server and has no networking capabilities until it is executed.

The hosting platform provisions compute resources (a virtual machine, container, or sandboxed environment) running a Linux kernel. Your application is then started as a normal operating system process using commands such as:

* `./app-binary`
* `java -jar app.jar`

From the operating system’s perspective, your backend is just another process with a PID, memory limits, CPU quotas, and file descriptors.

---

### 14.2 Port Binding and the Role of the OS

Inside your code, you configure the server to listen on a port (for example, `8080`). When the application starts:

* It requests the OS to bind to a TCP port using system calls
* The kernel reserves that port
* Incoming TCP connections on that port are delivered to your process

Your application never handles raw network packets. TCP/IP, retransmissions, congestion control, and packet reassembly are entirely handled by the kernel. Your framework receives clean byte streams that already represent HTTP requests.

---

### 14.3 The Real Request Path in Production

In real deployments, your application is almost never directly exposed to the internet. The typical request path is:

```
Client → DNS → Load Balancer → Reverse Proxy → Backend Application
```

The reverse proxy (commonly Nginx or a managed equivalent) is responsible for:

* Terminating TLS (HTTPS)
* Handling very high concurrency
* Enforcing timeouts, rate limits, and size limits
* Forwarding requests to backend instances over internal networks

Because of this setup, your application often sees requests coming from internal IP addresses rather than the original client. This is why headers such as `X-Forwarded-For` and `X-Forwarded-Proto` exist.

---

### 14.4 Request Lifecycle Inside the Server

For a single HTTP request:

1. A TCP connection is accepted by the load balancer
2. TLS handshake is performed (for HTTPS)
3. HTTP request is parsed by the reverse proxy
4. The request is forwarded to a backend instance
5. Your application framework parses headers and body
6. Application logic executes
7. A response is generated and sent back through the same chain

Your handler code executes only a small portion of this entire lifecycle, but it relies on all the layers beneath it.

---

### 14.5 Concurrency and Scaling

Modern web servers are designed to handle thousands to millions of concurrent connections. Scaling is achieved by:

* Running multiple instances of your application
* Placing them behind a load balancer
* Distributing requests statelessly

This architecture requires backend services to be stateless. Any shared state (sessions, caches, queues) must live in external systems such as Redis or databases.

---

### 14.6 Crashes, Restarts, and Health Checks

If your backend process crashes:

* The platform detects the process exit
* The instance is restarted automatically
* Health-check endpoints are probed before routing traffic again

This is why production systems rely on:

* `/health` or `/ready` endpoints
* Fast startup times
* Graceful shutdown handling

---

### 14.7 Logging and Observability

When your application writes logs to standard output:

* The container or VM captures the output
* Logs are shipped to centralized logging systems
* Metrics and traces are collected separately

Your application does not manage log storage directly. It simply emits structured information that the platform collects.

---

## 15. Why Web Servers Matter (Big Picture)

Web servers form the critical boundary between unreliable networks and reliable application logic. They:

* Absorb network complexity
* Enforce security and performance policies
* Enable horizontal scaling

Understanding what happens beneath your application code removes the illusion of magic from deployment and allows engineers to design systems that are scalable, resilient, and predictable.
