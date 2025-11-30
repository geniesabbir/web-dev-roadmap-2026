# Day 9: Custom Hooks - Reusable Stateful Logic

## Introduction

Custom hooks are the ultimate way to reuse stateful logic in React. They let you extract component logic into reusable functions that can be shared across your entire application. If you find yourself copying and pasting the same useState + useEffect combo, it's time for a custom hook!

## Learning Objectives

By the end of this lesson, you will:
- Understand what custom hooks are and why they're powerful
- Know the rules and naming conventions for custom hooks
- Create custom hooks for common use cases
- Learn to compose hooks together
- Build a library of reusable hooks

---

## What Is a Custom Hook?

A custom hook is a JavaScript function that:
1. **Starts with "use"** (e.g., `useCounter`, `useFetch`, `useLocalStorage`)
2. **Can call other hooks** (useState, useEffect, other custom hooks)
3. **Returns whatever you want** (values, functions, arrays, objects)

```jsx
// A simple custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// Using the custom hook
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

---

## Rules for Custom Hooks

### 1. Name Must Start with "use"

```jsx
// ✅ Correct
function useWindowSize() { ... }
function useDarkMode() { ... }
function useFormValidation() { ... }

// ❌ Wrong - React won't recognize these as hooks
function getWindowSize() { ... }
function windowSizeHook() { ... }
function withDarkMode() { ... }
```

### 2. Follow the Rules of Hooks

Custom hooks must follow all hook rules:
- Only call hooks at the top level
- Only call hooks from React functions (components or other hooks)

```jsx
function useData(id) {
  // ✅ Hooks at top level
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchData(id).then(setData);
    setLoading(false);
  }, [id]);

  // ❌ WRONG - hook inside condition
  // if (data) {
  //   const [extra, setExtra] = useState(null);
  // }

  return { data, loading };
}
```

### 3. Custom Hooks Are Isolated

Each component using a custom hook gets its own isolated state:

```jsx
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, increment: () => setCount(c => c + 1) };
}

function App() {
  // These are completely independent!
  const counter1 = useCounter(); // Has its own count state
  const counter2 = useCounter(); // Has its own count state

  return (
    <div>
      <p>Counter 1: {counter1.count}</p>
      <p>Counter 2: {counter2.count}</p>
      <button onClick={counter1.increment}>+1 to Counter 1</button>
      <button onClick={counter2.increment}>+1 to Counter 2</button>
    </div>
  );
}
```

---

## Essential Custom Hooks

### useToggle

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);

  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
}

// Usage
function Modal() {
  const { value: isOpen, toggle, setFalse: close } = useToggle();

  return (
    <>
      <button onClick={toggle}>Toggle Modal</button>
      {isOpen && (
        <div className="modal">
          <p>Modal Content</p>
          <button onClick={close}>Close</button>
        </div>
      )}
    </>
  );
}
```

### useLocalStorage

```jsx
function useLocalStorage(key, initialValue) {
  // Initialize state from localStorage or use initial value
  const [value, setValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Update localStorage when value changes
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, value]);

  // Remove from localStorage
  const remove = useCallback(() => {
    localStorage.removeItem(key);
    setValue(initialValue);
  }, [key, initialValue]);

  return [value, setValue, remove];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);

  return (
    <div>
      <select value={theme} onChange={e => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>

      <input
        type="range"
        min="12"
        max="24"
        value={fontSize}
        onChange={e => setFontSize(Number(e.target.value))}
      />
      <span>{fontSize}px</span>
    </div>
  );
}
```

### useFetch

```jsx
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) {
      setLoading(false);
      return;
    }

    const abortController = new AbortController();

    async function fetchData() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, {
          ...options,
          signal: abortController.signal
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    return () => abortController.abort();
  }, [url]); // Note: options excluded for simplicity

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(
    `/api/users/${userId}`
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### useDebounce

```jsx
function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timeoutId);
  }, [value, delay]);

  return debouncedValue;
}

