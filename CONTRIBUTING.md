# Contributing to APDS Plugin

Thank you for your interest in contributing to the APDS Developer Plugin for Claude Code!

## Ways to Contribute

### 1. Report Issues

Found a bug or have a suggestion? Open an issue with:

- **Bug reports**: Clear description, steps to reproduce, expected vs actual behavior
- **Feature requests**: Use case, proposed solution, alternatives considered
- **Documentation**: Unclear sections, missing information, typos

### 2. Improve Documentation

Documentation improvements are always welcome:

- Fix typos or unclear explanations
- Add examples for common use cases
- Improve code samples
- Translate to other languages

### 3. Add Skills, Agents, or Commands

Want to add new functionality? Consider:

- **Skills**: Guided workflows for specific APDS tasks
- **Agents**: Autonomous helpers for complex operations
- **Commands**: Quick actions for common operations
- **Hooks**: Validation or automation triggers

### 4. Test and Provide Feedback

Help improve the plugin by:

- Testing with different APDS configurations
- Reporting edge cases
- Suggesting UX improvements
- Sharing your use cases

## Development Setup

### Prerequisites

- Claude Code CLI installed
- Deno (for testing against APDS)
- Git

### Clone and Install

```bash
git clone <repository-url>
cd apds-plugin

# Add plugin to Claude Code for testing
claude plugins add .
```

### Project Structure

```
apds-plugin/
├── plugin.json          # Plugin manifest
├── agents/              # Agent definitions
├── skills/              # Skill definitions
├── commands/            # Command definitions
├── hooks/               # Hook definitions
├── docs/                # Extended documentation
├── .research/           # Research and notes
├── PLAN.md              # Development roadmap
├── CONTRIBUTING.md      # This file
└── README.md            # User documentation
```

## Creating New Components

### Skills

Skills are guided workflows. Create `skills/your-skill.md`:

```markdown
---
description: Brief description for when to trigger this skill
user_invocable: true
---

# Skill Title

You are helping with [specific task].

## Steps

1. First step
2. Second step
...
```

### Agents

Agents are autonomous helpers. Create `agents/your-agent.md`:

```markdown
---
description: When to use this agent
tools:
  - Glob
  - Grep
  - Read
  - Bash
model: sonnet
---

# Agent Title

You are an expert at [domain].

## Capabilities

...
```

### Commands

Commands are quick actions. Create `commands/your-command.md`:

```markdown
---
description: What this command does
user_invocable: true
arguments:
  - name: arg_name
    description: Argument description
    required: false
    default: "default_value"
---

# Command Title

## Usage

/your-command [args]

## Implementation

...
```

### Hooks

Hooks trigger on events. Create `hooks/your-hook.md`:

```markdown
---
description: What this hook validates
event: PreToolUse
match_tools:
  - Bash
match_commands:
  - git commit
---

# Hook Title

## Validation Logic

...
```

## Code Style Guidelines

### Markdown

- Use ATX-style headers (`#`, `##`, `###`)
- Include code blocks with language hints
- Use tables for structured data
- Keep lines under 100 characters when practical

### Code Examples

- Use consistent indentation (2 spaces for JS)
- Include comments for non-obvious logic
- Show both success and error cases
- Use realistic, working examples

### YAML Frontmatter

- Required fields first
- Optional fields after
- Use lowercase keys
- Quote strings with special characters

## Testing Your Changes

### Manual Testing

1. Add plugin to Claude Code: `claude plugins add .`
2. Test your component in a real session
3. Verify edge cases and error handling

### Integration Testing

```bash
# Start local APDS server
deno run -A serve.js

# Run integration tests
deno run -A docs/integration-testing.md
```

## Submitting Changes

### Commit Messages

Follow conventional commits:

```
type(scope): description

- feat: New feature
- fix: Bug fix
- docs: Documentation only
- refactor: Code change that neither fixes a bug nor adds a feature
- test: Adding or updating tests
- chore: Maintenance tasks
```

Examples:
```
feat(skills): add apds-backup skill for data export
fix(agents): correct WebSocket URL detection in troubleshooter
docs: improve integration testing examples
```

### Pull Request Process

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Make your changes
4. Test thoroughly
5. Commit with clear messages
6. Push and create a pull request
7. Describe your changes and motivation

### PR Checklist

- [ ] Changes tested locally
- [ ] Documentation updated if needed
- [ ] No sensitive data in commits
- [ ] Follows existing code style
- [ ] Commit messages are clear

## Questions?

- Check existing issues for similar questions
- Open a discussion for general questions
- Tag maintainers for urgent issues

## Code of Conduct

- Be respectful and constructive
- Welcome newcomers
- Focus on the technical merits
- Assume good intentions

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for helping improve the APDS Developer Plugin!
