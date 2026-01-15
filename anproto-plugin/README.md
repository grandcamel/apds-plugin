# ANProto Claude Code Plugin

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-purple)

A Claude Code plugin to assist developers working with [ANProto](https://github.com/evbogue/anproto) - a lightweight decentralized authentication and messaging protocol using ed25519 keypairs.

## Installation

```bash
# Add the marketplace
claude plugin marketplace add /path/to/anproto-plugin

# Install the plugin
claude plugin install anproto-plugin
```

## Features

### Skills (7 total)

| Skill | Triggers | Purpose |
|-------|----------|---------|
| `anproto-marketplace` | "list skills", "anproto help" | Browse all plugin capabilities |
| `anproto-explain` | "what is anproto", "explain anproto" | Protocol concepts and architecture |
| `anproto-setup` | "setup anproto", "add anproto" | Project initialization guidance |
| `anproto-debug` | "debug message", "parse anproto" | Message parsing and troubleshooting |
| `anproto-keygen` | "generate keypair", "create keys" | Key generation and storage |
| `anproto-sign` | "sign message", "create signature" | Signing workflow guidance |
| `anproto-verify` | "verify message", "check signature" | Verification workflow guidance |

### Commands (2 total)

| Command | Purpose |
|---------|---------|
| `/anproto-init` | Initialize ANProto in your project (Deno/Node/Browser) |
| `/anproto-parse` | Parse and explain an ANProto message structure |

### Agents (2 total)

| Agent | Purpose |
|-------|---------|
| `anproto-explorer` | Analyze ANProto usage patterns in a codebase |
| `anproto-reviewer` | Review implementations for security and best practices |

### Hooks (Documented)

| Hook | Event | Purpose |
|------|-------|---------|
| `validate-anproto-input` | PreToolUse | Validate formats, warn about security issues |

*Note: Hook is documented in `hooks/` but not currently registered in plugin.json.*

## Quick Start

### By Role

**New to ANProto:**
```
What is ANProto and how do I get started?
```

**Setting Up a Project:**
```
/anproto-init
```

**Debugging:**
```
/anproto-parse BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=...
```

**Security Review:**
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
├── skills/                  # 7 skills
│   ├── anproto-marketplace.md  # Discovery skill
│   ├── anproto-explain.md
│   ├── anproto-setup.md
│   ├── anproto-debug.md
│   ├── anproto-keygen.md
│   ├── anproto-sign.md
│   └── anproto-verify.md
├── commands/                # 2 commands
│   ├── anproto-init.md
│   └── anproto-parse.md
├── agents/                  # 2 agents
│   ├── anproto-explorer.md
│   └── anproto-reviewer.md
├── hooks/                   # Event hooks
│   └── validate-anproto-input.md
├── config/                  # Configuration examples
│   ├── settings.example.json
│   └── .env.example
├── templates/               # Project starter templates
│   ├── browser/
│   ├── deno/
│   └── node/
├── CHANGELOG.md             # Version history
├── CLAUDE.md                # Claude Code guidance
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE                  # MIT license
├── SECURITY.md              # Security policy
├── RESEARCH.md              # Deep research notes
└── PLAN.md                  # Implementation plan
```

## Configuration

Copy `config/.env.example` to `.env` and configure:

```bash
# Your ANProto keypair (132 characters)
ANPROTO_KEYPAIR=your-keypair-here

# Runtime environment
ANPROTO_RUNTIME=node

# Profile
ANPROTO_PROFILE=development
```

See `config/settings.example.json` for advanced configuration options.

## Language Support

| Language | Support Level | Repository |
|----------|---------------|------------|
| JavaScript | Full | [evbogue/anproto](https://github.com/evbogue/anproto) |
| Go | Guidance | [vic/goan](https://github.com/vic/goan) |
| Rust | Guidance | [vic/anproto-rs](https://github.com/vic/anproto-rs) |
| Python | Guidance | [macauleyjustin/ANproto-Python](https://github.com/macauleyjustin/ANproto-Python) |

## Security

See [SECURITY.md](SECURITY.md) for:
- Key management best practices
- Common vulnerabilities and fixes
- Complete verification checklists
- Cryptographic considerations

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to add new skills, commands, and agents
- Code style guidelines
- Testing procedures
- Pull request guidelines

## Resources

- **ANProto Demo**: https://anproto.com/try
- **Wiredove Client**: https://wiredove.net
- **Main Repository**: https://github.com/evbogue/anproto

## License

MIT
