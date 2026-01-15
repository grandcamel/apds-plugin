---
description: "Use this skill when helping with PearPass Desktop testing - writing Jest unit tests, Playwright E2E tests, mocking contexts, or understanding test patterns in the codebase."
invocable: true
---

# PearPass Desktop Testing Guide

You are helping a developer write tests for PearPass Desktop password manager.

## Testing Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Jest | 29.7.0 | Unit testing framework |
| @testing-library/react | - | Component testing |
| Playwright | 1.52.0 | E2E browser testing |

## Test Locations

```
pearpass-app-desktop/
├── src/
│   ├── components/*/index.test.js      # Feature component tests
│   ├── lib-react-components/components/*/index.test.js  # Base component tests
│   ├── hooks/*.test.js                 # Hook tests
│   ├── context/*.test.js               # Context tests
│   ├── services/*.test.js              # Service tests
│   └── utils/*.test.js                 # Utility tests
└── e2e/                                # Playwright tests
    └── *.spec.ts                       # E2E specs
```

## Running Tests

```bash
# Jest unit tests
npm test

# Jest with watch mode
npm test -- --watch

# Jest specific file
npm test -- src/hooks/useCopyToClipboard.test.js

# Jest with coverage
npm test -- --coverage

# Playwright E2E tests
npm run test:e2e

# Playwright with UI
npm run test:e2e -- --ui

# Playwright specific file
npm run test:e2e -- e2e/login.spec.ts
```

---

## Component Testing Patterns

### Basic Component Test

```javascript
// src/components/ButtonPrimary/index.test.js
import { render, fireEvent } from '@testing-library/react'
import { ThemeProvider } from 'pearpass-lib-ui-theme-provider'
import { ButtonPrimary } from './index'

describe('ButtonPrimary', () => {
  const renderWithTheme = (ui) => render(
    <ThemeProvider>{ui}</ThemeProvider>
  )

  test('renders children correctly', () => {
    const { getByText } = renderWithTheme(
      <ButtonPrimary onClick={jest.fn()}>Click me</ButtonPrimary>
    )
    expect(getByText('Click me')).toBeInTheDocument()
  })

  test('triggers onClick when clicked', () => {
    const onClick = jest.fn()
    const { getByText } = renderWithTheme(
      <ButtonPrimary onClick={onClick}>Click me</ButtonPrimary>
    )
    fireEvent.click(getByText('Click me'))
    expect(onClick).toHaveBeenCalledTimes(1)
  })

  test('does not trigger onClick when disabled', () => {
    const onClick = jest.fn()
    const { getByRole } = renderWithTheme(
      <ButtonPrimary onClick={onClick} disabled>Click me</ButtonPrimary>
    )
    fireEvent.click(getByRole('button'))
    expect(onClick).not.toHaveBeenCalled()
  })
})
```

### Component with Context

```javascript
// src/components/RecordList/index.test.js
import { render, screen } from '@testing-library/react'
import { ThemeProvider } from 'pearpass-lib-ui-theme-provider'
import { VaultProvider } from 'pearpass-lib-vault'
import { RecordList } from './index'

// Mock vault
jest.mock('pearpass-lib-vault', () => ({
  VaultProvider: ({ children }) => children,
  useRecords: () => ({
    data: [
      { id: '1', title: 'Test Record', type: 'login' }
    ],
    loading: false
  })
}))

describe('RecordList', () => {
  const renderWithProviders = (ui) => render(
    <ThemeProvider>
      <VaultProvider>{ui}</VaultProvider>
    </ThemeProvider>
  )

  test('renders records from vault', () => {
    renderWithProviders(<RecordList />)
    expect(screen.getByText('Test Record')).toBeInTheDocument()
  })
})
```

---

## Context Testing Patterns

### Simple Context Test

```javascript
// src/context/ToastContext.test.js
import { renderHook, act } from '@testing-library/react'
import { ToastProvider, useToast } from './ToastContext'

describe('ToastContext', () => {
  const wrapper = ({ children }) => (
    <ToastProvider>{children}</ToastProvider>
  )

  test('provides showToast function', () => {
    const { result } = renderHook(() => useToast(), { wrapper })
    expect(typeof result.current.showToast).toBe('function')
  })

  test('shows toast with message', () => {
    const { result } = renderHook(() => useToast(), { wrapper })

    act(() => {
      result.current.showToast('Success message')
    })

    expect(result.current.toast).toBe('Success message')
  })
})
```

### Modal Context with Component Test

