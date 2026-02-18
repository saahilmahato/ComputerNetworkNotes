# 🧠 Remote Access Protocols — Deep Study Notes

---

# SSH (Secure Shell)

---

### 1) Concept Snapshot

**Definition:** SSH is a cryptographic network protocol that provides a secure, encrypted channel over an unsecured network for remote login, command execution, and tunneling.

**Purpose:** Replaces plaintext remote access tools (Telnet, rsh, rcp) with an encrypted, authenticated alternative. Protects against eavesdropping, man-in-the-middle attacks, and credential theft.

---

### 2) Mental Model

Think of SSH like a **sealed, tamper-evident diplomatic pouch** between two embassies. The courier (TCP) delivers it, but nobody along the route can read or modify the contents. Before the first pouch is sent, both embassies do a secret handshake to verify each other's identity and agree on a private lock only they know.

The channel is so trusted that you can stuff *other* protocols inside it (tunneling) — like shipping a second, smaller locked box inside the first one.

---

### 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7), but SSH itself manages its own transport security below the application
- **TCP/IP Layer:** Application layer, runs over TCP
- **Port:** 22
- **Sits above:** TCP (Layer 4)
- **Talks to:** The OS shell, SFTP subsystem, or any forwarded port
- **Used by:** scp, sftp, git over SSH, X11 forwarding, port forwarding

---

### 4) Mechanics (How It Actually Works)

**Phase 1 — TCP Connection**
Client opens a TCP connection to port 22 on the server.

**Phase 2 — Version Exchange**
Both sides exchange SSH protocol version strings in plaintext.
```
SSH-2.0-OpenSSH_8.9   (server)
SSH-2.0-OpenSSH_8.4   (client)
```

**Phase 3 — Key Exchange (KEX)**
- Both sides advertise supported algorithms (cipher, MAC, compression, KEX method)
- Typically use **Diffie-Hellman** or **ECDH** to derive a shared secret *without transmitting it*
- Server sends its **host key** (public key) — client checks it against `~/.ssh/known_hosts`
- A **session key** is derived; all future traffic is encrypted with this symmetric key

**Phase 4 — Authentication**
Three common methods, tried in order of config:
1. **Public key auth** — client proves possession of private key by signing a challenge with it
2. **Password auth** — password sent *inside the already-encrypted channel*
3. **GSSAPI / Kerberos** — enterprise/domain auth

**Phase 5 — Channel Open**
SSH multiplexes *channels* over one connection. Client requests a channel type:
- `session` → shell or command execution
- `direct-tcpip` → local port forward
- `x11` → X11 forwarding

```
Client ──TCP──► Server :22
         ▼
   [Version Exchange]
         ▼
   [KEX + Host Key Verify]
         ▼
   [Symmetric Encryption ON]
         ▼
   [User Authentication]
         ▼
   [Channel: shell / sftp / tunnel]
```

**Important Packets/Fields:**
- `SSH_MSG_KEXINIT` — algorithm negotiation
- `SSH_MSG_NEWKEYS` — signals switch to new keys
- `SSH_MSG_USERAUTH_REQUEST` — carries auth method and credentials
- `SSH_MSG_CHANNEL_DATA` — actual shell I/O

---

### 5) Key Structures & Components

**`~/.ssh/known_hosts`** — client-side database of server host keys. First connection stores the fingerprint; future connections verify it (TOFU — Trust On First Use model).

**`~/.ssh/authorized_keys`** — server-side file listing public keys that are allowed to authenticate.

**SSH Agent** — a background process that holds decrypted private keys in memory. Clients talk to it via a Unix socket so the key is never exposed directly.

**SSH Config (`~/.ssh/config`)** — per-host configuration: identity file, username, port, ProxyJump, etc.

**Subsystems** — SFTP is implemented as an SSH subsystem, not a separate protocol layer.

**ControlMaster / Multiplexing** — SSH can reuse one TCP connection for multiple sessions, dramatically reducing latency for repeated connections.

---

### 6) Performance & Tradeoffs

- **Encryption overhead** is negligible on modern hardware (AES-NI hardware acceleration)
- **First connection** is slow due to KEX; multiplexing (`ControlMaster`) amortizes this
- **Key exchange algorithms** vary in speed: ECDH (Curve25519) is faster and more secure than classic DH
- **Compression** (`-C` flag) helps on slow links but hurts on fast ones due to CPU overhead
- **Port forwarding / tunneling** adds latency since all traffic is encapsulated inside SSH's encryption + framing
- **Agent forwarding** is convenient but a security risk — a compromised intermediate host can hijack your agent socket

