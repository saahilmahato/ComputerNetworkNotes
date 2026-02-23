# 🧠 Data Compression & Decompression — Presentation Layer

---

## 1) Concept Snapshot

**Definition:** Compression reduces the size of data by encoding it more efficiently — removing redundancy or approximating content. Decompression reverses this to restore data at the receiver's end.

**Purpose:** Saves bandwidth, reduces transmission time, and lowers storage costs. Critical wherever throughput is constrained or latency matters.

---

## 2) Mental Model

Imagine sending a letter that repeats "very very very very very cold" — instead you write "very ×5 cold." That's lossless compression. Now imagine describing a painting in words — you'll lose some detail, but the gist survives. That's lossy compression.

Your brain already compresses: when someone says "you know that tall guy from accounting?" you don't need his full description — context fills the gap. Compression exploits exactly this kind of *predictable redundancy*.

---

## 3) Layer Context

**OSI Layer:** Layer 6 — Presentation Layer. This is the layer responsible for *data translation, encryption, and compression* before it hands off to the Session layer (Layer 5) above and relies on Transport (Layer 4) below for delivery.

In the TCP/IP model, this functionality is typically absorbed into the Application layer — HTTP, TLS, and application codecs handle it directly.

**Who interacts with it:**
- Above: Application layer (your browser, video player, email client)
- Below: Session and Transport layers handle the *delivery* of whatever the Presentation layer packages up

---

## 4) Mechanics — How It Actually Works

### Lossless Compression Flow

```
Original Data → Redundancy Analysis → Encoded Output → Transmission → Decode → Original Data (exact)
```

**Run-Length Encoding (RLE):** Replaces consecutive repeated bytes with a count + value.
`AAAABBBCC → 4A3B2C`

**Huffman Coding:** Assigns shorter bit codes to frequent symbols, longer to rare ones. Builds a binary tree where frequent characters live near the root.
`'e' → 10` (2 bits) vs `'z' → 110100` (6 bits)

**LZ77 / LZ78 / LZW (used in gzip, GIF, ZIP):** Replaces repeated strings with references to earlier occurrences in a sliding window. Instead of repeating "compression" 10 times, you store it once and point back.

### Lossy Compression Flow

```
Original Data → Transform → Quantization (discard detail) → Encode → Transmit → Decode → Approximate Data
```

**DCT (Discrete Cosine Transform) — used in JPEG, MP3:**
Converts spatial/time domain data into frequency components. High-frequency components (fine detail, subtle sounds) are discarded first since humans perceive them least.

**Predictive Coding — used in video (H.264, H.265):**
Only differences between frames are encoded (inter-frame compression). A key frame (I-frame) is sent fully; subsequent frames only encode what changed.

---

## 5) Key Structures & Components

**Compression Ratio** = Original Size / Compressed Size. A ratio of 3:1 means the file is one-third its original size.

