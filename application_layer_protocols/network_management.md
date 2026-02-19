# 🧠 Computer Networks

---

# 📌 DHCP — Dynamic Host Configuration Protocol

---

## 1) Concept Snapshot

**Definition:** DHCP is an application-layer protocol that automatically assigns IP addresses and network configuration parameters (subnet mask, default gateway, DNS server) to hosts on a network, eliminating the need for manual configuration.

**Purpose:** Without DHCP, every device joining a network would need a manually configured static IP. DHCP automates this at scale — from home routers serving 5 devices to enterprise networks serving thousands.

---

## 2) Mental Model

Imagine a hotel. When you check in, the front desk assigns you a room (IP address) from available inventory. Your room is yours for your stay (lease duration). When you check out (lease expires or you leave), the room goes back to the pool for the next guest. If you want the same room again, you can request it — and if it's free, you get it.

The hotel ledger = DHCP lease table. The front desk = DHCP server. You = DHCP client.

```
New guest arrives → asks front desk → desk assigns room → guest uses room → guest leaves → room recycled
New device joins → DISCOVER → OFFER → REQUEST → ACK → uses IP → lease expires → IP recycled
```

---

## 3) Layer Context

| Layer | Detail |
|-------|--------|
| OSI Layer | Application (Layer 7) |
| Transport | UDP — port 67 (server), port 68 (client) |
| Network | Runs *before* the client has an IP, so initial messages are broadcast |
| Below it | IP/UDP carry the frames; Ethernet broadcasts at Layer 2 |

Why UDP and not TCP? Because the client has no IP yet — it can't establish a TCP connection. UDP broadcasts work without a pre-assigned address.

---

## 4) Mechanics — DORA Process

The four-step DHCP exchange is called **DORA**:

```
Client                          Server
  |                               |
  |---[DHCPDISCOVER]------------->|  broadcast (src: 0.0.0.0, dst: 255.255.255.255)
  |                               |
  |<--[DHCPOFFER]--------------   |  server offers IP + config
  |                               |
  |---[DHCPREQUEST]-------------->|  client formally requests the offered IP
  |                               |
  |<--[DHCPACK]----------------   |  server confirms, lease begins
  |                               |
```

**Step-by-step:**

**DISCOVER** — Client has no IP. It broadcasts onto the network: "Is there a DHCP server? I need an address." Source IP is 0.0.0.0, destination is 255.255.255.255.

**OFFER** — DHCP server receives the broadcast, picks an available IP from its pool, and responds with an offer containing: proposed IP, subnet mask, lease time, gateway, DNS servers. If multiple DHCP servers exist, the client may receive multiple offers.

**REQUEST** — Client selects one offer (typically the first received) and broadcasts a REQUEST. Broadcasting (not unicasting) is important — it tells *all* servers which offer was accepted, so unchosen servers can reclaim their tentative reservations.

**ACK** — The chosen server confirms with an ACK, committing the lease. Client now configures its interface. If the server can no longer honor the offer (IP was taken), it sends a **NACK**, and DORA restarts.

**Lease Renewal:** At 50% of lease time, client unicasts a REQUEST directly to the server that gave it the lease, attempting renewal. At 87.5%, if no response, it broadcasts to any DHCP server. At 100%, the lease expires and the IP is released.

---

## 5) Key Structures & Components

**DHCP Message Fields (key ones):**

| Field | Size | Purpose |
|-------|------|---------|
| `op` | 1 byte | 1 = request (client→server), 2 = reply |
| `xid` | 4 bytes | Transaction ID — matches requests to replies |
| `ciaddr` | 4 bytes | Client IP (if it already has one, for renewal) |
| `yiaddr` | 4 bytes | "Your" IP — the IP being offered/assigned |
| `siaddr` | 4 bytes | Server IP |
| `chaddr` | 16 bytes | Client hardware (MAC) address |
| Options | Variable | Subnet mask, router, DNS, lease time, message type |

**DHCP Server State:**
- IP address pool (available vs leased)
- Lease table: MAC → IP → expiry timestamp
- Reservations: MAC → static IP mapping (DHCP static assignment)

