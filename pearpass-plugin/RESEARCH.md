# PearPass Desktop Repository Research

Comprehensive research documentation from analysis of the PearPass desktop app codebase.

## Repository Overview

- **Repository**: https://github.com/tetherto/pearpass-app-desktop
- **Version**: 1.1.0
- **License**: Apache-2.0
- **Module Type**: ES Module
- **Local Clone**: `/Users/jasonkrueger/IdeaProjects/pearpass-app-desktop`

## Project Purpose

PearPass Desktop is an open-source password management application built on Pear Runtime, emphasizing:
- Privacy-first design
- Peer-to-peer synchronization
- End-to-end encryption
- Distributed architecture (no central server)
- Offline access to credentials

---

## Technology Stack

### Core Runtime

| Technology | Version | Purpose |
|------------|---------|---------|
| Pear Runtime | - | Distributed P2P runtime environment |
| pear-electron | 1.7.25-rc.0 | Desktop GUI framework |
| pear-run | 1.0.5 | Worklet runner |
| Node.js | (see .nvmrc) | JavaScript runtime |

### Frontend Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 19.1.0 | UI framework |
| react-dom | 19.1.0 | React DOM renderer |
| TypeScript | 5.9.3 | Type safety |

### State & Styling

| Technology | Version | Purpose |
|------------|---------|---------|
| React Context API | - | State management |
| styled-components | 6.1.19 | CSS-in-JS styling |

### Internationalization

| Technology | Version | Purpose |
|------------|---------|---------|
| @lingui/core | 5.1.2 | i18n core |
| @lingui/react | 5.1.2 | React bindings |
| @lingui/macro | 5.1.2 | Compile-time macros |

### Cryptography (vault-core)

| Technology | Version | Purpose |
|------------|---------|---------|
| sodium-native | 5.0.6 | libsodium bindings (primary crypto) |
| bare-crypto | 1.12.0 | Bare crypto primitives |

### P2P & Storage (vault-core)

| Technology | Version | Purpose |
|------------|---------|---------|
| hyperswarm | 4.16.0 | P2P swarm networking |
| hyperdht | 6.27.0 | Distributed hash table |
| corestore | 7.6.1 | Core data storage |
| rocksdb-native | 3.11.2 | RocksDB storage backend |
| autopass | 3.1.5 | Password autofill |

### Testing & Build

| Technology | Version | Purpose |
|------------|---------|---------|
| Jest | 29.7.0 | Unit testing |
| Playwright | 1.52.0 | E2E testing |
| Babel | 7.26.10 | Transpilation |
| ESLint | - | Linting |
| Husky | 9.1.7 | Git hooks |
| concurrently | 9.2.1 | Parallel commands |

---

## Repository Structure