// Usage - Search with debounce
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  const { data: results, loading } = useFetch(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null
  );

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {loading && <p>Searching...</p>}
      {results && (
        <ul>
          {results.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### useWindowSize

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);

    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
function ResponsiveComponent() {
  const { width } = useWindowSize();

  if (width < 768) {
    return <MobileLayout />;
  }

  return <DesktopLayout />;
}
```

### usePrevious

```jsx
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// Usage - Detect direction of change
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  const direction = count > prevCount ? '📈' : count < prevCount ? '📉' : '';

  return (
    <div>
      <p>Count: {count} {direction}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <button onClick={() => setCount(c => c - 1)}>-</button>
    </div>
  );
}
```

---

## Form Hooks

### useInput

```jsx
function useInput(initialValue = '') {
  const [value, setValue] = useState(initialValue);

  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);

  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue]);

  // Return props that can be spread onto an input
  return {
    value,
    onChange,
    reset,
    bind: { value, onChange } // Convenient for spreading
  };
}

// Usage
function SignupForm() {
  const username = useInput('');
  const email = useInput('');
  const password = useInput('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      username: username.value,
      email: email.value,
      password: password.value
    });

    // Reset all fields
    username.reset();
    email.reset();
    password.reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Username" {...username.bind} />
      <input placeholder="Email" type="email" {...email.bind} />
      <input placeholder="Password" type="password" {...password.bind} />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

### useForm (Advanced)

```jsx
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Handle input change
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  }, []);

  // Handle input blur (mark as touched)
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  }, []);

  // Validate on values change
  useEffect(() => {
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
    }
  }, [values, validate]);

  // Get field props
  const getFieldProps = useCallback((name) => ({
    name,
    value: values[name] || '',
    onChange: handleChange,
    onBlur: handleBlur
  }), [values, handleChange, handleBlur]);

  // Check if form is valid
  const isValid = Object.keys(errors).length === 0;

  // Reset form
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  // Handle submit
  const handleSubmit = useCallback((onSubmit) => async (e) => {
    e.preventDefault();

    // Touch all fields
    const allTouched = Object.keys(values).reduce(
      (acc, key) => ({ ...acc, [key]: true }),
      {}
    );
    setTouched(allTouched);

    // Validate
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);

      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }

    setIsSubmitting(true);

    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate]);

  return {
    values,
    errors,
    touched,
    isValid,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    getFieldProps,
    reset,
    setValues
  };
}

// Usage
function RegistrationForm() {
  const validate = (values) => {
    const errors = {};

    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = 'Invalid email format';
    }

    if (!values.password) {
      errors.password = 'Password is required';
    } else if (values.password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }

    if (values.password !== values.confirmPassword) {
      errors.confirmPassword = 'Passwords must match';
    }

    return errors;
  };

  const form = useForm(
    { email: '', password: '', confirmPassword: '' },
    validate
  );

  const onSubmit = async (values) => {
    console.log('Submitting:', values);
    await new Promise(r => setTimeout(r, 1000));
    alert('Registered!');
    form.reset();
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <div>
        <input
          type="email"
          placeholder="Email"
          {...form.getFieldProps('email')}
        />
        {form.touched.email && form.errors.email && (
          <span className="error">{form.errors.email}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          placeholder="Password"
          {...form.getFieldProps('password')}
        />
        {form.touched.password && form.errors.password && (
          <span className="error">{form.errors.password}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          placeholder="Confirm Password"
          {...form.getFieldProps('confirmPassword')}
        />
        {form.touched.confirmPassword && form.errors.confirmPassword && (
          <span className="error">{form.errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

---

## Composing Hooks

Custom hooks can use other custom hooks:

```jsx
// Base hooks
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

function useMediaQuery(query) {
  const [matches, setMatches] = useState(
    () => window.matchMedia(query).matches
  );

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    const handler = (e) => setMatches(e.matches);

    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);

  return matches;
}