---

### 7) Failure Modes

| Failure | Cause | Recovery |
|---|---|---|
| `Host key verification failed` | Server's host key changed (could be MITM or server rebuilt) | Manually remove old entry from `known_hosts` |
| `Connection refused` | SSH daemon not running or firewall blocking port 22 | Check `sshd` status, firewall rules |
| `Permission denied (publickey)` | Wrong key, wrong permissions on `authorized_keys`, or auth method disabled | Check file permissions (`chmod 600`), verify key is in `authorized_keys` |
| `Too many authentication failures` | SSH client tries too many keys before the right one | Use `-i` to specify key explicitly |
| Broken pipe / timeout | Idle connection killed by firewall/NAT | Set `ServerAliveInterval` in ssh config |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` | Host key mismatch — possible MITM | Investigate carefully before proceeding |

---

### 8) Real-World Usage

- Remote server administration (the primary use case)
- Git over SSH (`git@github.com:user/repo.git`)
- SFTP/SCP for secure file transfer
- SSH tunneling to bypass firewalls or secure other protocols (e.g., tunneling MySQL)
- CI/CD pipeline authentication to servers and code repositories
- Jump hosts / bastion hosts — `ssh -J bastion internal-server`
- VPN-like setups using SSH tunnels (poor man's VPN)

---

### 10) Packet Walkthrough

```
1. Client  → TCP SYN                          → Server :22
2. Server  → TCP SYN-ACK                      → Client
3. Client  → TCP ACK                          → Server
4. Server  → "SSH-2.0-OpenSSH_8.9\r\n"        → Client  (plaintext)
5. Client  → "SSH-2.0-OpenSSH_8.4\r\n"        → Server  (plaintext)
6. Both    ↔ SSH_MSG_KEXINIT (algo lists)
7. Both    ↔ ECDH key exchange messages
8. Both    → SSH_MSG_NEWKEYS  ── encryption starts here ──
9. Client  → SSH_MSG_SERVICE_REQUEST ("ssh-userauth")
10. Client → SSH_MSG_USERAUTH_REQUEST (pubkey or password)
11. Server → SSH_MSG_USERAUTH_SUCCESS
12. Client → SSH_MSG_CHANNEL_OPEN ("session")
13. Server → SSH_MSG_CHANNEL_OPEN_CONFIRMATION
14. Client → SSH_MSG_CHANNEL_REQUEST ("shell")
15. Both   ↔ SSH_MSG_CHANNEL_DATA  (interactive shell I/O)
```

---

### 11) Common Interview / Exam Traps

- **"SSH encrypts the password"** — Partially true but misleading. Password auth sends the password *inside an already-encrypted tunnel*. Public key auth never sends the private key at all — it only sends a *signature*.
- **SSH is not a VPN** — it encrypts one session/tunnel, not all traffic system-wide.
- **Host key vs user key** — host keys authenticate the *server* to the *client*. User keys authenticate the *client* to the *server*. These are completely separate key pairs.
- **Agent forwarding is dangerous** — forwarding your agent to an untrusted server lets the server use your key on your behalf. Use `ProxyJump` instead.
- **`known_hosts` is TOFU** — it doesn't use a CA by default. If you accept a fake key on first connection, you've been compromised from the start. SSH *can* use CA-signed host keys, but most setups don't.
- **RSA vs Ed25519** — RSA is still common but Ed25519 is faster, smaller, and considered more secure for new deployments.

---

### 12) Retrieval Prompts

- What happens during the SSH key exchange, and why can't a passive observer derive the session key?
- Why is public key authentication more secure than password authentication?
- What is the difference between `ssh-agent` forwarding and `ProxyJump`?
- How does SSH multiplexing work and when would you use it?
- What does `known_hosts` protect against, and what is its weakness?
- How would you tunnel a local port to a remote database securely?

---

### 13) TL;DR Compression

- SSH = encrypted, authenticated remote shell over TCP port 22
- Uses Diffie-Hellman/ECDH for key exchange → symmetric session key → AES encryption
- Auth via public key (sign a challenge) or password (sent inside encrypted tunnel)
- Multiplexes channels: shell, SFTP, port forwards — all over one TCP connection
- Replaces Telnet/rsh; also powers git, SFTP, bastion hops, and tunneling

---
---

# Telnet

---

### 1) Concept Snapshot

**Definition:** Telnet is a plaintext, bidirectional, byte-oriented communication protocol for remote terminal access over TCP, standardized in RFC 854 (1983).

**Purpose:** Provided the first standardized way to remotely log into and control a machine over a network. Now considered insecure and largely replaced by SSH, but still used for debugging, legacy systems, and protocol testing.

---

### 2) Mental Model

Telnet is a **walkie-talkie conversation in a public square**. Everyone standing nearby can hear exactly what you say — your username, your password, every command, every response. There is no envelope, no encryption, no whisper. Whatever travels through the wire is human-readable by anyone with a packet sniffer on the path.

It's also the networking equivalent of a **raw pipe** — it just connects stdin/stdout of your terminal to stdin/stdout of the remote machine with almost no ceremony.

---

### 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7)
- **Port:** 23
- **Runs over:** TCP
- **Below it:** TCP → IP → Link layer
- **Related to:** NVT (Network Virtual Terminal) — Telnet's abstraction for cross-platform terminal behavior

---

### 4) Mechanics (How It Actually Works)

Telnet is built around the **Network Virtual Terminal (NVT)** concept — a lowest-common-denominator terminal that both sides understand, translating from local terminal formats.

**Option Negotiation** happens before data flows. Both sides send `WILL/WONT/DO/DONT` commands to negotiate features:
- `WILL ECHO` — "I will echo characters"
- `DO SUPPRESS-GO-AHEAD` — "please suppress the old half-duplex signaling"

**Data flow** after negotiation is essentially raw byte streaming — your keystrokes go to the remote, remote output comes back.

```
Client ──TCP──► Server :23
       ↓
  [Option Negotiation: WILL/WONT/DO/DONT]
       ↓
  [NVT established]
       ↓
  [Login prompt — PLAINTEXT]
       ↓
  [Password — PLAINTEXT over wire]
       ↓
  [Shell session — PLAINTEXT]
