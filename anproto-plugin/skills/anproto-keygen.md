---
name: anproto-keygen
description: Use this skill when the user asks to "generate keypair", "create anproto keys", "new identity", "generate ed25519 keys", or needs help with ANProto key generation and secure storage practices.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
allowed-tools:
  - Bash
  - Read
  - Write
---

# ANProto Key Generation

Guide users through generating and securely managing ANProto ed25519 keypairs.

## Key Generation Basics

### What is a Keypair?

An ANProto keypair consists of:
- **Public Key** (44 chars): Your identity, safe to share
- **Secret Key** (88 chars): Your private signing key, NEVER share

Combined format (132 chars):
```
[publicKey:44][secretKey:88]
```

### Generate a Keypair

```javascript
import { an } from './an.js'

const keypair = await an.gen()

// Extract components
const publicKey = keypair.substring(0, 44)
const secretKey = keypair.substring(44)

console.log('Your public key (share this):', publicKey)
console.log('Your secret key (keep private):', secretKey)
```

### Run as One-Liner (Deno)

```bash
deno eval "import{an}from'https://raw.githubusercontent.com/evbogue/anproto/main/an.js';console.log(await an.gen())"
```

## Security Best Practices

### DO

- Store keypairs in environment variables or secure vaults
- Back up your keypair securely (encrypted)
- Use different keypairs for different applications
- Rotate keypairs if you suspect compromise

### DON'T

- Commit keypairs to git
- Share your secret key with anyone
- Store keypairs in plain text files in your repo
- Use the same keypair across untrusted services

## Storage Patterns

### Pattern 1: Environment Variables (Recommended)

```bash
# .env (add to .gitignore!)
ANPROTO_KEYPAIR="BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EFJjv96vhUki7Tyjf01pECI9j8wC93uhCGUYJEMAGNhQ=="
```

```javascript
const keypair = process.env.ANPROTO_KEYPAIR
```

### Pattern 2: Separate Key Files

```bash
# Create keys directory
mkdir -p .keys
echo "your-keypair-here" > .keys/identity.key
chmod 600 .keys/identity.key

# Add to .gitignore
echo ".keys/" >> .gitignore
```

```javascript
import { readFileSync } from 'fs'
const keypair = readFileSync('.keys/identity.key', 'utf8').trim()
```

### Pattern 3: System Keychain (macOS)

```bash
# Store
security add-generic-password -a "$USER" -s "anproto-keypair" -w "your-keypair-here"

# Retrieve
security find-generic-password -a "$USER" -s "anproto-keypair" -w
```

### Pattern 4: Local Storage (Browser)

```javascript
// Store (once)
localStorage.setItem('anproto-keypair', keypair)

// Retrieve
const keypair = localStorage.getItem('anproto-keypair')
```

**Warning**: Local storage is accessible to JavaScript on the same origin. Only use for low-security applications.

## Multiple Identities

For applications needing multiple identities:

```javascript
async function createIdentities(count) {
  const identities = {}

  for (let i = 0; i < count; i++) {
    const keypair = await an.gen()
    identities[`identity-${i}`] = {
      publicKey: keypair.substring(0, 44),
      keypair: keypair  // Full keypair for signing
    }
  }

  return identities
}
```

## Key Verification

Verify a keypair is valid:

```javascript
async function verifyKeypair(keypair) {
  if (keypair.length !== 132) {
    return { valid: false, error: 'Invalid length' }
  }

  try {
    // Test by signing and verifying
    const testHash = await an.hash('test')
    const signed = await an.sign(testHash, keypair)
    const opened = await an.open(signed)

    if (opened && opened.substring(13) === testHash) {
      return { valid: true, publicKey: keypair.substring(0, 44) }
    } else {
      return { valid: false, error: 'Signature verification failed' }
    }
  } catch (e) {
    return { valid: false, error: e.message }
  }
}
```

## Key Derivation (Advanced)

If you need deterministic keys from a seed:

```javascript
import nacl from './lib/nacl-fast-es.js'
import { encode } from './lib/base64.js'

async function deriveKeypair(seed) {
  // Hash the seed to get 32 bytes
  const seedBytes = new Uint8Array(
    await crypto.subtle.digest(
      'SHA-256',
      new TextEncoder().encode(seed)
    )
  )

  // Generate keypair from seed
  const keypair = nacl.sign.keyPair.fromSeed(seedBytes)

  return encode(keypair.publicKey) + encode(keypair.secretKey)
}

// Usage: Same seed = same keypair
const keypair = await deriveKeypair('my-secret-passphrase')
```

**Warning**: Deterministic keys are only as secure as the seed. Use strong, random seeds.

## Quick Reference

| Action | Code |
|--------|------|
| Generate keypair | `await an.gen()` |
| Extract public key | `keypair.substring(0, 44)` |
| Extract secret key | `keypair.substring(44)` |
| Verify keypair length | `keypair.length === 132` |
