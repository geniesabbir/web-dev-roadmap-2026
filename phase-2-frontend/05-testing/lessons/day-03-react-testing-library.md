# Day 3: React Testing Library

## Introduction

React Testing Library (RTL) is the standard way to test React components. Unlike older tools like Enzyme, RTL encourages testing components the way users actually interact with them - not testing implementation details, but testing behavior.

## Learning Objectives

By the end of this lesson, you will:
- Understand the React Testing Library philosophy
- Master queries for finding elements
- Simulate user interactions
- Test async behavior in components
- Write accessible, maintainable tests

---

## The Testing Library Philosophy

### Guiding Principle

> "The more your tests resemble the way your software is used, the more confidence they can give you."

### What This Means

```tsx
// ❌ Bad - Testing implementation details
test('sets isLoading state to true', () => {
  const { result } = renderHook(() => useMyHook());
  expect(result.current.state.isLoading).toBe(true);
});

// ✅ Good - Testing what users see
test('shows loading spinner while fetching data', () => {
  render(<MyComponent />);
  expect(screen.getByRole('progressbar')).toBeInTheDocument();
});
```

### Key Principles

1. **Test behavior, not implementation**
2. **Query elements like users do** (by role, label, text)
3. **Avoid testing internal state**
4. **Tests should be resilient to refactors**

---

## Setup

### Installation

```bash
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### Configuration

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
  },
});
```

---

## Rendering Components

### Basic Rendering

```tsx
import { render, screen } from '@testing-library/react';
import { Button } from './Button';

test('renders button with text', () => {
  render(<Button>Click me</Button>);

  expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
});
```

### Render Return Value

```tsx
const { container, debug, rerender, unmount } = render(<MyComponent />);

// container: The DOM node containing your component
// debug: Print the DOM to console
// rerender: Render with new props
// unmount: Unmount the component
```

### Rendering with Props

```tsx
test('renders with different variants', () => {
  const { rerender } = render(<Button variant="primary">Primary</Button>);
  expect(screen.getByRole('button')).toHaveClass('btn-primary');

  rerender(<Button variant="secondary">Secondary</Button>);
  expect(screen.getByRole('button')).toHaveClass('btn-secondary');
});
```

### Rendering with Context/Providers

```tsx
// Custom render function
import { render } from '@testing-library/react';
import { ThemeProvider } from './ThemeContext';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function customRender(ui: React.ReactElement, options = {}) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        {ui}
      </ThemeProvider>
    </QueryClientProvider>,
    options
  );
}

export * from '@testing-library/react';
export { customRender as render };
```

---

## Queries

### Query Priority

Use queries in this order (most preferred first):

1. **Accessible to everyone**: `getByRole`, `getByLabelText`, `getByPlaceholderText`, `getByText`, `getByDisplayValue`
2. **Semantic queries**: `getByAltText`, `getByTitle`
3. **Test IDs**: `getByTestId` (last resort)

### Query Types

| Query Type | 0 Matches | 1 Match | >1 Matches | Async? |
|------------|-----------|---------|------------|--------|
| `getBy...` | Throw | Return | Throw | No |
| `queryBy...` | `null` | Return | Throw | No |
| `findBy...` | Throw | Return | Throw | Yes |
| `getAllBy...` | Throw | Array | Array | No |
| `queryAllBy...` | `[]` | Array | Array | No |
| `findAllBy...` | Throw | Array | Array | Yes |

### ByRole (Most Preferred)

```tsx
// Button
screen.getByRole('button', { name: 'Submit' });

// Link
screen.getByRole('link', { name: 'Home' });

// Heading
screen.getByRole('heading', { name: 'Welcome', level: 1 });

// Checkbox
screen.getByRole('checkbox', { name: 'Accept terms' });

// Textbox (input/textarea)
screen.getByRole('textbox', { name: 'Email' });

// Combobox (select)
screen.getByRole('combobox', { name: 'Country' });

// List and listitem
screen.getByRole('list');
screen.getAllByRole('listitem');

// Navigation
screen.getByRole('navigation');

// Dialog/Modal
screen.getByRole('dialog');

// With states
screen.getByRole('button', { name: 'Submit', disabled: true });
screen.getByRole('checkbox', { checked: true });
screen.getByRole('tab', { selected: true });
```

### ByLabelText (For Form Elements)

```tsx
// <label for="email">Email</label>
// <input id="email" />
screen.getByLabelText('Email');

// <label>
//   Email <input />
// </label>
screen.getByLabelText('Email');

// aria-labelledby
screen.getByLabelText('Email Address');

// With regex
screen.getByLabelText(/email/i);
```

