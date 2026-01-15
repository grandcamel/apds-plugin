---
description: Run ESLint on PearPass codebase to check or fix code style issues
invocable: true
arguments:
  - name: mode
    description: "Lint mode: check or fix (default: check)"
    required: false
  - name: path
    description: "Specific path to lint (default: ./src)"
    required: false
---

# PearPass Lint

Run ESLint to check or auto-fix code style issues.

## Usage

```
/pearpass-lint [mode] [path]
```

**Arguments:**
- `mode`: `check` (default) or `fix`
- `path`: Specific file or directory (default: `./src`)

## Instructions

### Check for Issues (No Changes)

```bash
cd /path/to/pearpass-app-mobile
npm run lint
```

### Auto-Fix Issues

```bash
npm run lint:fix
```

### Lint Specific Path

```bash
# Single file
npx eslint src/hooks/useCopyToClipboard.js

# Directory
npx eslint src/components/

# With auto-fix
npx eslint --fix src/hooks/
```

## ESLint Configuration

PearPass uses `eslint-config-expo` (defined in package.json):

```json
{
  "devDependencies": {
    "eslint-config-expo": "8.0.1"
  }
}
```

## Common Lint Issues

### Unused Variables

```javascript
// ❌ Error: 'unused' is defined but never used
const unused = 'value'

// ✅ Fix: Remove or prefix with underscore
const _unused = 'value'  // Intentionally unused
```

### Missing Dependencies in useEffect

```javascript
// ❌ Warning: React Hook useEffect has missing dependency
useEffect(() => {
  doSomething(value)
}, [])

// ✅ Fix: Add dependency or use callback
useEffect(() => {
  doSomething(value)
}, [value])
```

### Import Order

```javascript
// ❌ Imports not properly ordered
import { useState } from 'react'
import axios from 'axios'
import { MyComponent } from './MyComponent'
import React from 'react'

// ✅ Correct order: external -> internal -> relative
import React, { useState } from 'react'
import axios from 'axios'
import { MyComponent } from './MyComponent'
```

### Console Statements

```javascript
// ⚠️ Warning: Unexpected console statement
console.log('debug')

// ✅ Remove for production or use proper logging
// logger.debug('debug')
```

## Pre-Commit Hook

PearPass uses Husky for pre-commit hooks:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

The hook runs lint checks before commits. To bypass (not recommended):

```bash
git commit --no-verify -m "message"
```

## Integrating with Editor

### VS Code

Install ESLint extension and add to `.vscode/settings.json`:

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact"
  ]
}
```

### WebStorm/IntelliJ

1. Preferences → Languages & Frameworks → JavaScript → Code Quality Tools → ESLint
2. Enable "Automatic ESLint configuration"

## Ignoring Rules

### File-Level Ignore

```javascript
/* eslint-disable no-console */
console.log('This is allowed')
/* eslint-enable no-console */
```

### Line-Level Ignore

```javascript
console.log('Allowed') // eslint-disable-line no-console
```

### Next-Line Ignore

```javascript
// eslint-disable-next-line no-console
console.log('Allowed')
```

## Lint Output Analysis

When reporting lint issues:

1. **Group by rule** - Same fixes often apply
2. **Prioritize errors over warnings** - Errors block builds
3. **Suggest auto-fix for fixable issues** - Most style issues are auto-fixable
4. **Note security-related rules** - These are high priority

Example output format:
```
src/hooks/useAuth.js
  12:5  error    Unexpected console statement  no-console
  24:1  warning  Missing dependency 'user'     react-hooks/exhaustive-deps

✖ 2 problems (1 error, 1 warning)
  1 error and 0 warnings potentially fixable with --fix
```
