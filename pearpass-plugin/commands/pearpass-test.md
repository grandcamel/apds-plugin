---
description: Run PearPass Desktop test suites - Jest unit tests, specific test files, or watch mode
invocable: true
arguments:
  - name: type
    description: "Test type: unit, watch, file, or coverage"
    required: false
  - name: path
    description: "Specific test file or pattern to run"
    required: false
---

# Test PearPass Desktop

Run the PearPass Desktop test suites.

## Usage

```
/pearpass-test [type] [path]
```

**Arguments:**
- `type`: `unit` (default), `watch`, `file`, or `coverage`
- `path`: Specific test file or pattern (used with `file` type)

## Instructions

### Run All Unit Tests

```bash
cd /path/to/pearpass-app-desktop
npm test
```

### Run Tests in Watch Mode

```bash
npm test -- --watch
```

### Run Specific Test File

```bash
# Single file
npm test -- src/hooks/useBiometricsAuthentication.test.js

# Pattern matching
npm test -- --testPathPattern="useCopyToClipboard"
```

### Run with Coverage

```bash
npm test -- --coverage
```

## Test Locations

| Test Type | Location | Framework |
|-----------|----------|-----------|
| Component tests | `src/libComponents/*/index.test.js` | Jest |
| Hook tests | `src/hooks/*.test.js` | Jest |
| Context tests | `src/context/*.test.jsx` | Jest |
| Utility tests | `src/utils/*.test.js` | Jest |
| E2E flows | `e2e/**/*.yaml` | Maestro |
| E2E specs | `e2e-mobile-tests/tests/specs/*.ts` | WebdriverIO |

## Reference Test Files

Study these for testing patterns:

| Test | Location | Pattern |
|------|----------|---------|
| Biometrics hook | `src/hooks/useBiometricsAuthentication.test.js` | Complex mocking |
| Clipboard hook | `src/hooks/useCopyToClipboard.test.js` | Timer testing |
| Context | `src/context/AutoLockContext.test.jsx` | Context testing |
| Modal context | `src/context/ModalContext.test.jsx` | Component + context |
| Button | `src/libComponents/ButtonPrimary/index.test.js` | Component testing |

## Common Mock Patterns

### SecureStore Mock

```javascript
jest.mock('expo-secure-store', () => ({
  setItemAsync: jest.fn().mockResolvedValue(),
  getItemAsync: jest.fn().mockResolvedValue(null),
  deleteItemAsync: jest.fn().mockResolvedValue()
}))
```

### Biometrics Mock

```javascript
jest.mock('expo-local-authentication', () => ({
  authenticateAsync: jest.fn().mockResolvedValue({ success: true }),
  hasHardwareAsync: jest.fn().mockResolvedValue(true),
  isEnrolledAsync: jest.fn().mockResolvedValue(true),
  supportedAuthenticationTypesAsync: jest.fn().mockResolvedValue([1, 2]),
  AuthenticationType: { FACIAL_RECOGNITION: 1, FINGERPRINT: 2 }
}))
```

### Clipboard Mock

```javascript
jest.mock('expo-clipboard', () => ({
  setStringAsync: jest.fn().mockResolvedValue(),
  getStringAsync: jest.fn().mockResolvedValue('')
}))
```

### Vault Mock

```javascript
jest.mock('pearpass-lib-vault', () => ({
  useVault: jest.fn(() => ({
    data: { isUnlocked: true },
    initVaults: jest.fn(),
    closeAllInstances: jest.fn()
  })),
  useRecords: jest.fn(() => ({
    data: [],
    createRecord: jest.fn()
  }))
}))
```

## Timer Testing Pattern

For hooks using `setTimeout` or `setInterval`:

```javascript
describe('useTimerHook', () => {
  beforeEach(() => {
    jest.clearAllMocks()
    jest.useFakeTimers()
  })

  afterEach(() => {
    jest.clearAllTimers()
    jest.useRealTimers()
  })

  test('executes after timeout', async () => {
    const { result } = renderHook(() => useTimerHook(), { wrapper })

    await act(async () => {
      await result.current.startTimer()
    })

    await act(async () => {
      jest.advanceTimersByTime(30000)
    })

    expect(mockCallback).toHaveBeenCalled()
  })
})
```

## Debugging Failing Tests

### Issue: Test Timeout

```bash
# Increase timeout
npm test -- --testTimeout=10000
```

### Issue: Module Not Found

Check that mocks are defined before imports:
```javascript
// Mocks MUST be before imports
jest.mock('expo-secure-store', () => ({...}))

// Then import
import { useHook } from './useHook'
```

### Issue: Act Warnings

Wrap state updates in `act()`:
```javascript
await act(async () => {
  await result.current.someAction()
})
```

## Running E2E Tests (Maestro)

```bash
# Install Maestro if not present
curl -Ls "https://get.maestro.mobile.dev" | bash

# Run specific flow
maestro test e2e/home/recordCreate.yaml

# Run all flows in directory
maestro test e2e/
```

## Test Output Analysis

When tests fail, check:
1. **Mock setup** - Are all native modules mocked?
2. **Async handling** - Using `waitFor` or `act` correctly?
3. **Provider wrapping** - Component wrapped in required providers?
4. **Timer handling** - Using fake timers for timeouts?