### ByPlaceholderText

```tsx
// <input placeholder="Enter your name" />
screen.getByPlaceholderText('Enter your name');
screen.getByPlaceholderText(/enter your name/i);
```

### ByText

```tsx
// For non-interactive elements
screen.getByText('Hello World');
screen.getByText(/hello/i);

// Partial match
screen.getByText('Hello', { exact: false });

// Custom text matcher
screen.getByText((content, element) => {
  return element?.tagName === 'SPAN' && content.includes('Hello');
});
```

### ByAltText (For Images)

```tsx
// <img alt="User avatar" />
screen.getByAltText('User avatar');
screen.getByAltText(/avatar/i);
```

### ByTitle

```tsx
// <span title="Close">×</span>
screen.getByTitle('Close');
```

### ByTestId (Last Resort)

```tsx
// <div data-testid="custom-element">Content</div>
screen.getByTestId('custom-element');

// When to use:
// - Dynamic text that's hard to query
// - No accessible role or label available
// - Highly unique elements
```

### Finding Multiple Elements

```tsx
// Get all list items
const items = screen.getAllByRole('listitem');
expect(items).toHaveLength(5);

// Get all buttons
const buttons = screen.getAllByRole('button');
expect(buttons[0]).toHaveTextContent('Save');
```

### Using `within`

```tsx
import { within } from '@testing-library/react';

test('finds element within container', () => {
  render(
    <div>
      <nav>
        <button>Home</button>
      </nav>
      <main>
        <button>Submit</button>
      </main>
    </div>
  );

  const nav = screen.getByRole('navigation');
  const navButton = within(nav).getByRole('button');
  expect(navButton).toHaveTextContent('Home');
});
```

---

## User Interactions

### Using userEvent

```tsx
import userEvent from '@testing-library/user-event';

test('user interactions', async () => {
  const user = userEvent.setup();

  render(<MyForm />);

  // Click
  await user.click(screen.getByRole('button'));

  // Type
  await user.type(screen.getByRole('textbox'), 'Hello');

  // Clear and type
  await user.clear(screen.getByRole('textbox'));
  await user.type(screen.getByRole('textbox'), 'New text');

  // Select option
  await user.selectOptions(screen.getByRole('combobox'), 'option1');

  // Keyboard
  await user.keyboard('{Enter}');
  await user.keyboard('{Shift>}A{/Shift}'); // Shift+A

  // Tab navigation
  await user.tab();

  // Hover
  await user.hover(screen.getByRole('button'));
  await user.unhover(screen.getByRole('button'));
});
```

### Click Events

```tsx
test('handles click events', async () => {
  const user = userEvent.setup();
  const handleClick = vi.fn();

  render(<Button onClick={handleClick}>Click me</Button>);

  await user.click(screen.getByRole('button'));

  expect(handleClick).toHaveBeenCalledTimes(1);
});

test('double click', async () => {
  const user = userEvent.setup();
  const handleDoubleClick = vi.fn();

  render(<div onDoubleClick={handleDoubleClick}>Double click me</div>);

  await user.dblClick(screen.getByText('Double click me'));

  expect(handleDoubleClick).toHaveBeenCalled();
});
```

### Typing in Inputs

```tsx
test('typing in form fields', async () => {
  const user = userEvent.setup();

  render(<LoginForm />);

  const emailInput = screen.getByLabelText('Email');
  const passwordInput = screen.getByLabelText('Password');

  await user.type(emailInput, 'user@example.com');
  await user.type(passwordInput, 'password123');

  expect(emailInput).toHaveValue('user@example.com');
  expect(passwordInput).toHaveValue('password123');
});

test('clearing and retyping', async () => {
  const user = userEvent.setup();

  render(<input defaultValue="initial" />);

  const input = screen.getByRole('textbox');

  await user.clear(input);
  await user.type(input, 'new value');

  expect(input).toHaveValue('new value');
});
```

### Selecting Options

```tsx
test('selecting from dropdown', async () => {
  const user = userEvent.setup();

  render(
    <select>
      <option value="">Choose...</option>
      <option value="apple">Apple</option>
      <option value="banana">Banana</option>
    </select>
  );

  const select = screen.getByRole('combobox');

  await user.selectOptions(select, 'banana');

  expect(select).toHaveValue('banana');
});

test('multi-select', async () => {
  const user = userEvent.setup();

  render(
    <select multiple>
      <option value="apple">Apple</option>
      <option value="banana">Banana</option>
      <option value="cherry">Cherry</option>
    </select>
  );

  await user.selectOptions(
    screen.getByRole('listbox'),
    ['apple', 'cherry']
  );
});
```

