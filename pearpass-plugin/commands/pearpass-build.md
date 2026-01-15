---
description: Build PearPass Desktop application for iOS, Android, or prepare bundles
invocable: true
arguments:
  - name: platform
    description: "Target platform: ios, android, or all (default: all)"
    required: false
  - name: clean
    description: "Clean build: true or false (default: false)"
    required: false
---

# Build PearPass Desktop

Build the PearPass Desktop application for the specified platform.

## Usage

```
/pearpass-build [platform] [--clean]
```

**Arguments:**
- `platform`: `ios`, `android`, or `all` (default)
- `--clean`: Perform a clean build (removes cached build artifacts)

## Build Process Overview

The PearPass build involves several steps:

1. **Custom Prebuild** - Expo prebuild with custom native modules
2. **Bundle Bare** - Package vault-core worklet for native execution
3. **Lingui Extract/Compile** - Process i18n translations
4. **Platform Build** - Compile native app

## Instructions

### Full Build (All Platforms)

```bash
# Navigate to project directory
cd /path/to/pearpass-app-desktop

# Run full build
npm run build
```

This executes:
1. `npm run custom-prebuild` - Expo prebuild with custom config
2. `npm run bundle-bare` - Bundle worklet for both platforms
3. `npm run lingui:extract` - Extract translation strings
4. `npm run lingui:compile` - Compile translations

### iOS Only

```bash
# Standard iOS build
npm run custom-prebuild:ios
npm run bundle:ios
npm run bundle:ios-extension
expo run:ios

# Clean iOS build
npm run custom-prebuild-ios:clean
npm run bundle:ios
npm run bundle:ios-extension
expo run:ios
```

### Android Only

```bash
# Standard Android build
npm run custom-prebuild:android
npm run bundle:android
npm run bundle:android-extension
expo run:android

# Clean Android build
npm run custom-prebuild-android:clean
npm run bundle:android
npm run bundle:android-extension
expo run:android
```

## Bundle Scripts

The worklet bundles are critical for vault operations:

| Script | Output | Purpose |
|--------|--------|---------|
| `bundle:ios` | `bundles/app-ios.bundle.js` | iOS vault worklet |
| `bundle:android` | `bundles/app-android.bundle.js` | Android vault worklet |
| `bundle:ios-extension` | `ios/PearPassAutofillExtension/extension.bundle` | iOS autofill extension |
| `bundle:android-extension` | `android/app/src/main/assets/extension.bundle` | Android autofill extension |

## Common Build Issues

### Issue: Worklet Bundle Missing

**Symptom:** App crashes on vault operations

**Fix:**
```bash
npm run bundle-bare
```

### Issue: Native Module Mismatch

**Symptom:** Build fails with native dependency errors

**Fix:**
```bash
npm run custom-prebuild:clean
npm run build
```

### Issue: Submodules Out of Sync

**Symptom:** Missing packages/ dependencies

**Fix:**
```bash
npm run update-submodules
npm install
```

### Issue: Lingui Compilation Errors

**Symptom:** Missing translations or i18n errors

**Fix:**
```bash
npm run lingui:extract
npm run lingui:compile
```

## EAS Build (Production)

For production builds using Expo Application Services:

```bash
# The pre/post install scripts handle bundling
eas build --platform ios
eas build --platform android
```

The `eas-build-post-install` script automatically runs:
- `bundle-bare`
- `lingui:extract`
- `lingui:compile`

## Verification

After building, verify:
1. App launches without crashes
2. Vault operations work (create/unlock)
3. Autofill extension appears in system settings
4. Translations display correctly
