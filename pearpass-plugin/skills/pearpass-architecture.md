---
description: "Use this skill to explore and understand PearPass Desktop architecture, including state management, component patterns, vault operations, P2P sync, and the worklet isolation model."
invocable: true
---

# PearPass Desktop Architecture

You are helping a developer understand the PearPass Desktop codebase architecture.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Pear Electron Window                         │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    React Application                         │ │
│  │  ┌──────────────────────────────────────────────────────┐   │ │
│  │  │              Provider Hierarchy (app.tsx)             │   │ │
│  │  │  LoadingProvider → ThemeProvider → VaultProvider →    │   │ │
│  │  │  I18nProvider → ToastProvider → RouterProvider →      │   │ │
│  │  │  ModalProvider → App                                  │   │ │
│  │  └──────────────────────────────────────────────────────┘   │ │
│  │                          │                                   │ │
│  │  ┌──────────┐  ┌────────┴────────┐  ┌──────────────────┐   │ │
│  │  │  Pages   │  │   Components    │  │     Services     │   │ │
│  │  │(screens) │  │ (UI elements)   │  │ (business logic) │   │ │
│  │  └──────────┘  └─────────────────┘  └────────┬─────────┘   │ │
│  └──────────────────────────────────────────────┼──────────────┘ │
│                                                  │                │
│  ┌──────────────────────────────────────────────┼──────────────┐ │
│  │                     pear-run Pipe            │              │ │
│  │              (IPC to isolated worklet)       ▼              │ │
│  └──────────────────────────────────────────────┬──────────────┘ │
└─────────────────────────────────────────────────┼────────────────┘
                                                  │
┌─────────────────────────────────────────────────┼────────────────┐
│                    Vault Worklet                │                │
│  ┌──────────────────────────────────────────────┼──────────────┐ │
│  │              pearpass-lib-vault-core         ▼              │ │
│  │  ┌──────────┐  ┌──────────────┐  ┌─────────────────────┐   │ │
│  │  │  Crypto  │  │   Storage    │  │    P2P Networking   │   │ │
│  │  │(sodium)  │  │ (corestore)  │  │   (hyperswarm)      │   │ │
│  │  └──────────┘  └──────────────┘  └─────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

## Provider Hierarchy

From `app.tsx` - providers wrap the entire app:

```javascript
<LoadingProvider>
  <ThemeProvider>                    {/* pearpass-lib-ui-theme-provider */}
    <VaultProvider>                  {/* pearpass-lib-vault */}
      <I18nProvider i18n={i18n}>     {/* @lingui/react */}
        <ToastProvider>              {/* src/context/ToastContext.js */}
          <RouterProvider>           {/* src/context/RouterContext.js */}
            <ModalProvider>          {/* src/context/ModalContext.js */}
              <App />
            </ModalProvider>
          </RouterProvider>
        </ToastProvider>
      </I18nProvider>
    </VaultProvider>
  </ThemeProvider>
</LoadingProvider>
```

## Context Providers

| Context | Location | Purpose |
|---------|----------|---------|
| `LoadingContext` | `src/context/LoadingContext.js` | Global loading overlay |
| `ModalContext` | `src/context/ModalContext.js` | Modal dialog management |
| `RouterContext` | `src/context/RouterContext.js` | Client-side navigation |
| `ToastContext` | `src/context/ToastContext.js` | Toast notifications |
| `BannerContext` | `src/context/BannerContext.js` | Banner notifications |
| `VaultProvider` | `pearpass-lib-vault` | Vault state & operations |
| `ThemeProvider` | `pearpass-lib-ui-theme-provider` | Theming |

**Usage Pattern:**
```javascript
import { useModal } from '../context/ModalContext'
import { useToast } from '../context/ToastContext'
import { useVault, useRecords } from 'pearpass-lib-vault'

const { openModal, closeModal } = useModal()
const { showToast } = useToast()
const { data: vaultData } = useVault()
```

## Worklet Architecture

Vault operations run in an **isolated worklet process** for security:

