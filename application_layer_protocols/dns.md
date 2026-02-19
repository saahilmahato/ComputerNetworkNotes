# 🌐 DNS

---

## 1) Concept Snapshot

**Definition:** DNS (Domain Name System) is a distributed, hierarchical naming system that translates human-readable domain names (e.g., `google.com`) into IP addresses (e.g., `142.250.80.46`) that machines use to route packets.

**Purpose:** Humans remember names; routers need numbers. DNS is the phonebook of the internet — it decouples the naming layer from the addressing layer, so IP addresses can change without users noticing.

---

## 2) Mental Model

**Real-world analogy:** You want to call "Mom" — you don't memorize her number, you look it up in your contacts. Your phone is the resolver, your contacts app is the local cache, and the carrier's directory is the authoritative server.

**Simplified story:**
> You type `github.com`. Your OS checks its memory (cache). No luck — it asks your ISP's resolver. The resolver doesn't know either, so it works its way down a hierarchy: asks a Root server → gets directed to `.com` nameserver → gets directed to GitHub's nameserver → finally gets the IP. Everyone caches the answer for next time.

**Visual intuition:**

```
                        Root (.)
                           │
              ┌────────────┼────────────┐
             .com         .org         .in
              │
           google.com
              │
         mail.google.com
```

The tree goes root → TLD → Second-level domain → Subdomain.

---

## 3) Layer Context

**OSI Layer:** Application Layer (Layer 7)
**Transport:** Runs over **UDP port 53** for standard queries (fast, low overhead). Falls back to **TCP port 53** for large responses (> 512 bytes, or always with DNSSEC/zone transfers).

**Who talks to it:**
- Above: Browsers, OS, any networked application that needs name resolution
- Below: UDP/TCP → IP → physical delivery

**Key point:** DNS is infrastructure — it's not a user-facing protocol, but nearly every user-facing protocol depends on it silently.

---

## 4) Mechanics (How It Actually Works)

### Resolution Flow — Step by Step

```
Application
    │  "What's the IP for github.com?"
    ▼
Stub Resolver (OS)
    │  Checks local cache → checks /etc/hosts
    │  Cache miss →
    ▼
Recursive Resolver (ISP / 8.8.8.8)
    │  Checks its own cache
    │  Cache miss →
    ▼
Root Nameserver (13 clusters worldwide, e.g., a.root-servers.net)
    │  "I don't know github.com, but .com is handled by Verisign"
    │  Returns NS records for .com TLD
    ▼
TLD Nameserver (.com)
    │  "I don't know github.com specifically, but here are GitHub's nameservers"
    │  Returns NS records for github.com
    ▼
Authoritative Nameserver (ns1.github.com)
    │  "Here is the actual A record: 140.82.112.4"
    ▼
Recursive Resolver caches it, returns answer to Stub Resolver
    ▼
Application gets IP → opens TCP connection
```

### Query Types

| Record Type | Meaning |
|---|---|
| **A** | Domain → IPv4 address |
| **AAAA** | Domain → IPv6 address |
| **CNAME** | Alias → canonical name |
| **MX** | Mail server for domain |
| **NS** | Authoritative nameservers for zone |
| **PTR** | IP → domain (reverse DNS) |
| **TXT** | Arbitrary text (used for SPF, DKIM, verification) |
| **SOA** | Start of Authority — zone metadata |

### DNS Packet Header Fields

```
┌──────────────────────────────────────┐
│  Transaction ID (16 bits)            │  Matches query to response
│  Flags (QR, Opcode, AA, TC, RD, RA) │  Query or response? Recursion?
│  QDCOUNT  │  ANCOUNT                 │  # of questions / answers
│  NSCOUNT  │  ARCOUNT                 │  # of authority / additional records
└──────────────────────────────────────┘
```

**Important flags:**
- **RD** (Recursion Desired) — client asks resolver to do the full lookup
- **RA** (Recursion Available) — server says it can do recursion
- **AA** (Authoritative Answer) — response came directly from authoritative server
- **TC** (Truncated) — response was cut off, retry over TCP

---

## 5) Key Structures & Components

**TTL (Time To Live):** Every DNS record has a TTL in seconds. Resolvers cache the record until TTL expires. Low TTL = faster propagation of changes, more DNS traffic. High TTL = less traffic, slower updates.

**Zone:** A zone is an administrative unit of the DNS namespace. A single organization manages its zone. The zone file contains all the records for that domain.

**Zone Transfer (AXFR):** Full copy of a zone from primary to secondary nameserver — uses TCP because it's large. A security risk if allowed from unauthorized IPs.

**Negative Caching:** If a domain doesn't exist (NXDOMAIN), that negative result is also cached — defined by the SOA record's minimum TTL.

---

## 6) Performance & Tradeoffs

**Latency:** Uncached resolution can involve 3+ round trips (root → TLD → authoritative). Each hop adds latency. First query to a domain is slow; subsequent ones hit cache and are near-instant.

**UDP vs TCP:** UDP is faster (no handshake) but limited to 512 bytes traditionally (4096 with EDNS0). TCP is reliable but slower — used only when necessary.

**TTL Tradeoff:** 
- Short TTL → rapid failover capability, but hammers DNS servers with traffic
- Long TTL → DNS server load is low, but IP changes propagate slowly (hurts incident response)

**Anycast routing:** The 13 root nameserver addresses are served by hundreds of machines worldwide using anycast — your query routes to the geographically nearest instance automatically.

---

## 7) Failure Modes

