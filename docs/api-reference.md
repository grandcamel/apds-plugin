# APDS API Reference

Complete API reference for the APDS library.

## Initialization

### `apds.start(appId)`

Initialize APDS with a storage namespace.

```javascript
await apds.start('my-app')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `appId` | string | Namespace for storage isolation |

**Returns**: `Promise<void>`

---

## Identity Management

### `apds.generate()`

Generate a new Ed25519 keypair.

```javascript
const keypair = await apds.generate()
// Returns: 88-character string (44 pubkey + 44 privkey)
```

**Returns**: `Promise<string>` - Full keypair

### `apds.keypair()`

Get the stored keypair.

```javascript
const keypair = await apds.keypair()
```

**Returns**: `Promise<string|undefined>` - Keypair or undefined if not set

### `apds.pubkey()`

Get public key (first 44 characters of keypair).

```javascript
const pubkey = await apds.pubkey()
// Returns: "ABC123..." (44 chars)
```

**Returns**: `Promise<string|undefined>`

### `apds.privkey()`

Get private key (last 44 characters of keypair).

```javascript
const privkey = await apds.privkey()
```

**Returns**: `Promise<string|undefined>`

### `apds.deletekey()`

Delete the stored keypair.

```javascript
await apds.deletekey()
```

**Returns**: `Promise<void>`

---

## Storage Operations

### `apds.put(key, value)`

Store a key-value pair.

```javascript
await apds.put('name', 'Alice')
await apds.put('settings', JSON.stringify({ theme: 'dark' }))
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Storage key |
| `value` | string | Value to store |

**Returns**: `Promise<void>`

### `apds.get(key)`

Retrieve a value by key or hash.

```javascript
const name = await apds.get('name')
const message = await apds.get('ABC123hash==')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Key or hash to retrieve |

**Returns**: `Promise<string|undefined>`

### `apds.rm(key)`

Remove a key-value pair.

```javascript
await apds.rm('temporary-data')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | string | Key to remove |

**Returns**: `Promise<void>`

### `apds.clear()`

Clear all stored data.

```javascript
await apds.clear()
```

**Returns**: `Promise<void>`

### `apds.make(data)`

Hash data and store it. Returns the hash.

```javascript
const hash = await apds.make('Hello, world!')
// Returns: "XYZ789..." (44 chars)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | string | Data to hash and store |

**Returns**: `Promise<string>` - Content hash

### `apds.hash(data)`

Hash data without storing.

```javascript
const hash = await apds.hash('some data')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | string | Data to hash |

**Returns**: `Promise<string>` - Hash

---

## Message Operations

### `apds.sign(data)`

Sign data with the stored keypair.

```javascript
const hash = await apds.sign('Message content')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | string | Data to sign |

**Returns**: `Promise<string>` - Hash of the signed message

### `apds.open(signature)`

Verify and open a signed message.

```javascript
const opened = await apds.open(signedMessage)
if (opened) {
  const timestamp = opened.substring(0, 13)
  const contentHash = opened.substring(13)
}
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `signature` | string | Signed message (ends with `==`) |

**Returns**: `Promise<string|undefined>` - `timestamp + contentHash` or undefined if invalid

### `apds.compose(content, metadata?)`

Create a signed message with YAML frontmatter.

```javascript
// Basic
const hash = await apds.compose('Hello!')

// With custom metadata
const hash = await apds.compose('Hello!', {
  topic: 'general',
  reply_to: 'someHash=='
})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `content` | string | Message body |
| `metadata` | object | Optional custom frontmatter fields |

**Returns**: `Promise<string>` - Hash of composed message

**Note**: Automatically includes `name`, `image`, and `previous` from storage if set.

### `apds.add(message)`

Add a signed message to the log.

```javascript
const added = await apds.add(signedMessage)
// Returns: true if added, undefined if invalid/duplicate
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `message` | string | Signed message to add |

**Returns**: `Promise<boolean|undefined>`

---

## Query Operations

### `apds.query(query?)`

Query messages from the log.

