---
description: "Use this skill when the user needs help with PearPass Desktop development workflow, including setup, running the app, building, debugging, testing, or understanding the Pear Runtime development environment."
invocable: true
---

# PearPass Desktop Development Workflow

You are assisting a developer working on PearPass Desktop, a distributed password manager built on Pear Runtime.

## Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Pear Runtime | - | Distributed P2P runtime |
| pear-electron | 1.7.25-rc.0 | Desktop GUI framework |
| React | 19.1.0 | UI framework |
| TypeScript | 5.9.3 | Type safety |
| styled-components | 6.1.19 | CSS-in-JS styling |
| React Context API | - | State management |
| Lingui | 5.1.2 | Internationalization |
| Jest | 29.7.0 | Unit testing |
| Playwright | 1.52.0 | E2E testing |

## Prerequisites

1. **Node.js** - Check version in `.nvmrc`
2. **Pear Runtime** - Install from [docs.pears.com/guides](https://docs.pears.com/guides)

## Initial Setup

```bash
# Clone repository
git clone https://github.com/tetherto/pearpass-app-desktop.git
cd pearpass-app-desktop

# Initialize git submodules (17 packages)
npm run update-submodules
# Or manually:
git submodule update --init --recursive

# Install dependencies
npm install

# Generate translations
npm run lingui:extract
npm run lingui:compile
```

## Running the App

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server with TypeScript watch + Pear |
| `pear run --dev .` | Run directly with Pear (after build) |

The `dev` script runs:
```bash
tsc && concurrently "tsc --watch" "pear run -d ."
```

## Building

| Command | Description |
|---------|-------------|
| `npm run build` | TypeScript compilation only |
| `npm run build:pear` | Full build (lingui extract + compile + tsc) |

### Production Staging
```bash
npm run build:pear
pear stage dev
pear run pear://GENERATED_URL
```

## Code Quality

| Command | Description |
|---------|-------------|
| `npm run lint` | Run ESLint on src/ |
| `npm run lint:fix` | Auto-fix lint issues |
| `npm test` | Run Jest unit tests |

## Internationalization

| Command | Description |
|---------|-------------|
| `npm run lingui:extract` | Extract translation strings |
| `npm run lingui:compile` | Compile translations |

## Project Structure

```
pearpass-app-desktop/
├── app.tsx                 # React root (provider hierarchy)
├── index.html              # HTML entry
├── index.js                # JS entry
├── styles.js               # Global styles/fonts
├── src/
│   ├── app/                # Main App component
│   ├── components/         # UI components (41 dirs)
│   ├── containers/         # Smart components (12 dirs)
│   ├── context/            # React Context providers
│   │   ├── BannerContext.js
│   │   ├── LoadingContext.js
│   │   ├── ModalContext.js
│   │   ├── RouterContext.js
│   │   └── ToastContext.js
│   ├── hooks/              # Custom hooks (30 files)
│   ├── lib-react-components/  # Reusable UI library
│   ├── pages/              # Page components
│   │   ├── InitialPage/
│   │   ├── Intro/
│   │   ├── LoadingPage/
│   │   ├── MainView/
│   │   ├── SettingsView/
│   │   └── WelcomePage/
│   ├── services/           # Service layer
│   │   ├── handlers/
│   │   ├── ipc/
│   │   ├── security/
│   │   └── *.js
│   ├── constants/          # App constants
│   ├── types/              # TypeScript types
│   ├── utils/              # Utilities (28 files)
│   ├── svgs/               # SVG assets
│   └── locales/            # i18n files
├── packages/               # Git submodules (17)
├── e2e/                    # Playwright tests
└── assets/                 # Static assets
```

## Key Configuration Files

| File | Purpose |
|------|---------|
| `tsconfig.json` | TypeScript configuration |
| `jest.config.js` | Jest testing (jsdom environment) |
| `babel.config.cjs` | Babel transpilation |
| `eslint.config.js` | ESLint rules |
| `lingui.config.js` | Lingui i18n configuration |
| `.nvmrc` | Node.js version |

## Monorepo Packages

The 17 submodule packages are in `packages/`:

**Core Libraries:**
- `pearpass-lib-vault` - Vault client API
- `pearpass-lib-vault-core` - Vault worklet (encryption)
- `pearpass-lib-ui-theme-provider` - Theme provider
- `pearpass-lib-constants` - Shared constants
- `pearpass-lib-data-export` - Export functionality
- `pearpass-lib-data-import` - Import functionality

**Utilities:**
- `pearpass-utils-password-check` - Password strength
- `pearpass-utils-password-generator` - Password generation
- `pear-apps-utils-validator` - Validation
- `pear-apps-utils-pattern-search` - Search patterns
- `pear-apps-utils-avatar-initials` - Avatar generation
- `pear-apps-utils-date` - Date utilities
- `pear-apps-utils-generate-unique-id` - UUID generation
- `pear-apps-utils-qr` - QR code utilities

**Other:**
- `pear-apps-lib-feedback` - Feedback system
- `pear-apps-lib-ui-react-hooks` - Shared React hooks
- `tether-dev-docs` - Documentation

## Adding Components

**Standard component structure:**
```
ComponentName/
├── index.js      # Component logic
├── index.test.js # Unit tests
└── styles.js     # Styled components
```

**Example component:**
```javascript
// src/components/MyComponent/index.js
import { Container, Title } from './styles'

export const MyComponent = ({ title }) => (
  <Container>
    <Title>{title}</Title>
  </Container>
)
```

```javascript
// src/components/MyComponent/styles.js
import styled from 'styled-components'

export const Container = styled.div`
  padding: 16px;
  background: ${({ theme }) => theme.colors.background};
`

export const Title = styled.h2`
  color: ${({ theme }) => theme.colors.text};
`
```

## Adding Translations

```javascript
import { Trans, t } from '@lingui/macro'

// In JSX
<Trans>Welcome to PearPass</Trans>

// In code
const message = t`Enter your password`
```

Then run:
```bash
npm run lingui:extract
npm run lingui:compile
```

## Running Tests

```bash
# Unit tests
npm test

# E2E tests (from e2e directory)
cd e2e
npm install
npx playwright test

# E2E with UI
npx playwright test --headed
```

## Common Issues

### TypeScript Errors
```bash
npm run build  # Compile to check errors
```

### Submodule Issues
```bash
npm run update-submodules
# Or manually:
git submodule update --init --recursive
git submodule foreach git pull origin main
```

### Pear Runtime Issues
```bash
# Ensure Pear is installed
pear --version

# Clear and restart
rm -rf node_modules/.cache
npm run dev
```

### Translation Missing
```bash
npm run lingui:extract
npm run lingui:compile
```

## Key Entry Points

- **App Entry**: `app.tsx` → Provider hierarchy → `src/app/App`
- **Providers**: See `app.tsx` for full hierarchy
- **Vault Client**: `src/services/createOrGetPearpassClient.js`
- **Pipe (Worklet)**: `src/services/createOrGetPipe.js`
- **IPC Server**: `src/services/nativeMessagingIPCServer.js`

## Guidelines

1. Use React Context for state (no Redux)
2. Follow styled-components theme pattern
3. Add i18n via Lingui macros
4. Write tests alongside components (*.test.js)
5. Security: Never log credentials
6. Check `src/lib-react-components/` for existing base components
7. Vault operations run in isolated worklet process
