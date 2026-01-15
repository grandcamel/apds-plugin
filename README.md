# APDS Developer Plugin for Claude Code

A Claude Code plugin to support developers building with [APDS](https://github.com/evbogue/apds) (ANProto Personal Data Server).

## About APDS

APDS is a Deno-based personal data server for ANProto, a decentralized social protocol. It provides:

- Ed25519 signed messages with content-addressed storage
- Real-time WebSocket sync between peers
- Gossip-based P2P message propagation
- Web Push notification support
- CLI and browser client interfaces

## Plugin Components

### Skills

| Skill | Description |
|-------|-------------|
| `apds-setup` | Guided setup for server, CLI, relay, or browser client |
| `apds-debug` | Troubleshoot connection, sync, keypair, and storage issues |
| `apds-message` | Compose messages with YAML frontmatter and understand message format |
| `apds-connect` | Manage peer connections, WebSocket sync, and gossip protocol |
| `apds-push` | Set up Web Push notifications with VAPID and service workers |

### Agents

| Agent | Description |
|-------|-------------|
| `apds-explorer` | Explore APDS codebases, trace message flows, understand architecture |
| `apds-troubleshooter` | Systematically diagnose issues with servers, connections, and sync |
| `apds-architect` | Design APDS applications - data models, topologies, integration patterns |

### Commands

| Command | Description |
|---------|-------------|
| `/apds-run` | Start APDS server, CLI, or relay with correct Deno permissions |
| `/apds-init` | Scaffold new APDS project (server, client, relay, or fullstack) |
| `/apds-test` | Test APDS functionality (push, server, signing, sync) |
| `/apds-peers` | Manage peer connections (list, add, remove, test, discover) |

### Hooks

| Hook | Description |
|------|-------------|
| `validate-apds-config` | Pre-commit validation for config files, sensitive data, and code quality |

## Installation

```bash
# From your Claude Code settings, add the plugin
claude plugins add /path/to/apds-plugin
```

## Usage Examples

### Setting Up APDS
```
> /apds-setup
# Follow the guided setup for your use case
```

### Debugging Issues
```
> /apds-debug
# Describe your issue for systematic diagnosis
```

### Running APDS
```
> /apds-run server   # Start server on port 9000
> /apds-run cli      # Start interactive CLI
> /apds-run relay    # Start WebSocket relay
```

### Exploring Code
The `apds-explorer` agent is automatically triggered when exploring APDS codebases.

## Project Structure

```
apds-plugin/
├── plugin.json           # Plugin manifest
├── agents/
│   └── apds-explorer.md  # Codebase exploration agent
├── skills/
│   ├── apds-setup.md     # Setup guide skill
│   └── apds-debug.md     # Debugging skill
├── commands/
│   └── apds-run.md       # Run command
├── hooks/                # (future)
├── .research/
│   ├── apds-overview.md  # Architecture documentation
│   └── apds-repo/        # Cloned APDS source (gitignored)
├── PLAN.md               # Development roadmap
└── README.md             # This file
```

## APDS Quick Reference

### Key Files
- `apds.js` - Core library (storage, signing, queries)
- `serve.js` - HTTP/WebSocket server (port 9000)
- `cli.js` - Terminal client
- `gossip.js` - P2P sync protocol
- `relay.js` - WebSocket relay

### Core API
```javascript
await apds.start('namespace')     // Initialize
await apds.compose('message')     // Create signed message
await apds.query()                // Get all messages
await apds.get(hash)              // Get by hash
await apds.pubkey()               // Get public key
```

### CLI Commands
```
/nick <name>     - Set display name
/connect <url>   - Connect to peer
/clear           - Erase local data
/exit            - Exit CLI
```

### Server Endpoints
```
GET  /all         - All messages
GET  /latest      - Last 5 minutes
GET  /<hash>      - By hash
WS   /            - Real-time sync
```

## Development Status

**Current Version**: 0.4.0

See [PLAN.md](./PLAN.md) for the full development roadmap.

## License

MIT