**Entropy:** The theoretical minimum bits needed to represent data without loss (Shannon's entropy). No lossless compressor can beat this floor.

**Codebook / Dictionary:** LZW-style algorithms build a table mapping patterns to short codes. Both sender and receiver maintain the same dictionary to encode/decode.

**Quantization Table (JPEG):** A matrix that determines how aggressively each frequency component is rounded. Higher quality → smaller divisors → less loss.

**I-frames, P-frames, B-frames (video):**
- I-frame: Full image, no reference needed
- P-frame: Encoded relative to previous frame
- B-frame: Encoded relative to both previous and next frames

---

## 6) Performance & Tradeoffs

**Lossless vs Lossy:** Lossless guarantees perfect recovery but achieves lower ratios (~2:1 to ~8:1 typically). Lossy achieves dramatic ratios (10:1 to 100:1+) at the cost of irreversible data loss — acceptable for perceptual media, catastrophic for text or executables.

**CPU cost vs bandwidth savings:** Compression shifts the bottleneck from the network to the CPU. In high-bandwidth, low-latency networks, compression overhead can actually *increase* total time. This is why HTTP/2 in LANs sometimes skips body compression.

**Compression ratio vs quality:** JPEG quality 10 vs quality 90 — smaller file, visibly degraded image. The tradeoff is tunable, not binary.

**Streaming vs block compression:** Real-time audio/video can't wait for the full file. Streaming codecs (like Opus, H.264) compress in chunks with bounded latency. Block compressors (like gzip) need the full input, making them unsuitable for live streams.

---

## 7) Failure Modes

**Corrupted compressed stream:** A single flipped bit in compressed data can cascade — one corrupted Huffman code desynchronizes the entire decode stream. Raw corrupt data usually damages only a localized region. This is why compressed streams are especially sensitive to transmission errors.

**Dictionary mismatch (LZW):** If sender and receiver fall out of sync on their dictionaries, decoding produces garbage. Protocols must agree on dictionary reset points.

**Over-compression (lossy):** Applying JPEG compression repeatedly (open → save → open → save) accumulates quantization error each round. Each generation degrades further — called *generation loss*.

**Decompression bombs:** A maliciously crafted file that decompresses to enormous size (a 1 KB zip expanding to 10 GB). Can crash or OOM-kill a server. Known as a *zip bomb*.

---

## 8) Real-World Usage

- **HTTP/HTTPS:** `Content-Encoding: gzip` or `br` (Brotli) — servers compress HTML, CSS, JS before sending. Browsers decompress transparently. Brotli typically achieves 15–25% better ratios than gzip for web assets.
- **Images:** JPEG (lossy photos), PNG (lossless graphics/screenshots), WebP (both modes, better than both for web).
- **Video streaming:** Netflix, YouTube use H.264/H.265/AV1. H.265 achieves same quality as H.264 at ~50% the bitrate.
- **Audio:** MP3, AAC (lossy), FLAC (lossless). Spotify streams AAC at 128–320 kbps depending on plan.
- **TLS compression:** Was used in HTTPS, then *disabled* after the CRIME attack (2012) showed that compression + encryption leaks information through side-channel size analysis.

---

## 9) Comparison Section

| Feature | Lossless | Lossy |
|---|---|---|
| Data recovery | Exact original | Approximation |
| Compression ratio | Low–moderate (2:1–8:1) | High (10:1–100:1+) |
| Use case | Text, executables, source code | Images, audio, video |
| Reversible | Yes | No |
| Examples | gzip, PNG, FLAC, ZIP | JPEG, MP3, H.264 |
| CPU cost | Moderate | High (especially decode) |

| Algorithm | Type | Where Used | Key Idea |
|---|---|---|---|
| Huffman | Lossless | gzip, DEFLATE | Variable-length codes |
| LZ77/LZW | Lossless | ZIP, GIF, gzip | Sliding window dictionary |
| Brotli | Lossless | Modern HTTPS | Pre-built dictionary + LZ |
| DCT | Lossy | JPEG, MP3 | Frequency domain quantization |
| H.264 | Lossy | Video streaming | Inter-frame prediction + DCT |
| AV1 | Lossy | YouTube, Netflix | Better than H.265, royalty-free |

---

## 10) Packet Walkthrough — HTTP with gzip

```
Browser → GET /index.html
          Accept-Encoding: gzip, br

Server  → Compresses HTML with gzip (say, 80KB → 18KB)
          Content-Encoding: gzip
          Content-Length: 18432
          [compressed body]

Network → Transmits 18KB instead of 80KB

Browser → Reads Content-Encoding header
          Decompresses body with gzip
          Parses resulting HTML
          Renders page
```

Neither TCP nor IP knows any of this happened. From their perspective, they just moved bytes.

---

## 11) Common Interview / Exam Traps

**"Presentation Layer doesn't exist in TCP/IP"** — Technically true. TCP/IP has no formal Presentation layer. Compression lives in application protocols (HTTP, TLS, codecs). OSI is a model; TCP/IP is reality.

**"Encrypted + compressed = more secure"** — Actually backwards. Compressing *before* encrypting leaks information. The CRIME and BREACH attacks exploited this. TLS disabled compression after CRIME. If you must do both, encrypt first, then compress (though this barely helps compression ratios).

**"Lossless can always compress anything"** — False. Random data has near-maximum entropy and cannot be compressed. Attempting to compress already-compressed or encrypted data often *increases* size slightly (due to headers).

**"Higher compression = better"** — Not always. Compression is compute-expensive. For low-latency applications (gaming, VoIP), lightweight or no compression beats high-ratio compression with encoding delay.

**Confusing codec with container:** MP4 is a container; H.264 is the codec inside it. MKV is a container; it might hold H.265, AV1, or VP9. These are distinct concepts.

---

## 12) Retrieval Prompts

- What is the difference between lossless and lossy compression, and how do you decide which to use?
- Why did TLS disable compression? What attack exploited it?
- How does Huffman coding work, and what property of data does it exploit?
- What is a decompression bomb, and how do you defend against it?
- Why does repeated JPEG compression degrade quality further each time?
- Where does compression sit in the OSI model vs the TCP/IP model?
- Why might compression *increase* total transmission time on a fast LAN?
- What are I-frames and P-frames, and why does losing an I-frame hurt so much?

---

## 13) TL;DR Compression *(pun intended)*

- Compression removes redundancy (lossless) or perceptually irrelevant information (lossy) to shrink data before transmission or storage.
- Lossless (gzip, PNG, FLAC) = perfect recovery, modest ratios. Lossy (JPEG, MP3, H.264) = irreversible, high ratios.
- Lives at OSI Layer 6 (Presentation), but in practice it's done inside application protocols — HTTP, TLS, codecs.
- Compressing before encrypting is a security vulnerability (CRIME/BREACH attacks). Don't do it.
- Random/encrypted data cannot be meaningfully compressed — you might even make it larger.