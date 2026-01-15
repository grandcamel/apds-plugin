# Changelog

All notable changes to the ANProto Claude Code Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-01-15

### Added

- **Marketplace skill** (`anproto-marketplace.md`) - Discovery catalog for browsing all plugin capabilities
- **Validation hook** (`validate-anproto-input.md`) - PreToolUse hook for security warnings (documented, not registered)
- **Configuration examples** - `config/settings.example.json` with dev/prod/test profiles
- **Environment template** - `config/.env.example` with documented variables
- **CONTRIBUTING.md** - Contribution guidelines with component templates
- **SECURITY.md** - Security policy with best practices and vulnerability patterns
- **LICENSE** - MIT license file

### Changed

- Enhanced all skills with rich frontmatter (version, author, license)
- Enhanced all commands with category and keywords metadata
- Enhanced all agents with examples array for better trigger matching
- Updated `plugin.json` with homepage, license, and repository fields
- Updated `marketplace.json` with full metadata (keywords, category, license)
- Improved README.md with comprehensive tables and quick start guides

### Fixed

- Corrected hook frontmatter format (tools instead of match_tools)

## [1.0.0] - 2025-01-15

### Added

- Initial plugin release
- **Core skills** (6 total):
  - `anproto-explain` - Protocol concepts and architecture
  - `anproto-setup` - Project initialization guidance
  - `anproto-debug` - Message parsing and troubleshooting
  - `anproto-keygen` - Key generation and storage
  - `anproto-sign` - Signing workflow guidance
  - `anproto-verify` - Verification workflow guidance
- **Commands** (2 total):
  - `/anproto-init` - Interactive project setup wizard
  - `/anproto-parse` - Parse and explain message structure
- **Agents** (2 total):
  - `anproto-explorer` - Analyze ANProto usage in codebases
  - `anproto-reviewer` - Security and best practices review
- **Documentation**:
  - README.md - Plugin overview and usage
  - CLAUDE.md - Claude Code guidance
  - RESEARCH.md - Deep protocol analysis
  - PLAN.md - Implementation tracking
- **Marketplace**:
  - `.claude-plugin/marketplace.json` - Marketplace definition

### Technical Details

- Supports JavaScript (Deno, Node.js, Browser), Go, Rust, Python
- Based on ANProto protocol from https://github.com/evbogue/anproto
- Uses ed25519 for signing, SHA-256 for hashing, Base64 for encoding

## [Unreleased]

### Planned

- Re-enable hooks registration when Claude Code hook validation stabilizes
- Add integration tests for skill/command triggers
- Add demo screenshots to README
- Consider encryption skill for content privacy

[1.1.0]: https://github.com/grandcamel/apds-plugin/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/grandcamel/apds-plugin/releases/tag/v1.0.0
[Unreleased]: https://github.com/grandcamel/apds-plugin/compare/v1.1.0...HEAD
