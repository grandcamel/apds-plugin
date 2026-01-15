# Contributing to ANProto Plugin

Thank you for your interest in contributing to the ANProto Claude Code plugin!

## Quick Start

1. Clone the repository
2. Install the plugin locally:
   ```bash
   claude plugin marketplace add /path/to/anproto-plugin
   claude plugin install anproto-plugin
   ```
3. Make your changes
4. Test by starting a new Claude Code session

## Project Structure

```
anproto-plugin/
├── plugin.json              # Plugin manifest
├── .claude-plugin/
│   └── marketplace.json     # Marketplace definition
├── skills/                  # 7 skills (guidance + discovery)
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
│   └── validate-anproto-input.md
└── config/                  # Configuration examples
    ├── settings.example.json
    └── .env.example
```

## Adding a New Skill

### 1. Create the Skill File

Create `skills/anproto-yourskill.md`:

```yaml
---
name: anproto-yourskill
description: Use this skill when the user asks to "...", "...", or needs help with ...
version: "1.1.0"
author: "your-name"
license: "MIT"
allowed-tools:  # Optional: restrict available tools
  - Bash
  - Read
---

# Skill Title

Brief description of what this skill does.

## Quick Reference
[Summary table or key points]

## Detailed Guidance
[Step-by-step instructions]

## Examples
[Working examples]

## Troubleshooting
[Common issues and solutions]
```

### 2. Register in plugin.json

Add the skill path to `skills` array in `plugin.json`:

```json
"skills": [
  "./skills/anproto-marketplace.md",
  "./skills/anproto-yourskill.md",
  ...
]
```

### 3. Update Marketplace Skill

Add entry to `skills/anproto-marketplace.md` table.

## Adding a New Command

### 1. Create the Command File

Create `commands/anproto-yourcommand.md`:

```yaml
---
name: anproto-yourcommand
description: Brief description of what the command does
version: "1.1.0"
category: "setup|debug|utility"
keywords: ["keyword1", "keyword2"]
---

# Command Title

Opening paragraph explaining the command.

## Step 1: [First Action]
[Instructions and code]

## Step 2: [Next Action]
[Instructions and code]

## Completion Message
[What to tell the user when done]
```

### 2. Register in plugin.json

Add to `commands` array in `plugin.json`.

## Adding a New Agent

### 1. Create the Agent File

Create `agents/anproto-youragent.md`:

```yaml
---
name: anproto-youragent
description: Description including trigger phrases
version: "1.1.0"
tools:
  - Glob
  - Grep
  - Read
---

# Agent Title

## Purpose
[What the agent does]

## What to Look For
[Patterns and targets]

## Output Format
[Expected report structure]
```

### 2. Register in plugin.json

Add to `agents` array in `plugin.json`.

## Code Style

### Skill Descriptions

- Start with "Use this skill when..."
- Include specific trigger phrases in quotes
- List variations (e.g., "setup anproto", "init anproto", "add anproto")

### Frontmatter Fields

Required:
- `name`: Skill/command/agent identifier
- `description`: Trigger description

Recommended:
- `version`: Semantic version
- `author`: Author name
- `license`: License type
- `allowed-tools`: Tool whitelist (skills only)
- `tools`: Required tools (agents only)

### Documentation Structure

Use 3-level progressive disclosure:

1. **Quick Reference** - Summary table, key points
2. **Detailed Guidance** - Step-by-step instructions
3. **Deep Dive** - Advanced topics, edge cases

### Code Examples

- Use JavaScript as primary language
- Include Deno, Node.js, and Browser variants where applicable
- Always show expected output in comments

## Testing

### Manual Testing

1. Install the plugin:
   ```bash
   claude plugin update anproto-plugin
   ```

2. Start a new Claude Code session

3. Test trigger phrases:
   ```
   What is ANProto?           → Should trigger anproto-explain
   /anproto-init              → Should start setup wizard
   /anproto-parse BSY7/er4... → Should parse the message
   ```

### Validation

Run plugin validation:
```bash
claude plugin validate /path/to/anproto-plugin
```

## Pull Request Guidelines

1. **One feature per PR** - Keep changes focused
2. **Update documentation** - Skills, marketplace, README
3. **Test locally** - Verify in a new Claude session
4. **Clear commit messages** - Follow conventional commits

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

Examples:
```
feat(skills): add anproto-encrypt skill for message encryption
fix(commands): correct runtime detection in anproto-init
docs(readme): add troubleshooting section
```

## Questions?

- Open an issue for bugs or feature requests
- Check existing skills for patterns and examples
- Review SECURITY.md for security-related contributions