```javascript
// src/context/ModalContext.test.js
import { render, act } from '@testing-library/react'
import { ModalProvider, useModal } from './ModalContext'

const TestComponent = ({ onMount }) => {
  const modalContext = useModal()
  React.useEffect(() => { onMount?.(modalContext) }, [])
  return null
}

describe('ModalContext', () => {
  test('opens modal with content', () => {
    let context
    const { getByText } = render(
      <ModalProvider>
        <TestComponent onMount={(ctx) => { context = ctx }} />
      </ModalProvider>
    )

    act(() => {
      context.openModal(<div>Modal Content</div>)
    })

    expect(getByText('Modal Content')).toBeInTheDocument()
  })

  test('closes modal', () => {
    let context
    const { queryByText } = render(
      <ModalProvider>
        <TestComponent onMount={(ctx) => { context = ctx }} />
      </ModalProvider>
    )

    act(() => {
      context.openModal(<div>Modal Content</div>)
    })

    act(() => {
      context.closeModal()
    })

    expect(queryByText('Modal Content')).not.toBeInTheDocument()
  })
})
```

---

## Hook Testing Patterns

### Basic Hook Test

```javascript
// src/hooks/useWindowResize.test.js
import { renderHook, act } from '@testing-library/react'
import { useWindowResize } from './useWindowResize'

describe('useWindowResize', () => {
  test('returns initial window dimensions', () => {
    const { result } = renderHook(() => useWindowResize())
    expect(result.current.width).toBe(window.innerWidth)
    expect(result.current.height).toBe(window.innerHeight)
  })

  test('updates on resize', () => {
    const { result } = renderHook(() => useWindowResize())

    act(() => {
      window.innerWidth = 800
      window.innerHeight = 600
      window.dispatchEvent(new Event('resize'))
    })

    expect(result.current.width).toBe(800)
    expect(result.current.height).toBe(600)
  })
})
```

### Timer Testing with Fake Timers

```javascript
// src/hooks/useCopyToClipboard.test.js
describe('useCopyToClipboard', () => {
  beforeEach(() => {
    jest.clearAllMocks()
    jest.useFakeTimers()
  })

  afterEach(() => {
    jest.clearAllTimers()
    jest.useRealTimers()
  })

  test('clears clipboard after timeout', async () => {
    const mockClipboard = {
      writeText: jest.fn(),
      readText: jest.fn().mockResolvedValue('password')
    }
    Object.assign(navigator, { clipboard: mockClipboard })

    const { result } = renderHook(() => useCopyToClipboard())

    await act(async () => {
      await result.current.copy('password')
    })

    // Fast-forward timeout
    await act(async () => {
      jest.advanceTimersByTime(30000)
    })

    expect(mockClipboard.writeText).toHaveBeenCalledWith('')
  })

  test('does not clear if clipboard changed', async () => {
    const mockClipboard = {
      writeText: jest.fn(),
      readText: jest.fn().mockResolvedValue('different value')
    }
    Object.assign(navigator, { clipboard: mockClipboard })

    const { result } = renderHook(() => useCopyToClipboard())

    await act(async () => {
      await result.current.copy('original')
    })

    await act(async () => {
      jest.advanceTimersByTime(30000)
    })

    // Should NOT have cleared clipboard
    expect(mockClipboard.writeText).not.toHaveBeenCalledWith('')
  })

  test('cancels timer on unmount', async () => {
    const mockClipboard = {
      writeText: jest.fn(),
      readText: jest.fn()
    }
    Object.assign(navigator, { clipboard: mockClipboard })

    const { result, unmount } = renderHook(() => useCopyToClipboard())

    await act(async () => {
      await result.current.copy('password')
    })

    unmount()

    await act(async () => {
      jest.advanceTimersByTime(30000)
    })

    // readText should not be called after unmount
    expect(mockClipboard.readText).not.toHaveBeenCalled()
  })
})
```

---

## Playwright E2E Tests

### Basic Flow

```typescript
// e2e/login.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Login Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })

  test('user can unlock vault with password', async ({ page }) => {
    await page.getByPlaceholder('Enter password').fill('TestPassword123!')
    await page.getByRole('button', { name: 'Unlock' }).click()

    await expect(page.getByText('Your Vault')).toBeVisible()
  })

  test('shows error for wrong password', async ({ page }) => {
    await page.getByPlaceholder('Enter password').fill('wrongpassword')
    await page.getByRole('button', { name: 'Unlock' }).click()

    await expect(page.getByText('Invalid password')).toBeVisible()
  })
})
```

### Record Management Flow

