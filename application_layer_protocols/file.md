# 🗂️ File Transfer Protocols — Deep Study Notes

---

# 1. FTP — File Transfer Protocol

---

## 1) Concept Snapshot

**Definition:** FTP is an application-layer protocol (RFC 959) for transferring files between a client and server over a TCP/IP network. It uses **two separate TCP connections** — one for control (commands) and one for data.

**Purpose:** Provides a standardized way to upload, download, list, rename, and delete files on a remote host. Designed in an era where simplicity and compatibility mattered more than security.

---

## 2) Mental Model

Imagine a telephone operator from the 1960s. You call them on one line (control channel) and say *"I want file X."* They then open a *second* physical line just to deliver the file to you (data channel), and once it's delivered, that second line hangs up — but you're still on the first line chatting away. That's FTP: one persistent conversation line, one on-demand delivery line.

---

## 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7)
- **Transport:** TCP — reliability is non-negotiable for file transfer
- **Talks to:** User agent (FTP client like FileZilla), or scripted FTP commands
- **Below it:** TCP sockets, IP routing, Ethernet frames

---

## 4) Mechanics (How It Actually Works)

FTP uses **two TCP connections**:

**Control Connection — Port 21**
Persistent. Used for sending commands (USER, PASS, LIST, RETR, STOR) and receiving response codes (200 OK, 530 Login incorrect, etc.). Stays open for the entire session.

**Data Connection — Port 20 (Active) or ephemeral (Passive)**
Temporary. Opened only when data needs to move (a file or directory listing). Closes after transfer completes.

**Active Mode (PORT):**
```
Client → Server:21   [Control: "PORT 192.168.1.5:4444, send data here"]
Server:20 → Client:4444  [Server initiates data connection back to client]
```
Problem: NAT/firewalls block inbound connections from the server.

**Passive Mode (PASV):**
```
Client → Server:21   [Control: "PASV — you pick a port"]
Server → Client:21   [Responds: "Connect to me on port 50234"]
Client → Server:50234  [Client initiates data connection]
```
Passive is NAT-friendly because *the client always initiates*.

**Step-by-step session:**
1. Client TCP connects to server port 21
2. Server sends `220 FTP Ready`
3. Client sends `USER alice` → Server: `331 Password required`
4. Client sends `PASS secret` → Server: `230 Login successful`
5. Client sends `PASV` → Server: `227 Entering Passive Mode (host, port)`
6. Client opens second TCP connection to that address/port
7. Client sends `RETR report.pdf` on control channel
8. Server streams file bytes over data connection
9. Data connection closes; control channel returns `226 Transfer Complete`
10. Client sends `QUIT` → `221 Goodbye` → control connection closes

---

## 5) Key Structures & Components

**FTP Response Codes (3-digit):**

| Code | Meaning |
|---|---|
| 150 | Opening data connection |
| 200 | Command OK |
| 220 | Service ready |
| 226 | Transfer complete |
| 230 | Logged in |
| 331 | Username OK, need password |
| 425 | Can't open data connection |
| 530 | Not logged in |
| 550 | File unavailable |

**Transfer Modes:**
- **ASCII mode** — for text files; translates line endings (CRLF ↔ LF). Can corrupt binary files if misused.
- **Binary (Image) mode** — raw byte stream; always use this for non-text files.

**FTP Commands:** USER, PASS, LIST, RETR (download), STOR (upload), DELE, MKD, CWD, PASV, PORT, QUIT

---

## 6) Performance & Tradeoffs

- Two connections per session = overhead, but clean separation of control and data
- No encryption — credentials and data fly in plaintext over the wire
- ASCII mode line-ending translation is a footgun that silently corrupts binary files
- Active mode breaks behind NAT; passive mode is almost always used in practice
- Large files transfer efficiently since TCP handles flow control and retransmission
- No built-in integrity check — a corrupted transfer won't be detected by FTP itself; you need to compare checksums separately

---

## 7) Failure Modes

**Authentication exposed:** Username and password are sent in cleartext. A passive network listener captures them trivially.

**Firewall/NAT issues with Active Mode:** The server tries to initiate a TCP connection back to the client — stateful firewalls drop this unless explicitly allowed.

**Interrupted transfers:** FTP has no built-in resume. If a 5 GB file transfer drops at 4.9 GB, you start over (unless the client implements REST command workarounds).

**Port scanning via FTP bounce attack:** An attacker sends PORT commands pointing to a third-party host, making the FTP server portscanning on their behalf. Most modern servers block this.

---

## 8) Real-World Usage

- Legacy enterprise systems (banks, insurance) still use FTP for batch file exchange
- Hosting providers offered FTP for decades to upload website files
- Automated scripts (cron jobs) pulling/pushing data files between systems
- Mostly **replaced by SFTP** in modern infrastructure, but the protocol lives on in legacy environments

---