**DHCP Relay Agent:** In large networks, each subnet doesn't need its own DHCP server. A relay agent (usually the router) intercepts broadcasts and forwards them as unicasts to a central DHCP server, adding the client's subnet info in the `giaddr` field so the server can offer the right IP range.

---

## 6) Performance & Tradeoffs

**Lease Duration:** Short leases = IP pool recycles faster, handles highly transient environments (coffee shops), but generates more DHCP traffic and server load. Long leases = less traffic, but IPs get "stuck" on departed devices, wasting pool space.

**Single point of failure:** If the DHCP server goes down, new devices can't join and expiring leases can't renew. Mitigation: DHCP failover pairs (primary + secondary), or short enough leases that clients fail gracefully.

**Broadcast overhead:** DISCOVER and REQUEST are broadcasts — every device on the subnet receives them. In large flat networks, this is wasteful. VLANs and relay agents contain this.

---

## 7) Failure Modes

**DHCP Server Down:**
- New devices can't obtain IPs
- Devices with unexpired leases keep working until expiry
- After expiry: Windows defaults to APIPA (169.254.x.x), Linux may drop the interface

**IP Pool Exhaustion:**
- Server has no free IPs — new clients get no response or a NACK
- Common in environments with many transient devices and long lease times
- Attack vector: DHCP starvation — attacker sends thousands of DISCOVER messages with spoofed MACs, exhausting the pool

**Rogue DHCP Server:**
- An unauthorized DHCP server on the network can hand out wrong gateway/DNS, enabling man-in-the-middle attacks
- Mitigation: **DHCP Snooping** — a switch feature that trusts DHCP responses only from designated ports

