# APDS Research Overview

## Project Summary

**APDS** (ANProto Personal Data Server) is a personal data server implementation for ANProto, a decentralized social protocol.

- **Repository**: https://github.com/evbogue/apds
- **Live Instance**: https://apds.anproto.com
- **License**: MIT
- **Runtime**: Deno (JavaScript)

## Architecture

### Core Components

| File | Purpose | Lines |
|------|---------|-------|
| `apds.js` | Core library - key/value storage, message signing, querying | 239 |
| `serve.js` | HTTP/WebSocket server (port 9000) | 135 |
| `cli.js` | Interactive terminal client | 87 |
| `gossip.js` | P2P gossip protocol for missing messages | 53 |
| `relay.js` | Simple WebSocket relay server | 42 |
| `example.js` | Browser-based client example | 27 |
| `composer.js` | UI message composition component | 27 |
| `profile.js` | User profile UI component | 91 |
| `render.js` | Message rendering | ~50 |
| `trender.js` | Terminal message rendering | ~20 |

### Library Dependencies (`lib/`)

| File | Purpose |
|------|---------|
| `base64.js` | Base64 encoding/decoding |
| `cachekv.js` | Key-value cache storage |
| `ed2curve.js` | Ed25519 to Curve25519 conversion |
| `frontmatter.js` | YAML frontmatter parsing |
| `h.js` | DOM element helper |
| `human.js` | Human-readable timestamps |
| `marked.esm.js` | Markdown parser |
| `nacl-fast-es.js` | NaCl cryptography library |
| `vb.js` | Visual blocky avatar generator |
| `yaml.js` | YAML parsing/creation |

### External Dependencies

```javascript
// ANProto core library
import { an } from 'https://esm.sh/gh/evbogue/anproto@ddc040c/an.js'

// Deno standard library
import { serveDir } from 'https://deno.land/std/http/file_server.ts'
```

## Key APIs

### `apds` Object (apds.js)

```javascript
// Initialization
apds.start(appId)           // Start with app ID for storage isolation
apds.generate()             // Generate new keypair
apds.keypair()              // Get full keypair
apds.pubkey()               // Get public key (first 44 chars)
apds.privkey()              // Get private key

// Storage
apds.put(key, value)        // Store key-value pair
apds.get(hash)              // Retrieve by hash
apds.rm(key)                // Remove key
apds.clear()                // Clear all data

// Messages
apds.compose(content, prev) // Create signed message with metadata
apds.sign(data)             // Sign data with keypair
apds.open(msg)              // Verify and open signed message
apds.add(msg)               // Add message to log
apds.make(data)             // Hash and store data

// Queries
apds.query(query)           // Query messages (by author, hash, or search)
apds.getHashLog()           // Get all message hashes
apds.getOpenedLog()         // Get all opened messages
apds.getPubkeys()           // Get unique authors
apds.getLatest(pubkey)      // Get latest message from author

// Utilities
apds.hash(data)             // Hash data
apds.human(ts)              // Human-readable timestamp
apds.visual(pubkey)         // Generate visual avatar
apds.parseYaml(doc)         // Parse YAML
apds.createYaml(obj, content) // Create YAML with frontmatter
```

### CLI Commands (cli.js)

```
/nick <name>      - Set display name
/connect <url>    - Connect to WebSocket peer
/clear            - Erase all local data
/exit             - Exit CLI
```

### HTTP Endpoints (serve.js)

```
GET  /all         - All messages
GET  /latest      - Messages from last 5 minutes
GET  /<hash>      - Get specific hash
GET  /<pubkey>    - Get messages from author
GET  /?<search>   - Search messages
WS   /            - WebSocket for real-time sync
```

## Developer Workflows

### 1. Initial Server Setup
```bash
# Run the server
deno run -A serve.js

# Server starts on port 9000
# Generates keypair on first run
# Creates local storage in 'apdsv1' namespace
```

### 2. CLI Client Usage
```bash
# Start CLI client
deno run -A cli.js [namespace]

# Set nickname
/nick MyName

# Connect to server
/connect wss://apds.anproto.com

# Type messages directly
```

### 3. Running a Relay
```bash
deno run -A relay.js [port]
# Default port: 8080
```

### 4. Browser Client Development
```javascript
// Import and initialize
import { apds } from './apds.js'
await apds.start('myapp')

// Generate keypair if needed
if (!await apds.pubkey()) {
  await apds.put('keypair', await apds.generate())
}

// Compose and send messages
const hash = await apds.compose('Hello world!')
```

## Data Structures

### Message Object
```javascript
{
  hash: "base64-hash==",
  sig: "signature==",
  author: "pubkey-44-chars",
  opened: "timestamp-hash",
  text: "raw-content",
  ts: "1234567890123"
}
```

### YAML Message Format
```yaml
---
name: Username
image: hash-of-avatar
previous: hash-of-previous-message
---
Message body content here
```

## Configuration

### Environment Variables
- `VAPID_SUBJECT` - Email or URL for Web Push (server mode)

### Local Files (auto-generated)
- `subscriptions.json` - Web Push subscriptions
- `vapid.json` - VAPID credentials

### Storage Namespaces
- `apdsv1` - Default server namespace
- `default` - Default CLI namespace
- Custom via first argument

## Common Developer Pain Points

1. **Initial Setup Confusion** - Need to understand Deno permissions, ports, namespaces
2. **Peer Discovery** - Manual WebSocket URL entry required
3. **Debugging Sync Issues** - Hard to trace message propagation
4. **Key Management** - Keypair stored in browser/file storage
5. **YAML Format Requirements** - Specific frontmatter structure needed
6. **WebSocket State** - Connection management and reconnection

## Plugin Opportunities

### Skills to Build
1. **apds-setup** - Guided server/client initialization
2. **apds-debug** - WebSocket and sync troubleshooting
3. **apds-connect** - Peer discovery and management
4. **apds-message** - Message composition helpers

### Agents to Build
1. **apds-explorer** - Codebase navigation and explanation
2. **apds-troubleshooter** - Issue diagnosis

### Commands to Build
1. `/apds-init` - Quick project scaffold
2. `/apds-run` - Start server with correct permissions
3. `/apds-connect` - Connect to known peers
