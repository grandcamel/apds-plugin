# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin (`apds-dev`) that helps developers build applications with APDS (ANProto Personal Data Server) - a Deno-based decentralized personal data server using Ed25519 signatures and P2P gossip sync.

## Plugin Architecture

Claude Code plugins use markdown files with YAML frontmatter to define components:

- **Skills** (`skills/*.md`) - Guided workflows with `description` and `user_invocable` frontmatter
- **Agents** (`agents/*.md`) - Autonomous helpers with `description`, `tools`, and `model` frontmatter
- **Commands** (`commands/*.md`) - Quick actions with `description`, `arguments`, and `user_invocable` frontmatter
- **Hooks** (`hooks/*.md`) - Event triggers with `event`, `match_tools`, and `match_commands` frontmatter

The `plugin.json` manifest uses glob patterns to auto-discover components.

## Component Conventions

### Skills
- Use third-person descriptions for triggering (e.g., "Help developers set up APDS")
- Body written in imperative form directed at Claude
- Include code examples and troubleshooting sections

### Agents
- Specify minimal required `tools` array
- Use `model: sonnet` for most agents
- Include systematic diagnostic approaches

### Commands
- Define `arguments` with `name`, `description`, `required`, and `default`
- Document usage patterns and expected outputs

## APDS Domain Knowledge

When working on this plugin, understand that APDS uses:
- 44-character base64 public keys as identity
- 88-character keypairs (pubkey + privkey)
- YAML frontmatter for message metadata (name, image, previous)
- Content-addressed storage (hash → data)
- WebSocket gossip for P2P sync
- Deno runtime with `-A` permissions flag

Key APDS files: `apds.js` (core), `serve.js` (server:9000), `cli.js` (terminal), `gossip.js` (sync).

## Research Reference

Detailed APDS architecture documentation is in `.research/apds-overview.md`.
