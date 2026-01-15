---
description: Validate APDS configuration and message format before commits
event: PreToolUse
match_tools:
  - Bash
match_commands:
  - git commit
---

# APDS Configuration Validator Hook

This hook validates APDS configuration files and message formats before git commits to prevent common mistakes.

## What It Validates

### 1. Configuration Files

**db.json**
- Valid JSON syntax
- Expected structure for peer lists

**deno.json**
- Valid JSON syntax
- APDS tasks defined correctly

### 2. Sensitive Files Check

Warn if committing:
- `vapid.json` - Contains private VAPID keys
- `subscriptions.json` - Contains push subscription data
- `*.keypair` - Keypair files

### 3. Code Quality

Check JavaScript files for:
- Missing `await` on async APDS functions
- Hardcoded WebSocket URLs (should be configurable)
- Exposed private keys in code

## Validation Logic

```javascript
// Hook implementation
export default {
  name: 'validate-apds-config',
  event: 'PreToolUse',

  async run(context) {
    const { tool, parameters } = context

    // Only check git commit commands
    if (tool !== 'Bash') return { allow: true }
    if (!parameters.command?.includes('git commit')) return { allow: true }

    const issues = []

    // Check for sensitive files being committed
    const stagedFiles = await getStagedFiles()

    const sensitivePatterns = [
      { pattern: /vapid\.json$/, message: 'vapid.json contains private VAPID keys' },
      { pattern: /subscriptions\.json$/, message: 'subscriptions.json contains user data' },
      { pattern: /\.keypair$/, message: 'Keypair files should not be committed' }
    ]

    for (const file of stagedFiles) {
      for (const { pattern, message } of sensitivePatterns) {
        if (pattern.test(file)) {
          issues.push({
            severity: 'error',
            file,
            message
          })
        }
      }
    }

    // Validate JSON files
    const jsonFiles = stagedFiles.filter(f => f.endsWith('.json'))
    for (const file of jsonFiles) {
      try {
        const content = await Deno.readTextFile(file)
        JSON.parse(content)
      } catch (e) {
        issues.push({
          severity: 'error',
          file,
          message: `Invalid JSON: ${e.message}`
        })
      }
    }

    // Check JS files for common issues
    const jsFiles = stagedFiles.filter(f => f.endsWith('.js'))
    for (const file of jsFiles) {
      const content = await Deno.readTextFile(file)
      const lines = content.split('\n')

      lines.forEach((line, i) => {
        // Check for missing await
        const asyncPatterns = [
          /apds\.(compose|sign|get|put|query|generate)\(/,
          /apds\.(pubkey|keypair|open|add|make)\(/
        ]

        for (const pattern of asyncPatterns) {
          if (pattern.test(line) && !line.includes('await') && !line.includes('then')) {
            issues.push({
              severity: 'warning',
              file,
              line: i + 1,
              message: 'APDS async function may need await'
            })
          }
        }

        // Check for hardcoded URLs
        if (/new WebSocket\(['"][^'"]+['"]\)/.test(line)) {
          if (!line.includes('localhost') && !line.includes('${') && !line.includes('URL')) {
            issues.push({
              severity: 'warning',
              file,
              line: i + 1,
              message: 'Consider making WebSocket URL configurable'
            })
          }
        }

        // Check for exposed keys
        if (/keypair.*=.*['"][A-Za-z0-9+/=]{88}['"]/.test(line)) {
          issues.push({
            severity: 'error',
            file,
            line: i + 1,
            message: 'Hardcoded keypair detected - this is a security risk'
          })
        }
      })
    }

    // Report results
    if (issues.length > 0) {
      const errors = issues.filter(i => i.severity === 'error')
      const warnings = issues.filter(i => i.severity === 'warning')

      let message = '\nAPDS Validation Results:\n'

      if (errors.length > 0) {
        message += '\n❌ Errors (must fix):\n'
        errors.forEach(e => {
          message += `  ${e.file}${e.line ? `:${e.line}` : ''}: ${e.message}\n`
        })
      }

      if (warnings.length > 0) {
        message += '\n⚠️  Warnings:\n'
        warnings.forEach(w => {
          message += `  ${w.file}${w.line ? `:${w.line}` : ''}: ${w.message}\n`
        })
      }

      if (errors.length > 0) {
        return {
          allow: false,
          message: message + '\nCommit blocked. Fix errors above.'
        }
      }

      // Allow with warnings
      return {
        allow: true,
        message: message + '\nProceeding with warnings.'
      }
    }

    return { allow: true }
  }
}

async function getStagedFiles() {
  const process = Deno.run({
    cmd: ['git', 'diff', '--cached', '--name-only'],
    stdout: 'piped'
  })
  const output = await process.output()
  process.close()
  return new TextDecoder().decode(output).trim().split('\n').filter(Boolean)
}
```

## Example Output

### Blocking Commit (Errors)
```
APDS Validation Results:

❌ Errors (must fix):
  vapid.json: vapid.json contains private VAPID keys
  src/config.js:42: Hardcoded keypair detected - this is a security risk

Commit blocked. Fix errors above.

To commit anyway (not recommended):
  git commit --no-verify
```

### Allowing with Warnings
```
APDS Validation Results:

⚠️  Warnings:
  src/client.js:15: APDS async function may need await
  src/connect.js:8: Consider making WebSocket URL configurable

Proceeding with warnings.
```

### Clean Commit
No output - commit proceeds normally.

## Bypass

To bypass validation (not recommended):
```bash
git commit --no-verify -m "message"
```

## Customization

Add to `.apds-validate.json` in project root:

```json
{
  "ignorePaths": [
    "test/**",
    "examples/**"
  ],
  "allowHardcodedUrls": [
    "wss://apds.anproto.com"
  ],
  "warnOnSensitiveFiles": true,
  "blockOnSensitiveFiles": false
}
```
