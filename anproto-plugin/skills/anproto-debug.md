---
name: anproto-debug
description: Use this skill when the user needs to "debug anproto message", "parse anproto", "explain message format", "decode keypair", "troubleshoot signature", or understand the structure of ANProto messages, keypairs, hashes, or signatures.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
---

# ANProto Message Debugger

Help users parse, understand, and troubleshoot ANProto messages and components.

## Message Component Reference

### Keypair (132 characters total)

```
Position   Content                Description
─────────────────────────────────────────────────
[0-43]     Public Key            44 chars, base64-encoded ed25519 public key
[44-131]   Secret Key            88 chars, base64-encoded ed25519 secret key
```

**Example:**
```
BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EFJjv96vhUki7Tyjf01pECI9j8wC93uhCGUYJEMAGNhQ==
├──────────────── Public Key (44) ───────────────┤├───────────────────────── Secret Key (88) ─────────────────────────┤
```

### Hash (44 characters)

```
Position   Content                Description
─────────────────────────────────────────────────
[0-43]     SHA-256 Hash          44 chars, base64-encoded SHA-256 of content
```

**Example:**
```
pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
├──────────────── SHA-256 Hash (44) ─────────────┤
```

### Signed Message (variable length)

```
Position   Content                Description
─────────────────────────────────────────────────
[0-43]     Public Key            44 chars, signer's public key
[44+]      Signed Payload        Variable, NaCl signed(timestamp + hash)
```

### Opened Message (57 characters)

```
Position   Content                Description
─────────────────────────────────────────────────
[0-12]     Timestamp             13 digits, Unix milliseconds (Date.now())
[13-56]    Hash                  44 chars, original content hash
```

**Example:**
```
1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
├─ Timestamp ─┤├──────────────── Hash (44) ─────────────────┤
```

## Debugging Guide

### How to Parse a Keypair

```javascript
function parseKeypair(keypair) {
  if (keypair.length !== 132) {
    return { error: `Invalid length: ${keypair.length} (expected 132)` }
  }

  return {
    publicKey: keypair.substring(0, 44),
    secretKey: keypair.substring(44),
    valid: true
  }
}
```

### How to Parse a Signed Message

```javascript
function parseSignedMessage(msg) {
  if (msg.length < 45) {
    return { error: `Message too short: ${msg.length} chars` }
  }

  return {
    publicKey: msg.substring(0, 44),
    signedPayload: msg.substring(44),
    payloadLength: msg.length - 44
  }
}
```

### How to Parse an Opened Message

```javascript
function parseOpenedMessage(opened) {
  if (opened.length !== 57) {
    return { error: `Invalid length: ${opened.length} (expected 57)` }
  }

  const timestamp = opened.substring(0, 13)
  const hash = opened.substring(13)

  return {
    timestamp: timestamp,
    timestampDate: new Date(parseInt(timestamp)),
    hash: hash,
    valid: true
  }
}
```

## Common Issues and Solutions

### Issue: `null` returned from `an.open()`

**Cause**: Signature verification failed.

**Debug steps**:
1. Check if the message was tampered with
2. Verify the public key matches the signer
3. Ensure the full message is being passed (no truncation)

```javascript
const opened = await an.open(signedMessage)
if (opened === null) {
  console.error('Signature verification failed')
  console.log('Public key in message:', signedMessage.substring(0, 44))
  console.log('Message length:', signedMessage.length)
}
```

### Issue: Hash mismatch

**Cause**: Content was modified after signing.

**Debug steps**:
```javascript
const originalHash = opened.substring(13)
const computedHash = await an.hash(content)

if (originalHash !== computedHash) {
  console.error('Content was modified!')
  console.log('Original hash:', originalHash)
  console.log('Computed hash:', computedHash)
}
```

### Issue: Keypair length wrong

**Cause**: Truncated or corrupted keypair.

**Debug steps**:
```javascript
console.log('Keypair length:', keypair.length)
console.log('Expected: 132')

// Check for common issues
if (keypair.includes('\n')) {
  console.error('Keypair contains newline characters')
}
if (keypair.includes(' ')) {
  console.error('Keypair contains spaces')
}
```

### Issue: Timestamp in the future

**Cause**: Clock skew between systems.

**Debug steps**:
```javascript
const timestamp = parseInt(opened.substring(0, 13))
const now = Date.now()
const diff = timestamp - now

if (diff > 0) {
  console.warn(`Message from ${diff}ms in the future`)
  console.log('Signer clock may be ahead')
}
```

## Visual Debugger

When a user provides a message to debug, break it down visually:

```
Input: BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=yVpD8i7d3d4...

Analysis:
┌─────────────────────────────────────────────────────────────┐
│ Component       │ Value                                     │
├─────────────────┼───────────────────────────────────────────┤
│ Type            │ Signed Message                            │
│ Length          │ 148 characters                            │
├─────────────────┼───────────────────────────────────────────┤
│ Public Key      │ BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRD... │
│ (chars 0-43)    │                                           │
├─────────────────┼───────────────────────────────────────────┤
│ Signed Payload  │ yVpD8i7d3d4dls3YThEg1x1vSdmqeEweV4e4Ej... │
│ (chars 44+)     │ (104 chars)                               │
└─────────────────┴───────────────────────────────────────────┘
```

## Quick Reference

| String Length | Likely Type |
|--------------|-------------|
| 44 | Hash OR Public Key |
| 57 | Opened message (timestamp + hash) |
| 132 | Full keypair |
| >44, variable | Signed message |

| First 13 chars | Type Hint |
|----------------|-----------|
| All digits (e.g., 1755197841319) | Opened message (starts with timestamp) |
| Base64 chars | Keypair or signed message (starts with public key) |