```javascript
// src/services/createOrGetPipe.js
import pearRun from 'pear-run'

const WORKLET_PATH_DEV = './node_modules/pearpass-lib-vault-core/src/worklet/app.js'
const WORKLET_PATH_PROD = Pear.config.applink + '/node_modules/pearpass-lib-vault-core/src/worklet/app.js'

export const createOrGetPipe = () => {
  pipe = pearRun(Pear.config.key ? WORKLET_PATH_PROD : WORKLET_PATH_DEV)

  Pear.teardown(() => {
    if (pipe) {
      pipe.end()
      pipe = null
    }
  })

  return pipe
}
```

The worklet (`pearpass-lib-vault-core`) handles:
- Password hashing (Argon2id via sodium-native)
- Vault key encryption/decryption (XSalsa20-Poly1305)
- P2P synchronization (hyperswarm + hyperdht)
- Data storage (corestore + RocksDB)

**Key worklet files:**
- `hashPassword.js` - Argon2id password hashing
- `encryptVaultKeyWithHashedPassword.js` - Key encryption
- `decryptVaultKey.js` - Key decryption
- `pearpassPairer.js` - Device pairing

## Directory Structure

### Pages (`src/pages/`)

| Page | Purpose |
|------|---------|
| `InitialPage/` | App initialization |
| `Intro/` | First-time user intro |
| `LoadingPage/` | Loading screen |
| `MainView/` | Main vault view |
| `SettingsView/` | Settings screen |
| `WelcomePage/` | Login/unlock |

### Components (`src/components/`)

41 feature-specific UI component directories. Each follows:
```
ComponentName/
├── index.js      # Component logic
├── index.test.js # Unit tests
└── styles.js     # Styled components
```

### Lib Components (`src/lib-react-components/`)

Reusable base UI library:

```
lib-react-components/
├── index.js                    # Exports
├── utils/getIconProps.js       # Icon utilities
└── components/
    ├── ButtonPrimary/          # Primary action button
    ├── ButtonThin/             # Thin variant
    ├── ButtonLittle/           # Small button
    ├── ButtonRadio/            # Radio selection
    ├── ButtonFilter/           # Filter button
    ├── ButtonCreate/           # Create action
    ├── ButtonFolder/           # Folder button
    ├── ButtonRoundIcon/        # Round icon button
    ├── ButtonSingleInput/      # Single input button
    ├── InputField/             # Text input
    ├── PasswordField/          # Password with toggle
    ├── PearPassInputField/     # Styled input
    ├── PearPassPasswordField/  # Styled password
    ├── TextArea/               # Multiline input
    ├── Switch/                 # Toggle switch
    ├── Slider/                 # Range slider
    ├── CompoundField/          # Combined fields
    ├── NoticeText/             # Notice display
    ├── HighlightString/        # Search highlighting
    └── TooltipWrapper/         # Tooltip wrapper
```

### Hooks (`src/hooks/`)

30 custom React hooks:

| Hook | Purpose |
|------|---------|
| `useTranslation.ts` | i18n wrapper |
| `useCopyToClipboard.js` | Clipboard with auto-clear |
| `usePasteFromClipboard.js` | Paste operations |
| `useCreateOrEditRecord.js` | Record CRUD |
| `useCustomFields.js` | Custom field logic |
| `useRecordActionItems.js` | Record actions menu |
| `useRecordMenuItems.js` | Record context menu |
| `useConnectExtension.js` | Browser extension connection |
| `usePearUpdate.js` | App updates |
| `useWindowResize.js` | Window resize handling |
| `useOutsideClick.js` | Click outside detection |
| `useAnimatedVisibility.js` | Visibility animations |
| `useLanguageOptions.js` | Language selection |
| `useSimulatedLoading.js` | Loading simulation |
| `useGetMultipleFiles.js` | Multi-file selection |

### Services (`src/services/`)

| Service | Purpose |
|---------|---------|
| `createOrGetPipe.js` | Worklet pipe creation |
| `createOrGetPearpassClient.js` | Vault client factory |
| `nativeMessagingIPCServer.js` | Browser extension IPC |
| `clipboardCleanup.js` | Auto-clear clipboard |
| `handlers/` | Request handlers |
| `ipc/` | IPC utilities |
| `security/` | Security utilities |