### Checkbox and Radio

```tsx
test('toggling checkbox', async () => {
  const user = userEvent.setup();

  render(
    <label>
      <input type="checkbox" /> Accept terms
    </label>
  );

  const checkbox = screen.getByRole('checkbox');

  expect(checkbox).not.toBeChecked();

  await user.click(checkbox);
  expect(checkbox).toBeChecked();

  await user.click(checkbox);
  expect(checkbox).not.toBeChecked();
});

test('selecting radio button', async () => {
  const user = userEvent.setup();

  render(
    <div role="radiogroup">
      <label><input type="radio" name="color" value="red" /> Red</label>
      <label><input type="radio" name="color" value="blue" /> Blue</label>
    </div>
  );

  await user.click(screen.getByLabelText('Blue'));

  expect(screen.getByLabelText('Blue')).toBeChecked();
  expect(screen.getByLabelText('Red')).not.toBeChecked();
});
```

### Keyboard Events

```tsx
test('keyboard interactions', async () => {
  const user = userEvent.setup();

  render(<SearchInput />);

  const input = screen.getByRole('textbox');

  await user.type(input, 'search query');
  await user.keyboard('{Enter}');

  // Or individual keys
  await user.keyboard('{Escape}');
  await user.keyboard('{Tab}');
  await user.keyboard('{ArrowDown}');
});

test('keyboard shortcuts', async () => {
  const user = userEvent.setup();
  const handleSave = vi.fn();

  render(<Editor onSave={handleSave} />);

  await user.keyboard('{Control>}s{/Control}');

  expect(handleSave).toHaveBeenCalled();
});
```

---

## Async Testing

### waitFor

```tsx
import { waitFor } from '@testing-library/react';

test('waits for element to appear', async () => {
  render(<AsyncComponent />);

  await waitFor(() => {
    expect(screen.getByText('Loaded!')).toBeInTheDocument();
  });
});

test('waits for element to disappear', async () => {
  render(<LoadingComponent />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();

  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});
```

### findBy Queries

```tsx
test('finds async element', async () => {
  render(<AsyncComponent />);

  // findBy returns a promise, auto-waits
  const element = await screen.findByText('Loaded!');
  expect(element).toBeInTheDocument();

  // With timeout
  const slowElement = await screen.findByText('Slow content', {}, { timeout: 5000 });
});
```

### waitForElementToBeRemoved

```tsx
import { waitForElementToBeRemoved } from '@testing-library/react';

test('waits for loading to finish', async () => {
  render(<DataLoader />);

  await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

  expect(screen.getByText('Data loaded')).toBeInTheDocument();
});
```

### Testing Data Fetching

```tsx
// Component that fetches data
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return <div>Welcome, {user.name}!</div>;
}
```

```tsx
// Test with mocked API
import { vi } from 'vitest';
import * as api from './api';

vi.mock('./api');

test('displays user name after loading', async () => {
  vi.mocked(api.fetchUser).mockResolvedValue({ id: '1', name: 'John' });

  render(<UserProfile userId="1" />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();

  await waitFor(() => {
    expect(screen.getByText('Welcome, John!')).toBeInTheDocument();
  });
});

test('displays error message on failure', async () => {
  vi.mocked(api.fetchUser).mockRejectedValue(new Error('Network error'));

  render(<UserProfile userId="1" />);

  await waitFor(() => {
    expect(screen.getByText('Error: Network error')).toBeInTheDocument();
  });
});
```

---

## Testing Forms

### Complete Form Test

```tsx
function LoginForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    const newErrors: Record<string, string> = {};
    if (!email) newErrors.email = 'Email is required';
    if (!password) newErrors.password = 'Password is required';
    if (password && password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    onSubmit({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && <span id="email-error" role="alert">{errors.email}</span>}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={e => setPassword(e.target.value)}
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <span id="password-error" role="alert">{errors.password}</span>
        )}
      </div>

      <button type="submit">Sign In</button>
    </form>
  );
}
```

