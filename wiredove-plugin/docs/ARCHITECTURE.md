# Wiredove Architecture Documentation

## Overview

Wiredove is a decentralized social networking application that runs entirely in the browser. It uses local-first data storage (IndexedDB via APDS), cryptographic identity (ANProto), and peer-to-peer communication (Trystero/WebRTC + WebSocket).

## File Map & Relationships

```
wiredove/
├── index.html          # App shell, loads modules via importmap
├── app.js              # Entry point - initializes all systems
├── route.js            # URL hash router for views
├── connect.js          # Network connection orchestrator
│
├── [Network Layer]
│   ├── gossip.js       # Trystero P2P rooms (WebRTC)
│   ├── websocket.js    # WebSocket connections to pub servers
│   ├── network_queue.js# Unified send queue for both transports
│   └── send.js         # Simple send wrapper
│
├── [Sync Layer]
│   └── sync.js         # Tiered sync scheduler (hot/warm/cold)
│
├── [UI Layer]
│   ├── render.js       # Message rendering, edits, replies
│   ├── composer.js     # Message composition with preview
│   ├── navbar.js       # Navigation bar
│   ├── adder.js        # Feed loading/pagination
│   ├── profile.js      # Avatar and name components
│   └── markdown.js     # Markdown rendering
│
├── [Identity Layer]
│   ├── identify.js     # Keypair generation (vanity support)
│   └── settings.js     # Key import/export, settings UI
│
├── [Notifications]
│   ├── notifications.js# Web Push notifications
│   ├── ntfy.js         # ntfy.sh integration
│   └── notifications_server.js # Server-side push handler
│
├── [Assets]
│   ├── style.css       # Styling
│   ├── sw.js           # Service worker
│   └── *.png           # Logo assets
│
└── [External Dependencies]
    ├── apds (via ESM)  # Data storage & crypto (from evbogue/apds)
    ├── h (via ESM)     # DOM helper from apds
    └── trystero-torrent.min.js # P2P WebRTC via BitTorrent DHT
```

## Initialization Flow

```
index.html
    └── app.js
        ├── apds.start('wiredovedbversion1')  # Initialize IndexedDB
        ├── navbar()                           # Render navigation
        ├── route()                            # Handle current URL
        ├── connect()                          # Start network connections
        │   ├── makeWs('wss://pub.wiredove.net/')  # WebSocket to pub
        │   └── makeRoom('wiredovev1')             # Trystero P2P room
        └── startSync(send)                    # Begin sync scheduler
```

## Data Flow

### Message Creation
```
composer.js
    ├── apds.compose(text, options)    # Create signed message
    ├── ntfy(signed)                   # Broadcast to ntfy.sh
    ├── send(signed)                   # Queue for network
    │   └── network_queue.queueSend()
    │       ├── websocket.sendWs()     # Send via WebSocket
    │       └── gossip.sendTry()       # Send via Trystero
    └── render.blob(signed)            # Display locally
```

### Message Reception
```
websocket.js / gossip.js
    ├── noteReceived(data)             # Remove from pending queue
    ├── apds.make(data)                # Verify signature
    ├── render.shouldWe(data)          # Check if relevant
    ├── apds.add(data)                 # Store in IndexedDB
    └── render.blob(data)              # Display message
```

### Sync Process
```
sync.js (every 15 seconds)
    ├── refreshPubkeys()               # Get known pubkeys from APDS
    ├── rebuildTiers()                 # Classify as hot/warm/cold
    │   ├── hot: <15min interest OR <24h seen → 4 per tick
    │   ├── warm: <24h interest OR <30d seen → 2 per tick
    │   └── cold: everything else → 1 per tick
    └── pickCandidates()               # Select pubkeys to sync
        └── send(pubkey)               # Request latest from network
```

## Key Concepts

### Message Structure (ANProto/APDS)
```
Signed Message (sig):
  [author_pubkey:44][signature:86][timestamp:13][content_hash:43]

Content Blob (YAML):
  name: "Author Name"
  image: "hash_of_avatar"
  body: "Message content in markdown"
  previous: "hash_of_previous_message"
  reply: "hash_of_parent_if_reply"
  edit: "hash_of_original_if_edit"
```

### Network Transports

| Transport | Purpose | Protocol |
|-----------|---------|----------|
| WebSocket | Pub server sync | wss://pub.wiredove.net/ |
| Trystero | Peer-to-peer | WebRTC via BitTorrent DHT |

### Routing Patterns (route.js)

| URL Hash | Behavior |
|----------|----------|
| `#` | Show all messages |
| `#settings` | Settings page |
| `#import` | Import data |
| `#[44 chars]` | Profile view (pubkey) |
| `#[<44 chars]` | Username lookup via pub.wiredove.net |
| `#[>44 chars]` | Direct message/blob view |
| `#?...` | Search query |

## Component Responsibilities

### render.js (928 lines)
- `render.blob()` - Main entry for rendering any message
- `render.meta()` - Render message metadata (author, time, controls)
- `render.content()` - Render message body with markdown
- `render.comments()` - Reply UI and threading
- `render.refreshEdits()` - Edit history navigation
- `insertByTimestamp()` - Chronological insertion

### sync.js (153 lines)
- Tiered sync prioritization
- Activity tracking (lastSeen, lastInterest)
- Rate limiting per tier
- Bootstrap activity from stored messages

### network_queue.js (114 lines)
- Unified queue for WS and gossip
- Deduplication via pending map
- Delayed drain for batching
- Target-specific sending (ws/gossip/both)

## External Dependencies

### APDS (Application Protocol Data Store)
Source: https://esm.sh/gh/evbogue/apds

Key functions used:
- `apds.start(dbname)` - Initialize database
- `apds.compose(text, options)` - Create signed message
- `apds.open(sig)` - Verify and extract message
- `apds.get(hash)` / `apds.put(key, value)` - Key-value storage
- `apds.query(filter)` - Query messages
- `apds.pubkey()` / `apds.keypair()` - Identity management
- `apds.parseYaml(blob)` - Parse message content
- `apds.hash(blob)` - Content addressing
- `apds.human(timestamp)` - Human-readable time

### Trystero
Source: trystero-torrent.min.js (bundled)

Key functions:
- `joinRoom({appId, password}, roomId)` - Join P2P room
- `room.makeAction(name)` - Create send/receive pair
- `room.onPeerJoin/Leave()` - Peer lifecycle events
