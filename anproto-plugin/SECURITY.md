# Security Policy

## Overview

ANProto is a cryptographic authentication protocol. This plugin helps developers use it correctly and securely.

## Reporting Security Issues

If you discover a security vulnerability in this plugin:

1. **Do NOT** open a public issue
2. Email the maintainer directly with details
3. Include steps to reproduce if possible
4. Allow reasonable time for a fix before disclosure

## Security Best Practices

### Key Management

#### DO

| Practice | Reason |
|----------|--------|
| Store keypairs in environment variables | Prevents accidental commits |
| Use `.gitignore` for key files | Keeps secrets out of repos |
| Generate unique keypairs per application | Limits blast radius if compromised |
| Back up keypairs securely (encrypted) | Prevents permanent data loss |
| Use file permissions (600) for key files | Restricts access to owner |

#### DON'T

| Anti-Pattern | Risk |
|--------------|------|
| Commit keypairs to version control | Permanent exposure in git history |
| Log full keypairs to console | Exposure in log files |
| Store keypairs in localStorage (production) | XSS can extract keys |
| Share secret keys between users | No accountability, mass compromise |
| Use deterministic keys from weak seeds | Predictable keys can be guessed |

### Message Verification

#### Complete Verification Checklist

```javascript
async function verifyMessage(signed, content, trustedKeys) {
  // 1. Verify signature
  const opened = await an.open(signed)
  if (!opened) {
    return { valid: false, error: 'INVALID_SIGNATURE' }
  }

  // 2. Check signer is trusted
  const publicKey = signed.substring(0, 44)
  if (!trustedKeys.includes(publicKey)) {
    return { valid: false, error: 'UNTRUSTED_SIGNER' }
  }

  // 3. Verify content hash
  const contentHash = await an.hash(content)
  if (opened.substring(13) !== contentHash) {
    return { valid: false, error: 'CONTENT_MODIFIED' }
  }

  // 4. Validate timestamp
  const timestamp = parseInt(opened.substring(0, 13))
  const age = Date.now() - timestamp
  if (age > 86400000) {  // 24 hours
    return { valid: false, error: 'MESSAGE_EXPIRED' }
  }
  if (age < -300000) {  // 5 minutes future
    return { valid: false, error: 'FUTURE_TIMESTAMP' }
  }

  return { valid: true, publicKey, timestamp }
}
```

### Common Vulnerabilities

#### 1. Signature Verification Bypass

**Vulnerable:**
```javascript
// BAD: No verification!
const data = JSON.parse(message.content)
processData(data)
```

**Secure:**
```javascript
// GOOD: Verify first
const opened = await an.open(message.signed)
if (!opened) throw new Error('Invalid signature')
const data = JSON.parse(message.content)
processData(data)
```

#### 2. Missing Content Hash Check

**Vulnerable:**
```javascript
// BAD: Only checks signature, not content
const opened = await an.open(signed)
if (opened) {
  // Content could be different from what was signed!
  processContent(content)
}
```

**Secure:**
```javascript
// GOOD: Verify content matches hash
const opened = await an.open(signed)
const contentHash = await an.hash(content)
if (opened && opened.substring(13) === contentHash) {
  processContent(content)
}
```

#### 3. Replay Attacks

**Vulnerable:**
```javascript
// BAD: No timestamp validation
if (await verifySignature(message)) {
  executeAction(message.action)  // Can replay old messages!
}
```

**Secure:**
```javascript
// GOOD: Check timestamp and track used nonces
const timestamp = parseInt(opened.substring(0, 13))
const messageId = opened  // Use full opened as nonce
if (Date.now() - timestamp > 60000) {
  throw new Error('Message expired')
}
if (usedNonces.has(messageId)) {
  throw new Error('Replay detected')
}
usedNonces.add(messageId)
```

#### 4. Key Extraction from Keypair

**Vulnerable:**
```javascript
// BAD: Storing full keypair when only public key needed
const author = message.keypair  // 132 chars including secret!
```

**Secure:**
```javascript
// GOOD: Only store public key
const author = message.keypair.substring(0, 44)  // 44 chars, public only
```

## Plugin Security Features

### Validation Hook

The `validate-anproto-input` hook provides:
- Detection of keypairs in code being committed
- Warnings about insecure storage patterns
- Suggestions for .gitignore additions
- Format validation for ANProto strings

### Security Reviewer Agent

The `anproto-reviewer` agent checks for:
- Exposed secret keys
- Incomplete verification
- Missing timestamp validation
- Insecure storage patterns
- Key management issues

## Cryptographic Considerations

### Algorithm Security

| Component | Algorithm | Security Level |
|-----------|-----------|----------------|
| Signing | Ed25519 | 128-bit |
| Hashing | SHA-256 | 256-bit |
| Encoding | Base64 | N/A (encoding only) |

### Known Limitations

1. **No encryption** - ANProto signs, doesn't encrypt. Content is visible.
2. **No forward secrecy** - Same key pair used for all messages
3. **Timestamp trust** - Signer controls timestamp; can backdate
4. **No revocation** - No built-in key revocation mechanism

### Recommendations

| Use Case | Recommendation |
|----------|----------------|
| Sensitive content | Encrypt before signing |
| High-security | Rotate keys periodically |
| Untrusted timestamps | Use server-side timestamps |
| Key compromise | Generate new identity, inform peers |

## Dependency Security

| Dependency | Purpose | Notes |
|------------|---------|-------|
| TweetNaCl | Ed25519 crypto | Well-audited, minimal |
| base64.js | Encoding | Deno standard library |
| Web Crypto API | SHA-256 hashing | Browser/runtime built-in |

## Updates

This security policy is reviewed with each major release.

Last reviewed: 2026-01-15
Version: 1.1.0