```tsx
describe('LoginForm', () => {
  const user = userEvent.setup();

  test('submits form with valid data', async () => {
    const handleSubmit = vi.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });

  test('shows error when email is empty', async () => {
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    expect(screen.getByRole('alert')).toHaveTextContent('Email is required');
  });

  test('shows error when password is too short', async () => {
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'short');
    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    expect(screen.getByRole('alert')).toHaveTextContent(
      'Password must be at least 8 characters'
    );
  });

  test('does not submit form with errors', async () => {
    const handleSubmit = vi.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    expect(handleSubmit).not.toHaveBeenCalled();
  });
});
```

---

## Testing with Context

```tsx
// ThemeContext.tsx
const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}>({ theme: 'light', toggleTheme: () => {} });

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeToggle() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  return (
    <button onClick={toggleTheme}>
      Current theme: {theme}
    </button>
  );
}
```

```tsx
// ThemeToggle.test.tsx
describe('ThemeToggle', () => {
  test('displays current theme', () => {
    render(
      <ThemeProvider>
        <ThemeToggle />
      </ThemeProvider>
    );

    expect(screen.getByRole('button')).toHaveTextContent('Current theme: light');
  });

  test('toggles theme on click', async () => {
    const user = userEvent.setup();

    render(
      <ThemeProvider>
        <ThemeToggle />
      </ThemeProvider>
    );

    await user.click(screen.getByRole('button'));

    expect(screen.getByRole('button')).toHaveTextContent('Current theme: dark');
  });
});
```

---

## Testing Hooks

```tsx
import { renderHook, act } from '@testing-library/react';

// Custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}
```

```tsx
// useCounter.test.tsx
describe('useCounter', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  test('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  test('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  test('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });
});
```

---

## Jest-DOM Matchers

```tsx
// All available matchers from @testing-library/jest-dom
describe('jest-dom matchers', () => {
  test('toBeInTheDocument', () => {
    render(<div>Hello</div>);
    expect(screen.getByText('Hello')).toBeInTheDocument();
  });

  test('toBeVisible', () => {
    render(<div style={{ display: 'none' }}>Hidden</div>);
    expect(screen.getByText('Hidden')).not.toBeVisible();
  });

  test('toBeDisabled / toBeEnabled', () => {
    render(<button disabled>Click</button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByRole('button')).not.toBeEnabled();
  });

  test('toHaveClass', () => {
    render(<div className="foo bar">Content</div>);
    expect(screen.getByText('Content')).toHaveClass('foo');
    expect(screen.getByText('Content')).toHaveClass('foo', 'bar');
  });

  test('toHaveStyle', () => {
    render(<div style={{ color: 'red' }}>Styled</div>);
    expect(screen.getByText('Styled')).toHaveStyle({ color: 'red' });
  });

  test('toHaveTextContent', () => {
    render(<div>Hello World</div>);
    expect(screen.getByText(/hello/i)).toHaveTextContent('Hello World');
    expect(screen.getByText(/hello/i)).toHaveTextContent(/world/i);
  });

  test('toHaveValue', () => {
    render(<input defaultValue="test" />);
    expect(screen.getByRole('textbox')).toHaveValue('test');
  });

  test('toHaveAttribute', () => {
    render(<a href="/home">Home</a>);
    expect(screen.getByRole('link')).toHaveAttribute('href', '/home');
  });

  test('toHaveFocus', () => {
    render(<input autoFocus />);
    expect(screen.getByRole('textbox')).toHaveFocus();
  });

  test('toBeChecked', () => {
    render(<input type="checkbox" defaultChecked />);
    expect(screen.getByRole('checkbox')).toBeChecked();
  });

  test('toBeRequired', () => {
    render(<input required />);
    expect(screen.getByRole('textbox')).toBeRequired();
  });

  test('toBeValid / toBeInvalid', () => {
    render(<input type="email" value="invalid" onChange={() => {}} />);
    expect(screen.getByRole('textbox')).toBeInvalid();
  });

  test('toHaveAccessibleName', () => {
    render(<button aria-label="Close dialog">×</button>);
    expect(screen.getByRole('button')).toHaveAccessibleName('Close dialog');
  });

  test('toHaveAccessibleDescription', () => {
    render(
      <>
        <button aria-describedby="desc">Submit</button>
        <p id="desc">Submits the form</p>
      </>
    );
    expect(screen.getByRole('button')).toHaveAccessibleDescription('Submits the form');
  });
});
```

---

## Debugging Tests

### Using `debug()`

```tsx
test('debugging example', () => {
  render(<MyComponent />);

  // Print entire DOM
  screen.debug();

  // Print specific element
  screen.debug(screen.getByRole('button'));

  // Increase output length
  screen.debug(undefined, 20000);
});
```