**DNS Cache Poisoning:** An attacker injects a forged DNS response into a resolver's cache, redirecting users to a malicious IP. Classic attack — exploits the fact that UDP has no authentication.
→ *Fix:* DNSSEC (cryptographic signing of records), source port randomization, 0x20 encoding trick.

**NXDOMAIN Hijacking:** Some ISPs intercept "domain not found" responses and redirect to ad pages instead of returning proper NXDOMAIN. Breaks applications that rely on NXDOMAIN for logic.

**Resolver Failure:** If your configured DNS resolver goes down, nothing works — even though the internet is fine. Users experience this as "the internet is broken."

**TTL Too Long During Incident:** You need to change your server IP urgently, but TTL is 24 hours. Everyone's caches have the old IP. You're stuck waiting — or accepting broken traffic.

**DDoS on DNS:** Amplification attacks use DNS — small query, large response, spoofed source IP. DNS response can be 70x larger than the request (amplification factor).

---

## 8) Real-World Usage

- **CDNs** use DNS to do load balancing and geo-routing — returning different IPs based on where you're querying from (GeoDNS).
- **Kubernetes** runs an internal CoreDNS server so pods can resolve each other by service name instead of IP.
- **Email authentication** lives entirely in DNS — SPF, DKIM, DMARC records are all TXT records.
- **Zero-downtime deployments** rely on careful TTL management before switching IPs.
- **DNS over HTTPS (DoH) / DNS over TLS (DoT)** — encrypts DNS queries so ISPs can't snoop on what sites you're visiting.

---

## 9) Comparison Section

| Feature | Iterative Resolution | Recursive Resolution |
|---|---|---|
| **Who does the work** | Client queries each server itself | Resolver does all lookups for client |
| **Used by** | Resolvers talking to root/TLD/auth servers | Stub resolvers (your OS) talking to ISP resolver |
| **Client complexity** | High | Low |
| **Typical path** | Resolver → Root → TLD → Auth | OS → ISP Resolver (one hop for client) |

| Feature | DNS | mDNS |
|---|---|---|
| **Scope** | Global internet | Local network only |
| **Port** | UDP 53 | UDP 5353 |
| **Use case** | Resolving github.com | Resolving `printer.local` |
| **Example** | Your ISP resolver | Apple Bonjour, Avahi |

---

## 10) Packet Walkthrough

> You open a browser and navigate to `api.example.com` for the first time.

1. Browser calls OS: *"Resolve api.example.com"*
2. OS checks `/etc/hosts` → not there
3. OS checks its DNS cache → cold cache, miss
4. OS sends **UDP query** to configured resolver (say, `8.8.8.8`), with **RD=1**
5. `8.8.8.8` checks its cache → miss
6. `8.8.8.8` queries a **Root nameserver**: *"Who handles `.com`?"*
   → Root returns NS records pointing to Verisign's TLD servers
7. `8.8.8.8` queries **`.com` TLD nameserver**: *"Who handles `example.com`?"*
   → TLD returns NS records: `ns1.example.com`, `ns2.example.com`
8. `8.8.8.8` queries **`ns1.example.com`**: *"What's the A record for `api.example.com`?"*
   → Returns: `A 93.184.216.34`, TTL `300`
9. `8.8.8.8` caches this record for 300 seconds, returns it to OS with **RA=1, AA=0**
10. OS caches it, hands IP to browser
11. Browser opens TCP connection to `93.184.216.34:443`

Total DNS time: ~50–150ms cold. ~0ms if cached.

---

## 11) Common Interview / Exam Traps

**"DNS uses TCP"** — Partially true. It primarily uses **UDP**, and only switches to TCP for large responses or zone transfers. Saying "DNS uses TCP" flat out is wrong.

**"There are 13 DNS root servers"** — There are 13 root server *addresses* (a through m), but each is an anycast cluster with hundreds of physical machines. Not 13 machines.

**"CNAME can point to an IP"** — No. A CNAME must point to another *domain name*, not an IP. That's what A/AAAA records are for. You also cannot have a CNAME at a zone apex (e.g., `example.com` itself) — use ALIAS or ANAME records for that.

**"Changing DNS is instant"** — No. Propagation depends on TTL of the old record. If TTL was 86400 (1 day), some resolvers will serve the old IP for up to 24 hours after you change it.

**"The recursive resolver is authoritative"** — No. The recursive resolver is a middleman. Only the nameserver that hosts the zone file is authoritative (AA=1 in response).

**"DNS is only for websites"** — DNS is foundational infrastructure. Email (MX), Kubernetes service discovery, CDN routing, microservice meshes — all of it relies on DNS.

---

## 12) Retrieval Prompts

- What happens when you type a URL and hit Enter — trace DNS specifically?
- Why does DNS use UDP instead of TCP by default?
- What is the difference between a recursive resolver and an authoritative nameserver?
- If you change a DNS record, why don't all users see it immediately?
- How does DNS cache poisoning work, and what prevents it?
- What's the difference between a CNAME and an A record? When can't you use CNAME?
- What happens when DNS goes down on your machine — why does "the whole internet stop working"?
- How does a CDN use DNS to route users to the nearest server?
- Why do root nameservers not get overwhelmed despite handling all unresolved queries?

---

## 13) TL;DR Compression

- DNS translates domain names → IP addresses via a **distributed, hierarchical, cached** system.
- Resolution goes: **Stub resolver → Recursive resolver → Root → TLD → Authoritative** (iterative at each step except the first).
- Uses **UDP/53** normally; **TCP/53** for large responses and zone transfers.
- **TTL controls caching** — the core tradeoff between freshness and performance.
- **DNSSEC** adds cryptographic signing to prevent cache poisoning; **DoH/DoT** encrypts queries for privacy.