```
pearpass-app-desktop/
├── app.tsx                 # Main application entry (React root)
├── index.html              # HTML entry point
├── index.js                # JS entry point
├── styles.js               # Global styles and fonts
├── src/
│   ├── app/                # Main App component
│   ├── components/         # React components (41 directories)
│   ├── containers/         # Container/smart components (12 dirs)
│   ├── context/            # React Context providers
│   │   ├── BannerContext.js
│   │   ├── LoadingContext.js
│   │   ├── ModalContext.js
│   │   ├── RouterContext.js
│   │   └── ToastContext.js
│   ├── hooks/              # Custom React hooks (30 files)
│   ├── lib-react-components/  # Shared UI component library
│   │   └── components/     # 20+ reusable components
│   ├── pages/              # Page components
│   │   ├── InitialPage/
│   │   ├── Intro/
│   │   ├── LoadingPage/
│   │   ├── MainView/
│   │   ├── SettingsView/
│   │   └── WelcomePage/
│   ├── services/           # Service layer
│   │   ├── handlers/       # Request handlers
│   │   ├── ipc/            # IPC communication
│   │   ├── security/       # Security utilities
│   │   ├── clipboardCleanup.js
│   │   ├── createOrGetPearpassClient.js
│   │   ├── createOrGetPipe.js
│   │   └── nativeMessagingIPCServer.js
│   ├── constants/          # Application constants (14 files)
│   ├── types/              # TypeScript types
│   ├── utils/              # Utility functions (28 files)
│   ├── svgs/               # SVG icon assets
│   ├── shared/             # Shared utilities
│   └── locales/            # i18n translation files
├── packages/               # Git submodules (17 packages)
│   ├── pearpass-lib-vault/
│   ├── pearpass-lib-vault-core/
│   ├── pearpass-lib-constants/
│   ├── pearpass-lib-data-export/
│   ├── pearpass-lib-data-import/
│   ├── pearpass-lib-ui-theme-provider/
│   ├── pearpass-utils-password-check/
│   ├── pearpass-utils-password-generator/
│   ├── pear-apps-lib-feedback/
│   ├── pear-apps-lib-ui-react-hooks/
│   ├── pear-apps-utils-avatar-initials/
│   ├── pear-apps-utils-date/
│   ├── pear-apps-utils-generate-unique-id/
│   ├── pear-apps-utils-pattern-search/
│   ├── pear-apps-utils-qr/
│   ├── pear-apps-utils-validator/
│   └── tether-dev-docs/
├── e2e/                    # Playwright E2E tests
│   ├── specs/              # Test specifications
│   │   ├── createLoginItem.test.js
│   │   ├── editingDeletingLoginItem.test.js
│   │   ├── login.test.js
│   │   └── password.test.js
│   ├── components/         # Page objects
│   ├── fixtures/           # Test fixtures
│   └── playwright.config.js
├── assets/                 # Static assets
│   ├── images/
│   ├── fonts/
│   ├── video/
│   ├── rive/               # Rive animations
│   ├── darwin/             # macOS assets
│   ├── linux/              # Linux assets
│   └── win32/              # Windows assets
├── appling/                # Pear app configuration
└── scripts/                # Build/maintenance scripts
```

---

## Architecture Patterns

### 1. Provider-Based State Management

The app uses React Context API (no Redux):

```
App Provider Hierarchy (from app.tsx):
├── LoadingProvider (global loading state)
├── ThemeProvider (UI theme - from pearpass-lib-ui-theme-provider)
├── VaultProvider (vault/encryption state - from pearpass-lib-vault)
├── I18nProvider (internationalization - Lingui)
├── ToastProvider (toast notifications)
├── RouterProvider (client-side routing)
└── ModalProvider (modal dialogs)
```

**Key Contexts:**
| Context | File | Purpose |
|---------|------|---------|
| `LoadingContext` | `src/context/LoadingContext.js` | Global loading overlay |
| `ModalContext` | `src/context/ModalContext.js` | Modal dialog state |
| `RouterContext` | `src/context/RouterContext.js` | Client-side navigation |
| `ToastContext` | `src/context/ToastContext.js` | Toast notifications |
| `BannerContext` | `src/context/BannerContext.js` | Banner notifications |

### 2. Pear Runtime Integration

**Worklet Architecture:**
- Main app runs in Pear Electron window
- Vault operations run in separate worklet (isolated process)
- Communication via `pear-run` pipe

```javascript
// From src/services/createOrGetPipe.js
import pearRun from 'pear-run'

const WORKLET_PATH_DEV = './node_modules/pearpass-lib-vault-core/src/worklet/app.js'
const WORKLET_PATH_PROD = Pear.config.applink + '/node_modules/pearpass-lib-vault-core/src/worklet/app.js'

export const createOrGetPipe = () => {
  pipe = pearRun(Pear.config.key ? WORKLET_PATH_PROD : WORKLET_PATH_DEV)
  return pipe
}
```

### 3. Custom Hooks

Key hooks in `src/hooks/`:

