---
name: anproto-sign
description: Use this skill when the user asks to "sign message", "create signature", "anproto sign", "sign content", or needs help signing content with ANProto including the hash-then-sign workflow.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
---

# ANProto Message Signing

Guide users through the ANProto signing workflow: hash content, then sign the hash.

## Signing Workflow Overview

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│  Content    │───▶│  an.hash()   │───▶│  an.sign()  │
│ "Hello..."  │    │  SHA-256     │    │  + timestamp│
└─────────────┘    └──────────────┘    └─────────────┘
                         │                    │
                         ▼                    ▼
                   ┌──────────┐        ┌──────────────┐
                   │  Hash    │        │ Signed Msg   │
                   │ (44 ch)  │        │ (pubkey+sig) │
                   └──────────┘        └──────────────┘
```

## Basic Signing

### Step 1: Hash the Content

```javascript
import { an } from './an.js'

const content = "Hello, World!"
const hash = await an.hash(content)

console.log('Content hash:', hash)
// e.g., "pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4="
```

### Step 2: Sign the Hash

```javascript
const keypair = await an.gen()  // or load existing keypair
const signedMessage = await an.sign(hash, keypair)

console.log('Signed message:', signedMessage)
// e.g., "BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=yVpD8i7d3d4..."
```

## Complete Example

```javascript
import { an } from './an.js'

async function signContent(content, keypair) {
  // 1. Hash the content
  const hash = await an.hash(content)

  // 2. Sign the hash (includes timestamp automatically)
  const signed = await an.sign(hash, keypair)

  return {
    content: content,
    hash: hash,
    signed: signed,
    publicKey: keypair.substring(0, 44),
    timestamp: Date.now()
  }
}

// Usage
const keypair = await an.gen()
const result = await signContent("My important message", keypair)

console.log('Signed by:', result.publicKey)
console.log('Signed message:', result.signed)
```

## What Gets Signed

The `an.sign()` function signs:
```
timestamp + hash
```

For example:
```
1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
└─ timestamp ─┘└───────────────── hash ─────────────────┘
```

This means:
- The signature proves WHO signed (ed25519 public key)
- The signature proves WHEN they signed (timestamp)
- The signature proves WHAT they signed (content hash)

## Signing Different Content Types

### Sign Text

```javascript
const text = "Plain text message"
const hash = await an.hash(text)
const signed = await an.sign(hash, keypair)
```

### Sign JSON

```javascript
const data = { user: "alice", action: "post", content: "Hello" }
const json = JSON.stringify(data)
const hash = await an.hash(json)
const signed = await an.sign(hash, keypair)
```

### Sign Files (Node.js/Deno)

```javascript
import { readFileSync } from 'fs'

const fileContent = readFileSync('document.pdf', 'utf8')
const hash = await an.hash(fileContent)
const signed = await an.sign(hash, keypair)
```

### Sign Binary Data

```javascript
// Convert binary to base64 first
const binaryData = new Uint8Array([...])
const base64 = btoa(String.fromCharCode(...binaryData))
const hash = await an.hash(base64)
const signed = await an.sign(hash, keypair)
```

## Sending Signed Messages

The recipient needs:
1. **The signed message** (contains public key + signature)
2. **The original content** (to verify the hash)

### Pattern 1: Inline Content

```javascript
// Package together
const package = {
  content: originalContent,
  signature: signedMessage
}

// Send as JSON
const payload = JSON.stringify(package)
```

### Pattern 2: Content-Addressed Storage

```javascript
// Store content separately using hash as key
const hash = await an.hash(content)
await storage.put(hash, content)

// Send only the signature (recipient fetches content by hash)
send(signedMessage)
```

### Pattern 3: URL Parameter

```javascript
// Encode for URL
const encoded = encodeURIComponent(signedMessage)
const url = `https://example.com/verify?sig=${encoded}`
```

## Batch Signing

Sign multiple items efficiently:

```javascript
async function signBatch(items, keypair) {
  const results = []

  for (const item of items) {
    const hash = await an.hash(JSON.stringify(item))
    const signed = await an.sign(hash, keypair)
    results.push({
      item: item,
      hash: hash,
      signed: signed
    })
  }

  return results
}
```

## Error Handling

```javascript
async function safeSigning(content, keypair) {
  try {
    // Validate keypair
    if (!keypair || keypair.length !== 132) {
      throw new Error('Invalid keypair')
    }

    // Validate content
    if (content === undefined || content === null) {
      throw new Error('Content is required')
    }

    const hash = await an.hash(String(content))
    const signed = await an.sign(hash, keypair)

    return { success: true, signed, hash }

  } catch (error) {
    return { success: false, error: error.message }
  }
}
```

## Quick Reference

| Task | Code |
|------|------|
| Hash content | `await an.hash(content)` |
| Sign hash | `await an.sign(hash, keypair)` |
| Get public key | `keypair.substring(0, 44)` |
| Get timestamp | `Date.now()` (also embedded in signature) |

## Output Format

After signing, the message format is:
```
[publicKey:44][signedPayload:variable]

Where signedPayload = NaCl.sign(timestamp + hash, secretKey)
```
