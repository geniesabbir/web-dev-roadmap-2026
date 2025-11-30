# React Advanced Patterns

**Duration:** 2-3 weeks

## Learning Objectives

By the end of this section, you will:
- Create reusable custom hooks
- Manage global state with Context
- Handle complex state with useReducer
- Optimize performance effectively
- Implement advanced component patterns

---

## Week 1: Custom Hooks & Context

### Day 1: Custom Hooks Basics
**Topics:**
- Why custom hooks?
- Rules of hooks
- Extracting logic into hooks
- Naming conventions
- Returning values and functions

**Exercises:**
```tsx
// exercises/01-custom-hooks-basics/

// 1. useToggle - boolean toggle state
// 2. useCounter - counter with increment/decrement/reset
// 3. useInput - controlled input logic
// 4. usePrevious - track previous value
```

### Day 2: Custom Hooks for Data
**Topics:**
- useFetch hook
- useLocalStorage hook
- Handling loading and errors
- Generic hooks with TypeScript
- Composing hooks

**Exercises:**
```tsx
// exercises/02-custom-hooks-data/

// 1. useFetch<T> - generic data fetching
// 2. useLocalStorage<T> - persist state
// 3. useDebounce<T> - debounced value
// 4. useAsync - async operation handler
```

### Day 3: Custom Hooks for DOM
**Topics:**
- useEventListener
- useOnClickOutside
- useMediaQuery
- useIntersectionObserver
- useWindowSize

**Exercises:**
```tsx
// exercises/03-custom-hooks-dom/

// 1. useEventListener - attach event listeners
// 2. useOnClickOutside - detect outside clicks
// 3. useMediaQuery - responsive hooks
// 4. useScrollPosition - track scroll
```

### Day 4: Context API Basics
**Topics:**
- What is Context?
- createContext
- Provider and Consumer
- useContext hook
- When to use Context
- Context with TypeScript

**Exercises:**
```tsx
// exercises/04-context-basics/

// 1. Create a ThemeContext
// 2. Create a UserContext
// 3. Consume context in nested components
// 4. Type context properly
```

### Day 5: Context Patterns
**Topics:**
- Context + useReducer
- Splitting contexts
- Context performance considerations
- Context vs props vs state management
- Provider composition

**Exercises:**
```tsx
// exercises/05-context-patterns/

// 1. Auth context with login/logout
// 2. Shopping cart context
// 3. Notification system context
// 4. Multi-provider setup
```

---

## Week 2: useReducer & Performance

### Day 1: useReducer Basics
**Topics:**
- When useState isn't enough
- Reducer pattern
- Actions and action types
- useReducer syntax
- TypeScript with reducers

**Exercises:**
```tsx
// exercises/06-usereducer-basics/

// 1. Counter with useReducer
// 2. Todo list with useReducer
// 3. Form state with useReducer
// 4. Type actions with discriminated unions
```

### Day 2: useReducer Advanced
**Topics:**
- Complex state management
- Middleware patterns
- Immer for immutable updates
- useReducer + Context
- When to use vs Redux

**Exercises:**
```tsx
// exercises/07-usereducer-advanced/

// 1. Shopping cart with complex actions
// 2. Multi-step form wizard
// 3. Undo/redo functionality
// 4. useReducer + Context store
```

### Day 3: React.memo
**Topics:**
- Why components re-render
- React.memo for memoization
- When to use memo
- Custom comparison functions
- Debugging re-renders

**Exercises:**
```tsx
// exercises/08-memo/

// 1. Identify unnecessary re-renders
// 2. Optimize a list component
// 3. Implement custom memo comparison
// 4. Use React DevTools Profiler
```

### Day 4: useMemo & useCallback
**Topics:**
- useMemo for expensive calculations
- useCallback for stable references
- Dependency arrays
- When NOT to optimize
- Common mistakes

**Exercises:**
```tsx
// exercises/09-memoization/

// 1. Memoize filtered/sorted lists
// 2. Stable callbacks for child components
// 3. Memoize expensive computations
// 4. Audit and remove unnecessary memoization
```

