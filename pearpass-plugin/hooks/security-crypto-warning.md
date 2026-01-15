---
event: PreToolUse
tools:
  - Edit
  - Write
match_files:
  - "**/pearpass-lib-vault-core/**"
  - "**/worklet/**"
  - "**/*encrypt*"
  - "**/*decrypt*"
  - "**/*hash*"
  - "**/*sodium*"
  - "**/*crypto*"
---

# Security Warning: Cryptographic Code Modification

You are about to modify security-critical cryptographic code in PearPass.

## ⚠️ Critical Security Review Required

Before proceeding with this edit, ensure:

### Encryption Standards
- [ ] Using `sodium_malloc` for sensitive buffer allocation (NOT `Buffer.alloc`)
- [ ] Using `randombytes_buf` for nonce generation (NOT sequential/predictable)
- [ ] Using `crypto_pwhash_OPSLIMIT_SENSITIVE` for password hashing (NOT MIN/MODERATE)
- [ ] Using authenticated encryption (XSalsa20-Poly1305 or AES-GCM)

### Memory Safety
- [ ] Sensitive buffers allocated with `sodium_malloc`
- [ ] Buffers cleared after use with `buffer.fill(0)`
- [ ] No sensitive data in error messages or logs

### Authentication
- [ ] Decryption checks authentication before returning data
- [ ] Returns `undefined` on auth failure (no error details exposed)
- [ ] Constant-time comparison for sensitive values

## Files Being Modified

The following file patterns trigger this warning:
- `packages/pearpass-lib-vault-core/` - Core encryption library
- `**/worklet/**` - Isolated crypto worklet
- Files containing: encrypt, decrypt, hash, sodium, crypto

## Secure Patterns Reference

### Password Hashing (Argon2id)
```javascript
const salt = sodium.sodium_malloc(sodium.crypto_pwhash_SALTBYTES)
sodium.randombytes_buf(salt)

sodium.crypto_pwhash(
  hashedPassword,
  Buffer.from(password),
  salt,
  sodium.crypto_pwhash_OPSLIMIT_SENSITIVE,   // ✅ Required
  sodium.crypto_pwhash_MEMLIMIT_INTERACTIVE,
  sodium.crypto_pwhash_ALG_DEFAULT
)
```

### Authenticated Encryption
```javascript
const nonce = sodium.sodium_malloc(sodium.crypto_secretbox_NONCEBYTES)
sodium.randombytes_buf(nonce)  // ✅ Random nonce

sodium.crypto_secretbox_easy(ciphertext, message, nonce, key)
```

### Decryption with Auth Check
```javascript
if (!sodium.crypto_secretbox_open_easy(plainText, ciphertext, nonce, key)) {
  return undefined  // ✅ No error details
}
```

## Red Flags to Avoid

- `console.log` with password, key, secret, or credential
- `Buffer.alloc` or `Buffer.from` for encryption keys
- Hardcoded nonces or sequential nonce generation
- `OPSLIMIT_MIN` or `OPSLIMIT_MODERATE` parameters
- Unauthenticated encryption modes (AES-CBC without MAC)
- Detailed error messages on decryption failure

## Recommendation

Consider running `/pearpass-security-audit crypto` after completing these changes to verify security compliance.

Proceed with caution. Security bugs in this code can expose user credentials.