// Composed hook using both
function useDarkMode() {
  // Check system preference
  const prefersDark = useMediaQuery('(prefers-color-scheme: dark)');

  // Store user preference (null = follow system)
  const [userPreference, setUserPreference] = useLocalStorage(
    'darkMode',
    null
  );

  // Determine actual mode
  const isDark = userPreference ?? prefersDark;

  // Apply to document
  useEffect(() => {
    document.documentElement.classList.toggle('dark', isDark);
  }, [isDark]);

  const toggle = () => setUserPreference(prev => !prev);
  const setDark = () => setUserPreference(true);
  const setLight = () => setUserPreference(false);
  const setSystem = () => setUserPreference(null);

  return {
    isDark,
    toggle,
    setDark,
    setLight,
    setSystem,
    isSystem: userPreference === null
  };
}

// Usage
function ThemeToggle() {
  const { isDark, toggle, setSystem, isSystem } = useDarkMode();

  return (
    <div>
      <button onClick={toggle}>
        {isDark ? '☀️ Light' : '🌙 Dark'}
      </button>
      <button onClick={setSystem} disabled={isSystem}>
        Use System Theme
      </button>
    </div>
  );
}
```

---

## Async Hooks Pattern

### useAsync

```jsx
function useAsync(asyncFunction, immediate = true) {
  const [state, setState] = useState({
    data: null,
    loading: immediate,
    error: null
  });

  const execute = useCallback(async (...args) => {
    setState({ data: null, loading: true, error: null });

    try {
      const data = await asyncFunction(...args);
      setState({ data, loading: false, error: null });
      return data;
    } catch (error) {
      setState({ data: null, loading: false, error });
      throw error;
    }
  }, [asyncFunction]);

  // Execute immediately if requested
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { ...state, execute };
}

// Usage
function UserProfile({ userId }) {
  const fetchUser = useCallback(
    () => fetch(`/api/users/${userId}`).then(r => r.json()),
    [userId]
  );

  const { data: user, loading, error, execute: refetch } = useAsync(fetchUser);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}
```

### useMutation

```jsx
function useMutation(mutationFn) {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null
  });

  const mutate = useCallback(async (...args) => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const data = await mutationFn(...args);
      setState({ data, loading: false, error: null });
      return data;
    } catch (error) {
      setState(prev => ({ ...prev, loading: false, error }));
      throw error;
    }
  }, [mutationFn]);

  const reset = useCallback(() => {
    setState({ data: null, loading: false, error: null });
  }, []);

  return { ...state, mutate, reset };
}

// Usage
function CreatePost() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const createPost = useCallback(
    (data) => fetch('/api/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    }).then(r => r.json()),
    []
  );

  const { mutate, loading, error, data } = useMutation(createPost);

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await mutate({ title, content });
      setTitle('');
      setContent('');
    } catch (err) {
      // Error handled by hook
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={content}
        onChange={e => setContent(e.target.value)}
        placeholder="Content"
      />
      <button disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      {error && <p className="error">{error.message}</p>}
      {data && <p className="success">Post created!</p>}
    </form>
  );
}
```

---

## Utility Hooks

### useClickOutside

```jsx
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useClickOutside(dropdownRef, () => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>
        Toggle Dropdown
      </button>
      {isOpen && (
        <ul className="dropdown-menu">
          <li>Option 1</li>
          <li>Option 2</li>
          <li>Option 3</li>
        </ul>
      )}
    </div>
  );
}
```

### useKeyPress

```jsx
function useKeyPress(targetKey) {
  const [isPressed, setIsPressed] = useState(false);

  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === targetKey) {
        setIsPressed(true);
      }
    };

    const handleKeyUp = (e) => {
      if (e.key === targetKey) {
        setIsPressed(false);
      }
    };

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, [targetKey]);

  return isPressed;
}

// Usage
function Game() {
  const upPressed = useKeyPress('ArrowUp');
  const downPressed = useKeyPress('ArrowDown');
  const spacePressed = useKeyPress(' ');

  return (
    <div>
      <p>Up: {upPressed ? '⬆️' : '○'}</p>
      <p>Down: {downPressed ? '⬇️' : '○'}</p>
      <p>Space: {spacePressed ? '🚀' : '○'}</p>
    </div>
  );
}
```

### useInterval

```jsx
function useInterval(callback, delay) {
  const savedCallback = useRef();

  // Remember the latest callback
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Set up the interval
  useEffect(() => {
    if (delay === null) return;

    const id = setInterval(() => {
      savedCallback.current();
    }, delay);

    return () => clearInterval(id);
  }, [delay]);
}

