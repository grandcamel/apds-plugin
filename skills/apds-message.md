---
description: Help compose APDS messages with proper YAML frontmatter format, message signing, and content structure
user_invocable: true
---

# APDS Message Composition

You are helping developers compose and understand APDS messages. APDS uses a specific format with YAML frontmatter for metadata and signed content.

## Message Structure

APDS messages follow this format:

```yaml
---
name: DisplayName
image: base64-hash-of-avatar==
previous: base64-hash-of-previous-message==
---
Message body content here (supports markdown)
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | No | Display name for the author |
| `image` | No | Hash of avatar image (stored separately) |
| `previous` | No | Hash linking to previous message (chain) |

## Composing Messages

### Basic Message (No Metadata)
```javascript
// Simple message without frontmatter
const hash = await apds.sign('Hello, ANProto!')
```

### Message with Profile
```javascript
// Set profile first
await apds.put('name', 'Alice')

// Compose includes name automatically
const hash = await apds.compose('Hello with my name!')
```

### Message with All Metadata
```javascript
// Set up profile
await apds.put('name', 'Alice')
await apds.put('image', avatarHash)

// Previous message is tracked automatically after first compose
const hash = await apds.compose('This links to my previous message')
```

### Custom Frontmatter
```javascript
// Add custom fields via second parameter
const hash = await apds.compose('Custom message', {
  topic: 'announcements',
  reply_to: someMessageHash
})
```

## Message Flow

```
Content String
     ↓
apds.compose(content)
     ↓
Add frontmatter (name, image, previous)
     ↓
apds.createYaml(obj, content)
     ↓
apds.sign(yamlString)
     ↓
apds.hash(signature) → store
     ↓
Return hash (44 chars, base64)
```

## Reading Messages

### Parse a Message
```javascript
const msg = await apds.get(hash)
const yaml = await apds.parseYaml(msg)

console.log(yaml.name)     // "Alice"
console.log(yaml.image)    // avatar hash
console.log(yaml.previous) // previous message hash
console.log(yaml.body)     // "Message content"
```

### Query Messages
```javascript
// All messages
const all = await apds.query()

// By author pubkey
const byAuthor = await apds.query('ABC123pubkey...')

// Search content
const search = await apds.query('?keyword')

// By hash
const byHash = await apds.query('exactHash==')
```

## Message Object Structure

When queried, messages return as objects:

```javascript
{
  hash: "WxYz...==",           // Content hash (44 chars)
  sig: "AbCd...==",            // Full signature
  author: "PuBk...44chars",    // Author's public key
  opened: "1234567890123hash", // Timestamp + content hash
  text: "---\nname: ...",      // Raw YAML content
  ts: "1234567890123"          // Unix timestamp (ms)
}
```

## Common Patterns

### Reply to a Message
```javascript
const replyHash = await apds.compose('My reply', {
  reply_to: originalMessageHash
})
```

### Thread/Topic Messages
```javascript
const threadMsg = await apds.compose('Thread message', {
  thread: threadRootHash,
  topic: 'general'
})
```

### Message with Attachment Reference
```javascript
// First, store the attachment
const attachmentHash = await apds.make(attachmentData)

// Reference in message
const msgHash = await apds.compose('Check this out', {
  attachment: attachmentHash,
  attachment_type: 'image/png'
})
```

## Validation

### Check if Message is Valid
```javascript
const msg = await apds.get(hash)
const opened = await apds.open(msg)

if (opened) {
  console.log('Valid signature!')
  console.log('Timestamp:', opened.substring(0, 13))
  console.log('Content hash:', opened.substring(13))
} else {
  console.log('Invalid or corrupted message')
}
```

### Verify Author
```javascript
const msg = await apds.get(hash)
// First 44 chars of signature = author's pubkey
const author = msg.substring(0, 44)
console.log('Message from:', author)
```

## Best Practices

1. **Always use `compose()`** for user messages - it handles frontmatter automatically
2. **Use `sign()`** only for raw data that doesn't need metadata
3. **Set profile once** at startup - it persists in storage
4. **Check for `previous`** when building message chains
5. **Parse YAML safely** - handle missing fields gracefully

## Troubleshooting

### Message Not Showing Metadata
- Ensure `name` is set: `await apds.put('name', 'YourName')`
- Check storage namespace matches

### Previous Chain Broken
- `previous` is set from last `compose()` call
- Switching namespaces breaks the chain
- Use `await apds.get('previous')` to check current value

### YAML Parse Errors
- Ensure proper `---` delimiters
- Escape special YAML characters in content
- Check for unbalanced quotes