| Hook | Purpose |
|------|---------|
| `useTranslation.ts` | i18n translations |
| `useCopyToClipboard.js` | Clipboard operations with auto-clear |
| `usePasteFromClipboard.js` | Paste operations |
| `useCreateOrEditRecord.js` | CRUD for vault records |
| `useCustomFields.js` | Custom field management |
| `useRecordActionItems.js` | Record action menu |
| `useRecordMenuItems.js` | Record context menu |
| `useConnectExtension.js` | Browser extension connection |
| `usePearUpdate.js` | Pear app updates |
| `useWindowResize.js` | Window resize handling |
| `useOutsideClick.js` | Click outside detection |
| `useAnimatedVisibility.js` | Animation transitions |
| `useLanguageOptions.js` | Language selection |
| `useSimulatedLoading.js` | Loading state simulation |
| `useGetMultipleFiles.js` | Multi-file selection |

### 4. Component Library Structure

Reusable components in `src/lib-react-components/`:

```
lib-react-components/
├── index.js              # Exports
├── utils/
│   └── getIconProps.js   # Icon utilities
└── components/
    ├── ButtonPrimary/    # Primary button
    ├── ButtonThin/       # Thin button variant
    ├── ButtonLittle/     # Small button
    ├── ButtonRadio/      # Radio button
    ├── ButtonFilter/     # Filter button
    ├── ButtonCreate/     # Create button
    ├── ButtonFolder/     # Folder button
    ├── ButtonRoundIcon/  # Round icon button
    ├── ButtonSingleInput/# Single input button
    ├── InputField/       # Text input
    ├── PasswordField/    # Password input
    ├── PearPassInputField/    # PearPass-styled input
    ├── PearPassPasswordField/ # PearPass password input
    ├── TextArea/         # Textarea
    ├── Switch/           # Toggle switch
    ├── Slider/           # Range slider
    ├── CompoundField/    # Compound input
    ├── NoticeText/       # Notice text
    ├── HighlightString/  # Text highlighting
    └── TooltipWrapper/   # Tooltip wrapper
```

Each component follows the pattern:
```
ComponentName/
├── index.js      # Component logic
├── index.test.js # Unit tests
└── styles.js     # Styled components
```

---

## Security Implementation

### Master Password Hashing

**From `packages/pearpass-lib-vault-core/src/worklet/hashPassword.js`:**
```javascript
import sodium from 'sodium-native'

export const hashPassword = (password) => {
  const salt = sodium.sodium_malloc(sodium.crypto_pwhash_SALTBYTES)
  sodium.randombytes_buf(salt)

  const hashedPassword = sodium.sodium_malloc(sodium.crypto_secretbox_KEYBYTES)

  // Argon2id with sensitive parameters
  sodium.crypto_pwhash(
    hashedPassword,
    Buffer.from(password),
    salt,
    sodium.crypto_pwhash_OPSLIMIT_SENSITIVE,  // High computation
    sodium.crypto_pwhash_MEMLIMIT_INTERACTIVE, // Moderate memory
    sodium.crypto_pwhash_ALG_DEFAULT           // Argon2id
  )

  return {
    hashedPassword: Buffer.from(hashedPassword).toString('hex'),
    salt: salt.toString('base64')
  }
}
```

### Vault Key Encryption

**From `encryptVaultKeyWithHashedPassword.js`:**
```javascript
export const encryptVaultKeyWithHashedPassword = (hashedPassword) => {
  const nonce = sodium.sodium_malloc(sodium.crypto_secretbox_NONCEBYTES)
  const key = sodium.sodium_malloc(32)

  sodium.randombytes_buf(key)
  sodium.randombytes_buf(nonce)

  // XSalsa20-Poly1305 authenticated encryption
  sodium.crypto_secretbox_easy(
    ciphertext, key, nonce,
    Buffer.from(hashedPassword, 'hex')
  )

  return { ciphertext, nonce }
}
```

### Vault Key Decryption

**From `decryptVaultKey.js`:**
```javascript
export const decryptVaultKey = (data) => {
  // Authenticated decryption - fails if tampered
  if (!sodium.crypto_secretbox_open_easy(
    plainText, ciphertext, nonce, hashedPassword
  )) {
    return undefined  // Authentication failed
  }
  return plainText.toString('base64')
}
```

