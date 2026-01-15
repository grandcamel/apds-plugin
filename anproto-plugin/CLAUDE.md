# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is a Claude Code plugin called "anproto-plugin" that helps developers work with the ANProto protocol - a lightweight decentralized authentication and messaging system using ed25519 keypairs.

## Repository Structure

```
anproto-plugin/
├── plugin.json              # Plugin manifest
├── .claude-plugin/
│   └── marketplace.json     # Marketplace definition for installation
├── skills/                  # 6 skills for protocol guidance
│   ├── anproto-explain.md   # Protocol concepts
│   ├── anproto-setup.md     # Project setup
│   ├── anproto-debug.md     # Message debugging
│   ├── anproto-keygen.md    # Key generation
│   ├── anproto-sign.md      # Signing workflow
│   └── anproto-verify.md    # Verification workflow
├── commands/                # 2 slash commands
│   ├── anproto-init.md      # Initialize ANProto in a project
│   └── anproto-parse.md     # Parse message components
├── agents/                  # 2 specialized agents
│   ├── anproto-explorer.md  # Analyze ANProto usage in codebases
│   └── anproto-reviewer.md  # Review implementations for security
├── anproto/                 # Cloned ANProto source (reference only)
├── RESEARCH.md              # Deep technical analysis
└── PLAN.md                  # Implementation plan
```

## Plugin Architecture

### Skills
Skills are context-loading documents that Claude uses to assist with specific topics:
- Triggered by natural language matching the `description` field
- Provide comprehensive guidance without executing code
- Each skill has YAML frontmatter with a description

### Commands
Slash commands for specific actions:
- `/anproto-init` - Interactive project setup
- `/anproto-parse` - Parse and explain message structures

### Agents
Specialized subagents for complex tasks:
- `anproto-explorer` - Uses Glob, Grep, Read to analyze codebases
- `anproto-reviewer` - Security and best practices review

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
```

## Key Gotchas

1. **plugin.json format**: Skills can be array of paths or single path string
2. **Skill frontmatter**: Must have `description` field for trigger matching
3. **Command frontmatter**: Must have `name` and `description` fields
4. **Agent frontmatter**: Must have `name`, `description`, and `tools` fields

## Development Notes

- The `anproto/` folder contains the cloned ANProto source for reference
- RESEARCH.md contains detailed protocol analysis
- PLAN.md tracks implementation status
- All skills focus on guidance, not code execution (ANProto crypto happens in user code)