```

**Special bytes:** `0xFF` (IAC — Interpret As Command) is the escape byte that signals a Telnet command is following rather than data.

---

### 5) Key Structures & Components

- **IAC (0xFF)** — "Interpret As Command" byte, escapes data stream to signal control
- **Option codes** — e.g., `ECHO (1)`, `SUPPRESS-GO-AHEAD (3)`, `TERMINAL-TYPE (24)`, `NAWS (31)` (window size)
- **NVT** — defines canonical line endings (`\r\n`), special characters, and control sequences
- **Subnegotiation** — `SB ... SE` brackets used for complex option data (e.g., sending terminal type string)

---

### 6) Performance & Tradeoffs

- **Zero encryption overhead** — fast on slow hardware, but catastrophically insecure
- **Very low complexity** — easy to implement, easy to debug *other* protocols with (e.g., `telnet mail.server.com 25` to manually speak SMTP)
- **No compression, no integrity checking** — data arrives as-is
- **Still useful as a raw TCP testing tool** — `telnet host port` to test if a port is open and speak text protocols manually

---

### 7) Failure Modes

| Failure | Cause |
|---|---|
| Credentials captured | Any passive network sniffer sees everything |
| MITM trivial | No host authentication — attacker can impersonate server |
| `Connection refused` | Telnet daemon not running |
| Terminal garbling | NVT option mismatch between client and server |

There is no "recovery" from the security failures — the protocol is fundamentally broken by design for untrusted networks.

---

### 8) Real-World Usage

- **Legacy industrial/embedded systems** — many old network devices (routers, switches, PLCs) still expose Telnet
- **Protocol debugging** — engineers use `telnet host 25` to manually test SMTP, `telnet host 80` for HTTP, etc.
- **Local/isolated networks** — still used in air-gapped or fully trusted lab environments
- **MUD (Multi-User Dungeon) games** — the classic Telnet use case that survived into the internet era
- **Network equipment configuration** — older Cisco/HP switches defaulted to Telnet before SSH was added

---

### 11) Common Interview / Exam Traps

- **Telnet isn't "useless"** — it's a powerful raw TCP debugging tool even today. `telnet host port` is a valid substitute for `nc` in many environments.
- **Telnet sends the IAC byte (0xFF) literally** — if you're sending binary data and it contains 0xFF, Telnet will misinterpret it. This is a real source of corruption bugs in legacy systems.
- **Telnet is still on by default on some embedded/IoT systems** — a real security vulnerability in the wild (Mirai botnet exploited Telnet credentials extensively).

---

### 13) TL;DR Compression

- Telnet = raw, plaintext remote terminal over TCP port 23
- Everything — username, password, commands — travels unencrypted
- Uses NVT and IAC-based option negotiation for terminal capability exchange
- Replaced by SSH for any serious use; still alive for protocol debugging and legacy/embedded systems
- Mirai botnet proved Telnet's real-world danger at massive scale

---
---

# RDP (Remote Desktop Protocol)

---

### 1) Concept Snapshot

**Definition:** RDP is Microsoft's proprietary protocol for transmitting a graphical desktop session (display, keyboard, mouse, audio, clipboard, drive redirection) between a Windows client and server over TCP (and optionally UDP for performance).

**Purpose:** Allows full GUI-based remote control of a Windows machine, as opposed to SSH which provides only a terminal. Designed for both IT administration and thin-client computing scenarios.

---

### 2) Mental Model

RDP is like a **remote-controlled television** where the TV (server) generates the picture and the remote control (client) sends button presses. The cable between them carries compressed snapshots of what's on screen going one way, and keystrokes/mouse movements going the other way. You're not *running* programs on your local machine — you're just seeing their output and sending inputs.

Compare to SSH where the analogy would be: you send *commands* and get *text back*. RDP sends *pixels* and gets *input events* back.

---

### 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7)
- **Port:** TCP 3389 (primary), UDP 3389 (used for UDP transport in newer versions for lower latency)
- **Transport:** Originally TCP only; newer versions (8.0+) add UDP for audio/video
- **Built on top of:** T.128 (ITU graphics protocol), T.125 (MCS — Multipoint Communication Service), T.123 (transport stack)
- **Security:** Originally weak (RDP Security Layer); modern deployments use TLS + Network Level Authentication (NLA)

---

### 4) Mechanics (How It Actually Works)

**Connection Sequence:**

```
1. TCP connection to :3389
2. X.224 Connection Request/Confirm  (legacy transport layer)
3. MCS Connect Initial/Response      (multipoint channel setup)
4. TLS handshake (if NLA/TLS mode)
5. NLA credential exchange (CredSSP — credentials sent before desktop appears)
6. Capability exchange (graphics, input, virtual channels)
7. Licensing                         (client license validation)
8. Demand Active / Confirm Active    (server sends capabilities, client confirms)
9. Desktop bitmap/graphics stream begins flowing
10. Client sends input events; server sends display updates
```

**Virtual Channels** are the key architectural feature — RDP multiplexes many data streams over one connection:
- `rdpsnd` — audio redirection
- `rdpdr` — disk/printer/port redirection
- `cliprdr` — clipboard sync
- `drdynvc` — dynamic virtual channels for extensions (RemoteFX, etc.)

**Display encoding** has evolved significantly:
- Early RDP: raw bitmap updates, RLE compression
- RDP 6+: RemoteFX (H.264-based), adaptive graphics
- RDP 10: H.264/AVC codec, progressive rendering

---

### 5) Key Structures & Components

- **PDU (Protocol Data Unit)** — RDP's packet unit; many types: Data PDUs, Control PDUs, Input PDUs
- **MCS (Multipoint Communication Service)** — legacy layer underneath RDP that manages virtual channels
- **CredSSP / NLA** — Network Level Authentication; credentials validated *before* desktop session starts (blocks unauthenticated users from reaching the Windows login screen, reducing attack surface)
- **RemoteFX** — extension for accelerated graphics/video using H.264 encoding
- **RD Gateway** — allows RDP over HTTPS (port 443) through a proxy server, so you don't have to expose port 3389 directly
- **Session Broker** — load balances RDP sessions across a farm of servers (Remote Desktop Session Host)

---

### 6) Performance & Tradeoffs

- **Bandwidth hungry** compared to terminal access — graphics require significant compression work
- **UDP transport** (RDP 8+) dramatically improves latency for audio/video over high-latency or lossy links
- **Adaptive bitrate** — RDP degrades graphics quality on poor connections (reduces color depth, disables animations)
- **Thin client sweet spot** — RDP shines when client hardware is weak; all computation happens server-side
- **WAN performance** — much worse than SSH terminal access for the same task; always use the minimum features needed (disable wallpaper, animations, audio)
- **RemoteFX GPU acceleration** — server needs a GPU and proper licensing; excellent for CAD/graphics workloads

---

### 7) Failure Modes

| Failure | Cause | Recovery |
|---|---|---|
| `This computer can't connect to the remote computer` | Port 3389 blocked, RDP service not running | Check firewall, verify `TermService` is running |
| NLA auth failure | Wrong credentials, domain issues, expired password | Fix credentials, check domain connectivity |
| Certificate warning | Self-signed cert or cert mismatch | Deploy proper TLS cert, or accept and verify fingerprint |
| BlueKeep / DejaBlue | Pre-auth RCE vulnerabilities in RDP (CVE-2019-0708 etc.) | Patch, enable NLA, restrict port 3389 exposure |
| Session freezes on poor link | TCP congestion causes display to stall | Enable UDP transport, reduce display quality settings |
| Black screen on connect | GPU driver issue, or session in wrong state | Restart `TermService`, check Event Viewer |