// Usage
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(true);

  useInterval(
    () => setCount(c => c + 1),
    isRunning ? 1000 : null // null stops the interval
  );

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Resume'}
      </button>
    </div>
  );
}
```

### useCopyToClipboard

```jsx
function useCopyToClipboard() {
  const [copiedText, setCopiedText] = useState(null);
  const [error, setError] = useState(null);

  const copy = useCallback(async (text) => {
    if (!navigator?.clipboard) {
      setError(new Error('Clipboard not supported'));
      return false;
    }

    try {
      await navigator.clipboard.writeText(text);
      setCopiedText(text);
      setError(null);
      return true;
    } catch (err) {
      setError(err);
      setCopiedText(null);
      return false;
    }
  }, []);

  return { copy, copiedText, error };
}

// Usage
function ShareButton({ url }) {
  const { copy, copiedText } = useCopyToClipboard();

  return (
    <button onClick={() => copy(url)}>
      {copiedText === url ? '✅ Copied!' : '📋 Copy Link'}
    </button>
  );
}
```

---

## Exercises

### Exercise 1: useHover

Create a hook that tracks whether an element is being hovered:

```jsx
function useHover() {
  // TODO: Return ref and isHovered
  // The ref should be attached to the element you want to track
}

// Expected usage:
function HoverCard() {
  const { ref, isHovered } = useHover();

  return (
    <div ref={ref} className={isHovered ? 'hovered' : ''}>
      {isHovered ? 'Hovering!' : 'Hover over me'}
    </div>
  );
}
```

### Exercise 2: useDocumentTitle

Create a hook that updates the document title:

```jsx
function useDocumentTitle(title) {
  // TODO: Update document.title
  // TODO: Restore original title on unmount
}

// Expected usage:
function Page() {
  useDocumentTitle('My Page Title');
  return <div>...</div>;
}
```

### Exercise 3: useArray

Create a hook for common array operations:

```jsx
function useArray(initialValue = []) {
  // TODO: Implement push, remove, filter, update, clear
}

// Expected usage:
function TodoList() {
  const { value: todos, push, remove, update, clear } = useArray([]);

  // push({ id: 1, text: 'New todo' })
  // remove(0) - removes at index
  // update(0, { ...todos[0], completed: true })
  // clear()
}
```

---

## Solutions

### Solution 1: useHover

```jsx
function useHover() {
  const [isHovered, setIsHovered] = useState(false);
  const ref = useRef(null);

  useEffect(() => {
    const node = ref.current;
    if (!node) return;

    const handleMouseEnter = () => setIsHovered(true);
    const handleMouseLeave = () => setIsHovered(false);

    node.addEventListener('mouseenter', handleMouseEnter);
    node.addEventListener('mouseleave', handleMouseLeave);

    return () => {
      node.removeEventListener('mouseenter', handleMouseEnter);
      node.removeEventListener('mouseleave', handleMouseLeave);
    };
  }, []);

  return { ref, isHovered };
}

// Usage
function HoverCard() {
  const { ref, isHovered } = useHover();

  return (
    <div
      ref={ref}
      style={{
        padding: '40px',
        background: isHovered ? '#4CAF50' : '#2196F3',
        color: 'white',
        transition: 'background 0.3s',
        cursor: 'pointer'
      }}
    >
      {isHovered ? '🎉 Hovering!' : '👋 Hover over me'}
    </div>
  );
}
```

### Solution 2: useDocumentTitle

```jsx
function useDocumentTitle(title) {
  const prevTitleRef = useRef(document.title);

  useEffect(() => {
    document.title = title;
  }, [title]);

  // Restore on unmount
  useEffect(() => {
    const prevTitle = prevTitleRef.current;

    return () => {
      document.title = prevTitle;
    };
  }, []);
}

