# Frontend Testing

**Duration:** 1 week

## Learning Objectives

By the end of this section, you will:
- Understand different types of tests
- Write unit tests with Jest
- Test React components with Testing Library
- Write E2E tests with Playwright
- Know what to test and what not to test

---

## Testing Philosophy

### The Testing Trophy
```
        ╱╲
       ╱  ╲      E2E Tests (few)
      ╱────╲
     ╱      ╲    Integration Tests (more)
    ╱────────╲
   ╱          ╲  Unit Tests (many, but not too many)
  ╱────────────╲
 ╱              ╲ Static Analysis (TypeScript, ESLint)
╱────────────────╲
```

### What to Test
- User interactions
- Business logic
- Edge cases
- Error states

### What NOT to Test
- Implementation details
- Third-party libraries
- CSS styles
- Constant values

---

## Day 1: Testing Setup

### Jest + React Testing Library Setup

```bash
npm install -D jest @types/jest ts-jest
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
npm install -D jest-environment-jsdom
```

**jest.config.js:**
```js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  transform: {
    '^.+\\.(ts|tsx)$': 'ts-jest',
  },
}
```

**jest.setup.ts:**
```ts
import '@testing-library/jest-dom'
```

**package.json:**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

---

## Day 2: Unit Testing

### Testing Utility Functions

```ts
// utils/formatters.ts
export function formatPrice(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`
}

export function truncate(str: string, maxLength: number): string {
  if (str.length <= maxLength) return str
  return str.slice(0, maxLength) + '...'
}

// utils/formatters.test.ts
import { formatPrice, truncate } from './formatters'

describe('formatPrice', () => {
  it('formats cents to dollars', () => {
    expect(formatPrice(1000)).toBe('$10.00')
    expect(formatPrice(1234)).toBe('$12.34')
    expect(formatPrice(99)).toBe('$0.99')
  })

  it('handles zero', () => {
    expect(formatPrice(0)).toBe('$0.00')
  })
})

describe('truncate', () => {
  it('returns original string if shorter than max', () => {
    expect(truncate('hello', 10)).toBe('hello')
  })

  it('truncates long strings with ellipsis', () => {
    expect(truncate('hello world', 5)).toBe('hello...')
  })

  it('handles exact length', () => {
    expect(truncate('hello', 5)).toBe('hello')
  })
})
```

### Testing Custom Hooks

```ts
// hooks/useCounter.ts
import { useState, useCallback } from 'react'

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial)

  const increment = useCallback(() => setCount((c) => c + 1), [])
  const decrement = useCallback(() => setCount((c) => c - 1), [])
  const reset = useCallback(() => setCount(initial), [initial])

  return { count, increment, decrement, reset }
}

// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('starts with initial value', () => {
    const { result } = renderHook(() => useCounter(10))
    expect(result.current.count).toBe(10)
  })

  it('increments', () => {
    const { result } = renderHook(() => useCounter())

    act(() => {
      result.current.increment()
    })

    expect(result.current.count).toBe(1)
  })

  it('decrements', () => {
    const { result } = renderHook(() => useCounter(5))

    act(() => {
      result.current.decrement()
    })

    expect(result.current.count).toBe(4)
  })

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10))

    act(() => {
      result.current.increment()
      result.current.increment()
      result.current.reset()
    })

    expect(result.current.count).toBe(10)
  })
})
```

---

## Day 3: Component Testing

### Testing Principles

1. **Test behavior, not implementation**
2. **Query like a user would** (role, label, text)
3. **Interact like a user would** (click, type)
4. **Assert visible outcomes**

### Query Priority
```ts
// Best: Accessible to everyone
getByRole('button', { name: 'Submit' })
getByLabelText('Email')
getByPlaceholderText('Search...')
getByText('Welcome')

// Good: Semantic queries
getByAltText('Profile picture')
getByTitle('Close')

// Last resort: Test IDs
getByTestId('custom-element')
```

### Basic Component Test

```tsx
// components/Button.tsx
interface ButtonProps {
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({ children, onClick, disabled, variant = 'primary' }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// components/Button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './Button'

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument()
  })

  it('calls onClick when clicked', async () => {
    const user = userEvent.setup()
    const handleClick = jest.fn()

    render(<Button onClick={handleClick}>Click me</Button>)
    await user.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })

  it('does not call onClick when disabled', async () => {
    const user = userEvent.setup()
    const handleClick = jest.fn()

    render(<Button onClick={handleClick} disabled>Click me</Button>)
    await user.click(screen.getByRole('button'))

    expect(handleClick).not.toHaveBeenCalled()
  })
})
```

### Testing Forms

```tsx
// components/LoginForm.tsx
import { useState } from 'react'

interface Props {
  onSubmit: (data: { email: string; password: string }) => void
}

export function LoginForm({ onSubmit }: Props) {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState('')

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    if (!email || !password) {
      setError('All fields are required')
      return
    }
    onSubmit({ email, password })
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && <div role="alert">{error}</div>}
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </div>
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </div>
      <button type="submit">Login</button>
    </form>
  )
}

// components/LoginForm.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { LoginForm } from './LoginForm'