**Security failures** deserve special mention — exposed RDP on port 3389 is one of the most attacked surfaces on the internet. Ransomware groups frequently enter via brute-forced RDP credentials.

---

### 8) Real-World Usage

- **Windows server administration** — the primary way sysadmins manage Windows Server
- **VDI (Virtual Desktop Infrastructure)** — users get a persistent virtual Windows desktop hosted in a data center
- **Remote work** — corporate desktops accessed from home over VPN + RDP
- **Help desk support** — IT support connecting to user machines
- **Gaming/GPU workloads** — cloud gaming and remote rendering workflows
- **Ransomware entry point** — exposed RDP is the #1 initial access vector for many ransomware attacks

---

### 9) Comparison

| Feature | SSH | Telnet | RDP |
|---|---|---|---|
| Encryption | Always (mandatory) | None | TLS (modern), weak (legacy) |
| Authentication | Public key / password | Password (plaintext) | Password / NLA / smartcard |
| Interface | Terminal (text) | Terminal (text) | Full GUI desktop |
| Port | 22 | 23 | 3389 |
| OS support | Universal | Universal | Windows-native (clients on all OS) |
| Bandwidth | Very low | Very low | High (graphics) |
| Primary use | Admin, automation, file transfer | Legacy/debugging | Windows desktop management |
| MFA support | Yes (PAM, etc.) | No | Yes (NLA + MFA) |
| Still secure today? | Yes | No | Yes, if patched + NLA + restricted |

