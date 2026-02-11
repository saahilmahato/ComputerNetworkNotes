# OSI Model: Presentation Layer (Layer 6)

## Table of Contents
- [Overview](#overview)
- [Three Core Functions](#three-core-functions)
- [Data Translation](#data-translation)
- [Encryption & Decryption](#encryption--decryption)
- [Data Compression](#data-compression)
- [File Formats & Protocols](#file-formats--protocols)
- [Data Flow Examples](#data-flow-examples)
- [Security Considerations](#security-considerations)
- [Performance Analysis](#performance-analysis)
- [Code Examples](#code-examples)
- [Troubleshooting Guide](#troubleshooting-guide)

---

## Overview

```
┌─────────────────────────────────────────────────────────┐
│         PRESENTATION LAYER (Layer 6)                    │
│                                                         │
│  Primary Functions:                                     │
│  • Data Translation & Format Conversion                 │
│  • Encryption & Decryption                              │
│  • Data Compression & Decompression                     │
│                                                         │
│  Role: Data Translator, Encoder, and Security Provider  │
└─────────────────────────────────────────────────────────┘
```

**Key Purpose:** Ensures data from the application layer is properly formatted, secure, and efficient for network transmission.

---

## Three Core Functions

```
┌──────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ TRANSLATION  │  │  ENCRYPTION  │  │ COMPRESSION  │      │
│  │              │  │              │  │              │      │
│  │ ASCII ↔ UTF-8│  │  AES, RSA    │  │ GZIP, JPEG   │      │
│  │ Endianness   │  │  TLS/SSL     │  │ LZ77, MPEG   │      │
│  │ Serialization│  │  Certificates│  │ Huffman Code │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Function Comparison Table

| Function      | Purpose                          | Examples                    | When Used                  |
|---------------|----------------------------------|-----------------------------|----------------------------|
| Translation   | Format conversion                | ASCII→Unicode, JSON→Binary  | Cross-platform transfers   |
| Encryption    | Data security & confidentiality  | TLS/SSL, AES-256           | Sensitive data transmission|
| Compression   | Reduce data size                 | GZIP, JPEG, MP4            | Large file transfers       |

---

## Data Translation

### Character Encoding Conversion

```
┌─────────────────────────────────────────────────────────┐
│  SENDER (Windows - UTF-16)                              │
│  "Hello World" = 48 00 65 00 6C 00 6C 00 6F 00...       │
└─────────────────────────────────────────────────────────┘
                          ↓
              [PRESENTATION LAYER]
              Converts UTF-16 → UTF-8
                          ↓
┌─────────────────────────────────────────────────────────┐
│  RECEIVER (Linux - UTF-8)                               │
│  "Hello World" = 48 65 6C 6C 6F 20 57 6F 72 6C 64       │
└─────────────────────────────────────────────────────────┘
```

### Endianness Conversion

**Big-Endian vs Little-Endian:**

```
Number: 0x12345678

Big-Endian (Network Byte Order):
┌────┬────┬────┬────┐
│ 12 │ 34 │ 56 │ 78 │  (Most significant byte first)
└────┴────┴────┴────┘
Address: 0x00  0x01  0x02  0x03

Little-Endian (x86 processors):
┌────┬────┬────┬────┐
│ 78 │ 56 │ 34 │ 12 │  (Least significant byte first)
└────┴────┴────┴────┘
Address: 0x00  0x01  0x02  0x03
```

### Common Character Encodings

| Encoding | Bytes/Char | Character Range        | Use Case                  |
|----------|------------|------------------------|---------------------------|
| ASCII    | 1          | 0-127 (128 chars)      | Basic English text        |
| UTF-8    | 1-4        | All Unicode            | Web, modern systems       |
| UTF-16   | 2-4        | All Unicode            | Windows, Java             |
| UTF-32   | 4          | All Unicode            | Internal processing       |
| EBCDIC   | 1          | IBM mainframe chars    | Legacy IBM systems        |

### Code Example: Character Encoding Conversion

```python
# Python example of character encoding translation

# Original text in UTF-8
text_utf8 = "Hello, 世界!"  # "Hello, World!" in Chinese
print(f"UTF-8: {text_utf8.encode('utf-8')}")
# Output: b'Hello, \xe4\xb8\x96\xe7\x95\x8c!'

# Convert to UTF-16
text_utf16 = text_utf8.encode('utf-16')
print(f"UTF-16: {text_utf16}")
# Output: b'\xff\xfeH\x00e\x00l\x00l\x00o\x00,\x00 \x00\x16N\x8c.'

# Convert to Latin-1 (will fail for non-Latin characters)
try:
    text_latin1 = text_utf8.encode('latin-1')
except UnicodeEncodeError as e:
    print(f"Cannot encode to Latin-1: {e}")
```

```c
// C example: Network byte order conversion
#include <arpa/inet.h>

uint32_t host_number = 0x12345678;

// Convert to network byte order (big-endian)
uint32_t network_number = htonl(host_number);
printf("Host: 0x%x\n", host_number);      // 0x12345678
printf("Network: 0x%x\n", network_number); // Could be 0x78563412 on little-endian

// Convert back to host byte order
uint32_t converted_back = ntohl(network_number);
printf("Back to host: 0x%x\n", converted_back); // 0x12345678
```

---

## Encryption & Decryption

### Encryption Process Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    SENDING SYSTEM                            │
│                                                              │
│  Plain Text Data                                             │
│  "Confidential Message"                                      │
│          ↓                                                   │
│  ┌──────────────────┐                                        │
│  │ ENCRYPTION       │  Key: [Secret Key]                     │
│  │ Algorithm: AES   │                                        │
│  └──────────────────┘                                        │
│          ↓                                                   │
│  Encrypted Data (Ciphertext)                                 │
│  "8f3a9c2e7b1d..."                                           │
└──────────────────────────────────────────────────────────────┘
                          ↓
                  [Network Transfer]
                          ↓
┌──────────────────────────────────────────────────────────────┐
│                   RECEIVING SYSTEM                           │
│                                                              │
│  Encrypted Data                                              │
│  "8f3a9c2e7b1d..."                                           │
│          ↓                                                   │
│  ┌──────────────────┐                                        │
│  │ DECRYPTION       │  Key: [Secret Key]                     │
│  │ Algorithm: AES   │                                        │
│  └──────────────────┘                                        │
│          ↓                                                   │
│  Plain Text Data                                             │
│  "Confidential Message"                                      │
└──────────────────────────────────────────────────────────────┘
```

### Encryption Types Comparison

| Type       | Key Structure      | Speed  | Use Case           | Examples      |
|------------|-------------------|--------|---------------------|---------------|
| Symmetric  | Same key both ways| Fast   | Bulk data encryption| AES, DES, 3DES|
| Asymmetric | Public/Private pair| Slow  | Key exchange, auth  | RSA, ECC      |
| Hybrid     | Both types        | Optimal| SSL/TLS connections | TLS Handshake |

### Symmetric Encryption Example

```
┌────────────────────────────────────────────┐
│         SYMMETRIC ENCRYPTION               │
│                                            │
│  Sender              Key: K123             │
│    │                                       │
│    │ Encrypt with K123                     │
│    ↓                                       │
│  Ciphertext ═══════════════════════════→   │
│                                       ↓    │
│                           Decrypt with K123│
│                                     ↓      │
│                           Receiver         │
│                                            │
│  ⚠️  Same key must be shared securely!     │
└────────────────────────────────────────────┘
```

### Asymmetric Encryption Example

```
┌────────────────────────────────────────────────────────┐
│           ASYMMETRIC ENCRYPTION                        │
│                                                        │
│  Sender                          Receiver             │
│    │                                │                 │
│    │  Encrypt with                 │ Private Key      │
│    │  Receiver's Public Key        │ (Secret)         │
│    │                               │                  │
│    ↓                               ↓                  │
│  Ciphertext ═══════════════════════→                  │
│                                    │                  │
│                     Decrypt with   │                  │
│                     Private Key    │                  │
│                                    ↓                  │
│                                  Plaintext            │
│                                                        │
│  ✓ No shared secret needed!                           │
│  ✓ Public key can be distributed openly               │
└────────────────────────────────────────────────────────┘
```

### TLS/SSL Handshake Diagram

```
CLIENT                                              SERVER
  │                                                   │
  │──────── 1. ClientHello ──────────────────────────→│
  │    (Supported ciphers, TLS version)               │
  │                                                   │
  │←─────── 2. ServerHello ──────────────────────────│
  │    (Selected cipher, certificate)                 │
  │                                                   │
  │←─────── 3. Certificate ──────────────────────────│
  │    (Server's public key + CA signature)           │
  │                                                   │
  │──────── 4. Key Exchange ─────────────────────────→│
  │    (Encrypted pre-master secret)                  │
  │                                                   │
  │──────── 5. Finished ─────────────────────────────→│
  │    (Encrypted with session key)                   │
  │                                                   │
  │←─────── 6. Finished ─────────────────────────────│
  │    (Encrypted with session key)                   │
  │                                                   │
  │═══════ ENCRYPTED DATA TRANSFER ══════════════════│
```

### Code Example: Encryption/Decryption

```python
# Python example using AES encryption
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

# AES requires 16-byte blocks
key = get_random_bytes(16)  # 128-bit key
cipher = AES.new(key, AES.MODE_CBC)

# Encryption
plaintext = b"Confidential Message"
ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
iv = cipher.iv  # Initialization vector

print(f"Plain: {plaintext}")
print(f"Encrypted: {ciphertext.hex()}")

# Decryption
decipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = unpad(decipher.decrypt(ciphertext), AES.block_size)
print(f"Decrypted: {decrypted}")
```

```javascript
// Node.js example using crypto module
const crypto = require('crypto');

// Encryption function
function encrypt(text, password) {
    const algorithm = 'aes-256-cbc';
    const key = crypto.scryptSync(password, 'salt', 32);
    const iv = crypto.randomBytes(16);
    
    const cipher = crypto.createCipheriv(algorithm, key, iv);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return {
        iv: iv.toString('hex'),
        content: encrypted
    };
}

// Decryption function
function decrypt(encrypted, password) {
    const algorithm = 'aes-256-cbc';
    const key = crypto.scryptSync(password, 'salt', 32);
    const iv = Buffer.from(encrypted.iv, 'hex');
    
    const decipher = crypto.createDecipheriv(algorithm, key, iv);
    let decrypted = decipher.update(encrypted.content, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
}

// Usage
const message = "Secret Data";
const password = "mySecurePassword123";

const encrypted = encrypt(message, password);
console.log("Encrypted:", encrypted);

const decrypted = decrypt(encrypted, password);
console.log("Decrypted:", decrypted);
```

### Common Encryption Algorithms

| Algorithm | Type       | Key Size       | Speed    | Security Level | Use Case              |
|-----------|------------|----------------|----------|----------------|-----------------------|
| AES       | Symmetric  | 128/192/256    | Very Fast| Very High      | Bulk encryption       |
| 3DES      | Symmetric  | 168 bits       | Slow     | Moderate       | Legacy systems        |
| RSA       | Asymmetric | 1024-4096 bits | Slow     | High           | Key exchange          |
| ECC       | Asymmetric | 256-521 bits   | Fast     | Very High      | Mobile devices        |
| ChaCha20  | Symmetric  | 256 bits       | Very Fast| Very High      | Modern applications   |

---

## Data Compression

### Compression Flow Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  ORIGINAL DATA: 1,000,000 bytes                              │
│  "AAAABBBBCCCCDDDD..." (Repetitive text/data)                │
└──────────────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────────────┐
│              COMPRESSION ALGORITHM                           │
│  ┌────────────────┐    ┌────────────────┐                   │
│  │  Lossless      │    │  Lossy         │                   │
│  │  (GZIP, ZIP)   │    │  (JPEG, MP3)   │                   │
│  │  • Full data   │    │  • Reduced     │                   │
│  │    recovery    │    │    quality     │                   │
│  └────────────────┘    └────────────────┘                   │
└──────────────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────────────┐
│  COMPRESSED DATA: 200,000 bytes (80% reduction)              │
│  "4A4B4C4D..." (Encoded representation)                      │
└──────────────────────────────────────────────────────────────┘
                          ↓
                  [Network Transfer]
                          ↓
┌──────────────────────────────────────────────────────────────┐
│  DECOMPRESSION AT RECEIVER                                   │
│  Restores original data (lossless)                           │
│  or approximation (lossy)                                    │
└──────────────────────────────────────────────────────────────┘
```

### Lossless vs Lossy Compression

```
┌─────────────────────────────────────────────────────────────┐
│              LOSSLESS COMPRESSION                           │
│                                                             │
│  Original: "AAAABBBBCCCC"                                   │
│      ↓                                                      │
│  Compressed: "4A4B4C" (run-length encoding)                 │
│      ↓                                                      │
│  Decompressed: "AAAABBBBCCCC" ✓ EXACT MATCH                │
│                                                             │
│  Use for: Text, Code, Executables, Databases                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               LOSSY COMPRESSION                             │
│                                                             │
│  Original Image: 10 MB, High Quality                        │
│      ↓                                                      │
│  Compressed: 500 KB, Good Quality                           │
│      ↓                                                      │
│  Decompressed: Similar but NOT exact ⚠️                     │
│                                                             │
│  Use for: Images, Audio, Video (where perfect               │
│           reproduction is not required)                     │
└─────────────────────────────────────────────────────────────┘
```

### Compression Algorithms Comparison

| Algorithm    | Type     | Ratio    | Speed      | Best For                  |
|--------------|----------|----------|------------|---------------------------|
| GZIP/DEFLATE | Lossless | 60-70%   | Fast       | Web, text files           |
| Brotli       | Lossless | 70-80%   | Medium     | Web assets                |
| LZ4          | Lossless | 50%      | Very Fast  | Real-time compression     |
| JPEG         | Lossy    | 90-95%   | Fast       | Photos                    |
| PNG          | Lossless | 40-60%   | Medium     | Graphics, screenshots     |
| MP3          | Lossy    | 90%      | Fast       | Music                     |
| H.264        | Lossy    | 95%      | Medium     | Video streaming           |

### Code Example: Data Compression

```python
# Python compression examples
import gzip
import zlib

# Original data
original_data = b"This is a test. " * 100  # Repetitive data
print(f"Original size: {len(original_data)} bytes")

# GZIP compression
compressed_gzip = gzip.compress(original_data)
print(f"GZIP compressed: {len(compressed_gzip)} bytes")
print(f"Compression ratio: {len(compressed_gzip)/len(original_data)*100:.2f}%")

# Decompression
decompressed = gzip.decompress(compressed_gzip)
assert decompressed == original_data
print("✓ Decompression successful!")

# ZLIB compression (faster)
compressed_zlib = zlib.compress(original_data)
print(f"ZLIB compressed: {len(compressed_zlib)} bytes")
```

```javascript
// JavaScript compression example (Node.js)
const zlib = require('zlib');

const originalData = Buffer.from('This is test data. '.repeat(100));

// Compress
zlib.gzip(originalData, (err, compressed) => {
    console.log(`Original: ${originalData.length} bytes`);
    console.log(`Compressed: ${compressed.length} bytes`);
    console.log(`Ratio: ${(compressed.length/originalData.length*100).toFixed(2)}%`);
    
    // Decompress
    zlib.gunzip(compressed, (err, decompressed) => {
        console.log(`Decompressed: ${decompressed.length} bytes`);
        console.log(`Match: ${decompressed.equals(originalData)}`);
    });
});
```

### Compression Performance Chart

```
File Type vs Compression Efficiency

Text Files:     ████████████████████ 80%
HTML/CSS:       ███████████████████  75%
JSON/XML:       ██████████████████   70%
JavaScript:     ████████████████     65%
Images (PNG):   ██████████           40%
Images (JPEG):  ██                   10% (already compressed)
Video:          █                    5%  (already compressed)
Binary/Exec:    ████████             30%
```

---

## File Formats & Protocols

### Data Format Categories

```
┌──────────────────────────────────────────────────────────────┐
│               FILE FORMATS BY CATEGORY                       │
└──────────────────────────────────────────────────────────────┘

TEXT & DOCUMENTS:
┌─────────┬──────────┬────────────┬───────────────────────────┐
│ Format  │ Type     │ Compressed │ Use Case                  │
├─────────┼──────────┼────────────┼───────────────────────────┤
│ TXT     │ Plain    │ No         │ Simple text               │
│ RTF     │ Rich     │ No         │ Formatted documents       │
│ PDF     │ Document │ Yes        │ Universal documents       │
│ DOCX    │ Document │ Yes        │ Word processing           │
│ HTML    │ Markup   │ No         │ Web pages                 │
│ XML     │ Markup   │ No         │ Data structure            │
│ JSON    │ Data     │ No         │ API responses             │
└─────────┴──────────┴────────────┴───────────────────────────┘

IMAGES:
┌─────────┬──────────┬────────────┬───────────────────────────┐
│ Format  │ Type     │ Compressed │ Use Case                  │
├─────────┼──────────┼────────────┼───────────────────────────┤
│ JPEG    │ Lossy    │ Yes        │ Photos                    │
│ PNG     │ Lossless │ Yes        │ Graphics, transparency    │
│ GIF     │ Lossless │ Yes        │ Simple animations         │
│ BMP     │ None     │ No         │ Raw bitmaps               │
│ TIFF    │ Both     │ Optional   │ Professional photography  │
│ WebP    │ Both     │ Yes        │ Modern web images         │
│ SVG     │ Vector   │ No         │ Scalable graphics         │
└─────────┴──────────┴────────────┴───────────────────────────┘

AUDIO:
┌─────────┬──────────┬────────────┬───────────────────────────┐
│ Format  │ Type     │ Compressed │ Use Case                  │
├─────────┼──────────┼────────────┼───────────────────────────┤
│ MP3     │ Lossy    │ Yes        │ Music (general)           │
│ AAC     │ Lossy    │ Yes        │ Modern music, streaming   │
│ FLAC    │ Lossless │ Yes        │ High-quality audio        │
│ WAV     │ None     │ No         │ Professional audio        │
│ OGG     │ Lossy    │ Yes        │ Open-source audio         │
└─────────┴──────────┴────────────┴───────────────────────────┘

VIDEO:
┌─────────┬──────────┬────────────┬───────────────────────────┐
│ Format  │ Codec    │ Compressed │ Use Case                  │
├─────────┼──────────┼────────────┼───────────────────────────┤
│ MP4     │ H.264    │ Yes        │ Universal video           │
│ WebM    │ VP9      │ Yes        │ Web video                 │
│ AVI     │ Various  │ Optional   │ Legacy video              │
│ MKV     │ Various  │ Yes        │ High-quality video        │
│ MOV     │ Various  │ Yes        │ Apple ecosystem           │
└─────────┴──────────┴────────────┴───────────────────────────┘
```

### Common Protocols

```
┌──────────────────────────────────────────────────────────────┐
│           PRESENTATION LAYER PROTOCOLS                       │
└──────────────────────────────────────────────────────────────┘

SSL/TLS (Secure Sockets Layer / Transport Layer Security)
├─ Purpose: Encryption for secure communication
├─ Versions: TLS 1.2, TLS 1.3 (recommended)
├─ Uses: HTTPS, FTPS, SMTPS, VPN
└─ Port: Depends on application (443 for HTTPS)

MIME (Multipurpose Internet Mail Extensions)
├─ Purpose: Email attachments and content types
├─ Functions: Encodes non-text data for email
├─ Uses: Email, HTTP headers
└─ Example: Content-Type: image/jpeg

XDR (External Data Representation)
├─ Purpose: Platform-independent data format
├─ Functions: Data serialization
├─ Uses: RPC (Remote Procedure Calls), NFS
└─ Features: Big-endian byte order

ASN.1 (Abstract Syntax Notation One)
├─ Purpose: Define data structures
├─ Encoding: BER, DER, PER, XER
├─ Uses: SSL certificates, SNMP, LDAP
└─ Features: Platform and language independent
```

### MIME Types Reference

```
Common MIME Content Types:

TEXT TYPES:
text/plain              - Plain text file
text/html               - HTML document
text/css                - CSS stylesheet
text/javascript         - JavaScript code
text/csv                - CSV data file

IMAGE TYPES:
image/jpeg              - JPEG image
image/png               - PNG image
image/gif               - GIF image
image/svg+xml           - SVG vector graphic
image/webp              - WebP image

APPLICATION TYPES:
application/json        - JSON data
application/xml         - XML data
application/pdf         - PDF document
application/zip         - ZIP archive
application/octet-stream - Binary data

AUDIO TYPES:
audio/mpeg              - MP3 audio
audio/wav               - WAV audio
audio/ogg               - OGG audio

VIDEO TYPES:
video/mp4               - MP4 video
video/webm              - WebM video
video/quicktime         - QuickTime video
```

---

## Data Flow Examples

### Example 1: HTTPS Web Browsing

```
┌──────────────────────────────────────────────────────────────┐
│  USER → BROWSER → HTTPS → WEB SERVER                         │
└──────────────────────────────────────────────────────────────┘

STEP 1: User Types URL
┌──────────┐
│ Browser  │  User enters: https://example.com
└──────────┘
     ↓

STEP 2: Application Layer (Layer 7)
┌──────────┐
│  HTTP    │  GET /index.html HTTP/1.1
│ Request  │  Host: example.com
└──────────┘
     ↓

STEP 3: Presentation Layer (Layer 6) ← OUR FOCUS
┌──────────────────────────────────────────┐
│ ┌─────────────────────────────────────┐  │
│ │ TLS/SSL Encryption                  │  │
│ │ • Cipher: AES-256-GCM               │  │
│ │ • Protocol: TLS 1.3                 │  │
│ │ • Certificate verified              │  │
│ └─────────────────────────────────────┘  │
│ ┌─────────────────────────────────────┐  │
│ │ Data Compression                    │  │
│ │ • Algorithm: Brotli                 │  │
│ │ • Original: 50KB → Compressed: 12KB │  │
│ └─────────────────────────────────────┘  │
│ ┌─────────────────────────────────────┐  │
│ │ Character Encoding                  │  │
│ │ • Format: UTF-8                     │  │
│ │ • Handles international characters  │  │
│ └─────────────────────────────────────┘  │
└──────────────────────────────────────────┘
     ↓

STEP 4-7: Lower Layers Handle Transport & Delivery
┌──────────┐
│ Session  │  Establishes connection
│ Transport│  TCP ensures delivery
│ Network  │  IP routing
│ Data Link│  Frame transmission
│ Physical │  Bits on wire
└──────────┘
     ↓

SERVER RECEIVES AND REVERSES THE PROCESS
┌──────────────────────────────────────────┐
│  Layer 6 at Server:                      │
│  1. Decompress Brotli data               │
│  2. Decrypt TLS encrypted data           │
│  3. Decode UTF-8 characters              │
│  4. Pass to web server application       │
└──────────────────────────────────────────┘
```

### Example 2: Sending Email with Attachment

```
┌──────────────────────────────────────────────────────────────┐
│  EMAIL WITH IMAGE ATTACHMENT WORKFLOW                        │
└──────────────────────────────────────────────────────────────┘

CLIENT SIDE:
┌─────────────────────────────────────────┐
│ Email Client (Outlook, Gmail)           │
│                                         │
│ To: user@example.com                    │
│ Subject: Vacation Photos                │
│ Body: "Check out these photos!"         │
│ Attachment: photo.jpg (2.5 MB)          │
└─────────────────────────────────────────┘
                ↓
┌─────────────────────────────────────────┐
│ PRESENTATION LAYER PROCESSING           │
│                                         │
│ 1. TEXT ENCODING:                       │
│    "Check out..." → UTF-8 encoding      │
│                                         │
│ 2. MIME ENCODING:                       │
│    ┌───────────────────────────────┐   │
│    │ Content-Type: multipart/mixed │   │
│    │                               │   │
│    │ --boundary123                 │   │
│    │ Content-Type: text/plain      │   │
│    │ Check out these photos!       │   │
│    │                               │   │
│    │ --boundary123                 │   │
│    │ Content-Type: image/jpeg      │   │
│    │ Content-Transfer-Encoding:    │   │
│    │   base64                      │   │
│    │ /9j/4AAQSkZJRgABAQEA...      │   │
│    │ --boundary123--               │   │
│    └───────────────────────────────┘   │
│                                         │
│ 3. OPTIONAL ENCRYPTION (S/MIME):        │
│    Entire message encrypted with        │
│    recipient's public key               │
└─────────────────────────────────────────┘
                ↓
         [Network Transfer]
                ↓
┌─────────────────────────────────────────┐
│ SERVER RECEIVES AND PROCESSES           │
│                                         │
│ 1. Decrypt if encrypted                 │
│ 2. Parse MIME structure                 │
│ 3. Decode base64 attachment             │
│ 4. Decode UTF-8 text                    │
│ 5. Store in mailbox                     │
└─────────────────────────────────────────┘
```

---

## Code Examples

### Complete HTTPS Server (Python)

```python
#!/usr/bin/env python3
"""
HTTPS Server with TLS, Compression, and Character Encoding
Demonstrates Presentation Layer functions
"""

from http.server import HTTPServer, SimpleHTTPRequestHandler
import ssl
import gzip
import json

class PresentationLayerHandler(SimpleHTTPRequestHandler):
    """HTTP Handler with presentation layer features"""
    
    def do_GET(self):
        """Handle GET requests with compression and encoding"""
        
        # Prepare response data
        data = {
            "message": "Hello from Presentation Layer!",
            "features": ["TLS Encryption", "GZIP Compression", "UTF-8 Encoding"],
            "timestamp": "2026-02-11T10:30:00Z"
        }
        
        # FUNCTION 1: Data Translation (JSON serialization)
        response_json = json.dumps(data, ensure_ascii=False)
        
        # FUNCTION 2: Character Encoding (UTF-8)
        response_bytes = response_json.encode('utf-8')
        
        # FUNCTION 3: Compression (if client supports it)
        accept_encoding = self.headers.get('Accept-Encoding', '')
        if 'gzip' in accept_encoding:
            # Compress the response
            response_bytes = gzip.compress(response_bytes)
            self.send_response(200)
            self.send_header('Content-Type', 'application/json; charset=utf-8')
            self.send_header('Content-Encoding', 'gzip')
            self.send_header('Content-Length', len(response_bytes))
            self.end_headers()
            
            print(f"[Compression] Original: {len(response_json)} bytes, "
                  f"Compressed: {len(response_bytes)} bytes "
                  f"({100 - len(response_bytes)/len(response_json)*100:.1f}% reduction)")
        else:
            self.send_response(200)
            self.send_header('Content-Type', 'application/json; charset=utf-8')
            self.send_header('Content-Length', len(response_bytes))
            self.end_headers()
        
        self.wfile.write(response_bytes)

def run_https_server(port=8443):
    """Run HTTPS server with TLS encryption"""
    
    server_address = ('', port)
    httpd = HTTPServer(server_address, PresentationLayerHandler)
    
    # FUNCTION 4: Encryption (TLS/SSL)
    # Note: You need to generate certificate and key first:
    # openssl req -new -x509 -keyout server.key -out server.crt -days 365 -nodes
    
    context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
    context.load_cert_chain('server.crt', 'server.key')
    
    # Configure strong ciphers
    context.set_ciphers('ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS')
    
    # Enable TLS 1.2 and 1.3 only
    context.minimum_version = ssl.TLSVersion.TLSv1_2
    
    httpd.socket = context.wrap_socket(httpd.socket, server_side=True)
    
    print(f"[TLS] Server running on https://localhost:{port}")
    print(f"[Features] Encryption: TLS 1.2+, Compression: GZIP, Encoding: UTF-8")
    print("[Info] Press Ctrl+C to stop")
    
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\n[Info] Server stopped")

if __name__ == '__main__':
    run_https_server()
```

---

## Troubleshooting Guide

### Common Issues and Solutions

```bash
#!/bin/bash
# Presentation Layer Debugging Toolkit

echo "PRESENTATION LAYER DEBUGGING TOOLS"
echo "==================================="

# 1. SSL/TLS Testing
echo -e "\n1. Test SSL/TLS Connection:"
echo "   $ openssl s_client -connect example.com:443"
echo "   $ openssl s_client -connect example.com:443 -tls1_3"

# 2. Certificate Information
echo -e "\n2. View Certificate Details:"
echo "   $ openssl x509 -in cert.pem -text -noout"
echo "   $ openssl x509 -in cert.pem -noout -fingerprint"

# 3. Encryption/Decryption Testing
echo -e "\n3. Test Encryption:"
echo "   # Encrypt"
echo "   $ openssl enc -aes-256-cbc -in plain.txt -out encrypted.bin"
echo "   # Decrypt"
echo "   $ openssl enc -d -aes-256-cbc -in encrypted.bin -out plain.txt"

# 4. Compression Testing
echo -e "\n4. Test Compression:"
echo "   $ gzip -9 file.txt"
echo "   $ gunzip file.txt.gz"
echo "   $ file compressed.gz"

# 5. Character Encoding
echo -e "\n5. Test Character Encoding:"
echo "   $ file -i filename.txt"
echo "   $ iconv -f UTF-8 -t ISO-8859-1 in.txt -o out.txt"
```

---

## Summary

The Presentation Layer (Layer 6) serves as the data translator and security provider of the OSI model.

**Key Functions:**
- **Translation**: Data format conversion
- **Encryption**: TLS/SSL security
- **Compression**: Bandwidth optimization