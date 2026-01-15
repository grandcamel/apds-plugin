# APDS Plugin Development Plan

## Vision

Create a Claude Code plugin that accelerates development with APDS (ANProto Personal Data Server) by providing specialized skills, agents, and commands for the decentralized social protocol ecosystem.

## Completed

### Phase 1: Research & Foundation ✓

- [x] Clone and analyze APDS codebase
- [x] Document architecture and APIs
- [x] Identify developer workflows and pain points
- [x] Create plugin structure

### Phase 2: Initial Implementation ✓

- [x] `apds-setup` skill - Guided setup for all deployment modes
- [x] `apds-debug` skill - Troubleshooting common issues
- [x] `apds-explorer` agent - Codebase exploration
- [x] `/apds-run` command - Quick start with correct permissions

### Phase 3: Enhanced Skills ✓

- [x] `apds-message` skill - Message composition and YAML formatting
- [x] `apds-connect` skill - Peer discovery and management
- [x] `apds-push` skill - Web Push notification setup

### Phase 4: Additional Agents ✓

- [x] `apds-troubleshooter` agent - Automated issue diagnosis
- [x] `apds-architect` agent - Help design APDS-based applications

### Phase 5: Commands & Hooks ✓

- [x] `/apds-init` command - Scaffold new APDS project
- [x] `/apds-test` command - Test push notifications
- [x] `/apds-peers` command - Manage peer connections
- [x] Pre-commit hook for message format validation

## In Progress

### Phase 6: Integration & Polish

- [ ] Integration testing with live APDS instances
- [ ] User feedback and iteration
- [ ] Performance optimization
- [ ] Extended documentation

## Architecture Decisions

### Why These Components?

1. **Skills over commands** for guided workflows - setup and debug require context and interaction
2. **Explorer agent** for deep codebase understanding - APDS has non-obvious message flow patterns
3. **Simple commands** for quick actions - running servers doesn't need conversation

### Key Insights from Research

1. **Deno-first** - All tooling must understand Deno permissions and module system
2. **Namespace isolation** - Storage namespaces are a common confusion point
3. **P2P complexity** - Gossip and sync behavior needs clear explanation
4. **YAML frontmatter** - Message format is specific and easy to get wrong

## Resources

- APDS Repository: https://github.com/evbogue/apds
- Live Instance: https://apds.anproto.com
- Research Notes: `.research/apds-overview.md`

## Changelog

### v0.4.0 (2026-01-15)
- New commands: /apds-init, /apds-test, /apds-peers
- Pre-commit hook for config and code validation
- Project scaffolding with multiple templates
- Comprehensive peer management tooling

### v0.3.0 (2026-01-15)
- New agents: apds-troubleshooter, apds-architect
- Systematic diagnostic workflows for common issues
- Application design patterns and architecture guidance

### v0.2.0 (2026-01-15)
- Enhanced skills: apds-message, apds-connect, apds-push
- Complete message composition and YAML formatting guide
- Peer connection management and gossip protocol documentation
- Web Push notification setup with service worker examples

### v0.1.0 (2026-01-15)
- Initial plugin structure
- Core skills: apds-setup, apds-debug
- Explorer agent
- Run command
- Comprehensive research documentation
