---
description: Use this skill when developers ask about Wiredove architecture, how it works, or need guidance on getting started with Wiredove development. Provides comprehensive overview of the decentralized social networking stack.
---

# Wiredove Architecture Overview

Wiredove is a decentralized social networking application designed to run entirely in the browser with no server dependencies.

**Repository**: https://github.com/evbogue/wiredove
**Live Demo**: https://wiredove.net

## Core Technologies

| Component | Purpose | File |
|-----------|---------|------|
| **APDS** | Local-first storage, cryptographic signing | via ESM import |
| **Trystero** | P2P WebRTC via BitTorrent DHT | gossip.js |
| **WebSocket** | Pub server sync | websocket.js |
| **ntfy.sh** | Push notifications | ntfy.js |

## Message Structure

### Signed Message (sig)
```
[author_pubkey:44][signature:86][timestamp:13][content_hash:43]
```

### Content Blob (YAML)
```yaml
name: "Author Name"
image: "hash_of_avatar"
body: "Message in **markdown**"
previous: "hash_of_previous_post"
reply: "hash_of_parent"        # if reply
edit: "hash_of_original"       # if edit
```

## Key Source Files

| File | Lines | Purpose |
|------|-------|---------|
| `app.js` | 15 | Entry point, initializes all systems |
| `route.js` | 110 | URL hash-based routing |
| `render.js` | 928 | Message rendering, edits, replies |
| `composer.js` | 200 | Message composition |
| `sync.js` | 153 | Tiered sync scheduler |
| `gossip.js` | 77 | Trystero P2P rooms |
| `websocket.js` | 145 | WebSocket transport |
| `network_queue.js` | 114 | Unified send queue |

## Development Setup

```bash
# Clone the repository
git clone https://github.com/evbogue/wiredove.git
cd wiredove

# Run with Docker
docker compose up --build

# Or serve directly
npx serve .

# Open browser to http://localhost:3000 (or 8000 with Docker)
```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                       Browser                                │
├─────────────────────────────────────────────────────────────┤
│  app.js → route.js → render.js → composer.js                │
├─────────────────────────────────────────────────────────────┤
│                    sync.js (Scheduler)                       │
│         ┌──────────┬──────────┬──────────┐                  │
│         │   HOT    │   WARM   │   COLD   │                  │
│         └──────────┴──────────┴──────────┘                  │
├─────────────────────────────────────────────────────────────┤
│                  network_queue.js                            │
│              ┌─────────────────────┐                        │
│              │    Unified Queue    │                        │
│              └──────────┬──────────┘                        │
│         ┌───────────────┴───────────────┐                   │
│         ▼                               ▼                   │
│  ┌─────────────┐                ┌─────────────┐            │
│  │  WebSocket  │                │   Trystero  │            │
│  │ (websocket) │                │  (gossip)   │            │
│  └──────┬──────┘                └──────┬──────┘            │
└─────────┼───────────────────────────────┼───────────────────┘
          │                               │
          ▼                               ▼
   pub.wiredove.net              BitTorrent DHT (P2P)
```

## Related Skills

For detailed information on specific components:
- `apds-integration` - Data storage and cryptography
- `trystero-p2p` - P2P room management
- `websocket-transport` - Server sync
- `message-handling` - Message lifecycle
- `feed-sync` - Tiered synchronization
- `ui-components` - UI rendering

## Quick Reference

### Import Pattern
```javascript
import { apds } from 'apds'
import { h } from 'h'
```

### Create a Message
```javascript
const hash = await apds.compose('Hello, Wiredove!')
const signed = await apds.get(hash)
await send(signed)
```

### Verify a Message
```javascript
const opened = await apds.open(signedBlob)
if (opened) {
  const author = signedBlob.substring(0, 44)
  const timestamp = opened.substring(0, 13)
  const contentHash = opened.substring(13)
}
```
