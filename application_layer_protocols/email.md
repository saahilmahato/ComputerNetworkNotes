# 🧠 Computer Networks

## Email Protocols: SMTP, POP3, IMAP

---

# 📧 SMTP — Simple Mail Transfer Protocol

---

### 1) Concept Snapshot

**Definition:** SMTP is an application-layer protocol used to **send and relay email messages** between mail servers, and from a mail client to a mail server. It operates over TCP, typically on port 25 (server-to-server), 587 (client submission), or 465 (SMTPS).

**Purpose:** SMTP's sole job is *pushing* mail outward — from your client to your outgoing server, and then from server to server until it reaches the destination mail server. It has no mechanism for retrieving mail.

---

### 2) Mental Model

Think of SMTP as a **postal courier service**. When you hand a letter to a courier (your mail client sends to your SMTP server), the courier doesn't deliver it directly to the recipient's home — they pass it through a chain of sorting facilities (relay servers) until it reaches the local post office (destination mail server) closest to the recipient. The courier only *delivers outward*. Someone else handles pickup.

```
[You] → [Your SMTP Server] → [Relay/MTA] → [Recipient's Mail Server]
         (submission)         (relay)         (final delivery)
```

---

### 3) Layer Context

| Property | Value |
|---|---|
| OSI Layer | Application (Layer 7) |
| Transport | TCP |
| Default Ports | 25 (relay), 587 (submission), 465 (SMTPS) |
| Talks to | MUA (Mail User Agent) above, TCP below |
| Peer | Another SMTP server (MTA to MTA) |

**MUA** = your email client (Outlook, Thunderbird). **MTA** = Mail Transfer Agent (the SMTP server doing the relaying).

---

### 4) Mechanics — How It Actually Works

**SMTP is a command-response protocol.** The client issues plain-text commands; the server responds with 3-digit status codes.

**Step-by-step connection flow:**

**Step 1 — TCP Connection:** Client opens TCP connection to server on port 25/587.

**Step 2 — Greeting:** Server sends `220 mail.example.com ESMTP ready`.

**Step 3 — EHLO/HELO:** Client identifies itself. `EHLO` is the modern extended version; it also lets the server advertise capabilities (STARTTLS, AUTH, SIZE, etc.).

**Step 4 — AUTH (if submission):** Client authenticates using mechanisms like LOGIN or PLAIN.

**Step 5 — MAIL FROM:** Client declares the envelope sender — `MAIL FROM:<alice@example.com>`.

**Step 6 — RCPT TO:** Client specifies recipient(s) — `RCPT TO:<bob@other.com>`. Each recipient needs its own RCPT TO command.

**Step 7 — DATA:** Client sends `DATA`, then the full message (headers + body), terminated by a line containing only a `.` (period).

**Step 8 — QUIT:** Client closes the session.

**Sample Dialogue:**

```
S: 220 mail.server.com ESMTP Postfix
C: EHLO client.example.com
S: 250-mail.server.com
S: 250-STARTTLS
S: 250 AUTH LOGIN PLAIN
C: AUTH LOGIN
C: [base64 credentials]
S: 235 Authentication successful
C: MAIL FROM:<alice@example.com>
S: 250 Ok
C: RCPT TO:<bob@other.com>
S: 250 Ok
C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: From: Alice <alice@example.com>
C: To: Bob <bob@other.com>
C: Subject: Hello
C:
C: Hi Bob!
C: .
S: 250 Ok: queued as A1B2C3
C: QUIT
S: 221 Bye
```

---

### 5) Key Structures & Components

**Important SMTP Commands:**

| Command | Purpose |
|---|---|
| `EHLO` | Extended greeting, negotiates capabilities |
| `MAIL FROM` | Sets envelope sender |
| `RCPT TO` | Sets envelope recipient |
| `DATA` | Begins message transfer |
| `STARTTLS` | Upgrades to TLS encryption |
| `AUTH` | Authenticates the client |
| `QUIT` | Closes session |
| `VRFY` | Verify if address exists (often disabled) |
| `RSET` | Abort current mail transaction |

**Status Code Classes:**

