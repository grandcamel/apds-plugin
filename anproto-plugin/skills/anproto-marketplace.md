---
name: anproto-marketplace
description: Browse and discover ANProto plugin capabilities. Use when the user asks "what can anproto plugin do", "list anproto skills", "anproto help", "browse anproto features", or wants to understand available ANProto automation capabilities.
version: "1.1.0"
author: "grandcamel"
license: "MIT"
---

# ANProto Plugin Marketplace

Complete catalog of ANProto developer tools and automation capabilities.

## Quick Reference

| Component | Count | Purpose |
|-----------|-------|---------|
| Skills | 6 | Protocol guidance and workflows |
| Commands | 2 | Interactive slash commands |
| Agents | 2 | Codebase analysis and review |

## Available Skills

### Guidance Skills

| Skill | Triggers | Purpose |
|-------|----------|---------|
| **anproto-explain** | "what is anproto", "explain anproto", "anproto vs nostr" | Protocol concepts, architecture, comparisons |
| **anproto-setup** | "setup anproto", "add anproto", "install anproto" | Project initialization for Deno/Node/Browser |
| **anproto-debug** | "debug message", "parse anproto", "decode keypair" | Message parsing and troubleshooting |

### Workflow Skills

| Skill | Triggers | Purpose |
|-------|----------|---------|
| **anproto-keygen** | "generate keypair", "create keys", "new identity" | Key generation and secure storage |
| **anproto-sign** | "sign message", "create signature" | Hash-then-sign workflow |
| **anproto-verify** | "verify message", "check signature", "open message" | Signature verification workflow |

## Available Commands

| Command | Usage | Purpose |
|---------|-------|---------|
| `/anproto-init` | `/anproto-init` | Interactive project setup wizard |
| `/anproto-parse` | `/anproto-parse <message>` | Parse and explain message structure |

## Available Agents

| Agent | Purpose | Tools |
|-------|---------|-------|
| **anproto-explorer** | Analyze ANProto usage in codebases | Glob, Grep, Read |
| **anproto-reviewer** | Security and best practices review | Glob, Grep, Read |

## Quick Start by Role

### Developer - New to ANProto
```
"What is ANProto and how do I get started?"
→ Triggers anproto-explain skill

/anproto-init
→ Sets up ANProto in your project
```

### Developer - Implementing Features
```
"Help me sign a message with ANProto"
→ Triggers anproto-sign skill

"How do I verify an ANProto signature?"
→ Triggers anproto-verify skill
```

### Developer - Debugging
```
/anproto-parse BSY7/er4VJIu08o39NaRAiPY/MAvd7oQhlGCRDABjYU=...
→ Parses and explains message components

"Why is my signature verification failing?"
→ Triggers anproto-debug skill
```

### Security Reviewer
```
"Review my ANProto implementation for security issues"
→ Invokes anproto-reviewer agent

"Check this codebase for exposed keys"
→ Invokes anproto-reviewer agent
```

### Code Auditor
```
"How is ANProto used in this project?"
→ Invokes anproto-explorer agent

"Find all ANProto signing operations"
→ Invokes anproto-explorer agent
```

## Message Format Reference

| Type | Length | Structure |
|------|--------|-----------|
| Keypair | 132 chars | `[publicKey:44][secretKey:88]` |
| Hash | 44 chars | Base64 SHA-256 |
| Signed | Variable | `[publicKey:44][signedPayload]` |
| Opened | 57 chars | `[timestamp:13][hash:44]` |

## Language Support

| Language | Support Level | Repository |
|----------|---------------|------------|
| JavaScript | Full | [evbogue/anproto](https://github.com/evbogue/anproto) |
| Go | Guidance | [vic/goan](https://github.com/vic/goan) |
| Rust | Guidance | [vic/anproto-rs](https://github.com/vic/anproto-rs) |
| Python | Guidance | [macauleyjustin/ANproto-Python](https://github.com/macauleyjustin/ANproto-Python) |

## Resources

- **Protocol Demo**: https://anproto.com/try
- **Wiredove Client**: https://wiredove.net
- **Source Code**: https://github.com/evbogue/anproto
