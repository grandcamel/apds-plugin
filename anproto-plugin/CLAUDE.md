# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is a Claude Code plugin called "anproto-plugin" that helps developers work with the ANProto protocol - a lightweight decentralized authentication and messaging system using ed25519 keypairs.

## Repository Structure

```
anproto-plugin/
├── plugin.json              # Plugin manifest (v1.1.0)
├── .claude-plugin/
│   └── marketplace.json     # Marketplace definition with full metadata
├── skills/                  # 7 skills for protocol guidance
│   ├── anproto-marketplace.md  # Discovery/navigation skill
│   ├── anproto-explain.md      # Protocol concepts
│   ├── anproto-setup.md        # Project setup
│   ├── anproto-debug.md        # Debugging
│   ├── anproto-keygen.md       # Key generation
│   ├── anproto-sign.md         # Signing workflow
│   └── anproto-verify.md       # Verification workflow
├── commands/                # 2 slash commands
│   ├── anproto-init.md
│   └── anproto-parse.md
├── agents/                  # 2 specialized agents
│   ├── anproto-explorer.md
│   └── anproto-reviewer.md
├── hooks/                   # Event hooks
│   └── validate-anproto-input.md  # PreToolUse validation
├── config/                  # Configuration examples
│   ├── settings.example.json
│   └── .env.example
├── anproto/                 # Cloned ANProto source (reference only, gitignored)
├── CONTRIBUTING.md          # Contribution guidelines
├── SECURITY.md              # Security policy
├── RESEARCH.md              # Deep technical analysis
└── PLAN.md                  # Implementation plan
```

## Plugin Architecture

### Skills
Skills are context-loading documents with enhanced frontmatter:
- `name`: Skill identifier
- `description`: Trigger phrases and purpose
- `version`: Semantic version (1.1.0)
- `author`: Author name
- `license`: MIT
- `allowed-tools`: Optional tool whitelist

### Commands
Slash commands with metadata:
- `name`: Command identifier
- `description`: Brief description
- `version`: Semantic version
- `category`: setup, debug, utility
- `keywords`: Search terms

### Agents
Specialized subagents with examples:
- `name`: Agent identifier
- `description`: Purpose and triggers
- `version`: Semantic version
- `category`: analysis, security
- `tools`: Required tools
- `examples`: Example trigger phrases

### Hooks
Event-driven validation:
- `validate-anproto-input`: PreToolUse hook for format validation and security warnings

## ANProto Quick Reference

### Core API
```javascript
an.gen()           // Generate 132-char keypair
an.hash(content)   // SHA-256 hash (44 chars)
an.sign(hash, key) // Sign with timestamp
an.open(signed)    // Verify and extract (57 chars)
```

### Message Formats
| Type | Length | Structure |
|------|--------|-----------|
| Keypair | 132 | `[pubkey:44][secret:88]` |
| Hash | 44 | Base64 SHA-256 |
| Signed | Variable | `[pubkey:44][payload]` |
| Opened | 57 | `[timestamp:13][hash:44]` |

## Installation Commands

```bash
# Add marketplace
claude plugin marketplace add /path/to/anproto-plugin

# Install plugin
claude plugin install anproto-plugin

# Validate
claude plugin validate /path/to/anproto-plugin
```

## Key Improvements (v1.1.0)

1. **Enhanced Marketplace**: Full metadata with keywords, category, license
2. **Discovery Skill**: `anproto-marketplace` for browsing capabilities
3. **Validation Hook**: PreToolUse hook for security warnings
4. **Configuration**: Example settings and environment files
5. **Documentation**: CONTRIBUTING.md, SECURITY.md
6. **Rich Frontmatter**: All components have version, author, license

## Development Notes

- The `anproto/` folder contains the cloned ANProto source for reference (gitignored)
- RESEARCH.md contains detailed protocol analysis
- PLAN.md tracks implementation status
- All skills use 3-level progressive disclosure where applicable
- Agents include example trigger phrases for better matching
