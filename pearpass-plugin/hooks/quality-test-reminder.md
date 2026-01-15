---
event: PostToolUse
tools:
  - Write
match_files:
  - "**/components/**/index.js"
  - "**/components/**/index.jsx"
  - "**/containers/**/index.js"
  - "**/containers/**/index.jsx"
  - "**/hooks/use*.js"
  - "**/context/*.js"
  - "**/context/*.jsx"
---

# Test Coverage Reminder

You've created or modified a component, hook, or context. Consider adding tests.

## Test File Locations

| Source | Test Location |
|--------|---------------|
| `src/components/Foo/index.js` | `src/components/Foo/index.test.js` |
| `src/hooks/useFoo.js` | `src/hooks/useFoo.test.js` |
| `src/context/FooContext.js` | `src/context/FooContext.test.jsx` |

## Quick Test Templates

### Component Test
```javascript
import { render, fireEvent } from '@testing-library/react-native'
import { ThemeProvider } from 'pearpass-lib-ui-theme-provider/native'
import { ComponentName } from './index'

const renderWithTheme = (component) =>
  render(<ThemeProvider>{component}</ThemeProvider>)

describe('ComponentName', () => {
  test('renders correctly', () => {
    const { getByText } = renderWithTheme(
      <ComponentName>Test</ComponentName>
    )
    expect(getByText('Test')).toBeTruthy()
  })

  test('handles press events', () => {
    const onPress = jest.fn()
    const { getByText } = renderWithTheme(
      <ComponentName onPress={onPress}>Click</ComponentName>
    )
    fireEvent.press(getByText('Click'))
    expect(onPress).toHaveBeenCalled()
  })
})
```

### Hook Test
```javascript
import { renderHook, act, waitFor } from '@testing-library/react-native'
import { useHookName } from './useHookName'

// Mock native modules first
jest.mock('expo-secure-store', () => ({
  setItemAsync: jest.fn(),
  getItemAsync: jest.fn(),
  deleteItemAsync: jest.fn()
}))

describe('useHookName', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  test('initializes with default values', async () => {
    const { result } = renderHook(() => useHookName())

    await waitFor(() => {
      expect(result.current.value).toBeDefined()
    })
  })

  test('updates state correctly', async () => {
    const { result } = renderHook(() => useHookName())

    await act(async () => {
      await result.current.setValue('new value')
    })

    expect(result.current.value).toBe('new value')
  })
})
```

### Context Test
```javascript
import { renderHook, act } from '@testing-library/react-native'
import { FooProvider, useFoo } from './FooContext'

describe('FooContext', () => {
  const wrapper = ({ children }) => (
    <FooProvider>{children}</FooProvider>
  )

  test('provides default values', () => {
    const { result } = renderHook(() => useFoo(), { wrapper })
    expect(result.current.value).toBeDefined()
  })

  test('updates value', () => {
    const { result } = renderHook(() => useFoo(), { wrapper })

    act(() => {
      result.current.setValue('updated')
    })

    expect(result.current.value).toBe('updated')
  })
})
```

## Common Mocks Needed

### Expo Native Modules
```javascript
jest.mock('expo-secure-store', () => ({...}))
jest.mock('expo-local-authentication', () => ({...}))
jest.mock('expo-clipboard', () => ({...}))
```

### Vault Library
```javascript
jest.mock('pearpass-lib-vault', () => ({
  useVault: jest.fn(() => ({ data: { isUnlocked: true } })),
  useRecords: jest.fn(() => ({ data: [] }))
}))
```

### Navigation
```javascript
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({ navigate: jest.fn() }),
  useRoute: () => ({ params: {} })
}))
```

## Reference Tests

Look at these existing tests for patterns:

| Pattern | Example |
|---------|---------|
| Complex mocking | `src/hooks/useBiometricsAuthentication.test.js` |
| Timer testing | `src/hooks/useCopyToClipboard.test.js` |
| Context testing | `src/context/ModalContext.test.jsx` |

## Running Tests

```bash
# Run all tests
npm test

# Run specific test
npm test -- path/to/file.test.js

# Watch mode
npm test -- --watch
```

## Recommendation

If you created a new component/hook/context, consider using the test-generator agent:

```
Use the test-generator agent to create tests for [file path]
```

This will generate a complete test file following project patterns.