```typescript
// e2e/records.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Record Management', () => {
  test.beforeEach(async ({ page }) => {
    // Login first
    await page.goto('/')
    await page.getByPlaceholder('Enter password').fill('TestPassword123!')
    await page.getByRole('button', { name: 'Unlock' }).click()
    await expect(page.getByText('Your Vault')).toBeVisible()
  })

  test('user can create a login record', async ({ page }) => {
    await page.getByRole('button', { name: 'Create' }).click()
    await page.getByRole('button', { name: 'Login' }).click()

    await page.getByPlaceholder('Title').fill('Test Login')
    await page.getByPlaceholder('Username').fill('testuser')
    await page.getByPlaceholder('Password').fill('testpass123')
    await page.getByPlaceholder('URL').fill('https://example.com')

    await page.getByRole('button', { name: 'Save' }).click()

    await expect(page.getByText('Test Login')).toBeVisible()
  })

  test('user can search records', async ({ page }) => {
    await page.getByPlaceholder('Search').fill('Test')

    await expect(page.getByText('Test Login')).toBeVisible()
  })

  test('user can copy password', async ({ page }) => {
    await page.getByText('Test Login').click()
    await page.getByRole('button', { name: 'Copy password' }).click()

    await expect(page.getByText('Copied')).toBeVisible()
  })
})
```

### Playwright Selectors

```javascript
// By role (preferred)
page.getByRole('button', { name: 'Submit' })
page.getByRole('textbox', { name: 'Username' })
page.getByRole('checkbox', { name: 'Remember me' })
page.getByRole('link', { name: 'Settings' })

// By placeholder
page.getByPlaceholder('Enter password')

// By text
page.getByText('Welcome to PearPass')

// By test ID
page.getByTestId('record-list')

// By label
page.getByLabel('Password')
```

---

## Mock Patterns

### Vault Context Mock

```javascript
jest.mock('pearpass-lib-vault', () => ({
  VaultProvider: ({ children }) => children,
  useVault: jest.fn(() => ({
    data: { isLocked: false },
    lock: jest.fn(),
    unlock: jest.fn()
  })),
  useRecords: jest.fn(() => ({
    data: [
      { id: '1', title: 'Test Record', type: 'login' }
    ],
    loading: false,
    createRecord: jest.fn(),
    updateRecord: jest.fn(),
    deleteRecord: jest.fn()
  }))
}))
```

### Router Context Mock

```javascript
const mockNavigate = jest.fn()
jest.mock('../context/RouterContext', () => ({
  useRouter: () => ({
    navigate: mockNavigate,
    currentRoute: 'main'
  })
}))
```

### Toast Context Mock

```javascript
const mockShowToast = jest.fn()
jest.mock('../context/ToastContext', () => ({
  useToast: () => ({
    showToast: mockShowToast
  })
}))
```

### Modal Context Mock

```javascript
const mockOpenModal = jest.fn()
const mockCloseModal = jest.fn()
jest.mock('../context/ModalContext', () => ({
  useModal: () => ({
    openModal: mockOpenModal,
    closeModal: mockCloseModal,
    isOpen: false
  })
}))
```

---

## Provider Wrapper Pattern

```javascript
import { ThemeProvider } from 'pearpass-lib-ui-theme-provider'
import { VaultProvider } from 'pearpass-lib-vault'
import { ToastProvider } from '../context/ToastContext'
import { RouterProvider } from '../context/RouterContext'
import { ModalProvider } from '../context/ModalContext'

const AllProviders = ({ children }) => (
  <ThemeProvider>
    <VaultProvider>
      <ToastProvider>
        <RouterProvider>
          <ModalProvider>
            {children}
          </ModalProvider>
        </RouterProvider>
      </ToastProvider>
    </VaultProvider>
  </ThemeProvider>
)

const renderWithProviders = (ui) =>
  render(ui, { wrapper: AllProviders })
```

---

## Test Writing Guidelines

1. **Wrap with ThemeProvider** - All UI components use styled-components theme
2. **Use renderHook for hooks** - Not render() for hook tests
3. **Mock contexts completely** - Vault, Router, Toast, Modal
4. **Use act() for state updates** - Wrap async operations
5. **Use waitFor() for async assertions** - Wait for effects
6. **Use fake timers for timeouts** - `jest.useFakeTimers()`
7. **Clear mocks in beforeEach** - `jest.clearAllMocks()`
8. **Test error cases** - Not just happy paths
9. **Use Playwright for E2E** - Full user journey tests

## Existing Test Examples

Reference these for patterns:
- `src/hooks/useCopyToClipboard.test.js` - Timer testing
- `src/context/ModalContext.test.js` - Context testing
- `src/components/ButtonPrimary/index.test.js` - Component testing
- `e2e/*.spec.ts` - Playwright E2E tests
