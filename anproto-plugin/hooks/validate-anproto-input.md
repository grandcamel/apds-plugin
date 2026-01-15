---
event: PreToolUse
tools:
  - Edit
  - Write
match_files:
  - "**/*.key"
  - "**/.env*"
  - "**/secrets/**"
  - "**/.keys/**"
  - "**/an.js"
  - "**/an-node.js"
---

# ANProto Input Validation Hook

This hook validates ANProto-related inputs before tool execution.

## Validation Rules

### 1. Keypair Detection and Warning

When content contains a 132-character base64 string that looks like a keypair:

```
Pattern: [A-Za-z0-9+/]{43}=[A-Za-z0-9+/]{86}==
```

**Actions:**
- Warn if keypair appears in code being written to a file
- Suggest using environment variables instead
- Check if the target file is in .gitignore

**Warning message:**
```
⚠️  SECURITY WARNING: Detected what appears to be an ANProto keypair.
    Secret keys should NEVER be committed to version control.

    Recommendations:
    - Store in environment variable: ANPROTO_KEYPAIR
    - Add to .gitignore if using a key file
    - Use only the public key (first 44 chars) in code
```

### 2. Public Key vs Full Keypair

When code uses `substring(0, 44)` to extract a public key, this is safe.
When code stores the full keypair (132 chars), warn if:
- Writing to a non-ignored file
- Logging to console without redaction

### 3. Message Format Validation

For `/anproto-parse` command inputs, validate:

| Length | Expected Type | Validation |
|--------|---------------|------------|
| 44 | Hash or Public Key | Valid base64 ending in `=` |
| 57 | Opened Message | Starts with 13 digits |
| 132 | Keypair | Two base64 segments |
| >44 | Signed Message | Starts with valid public key |

**If invalid:**
```
⚠️  The provided string doesn't match expected ANProto formats:
    - Hash/Public Key: 44 characters, base64
    - Opened Message: 57 characters, starts with timestamp
    - Keypair: 132 characters, two base64 segments
    - Signed Message: >44 characters, starts with public key

    Provided: [length] characters
    Issue: [specific problem]
```

### 4. .gitignore Check

When writing files that might contain keys:
- `*.key` files
- `.env` files
- Files in `secrets/` or `.keys/` directories

**Check if .gitignore includes the path. If not, suggest:**
```
💡 Consider adding to .gitignore:
   echo "path/to/file" >> .gitignore
```

## Implementation Notes

This hook operates as a prompt-based validation that:
1. Scans tool input for ANProto patterns
2. Provides warnings but does not block execution
3. Suggests security best practices
4. Helps developers avoid common mistakes

## Pattern Detection

```javascript
// Keypair pattern (132 chars)
const keypairPattern = /^[A-Za-z0-9+/]{43}=[A-Za-z0-9+/]{86}==$/

// Public key pattern (44 chars)
const pubkeyPattern = /^[A-Za-z0-9+/]{43}=$/

// Opened message pattern (57 chars, starts with timestamp)
const openedPattern = /^\d{13}[A-Za-z0-9+/]{43}=$/

// Signed message pattern (starts with public key)
const signedPattern = /^[A-Za-z0-9+/]{43}=.+$/
```
