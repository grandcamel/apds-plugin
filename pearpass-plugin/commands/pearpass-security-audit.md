---
description: Run security audit on PearPass codebase - check for credential exposure, crypto weaknesses, and security issues
invocable: true
arguments:
  - name: scope
    description: "Audit scope: full, crypto, credentials, or ipc (default: full)"
    required: false
---

# PearPass Security Audit

Run security checks on the PearPass codebase.

## Usage

```
/pearpass-security-audit [scope]
```

**Arguments:**
- `scope`: `full` (default), `crypto`, `credentials`, or `ipc`

## Instructions

Run the following security checks based on scope:

### Full Audit (All Checks)

Execute all security checks below in sequence.

### Credential Exposure Check

Search for potential credential logging:

```bash
cd /path/to/pearpass-app-mobile

# Check for password/credential logging
grep -rn "console\.\|logger\." src/ packages/pearpass-lib-vault-core/ | grep -i "password\|credential\|secret\|key"

# Check for JSON.stringify of sensitive data
grep -rn "JSON.stringify.*password" src/
grep -rn "JSON.stringify.*credential" src/
grep -rn "JSON.stringify.*secret" src/
```

**Expected Result:** No matches (or only false positives in test files)

### Cryptographic Implementation Check

Verify proper sodium-native usage:

```bash
# Check for sodium_malloc usage (secure memory allocation)
grep -rn "sodium_malloc" packages/pearpass-lib-vault-core/

# Check for proper password hashing parameters
grep -rn "OPSLIMIT_SENSITIVE\|OPSLIMIT_INTERACTIVE" packages/pearpass-lib-vault-core/

# Check for random nonce generation
grep -rn "randombytes_buf" packages/pearpass-lib-vault-core/

# WARNING: Check for weak parameters
grep -rn "OPSLIMIT_MIN\|OPSLIMIT_MODERATE" packages/pearpass-lib-vault-core/
```

**Expected Results:**
- ✅ `sodium_malloc` used for sensitive buffers
- ✅ `OPSLIMIT_SENSITIVE` for password hashing
- ✅ `randombytes_buf` for nonce generation
- ❌ No `OPSLIMIT_MIN` or `OPSLIMIT_MODERATE`

### Insecure Buffer Allocation Check

```bash
# Check for Buffer.alloc in crypto code (should use sodium_malloc)
grep -rn "Buffer.alloc\|Buffer.from" packages/pearpass-lib-vault-core/src/worklet/ | grep -v "test"
```

**Expected Result:** Minimal usage, prefer `sodium_malloc` for sensitive data

### Authentication Check

```bash
# Verify auth check before using decrypted data
grep -rn "crypto_secretbox_open_easy" packages/pearpass-lib-vault-core/ -A 5
```

**Expected Result:** All decryption returns undefined on auth failure

### Hardcoded Secrets Check

```bash
# Search for potential hardcoded secrets
grep -rn "apiKey\s*=\|secret\s*=\|password\s*=" src/ | grep -v "test\|\.test\."
grep -rn "Bearer\s" src/ | grep -v "test"
```

**Expected Result:** No hardcoded secrets

### IPC Security Check

```bash
# Check native messaging IPC
grep -rn "nativeMessagingIPC" src/
grep -rn "validateOrigin" src/services/
```

**Expected Result:** Origin validation implemented for browser extension IPC

### Clipboard Security Check

```bash
# Verify clipboard auto-clear
grep -rn "clipboardCleanup\|Clipboard" src/hooks/
grep -rn "setTimeout.*clipboard\|setStringAsync" src/
```

**Expected Result:** Clipboard clears after timeout

### Inactivity Detection Check

```bash
# Verify auto-lock on inactivity
grep -rn "user-inactive\|inactivity" src/
grep -rn "resetInactivityTimer\|inactivityTimeout" src/
```

**Expected Result:** Vault locks on user inactivity (~60 seconds)

## Security Checklist

### Critical (Must Pass)

- [ ] No credentials in console.log or logger calls
- [ ] Uses sodium_malloc for sensitive buffers (not Buffer.alloc)
- [ ] Random nonce for each encryption (randombytes_buf)
- [ ] OPSLIMIT_SENSITIVE for password hashing
- [ ] Auth check before using decrypted data
- [ ] No hardcoded secrets or API keys

### High Priority

- [ ] Clipboard auto-clears after timeout
- [ ] Vault locks on inactivity
- [ ] IPC validates origins
- [ ] Error messages don't expose sensitive data
- [ ] Worklet cleanup on app teardown (Pear.teardown)

### Medium Priority

- [ ] Input validation on imported data
- [ ] Rate limiting on vault operations
- [ ] No verbose error messages with internal details

## Report Format

When reporting findings, use this format:

```markdown
## Security Finding

**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**File:** path/to/file.js:line
**Category:** Credential Exposure | Crypto Weakness | IPC | Clipboard

### Description
[What the issue is]

### Risk
[Potential impact if exploited]

### Evidence
```javascript
[Relevant code snippet]
```

### Recommendation
[How to fix with secure pattern]
```

## Reference: Secure Patterns

### Secure Password Hashing

```javascript
import sodium from 'sodium-native'

const salt = sodium.sodium_malloc(sodium.crypto_pwhash_SALTBYTES)
sodium.randombytes_buf(salt)

const hashedPassword = sodium.sodium_malloc(sodium.crypto_secretbox_KEYBYTES)
sodium.crypto_pwhash(
  hashedPassword,
  Buffer.from(password),
  salt,
  sodium.crypto_pwhash_OPSLIMIT_SENSITIVE,   // ✅ High computation
  sodium.crypto_pwhash_MEMLIMIT_INTERACTIVE, // ✅ Memory-hard
  sodium.crypto_pwhash_ALG_DEFAULT           // ✅ Argon2id
)
```

### Secure Encryption

```javascript
const nonce = sodium.sodium_malloc(sodium.crypto_secretbox_NONCEBYTES)
sodium.randombytes_buf(nonce)  // ✅ Random nonce

sodium.crypto_secretbox_easy(ciphertext, message, nonce, key)
```

### Secure Decryption

```javascript
if (!sodium.crypto_secretbox_open_easy(
  plainText, ciphertext, nonce, hashedPassword
)) {
  return undefined  // ✅ Don't expose why it failed
}
```

## Critical Files to Review

| File | Purpose | Priority |
|------|---------|----------|
| `packages/pearpass-lib-vault-core/src/worklet/hashPassword.js` | Password hashing | CRITICAL |
| `packages/pearpass-lib-vault-core/src/worklet/encryptVaultKeyWithHashedPassword.js` | Key encryption | CRITICAL |
| `packages/pearpass-lib-vault-core/src/worklet/decryptVaultKey.js` | Key decryption | CRITICAL |
| `src/services/clipboardCleanup.js` | Clipboard security | HIGH |
| `src/services/nativeMessagingIPCServer.js` | Browser extension IPC | HIGH |