## 9) Comparison (FTP Modes)

| Feature | Active Mode | Passive Mode |
|---|---|---|
| Data conn initiated by | Server | Client |
| Data source port | 20 | Ephemeral (server-chosen) |
| Works behind client NAT | ❌ Often blocked | ✅ Yes |
| Works behind server firewall | ✅ Easier | Needs port range open |
| Modern default | No | Yes |

---

## 10) Packet Walkthrough

```
[TCP 3-way handshake to port 21]
Server → Client: "220 FTP server ready"
Client → Server: "USER alice"
Server → Client: "331 Password required"
Client → Server: "PASS hunter2"
Server → Client: "230 User logged in"
Client → Server: "TYPE I"            ← Binary mode
Server → Client: "200 Type set to I"
Client → Server: "PASV"
Server → Client: "227 Entering Passive Mode (10,0,0,1,196,18)"
                                      ← port = 196×256+18 = 50194
[Client opens new TCP connection to 10.0.0.1:50194]
Client → Server: "RETR backup.tar.gz"
Server → Client: "150 Opening BINARY connection"
[File bytes stream over data connection]
[Data connection closes]
Server → Client: "226 Transfer complete"
Client → Server: "QUIT"
Server → Client: "221 Goodbye"
[Control connection closes]
```

---

## 11) Common Interview / Exam Traps