### Using `logRoles`

```tsx
import { logRoles } from '@testing-library/react';

test('log available roles', () => {
  const { container } = render(<MyComponent />);

  logRoles(container);
  // Outputs all roles in the component
});
```

### Testing Playground

```tsx
// Add to browser console or use the Chrome extension
// https://testing-playground.com

// Generates optimal queries for elements
screen.logTestingPlaygroundURL();
```

---

## Exercise: Test a Todo List Component

```tsx
// TodoList.tsx
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface Props {
  initialTodos?: Todo[];
}

export function TodoList({ initialTodos = [] }: Props) {
  const [todos, setTodos] = useState<Todo[]>(initialTodos);
  const [input, setInput] = useState('');
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos([...todos, { id: Date.now(), text: input, completed: false }]);
    setInput('');
  };

  const toggleTodo = (id: number) => {
    setTodos(todos.map(t =>
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
  };

  const deleteTodo = (id: number) => {
    setTodos(todos.filter(t => t.id !== id));
  };

  const filteredTodos = todos.filter(t => {
    if (filter === 'active') return !t.completed;
    if (filter === 'completed') return t.completed;
    return true;
  });

  const activeCount = todos.filter(t => !t.completed).length;

  return (
    <div>
      <h1>Todo List</h1>

      <div>
        <input
          type="text"
          value={input}
          onChange={e => setInput(e.target.value)}
          onKeyDown={e => e.key === 'Enter' && addTodo()}
          placeholder="What needs to be done?"
          aria-label="New todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
              aria-label={`Mark "${todo.text}" as ${todo.completed ? 'incomplete' : 'complete'}`}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              aria-label={`Delete "${todo.text}"`}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>

      {todos.length > 0 && (
        <div>
          <span>{activeCount} items left</span>
          <button onClick={() => setFilter('all')} aria-pressed={filter === 'all'}>
            All
          </button>
          <button onClick={() => setFilter('active')} aria-pressed={filter === 'active'}>
            Active
          </button>
          <button onClick={() => setFilter('completed')} aria-pressed={filter === 'completed'}>
            Completed
          </button>
        </div>
      )}
    </div>
  );
}
```

---

## Solution