---

### 10) Packet Walkthrough

```
1.  Client  → TCP SYN                      → Server :3389
2.  Server  → SYN-ACK
3.  Client  → X.224 Connection Request     (requests RDP)
4.  Server  → X.224 Connection Confirm
5.  Both    ↔ MCS Connect Initial/Response  (channel setup)
6.  Both    ↔ TLS ClientHello/ServerHello   (NLA mode: TLS starts here)
7.  Client  → CredSSP / NTLM/Kerberos auth  (NLA: credentials exchanged inside TLS)
8.  Server  → Auth success
9.  Both    ↔ Capability Sets               (client/server declare supported features)
10. Server  → Demand Active PDU             (server demands client to activate)
11. Client  → Confirm Active PDU
12. Server  → Synchronize, Control, Font PDUs (session initialization)
13. Server  → Bitmap/Graphics Update PDUs  ── desktop rendering begins ──
14. Client  → Input PDUs (keyboard/mouse events)
15. Both    ↔ Virtual Channel data (audio, clipboard, drive redirection)
```

---

### 11) Common Interview / Exam Traps

- **RDP is not just Windows-to-Windows** — Linux clients (Remmina, FreeRDP) and macOS clients (Microsoft Remote Desktop) exist. There are also open-source RDP servers like xrdp for Linux.
- **NLA is not optional for security** — without NLA, unauthenticated users can reach the Windows login screen and potentially exploit pre-auth vulnerabilities (e.g., BlueKeep).
- **Port 3389 should never be directly internet-exposed** — use RD Gateway (RDP over HTTPS) or VPN in front of it.
- **RDP uses UDP too** — many people think it's TCP-only. UDP 3389 was added for improved audio/video latency in RDP 8.0.
- **Session vs. Console** — by default RDP creates a *new session*. To connect to the physical console session (`/console` or `/admin` flag), you need to specifically request it. Important for troubleshooting.
- **BlueKeep was pre-authentication** — it exploited RDP before any login, meaning even machines with strong passwords were vulnerable if unpatched.

---

### 12) Retrieval Prompts

- What is Network Level Authentication and why does it improve security over basic RDP?
- Why was BlueKeep so dangerous, and what architectural property of RDP enabled it?
- How does RDP handle audio redirection, and what protocol layer manages it?
- What is the role of RD Gateway, and how does it differ from a VPN?
- Why would you choose RDP over SSH for Windows administration?
- How does RDP degrade gracefully on a poor network connection?

---

### 13) TL;DR Compression

- RDP = Microsoft's GUI remote desktop protocol over TCP/UDP port 3389
- Streams compressed screen updates server→client; sends keyboard/mouse client→server
- Virtual channels multiplex audio, clipboard, drive redirection over one connection
- Security requires NLA + TLS + patching — exposed RDP is a massive attack surface
- The go-to tool for Windows server/desktop remote management; VDI backbone

---