describe('LoginForm', () => {
  it('renders form fields', () => {
    render(<LoginForm onSubmit={jest.fn()} />)

    expect(screen.getByLabelText('Email')).toBeInTheDocument()
    expect(screen.getByLabelText('Password')).toBeInTheDocument()
    expect(screen.getByRole('button', { name: 'Login' })).toBeInTheDocument()
  })

  it('submits form data', async () => {
    const user = userEvent.setup()
    const handleSubmit = jest.fn()

    render(<LoginForm onSubmit={handleSubmit} />)

    await user.type(screen.getByLabelText('Email'), 'test@example.com')
    await user.type(screen.getByLabelText('Password'), 'password123')
    await user.click(screen.getByRole('button', { name: 'Login' }))

    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    })
  })

  it('shows error when fields are empty', async () => {
    const user = userEvent.setup()

    render(<LoginForm onSubmit={jest.fn()} />)
    await user.click(screen.getByRole('button', { name: 'Login' }))

    expect(screen.getByRole('alert')).toHaveTextContent('All fields are required')
  })
})
```

---

## Day 4: Integration Testing

### Testing with API Mocking

```tsx
// Using MSW (Mock Service Worker)
// npm install -D msw

// mocks/handlers.ts
import { rest } from 'msw'

export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' },
      ])
    )
  }),

  rest.post('/api/login', async (req, res, ctx) => {
    const { email, password } = await req.json()
    if (email === 'test@example.com' && password === 'password') {
      return res(ctx.json({ token: 'fake-token' }))
    }
    return res(ctx.status(401), ctx.json({ error: 'Invalid credentials' }))
  }),
]

// mocks/server.ts
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

export const server = setupServer(...handlers)

// jest.setup.ts
import { server } from './mocks/server'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### Testing Async Components

```tsx
// components/UserList.test.tsx
import { render, screen, waitFor } from '@testing-library/react'
import { UserList } from './UserList'

describe('UserList', () => {
  it('shows loading state initially', () => {
    render(<UserList />)
    expect(screen.getByText('Loading...')).toBeInTheDocument()
  })

  it('displays users after loading', async () => {
    render(<UserList />)

    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument()
    })

    expect(screen.getByText('Jane')).toBeInTheDocument()
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument()
  })

  it('shows error on fetch failure', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500))
      })
    )

    render(<UserList />)

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument()
    })
  })
})
```

---

## Day 5: E2E Testing with Playwright

### Setup

```bash
npm init playwright@latest
```

### Basic E2E Test

```ts
// e2e/login.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Login Flow', () => {
  test('successful login redirects to dashboard', async ({ page }) => {
    await page.goto('/login')

    await page.fill('input[name="email"]', 'user@example.com')
    await page.fill('input[name="password"]', 'password123')
    await page.click('button[type="submit"]')

    await expect(page).toHaveURL('/dashboard')
    await expect(page.locator('h1')).toHaveText('Welcome back!')
  })

  test('invalid credentials shows error', async ({ page }) => {
    await page.goto('/login')

    await page.fill('input[name="email"]', 'wrong@example.com')
    await page.fill('input[name="password"]', 'wrongpassword')
    await page.click('button[type="submit"]')

    await expect(page.locator('[role="alert"]')).toHaveText('Invalid credentials')
    await expect(page).toHaveURL('/login')
  })
})

test.describe('Shopping Cart', () => {
  test('add and remove items', async ({ page }) => {
    await page.goto('/products')

    // Add item
    await page.click('button:has-text("Add to Cart")').first()
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1')

    // Go to cart
    await page.click('a:has-text("Cart")')
    await expect(page.locator('.cart-item')).toHaveCount(1)

    // Remove item
    await page.click('button:has-text("Remove")')
    await expect(page.locator('.cart-item')).toHaveCount(0)
  })
})
```

### Page Object Model

```ts
// e2e/pages/LoginPage.ts
import { Page, Locator } from '@playwright/test'

export class LoginPage {
  readonly page: Page
  readonly emailInput: Locator
  readonly passwordInput: Locator
  readonly submitButton: Locator
  readonly errorAlert: Locator

  constructor(page: Page) {
    this.page = page
    this.emailInput = page.locator('input[name="email"]')
    this.passwordInput = page.locator('input[name="password"]')
    this.submitButton = page.locator('button[type="submit"]')
    this.errorAlert = page.locator('[role="alert"]')
  }

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email)
    await this.passwordInput.fill(password)
    await this.submitButton.click()
  }
}

// e2e/login.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from './pages/LoginPage'

test('login with POM', async ({ page }) => {
  const loginPage = new LoginPage(page)

  await loginPage.goto()
  await loginPage.login('user@example.com', 'password123')

  await expect(page).toHaveURL('/dashboard')
})
```

---

## Testing Checklist

### What to Test
- [ ] User interactions (clicks, inputs)
- [ ] Form submissions
- [ ] Conditional rendering
- [ ] Loading and error states
- [ ] Edge cases
- [ ] Accessibility

### What NOT to Test
- [ ] Implementation details
- [ ] Styles/CSS
- [ ] Third-party libraries
- [ ] Private methods
- [ ] Constants

---

## Resources

### Documentation
- [Testing Library](https://testing-library.com/)
- [Jest](https://jestjs.io/)
- [Playwright](https://playwright.dev/)

### Articles
- [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles)
- [Common Testing Mistakes](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)

---

## Checklist Before Moving On

- [ ] Can write unit tests for utilities
- [ ] Can test custom hooks
- [ ] Can test React components
- [ ] Know how to mock API calls
- [ ] Can write integration tests
- [ ] Can write E2E tests with Playwright
- [ ] Understand what to test and what not to test

---

**Phase 2 Complete!**

Before moving to Phase 3, ensure you:
- [ ] Completed all Phase 2 sections
- [ ] Built Task Management App
- [ ] Built E-commerce Product Page
- [ ] Built Dashboard Application
- [ ] Added tests to your projects

**Next:** [Phase 3: Backend Development](../../phase-3-backend/README.md)
