# IP Addresses

---

## 1. The Fundamental Problem IP Addresses Solve

At its core, the Internet is a **massive interconnection of independent networks**. Each network may contain anywhere from a handful to millions of devices. The central problem is this:

> How can a packet be delivered to the correct device **without every router knowing about every device**?

If routers tried to store a route for every individual computer, the Internet would collapse under its own weight. Memory, update traffic, and computation would explode. Therefore, the Internet is designed around a **hierarchical routing model**.

This hierarchy is enabled by **IP addresses**.

---

## 2. What an IP Address Really Is

An **IP address** is not just an identifier. It encodes **location information**.

More precisely, an IP address tells the network:

* Which **network** a device belongs to
* Which **host** within that network the device is

This is fundamentally different from a MAC address, which only answers *who* a device is, not *where* it is.

You should think of an IP address as a **routing coordinate**, not a name.

---

## 3. IPv4 Address Structure

An IPv4 address is **32 bits long**. Humans write it in dotted-decimal form only for convenience.

Example:

```
192.168.1.10
```

Internally, the network sees:

```
11000000 10101000 00000001 00001010
```

Every routing, filtering, and comparison operation happens on these **bits**, not on decimal numbers.

---

## 4. Why IP Addresses Must Be Split (Network vs Host)

Consider a router in the middle of the Internet. When a packet arrives, the router must answer a single question:

> Which direction should I forward this packet?

The router **does not care** which exact computer the packet is going to. It only cares about **which network** the packet should be forwarded toward.

This leads to a crucial design requirement:

> IP addresses must be divisible into a **network part** and a **host part**.

* The **network part** identifies a group of devices reachable via a common path
* The **host part** identifies a specific device inside that group

Without this division, routers would need per-host routes, which is not scalable.

---

## 5. Why Subnet Masks Exist

An IP address by itself is meaningless.

For example:

```
192.168.1.10
```

This address could belong to:

* A network with 2 devices
* A network with 254 devices
* A network with 65,000 devices

There is **no way to know** unless we also know **where the network part ends**.

This is the sole reason **subnet masks exist**.

> A subnet mask defines how many bits of the IP address represent the network.

Nothing more. Nothing less.

---

## 6. How Subnet Masks Actually Work (Binary Reality)

A subnet mask is a **32-bit binary number** with a very strict rule:

* All `1`s first (network bits)
* All `0`s after (host bits)

Example:

```
Subnet Mask: 255.255.255.0
Binary:      11111111.11111111.11111111.00000000
```

This mask says:

* First 24 bits → network
* Last 8 bits → host

---

## 7. The Bitwise AND Operation (The Heart of IP Routing)

Routers and hosts determine the network address using a **bitwise AND**.

Rule:

```
IP address AND Subnet mask = Network address
```

Example:

```
IP:   192.168.1.10 → 11000000.10101000.00000001.00001010
Mask: 255.255.255.0 → 11111111.11111111.11111111.00000000
------------------------------------------------------
Net:  192.168.1.0  → 11000000.10101000.00000001.00000000
```

This operation is not conceptual or optional. **Every IP stack and every router performs this exact computation**.

---

## 8. How Hosts Use Subnet Masks (Local vs Remote)

When a host wants to send a packet, it performs the following logic:

1. Compute its own network address using the subnet mask
2. Compute the destination’s network address using the same mask
3. Compare the two results

* If they are equal → destination is **local** → send directly using ARP
* If they differ → destination is **remote** → send to the default gateway

Thus, subnet masks directly control **traffic flow decisions**.

---

## 9. CIDR and Prefix Lengths

Writing subnet masks in dotted-decimal form is inconvenient and error-prone. CIDR notation solves this.

Example:

```
192.168.1.0/24
```

This means:

* First 24 bits are network bits
* Remaining 8 bits are host bits

CIDR notation directly reflects the **binary structure** of the address.

---

## 10. Subnetting: Dividing Networks Intentionally

Subnetting is the act of **borrowing bits from the host portion** to create smaller networks.

Why do this?

* Limit broadcast domains
* Improve security isolation
* Use IP address space efficiently

Subnetting does not create new addresses. It **reorganizes** existing ones.

---

## 11. Subnetting Mathematics

Let:

* Total bits = 32
* Network bits = n
* Host bits = h = 32 − n

Total addresses:

```
2^h
```

Usable host addresses:

```
2^h − 2
```

The two excluded addresses are:

* All host bits = 0 → network address
* All host bits = 1 → broadcast address

---

## 12. Fully Worked Numerical Example

### Problem

You are given the network:

```
192.168.10.0/26
```

Assign IP addresses to devices.

---

### Step 1: Understand the Prefix

`/26` means:

* Network bits = 26
* Host bits = 6

Total addresses:

```
2^6 = 64
```

Usable hosts:

```
64 − 2 = 62
```

---

### Step 2: Determine Address Range

Block size:

```
256 − 192 = 64
```

So the subnet contains:

* Network address: 192.168.10.0
* First usable:   192.168.10.1
* Last usable:    192.168.10.62
* Broadcast:      192.168.10.63

---

### Step 3: Assign Addresses

* Router (gateway): 192.168.10.1
* Server:           192.168.10.2
* Clients:          192.168.10.3 – 192.168.10.20

Subnet mask for all devices:

```
255.255.255.192
```

---

### Step 4: Routing Decision Check

Destination: `192.168.10.50`

AND with mask → `192.168.10.0`

Same network → direct delivery.

Destination: `8.8.8.8`

Different network → send to gateway.

---

## 13. Why This Design Scales

Because routers only store **network prefixes**, not host routes:

* Routing tables remain manageable
* Updates are localized
* The Internet can grow indefinitely

Subnet masks and CIDR are what make this possible.

---

## 14. Final Mental Model

Think of IP addressing as:

> A tree where each prefix narrows your location.

Subnet masks define how deep you are in that tree.

---

## 15. Key Takeaways

* IP addresses encode location
* Subnet masks define routing boundaries
* Bitwise AND is the core operation
* Subnetting is controlled hierarchy
* Without subnet masks, IP routing cannot exist

---