## Monorepo Packages

17 git submodules in `packages/`:

### Core Libraries
- `pearpass-lib-vault` - Vault client API (React hooks)
- `pearpass-lib-vault-core` - Vault worklet (encryption/storage)
- `pearpass-lib-ui-theme-provider` - Theme provider
- `pearpass-lib-constants` - Shared constants
- `pearpass-lib-data-export` - Export functionality
- `pearpass-lib-data-import` - Import (1Password, Bitwarden, etc.)

### Utilities
- `pearpass-utils-password-check` - Password strength analysis
- `pearpass-utils-password-generator` - Secure password generation
- `pear-apps-utils-validator` - Form validation
- `pear-apps-utils-pattern-search` - Search patterns
- `pear-apps-utils-avatar-initials` - Avatar generation
- `pear-apps-utils-date` - Date utilities
- `pear-apps-utils-generate-unique-id` - UUID generation
- `pear-apps-utils-qr` - QR code utilities

### Other
- `pear-apps-lib-feedback` - User feedback system
- `pear-apps-lib-ui-react-hooks` - Shared React hooks
- `tether-dev-docs` - Developer documentation

## Data Flow

### User Action → Vault Operation

```
1. User clicks "Save Password"
           ↓
2. Component calls hook (useCreateOrEditRecord)
           ↓
3. Hook uses VaultContext API
           ↓
4. VaultContext sends via pipe to worklet
           ↓
5. Worklet encrypts data (sodium-native)
           ↓
6. Worklet stores in corestore
           ↓
7. Hyperswarm syncs to peers
           ↓
8. Response returns via pipe
           ↓
9. Context updates, component re-renders
```

### Inactivity Detection

From `app.tsx`:
```javascript
const activityEvents = ['mousemove', 'mousedown', 'keydown', 'touchstart', 'scroll']

const resetInactivityTimer = () => {
  clearTimeout(inactivityTimeout)
  inactivityTimeout = setTimeout(() => {
    window.dispatchEvent(new Event('user-inactive'))
  }, 60 * 1000)  // 1 minute timeout
}

activityEvents.forEach(event =>
  window.addEventListener(event, resetInactivityTimer)
)
```

## Styling System

From `pearpass-lib-ui-theme-provider`:

```javascript
// Theme usage in styled-components
import styled from 'styled-components'

const Container = styled.div`
  background: ${({ theme }) => theme.colors.background};
  padding: ${({ theme }) => theme.spacing.md};
  border-radius: ${({ theme }) => theme.borderRadius.default};
`

const Text = styled.p`
  color: ${({ theme }) => theme.colors.text};
  font-family: ${({ theme }) => theme.fonts.primary};
`
```

## Key Entry Points

| File | Purpose |
|------|---------|
| `index.html` | HTML entry point |
| `index.js` | JS entry point |
| `app.tsx` | React root, provider hierarchy |
| `src/app/App` | Main App component |
| `src/services/createOrGetPipe.js` | Worklet connection |
| `src/services/createOrGetPearpassClient.js` | Vault client |

## Exploration Commands

### Find Components
```bash
# List component directories
ls -d src/components/*/
ls -d src/lib-react-components/components/*/

# Find specific component
find src -type d -name "RecordList"
```

### Find Hooks
```bash
# List all hooks
ls src/hooks/

# Search hook usage
grep -r "useCreateOrEditRecord" src/
```

### Find Context Usage
```bash
grep -r "useContext" src/
grep -r "useModal\|useToast\|useRouter" src/
```

### Trace Data Flow
```bash
# Find vault operations
grep -r "useVault\|useRecords" src/

# Find service calls
grep -r "createOrGetPearpassClient\|createOrGetPipe" src/
```

## Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| React Context over Redux | Simpler for feature-organized state |
| Worklet isolation | Security - encryption in separate process |
| Monorepo submodules | Code sharing across PearPass apps |
| styled-components | Component-scoped styling |
| Playwright for E2E | Modern, reliable browser automation |
| TypeScript | Type safety, better IDE support |
