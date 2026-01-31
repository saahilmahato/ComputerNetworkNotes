# Introduction to Computer Networks

---

## 1. What is a Computer Network?

### Formal Definition

A **computer network** is a collection of **autonomous computing devices** (called *nodes*) that are interconnected by **communication links** and governed by **protocols**, enabling them to exchange data and share resources.

### Intuitive Understanding

* Multiple independent machines
* Connected via physical or wireless media
* Communicate using well-defined rules

A key idea here is **autonomy**:

* Each computer has its own CPU, memory, OS
* Networking is about cooperation, not control

**Real-world perspective**
When you open a website:

* Your device talks to a router
* Router talks to ISP
* ISP talks to backbone networks
* Eventually reaches a remote server

All of this is *one logical conversation* across many physical networks.

---

## 2. Goals of Computer Networks

Computer networks were designed to solve **fundamental system-level problems**.

---

### 2.1 Resource Sharing

**What is shared?**

* Hardware (printers, storage, GPUs)
* Software (databases, services)
* Data (files, streams, APIs)

**Why it exists**
Instead of duplicating expensive resources, networks allow:

* Centralization of power
* Decentralized access

**Example**
A cloud server with 64 cores serves thousands of clients concurrently.

---

### 2.2 Communication & Collaboration

Networks enable:

* Asynchronous communication (email)
* Synchronous communication (calls, video)
* Real-time collaboration

**Key insight**
Networking is fundamentally about **state synchronization across machines**.

---

### 2.3 Scalability

Single-machine limits:

* CPU
* Memory
* Disk

Networks allow systems to scale by:

* Horizontal scaling (add machines)
* Geographic distribution

**Example**
Modern web apps scale by adding backend nodes behind a load balancer.

---

### 2.4 Reliability & Fault Tolerance

Assumption: **Failures are inevitable**

* Links fail
* Routers crash
* Servers go down

Networks provide:

* Redundant paths
* Replication
* Automatic rerouting

**Internet design principle**
The network survives even if parts of it fail.

---

## 3. Basic Components of a Network

---

### 3.1 Nodes (End Systems)

**Definition**
Devices that generate, process, or consume data.

**Examples**

* PCs and laptops
* Servers (web, database, cloud VMs)
* Smartphones
* Embedded and IoT devices

**Key distinction**

* End systems run *applications*
* Network core focuses on *packet forwarding*

---

### 3.2 Communication Links

Links carry data between nodes.

#### Wired Media

* Twisted pair (Ethernet)
* Coaxial cable
* Optical fiber

**Fiber advantages**

* Extremely high bandwidth
* Low signal loss
* Immune to electromagnetic interference

---

#### Wireless Media

* Wi-Fi (radio waves)
* Cellular networks
* Satellite

**Trade-off**
Wireless offers mobility at the cost of:

* Higher error rates
* Shared medium

---

### 3.3 Network Devices

#### Switch

* Operates mostly at Layer 2
* Connects devices within a LAN
* Forwards frames using MAC addresses

#### Router

* Operates at Layer 3
* Connects multiple networks
* Makes decisions using IP addresses

#### Modem

* Modulates/demodulates signals
* Enables digital data over analog media

**Modern reality**
Most consumer devices combine all of these into one box.

---

## 4. Network Classification

---

### 4.1 PAN (Personal Area Network)

* Range: a few meters
* Technologies: Bluetooth, NFC

**Purpose**
Low power, short-range connectivity.

---

### 4.2 LAN (Local Area Network)

* Small geographic area
* High speed, low latency

**Characteristics**

* Privately owned
* Ethernet or Wi-Fi

---

### 4.3 MAN (Metropolitan Area Network)

* City-scale
* Often ISP-managed

---

### 4.4 WAN (Wide Area Network)

* Country to global scale
* Uses long-distance links

**Key idea**
Latency becomes a dominant factor at this scale.

---

## 5. Internet as a Network of Networks

The Internet is:

* Decentralized
* Globally interconnected
* Based on TCP/IP

**No central controller**
Routing decisions are distributed across routers.

**Important concept**
Autonomous Systems (AS):

* Each ISP or large organization controls its own network
* ASes interconnect using standardized protocols

---

## 6. Data Communication Fundamentals

---

### 6.1 Data Representation

All data is ultimately represented as:

* Bits (0 and 1)

Higher-level abstractions (text, images, video) are encoded into bits before transmission.

---

### 6.2 Signals

#### Analog Signals

* Continuous values
* Susceptible to noise

#### Digital Signals

* Discrete levels
* Easier error detection and correction

Modern networks transmit **digital data**, even over analog media.

---

### 6.3 Performance Metrics

#### Bandwidth

* Maximum data rate
* Measured in bps

#### Latency

* Time for a packet to travel end-to-end

#### Jitter

* Variation in latency

#### Packet Loss

* Percentage of packets dropped

**Critical insight**
User experience is often limited by latency, not bandwidth.

---

## 7. Protocols and Standards

### What is a Protocol?

A protocol defines:

* Message format
* Message ordering
* Error handling
* State transitions

Protocols enable **interoperability** between heterogeneous systems.

---

### Examples

* HTTP → web communication
* TCP → reliable transport
* UDP → fast, connectionless transport
* IP → logical addressing and routing

---

## 8. Layered Network Architecture (Preview)

Networking complexity is managed using **layers**.

Each layer:

* Solves a specific problem
* Provides services to the layer above
* Hides implementation details below

Benefits:

* Modularity
* Easier debugging
* Independent evolution

Two dominant models:

* OSI (conceptual)
* TCP/IP (practical)

---

## 9. Summary

* Networks enable distributed computing
* Protocols are the language of networks
* Performance is multi-dimensional
* Layering is foundational to network design

---