- **"FTP uses port 20 and 21"** — Port 21 is *always* the control port. Port 20 is the data port *only in active mode*. Passive mode uses a random high port.
- **Conflating FTP with FTPS vs SFTP** — FTPS adds TLS to FTP (it's still FTP underneath). SFTP is a completely different protocol running over SSH. They share a name but nothing else.
- **ASCII mode corruption** — Students forget that FTP ASCII mode modifies file bytes (line ending translation). Transferring a .jpg in ASCII mode produces a corrupted image.
- **"Passive is the client's passive"** — No. In passive mode, the *server* passively waits for the client to connect on the data port. The client is active in making the connection.

---

## 12) Retrieval Prompts

- Why does FTP need two connections? What problem does separating control and data solve?
- What breaks active mode FTP in modern networks and why does passive mode fix it?
- If your FTP download silently corrupted a zip file, what's the first thing you'd check?
- Why is anonymous FTP a security concern?
- How does the PORT command encode the IP address and port number?

---

## 13) TL;DR

- FTP = application-layer file transfer over TCP, two connections: port 21 (control, persistent) + data (temporary)
- Active mode: server initiates data connection — breaks behind NAT. Passive mode: client initiates — NAT-friendly, modern default
- Zero encryption — credentials and files sent in plaintext
- ASCII vs Binary transfer modes — always use Binary unless you know you need ASCII
- Still alive in legacy systems; largely replaced by SFTP in secure environments

---
---

# 2. SFTP — SSH File Transfer Protocol

---

## 1) Concept Snapshot

**Definition:** SFTP (SSH File Transfer Protocol) is a binary protocol for file access, transfer, and management, running as a subsystem *inside an SSH-2 session* (RFC draft, OpenSSH). It is not FTP with SSH bolted on — it is an entirely different protocol.

**Purpose:** Provides all FTP-like operations (upload, download, list, rename, delete, permissions) with full encryption, integrity verification, and authentication through SSH — solving every security problem that FTP ignores.

---

## 2) Mental Model

Imagine FTP is sending confidential documents through the regular mail in a transparent envelope — anyone who handles it can read it. SFTP is using an armored diplomatic pouch that only opens at the destination. The pouch (SSH tunnel) carries everything: your identity credentials, the commands, and the files themselves — all encrypted, all over a single connection.

---

## 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7), tunneled inside SSH (also Layer 7 over TCP Layer 4)
- **Transport:** Single TCP connection, port 22 (SSH port)
- **Talks to:** SSH daemon (sshd) on the server side; SFTP client on the user side
- **Below it:** SSH session → TCP → IP

---

## 4) Mechanics (How It Actually Works)

SFTP runs as a **subsystem of SSH-2**. There is no separate control/data split — everything flows through the one encrypted SSH tunnel.

**Connection establishment:**
1. Client TCP connects to server port 22
2. SSH handshake: key exchange (Diffie-Hellman), algorithm negotiation
3. Server authenticates itself via host key (client checks known_hosts)
4. Client authenticates: password OR public key (private key on client, public key on server in `~/.ssh/authorized_keys`)
5. Client requests SSH subsystem: `sftp`
6. SSH channel is established for SFTP; binary SFTP protocol begins

**SFTP Protocol Packets (binary, not human-readable text like FTP):**
- Client sends request packets: SSH_FXP_OPEN, SSH_FXP_READ, SSH_FXP_WRITE, SSH_FXP_STAT, etc.
- Server responds with SSH_FXP_DATA, SSH_FXP_STATUS, SSH_FXP_ATTRS
- Requests have a **request ID** — multiple requests can be in-flight simultaneously (pipelining)

**File download flow:**
```
Client → SSH_FXP_OPEN (filename, read flag)
Server → SSH_FXP_HANDLE (file handle = integer token)
Client → SSH_FXP_READ (handle, offset=0, length=32768)
Server → SSH_FXP_DATA (actual bytes)
Client → SSH_FXP_READ (handle, offset=32768, length=32768)
Server → SSH_FXP_DATA (next chunk)
... (continues until EOF)
Server → SSH_FXP_STATUS (EOF)
Client → SSH_FXP_CLOSE (handle)
Server → SSH_FXP_STATUS (OK)
```

---

## 5) Key Structures & Components

**Authentication Methods (via SSH):**
- **Password** — SSH encrypts it; not stored on server
- **Public Key** — Client signs a challenge with private key; server verifies with stored public key. No password transmitted ever.
- **Host keys** — Stored in `~/.ssh/known_hosts`; prevent MITM by binding server identity to a key fingerprint

**SFTP Packet Types:**

| Type | Direction | Purpose |
|---|---|---|
| SSH_FXP_INIT | Client → Server | Protocol version negotiation |
| SSH_FXP_OPEN | Client → Server | Open file, get handle |
| SSH_FXP_READ | Client → Server | Request bytes |
| SSH_FXP_WRITE | Client → Server | Upload bytes |
| SSH_FXP_STAT | Client → Server | Get file attributes |
| SSH_FXP_MKDIR | Client → Server | Create directory |
| SSH_FXP_STATUS | Server → Client | Result code |
| SSH_FXP_DATA | Server → Client | File bytes |

**Status Codes:** SSH_FX_OK (0), SSH_FX_EOF (1), SSH_FX_NO_SUCH_FILE (2), SSH_FX_PERMISSION_DENIED (3), SSH_FX_FAILURE (4)

---

## 6) Performance & Tradeoffs

- **Single TCP connection** — simpler than FTP, no firewall headaches, but SSH encryption overhead adds CPU cost (mitigated by hardware AES acceleration on modern CPUs)
- **Pipelining** — SFTP allows multiple outstanding requests; good clients (like WinSCP) pipeline reads for high throughput, poor clients send one request at a time and waste round-trip time
- **High-latency links** — pipelining is essential; without it, each 32KB chunk waits for a full RTT before the next is requested
- **Security cost** — every byte is encrypted and MAC'd; negligible on LAN, measurable on very high-throughput transfers over fast links
- **No compression at SFTP level** — SSH itself can compress (`-C` flag), but it rarely helps with already-compressed files

---

## 7) Failure Modes

**Host key mismatch:** If server host key changes (server reinstalled, MITM attack), client refuses to connect with a scary warning. This is a *feature*, not a bug — but it trips up automation when servers are legitimately reprovisioned.

**Key permission errors:** SSH is paranoid about private key file permissions. If `~/.ssh/id_rsa` is world-readable (chmod 644), SSH refuses to use it. Must be `600`.

**Broken pipe on large transfers:** Network drops, server restarts, SSH timeouts (`ServerAliveInterval` config) kill the session. No auto-resume — the client must restart the transfer (or use tools like `rsync` over SSH instead).

**Subsystem not enabled:** Server must have `Subsystem sftp /usr/lib/openssh/sftp-server` in `/etc/ssh/sshd_config`. Missing this = "subsystem request failed" on connection.

---

## 8) Real-World Usage

- Standard secure file transfer between Linux/Unix servers
- CI/CD pipelines deploying artifacts to remote hosts
- Managed file transfer (MFT) platforms replacing legacy FTP
- Cloud storage gateways offering SFTP endpoints (AWS Transfer Family, Azure SFTP)
- Developers uploading to web servers (replaced FTP in this role almost universally)
- Automated secure data feeds between organizations

---

## 9) Comparison (SFTP vs FTPS)

| Feature | SFTP | FTPS |
|---|---|---|
| Based on | SSH-2 | FTP + TLS |
| Port | 22 (single) | 21 control + data ports |
| Connections | 1 | 2 (control + data) |
| Firewall friendly | ✅ Always | ⚠️ Passive needed |
| Authentication | SSH keys or password | TLS certificates + FTP login |
| Protocol type | Binary | Text commands + binary data |
| Resume support | Depends on client | Partial (REST command) |
| Widely supported | ✅ Every Unix system | Legacy-heavy |

---

## 10) Packet Walkthrough

```
[TCP SYN to port 22]
[SSH Version exchange: "SSH-2.0-OpenSSH_8.9"]
[Key Exchange: Diffie-Hellman, agree on AES-256-CTR + HMAC-SHA2-256]
[Server sends host key — client checks known_hosts]
[Client authenticates with public key: sends pubkey signature]
[SSH User Auth Success]
Client → Server: "SSH channel request: subsystem sftp"
Server → Client: "SSH channel open confirmation"

[SFTP protocol begins inside SSH channel]
Client → Server: SSH_FXP_INIT (version=3)
Server → Client: SSH_FXP_VERSION (version=3)
Client → Server: SSH_FXP_OPEN (id=1, filename="/data/report.pdf", flags=READ)
Server → Client: SSH_FXP_HANDLE (id=1, handle="\x00\x00\x00\x01")
Client → Server: SSH_FXP_READ (id=2, handle, offset=0, len=32768)
Client → Server: SSH_FXP_READ (id=3, handle, offset=32768, len=32768) ← pipelined
Server → Client: SSH_FXP_DATA (id=2, data=<32768 bytes>)
Server → Client: SSH_FXP_DATA (id=3, data=<32768 bytes>)
...
Server → Client: SSH_FXP_STATUS (EOF)
Client → Server: SSH_FXP_CLOSE (handle)
[SSH channel closes]
```

---

## 11) Common Interview / Exam Traps

- **"SFTP = FTP over SSH"** — No. SFTP is a completely separate protocol that *runs inside* SSH. FTPS is FTP over SSL/TLS. They are not the same. This distinction appears constantly in interviews.
- **"SFTP uses port 22 and 21"** — Only port 22. No port 21 involved at all.
- **"SSH keys are optional for SFTP"** — Password auth works too, but key-based is strongly preferred for automation (no interactive password prompt).
- **Confusing SFTP with SCP** — Both run over SSH on port 22, but SCP (`scp`) is a simpler file copy command; SFTP is a full interactive file transfer subsystem. SCP is being deprecated in many OpenSSH distributions.

---

## 12) Retrieval Prompts

- Why does SFTP only need one port when FTP needs two?
- What is a host key and what security property does it provide?
- How does SFTP request pipelining improve performance over high-latency links?
- What happens if you accidentally send an SFTP session over an unencrypted channel? (Trick: impossible by design — SSH is the transport)
- Why might SFTP be slower than FTP on a 10Gbps local network?

---

## 13) TL;DR

- SFTP ≠ FTP + SSH; it's a completely different binary protocol that runs as an SSH subsystem
- Single TCP connection on port 22 — encrypted, integrity-checked, NAT-friendly
- Authentication via SSH keys (preferred) or password; host keys prevent MITM
- Binary packet protocol with request IDs enabling pipelining for performance
- The modern default for any secure file transfer between Unix systems

---
---

# 3. TFTP — Trivial File Transfer Protocol

---

## 1) Concept Snapshot

**Definition:** TFTP (Trivial File Transfer Protocol, RFC 1350) is a stripped-down file transfer protocol that runs over **UDP** rather than TCP, has no authentication, no directory listing, no encryption, and fits in a few hundred lines of code.

**Purpose:** Designed for environments where simplicity and small footprint matter more than features or security — classically used for **network booting** (PXE), bootstrapping routers/switches, and transferring firmware where a full TCP stack may not even exist yet.

---

## 2) Mental Model

Imagine you need to hand someone a document through a mail slot, but you can only slip one page at a time, and you have to wait for them to knock back before sending the next page. There's no envelope, no return address on the package, no verification of who's on the other side — just a simple knock-pass-knock-pass rhythm. That's TFTP: dumb, tiny, reliable-enough through its own stop-and-wait mechanism.

---

## 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7)
- **Transport:** **UDP** (not TCP) — port 69 for initial request; random ephemeral port for data transfer
- **Talks to:** Network boot loaders (PXE/iPXE), router OS installers, embedded devices
- **Below it:** UDP → IP → typically on LAN (rarely crosses the internet)

---

## 4) Mechanics (How It Actually Works)

TFTP uses a **stop-and-wait ARQ** (Automatic Repeat reQuest) at the application layer since it uses UDP (which provides no reliability). Every data block must be acknowledged before the next is sent.

**Block size:** 512 bytes default (can negotiate larger with tsize/blksize options via RFC 2347).

**Step-by-step Read (RRQ):**
1. Client sends **RRQ** (Read Request) to server UDP port 69: `[opcode=1][filename][mode]`
2. Server picks a new ephemeral port (TID — Transfer ID) and sends **DATA block 1** (up to 512 bytes) from this new port
3. Client sends **ACK 1** to server's TID port
4. Server sends **DATA block 2**
5. Client sends **ACK 2**
6. ... (repeats)
7. When a DATA packet is **less than 512 bytes** → that's the last block (EOF signal)
8. Client sends final ACK → transfer complete

**Retry:** If ACK doesn't arrive within timeout, sender retransmits the last DATA/ACK. Default retries: 5. Default timeout: 5 seconds.

**The Sorcerer's Apprentice Bug:** If a delayed ACK (not a lost one) triggers a retransmit, you get duplicate data packets and duplicate ACKs, and they compound. Fixed by ignoring duplicate ACKs.

```
Client:random → Server:69   [RRQ "pxelinux.0" octet]
Server:TID   → Client:random  [DATA block=1, 512 bytes]
Client:random → Server:TID    [ACK block=1]
Server:TID   → Client:random  [DATA block=2, 512 bytes]
Client:random → Server:TID    [ACK block=2]
...
Server:TID   → Client:random  [DATA block=N, <512 bytes] ← EOF
Client:random → Server:TID    [ACK block=N]
```

---

## 5) Key Structures & Components

**TFTP Opcodes:**

| Opcode | Name | Meaning |
|---|---|---|
| 1 | RRQ | Read Request |
| 2 | WRQ | Write Request |
| 3 | DATA | Data block |
| 4 | ACK | Acknowledgment |
| 5 | ERROR | Error message |

**ERROR Codes:** 0=Not defined, 1=File not found, 2=Access violation, 3=Disk full, 4=Illegal operation, 5=Unknown TID, 6=File already exists, 7=No such user

**Transfer Modes:**
- `octet` (binary) — raw bytes, use for everything binary
- `netascii` — ASCII with CRLF translation (largely historical)
- `mail` — obsolete

**TFTP Options (RFC 2347):** blksize (increase block size beyond 512), timeout, tsize (transfer size). Negotiated in OACK packet.

---

## 6) Performance & Tradeoffs

- Stop-and-wait is **brutally inefficient over high-latency links** — one 512-byte block per RTT. On a WAN with 100ms RTT, maximum throughput is ~5 KB/s at default block size
- On a **local LAN** (sub-1ms RTT), this is rarely an issue for small files like boot images
- **No TCP overhead** — no handshake, no connection state, starts transferring almost immediately; critical for PXE where speed of first byte matters
- **Block size negotiation** (blksize=1468 to fit Ethernet MTU, avoiding fragmentation) dramatically improves performance when supported
- **Block number wraps at 65535** — for large files with 512-byte blocks, this causes problems at 32 MB. With larger block sizes, the threshold scales up.

---

## 7) Failure Modes

**No authentication:** Any client that can reach UDP port 69 can read or write files. TFTP servers are usually locked down to read-only from a specific directory (`/tftpboot`), or firewalled to the LAN only.

**Block number wraparound:** File larger than 65535 × block_size bytes causes block numbers to wrap around (they're 16-bit). With 512-byte blocks, that's a ~32 MB limit. Rollover Option (RFC 2347) or larger block sizes mitigate this.

**Silent corruption:** No checksums on data. A corrupt UDP payload (rare with hardware checksums, but possible) will silently corrupt the file. TFTP has no integrity verification.

**Stuck transfer:** If both sides lose their state and a packet is lost at exactly the wrong moment, the connection silently hangs until timeout. With no TCP keepalives, this requires external detection.

**The "last ACK" problem:** After the client sends the final ACK, if it's lost, the server will retransmit the last DATA block. The client must handle this gracefully (and it usually does — most implementations wait a short time for retransmits before exiting).

---

## 8) Real-World Usage

- **PXE (Preboot Execution Environment):** Servers and desktops with no OS fetch a boot image (pxelinux, grub, iPXE) via TFTP before any OS is loaded. This is TFTP's killer use case.
- **Router/switch provisioning:** Cisco, Juniper, and other network devices use TFTP to backup and restore configs (`copy running-config tftp://...`)
- **Firmware updates** for embedded devices, VoIP phones (phones fetch config from `tftp://provisioning-server/`)
- **DOCSIS cable modems** fetch configuration files via TFTP at startup
- **Network-based OS installation** (Kickstart, PXE + Anaconda on Linux)

---

## 9) Comparison (FTP vs SFTP vs TFTP)

| Feature | FTP | SFTP | TFTP |
|---|---|---|---|
| Transport | TCP | TCP (via SSH) | UDP |
| Port | 21 + data | 22 | 69 |
| Authentication | Username/Password | SSH key or password | None |
| Encryption | None | Full (SSH) | None |
| Directory listing | ✅ | ✅ | ❌ |
| Resume transfers | Partial | Partial | ❌ |
| File size limit | Unlimited | Unlimited | ~32 MB (512B blocks) |
| Reliability | TCP | TCP | Application-level ARQ |
| Typical use | Legacy file sharing | Secure file transfer | PXE boot, device config |
| Complexity | Medium | Low (1 connection) | Trivial |
| Works without OS | ❌ | ❌ | ✅ |

---

## 10) Packet Walkthrough (PXE Boot Scenario)

```
[DHCP: server tells PXE client to get "pxelinux.0" from 10.0.0.1]

Client:1234 → 10.0.0.1:69
  [RRQ opcode=1, filename="pxelinux.0", mode="octet"]

10.0.0.1:54321 → Client:1234
  [DATA opcode=3, block=1, 512 bytes of boot image]

Client:1234 → 10.0.0.1:54321
  [ACK opcode=4, block=1]

10.0.0.1:54321 → Client:1234
  [DATA opcode=3, block=2, 512 bytes]

Client:1234 → 10.0.0.1:54321
  [ACK opcode=4, block=2]

... (N blocks) ...

10.0.0.1:54321 → Client:1234
  [DATA opcode=3, block=N, 87 bytes]  ← less than 512 = EOF

Client:1234 → 10.0.0.1:54321
  [ACK opcode=4, block=N]

[Transfer complete — PXE client executes boot image]
```

---

## 11) Common Interview / Exam Traps

- **"TFTP is unreliable because it uses UDP"** — Partially true, but TFTP implements its own stop-and-wait reliability at the application layer. It *is* reliable for data delivery; it just has no security and is slow.
- **"TFTP uses port 69 throughout"** — Only the initial request goes to port 69. All subsequent DATA and ACK packets use a new ephemeral TID (port) chosen by the server. This trips up firewall rules.
- **"TFTP can list directories"** — No. You must know the exact filename. There is no LIST command. This is intentional — simplicity and security surface reduction.
- **End-of-transfer detection** — A DATA packet shorter than the block size signals EOF. If a file is an exact multiple of block size, a zero-byte DATA packet must be sent as the final "it's over" signal. Students often forget this edge case.

---

## 12) Retrieval Prompts

- Why does TFTP use UDP instead of TCP? What does it give up and what does it gain?
- How does TFTP know the transfer is over if there's no FIN packet?
- What is the "Sorcerer's Apprentice Bug" in TFTP and why does it happen?
- Why is TFTP essential for PXE booting even though FTP exists?
- A TFTP server on a Cisco switch lets anyone on the internet access it — what's the attack surface?

---

## 13) TL;DR

- TFTP = UDP-based, no auth, no encryption, no directory listing — deliberately trivial
- Stop-and-wait ARQ: one 512-byte block per round trip; efficient only on LAN
- End of file = DATA packet shorter than block size (zero-byte packet if file is exact multiple)
- Primary use case: PXE network booting, router config backup, embedded device provisioning
- Never use on the internet or for anything sensitive — it's a LAN tool by design

---
---

# 4. SCP — Secure Copy Protocol

---

## 1) Concept Snapshot

**Definition:** SCP (Secure Copy Protocol) is a file copy utility and protocol running over SSH-2. It is essentially an SSH-tunneled version of the older `rcp` (remote copy) protocol. Unlike SFTP, it is not a full interactive file transfer subsystem — it is a one-shot copy command.

**Purpose:** Fast, simple, authenticated, encrypted file transfer between hosts when you just want to copy a file and don't need browsing, resuming, or interactive sessions.

---

## 2) Mental Model

If SFTP is an encrypted FTP session (interactive, stateful, can browse), SCP is an encrypted version of `cp`. You run one command, one file (or directory) moves, the connection closes. No browsing, no session, no going back for more. It's a point-and-shoot camera, not a full studio setup.

---

## 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7), inside SSH
- **Transport:** TCP port 22 (SSH)
- **Talks to:** SSH daemon; spawns a remote `scp` process on the server side
- **Below it:** SSH session → TCP → IP

---

## 4) Mechanics (How It Actually Works)

SCP works by opening an SSH connection and running a remote `scp -t` (sink mode) or `scp -f` (source mode) process on the server. The data flows through the SSH channel as a byte stream with a minimal text-based control protocol on top.

**Copy local to remote:**
```bash
scp file.txt user@host:/remote/path/
```
1. SSH connection established and authenticated
2. Remote `scp -t /remote/path/` is executed on the server
3. Local `scp` sends a control line: `C0644 <size> <filename>\n`
4. Remote acknowledges with `\0` (null byte)
5. Local sends raw file bytes
6. Local sends `\0` (signals end of data)
7. Remote responds `\0` (confirms receipt)
8. SSH channel closes

**Recursive copy (`-r` flag):** sends directory control lines `D<perms> 0 <dirname>` before files, `E` to end a directory. It's a simple recursive tree protocol.

---

## 5) Key Structures & Components

SCP control protocol is minimal text lines over SSH:

| Line | Meaning |
|---|---|
| `C<perms> <size> <name>` | Incoming file with permissions, size, name |
| `D<perms> 0 <name>` | Start of directory |
| `E` | End of directory |
| `\0` | ACK / end of data |
| `\1<message>` | Warning message |
| `\2<message>` | Fatal error |

No file handles, no opcodes, no request IDs — just sequential control lines and raw bytes.

---

## 6) Performance & Tradeoffs

- **Faster than SFTP for single large files** in some implementations because the protocol overhead is minimal — fewer round trips
- **No pipelining, no concurrency** — sequential transfers only; SFTP with pipelining beats SCP for many-small-files scenarios
- **No integrity verification** — file bytes stream through with no checksum or hash at the SCP protocol level (SSH MAC protects against in-transit corruption but not server-side issues)
- **No resume** — transfer interrupted = start over
- Being **deprecated** in OpenSSH in favor of SFTP mode (`ssh -s sftp`) due to security concerns (see below)

---

## 7) Failure Modes

**Security design flaw (CVE-2018-20685, etc.):** The server-side `scp` process has some control over how files are named and where they land on the client. A malicious server can overwrite arbitrary files on the client during an `scp` download. This is a protocol-level issue, not fixable without breaking the protocol — one of the primary reasons SCP is being deprecated.

**Silent failure on some errors:** SCP's minimal protocol means some error conditions are ambiguously reported or silently dropped.

---

## 8) Real-World Usage

- Quick ad-hoc file copying between Linux servers in sysadmin work
- CI/CD pipelines (though increasingly replaced by rsync or SFTP)
- Copying config files, logs, scripts to/from remote hosts
- Being phased out — modern OpenSSH encourages using `sftp` or `rsync --rsh=ssh`

---

## 9) Comparison (SCP vs SFTP)

| Feature | SCP | SFTP |
|---|---|---|
| Protocol | SSH + rcp-style | SSH subsystem (binary) |
| Interactive session | ❌ | ✅ |
| Directory browsing | ❌ | ✅ |
| Resume transfers | ❌ | ❌ (usually) |
| Pipelining | ❌ | ✅ |
| Security (protocol level) | ⚠️ Design flaws | ✅ Cleaner |
| Speed (single large file) | ≈ same | ≈ same |
| Speed (many small files) | Slower | Faster (pipelining) |
| Deprecation status | Being deprecated | Actively maintained |

---

## 11) Common Interview / Exam Traps

- **"SCP and SFTP are the same thing"** — Both use SSH port 22, but SCP is a simple copy protocol derived from rcp; SFTP is a rich file management subsystem. Different protocols entirely.
- **"SCP is more secure than SFTP"** — Actually the opposite. SCP has known protocol-level vulnerabilities; SFTP is the safer choice.

---

## 13) TL;DR

- SCP = one-shot encrypted file copy over SSH; no interactivity, no browsing
- Built on SSH → encrypted + authenticated; inherits all SSH auth methods
- Minimal rcp-style protocol: control lines + raw bytes; fast but limited
- No resume, no pipelining, has known security design issues
- Being deprecated in favor of SFTP — prefer SFTP for new tooling

---
---

# 5. FTPS — FTP Secure

---

## 1) Concept Snapshot

**Definition:** FTPS (FTP Secure, RFC 4217) is FTP with TLS/SSL encryption added. It is not a new protocol — it is standard FTP with a TLS layer wrapping the control connection and optionally the data connection. Also called "FTP over TLS" or "FTP/S."

**Purpose:** Adds encryption and authentication to FTP for organizations that need secure file transfer but are committed to the FTP ecosystem (legacy systems, compliance requirements, existing FTP infrastructure).

---

## 2) Mental Model

If FTP is sending documents in a transparent envelope, FTPS is taking that same postal system — same trucks, same mail slots, same routes — but putting each envelope inside an opaque locked box. The underlying postal infrastructure is unchanged; security is layered on top.

---

## 3) Layer Context

- **OSI Layer:** Application Layer (Layer 7), TLS at Session/Transport layer boundary
- **Transport:** TCP; port 21 (Explicit FTPS) or port 990 (Implicit FTPS)
- **Talks to:** FTPS-capable FTP client/server
- **Below it:** TLS record layer → TCP → IP

---

## 4) Mechanics — Two Modes

**Explicit FTPS (FTPES) — preferred:**
1. Client connects to port 21 (normal FTP port)
2. Client sends `AUTH TLS` command on the control channel
3. TLS handshake happens on the control connection
4. Control channel is now encrypted
5. Client optionally requests encrypted data connections via `PROT P` (Protected)
6. Client issues `PASV`/`PORT`, and data connections are TLS-wrapped

**Implicit FTPS — legacy:**
1. Client connects to port 990
2. TLS handshake immediately (no plaintext negotiation)
3. Control and data channels both TLS-wrapped from the start
4. Considered legacy; RFC 4217 recommends Explicit

**Key TLS-specific FTP commands:**
- `AUTH TLS` — upgrade control connection to TLS
- `AUTH SSL` — older variant (deprecated)
- `PROT P` — require encrypted data connections (Protected)
- `PROT C` — clear (unencrypted) data connections
- `PBSZ 0` — required before PROT; sets protection buffer size

---

## 5) Key Structures & Components

- **TLS Certificate:** Server presents X.509 certificate; client optionally presents one for mutual TLS (mTLS). Certificate validates server identity.
- **Session resumption:** TLS session can be resumed for data connections to avoid full handshake overhead for each transfer
- **CCC (Clear Command Channel):** Some clients send `CCC` after authentication to drop TLS on the control channel — useful for compatibility with certain NAT devices, but reduces security

---

## 6) Performance & Tradeoffs

- **Two connections + TLS overhead** — FTPS retains all the firewall complexity of FTP *plus* adds TLS negotiation overhead to each connection
- **Certificate management** — servers need valid TLS certificates; adds operational overhead
- **Passive mode firewall rules** — same headache as FTP; must open a port range for data connections
- **Data connection TLS sessions** — each data connection (each file transfer) requires its own TLS handshake unless session resumption is used
- **Compatibility** — widely supported in enterprise FTP servers (FileZilla Server, ProFTPD, vsftpd, Microsoft IIS) and client software

---

## 7) Failure Modes

**Certificate validation disabled:** Many legacy FTPS clients and servers have TLS verification turned off by default (accepting any certificate). This defeats the purpose of FTPS — no MITM protection.

**Firewall TLS inspection breaks FTPS:** Deep packet inspection firewalls that intercept TLS can disrupt FTPS data connections because the TLS session for the data channel must be established between the client and server directly.

**Passive port range misconfiguration:** Must explicitly open a range of TCP ports on the server firewall for passive data connections. Often misconfigured, causing "425 Can't open data connection" errors.

---

## 8) Real-World Usage

- Payment processing companies using EDI/file transfer under PCI-DSS compliance (which mandates encryption)
- Legacy enterprise Managed File Transfer (MFT) platforms
- Healthcare file exchanges (HIPAA compliance driving encryption requirement)
- Wherever existing FTP infrastructure can't be replaced but must be made compliant

---

## 9) Comparison (FTPS vs SFTP — the important one)

| Feature | FTPS | SFTP |
|---|---|---|
| Based on | FTP + TLS | SSH subsystem |
| Port(s) | 21 (+ passive range) or 990 | 22 only |
| Connections | 2 (control + data) | 1 |
| Firewall complexity | High (passive port range) | Low (single port) |
| Certificate type | X.509 (TLS) | SSH host keys |
| Auth | TLS cert + FTP credentials | SSH keys or password |
| Protocol age | FTP (1971) + TLS (1999) | SFTP (1990s) |
| Legacy compatibility | High | Medium |
| Modern preference | ❌ | ✅ |

---

## 11) Common Interview / Exam Traps

- **"FTPS and SFTP are both 'secure FTP'"** — They are completely different protocols. FTPS = FTP + TLS. SFTP = SSH File Transfer Protocol. Different ports, different architectures, different everything.
- **"FTPS is just FTP on port 443"** — No. Explicit FTPS still uses port 21. Implicit FTPS uses port 990. Neither uses 443.
- **"PROT P is automatic"** — It is not. You must explicitly negotiate `PBSZ 0` then `PROT P` to get encrypted data connections. Many misconfigured servers encrypt the control channel but leave data connections unencrypted.

---

## 13) TL;DR

- FTPS = FTP + TLS; same two-connection architecture, same commands, with encryption added
- Two modes: Explicit (AUTH TLS on port 21, preferred) and Implicit (TLS-first on port 990, legacy)
- Retains all of FTP's firewall headaches plus adds TLS certificate management
- PROT P needed to encrypt data connections — not just control channel
- Prefer SFTP for new systems; FTPS exists for legacy FTP environments needing compliance

---
---

# 🔑 Master Comparison Table — All File Transfer Protocols

| Feature | FTP | FTPS | SFTP | SCP | TFTP |
|---|---|---|---|---|---|
| Transport | TCP | TCP + TLS | SSH/TCP | SSH/TCP | UDP |
| Port(s) | 21 + data | 21/990 + data | 22 | 22 | 69 + TID |
| Encryption | ❌ | ✅ TLS | ✅ SSH | ✅ SSH | ❌ |
| Authentication | User/Pass | TLS cert + User/Pass | SSH key / password | SSH key / password | None |
| Directory listing | ✅ | ✅ | ✅ | ❌ | ❌ |
| Firewall friendly | ❌ | ❌ | ✅ | ✅ | Mostly LAN |
| Resume transfers | Partial | Partial | Partial | ❌ | ❌ |
| Interactive | ✅ | ✅ | ✅ | ❌ | ❌ |
| Integrity check | ❌ | ✅ TLS MAC | ✅ SSH MAC | ✅ SSH MAC | ❌ |
| Typical use case | Legacy file sharing | Compliance FTP | Secure server-to-server | Quick CLI copy | PXE boot, device config |
| Status | Legacy | Compliance use | Modern standard | Deprecated | Niche/embedded |

---

*Use this table as your rapid-revision anchor. Every row is a potential exam question or interview follow-up.*