// Usage
function ProductPage({ product }) {
  useDocumentTitle(`${product.name} | My Store`);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}

// Dynamically updating title
function NotificationsPage() {
  const [unreadCount, setUnreadCount] = useState(0);

  useDocumentTitle(
    unreadCount > 0
      ? `(${unreadCount}) Notifications`
      : 'Notifications'
  );

  return <div>...</div>;
}
```

### Solution 3: useArray

```jsx
function useArray(initialValue = []) {
  const [value, setValue] = useState(initialValue);

  const push = useCallback((element) => {
    setValue(arr => [...arr, element]);
  }, []);

  const remove = useCallback((index) => {
    setValue(arr => arr.filter((_, i) => i !== index));
  }, []);

  const update = useCallback((index, newElement) => {
    setValue(arr => arr.map((el, i) => (i === index ? newElement : el)));
  }, []);

  const filter = useCallback((callback) => {
    setValue(arr => arr.filter(callback));
  }, []);

  const clear = useCallback(() => {
    setValue([]);
  }, []);

  const isEmpty = value.length === 0;

  return {
    value,
    setValue,
    push,
    remove,
    update,
    filter,
    clear,
    isEmpty
  };
}

// Usage - Complete Todo App
function TodoApp() {
  const {
    value: todos,
    push,
    remove,
    update,
    filter,
    clear,
    isEmpty
  } = useArray([]);

  const [input, setInput] = useState('');

  const addTodo = (e) => {
    e.preventDefault();
    if (!input.trim()) return;

    push({
      id: Date.now(),
      text: input,
      completed: false
    });
    setInput('');
  };

  const toggleTodo = (index) => {
    update(index, {
      ...todos[index],
      completed: !todos[index].completed
    });
  };

  const removeCompleted = () => {
    filter(todo => !todo.completed);
  };

  return (
    <div>
      <form onSubmit={addTodo}>
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          placeholder="Add todo..."
        />
        <button type="submit">Add</button>
      </form>

      {isEmpty ? (
        <p>No todos yet!</p>
      ) : (
        <ul>
          {todos.map((todo, index) => (
            <li key={todo.id}>
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => toggleTodo(index)}
              />
              <span style={{
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}>
                {todo.text}
              </span>
              <button onClick={() => remove(index)}>×</button>
            </li>
          ))}
        </ul>
      )}

      <div>
        <button onClick={removeCompleted}>Remove Completed</button>
        <button onClick={clear}>Clear All</button>
      </div>
    </div>
  );
}
```

---

## Best Practices

### 1. Keep Hooks Focused

```jsx
// ❌ Too many responsibilities
function useEverything() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  // ... 500 lines of code
}

// ✅ Single responsibility
function useUser() { ... }
function usePosts(userId) { ... }
function useTheme() { ... }
function useNotifications(userId) { ... }
```

### 2. Return Consistent Types

```jsx
// ✅ Consistent return shape
function useFetch(url) {
  return { data, loading, error }; // Always an object
}

function useToggle() {
  return [value, toggle]; // Always an array
}
```

### 3. Document Your Hooks

```jsx
/**
 * Hook to manage a boolean toggle state
 * @param {boolean} initialValue - Initial toggle state
 * @returns {{ value: boolean, toggle: () => void, setTrue: () => void, setFalse: () => void }}
 */
function useToggle(initialValue = false) {
  // ...
}
```

---

## Key Takeaways

1. **Custom hooks start with "use"** and can call other hooks
2. **Each component gets its own state** when using a custom hook
3. **Compose hooks** to build complex functionality from simple pieces
4. **Extract repeated logic** into custom hooks for reusability
5. **Keep hooks focused** on a single responsibility
6. **Use TypeScript** for better documentation and autocomplete
7. **Test custom hooks** separately from components

---

## What's Next?

Tomorrow we'll put everything together in a **Mini Project** - building a complete React application that uses components, state, effects, and custom hooks!
