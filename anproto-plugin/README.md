# ANProto Claude Code Plugin

A Claude Code plugin to assist developers working with [ANProto](https://github.com/evbogue/anproto) - a lightweight decentralized authentication and messaging protocol using ed25519 keypairs.

## Installation

```bash
# Add the marketplace
claude plugin marketplace add /path/to/anproto-plugin

# Install the plugin
claude plugin install anproto-plugin
```

## Features

### Skills (Automatic Context Loading)

| Skill | Trigger Phrases | Purpose |
|-------|-----------------|---------|
| `anproto-explain` | "what is anproto", "explain anproto" | Protocol concepts and architecture |
| `anproto-setup` | "setup anproto", "add anproto" | Project initialization guidance |
| `anproto-debug` | "debug message", "parse anproto" | Message parsing and troubleshooting |
| `anproto-keygen` | "generate keypair", "create keys" | Key generation and storage |
| `anproto-sign` | "sign message", "create signature" | Signing workflow guidance |
| `anproto-verify` | "verify message", "check signature" | Verification workflow guidance |

### Commands (Slash Commands)

| Command | Purpose |
|---------|---------|
| `/anproto-init` | Initialize ANProto in your project (Deno/Node/Browser) |
| `/anproto-parse` | Parse and explain an ANProto message structure |

### Agents (Specialized Tasks)

| Agent | Purpose |
|-------|---------|
| `anproto-explorer` | Analyze ANProto usage patterns in a codebase |
| `anproto-reviewer` | Review ANProto implementations for security and best practices |

## Quick Start

### Explain ANProto
```
What is ANProto and how does it compare to Nostr?
```

### Set Up a Project
```
/anproto-init
```
Or just ask: "Help me add ANProto to my Node.js project"

### Parse a Message
```
/anproto-parse BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=yVpD8i7...
```

### Review Code
```
Review my ANProto implementation for security issues
```

## ANProto Overview

ANProto is a minimal protocol where:
- **ed25519 keypairs** provide identity and signing
- **SHA-256 hashes** ensure content integrity
- **Base64 encoding** enables transport over any medium
- **No network required** - works over HTTP, WebSocket, USB, email, etc.

### Core API (4 functions)

```javascript
import { an } from './an.js'

// Generate identity
const keypair = await an.gen()

// Hash content
const hash = await an.hash("Hello World")

// Sign (includes timestamp)
const signed = await an.sign(hash, keypair)

// Verify and open
const opened = await an.open(signed)
```

### Message Formats

| Type | Length | Format |
|------|--------|--------|
| Keypair | 132 chars | `[publicKey:44][secretKey:88]` |
| Hash | 44 chars | Base64 SHA-256 |
| Signed Message | Variable | `[publicKey:44][signedPayload]` |
| Opened Message | 57 chars | `[timestamp:13][hash:44]` |

## Project Structure

```
anproto-plugin/
├── plugin.json              # Plugin manifest
├── .claude-plugin/
│   └── marketplace.json     # Marketplace definition
├── skills/
│   ├── anproto-explain.md   # Protocol explainer
│   ├── anproto-setup.md     # Project setup guide
│   ├── anproto-debug.md     # Message debugger
│   ├── anproto-keygen.md    # Key generation guide
│   ├── anproto-sign.md      # Signing guide
│   └── anproto-verify.md    # Verification guide
├── commands/
│   ├── anproto-init.md      # Project initializer
│   └── anproto-parse.md     # Message parser
├── agents/
│   ├── anproto-explorer.md  # Codebase analyzer
│   └── anproto-reviewer.md  # Code reviewer
├── anproto/                 # Cloned ANProto source (reference)
├── RESEARCH.md              # Deep research notes
└── PLAN.md                  # Implementation plan
```

## Language Support

The plugin helps with ANProto in multiple languages:

| Language | Repository | Support Level |
|----------|------------|---------------|
| JavaScript | [evbogue/anproto](https://github.com/evbogue/anproto) | Full |
| Go | [vic/goan](https://github.com/vic/goan) | Guidance |
| Rust | [vic/anproto-rs](https://github.com/vic/anproto-rs) | Guidance |
| Python | [macauleyjustin/ANproto-Python](https://github.com/macauleyjustin/ANproto-Python) | Guidance |

## Resources

- **ANProto Demo**: https://anproto.com/try
- **Wiredove Client**: https://wiredove.net
- **Main Repository**: https://github.com/evbogue/anproto

## License

MIT
