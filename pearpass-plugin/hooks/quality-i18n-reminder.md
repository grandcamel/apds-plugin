---
event: PostToolUse
tools:
  - Write
  - Edit
match_files:
  - "**/components/**/*.js"
  - "**/components/**/*.jsx"
  - "**/containers/**/*.js"
  - "**/containers/**/*.jsx"
  - "**/pages/**/*.js"
  - "**/pages/**/*.jsx"
---

# Internationalization Reminder

You've modified a UI component. Remember to internationalize any user-facing strings.

## Quick Check

Does this file contain any of these patterns that need i18n?

### Needs Translation ❌
```javascript
// Hardcoded user-facing strings
<Text>Welcome to PearPass</Text>
<Button title="Save" />
const message = "Enter your password"
placeholder="Search..."
```

### Properly Internationalized ✅
```javascript
import { Trans, t } from '@lingui/macro'

<Trans>Welcome to PearPass</Trans>
<Button title={t`Save`} />
const message = t`Enter your password`
placeholder={t`Search...`}
```

## Lingui Patterns

### JSX Text
```javascript
import { Trans } from '@lingui/macro'

// Simple text
<Trans>Save changes</Trans>

// With variables
<Trans>Hello, {userName}!</Trans>

// With components
<Trans>
  Read our <Link to="/terms">Terms</Link>
</Trans>
```

### JavaScript Strings
```javascript
import { t } from '@lingui/macro'

// Labels and messages
const label = t`Password`
const error = t`Invalid credentials`

// With variables
const greeting = t`Welcome back, ${name}`
```

### Plurals
```javascript
import { plural } from '@lingui/macro'

const message = plural(count, {
  one: '# item',
  other: '# items'
})
```

## What NOT to Translate

- Technical identifiers: `testID`, `accessibilityLabel` (unless user-visible)
- Log messages (internal only)
- Error codes
- API field names
- CSS class names

## After Adding Translations

Run these commands to update translation catalogs:

```bash
npm run lingui:extract
npm run lingui:compile
```

## Translation File Locations

New strings will appear in:
```
src/locales/
├── en/messages.po  # Source language
├── es/messages.po  # Spanish
├── fr/messages.po  # French
└── ...
```

## Verification

To check if all strings are properly internationalized:

```bash
# Check for hardcoded strings (may have false positives)
grep -rn ">[A-Z][a-z]" src/components/ | grep -v "Trans\|import\|//"
```

## Recommendation

If you added new user-facing strings, run:
```bash
/pearpass-i18n extract
```

This ensures all translatable strings are captured in the catalog.
