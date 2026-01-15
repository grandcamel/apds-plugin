---
description: "Use this skill for security-focused code review of PearPass Desktop, when reviewing credential handling, encryption implementation, vault operations, clipboard security, or P2P sync security."
invocable: true
---

# PearPass Desktop Security Review

You are conducting security-focused code review for PearPass Desktop, a password manager where security is critical.

## Security Architecture Overview

### Encryption Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| sodium-native | 5.0.6 | libsodium bindings (primary crypto) |
| bare-crypto | 1.12.0 | Bare crypto primitives |
| hyperswarm | 4.16.0 | P2P encrypted networking |
| hyperdht | 6.27.0 | Distributed hash table |
| corestore | 7.6.1 | Encrypted data storage |

### Security Layers

1. **Master Password** → Argon2id → Derived key
2. **Vault Key** → XSalsa20-Poly1305 → Encrypted credentials
3. **Worklet Isolation** → Separate process for crypto
4. **Clipboard** → Auto-clear after timeout
5. **Inactivity** → Auto-lock after 60 seconds
6. **P2P Sync** → End-to-end encrypted

## Critical Security Files

| File | Purpose | Priority |
|------|---------|----------|
| `packages/pearpass-lib-vault-core/src/worklet/hashPassword.js` | Argon2id password hashing | CRITICAL |
| `packages/pearpass-lib-vault-core/src/worklet/encryptVaultKeyWithHashedPassword.js` | Vault key encryption | CRITICAL |
| `packages/pearpass-lib-vault-core/src/worklet/decryptVaultKey.js` | Vault key decryption | CRITICAL |
| `src/services/clipboardCleanup.js` | Clipboard auto-clear | HIGH |
| `src/services/nativeMessagingIPCServer.js` | Browser extension IPC | HIGH |
| `packages/pearpass-lib-vault-core/src/worklet/rateLimiter.js` | Rate limiting | HIGH |

## Cryptographic Implementation

### Password Hashing (Argon2id)

From `packages/pearpass-lib-vault-core/src/worklet/hashPassword.js`:

```javascript
import sodium from 'sodium-native'

export const hashPassword = (password) => {
  // Memory-safe buffer allocation
  const salt = sodium.sodium_malloc(sodium.crypto_pwhash_SALTBYTES)
  sodium.randombytes_buf(salt)  // Cryptographically random salt

  const hashedPassword = sodium.sodium_malloc(sodium.crypto_secretbox_KEYBYTES)

  // Argon2id with sensitive parameters
  sodium.crypto_pwhash(
    hashedPassword,
    Buffer.from(password),
    salt,
    sodium.crypto_pwhash_OPSLIMIT_SENSITIVE,   // High computation cost
    sodium.crypto_pwhash_MEMLIMIT_INTERACTIVE, // Memory-hard
    sodium.crypto_pwhash_ALG_DEFAULT           // Argon2id algorithm
  )

  return {
    hashedPassword: Buffer.from(hashedPassword).toString('hex'),
    salt: salt.toString('base64')
  }
}
```

**Security Properties:**
- Uses `sodium_malloc` for memory-safe buffers
- `OPSLIMIT_SENSITIVE` - High computation cost (resistant to brute force)
- `MEMLIMIT_INTERACTIVE` - Memory-hard (resistant to GPU attacks)
- Random salt per password

### Vault Key Encryption (XSalsa20-Poly1305)

From `encryptVaultKeyWithHashedPassword.js`:

```javascript
export const encryptVaultKeyWithHashedPassword = (hashedPassword) => {
  const nonce = sodium.sodium_malloc(sodium.crypto_secretbox_NONCEBYTES)
  const key = sodium.sodium_malloc(32)  // 256-bit vault key

  sodium.randombytes_buf(key)    // Random vault key
  sodium.randombytes_buf(nonce)  // Random nonce

  const ciphertext = sodium.sodium_malloc(
    key.length + sodium.crypto_secretbox_MACBYTES  // Includes auth tag
  )

  // XSalsa20-Poly1305 authenticated encryption
  sodium.crypto_secretbox_easy(
    ciphertext,
    key,
    nonce,
    Buffer.from(hashedPassword, 'hex')
  )

  return {
    ciphertext: ciphertext.toString('base64'),
    nonce: nonce.toString('base64')
  }
}
```

**Security Properties:**
- XSalsa20-Poly1305 (authenticated encryption)
- Random 192-bit nonce per encryption
- MAC tag for tamper detection
- Memory-safe buffers

### Vault Key Decryption

From `decryptVaultKey.js`:

```javascript
export const decryptVaultKey = (data) => {
  const ciphertext = Buffer.from(data.ciphertext, 'base64')
  const nonce = Buffer.from(data.nonce, 'base64')

  const hashedPassword = sodium.sodium_malloc(sodium.crypto_secretbox_KEYBYTES)
  Buffer.from(data.hashedPassword, 'hex').copy(hashedPassword)

  const plainText = sodium.sodium_malloc(
    ciphertext.length - sodium.crypto_secretbox_MACBYTES
  )

  // Authenticated decryption - fails if tampered
  if (!sodium.crypto_secretbox_open_easy(
    plainText, ciphertext, nonce, hashedPassword
  )) {
    return undefined  // Authentication failed - wrong password or tampered
  }

  return plainText.toString('base64')
}
```