| Code Range | Meaning |
|---|---|
| 2xx | Success |
| 3xx | Intermediate (more input needed) |
| 4xx | Temporary failure (try again later) |
| 5xx | Permanent failure (don't retry) |

**Envelope vs. Headers:** A critical distinction — SMTP operates on the *envelope* (MAIL FROM / RCPT TO), which is separate from the message headers (From: / To:). This is how BCC works — the RCPT TO contains the BCC address, but the message headers don't mention them.

---

### 6) Performance & Tradeoffs

- **Push-only design** keeps it simple but means you need a separate protocol (POP3/IMAP) for retrieval. This separation of concerns is actually good design.
- **No built-in authentication in original spec** — SMTP was designed in a trusting academic environment. Open relays were the norm, and spam abuse followed. AUTH and STARTTLS were bolted on later.
- **Port 25 vs 587:** ISPs block port 25 for end users to prevent spam from compromised machines. Port 587 (submission) requires authentication and is the correct port for clients.
- **Store-and-forward model** adds latency but provides resilience — if the next hop is down, the current server queues and retries.

---

### 7) Failure Modes

**What breaks and why:**

- **550 User Unknown** — recipient address doesn't exist. Permanent failure. Your server stops trying.
- **421 / 4xx errors** — destination server temporarily unavailable. SMTP queues and retries with exponential backoff (typically for up to 5 days).
- **Relay denied** — you're trying to use a server that isn't yours. Open relay abuse shut this down; servers now only relay for authenticated or locally-sourced mail.
- **Spam filters rejecting mail** — SPF, DKIM, DMARC failures cause legitimate mail to be rejected. These are DNS-based extensions layered on top of SMTP.
- **TLS negotiation failure** — if STARTTLS is advertised but fails (e.g., cert mismatch), many servers fall back to plaintext (OPPORTUNISTIC TLS), which is a security problem. DANE and MTA-STS address this.

---

### 8) Real-World Usage

- Every email you send goes through SMTP.
- Gmail, Outlook, ProtonMail all run SMTP servers for outbound mail.
- DevOps: transactional email services like SendGrid, Mailgun, and AWS SES expose SMTP endpoints.
- Used in monitoring/alerting systems to send notification emails programmatically.
- **Not** used to read your inbox — that's POP3 or IMAP.

---

### 11) Common Interview / Exam Traps

- **"SMTP is used for sending and receiving email"** — WRONG. SMTP is send/relay only. Receiving is POP3 or IMAP.
- **Port confusion:** 25 = server-to-server relay. 587 = client-to-server submission (with AUTH). 465 = legacy SMTPS (implicit TLS).
- **Envelope ≠ Headers:** `MAIL FROM` is the envelope sender (used for bounces), not necessarily the `From:` header the user sees. These can differ — this is how spoofing works, and why SPF checks MAIL FROM.
- **EHLO vs HELO:** HELO is plain SMTP (RFC 821). EHLO is extended SMTP (ESMTP, RFC 1869) and is always preferred. If a server doesn't support EHLO, you fall back to HELO.
- **SMTP doesn't encrypt by default** — STARTTLS upgrades an existing connection; SMTPS (port 465) uses TLS from the start.

---

### 12) Retrieval Prompts

- What is the difference between `MAIL FROM` and the `From:` header?
- Why does SMTP have two separate ports for sending (25 vs 587)?
- What happens when a 4xx error is returned vs a 5xx?
- How does BCC work at the SMTP protocol level?
- Why was SMTP originally designed without authentication?

---

### 13) TL;DR

- SMTP = outbound only. Sending and relaying email, not receiving.
- Command-response protocol over TCP. Plain text commands, 3-digit response codes.
- Port 25 (relay), 587 (client submission with auth), 465 (implicit TLS).
- Envelope (MAIL FROM / RCPT TO) is separate from message headers — critical distinction.
- Auth and encryption (STARTTLS/TLS) were retrofitted additions; not in the original design.

---
---

# 📥 POP3 — Post Office Protocol v3

---

### 1) Concept Snapshot

**Definition:** POP3 is an application-layer protocol that allows a mail client to **download email from a mail server to a local device**, typically deleting the messages from the server afterward. It operates over TCP on port 110 (plaintext) or 995 (POP3S with TLS).

**Purpose:** POP3 solves the retrieval half of email — once SMTP has delivered a message to your mail server, POP3 lets your client pick it up. It was designed for the era of a single device per user and intermittent connectivity (dial-up).

---

### 2) Mental Model

POP3 is like **collecting your physical mail from a P.O. box**. You go to the post office (connect to the server), open your box, take all your letters home (download to local device), and the box is now empty (messages deleted from server). If you lose your letters at home, they're gone — the P.O. box has nothing left.

This works fine if you only ever check email from one place. The moment you have a phone *and* a laptop, it breaks down.

---

### 3) Layer Context

| Property | Value |
|---|---|
| OSI Layer | Application (Layer 7) |
| Transport | TCP |
| Default Ports | 110 (plaintext), 995 (POP3S) |
| Direction | Client pulls from server |
| Works With | SMTP (which delivered the mail to the server) |

---

### 4) Mechanics — How It Actually Works

POP3 has three phases: **Authorization**, **Transaction**, and **Update**.

**Authorization Phase:** Client connects, server greets, client sends `USER` and `PASS`.

**Transaction Phase:** Client can list, retrieve, and delete messages. All deletions are just *marked* for deletion during this phase.

**Update Phase:** When the client sends `QUIT`, the server permanently deletes anything marked for deletion and closes the session.

**Sample Dialogue:**

```
S: +OK POP3 server ready
C: USER alice
S: +OK
C: PASS secret
S: +OK alice's mailbox has 3 messages (4200 octets)
C: LIST
S: +OK 3 messages
S: 1 1200
S: 2 800
S: 3 2200
S: .
C: RETR 1
S: +OK 1200 octets
S: [full message content]
S: .
C: DELE 1
S: +OK message 1 deleted
C: QUIT
S: +OK POP3 server signing off (messages deleted)
```

**Status:** Responses are either `+OK` (success) or `-ERR` (failure). Simple binary.

---

### 5) Key Structures & Components

| Command | Purpose |
|---|---|
| `USER` | Send username |
| `PASS` | Send password |
| `STAT` | Get mailbox summary (count, total size) |
| `LIST` | List messages with sizes |
| `RETR n` | Download message number n |
| `DELE n` | Mark message n for deletion |
| `NOOP` | Keep-alive, do nothing |
| `RSET` | Unmark all deletions |
| `QUIT` | Enter update phase and disconnect |
| `TOP n l` | Download headers + first l lines of message n |
| `UIDL` | Get unique ID per message (for "leave on server" logic) |

**The UIDL command** is what enables "leave a copy on the server" mode in email clients — the client tracks which UIDs it has already downloaded and only fetches new ones.

---

### 6) Performance & Tradeoffs

- **Simple and fast** — minimal protocol overhead. Good for constrained environments and slow connections.
- **Terrible for multi-device use** — mail downloaded to one device is gone from the server, so other devices never see it.
- **"Leave on server" workaround** is messy — clients use UIDL to track what they've downloaded, but the server's inbox grows unboundedly since nothing gets deleted.
- **No server-side state** — the server doesn't know what you've read, flagged, or organized. All of that is local only.
- **No partial download** — POP3 downloads the entire message. On a slow connection with large attachments, you're downloading everything whether you want it or not (though TOP helps for headers).

---

### 7) Failure Modes

- **Message loss on crash** — if your client crashes between RETR and DELE, the message may be downloaded but then re-downloaded next time (since DELE wasn't confirmed) — or lost if the opposite order happens.
- **Credential exposure** — original POP3 sends `USER` and `PASS` in plaintext. POP3S (port 995) or STARTTLS is required for security.
- **Mailbox lock** — some POP3 servers lock the mailbox during a session. A second client connecting simultaneously will be refused, causing "mailbox busy" errors.
- **No sync between devices** — if you read an email on your phone via POP3, your laptop has no idea.

---

### 9) Comparison — POP3 Variants

| Feature | POP3 (plain) | POP3 + UIDL | POP3S |
|---|---|---|---|
| Port | 110 | 110 | 995 |
| Encryption | None | None | TLS |
| Multi-device | No | Limited | No |
| Server state after session | Empty | Can retain | Empty |

---

### 11) Common Interview / Exam Traps

- **"POP3 always deletes messages"** — not necessarily true. With UIDL and client-side configuration, messages can be left on the server. But the protocol's default behavior and *design intent* is download-and-delete.
- **Deletions aren't immediate** — messages are only marked during the Transaction phase. The actual deletion happens in the Update phase when `QUIT` is issued. If the connection drops before `QUIT`, nothing is deleted.
- **POP3 has no folders** — there is one inbox. That's it. Folder organization is purely local to your client.
- **Port 995 vs 110** — 110 is plaintext, 995 is implicit TLS (POP3S). Don't confuse with IMAP's 143/993 pair.

---

### 13) TL;DR

- POP3 = download email to local device, typically delete from server.
- Three phases: Authorization → Transaction → Update (deletion happens on QUIT).
- Designed for single-device, offline-first usage.
- No server-side state — no folders, no read/unread sync, nothing.
- Port 110 (plain), 995 (TLS). Binary responses: `+OK` or `-ERR`.

---
---

# 📂 IMAP — Internet Message Access Protocol

---

### 1) Concept Snapshot

**Definition:** IMAP is an application-layer protocol that allows email clients to **access and manage email messages stored on a remote mail server**, keeping them on the server and synchronizing state (read, flagged, folders, etc.) across multiple clients. It operates over TCP on port 143 (plaintext) or 993 (IMAPS with TLS).

**Purpose:** IMAP solves what POP3 couldn't: working with email from multiple devices while keeping everything in sync. The authoritative copy lives on the server, not your device.

---

### 2) Mental Model

IMAP is like **working directly at your office desk in a shared building**. The files (emails) stay in the office's filing cabinets (the server). You can go to any desk in any building (any device, anywhere) and always see the same filing cabinets in the same state. If you move a file to a drawer (folder), mark it with a sticky note (flag), or throw it away (delete), everyone sees that change. You don't take the files home — you access them remotely.

POP3 is like taking files home. IMAP is like working in the office.

---

### 3) Layer Context

| Property | Value |
|---|---|
| OSI Layer | Application (Layer 7) |
| Transport | TCP |
| Default Ports | 143 (plaintext/STARTTLS), 993 (IMAPS) |
| Direction | Bidirectional — client reads, writes, manages; server stores |
| Works With | SMTP (delivers inbound), client MUA for display |

---

### 4) Mechanics — How It Actually Works

IMAP is **stateful and session-based**. Unlike POP3's simple three-phase model, IMAP has richer state and a connection stays active while you work.

**Connection States:**

- **Not Authenticated** → client connects, server greets.
- **Authenticated** → after successful LOGIN or AUTHENTICATE.
- **Selected** → a specific mailbox (folder) is opened with SELECT or EXAMINE.
- **Logout** → session ends with LOGOUT.

**IMAP uses tagged commands.** Every command the client sends is prefixed with a unique tag (like `A001`, `A002`). The server's response for that command starts with the same tag. This allows the client to pipeline multiple commands and match responses.

**Untagged responses** (prefixed with `*`) are used for unsolicited server data — like notifying the client that new mail arrived while the session is open.

**Sample Dialogue:**

```
S: * OK IMAP4rev1 server ready
C: A001 LOGIN alice secret
S: A001 OK LOGIN completed
C: A002 LIST "" "*"
S: * LIST (\HasNoChildren) "/" INBOX
S: * LIST (\HasNoChildren) "/" Sent
S: * LIST (\HasNoChildren) "/" Drafts
S: A002 OK LIST completed
C: A003 SELECT INBOX
S: * 47 EXISTS        ← 47 messages in inbox
S: * 2 RECENT         ← 2 new since last session
S: * FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
S: A003 OK [READ-WRITE] SELECT completed
C: A004 FETCH 47 (FLAGS BODY[HEADER])
S: * 47 FETCH (FLAGS (\Recent) BODY[HEADER] {542}
S: [message headers]
S: )
S: A004 OK FETCH completed
C: A005 STORE 47 +FLAGS (\Seen)
S: * 47 FETCH (FLAGS (\Recent \Seen))
S: A005 OK STORE completed
C: A006 LOGOUT
S: * BYE IMAP4rev1 server logging out
S: A006 OK LOGOUT completed
```

---

### 5) Key Structures & Components

**Key Commands:**

| Command | Purpose |
|---|---|
| `LOGIN` / `AUTHENTICATE` | Authenticate with the server |
| `LIST` | List available mailboxes |
| `SELECT` / `EXAMINE` | Open a mailbox (read-write vs read-only) |
| `FETCH` | Retrieve message data (headers, body, flags, etc.) |
| `STORE` | Update message flags |
| `SEARCH` | Server-side search across messages |
| `COPY` / `MOVE` | Copy/move messages between mailboxes |
| `CREATE` / `DELETE` / `RENAME` | Manage mailboxes (folders) |
| `EXPUNGE` | Permanently remove messages flagged \Deleted |
| `IDLE` | Keep connection open; server pushes new mail notifications |
| `NOOP` | Ping server and receive any pending untagged responses |

**System Flags:**

| Flag | Meaning |
|---|---|
| `\Seen` | Message has been read |
| `\Answered` | Message has been replied to |
| `\Flagged` | Marked/starred |
| `\Deleted` | Marked for deletion (not yet expunged) |
| `\Draft` | Message is a draft |
| `\Recent` | New since last session (server-set only) |

**Message Identifiers:**
- **Sequence numbers** — position of message in the mailbox (can change as messages are added/removed).
- **UIDs** — persistent unique identifiers per message per mailbox. Clients use UIDs for reliable operations. UID validity flag tells the client if UIDs have been reset.

**BODYSTRUCTURE:** IMAP can return the MIME structure of a message *without downloading the full body*. This lets a client display "this email has a 10MB PDF attachment" without fetching the PDF. You only download what you explicitly request.

---

### 6) Performance & Tradeoffs

- **Bandwidth efficiency** — you can fetch just headers, just the text part, just one attachment. POP3 downloads everything.
- **Server storage cost** — since mail stays on the server indefinitely, server storage is the bottleneck. This is why Gmail gives you 15GB and charges for more.
- **Connection overhead** — IMAP sessions stay alive (especially with IDLE). This is more connections held open compared to POP3's quick connect-download-disconnect model.
- **IDLE vs polling** — IDLE allows the server to push new mail notifications to the client in real time. Without IDLE, clients have to poll (NOOP or re-SELECT) which wastes resources.
- **Complexity** — IMAP is dramatically more complex than POP3 to implement correctly. This complexity is the cost of its power.
- **Sequence number fragility** — operations using sequence numbers can be ambiguous if the mailbox changes mid-session. UID-based commands are safer but more verbose.

---

### 7) Failure Modes

- **UID validity reset** — if the server reassigns UIDs (e.g., after a mailbox rebuild), clients using cached UIDs will get confused. The UIDVALIDITY value signals this; clients must re-sync from scratch.
- **Network interruption mid-session** — unlike POP3, IMAP sessions are long-lived. A dropped connection can leave the client in an inconsistent state (e.g., a STORE that didn't complete). Clients must reconnect and re-check state.
- **Concurrent access conflicts** — multiple clients accessing the same mailbox simultaneously can see interleaved changes. IMAP handles this with untagged EXISTS/EXPUNGE notifications, but poorly written clients can get confused.
- **EXPUNGE timing** — a message marked `\Deleted` isn't gone until EXPUNGE is issued. A user deleting mail on one client doesn't see it disappear on another client until that client receives an EXPUNGE notification or re-syncs.
- **Server-side search limitations** — SEARCH runs on the server, which is powerful, but if the server has no index, it must scan all messages linearly. Large mailboxes make this slow.

---

### 8) Real-World Usage

- Gmail, Outlook, Apple Mail, Thunderbird — all use IMAP by default.
- Mobile email clients rely on IMAP (or proprietary push protocols like Exchange ActiveSync/EAS built on top of similar concepts).
- **IMAP IDLE** is what makes your phone receive email within seconds — the connection stays open, server pushes a notification.
- Corporate environments often use Exchange/MAPI, which provides IMAP access as a compatibility layer.
- The `dovecot` and `Cyrus` servers are common open-source IMAP implementations.

---

### 10) Packet Walkthrough — IMAP Fetch a New Email

> You open your phone's email app. It reconnects to the IMAP server.

1. **TCP connection** established to mail.example.com:993 (IMAPS).
2. **TLS handshake** completes.
3. Server: `* OK Dovecot ready`
4. Client: `A001 LOGIN alice@example.com password` → Server: `A001 OK`
5. Client: `A002 SELECT INBOX` → Server reports 50 EXISTS, 1 RECENT
6. Client: `A003 UID FETCH 1:* (FLAGS)` — fetches all UID-to-flag mappings to detect new messages.
7. Client identifies message UID 1147 as `\Recent` (new).
8. Client: `A004 UID FETCH 1147 (BODY.PEEK[HEADER])` — fetches headers only, without marking as read (PEEK).
9. App displays the email summary in the inbox list.
10. User taps to open: Client: `A005 UID FETCH 1147 (BODY[TEXT])` — fetches body.
11. Client: `A006 UID STORE 1147 +FLAGS (\Seen)` — marks as read.
12. Server propagates the `\Seen` flag to all other connected sessions for this account.
13. Client enters **IDLE**: `A007 IDLE` — server will now push any new mail without polling.

---

### 11) Common Interview / Exam Traps

- **"IMAP downloads email to your device"** — IMAP *accesses* mail on the server. It can cache locally (and most clients do for offline access), but the authoritative copy stays on the server. This is the core conceptual difference from POP3.
- **EXPUNGE ≠ DELETE flag** — setting `\Deleted` is just a flag. The message isn't removed until EXPUNGE. Many clients issue EXPUNGE only when you "empty trash" or close the mailbox.
- **SELECT vs EXAMINE** — both open a mailbox, but EXAMINE opens it read-only. No flags can be modified. Useful for background sync without accidentally changing state.
- **Sequence numbers change; UIDs don't** — if you EXPUNGE message 5, what was message 6 becomes message 5. Clients relying on sequence numbers after an EXPUNGE get wrong messages. Always prefer UID commands for reliability.
- **IMAP IDLE is not push in the mobile OS sense** — IDLE keeps a TCP connection open, which drains battery. Modern mobile platforms (iOS, Android) use their OS-level push notifications (APNs, FCM) that wake the app to reconnect, rather than maintaining persistent IDLE connections.
- **Port 143 supports both plaintext and STARTTLS** — the connection starts unencrypted, and the client upgrades with `STARTTLS`. Port 993 (IMAPS) is TLS from the first byte. Both result in encrypted sessions; the difference is when TLS starts.

---

### 12) Retrieval Prompts

- Why can IMAP show the same email on your phone and laptop simultaneously?
- What is the difference between FETCH and BODY.PEEK?
- What happens to other clients when you EXPUNGE a message?
- Why are UID commands preferred over sequence number commands?
- What does UIDVALIDITY tell a client?
- How does IMAP IDLE differ from polling?
- What is the difference between SELECT and EXAMINE?

---

### 13) TL;DR

- IMAP = email lives on the server, clients synchronize against it. Multi-device by design.
- Stateful, session-based, tagged command-response protocol.
- Can fetch partial messages (headers only, specific MIME parts) — very bandwidth efficient.
- Supports server-side folders, flags, search, and real-time push via IDLE.
- Port 143 (plain + STARTTLS), 993 (implicit TLS).
- More complex than POP3, but that complexity enables everything modern email requires.

---
---

# ⚔️ Comparison: SMTP vs POP3 vs IMAP

| Feature | SMTP | POP3 | IMAP |
|---|---|---|---|
| **Direction** | Send / Relay | Receive (download) | Receive (access) |
| **Mail location after access** | N/A | Local device | Server |
| **Multi-device support** | N/A | No (by design) | Yes |
| **Server-side folders** | No | No | Yes |
| **Read/flag sync** | No | No | Yes |
| **Partial fetch** | No | Partial (TOP) | Yes (BODY sections) |
| **Default port (plain)** | 25 / 587 | 110 | 143 |
| **Default port (TLS)** | 465 / 587+STARTTLS | 995 | 993 |
| **State on server** | Queue only | Inbox only | Full mailbox |
| **Complexity** | Medium | Low | High |
| **Designed for** | Sending & relay | Single device | Multi-device, always connected |
| **Real-time notifications** | No | No | Yes (IDLE) |

---

# 🔐 Security Addendum — Applies to All Three

All three protocols were designed in an era of academic trust. Security was retrofitted:

**Encryption:** All three have TLS variants. Always use SMTPS/POP3S/IMAPS or STARTTLS in production. Plaintext ports transmit credentials in the clear.

**Authentication:** SMTP AUTH, POP3's USER/PASS, and IMAP's LOGIN are all vulnerable to credential interception without TLS. Modern deployments use SASL mechanisms (PLAIN over TLS, OAUTHBEARER, etc.).

**Email authentication (SMTP-specific):**
- **SPF** — DNS record specifying which IPs are authorized to send for a domain. Checked against `MAIL FROM`.
- **DKIM** — Cryptographic signature in message headers. Verified against a DNS public key.
- **DMARC** — Policy layer that ties SPF and DKIM together and tells receivers what to do with failures (quarantine, reject, report).

These three together are why modern email is harder to spoof than it used to be, but not impossible.

---

# 🧩 How They All Fit Together

```
[Alice's Email Client]
        │
        │  SMTP (port 587) — Alice sends
        ▼
[Alice's Outgoing Mail Server / MTA]
        │
        │  SMTP (port 25) — Server-to-server relay
        ▼
[Bob's Incoming Mail Server]
        │
        ├──── POP3 (port 110/995) ────► Bob's Desktop (downloads, local copy)
        │
        └──── IMAP (port 143/993) ────► Bob's Phone / Laptop / Webmail (all in sync)
```

SMTP handles the entire left side of this diagram. POP3 and IMAP are two different answers to the same question on the right side: *how does Bob get his mail?* One answer is simple and limited; the other is powerful and complex. Modern usage has almost entirely moved to IMAP.