---
name: anproto-explain
description: Use this skill when the user asks "what is anproto", "explain anproto", "how does anproto work", "anproto vs nostr", "anproto vs atproto", or wants to understand the ANProto protocol concepts, architecture, or design philosophy.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
---

# ANProto Explainer

## What is ANProto?

**ANProto** = **A**uthenticated and **N**on-networked Protocol (or **AN**other Protocol)

A lightweight decentralized authentication and messaging protocol that uses ed25519 keypairs to sign messages containing timestamps and content hashes, all encoded in base64.

**Repository**: https://github.com/evbogue/anproto

## Core Concepts

### Design Philosophy

ANProto is the spiritual successor to [Secure Scuttlebot](https://scuttlebot.io), designed with these principles:

1. **Extreme Simplicity**: The entire JavaScript implementation is ~37 lines of code
2. **Network Agnostic**: Works over ANY transport - HTTP, WebSocket, email, USB, Bluetooth, LoRa
3. **No Infrastructure Lock-in**: Unlike ATProto, no required networking infrastructure
4. **Universal Appeal**: Unlike Nostr, designed to reach beyond cryptocurrency communities

### How It Works

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Content    │────▶│  SHA-256     │────▶│   Sign with  │
│ "Hello World"│     │  Hash        │     │  ed25519 key │
└──────────────┘     └──────────────┘     └──────────────┘
                            │                     │
                            ▼                     ▼
                     ┌──────────────┐     ┌──────────────┐
                     │ pZGm1Av0... │     │ BSY7/er4... │
                     │ (44 chars)   │     │ + signature  │
                     └──────────────┘     └──────────────┘
```

### The Four Core Functions

| Function | Purpose | Input | Output |
|----------|---------|-------|--------|
| `gen()` | Generate keypair | none | 132-char keypair string |
| `hash(data)` | Hash content | string | 44-char base64 hash |
| `sign(hash, key)` | Sign with timestamp | hash + keypair | signed message |
| `open(msg)` | Verify & extract | signed message | timestamp + hash |

## Message Format Specification

### Keypair Format (132 characters)
```
[publicKey:44][secretKey:88]
```
Example:
```
BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=tQa03kqUWG3VtHZ98++lHFBeQ4JKZwuTH2CjC/K6P8EFJjv96vhUki7Tyjf01pECI9j8wC93uhCGUYJEMAGNhQ==
└───────────── public key ─────────────┘└──────────────────────── secret key ────────────────────────┘
```

### Signed Message Format
```
[publicKey:44][signedPayload:variable]
```

### Opened Message Format (57 characters)
```
[timestamp:13][hash:44]
```
Example:
```
1755197841319pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=
└─ timestamp ─┘└───────────────── hash ─────────────────┘
```

## Comparison with Alternatives

| Feature | ANProto | ATProto (Bluesky) | Nostr | Scuttlebot |
|---------|---------|-------------------|-------|------------|
| Code size | ~37 lines | Massive | Medium | Large |
| Network required | No | Yes (PDS) | Relays | Yes |
| Crypto | ed25519 | Various | secp256k1 | ed25519 |
| Transport | Any | HTTPS | WebSocket | TCP |
| Complexity | Minimal | High | Low | Medium |

## Language Implementations

| Language | Repository | Status |
|----------|------------|--------|
| JavaScript | [evbogue/anproto](https://github.com/evbogue/anproto) | Primary |
| Go | [vic/goan](https://github.com/vic/goan) | Complete |
| Rust | [vic/anproto-rs](https://github.com/vic/anproto-rs) | Complete |
| Python | [macauleyjustin/ANproto-Python](https://github.com/macauleyjustin/ANproto-Python) | Complete |

## Ecosystem

### APDS (A Personal Data Server)
Server component for storing and authenticating ANProto messages.

### Wiredove
Proof-of-concept PWA client using Trystero and WebSockets.

## Quick Start Example

```javascript
import { an } from './an.js'

// 1. Generate identity (once, store securely)
const keypair = await an.gen()

// 2. Create and hash content
const content = "Hello World"
const hash = await an.hash(content)

// 3. Sign the hash
const signed = await an.sign(hash, keypair)

// 4. Recipient verifies and opens
const opened = await an.open(signed)

// 5. Extract components
const timestamp = opened.substring(0, 13)
const extractedHash = opened.substring(13)
```

## Resources

- **Demo**: https://anproto.com/try
- **Main Site**: https://anproto.com
- **Client**: https://wiredove.net
