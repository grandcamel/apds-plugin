---
description: Use this skill when the user asks to "verify message", "check signature", "anproto open", "validate signature", or needs help verifying ANProto signed messages and extracting their contents.
---

# ANProto Message Verification

Guide users through verifying ANProto signed messages using `an.open()`.

## Verification Overview

```
┌────────────────┐    ┌──────────────┐    ┌─────────────┐
│ Signed Message │───▶│  an.open()   │───▶│  Opened     │
│ (pubkey+sig)   │    │  Verify sig  │    │ timestamp+  │
└────────────────┘    └──────────────┘    │ hash        │
                                          └─────────────┘
                                                │
                                                ▼
                                          ┌─────────────┐
                                          │ Compare to  │
                                          │ an.hash()   │
                                          │ of content  │
                                          └─────────────┘
```

## Basic Verification

### Step 1: Open the Signed Message

```javascript
import { an } from './an.js'

const signedMessage = "BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=yVpD8i7d3d4..."
const opened = await an.open(signedMessage)

if (opened === null) {
  console.error('Signature verification FAILED')
} else {
  console.log('Signature is VALID')
  console.log('Timestamp:', opened.substring(0, 13))
  console.log('Hash:', opened.substring(13))
}
```

### Step 2: Verify Content Matches Hash

```javascript
const originalContent = "Hello, World!"
const contentHash = await an.hash(originalContent)
const messageHash = opened.substring(13)

if (contentHash === messageHash) {
  console.log('Content integrity VERIFIED')
} else {
  console.error('Content has been MODIFIED')
}
```

## Complete Verification Example

```javascript
import { an } from './an.js'

async function verifyMessage(signedMessage, content) {
  // Step 1: Extract public key from message
  const publicKey = signedMessage.substring(0, 44)

  // Step 2: Open (verify) the message
  const opened = await an.open(signedMessage)

  if (opened === null) {
    return {
      valid: false,
      error: 'Signature verification failed',
      publicKey: publicKey
    }
  }

  // Step 3: Extract timestamp and hash
  const timestamp = parseInt(opened.substring(0, 13))
  const messageHash = opened.substring(13)

  // Step 4: Verify content matches hash
  const contentHash = await an.hash(content)
  const contentValid = contentHash === messageHash

  return {
    valid: contentValid,
    publicKey: publicKey,
    timestamp: timestamp,
    timestampDate: new Date(timestamp),
    messageHash: messageHash,
    contentHash: contentHash,
    contentMatch: contentValid,
    error: contentValid ? null : 'Content hash mismatch'
  }
}

// Usage
const result = await verifyMessage(signedMessage, originalContent)

if (result.valid) {
  console.log('Message verified!')
  console.log('Signed by:', result.publicKey)
  console.log('Signed at:', result.timestampDate)
} else {
  console.error('Verification failed:', result.error)
}
```

## What Verification Proves

| Check | What It Proves |
|-------|----------------|
| `an.open()` returns non-null | Signature is cryptographically valid |
| `contentHash === messageHash` | Content hasn't been modified |
| `publicKey` extraction | Identity of the signer |
| `timestamp` extraction | When the message was signed |

## Verification Without Original Content

Sometimes you only have the signed message (content stored elsewhere):

```javascript
async function verifySignatureOnly(signedMessage) {
  const publicKey = signedMessage.substring(0, 44)
  const opened = await an.open(signedMessage)

  if (opened === null) {
    return { valid: false, error: 'Invalid signature' }
  }

  const timestamp = parseInt(opened.substring(0, 13))
  const hash = opened.substring(13)

  return {
    valid: true,
    publicKey: publicKey,
    timestamp: timestamp,
    timestampDate: new Date(timestamp),
    contentHash: hash  // Use this to fetch content
  }
}
```

## Verify Against Known Signer

```javascript
async function verifyFromSigner(signedMessage, content, expectedPublicKey) {
  const actualPublicKey = signedMessage.substring(0, 44)

  // Check signer identity first
  if (actualPublicKey !== expectedPublicKey) {
    return {
      valid: false,
      error: 'Wrong signer',
      expected: expectedPublicKey,
      actual: actualPublicKey
    }
  }

  // Then verify signature and content
  const opened = await an.open(signedMessage)
  if (opened === null) {
    return { valid: false, error: 'Invalid signature' }
  }

  const contentHash = await an.hash(content)
  const messageHash = opened.substring(13)

  return {
    valid: contentHash === messageHash,
    publicKey: actualPublicKey,
    timestamp: parseInt(opened.substring(0, 13))
  }
}
```

## Timestamp Validation

```javascript
function validateTimestamp(timestamp, options = {}) {
  const now = Date.now()
  const maxAge = options.maxAge || 24 * 60 * 60 * 1000  // Default 24 hours
  const maxFuture = options.maxFuture || 5 * 60 * 1000  // Default 5 minutes

  const age = now - timestamp

  if (age > maxAge) {
    return { valid: false, error: 'Message too old', age: age }
  }

  if (age < -maxFuture) {
    return { valid: false, error: 'Message from future', offset: -age }
  }

  return { valid: true, age: age }
}

// Usage in verification
const result = await verifyMessage(signedMessage, content)
if (result.valid) {
  const timeCheck = validateTimestamp(result.timestamp, { maxAge: 3600000 })
  if (!timeCheck.valid) {
    console.warn('Timestamp issue:', timeCheck.error)
  }
}
```

## Batch Verification

```javascript
async function verifyBatch(messages) {
  const results = []

  for (const { signed, content } of messages) {
    const result = await verifyMessage(signed, content)
    results.push(result)
  }

  const valid = results.filter(r => r.valid).length
  const invalid = results.filter(r => !r.valid).length

  return {
    total: messages.length,
    valid: valid,
    invalid: invalid,
    results: results
  }
}
```

## Error Handling

| `an.open()` Returns | Meaning |
|---------------------|---------|
| `null` | Signature is invalid (tampered or wrong key) |
| String (57 chars) | Valid signature, returns timestamp + hash |

Common failure causes:
- Message was truncated
- Message was modified
- Wrong public key used in verification
- Corrupted base64 encoding

## Quick Reference

| Task | Code |
|------|------|
| Open/verify message | `await an.open(signedMessage)` |
| Extract public key | `signedMessage.substring(0, 44)` |
| Extract timestamp | `parseInt(opened.substring(0, 13))` |
| Extract hash | `opened.substring(13)` |
| Verify content | `await an.hash(content) === messageHash` |