### Key Security Features

1. **sodium-native** - High-quality libsodium bindings
2. **Memory-safe buffers** - `sodium_malloc` for sensitive data
3. **Argon2id** - Memory-hard password hashing
4. **XSalsa20-Poly1305** - Authenticated encryption
5. **Random nonce** - Unique nonce per encryption

### Clipboard Security

**From `src/services/clipboardCleanup.js`:**
- Auto-clears clipboard after timeout
- Checks if clipboard unchanged before clearing
- Cancels timer on new copy

### Native Messaging IPC

**From `src/services/nativeMessagingIPCServer.js`:**
- Handles browser extension communication
- Secure IPC protocol
- Rate limiting (from `rateLimiter.js`)

---

## UI Component Patterns

### Styled Components with Theme

```javascript
// Standard pattern from lib-react-components
import styled from 'styled-components'

export const Button = styled.button`
  padding: ${({ size }) => (size === 'sm' ? '10px 15px' : '10px 40px')};
  border-radius: ${({ size }) => (size === 'sm' ? '10px' : '20px')};
  background: ${({ theme }) => theme.colors.primary400};
  opacity: ${({ disabled }) => (disabled ? 0.6 : 1)};
`
```

### Internationalization Pattern

```javascript
import { i18n } from '@lingui/core'
import { Trans, t } from '@lingui/macro'

// JSX
<Trans>Welcome to PearPass</Trans>

// String in code
const message = t`Enter your password`
```

### Inactivity Detection

**From `app.tsx`:**
```javascript
const activityEvents = ['mousemove', 'mousedown', 'keydown', 'touchstart', 'scroll']

const resetInactivityTimer = () => {
  clearTimeout(inactivityTimeout)
  inactivityTimeout = setTimeout(() => {
    window.dispatchEvent(new Event('user-inactive'))
  }, 60 * 1000)  // 1 minute
}

activityEvents.forEach(event =>
  window.addEventListener(event, resetInactivityTimer)
)
```

---

## Testing Patterns

### Jest Unit Tests

**Configuration (`jest.config.js`):**
```javascript
export default {
  testEnvironment: 'jsdom',
  transform: { '^.+\\.[jt]sx?$': 'babel-jest' },
  testPathIgnorePatterns: ['/node_modules/', '/packages/', '/e2e/'],
  globals: {
    Pear: { config: { tier: 'dev' } }  // Mock Pear global
  }
}
```

**Component Test Example:**
```javascript
import { render, fireEvent } from '@testing-library/react'
import { ThemeProvider } from 'pearpass-lib-ui-theme-provider'

test('button triggers onPress', () => {
  const onPress = jest.fn()
  const { getByText } = render(
    <ThemeProvider>
      <ButtonPrimary onPress={onPress}>Click</ButtonPrimary>
    </ThemeProvider>
  )
  fireEvent.click(getByText('Click'))
  expect(onPress).toHaveBeenCalled()
})
```

### Playwright E2E Tests

**Configuration (`e2e/playwright.config.js`):**
```javascript
module.exports = {
  timeout: 5 * 60 * 3000,
  use: {
    screenshot: 'only-on-failure',
    actionTimeout: 30000,
    navigationTimeout: 60000
  },
  reporter: [
    ['html', { outputFolder: 'test-artifacts/report' }],
    ['playwright-qase-reporter', { /* Qase integration */ }]
  ],
  workers: 1,
  fullyParallel: false
}
```

**Page Object Pattern:**
```javascript
// e2e/components/LoginPage.js
class LoginPage {
  constructor(locator) {
    this.root = locator
  }
  get passwordInput() { return this.root.locator('[data-testid="password"]') }
  get continueButton() { return this.root.locator('[data-testid="continue"]') }

  async login(password) {
    await this.passwordInput.fill(password)
    await this.continueButton.click()
  }
}
```

---

## NPM Scripts