```tsx
// TodoList.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoList } from './TodoList';

describe('TodoList', () => {
  const user = userEvent.setup();

  describe('rendering', () => {
    it('renders with heading', () => {
      render(<TodoList />);
      expect(screen.getByRole('heading', { name: 'Todo List' })).toBeInTheDocument();
    });

    it('renders input and add button', () => {
      render(<TodoList />);
      expect(screen.getByLabelText('New todo')).toBeInTheDocument();
      expect(screen.getByRole('button', { name: 'Add' })).toBeInTheDocument();
    });

    it('renders initial todos', () => {
      const todos = [
        { id: 1, text: 'Learn React', completed: false },
        { id: 2, text: 'Write tests', completed: true },
      ];
      render(<TodoList initialTodos={todos} />);

      expect(screen.getByText('Learn React')).toBeInTheDocument();
      expect(screen.getByText('Write tests')).toBeInTheDocument();
    });
  });

  describe('adding todos', () => {
    it('adds todo when clicking Add button', async () => {
      render(<TodoList />);

      await user.type(screen.getByLabelText('New todo'), 'New task');
      await user.click(screen.getByRole('button', { name: 'Add' }));

      expect(screen.getByText('New task')).toBeInTheDocument();
    });

    it('adds todo when pressing Enter', async () => {
      render(<TodoList />);

      await user.type(screen.getByLabelText('New todo'), 'New task{Enter}');

      expect(screen.getByText('New task')).toBeInTheDocument();
    });

    it('clears input after adding', async () => {
      render(<TodoList />);

      await user.type(screen.getByLabelText('New todo'), 'New task{Enter}');

      expect(screen.getByLabelText('New todo')).toHaveValue('');
    });

    it('does not add empty todos', async () => {
      render(<TodoList />);

      await user.click(screen.getByRole('button', { name: 'Add' }));

      expect(screen.queryByRole('listitem')).not.toBeInTheDocument();
    });

    it('does not add whitespace-only todos', async () => {
      render(<TodoList />);

      await user.type(screen.getByLabelText('New todo'), '   ');
      await user.click(screen.getByRole('button', { name: 'Add' }));

      expect(screen.queryByRole('listitem')).not.toBeInTheDocument();
    });
  });

  describe('toggling todos', () => {
    it('marks todo as complete', async () => {
      const todos = [{ id: 1, text: 'Test todo', completed: false }];
      render(<TodoList initialTodos={todos} />);

      const checkbox = screen.getByRole('checkbox', {
        name: /mark "Test todo" as complete/i
      });

      await user.click(checkbox);

      expect(checkbox).toBeChecked();
    });

    it('marks todo as incomplete', async () => {
      const todos = [{ id: 1, text: 'Test todo', completed: true }];
      render(<TodoList initialTodos={todos} />);

      const checkbox = screen.getByRole('checkbox', {
        name: /mark "Test todo" as incomplete/i
      });

      await user.click(checkbox);

      expect(checkbox).not.toBeChecked();
    });

    it('applies strikethrough to completed todos', async () => {
      const todos = [{ id: 1, text: 'Test todo', completed: false }];
      render(<TodoList initialTodos={todos} />);

      await user.click(screen.getByRole('checkbox'));

      expect(screen.getByText('Test todo')).toHaveStyle({
        textDecoration: 'line-through'
      });
    });
  });

  describe('deleting todos', () => {
    it('removes todo when delete button is clicked', async () => {
      const todos = [{ id: 1, text: 'Delete me', completed: false }];
      render(<TodoList initialTodos={todos} />);

      await user.click(screen.getByRole('button', { name: /delete "Delete me"/i }));

      expect(screen.queryByText('Delete me')).not.toBeInTheDocument();
    });
  });

  describe('filtering', () => {
    const todos = [
      { id: 1, text: 'Active task', completed: false },
      { id: 2, text: 'Completed task', completed: true },
    ];

    it('shows all todos by default', () => {
      render(<TodoList initialTodos={todos} />);

      expect(screen.getByText('Active task')).toBeInTheDocument();
      expect(screen.getByText('Completed task')).toBeInTheDocument();
    });

    it('filters to show only active todos', async () => {
      render(<TodoList initialTodos={todos} />);

      await user.click(screen.getByRole('button', { name: 'Active' }));

      expect(screen.getByText('Active task')).toBeInTheDocument();
      expect(screen.queryByText('Completed task')).not.toBeInTheDocument();
    });

    it('filters to show only completed todos', async () => {
      render(<TodoList initialTodos={todos} />);

      await user.click(screen.getByRole('button', { name: 'Completed' }));

      expect(screen.queryByText('Active task')).not.toBeInTheDocument();
      expect(screen.getByText('Completed task')).toBeInTheDocument();
    });

    it('shows all todos when All filter clicked', async () => {
      render(<TodoList initialTodos={todos} />);

      await user.click(screen.getByRole('button', { name: 'Completed' }));
      await user.click(screen.getByRole('button', { name: 'All' }));

      expect(screen.getByText('Active task')).toBeInTheDocument();
      expect(screen.getByText('Completed task')).toBeInTheDocument();
    });
  });

  describe('item count', () => {
    it('shows correct count of active items', () => {
      const todos = [
        { id: 1, text: 'Task 1', completed: false },
        { id: 2, text: 'Task 2', completed: false },
        { id: 3, text: 'Task 3', completed: true },
      ];
      render(<TodoList initialTodos={todos} />);

      expect(screen.getByText('2 items left')).toBeInTheDocument();
    });

    it('updates count when todo is toggled', async () => {
      const todos = [{ id: 1, text: 'Task 1', completed: false }];
      render(<TodoList initialTodos={todos} />);

      expect(screen.getByText('1 items left')).toBeInTheDocument();

      await user.click(screen.getByRole('checkbox'));

      expect(screen.getByText('0 items left')).toBeInTheDocument();
    });

    it('does not show footer when no todos', () => {
      render(<TodoList />);

      expect(screen.queryByText(/items left/)).not.toBeInTheDocument();
    });
  });
});
```

---

## Key Takeaways

1. **Query by accessibility** - Use `getByRole` and `getByLabelText` first
2. **Use userEvent over fireEvent** - More realistic interactions
3. **Test user behavior** - Not internal state or implementation
4. **Async operations need await** - Use `findBy` or `waitFor`
5. **Debug with screen.debug()** - See what RTL sees
6. **Custom render for providers** - Wrap with context as needed
7. **jest-dom matchers** - Make assertions more readable

---

## What's Next?

Tomorrow we'll learn about **Integration Tests** - testing how multiple components work together, including API mocking with MSW!