### Day 5: Performance Patterns
**Topics:**
- Virtualization (react-window)
- Code splitting (lazy, Suspense)
- Optimizing context
- Batching updates
- Profiling and measuring

**Exercises:**
```tsx
// exercises/10-performance/

// 1. Implement virtualized list
// 2. Add code splitting to routes
// 3. Optimize a slow component
// 4. Profile and document improvements
```

---

## Week 3: Advanced Patterns

### Day 1: Error Boundaries
**Topics:**
- What are error boundaries?
- Class component requirement
- Error boundary patterns
- Fallback UI
- Error recovery

**Exercises:**
```tsx
// exercises/11-error-boundaries/

// 1. Create a basic error boundary
// 2. Add error logging
// 3. Implement retry functionality
// 4. Granular error boundaries
```

### Day 2: Portals
**Topics:**
- What are portals?
- Use cases (modals, tooltips)
- Event bubbling with portals
- Accessibility considerations
- Portal patterns

**Exercises:**
```tsx
// exercises/12-portals/

// 1. Create a modal with portal
// 2. Build a tooltip system
// 3. Create a notification toast
// 4. Handle focus trapping
```

### Day 3: Compound Components
**Topics:**
- What are compound components?
- Implicit state sharing
- Context for compound components
- Flexible APIs
- Real-world examples

**Exercises:**
```tsx
// exercises/13-compound-components/

// 1. Build a Tabs compound component
// 2. Build an Accordion compound component
// 3. Build a Menu compound component
// 4. Add TypeScript for component children
```

### Day 4: Render Props & HOCs
**Topics:**
- Render props pattern
- Higher-order components
- When to use each
- Hooks vs these patterns
- Migration strategies

**Exercises:**
```tsx
// exercises/14-render-props-hocs/

// 1. Convert render prop to hook
// 2. Understand existing HOC code
// 3. Build a withAuth HOC
// 4. Compare patterns
```

### Day 5: Project Day
Build an advanced component library:

**Project: UI Component Library**
- Modal with portal
- Dropdown with compound pattern
- Toast notification system
- Tabs component
- Accordion component
- All fully typed
- All accessible

---

## Custom Hooks Library

Build these hooks for your projects:

```tsx
// hooks/useToggle.ts
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  return [value, toggle] as const;
}

// hooks/useDebounce.ts
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// hooks/useLocalStorage.ts
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    window.localStorage.setItem(key, JSON.stringify(valueToStore));
  };

  return [storedValue, setValue] as const;
}

// hooks/useOnClickOutside.ts
function useOnClickOutside<T extends HTMLElement>(
  ref: RefObject<T>,
  handler: (event: MouseEvent | TouchEvent) => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) return;
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
```

---

## Context Pattern Template

```tsx
// contexts/AuthContext.tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE' }
  | { type: 'LOGOUT' };

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: false,
};

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true };
    case 'LOGIN_SUCCESS':
      return { user: action.payload, isAuthenticated: true, isLoading: false };
    case 'LOGIN_FAILURE':
      return { ...state, isLoading: false };
    case 'LOGOUT':
      return initialState;
    default:
      return state;
  }
}

const AuthContext = createContext<{
  state: AuthState;
  dispatch: React.Dispatch<AuthAction>;
} | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState);

  return (
    <AuthContext.Provider value={{ state, dispatch }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

---

## Resources

### Documentation
- [React Hooks Reference](https://react.dev/reference/react/hooks)
- [Patterns.dev](https://www.patterns.dev/)

### Advanced Reading
- [Kent C. Dodds Blog](https://kentcdodds.com/blog)
- [Dan Abramov's Overreacted](https://overreacted.io/)

---

## Checklist Before Moving On

- [ ] Can create custom hooks
- [ ] Understand Context API and when to use it
- [ ] Can implement useReducer for complex state
- [ ] Know when and how to optimize with memo/useMemo/useCallback
- [ ] Can implement error boundaries
- [ ] Understand portals and their use cases
- [ ] Can build compound components
- [ ] Completed UI Component Library project

---

**Next:** [Next.js](../03-nextjs/README.md)