| Script | Command | Description |
|--------|---------|-------------|
| `dev` | `tsc && concurrently "tsc --watch" "pear run -d ."` | Development with hot reload |
| `build` | `tsc` | TypeScript compilation |
| `build:pear` | `lingui extract && lingui compile && tsc` | Full Pear build |
| `test` | `jest` | Run unit tests |
| `lint` | `eslint ./src` | Run ESLint |
| `lint:fix` | `eslint --fix ./src` | Auto-fix lint issues |
| `lingui:extract` | `lingui extract` | Extract translation strings |
| `lingui:compile` | `lingui compile` | Compile translations |
| `update-submodules` | `bash scripts/update-submodules.sh` | Update git submodules |

---

## Development Workflow

### Prerequisites
1. Node.js (version specified in `.nvmrc`)
2. Pear Runtime installed ([docs.pears.com/guides](https://docs.pears.com/guides))

### Setup
```bash
git clone git@github.com:tetherto/pearpass-app-desktop.git
cd pearpass-app-desktop
npm run update-submodules
npm install
npm run lingui:extract
npm run lingui:compile
```

### Running Development
```bash
# Option 1: Using npm script (recommended)
npm run dev

# Option 2: Direct Pear command
pear run --dev .
```

### Running Tests
```bash
# Unit tests
npm test

# E2E tests (from e2e/ directory)
cd e2e && npm install && npx playwright test
```

### Production Build
```bash
npm run build:pear
pear stage dev
pear run pear://GENERATED_URL
```

---

## Key Files for Plugin Development

### Security-Critical Files
- `packages/pearpass-lib-vault-core/src/worklet/hashPassword.js` - Password hashing
- `packages/pearpass-lib-vault-core/src/worklet/decryptVaultKey.js` - Key decryption
- `packages/pearpass-lib-vault-core/src/worklet/encryptVaultKeyWithHashedPassword.js` - Key encryption
- `src/services/clipboardCleanup.js` - Clipboard security
- `src/services/nativeMessagingIPCServer.js` - IPC security

### Component Patterns
- `src/lib-react-components/` - Base UI components
- `src/components/` - Application components
- `src/containers/` - Smart/container components

### Testing Examples
- `src/lib-react-components/components/*/index.test.js` - Component tests
- `src/hooks/*.test.js` - Hook tests
- `src/services/*.test.js` - Service tests
- `e2e/specs/*.test.js` - E2E tests

### Context/State
- `src/context/*.js` - All context providers
- `packages/pearpass-lib-vault/` - Vault state management

---

## Related Projects

- [pearpass-app-mobile](https://github.com/tetherto/pearpass-app-mobile) - Mobile app
- [pearpass-app-browser-extension](https://github.com/tetherto/pearpass-app-browser-extension) - Browser extension
- [pearpass-lib-vault](https://github.com/tetherto/pearpass-lib-vault) - Vault library
- [pearpass-lib-vault-core](https://github.com/tetherto/pearpass-lib-vault-core) - Core vault worker
- [tether-dev-docs](https://github.com/tetherto/tether-dev-docs) - Developer documentation

---

## Plugin Opportunities Summary

### High-Priority Skills
1. **pearpass-dev** - Development setup, Pear commands, debugging
2. **pearpass-security** - Security review, crypto best practices
3. **pearpass-pear** - Pear Runtime architecture and patterns

### High-Priority Agents
1. **security-reviewer** - Audit crypto code, vault operations
2. **code-explorer** - Navigate monorepo, find patterns
3. **component-dev** - Create components following established patterns

### Commands
1. `/pearpass-dev` - Start development server
2. `/pearpass-build` - Build the application
3. `/pearpass-test` - Run test suites
4. `/pearpass-lint` - Run linting
5. `/pearpass-i18n` - i18n extract/compile

### Hooks
1. **pre-edit-crypto** - Warn on vault-core file changes
2. **post-edit-component** - Suggest tests for new components

---

*Research completed: January 2026*
*Repository analyzed: pearpass-app-desktop v1.1.0*