```javascript
// All messages
const all = await apds.query()

// By author pubkey
const byAuthor = await apds.query('ABC123pubkey...')

// By hash
const byHash = await apds.query('XYZ789hash==')

// Search content (prefix with ?)
const search = await apds.query('?keyword')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | string | Optional: pubkey, hash, or `?search` |

**Returns**: `Promise<MessageObject[]>`

**MessageObject structure:**
```javascript
{
  hash: "contentHash==",      // 44-char hash
  sig: "signature==",         // Full signature
  author: "pubkey",           // 44-char author pubkey
  opened: "timestamphash",    // Opened signature content
  text: "raw content",        // Raw message content
  ts: "1234567890123"         // Unix timestamp (ms)
}
```

### `apds.getHashLog()`

Get array of all message hashes.

```javascript
const hashes = await apds.getHashLog()
// Returns: ["hash1==", "hash2==", ...]
```

**Returns**: `Promise<string[]>`

### `apds.getOpenedLog()`

Get array of all parsed message objects.

```javascript
const messages = await apds.getOpenedLog()
```

**Returns**: `Promise<MessageObject[]>`

### `apds.getPubkeys()`

Get unique author public keys.

```javascript
const authors = await apds.getPubkeys()
// Returns: ["pubkey1", "pubkey2", ...]
```

**Returns**: `Promise<string[]>`

### `apds.getLatest(pubkey)`

Get the most recent message from an author.

```javascript
const latest = await apds.getLatest('ABC123pubkey...')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `pubkey` | string | Author's public key |

**Returns**: `Promise<MessageObject|undefined>`

---

## YAML Operations

### `apds.parseYaml(document)`

Parse YAML frontmatter from a message.

```javascript
const yaml = await apds.parseYaml(messageContent)
// Returns: { name: "Alice", body: "Hello!", ... }
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `document` | string | Content with YAML frontmatter |

**Returns**: `Promise<object>` - Parsed YAML with `body` containing content after frontmatter

### `apds.createYaml(metadata, content)`

Create content with YAML frontmatter.

```javascript
const yaml = await apds.createYaml(
  { name: 'Alice', topic: 'general' },
  'Hello, world!'
)
// Returns:
// ---
// name: Alice
// topic: general
// ---
// Hello, world!
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `metadata` | object | Frontmatter fields |
| `content` | string | Body content |

**Returns**: `Promise<string>`

---

## Utility Functions

### `apds.human(timestamp)`

Convert timestamp to human-readable string.

```javascript
const readable = await apds.human('1234567890123')
// Returns: "2 hours ago" or similar
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `timestamp` | string | Unix timestamp in milliseconds |

**Returns**: `Promise<string>`

### `apds.visual(pubkey)`

Generate a visual avatar from a public key.

```javascript
const avatarElement = await apds.visual('ABC123pubkey...')
// Returns: HTMLImageElement with generated avatar
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `pubkey` | string | Public key to visualize |

**Returns**: `Promise<HTMLImageElement>` - 256x256 visual blocky avatar

---

## Gossip Module

### `createGossip(options)`

Create a gossip queue for P2P sync.

```javascript
import { createGossip } from './gossip.js'

const gossip = createGossip({
  getPeers: () => sockets,
  has: async (hash) => !!(await apds.get(hash)),
  request: (peer, hash) => peer.send(hash),
  intervalMs: 10000
})

gossip.start()
```

| Option | Type | Description |
|--------|------|-------------|
| `getPeers` | function | Returns Set/Array of WebSocket peers |
| `has` | async function | Check if hash exists locally |
| `request` | function | Send hash request to peer |
| `intervalMs` | number | Retry interval (default: 10000) |

**Returns**: `GossipQueue`

### GossipQueue Methods

```javascript
gossip.start()           // Start the gossip loop
gossip.stop()            // Stop the gossip loop
gossip.enqueue(hash)     // Add hash to missing queue
gossip.resolve(hash)     // Mark hash as resolved
```

---

## HTTP Endpoints (serve.js)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/all` | GET | All messages as JSON array |
| `/latest` | GET | Messages from last 5 minutes |
| `/<hash>` | GET | Get content by hash |
| `/<pubkey>` | GET | Messages from specific author |
| `/?<search>` | GET | Search messages |
| `/push/vapidPublicKey` | GET | Get VAPID public key |
| `/push/subscribe` | POST | Store push subscription |
| `/push/test` | POST | Send test notification |
| `/` | WS | WebSocket for real-time sync |

---

## Constants

| Name | Value | Description |
|------|-------|-------------|
| Pubkey length | 44 | Characters in public key |
| Keypair length | 88 | Characters in full keypair |
| Hash length | 44 | Characters in content hash |
| Default port | 9000 | Server port |
| Gossip interval | 10000 | Milliseconds between gossip ticks |
| Sort interval | 20000 | Milliseconds between log sorts |
