# PearPass Plugin for Claude Code

A Claude Code plugin to support developers working on [PearPass Desktop](https://github.com/tetherto/pearpass-app-desktop) - a distributed password manager built with React Native, Expo, and end-to-end encryption.

## Installation

```bash
# Add to your Claude Code plugins
claude plugins add /path/to/pearpass-plugin
```

## Overview

This plugin provides comprehensive development support for the PearPass password manager, including:

- **Skills** for development workflows, security review, architecture understanding, and testing
- **Agents** for specialized tasks like security auditing, code exploration, and test generation
- **Commands** for common development operations
- **Hooks** for security warnings and quality reminders

## Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| React Native | 0.79.5 | Mobile framework |
| Expo | ~53.0.17 | Development platform |
| React | 19.0.0 | UI library |
| styled-components | 6.1.19 | CSS-in-JS styling |
| Lingui | 5.3.3 | Internationalization |
| sodium-native | 5.0.9 | libsodium crypto bindings |
| Jest | 29.7.0 | Unit testing |
| Maestro | - | E2E testing |

## Skills

Invoke skills directly in conversation for contextual guidance.

| Skill | Command | Description |
|-------|---------|-------------|
| Development | `/pearpass-dev` | Development workflow, setup, debugging |
| Security | `/pearpass-security` | Security review, encryption patterns, audit checklist |
| Architecture | `/pearpass-architecture` | Codebase structure, provider hierarchy, navigation |
| Testing | `/pearpass-test` | Jest patterns, Maestro flows, mock strategies |

### Example Usage

```
/pearpass-security
```
Returns security review guidelines, encryption stack details, and audit checklist.

## Agents

Specialized agents for complex tasks. Agents are triggered automatically or can be invoked explicitly.

| Agent | Purpose | Trigger |
|-------|---------|---------|
| **security-reviewer** | Audit encryption, credential handling, IPC security | Changes to vault-core, security hooks |
| **code-explorer** | Navigate architecture, trace data flow, find patterns | Codebase exploration questions |
| **component-dev** | Create React components with styling, i18n, context | UI component development |
| **test-generator** | Generate Jest tests, Maestro flows, mock patterns | Test creation requests |

### Example Usage

```
Use the security-reviewer agent to audit the clipboard hook
```

## Commands

Quick commands for common development tasks.

| Command | Description |
|---------|-------------|
| `/pearpass-dev` | Start development server, first-time setup |
| `/pearpass-build [platform]` | Build for iOS, Android, or all |
| `/pearpass-test [type]` | Run Jest tests, Maestro E2E, coverage |
| `/pearpass-lint [mode]` | ESLint check or auto-fix |
| `/pearpass-i18n [action]` | Extract, compile, or add translations |
| `/pearpass-security-audit [scope]` | Run security checks on codebase |

### Command Examples

```bash
# Start development
/pearpass-dev

# Build for iOS
/pearpass-build ios

# Run tests with coverage
/pearpass-test coverage

# Fix lint issues
/pearpass-lint fix

# Extract translations
/pearpass-i18n extract

# Full security audit
/pearpass-security-audit full
```

## Hooks

Automatic warnings and reminders triggered during development.

| Hook | Event | Trigger |
|------|-------|---------|
| **security-crypto-warning** | PreToolUse | Editing crypto/vault-core files |
| **security-secrets-check** | PreToolUse | Git commit/add commands |
| **quality-i18n-reminder** | PostToolUse | Writing UI components |
| **quality-test-reminder** | PostToolUse | Creating components/hooks/contexts |

### Hook Behavior

- **Security Crypto Warning**: Displays encryption checklist when modifying vault-core or crypto files
- **Security Secrets Check**: Warns about potential secrets before git commits
- **i18n Reminder**: Reminds to use Lingui for user-facing strings
- **Test Reminder**: Suggests test patterns for new components

## Project Structure

```
pearpass-plugin/
├── plugin.json           # Plugin manifest
├── README.md             # This file
├── RESEARCH.md           # Codebase analysis findings
├── PLAN.md               # Implementation roadmap
├── skills/
│   ├── pearpass-dev.md
│   ├── pearpass-security.md
│   ├── pearpass-architecture.md
│   └── pearpass-test.md
├── agents/
│   ├── security-reviewer.md
│   ├── code-explorer.md
│   ├── component-dev.md
│   └── test-generator.md
├── commands/
│   ├── pearpass-dev.md
│   ├── pearpass-build.md
│   ├── pearpass-test.md
│   ├── pearpass-lint.md
│   ├── pearpass-i18n.md
│   └── pearpass-security-audit.md
└── hooks/
    ├── security-crypto-warning.md
    ├── security-secrets-check.md
    ├── quality-i18n-reminder.md
    └── quality-test-reminder.md
```

## PearPass Architecture

### Security Layers

1. **Master Password** → Argon2id → Derived key
2. **Vault Key** → XSalsa20-Poly1305 → Encrypted credentials
3. **Worklet Isolation** → Crypto in separate thread
4. **Clipboard** → Auto-clear after timeout
5. **Inactivity** → Auto-lock after 60 seconds

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `src/components/` | UI components (41 directories) |
| `src/hooks/` | Custom React hooks (30 files) |
| `src/context/` | React Context providers |
| `src/services/` | Service layer (IPC, clipboard) |
| `packages/` | Git submodules (17 packages) |
| `packages/pearpass-lib-vault-core/` | Encryption worklet |

### Provider Hierarchy

```
LoadingProvider
└── ThemeProvider
    └── VaultProvider
        └── I18nProvider (Lingui)
            └── ToastProvider
                └── RouterProvider
                    └── ModalProvider
                        └── App
```

## Security Guidelines

When working with PearPass:

### DO ✅
- Use `sodium_malloc` for sensitive buffers
- Use `randombytes_buf` for nonce generation
- Use `OPSLIMIT_SENSITIVE` for password hashing
- Check authentication before using decrypted data
- Clear sensitive buffers after use

### DON'T ❌
- Log passwords, keys, or credentials
- Use `Buffer.alloc` for encryption keys
- Use predictable or sequential nonces
- Expose detailed error messages on auth failure
- Commit `.env` files or secrets

## Development Workflow

### First-Time Setup

```bash
cd pearpass-app-desktop
git submodule update --init --recursive
npm install
npm run bundle-bare
npm run lingui:compile
```

### Daily Development

```bash
npm start              # Expo dev server
npm run ios            # Run on iOS
npm run android        # Run on Android
npm test               # Run Jest tests
npm run lint           # Check code style
```

### Before Committing

```bash
npm run lint:fix       # Fix style issues
npm test               # Verify tests pass
npm run lingui:extract # Update translations
```

## Contributing

Contributions are welcome! When modifying this plugin:

1. Test skills with real PearPass development scenarios
2. Verify agents produce accurate guidance
3. Ensure hooks trigger appropriately
4. Update documentation for any changes

## License

This plugin is provided for use with PearPass development. PearPass is licensed under Apache 2.0.