**IP Conflict:**
- Rare, but if a server offers an IP that was manually assigned elsewhere, both devices conflict
- DHCP servers can send an ARP probe before offering an IP (checking if it's in use)
- Clients can also send a gratuitous ARP after receiving ACK to detect conflicts

---

## 8) Real-World Usage

- Every home router runs a DHCP server for connected devices
- ISPs run DHCP servers to assign IPs to customer modems/routers
- Enterprise environments use Windows DHCP Server or ISC DHCP (Linux) with failover pairs
- Cloud environments (AWS, GCP) use internal DHCP to assign private IPs to VMs at boot
- Wi-Fi hotspots use very short leases (1–2 hours) to recycle IPs from departed users

---

## 9) Comparison

| Feature | DHCP | Static IP | APIPA |
|---------|------|-----------|-------|
| Configuration | Automatic | Manual | Automatic (fallback) |
| Scope | Any network | Any network | Link-local only (169.254.x.x) |
| Internet access | Yes | Yes | No |
| Use case | General hosts | Servers, printers | No DHCP fallback |
| Persistence | Lease-based | Permanent | Until DHCP found |

---

## 10) Packet Walkthrough

> Laptop joins a Wi-Fi network for the first time.

1. Laptop's NIC sends a **DHCPDISCOVER** from `0.0.0.0:68` to `255.255.255.255:67`. The frame contains the laptop's MAC in `chaddr` and a random `xid = 0xA1B2C3D4`.

2. The home router's DHCP server receives it, picks `192.168.1.105` from its pool, and sends a **DHCPOFFER** back to the broadcast address (client has no IP yet) with `yiaddr = 192.168.1.105`, lease = 24h, gateway = `192.168.1.1`, DNS = `8.8.8.8`. Same `xid`.

3. Laptop sends **DHCPREQUEST** broadcast: "I'm accepting the offer from server `192.168.1.1` for IP `192.168.1.105`." Broadcast so other servers see their offers were declined.

4. Router sends **DHCPACK**: "Confirmed. `192.168.1.105` is yours for 24 hours." Laptop configures its interface, sets default route to `192.168.1.1`, DNS to `8.8.8.8`.

5. 12 hours later, laptop unicasts a **DHCPREQUEST** renewal to `192.168.1.1`. Router responds with **DHCPACK** resetting the 24-hour clock.

---

## 11) Common Interview / Exam Traps

**"DHCP uses TCP"** — Wrong. It uses UDP. The client can't use TCP because it has no IP yet and can't complete a handshake.

**"DHCPREQUEST is unicast"** — The initial REQUEST (step 3 of DORA) is a *broadcast*, not a unicast. Only renewal REQUESTs are unicast. This surprises many people.

**"DHCP gives a permanent IP"** — No. It's a *lease*. The IP is returned to the pool after expiry unless renewed.

**"One DHCP server per subnet is required"** — False. Relay agents (DHCP helpers) allow a single server to serve multiple subnets by forwarding broadcasts as unicasts.

**Confusing `yiaddr` with `ciaddr`** — `yiaddr` is the IP being *offered* by the server ("your address"). `ciaddr` is the IP the *client* already has (used in renewals, 0.0.0.0 in initial DORA).

---

## 12) Retrieval Prompts

- Why does DHCP use UDP and not TCP?
- Walk me through DORA — what happens at each step and *why*?
- Why is the DHCPREQUEST in step 3 a broadcast rather than a unicast to the server?
- What happens to a client when its DHCP server goes down and the lease expires?
- What is a DHCP relay agent and when is it needed?
- How does DHCP starvation work? How is it mitigated?
- What's the difference between a DHCP reservation and a static IP?
- At what % of lease time does renewal begin, and what happens if it fails?

---

## 13) TL;DR

- DHCP automatically assigns IPs via the **DORA** exchange: Discover → Offer → Request → ACK
- Uses **UDP** (ports 67/68) because clients have no IP yet at the start
- IPs are **leased**, not permanent — renewal happens at 50% of lease time
- **Relay agents** allow one DHCP server to serve many subnets
- Key attacks: **DHCP starvation** (pool exhaustion) and **rogue DHCP servers** — mitigated by DHCP Snooping

---
---

# 📌 SNMP — Simple Network Management Protocol

---

## 1) Concept Snapshot

**Definition:** SNMP is an application-layer protocol used to monitor and manage network devices (routers, switches, servers, printers). It defines a standardized way to read and write device configuration/status variables organized in a hierarchical database called the **MIB**.

**Purpose:** Network administrators need a unified way to query device health, receive alerts about failures, and configure devices remotely — across thousands of heterogeneous devices from different vendors. SNMP provides that common language.

---

## 2) Mental Model

Think of a hospital with a central nurses' station monitoring every room. Each room has sensors (heart rate, temperature) that report to a shared chart system.

- **NMS (Network Management System)** = nurses' station
- **SNMP Agent** = sensor/monitor in each room (runs on each device)
- **MIB** = the standardized chart format (same fields, same meaning, across all rooms)
- **OID** = the specific chart entry (e.g., "heart rate" = a specific numbered field)
- **Trap** = a patient pressing the emergency button (unsolicited alert)

The manager normally *polls* (asks rooms for readings). But if a room has an emergency, it *pushes* an alert without being asked.

---

## 3) Layer Context

| Layer | Detail |
|-------|--------|
| OSI Layer | Application (Layer 7) |
| Transport | UDP — port 161 (agent, for queries), port 162 (manager, for traps) |
| Runs on | Every managed network device — routers, switches, servers, UPS, printers |

Why UDP? SNMP is designed for monitoring a potentially degraded network. Using TCP would be ironic — if the network is failing, you need a lightweight protocol to still get *some* data out. UDP's fire-and-forget approach is acceptable for polling.

---

## 4) Mechanics — How SNMP Works

**Three versions exist: SNMPv1, SNMPv2c, SNMPv3** (increasingly secure — more in section 9)

**Core Operations:**

| Operation | Direction | Purpose |
|-----------|-----------|---------|
| `GET` | Manager → Agent | Read a single OID value |
| `GET-NEXT` | Manager → Agent | Walk the MIB tree (read next OID) |
| `GET-BULK` | Manager → Agent | Retrieve large amounts of data efficiently (v2c+) |
| `SET` | Manager → Agent | Write/change a value on the device |
| `TRAP` | Agent → Manager | Unsolicited alert from device |
| `INFORM` | Agent → Manager | Acknowledged TRAP (v2c+) |
| `RESPONSE` | Agent → Manager | Reply to GET/SET |

**Polling Flow:**
```
NMS                           Agent (e.g., router)
 |                                 |
 |---[GET: OID 1.3.6.1.2.1.1.1]-->|   "What is your sysDescr?"
 |                                 |
 |<--[RESPONSE: "Cisco IOS 15.x"]--|
 |                                 |
```

**Trap Flow (unsolicited):**
```
Agent                           NMS
 |                               |
 |---[TRAP: ifDown, port Gi0/1]->|   "Interface went down!"
 |                               |  (no prior request)
```

---

## 5) Key Structures & Components

**MIB — Management Information Base:**
A virtual database defining what variables an agent exposes. It's not an actual database file — it's a schema/definition. The real data lives on the device itself.

MIB is organized as a tree:
```
root
└── iso (1)
    └── org (3)
        └── dod (6)
            └── internet (1)
                └── mgmt (2)
                    └── mib-2 (1)
                        ├── system (1)      ← sysDescr, sysUpTime...
                        ├── interfaces (2)  ← ifTable, ifSpeed, ifInOctets...
                        ├── ip (4)
                        └── tcp (6)
```

**OID — Object Identifier:**
A dotted numeric string identifying a specific variable in the MIB tree.

Example: `1.3.6.1.2.1.1.3.0` = `sysUpTime` — how long the device has been running.
Example: `1.3.6.1.2.1.2.2.1.10.1` = bytes received on interface 1.

**Community String (v1/v2c):**
A plaintext password included in every SNMP message. `public` = read-only (convention). `private` = read-write (convention). These are transmitted in cleartext — a major security flaw.

**SNMPv3 Security:**
Replaces community strings with actual authentication (HMAC-MD5 or HMAC-SHA) and optionally encryption (DES or AES). Introduces the concept of **security levels:**
- `noAuthNoPriv` — no authentication, no encryption
- `authNoPriv` — authenticated, not encrypted
- `authPriv` — authenticated + encrypted

---

## 6) Performance & Tradeoffs

**Polling vs Traps:** Pure polling generates predictable but potentially high traffic. Relying only on traps means silence ≠ healthy (the trap itself could be lost over UDP). Best practice: use both — polling for baseline, traps for immediate alerting.

**UDP for monitoring:** Acceptable because: (a) individual lost polls are tolerable, (b) you don't want your monitoring protocol to contribute to congestion, (c) stateless queries scale better. Downside: lost traps aren't retransmitted unless you use INFORMs (acknowledged traps in v2c+).

**MIB Walking:** Repeatedly calling GET-NEXT to traverse the entire MIB tree is expensive. GET-BULK (v2c+) reduces round trips significantly for large polls.

**Scalability:** A single NMS polling thousands of devices every 60 seconds creates significant query volume. Distributed collectors, polling intervals tuning, and selective OID polling are common optimizations.

---

## 7) Failure Modes

**Agent unreachable:** Poll request is sent, no response. NMS must implement its own timeout/retry since UDP is connectionless. Typically: 3 retries → mark as down → alert.

**Trap loss:** Traps are UDP — fire and forget. A trap sent during a network outage may never arrive. INFORMs (v2c+) add an acknowledgment mechanism to address this.

**Community string compromise:** In v1/v2c, community strings travel in cleartext. A captured packet reveals the string, giving an attacker read (or write) access to all managed devices. This is why SNMPv3 exists and v1/v2c should be retired.

**SET abuse:** SNMP SET allows remote configuration changes. With weak community strings, an attacker could reconfigure routing tables, bring down interfaces, or change device configs. Should be restricted to trusted management IP ranges.

**MIB mismatch:** A device exposes a vendor-specific MIB that the NMS doesn't have loaded — the OIDs are unresolvable. The data exists but is uninterpretable without the right MIB file.

---

## 8) Real-World Usage

- Network operations centers (NOCs) use NMS platforms like **Zabbix, Nagios, PRTG, SolarWinds, Cacti** — all built on SNMP polling
- ISPs monitor router interface utilization, error rates, and BGP session state via SNMP
- Data centers poll server power draw, temperature, and fan speed from PDUs and IPMI via SNMP
- **MRTG (Multi Router Traffic Grapher)** pioneered bandwidth graphs using SNMP `ifInOctets`/`ifOutOctets` polls
- Cloud providers expose SNMP on network appliances but increasingly supplement with streaming telemetry (gRPC/gNMI) for higher frequency data

---

## 9) Comparison — SNMP Versions

| Feature | SNMPv1 | SNMPv2c | SNMPv3 |
|---------|--------|---------|--------|
| Security | Community string (cleartext) | Community string (cleartext) | Auth + Encryption |
| Bulk retrieval | No | GET-BULK | GET-BULK |
| 64-bit counters | No | Yes | Yes |
| INFORM (ACK traps) | No | Yes | Yes |
| Error handling | Basic | Improved | Improved |
| Use today | Legacy only | Common but insecure | Recommended |

---

## 10) Packet Walkthrough

> NMS polls a router for its uptime and gets a trap when an interface goes down.

**Poll:**
1. NMS sends UDP packet to router:67 → `GET` request for OID `1.3.6.1.2.1.1.3.0` (sysUpTime), community = `public`
2. Router's SNMP agent looks up the value in its local MIB data
3. Agent sends UDP `RESPONSE` back to NMS with value: `sysUpTime = 4823600` (in hundredths of a second ≈ 13.4 hours)
4. NMS logs the value, graphs it, compares to previous poll

**Trap:**
1. Interface Gi0/1 on router goes down (cable pulled)
2. Agent immediately sends UDP packet to NMS port 162: `TRAP` with OID `ifDown`, `ifIndex = 3` (identifying the interface)
3. NMS receives it unsolicited, triggers alert, pages on-call engineer
4. Note: if this were an **INFORM**, the NMS would send an ACK; agent would retry if ACK not received

---

## 11) Common Interview / Exam Traps

**"SNMP is secure"** — Only SNMPv3 is. v1 and v2c send community strings in **cleartext** — Wireshark shows them instantly. Many networks still run v2c with `public`/`private` strings on internet-facing devices.

**"Traps are reliable"** — No. Traps use UDP. They're fire-and-forget. Use INFORMs if acknowledgment matters.

**"The MIB is a database on the device"** — The MIB is a *schema* — a definition of what variables exist and what they mean. The actual data lives in the device's memory/OS. The MIB file (`.mib`) is loaded by the NMS to interpret OIDs.

**"SNMP port 161 for everything"** — Port 161 is for queries to the agent. Traps go to port **162** on the NMS. Different directions, different ports.

**"GET-NEXT is inefficient"** — Yes, that's why GET-BULK was introduced in v2c. Knowing this distinction matters.

---

## 12) Retrieval Prompts

- What is the difference between a TRAP and an INFORM?
- Why does SNMP use UDP instead of TCP?
- Explain what a MIB is and what an OID is — and how they relate
- What's wrong with SNMPv2c from a security standpoint?
- How does an NMS know when a device is unreachable if SNMP is UDP?
- What does GET-BULK improve over GET-NEXT?
- Walk me through what happens when an interface on a router goes down — SNMP perspective
- What are the three security levels in SNMPv3?

---

## 13) TL;DR

- SNMP lets a **Network Management System** monitor and configure devices via a common protocol
- Operates over **UDP 161** (queries) and **UDP 162** (traps); uses **OIDs** to identify variables in a hierarchical **MIB**
- Core operations: **GET** (read), **SET** (write), **TRAP** (unsolicited alert), **INFORM** (acknowledged trap)
- **v1/v2c = insecure** (cleartext community strings); **v3 = use this** (auth + encryption)
- Traps are **unreliable (UDP)** — combine with polling; use INFORMs where acknowledgment is needed