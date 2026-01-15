---
description: Manage PearPass internationalization - extract, compile, or add translations using Lingui
invocable: true
arguments:
  - name: action
    description: "Action: extract, compile, add, or status (default: status)"
    required: false
  - name: locale
    description: "Target locale code (e.g., es, fr, de) for 'add' action"
    required: false
---

# PearPass i18n

Manage internationalization with Lingui.

## Usage

```
/pearpass-i18n [action] [locale]
```

**Arguments:**
- `action`: `extract`, `compile`, `add`, or `status` (default)
- `locale`: Locale code for adding new languages (e.g., `es`, `fr`)

## Instructions

### Check Translation Status

```bash
cd /path/to/pearpass-app-mobile
npx lingui extract --dry-run
```

### Extract Translation Strings

Scans codebase for translatable strings and updates `.po` files:

```bash
npm run lingui:extract
```

### Compile Translations

Compiles `.po` files to optimized format for runtime:

```bash
npm run lingui:compile
```

### Full i18n Workflow

```bash
npm run lingui:extract && npm run lingui:compile
```

## Translation File Locations

```
src/locales/
├── en/
│   └── messages.po    # English translations (source)
├── es/
│   └── messages.po    # Spanish translations
├── fr/
│   └── messages.po    # French translations
└── ... (other locales)
```

## Using Translations in Code

### In JSX Components

```javascript
import { Trans } from '@lingui/macro'

// Simple text
<Trans>Welcome to PearPass</Trans>

// With variables
<Trans>Hello, {name}!</Trans>

// With components
<Trans>
  Read our <Link to="/terms">Terms of Service</Link>
</Trans>
```

### In JavaScript Code

```javascript
import { t } from '@lingui/macro'

// String messages
const message = t`Enter your password`
const error = t`Invalid credentials`

// With variables
const greeting = t`Hello, ${name}`
```

### Plurals

```javascript
import { plural } from '@lingui/macro'

const message = plural(count, {
  one: '# item',
  other: '# items'
})
```

## Adding New Language

### 1. Update Lingui Config

Edit `lingui.config.js` (or create if missing):

```javascript
module.exports = {
  locales: ['en', 'es', 'fr', 'de', 'NEW_LOCALE'],
  sourceLocale: 'en',
  catalogs: [{
    path: 'src/locales/{locale}/messages',
    include: ['src']
  }],
  format: 'po'
}
```

### 2. Extract to Create New Catalog

```bash
npm run lingui:extract
```

This creates `src/locales/NEW_LOCALE/messages.po`

### 3. Translate the .po File

Edit `src/locales/NEW_LOCALE/messages.po`:

```po
#: src/components/Welcome/index.js:15
msgid "Welcome to PearPass"
msgstr "Bienvenido a PearPass"  # Add translation here
```

### 4. Compile

```bash
npm run lingui:compile
```

## Translation Guidelines

### DO:

```javascript
// ✅ Use Trans for JSX
<Trans>Save changes</Trans>

// ✅ Use t for strings
const label = t`Password`

// ✅ Keep translations simple
<Trans>Enter your master password</Trans>
```

### DON'T:

```javascript
// ❌ Don't concatenate translatable strings
<Trans>{"Hello " + name}</Trans>

// ❌ Don't translate technical terms
<Trans>API Key</Trans>  // Keep as-is

// ❌ Don't include variables without context
<Trans>{errorCode}</Trans>  // Meaningless to translators
```

## Common i18n Patterns

### Conditional Text

```javascript
import { Trans } from '@lingui/macro'

{isLocked ? (
  <Trans>Vault is locked</Trans>
) : (
  <Trans>Vault is unlocked</Trans>
)}
```

### Dynamic Content

```javascript
import { t } from '@lingui/macro'

const getErrorMessage = (code) => {
  switch (code) {
    case 'INVALID':
      return t`Invalid password`
    case 'EXPIRED':
      return t`Session expired`
    default:
      return t`Unknown error`
  }
}
```

### Dates and Numbers

```javascript
import { i18n } from '@lingui/core'

// Format date
i18n.date(new Date())

// Format number
i18n.number(1234.56)
```

## Troubleshooting

### Issue: Missing Translations at Runtime

**Symptom:** Shows message IDs instead of translations

**Fix:**
```bash
npm run lingui:compile
# Restart dev server
npm start
```

### Issue: Extract Not Finding Strings

**Symptom:** New `<Trans>` components not appearing in .po files

**Check:**
1. Using correct import: `import { Trans } from '@lingui/macro'`
2. File is in `src/` directory
3. Run extract: `npm run lingui:extract`

### Issue: Compile Errors

**Symptom:** lingui:compile fails

**Fix:**
1. Check .po file syntax
2. Ensure all msgstr have valid format
3. Run: `npx lingui extract --clean` to reset

## CI/CD Integration

The build process includes i18n:

```json
{
  "scripts": {
    "build": "... && npm run lingui:extract && npm run lingui:compile",
    "eas-build-post-install": "... && npm run lingui:extract && npm run lingui:compile"
  }
}
```

Ensure translations are up-to-date before building.
