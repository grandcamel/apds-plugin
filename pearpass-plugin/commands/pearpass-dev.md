---
description: Start PearPass development server and prepare development environment
invocable: true
arguments:
  - name: platform
    description: "Target platform: ios, android, or expo (default: expo)"
    required: false
---

# PearPass Development

Start the PearPass development environment.

## Usage

```
/pearpass-dev [platform]
```

**Arguments:**
- `platform`: `ios`, `android`, or `expo` (default)

## Quick Start

### 1. First-Time Setup

```bash
cd /path/to/pearpass-app-mobile

# Initialize submodules (17 packages)
git submodule update --init --recursive

# Install dependencies
npm install

# Bundle worklets (required for vault operations)
npm run bundle-bare

# Compile translations
npm run lingui:extract
npm run lingui:compile
```

### 2. Start Development Server

```bash
# Expo development client (recommended)
npm start
# or
expo start --dev-client

# Run on iOS simulator
npm run ios
# or
expo run:ios

# Run on Android emulator
npm run android
# or
expo run:android
```

## Development Prerequisites

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| Node.js | 18+ | `node --version` |
| npm | 9+ | `npm --version` |
| Expo CLI | Latest | `npx expo --version` |
| Xcode | 15+ | `xcodebuild -version` |
| Android Studio | Latest | Check Android Studio |
| CocoaPods | Latest | `pod --version` |

## Project Structure

```
pearpass-app-mobile/
├── src/
│   ├── components/       # UI components
│   ├── containers/       # Smart components
│   ├── context/          # React Context providers
│   ├── hooks/            # Custom hooks
│   ├── pages/            # Screen components
│   ├── services/         # Service layer
│   └── locales/          # i18n translations
├── packages/             # Git submodules (17)
│   ├── pearpass-lib-vault/
│   ├── pearpass-lib-vault-core/
│   └── ...
├── bundles/              # Compiled worklet bundles
├── e2e/                  # Maestro E2E tests
└── e2e-mobile-tests/     # WebdriverIO E2E tests
```

## Key Development Commands

| Command | Description |
|---------|-------------|
| `npm start` | Start Expo dev server |
| `npm run ios` | Run on iOS |
| `npm run android` | Run on Android |
| `npm test` | Run Jest tests |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Auto-fix lint issues |
| `npm run bundle-bare` | Rebuild worklet bundles |
| `npm run update-submodules` | Update git submodules |

## Provider Hierarchy

The app uses nested React Context providers:

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

## Common Development Tasks

### Adding a New Component

1. Create component folder in `src/components/`
2. Add `index.js`, `styles.js`, `index.test.js`
3. Use styled-components with theme
4. Add Lingui translations

### Adding a New Hook

1. Create in `src/hooks/`
2. Mock native modules in tests
3. Use React Context for global state

### Modifying Vault Logic

1. Changes go in `packages/pearpass-lib-vault-core/`
2. Rebuild bundles: `npm run bundle-bare`
3. Test vault operations thoroughly

## Hot Reload Troubleshooting

### Issue: Changes Not Reflecting

```bash
# Clear Metro cache
npx expo start --clear

# Or manually
rm -rf node_modules/.cache
```

### Issue: Native Code Changes

Native changes require rebuild:
```bash
npm run custom-prebuild:clean
npm run ios  # or android
```

### Issue: Submodule Changes Not Applied

```bash
npm run update-submodules
npm install
npm run bundle-bare
```

## Environment Variables

Create `.env` file if needed:
```bash
# Copy from example if exists
cp .env.example .env
```

## Debugging

### React Native Debugger

1. Shake device or `Cmd+D` (iOS) / `Cmd+M` (Android)
2. Select "Debug with Chrome" or use Flipper

### Console Logging

```javascript
console.log('Debug:', value)
// View in Metro bundler terminal
```

### Vault Operations Debugging

Vault operations run in isolated worklet - check:
1. Bundle is up to date: `npm run bundle-bare`
2. Check worklet console output
3. Verify encryption keys in SecureStore

## Performance Tips

1. **Use production builds for performance testing** - Dev builds are slower
2. **Profile with Flipper** - Memory, CPU, network
3. **Check re-renders** - Use React DevTools
4. **Monitor bundle size** - Large bundles slow startup