**Security Properties:**
- Returns `undefined` on auth failure (constant-time rejection)
- No information leakage on failure
- Detects any tampering

## Worklet Isolation Model

Vault crypto operations run in **isolated process**:

```javascript
// src/services/createOrGetPipe.js
import pearRun from 'pear-run'

export const createOrGetPipe = () => {
  pipe = pearRun(WORKLET_PATH)

  Pear.teardown(() => {
    if (pipe) {
      pipe.end()
      pipe = null
    }
  })

  return pipe
}
```

**Security Benefits:**
- Encryption keys never in main process memory
- Crash in main process doesn't expose keys
- Worklet has minimal attack surface

## Clipboard Security

From `src/services/clipboardCleanup.js`:

**Expected behavior:**
- Auto-clear clipboard after timeout
- Only clear if clipboard unchanged
- Cancel timer on new copy

## Inactivity Detection

From `app.tsx`:

```javascript
const activityEvents = ['mousemove', 'mousedown', 'keydown', 'touchstart', 'scroll']

const resetInactivityTimer = () => {
  clearTimeout(inactivityTimeout)
  inactivityTimeout = setTimeout(() => {
    window.dispatchEvent(new Event('user-inactive'))  // Triggers vault lock
  }, 60 * 1000)  // 1 minute
}
```

**Security Properties:**
- Vault auto-locks on inactivity
- Multiple event types monitored
- Configurable timeout

## Security Checklist

### Code Review Points
- [ ] No credentials in `console.log` or `logger.error`
- [ ] Uses `sodium_malloc` for sensitive buffers
- [ ] Random nonce for each encryption operation
- [ ] Authentication checked before using decrypted data
- [ ] Vault operations go through isolated worklet
- [ ] Clipboard auto-clear implemented
- [ ] No hardcoded secrets or keys

### Cryptographic Review
- [ ] Using Argon2id (not older algorithms)
- [ ] Using XSalsa20-Poly1305 or AES-GCM (authenticated)
- [ ] Proper nonce generation (random, not sequential)
- [ ] OPSLIMIT_SENSITIVE for password hashing
- [ ] Memory-hard parameters for key derivation

### IPC Security
- [ ] Native messaging validates origins
- [ ] Rate limiting on API endpoints
- [ ] No sensitive data in IPC error messages

## Common Vulnerabilities

### DO NOT:
```javascript
// INSECURE: Logging sensitive data
console.log('Password:', password)
console.log('Vault key:', key)

// INSECURE: Using regular Buffer (not sodium_malloc)
const key = Buffer.alloc(32)

// INSECURE: Predictable nonce
const nonce = Buffer.from('0000000000000000')

// INSECURE: Weak key derivation
crypto.pbkdf2Sync(password, salt, 1000, 32, 'sha256')

// INSECURE: Unauthenticated encryption
crypto.createCipheriv('aes-256-cbc', key, iv)
```

### DO:
```javascript
// SECURE: Memory-safe buffer
const key = sodium.sodium_malloc(32)

// SECURE: Random nonce
sodium.randombytes_buf(nonce)

// SECURE: Authenticated encryption
sodium.crypto_secretbox_easy(ciphertext, message, nonce, key)

// SECURE: Auth check before use
if (!sodium.crypto_secretbox_open_easy(...)) {
  return undefined  // Don't expose why it failed
}

// SECURE: Clear sensitive data
buffer.fill(0)
```

## P2P Security

### Hyperswarm/HyperDHT

- All data encrypted at rest
- P2P connections use Noise protocol
- Blind mirrors cannot read data
- End-to-end encryption between devices

### Key Files
- `packages/pearpass-lib-vault-core/src/worklet/utils/swarm.js`
- `packages/pearpass-lib-vault-core/src/worklet/utils/dht.js`
- `packages/pearpass-lib-vault-core/src/worklet/pearpassPairer.js`

## Security Testing

### Unit Tests to Check
- `packages/pearpass-lib-vault-core/src/worklet/hashPassword.test.js`
- `packages/pearpass-lib-vault-core/src/worklet/decryptVaultKey.test.js`
- `src/services/clipboardCleanup.test.js`
- `src/services/nativeMessagingIPCServer.test.js`

### Manual Testing
1. Verify vault locks on inactivity
2. Verify clipboard clears after timeout
3. Verify wrong password rejected (constant-time)
4. Verify IPC rejects invalid origins

## Review Guidelines

1. **Audit all vault-core changes** - This is the encryption core
2. **Check sodium-native usage** - Ensure using memory-safe APIs
3. **Verify nonce uniqueness** - Must be random per encryption
4. **Review IPC handlers** - Validate all inputs
5. **Check error messages** - No sensitive data leakage
6. **Trace data flow** - Ensure encryption at all boundaries
