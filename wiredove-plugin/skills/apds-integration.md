---
description: Use this skill when developers need to work with APDS (Application Protocol Data Store) in Wiredove - including message storage, cryptographic operations, identity management, and data queries. APDS is the core data layer for all Wiredove operations.
---

# APDS Integration for Wiredove

APDS is the foundational library providing local-first data storage and cryptographic operations for Wiredove.

**Source**: https://esm.sh/gh/evbogue/apds

## Importing APDS

```javascript
import { apds } from 'apds'
import { h } from 'h'  // DOM helper also from APDS
```

## Database Initialization

```javascript
// Initialize with a unique database name
await apds.start('wiredovedbversion1')

// This creates an IndexedDB database for local storage
// Must be called before any other APDS operations
```

## Identity Management

### Generate a New Keypair
```javascript
const keypair = await apds.generate()
// Returns base64-encoded keypair string

// Save the keypair
await apds.put('keypair', keypair)
```

### Vanity Address Generation
```javascript
// Generate keypairs until prefix matches desired pattern
const name = 'ev'
let keypair
do {
  keypair = await apds.generate()
} while (keypair.substring(0, 2).toUpperCase() !== name.substring(0, 2).toUpperCase())

await apds.put('keypair', keypair)
```

### Access Identity
```javascript
const pubkey = await apds.pubkey()   // 44-char public key
const keypair = await apds.keypair() // Full keypair (private)
```

### Delete Identity
```javascript
await apds.deletekey()  // Remove keypair from storage
await apds.clear()      // Clear all data including keypair
```

## Message Creation

### Compose a Message
```javascript
// Simple message
const hash = await apds.compose('Hello, Wiredove!')

// Reply to another message
const hash = await apds.compose('This is my reply', {
  reply: parentMessageHash,
  replyto: parentAuthorPubkey
})

// Edit a previous message
const hash = await apds.compose('Updated content', {
  edit: originalMessageHash
})
```

### Message Structure
Messages are composed of:
- **Signature blob**: `[author:44][signature:86][timestamp:13][contentHash:43]`
- **Content blob**: YAML with fields like `name`, `image`, `body`, `previous`, `reply`, `edit`

## Message Verification & Parsing

### Open (Verify) a Signed Message
```javascript
const signed = 'base64-encoded-signature-blob'
const opened = await apds.open(signed)

if (opened) {
  const timestamp = opened.substring(0, 13)
  const contentHash = opened.substring(13)
  const author = signed.substring(0, 44)
}
```

### Parse Content YAML
```javascript
const content = await apds.get(contentHash)
const yaml = await apds.parseYaml(content)

if (yaml) {
  console.log(yaml.name)      // Author's display name
  console.log(yaml.image)     // Hash of avatar image
  console.log(yaml.body)      // Message content (markdown)
  console.log(yaml.previous)  // Hash of author's previous message
  console.log(yaml.reply)     // Hash of parent message (if reply)
  console.log(yaml.edit)      // Hash of original (if edit)
}
```

## Data Storage

### Key-Value Storage
```javascript
// Store a value
await apds.put('name', 'John Doe')
await apds.put('avatar', avatarDataUrl)

// Retrieve a value
const name = await apds.get('name')

// Content-addressed storage
const hash = await apds.hash(blob)
// apds.make() stores content by hash automatically
```

### Store Messages
```javascript
// Make (store content, verify if signed)
await apds.make(blob)

// Add to message log (index by author)
await apds.add(signedBlob)
```

## Querying Messages

### Query All Messages
```javascript
const log = await apds.query()
// Returns array of { hash, sig, opened, text, author, ts }
```

### Query by Author
```javascript
const authorLog = await apds.query(pubkey)
```

### Query by Search Term
```javascript
const results = await apds.query('?searchterm')
```

### Get Latest Message from Author
```javascript
const latest = await apds.getLatest(pubkey)
// Returns { sig, opened, text }
```

### Get All Known Pubkeys
```javascript
const pubkeys = await apds.getPubkeys()
// Returns array of all author pubkeys in database
```

### Get Opened Log (with parsed data)
```javascript
const log = await apds.getOpenedLog()
// Returns messages with opened timestamps and content
```

## Utility Functions

### Hash Content
```javascript
const hash = await apds.hash(blob)
// Returns 43-44 character base64 hash
```

### Human-Readable Time
```javascript
const timeAgo = await apds.human(timestamp)
// Returns "2 hours ago", "yesterday", etc.
```

### Generate Avatar
```javascript
const img = await apds.visual(pubkey)
// Returns an img element with generated avatar
```

## Common Patterns

### Check if Message Exists
```javascript
const exists = await apds.get(hash)
if (!exists) {
  // Request from network
  send(hash)
}
```

### Process Incoming Message
```javascript
const processBlob = async (blob) => {
  // 1. Verify and store
  await apds.make(blob)

  // 2. Check if it's a signed message
  const opened = await apds.open(blob)
  if (opened) {
    // 3. Add to indexed log
    await apds.add(blob)

    // 4. Get content
    const content = await apds.get(opened.substring(13))
    if (content) {
      const yaml = await apds.parseYaml(content)
      // Process yaml fields...
    }
  }
}
```

### Build Message Index
```javascript
const buildIndex = async () => {
  const log = await apds.getOpenedLog()
  const index = new Map()

  for (const msg of log) {
    if (!msg.text) continue
    const yaml = await apds.parseYaml(msg.text)
    if (yaml && yaml.reply) {
      const list = index.get(yaml.reply) || []
      list.push(msg.hash)
      index.set(yaml.reply, list)
    }
  }

  return index
}
```

## Key Files in Wiredove

- `app.js:8` - Database initialization
- `composer.js:87` - Message composition
- `render.js:816-818` - Message verification
- `sync.js:94` - Latest message retrieval
- `settings.js` - Keypair management
- `identify.js` - Keypair generation
