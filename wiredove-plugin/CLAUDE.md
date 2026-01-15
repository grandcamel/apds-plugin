# Wiredove Plugin for Claude Code

This plugin provides skills, agents, and commands to support developers working with the Wiredove decentralized social networking application.

## About Wiredove

Wiredove is an experimental distributed social networking application that runs entirely client-side in browsers with local-first data storage.

**Repository**: https://github.com/evbogue/wiredove
**Live Instance**: https://wiredove.net

## Technology Stack

- **ANProto**: Message authentication and cryptographic signing
- **APDS**: Message distribution across the network
- **Trystero**: Serverless P2P collaboration via WebRTC
- **BitTorrent DHT**: WebRTC connection bootstrapping

## Key Concepts

### Messages
- Include author identification, timestamps, and hash chains
- Cryptographically verified using ANProto
- Link to previous posts via hashes

### Feed Synchronization
- Peers compare latest posts to sync feeds
- Uses Trystero rooms keyed by public keys

### Directory Services
- Maps usernames to public key arrays
- Enables identity resolution in the network

## Development Guidelines

1. **Local-First**: All data storage is client-side
2. **No Servers Required**: Application is fully decentralized
3. **Cryptographic Identity**: Users identified by public keys
4. **P2P Communication**: WebRTC via Trystero

## Plugin Components

- `/commands` - CLI commands for wiredove development tasks
- `/skills` - Specialized knowledge about wiredove architecture
- `/agents` - Autonomous agents for code exploration and implementation
- `/hooks` - Event-driven automation for development workflow
