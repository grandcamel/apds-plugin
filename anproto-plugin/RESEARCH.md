# ANProto Deep Research

## Overview

**ANProto** = **A**uthenticated and **N**on-networked Protocol (or **AN**other Protocol)

A lightweight decentralized authentication and messaging protocol using ed25519 keypairs to sign timestamp + hash in base64. Designed as the spiritual successor to secure-scuttlebot without the maintenance burden.

**Repository**: https://github.com/evbogue/anproto

---

## Core Implementation Analysis (JavaScript)

### File Structure
```
anproto/
├── an.js              # Core library (37 lines!)
├── lib/
│   ├── base64.js      # Base64 encode/decode (Deno standard library)
│   └── nacl-fast-es.js # TweetNaCl ES module (ed25519 crypto)
├── ex.js              # Deno example
├── node_ex.js         # Node.js example (with crypto polyfills)
├── serve.js           # Hono web server
├── try_app.js         # Interactive demo app
├── template.js        # HTML templates
└── style.css          # Styling
```

### Core API (an.js)

| Function | Signature | Purpose |
|----------|-----------|---------|
| `an.gen()` | `async () => string` | Generate ed25519 keypair |
| `an.hash(data)` | `async (string) => string` | SHA-256 hash of content |
| `an.sign(hash, keypair)` | `async (string, string) => string` | Sign hash with timestamp |
| `an.open(message)` | `async (string) => string` | Verify and extract payload |

### Detailed Function Analysis

#### `an.gen()` - Keypair Generation
```javascript
an.gen = async () => {
  const g = await nacl.sign.keyPair();
  const k = await encode(g.publicKey) + encode(g.secretKey);
  return k;
};
```
- Uses TweetNaCl's `sign.keyPair()` for ed25519
- Returns concatenated base64: `publicKey (44 chars) + secretKey (88 chars)`
- Total length: **132 characters**

#### `an.hash(data)` - Content Hashing
```javascript
an.hash = async (d) => {
  return encode(
    Array.from(
      new Uint8Array(
        await crypto.subtle.digest("SHA-256", new TextEncoder().encode(d)),
      ),
    ),
  );
};
```
- Uses Web Crypto API for SHA-256
- Returns base64-encoded hash (44 characters)

#### `an.sign(hash, keypair)` - Message Signing
```javascript
an.sign = async (h, k) => {
  const ts = Date.now();
  const s = encode(
    nacl.sign(new TextEncoder().encode(ts + h), decode(k.substring(44))),
  );
  return k.substring(0, 44) + s;
};
```
- Prepends Unix timestamp (milliseconds) to hash
- Signs with secret key (extracted from position 44+)
- Returns: `publicKey (44 chars) + signedPayload`

#### `an.open(message)` - Verify & Extract
```javascript
an.open = async (m) => {
  const o = new TextDecoder().decode(
    nacl.sign.open(decode(m.substring(44)), decode(m.substring(0, 44))),
  );
  return o;
};
```
- Extracts public key from first 44 chars
- Verifies signature using NaCl
- Returns: `timestamp + hash` (e.g., `1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=`)

---

## Message Format Specification

### Keypair Format
```
[publicKey:44][secretKey:88] = 132 characters total
```
Example:
```
BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EFJjv96vhUki7Tyjf01pECI9j8wC93uhCGUYJEMAGNhQ==
```

### Hash Format
```
[sha256-base64:44]
```
Example:
```
pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
```

### Signed Message Format
```
[publicKey:44][signedPayload:variable]
```

### Opened Message Format
```
[timestamp:13][hash:44]
```
Example:
```
1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
```

---

## Developer Workflow

### Standard Flow
```javascript
import { an } from './an.js'

// 1. Generate keypair (once, store securely)
const keypair = await an.gen()

// 2. Create content and hash it
const content = "Hello World"
const hash = await an.hash(content)

// 3. Sign the hash
const signedMessage = await an.sign(hash, keypair)

// 4. Verify and open (recipient)
const opened = await an.open(signedMessage)

// 5. Extract timestamp and hash from opened message
const timestamp = opened.substring(0, 13)
const extractedHash = opened.substring(13)

// 6. Verify content matches hash
const contentHash = await an.hash(content)
const isValid = (extractedHash === contentHash)
```

### Node.js Compatibility
Requires crypto polyfills:
```javascript
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

globalThis.self = {
  crypto: {
    getRandomValues: (buf) => {
      require('crypto').randomFillSync(buf);
      return buf;
    }
  }
};

globalThis.crypto = globalThis.crypto || {};
globalThis.crypto.subtle = require('crypto').webcrypto.subtle;
```

---

## Cross-Language Implementations

### Go (goan)
- Repository: https://github.com/vic/goan
- License: Apache 2.0
- Features interoperability tests with JavaScript
- Functions: `Gen()`, `Hash()`, `Sign()`, `Open()`

### Rust (anproto-rs)
- Repository: https://github.com/vic/anproto-rs
- Available on crates.io
- License: Apache 2.0
- Functions: `gen()`, `hash()`, `sign()`, `open()`

### Python (ANproto-Python)
- Repository: https://github.com/macauleyjustin/ANproto-Python
- Functions mirror the JavaScript API

---

## Ecosystem Components

### APDS (A Personal Data Server)
Server component that:
- [x] Generates and saves keypairs
- [x] Composes messages
- [x] Verifies messages
- [x] Adds messages to DB
- [x] Queries and searches DB
- [ ] WebSocket server
- [ ] HTTP server
- [ ] Web 2.0 login with server-held keys

### Wiredove
Proof of concept PWA client:
- Uses Trystero and WebSockets for networking
- Client-side hash router
- Profile pages, chronological feed
- Post composer

---

## Dependencies

| Dependency | Purpose | Size |
|------------|---------|------|
| TweetNaCl (nacl-fast-es.js) | Ed25519 signing | ~45KB |
| base64.js | Encoding/decoding | ~3KB |
| Web Crypto API | SHA-256 hashing | Browser built-in |
| Hono | Web server (Deno) | External |

---

## Developer Pain Points (Identified)

1. **Node.js Setup Friction** - Requires crypto polyfills
2. **Key Management** - No built-in secure storage guidance
3. **Message Parsing** - Manual string slicing by character positions
4. **Content Association** - Hash-to-content mapping left to developer
5. **Error Handling** - Minimal error messages in core library
6. **Documentation** - Sparse, relies on code examples

---

## Plugin Opportunities

### High Value
1. **Scaffold ANProto projects** with proper setup for Deno/Node/Browser
2. **Key management guidance** and secure storage patterns
3. **Message debugging** - parse and explain message components
4. **Cross-language code generation** for Go/Rust/Python

### Medium Value
1. **APDS setup assistance**
2. **Wiredove integration help**
3. **Transport layer examples** (WebSocket, fetch, etc.)

### Nice to Have
1. **Interactive message explorer**
2. **Signature verification troubleshooting**
3. **Migration guides** from Scuttlebot/Nostr

---

## Technical Notes

### Character Position Reference
| Position | Content | Length |
|----------|---------|--------|
| 0-43 | Public Key | 44 chars |
| 44-131 | Secret Key | 88 chars |
| 0-12 | Timestamp | 13 chars |
| 13+ | Hash | 44 chars |

### Timestamp Format
- Unix milliseconds (Date.now())
- 13 digits as of 2025
- Example: `1755197841319`

---

## Next Steps for Plugin Development

1. Define skill triggers based on developer workflows
2. Create scaffolding templates for each runtime
3. Build message parsing/debugging utilities
4. Document common error patterns and solutions
