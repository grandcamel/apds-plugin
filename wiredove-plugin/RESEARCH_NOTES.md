# Wiredove Plugin Research Notes

## Project Overview

**Goal**: Create a Claude Code plugin to support developers working with the Wiredove decentralized social networking application.

**Repository**: https://github.com/evbogue/wiredove
**Live Demo**: https://wiredove.net

## Research Findings

### Technology Stack
| Component | Purpose |
|-----------|---------|
| APDS | Local-first data storage, cryptographic signing, message queries |
| Trystero | Serverless P2P via WebRTC + BitTorrent DHT |
| WebSocket | Pub server sync (wss://pub.wiredove.net/) |
| ntfy.sh | Push notification integration |

### Codebase Composition
- JavaScript: 92% (42 files, ~3000 lines)
- CSS: 6.4% (style.css)
- HTML: 1.5% (index.html)

### Architecture Characteristics
- **Local-first**: IndexedDB storage via APDS
- **Dual transport**: WebSocket + P2P (Trystero)
- **Cryptographic identity**: Ed25519 keypairs
- **Hash-based content addressing**: 44-char base64 hashes
- **Tiered sync**: Hot/warm/cold prioritization

### Key Source Files
| File | Lines | Purpose |
|------|-------|---------|
| render.js | 928 | Message rendering, edits, replies |
| composer.js | 200 | Message composition UI |
| sync.js | 153 | Tiered sync scheduler |
| websocket.js | 145 | WebSocket transport |
| route.js | 110 | URL hash routing |
| network_queue.js | 114 | Unified send queue |
| gossip.js | 77 | Trystero P2P rooms |

## Completed Plugin Components

### Documentation
- [x] `docs/ARCHITECTURE.md` - Complete architecture documentation
- [x] `docs/DEVELOPMENT_PATTERNS.md` - Common coding patterns

### Skills (Knowledge Base)
- [x] `wiredove-overview` - Architecture and getting started
- [x] `apds-integration` - Working with APDS for storage/crypto
- [x] `trystero-p2p` - P2P room management and WebRTC
- [x] `message-handling` - Message structure, signing, distribution
- [x] `feed-sync` - Tiered synchronization patterns
- [x] `websocket-transport` - WebSocket connections
- [x] `ui-components` - UI rendering and components

### Agents
- [x] `wiredove-explorer` - Explore and understand wiredove codebase
- [x] `wiredove-implementer` - Implement features following patterns

### Commands
- [x] `/wiredove-setup` - Initialize development environment
- [x] `/wiredove-debug` - Debug P2P connections and sync

## Related Projects

- **anproto-plugin**: Existing plugin for ANProto (may be useful to reference)
- **dovepub**: Pub server implementation (https://github.com/evbogue/dovepub)

## Resources

- Wiredove GitHub: https://github.com/evbogue/wiredove
- APDS Source: https://esm.sh/gh/evbogue/apds
- Trystero Documentation: https://github.com/dmotz/trystero
- ntfy.sh (notifications): https://ntfy.sh
