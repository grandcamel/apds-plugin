---
name: anproto-parse
description: Parse and explain the structure of an ANProto message, keypair, or hash
version: "1.1.0"
author: "grandcamel"
license: "MIT"
category: "debug"
keywords:
  - anproto
  - parse
  - debug
  - message
  - keypair
  - signature
---

# ANProto Message Parser

You are helping the user understand an ANProto message or component. Parse the input and explain its structure.

## How to Use This Command

The user will provide an ANProto string. Analyze it and explain what it is.

## Detection Logic

Determine the type based on length and content:

| Length | Likely Type | Characteristics |
|--------|-------------|-----------------|
| 44 | Hash OR Public Key | Base64, ends with `=` |
| 57 | Opened Message | Starts with 13 digits (timestamp) |
| 132 | Full Keypair | Two base64 segments |
| >44, variable | Signed Message | Starts with 44-char public key |

## Parsing Instructions

### For a 44-character string (Hash or Public Key)

```
Input: pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=

Analysis:
┌──────────────────────────────────────────────────────┐
│ Type: SHA-256 Hash (or possibly a Public Key)        │
│ Length: 44 characters                                │
│ Encoding: Base64                                     │
├──────────────────────────────────────────────────────┤
│ This could be:                                       │
│ • A content hash from an.hash()                      │
│ • A public key extracted from a keypair              │
│                                                      │
│ To determine which:                                  │
│ • If you have the original content, hash it and      │
│   compare to see if it's a content hash              │
│ • If it came from keypair.substring(0,44), it's      │
│   a public key                                       │
└──────────────────────────────────────────────────────┘
```

### For a 57-character string (Opened Message)

```
Input: 1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=

Analysis:
┌──────────────────────────────────────────────────────┐
│ Type: Opened Message (from an.open())                │
│ Length: 57 characters                                │
├──────────────────────────────────────────────────────┤
│ TIMESTAMP (chars 0-12):                              │
│   Value: 1755197841319                               │
│   Date: [Convert to human-readable date]             │
│                                                      │
│ CONTENT HASH (chars 13-56):                          │
│   Value: pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=│
│   Use this to verify content with an.hash()          │
└──────────────────────────────────────────────────────┘

To verify content integrity:
  const contentHash = await an.hash(yourContent)
  const matches = contentHash === "pZGm1Av0..."
```

Convert the timestamp to a human-readable date:

```javascript
new Date(1755197841319).toISOString()
// e.g., "2025-08-14T12:30:41.319Z"
```

### For a 132-character string (Keypair)

```
Input: BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EFJjv96vhUki7Tyjf01pECI9j8wC93uhCGUYJEMAGNhQ==

Analysis:
┌──────────────────────────────────────────────────────┐
│ Type: Full Keypair                                   │
│ Length: 132 characters                               │
├──────────────────────────────────────────────────────┤
│ PUBLIC KEY (chars 0-43):                             │
│   BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=       │
│   ✓ Safe to share - this is your identity           │
│                                                      │
│ SECRET KEY (chars 44-131):                           │
│   tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EF...   │
│   ⚠️  NEVER SHARE - keep this private!               │
└──────────────────────────────────────────────────────┘

⚠️  WARNING: If this keypair was shared publicly,
    consider it compromised. Generate a new one.
```

### For a variable-length string >44 (Signed Message)

```
Input: BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=yVpD8i7d3d4dls3YThEg1x1vSdmqeEweV4e4Ejl/8yPoVG7JR0YAKDPagQOgxXMrlCVLNNqvlNvj4xRDOYDLBjE3NTUxOTc4NDEzMTlwWkdtMUF2MElFQktBUmN6ejdleGtOWXNaYjhMemFNclY3SjMyYTJmRkc0PQ==

Analysis:
┌──────────────────────────────────────────────────────┐
│ Type: Signed Message                                 │
│ Total Length: [length] characters                    │
├──────────────────────────────────────────────────────┤
│ PUBLIC KEY (chars 0-43):                             │
│   BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=       │
│   This identifies the signer                         │
│                                                      │
│ SIGNED PAYLOAD (chars 44+):                          │
│   yVpD8i7d3d4dls3YThEg1x1vSdmqeEweV4e4Ejl/8yPo...   │
│   Length: [payload length] characters                │
│   Contains: NaCl signature of (timestamp + hash)     │
└──────────────────────────────────────────────────────┘

To verify this message:
  const opened = await an.open(signedMessage)
  if (opened) {
    console.log('Valid! Timestamp:', opened.substring(0,13))
    console.log('Hash:', opened.substring(13))
  }
```

## Interactive Verification

If the user wants to verify a signed message, offer to help:

"Would you like me to help you verify this message? I'll need:
1. The signed message (which you've provided)
2. The original content that was signed

Then we can:
- Verify the signature is valid
- Check the content matches the hash
- Show you when it was signed"

## Common Issues to Check

1. **Wrong length**: Check if the string was truncated or has extra characters
2. **Invalid base64**: Look for characters outside the base64 alphabet
3. **Whitespace**: Check for hidden newlines or spaces
4. **URL encoding**: Check if it needs to be URL-decoded first

```javascript
// Clean common issues
const cleaned = input
  .trim()
  .replace(/\s/g, '')
  .replace(/%3D/g, '=')  // URL-encoded equals
```

## Output Format

Always provide:
1. The detected type
2. A visual breakdown of the components
3. What each component means
4. Next steps or verification instructions
