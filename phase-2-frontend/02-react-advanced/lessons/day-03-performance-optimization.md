# Day 3: Performance Optimization - Making React Apps Fast

## Introduction

React is fast by default, but as applications grow, performance issues can emerge. Understanding how React renders components and learning to optimize when necessary is crucial for building responsive applications. This lesson covers React's optimization tools and strategies.

## Learning Objectives

By the end of this lesson, you will:
- Understand React's rendering behavior
- Master useMemo and useCallback hooks
- Use React.memo for component optimization
- Profile and identify performance issues
- Apply optimization strategies effectively

---

## Understanding React Rendering

### When Does React Re-render?

A component re-renders when:
1. Its **state** changes
2. Its **props** change
3. Its **parent** re-renders
4. Its **context** value changes

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // Every time Parent re-renders, Child also re-renders
  // even if Child doesn't use count!
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Child /> {/* Re-renders unnecessarily */}
    </div>
  );
}

function Child() {
  console.log('Child rendered');
  return <p>I'm a child component</p>;
}
```

### The Problem

Re-renders aren't inherently bad, but unnecessary re-renders can cause:
- Slow UI updates
- Laggy interactions
- Wasted CPU cycles
- Battery drain on mobile devices

---

## React.memo - Preventing Component Re-renders

`React.memo` is a higher-order component that memoizes a component, preventing re-renders if props haven't changed.

### Basic Usage

```jsx
import { memo } from 'react';

// Without memo - re-renders every time parent renders
function ExpensiveComponent({ data }) {
  console.log('ExpensiveComponent rendered');
  return <div>{/* complex rendering */}</div>;
}

// With memo - only re-renders when props change
const MemoizedComponent = memo(function ExpensiveComponent({ data }) {
  console.log('MemoizedComponent rendered');
  return <div>{/* complex rendering */}</div>;
});

// Or using arrow function
const MemoizedComponent = memo(({ data }) => {
  return <div>{/* complex rendering */}</div>;
});
```

### Complete Example

```jsx
import { useState, memo } from 'react';

// Expensive child component
const TodoItem = memo(function TodoItem({ todo, onToggle, onDelete }) {
  console.log(`Rendering TodoItem: ${todo.id}`);

  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build app', completed: false }
  ]);
  const [filter, setFilter] = useState('all');

  // Without useCallback, these create new functions every render
  // causing ALL TodoItems to re-render!
  const handleToggle = (id) => {
    setTodos(todos.map(t =>
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
  };

  const handleDelete = (id) => {
    setTodos(todos.filter(t => t.id !== id));
  };

  return (
    <div>
      <select value={filter} onChange={e => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="active">Active</option>
        <option value="completed">Completed</option>
      </select>

      <ul>
        {todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}
```

### Custom Comparison Function

By default, React.memo does shallow comparison. For custom comparison:

```jsx
const MyComponent = memo(
  function MyComponent({ user, settings }) {
    return <div>{user.name} - {settings.theme}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.settings.theme === nextProps.settings.theme
    );
  }
);
```

---

## useCallback - Memoizing Functions

`useCallback` returns a memoized version of a callback function that only changes if dependencies change.

### The Problem

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // New function created every render!
  const handleClick = () => {
    console.log('Clicked');
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      {/* MemoizedChild re-renders because handleClick is new */}
      <MemoizedChild onClick={handleClick} />
    </div>
  );
}

const MemoizedChild = memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});
```

### The Solution

```jsx
import { useCallback, memo, useState } from 'react';

function Parent() {
  const [count, setCount] = useState(0);

  // Same function reference across renders
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // Empty deps = never recreated

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      {/* MemoizedChild doesn't re-render when count changes */}
      <MemoizedChild onClick={handleClick} />
    </div>
  );
}
```

### useCallback with Dependencies

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  // Only recreated when todos changes
  const handleToggle = useCallback((id) => {
    setTodos(currentTodos =>
      currentTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []); // Using functional update, no deps needed!

  const handleDelete = useCallback((id) => {
    setTodos(currentTodos => currentTodos.filter(todo => todo.id !== id));
  }, []);

  // This needs userId as dependency
  const [userId, setUserId] = useState(1);

  const fetchUserTodos = useCallback(async () => {
    const response = await fetch(`/api/users/${userId}/todos`);
    const data = await response.json();
    setTodos(data);
  }, [userId]); // Recreated when userId changes

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}
```

---

## useMemo - Memoizing Values

`useMemo` memoizes the result of a computation, only recalculating when dependencies change.

### Expensive Calculations

```jsx
import { useMemo, useState } from 'react';

function ProductList({ products }) {
  const [sortBy, setSortBy] = useState('price');
  const [filterText, setFilterText] = useState('');

  // Without useMemo - recalculates on every render
  // const sortedAndFilteredProducts = products
  //   .filter(p => p.name.includes(filterText))
  //   .sort((a, b) => a[sortBy] - b[sortBy]);

  // With useMemo - only recalculates when dependencies change
  const sortedAndFilteredProducts = useMemo(() => {
    console.log('Calculating sorted products...');
    return products
      .filter(p => p.name.toLowerCase().includes(filterText.toLowerCase()))
      .sort((a, b) => {
        if (sortBy === 'price') return a.price - b.price;
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        return 0;
      });
  }, [products, sortBy, filterText]);

  return (
    <div>
      <input
        value={filterText}
        onChange={e => setFilterText(e.target.value)}
        placeholder="Filter..."
      />
      <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
        <option value="price">Price</option>
        <option value="name">Name</option>
      </select>

      {sortedAndFilteredProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Memoizing Objects and Arrays

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  // Without useMemo - new object every render
  // const userConfig = { theme: 'dark', userId: user?.id };

  // With useMemo - same reference if user hasn't changed
  const userConfig = useMemo(() => ({
    theme: 'dark',
    userId: user?.id,
    permissions: calculatePermissions(user)
  }), [user]);

  return <Settings config={userConfig} />;
}
```

### Memoizing Context Values

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  // Memoize to prevent all consumers from re-rendering
  const value = useMemo(() => ({
    theme,
    setTheme,
    isDark: theme === 'dark'
  }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

## When to Optimize (and When Not To)

### Don't Optimize Prematurely

```jsx
// ❌ Unnecessary - simple string concatenation
const greeting = useMemo(() => `Hello, ${name}!`, [name]);

// ❌ Unnecessary - simple callback
const handleClick = useCallback(() => setCount(c => c + 1), []);

// ✅ Just write it simply
const greeting = `Hello, ${name}!`;
const handleClick = () => setCount(c => c + 1);
```

### Do Optimize When

1. **Expensive calculations** take measurable time
2. **Large lists** of components
3. **Frequent re-renders** causing lag
4. **Callbacks passed to memoized children**
5. **Context values** used by many components

```jsx
// ✅ Optimize: Expensive computation
const sortedData = useMemo(() =>
  data.sort((a, b) => complexComparison(a, b)),
  [data]
);

// ✅ Optimize: Callback to memoized child
const handleSubmit = useCallback((data) => {
  submitForm(data);
}, []);

// ✅ Optimize: Large list items
const MemoizedRow = memo(({ item, onSelect }) => (
  <tr onClick={() => onSelect(item.id)}>
    <td>{item.name}</td>
    <td>{item.value}</td>
  </tr>
));
```

---

## Profiling React Applications

### React DevTools Profiler

1. Install React DevTools browser extension
2. Open DevTools → Profiler tab
3. Click "Record" and interact with your app
4. Analyze the flame graph

### Identifying Slow Components

```jsx
// Add a render counter during development
function MyComponent() {
  const renderCount = useRef(0);
  renderCount.current += 1;

  console.log(`MyComponent rendered ${renderCount.current} times`);

  return <div>...</div>;
}
```

### Using why-did-you-render

```jsx
// Install: npm install @welldone-software/why-did-you-render

// In your app entry point (development only)
import React from 'react';

if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, {
    trackAllPureComponents: true,
  });
}

// Mark components to track
MyComponent.whyDidYouRender = true;
```

---

## Optimization Patterns

### 1. Move State Down

```jsx
// ❌ Bad - entire app re-renders on input change
function App() {
  const [text, setText] = useState('');

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <ExpensiveTree /> {/* Re-renders on every keystroke! */}
    </div>
  );
}

// ✅ Good - isolate frequently changing state
function App() {
  return (
    <div>
      <SearchInput />
      <ExpensiveTree /> {/* Doesn't re-render on input change */}
    </div>
  );
}

function SearchInput() {
  const [text, setText] = useState('');
  return <input value={text} onChange={e => setText(e.target.value)} />;
}
```

### 2. Lift Content Up

```jsx
// ❌ Bad - ExpensiveTree in component with changing state
function ColorPicker() {
  const [color, setColor] = useState('red');

  return (
    <div style={{ backgroundColor: color }}>
      <input value={color} onChange={e => setColor(e.target.value)} />
      <ExpensiveTree /> {/* Re-renders on color change */}
    </div>
  );
}

// ✅ Good - Pass ExpensiveTree as children
function App() {
  return (
    <ColorPicker>
      <ExpensiveTree /> {/* Passed as children, won't re-render */}
    </ColorPicker>
  );
}

function ColorPicker({ children }) {
  const [color, setColor] = useState('red');

  return (
    <div style={{ backgroundColor: color }}>
      <input value={color} onChange={e => setColor(e.target.value)} />
      {children}
    </div>
  );
}
```

### 3. Virtualization for Long Lists

```jsx
// For very long lists, render only visible items
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={35}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### 4. Debouncing Expensive Operations

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      search(debouncedQuery).then(setResults);
    }
  }, [debouncedQuery]);

  return <ResultsList results={results} />;
}

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

### 5. Code Splitting with React.lazy

```jsx
import { lazy, Suspense } from 'react';

// Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const AdminDashboard = lazy(() => import('./AdminDashboard'));

function App() {
  const [showHeavy, setShowHeavy] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavy(true)}>
        Load Heavy Component
      </button>

      {showHeavy && (
        <Suspense fallback={<div>Loading...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}
```

---

## Exercises

### Exercise 1: Optimize a Slow List

The following component is slow. Optimize it:

```jsx
function SlowList({ items, onItemClick }) {
  const [searchTerm, setSearchTerm] = useState('');

  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <input
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {filteredItems.map(item => (
          <ListItem
            key={item.id}
            item={item}
            onClick={() => onItemClick(item)}
          />
        ))}
      </ul>
    </div>
  );
}

function ListItem({ item, onClick }) {
  // Simulate expensive render
  const start = performance.now();
  while (performance.now() - start < 1) {}

  return (
    <li onClick={onClick}>
      {item.name} - ${item.price}
    </li>
  );
}
```

### Exercise 2: Fix Context Performance

This context causes unnecessary re-renders. Fix it:

```jsx
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);

  const value = {
    user, setUser,
    theme, setTheme,
    notifications, setNotifications
  };

  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}
```

---

## Solutions

### Solution 1: Optimized List

```jsx
import { useState, useMemo, useCallback, memo } from 'react';

function OptimizedList({ items, onItemClick }) {
  const [searchTerm, setSearchTerm] = useState('');

  // Memoize filtered items
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);

  // Memoize click handler
  const handleItemClick = useCallback((item) => {
    onItemClick(item);
  }, [onItemClick]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {filteredItems.map(item => (
          <MemoizedListItem
            key={item.id}
            item={item}
            onClick={handleItemClick}
          />
        ))}
      </ul>
    </div>
  );
}

// Memoize the list item
const MemoizedListItem = memo(function ListItem({ item, onClick }) {
  // Simulate expensive render
  const start = performance.now();
  while (performance.now() - start < 1) {}

  return (
    <li onClick={() => onClick(item)}>
      {item.name} - ${item.price}
    </li>
  );
});

// Even better: Use virtualization for very long lists
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items, onItemClick }) {
  const Row = useCallback(({ index, style }) => (
    <div style={style} onClick={() => onItemClick(items[index])}>
      {items[index].name} - ${items[index].price}
    </div>
  ), [items, onItemClick]);

  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={35}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Solution 2: Split Context for Performance

```jsx
import { createContext, useContext, useState, useMemo } from 'react';

// Split into separate contexts
const UserContext = createContext(null);
const ThemeContext = createContext(null);
const NotificationContext = createContext(null);

// Individual providers
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const value = useMemo(() => ({ user, setUser }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const value = useMemo(() => ({
    theme,
    setTheme,
    isDark: theme === 'dark'
  }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);

  const value = useMemo(() => ({
    notifications,
    addNotification: (n) => setNotifications(prev => [...prev, n]),
    removeNotification: (id) => setNotifications(prev =>
      prev.filter(n => n.id !== id)
    )
  }), [notifications]);

  return (
    <NotificationContext.Provider value={value}>
      {children}
    </NotificationContext.Provider>
  );
}

// Combined provider
function AppProvider({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Custom hooks
export const useUser = () => useContext(UserContext);
export const useTheme = () => useContext(ThemeContext);
export const useNotifications = () => useContext(NotificationContext);

// Now components only re-render when their specific context changes
function Header() {
  const { user } = useUser(); // Only re-renders on user changes
  return <header>{user?.name}</header>;
}

function ThemeToggle() {
  const { theme, setTheme } = useTheme(); // Only re-renders on theme changes
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}
```

---

## Performance Checklist

Before optimizing:
- [ ] Is there actually a performance problem?
- [ ] Have you profiled to identify the bottleneck?

Optimization strategies:
- [ ] Use React.memo for expensive components
- [ ] Use useCallback for functions passed as props
- [ ] Use useMemo for expensive calculations
- [ ] Move state down to isolate re-renders
- [ ] Split context to prevent unnecessary updates
- [ ] Virtualize long lists
- [ ] Lazy load heavy components
- [ ] Debounce frequent updates

---

## Key Takeaways

1. **Don't optimize prematurely** - Profile first, optimize second
2. **React.memo** prevents re-renders when props haven't changed
3. **useCallback** memoizes functions to maintain referential equality
4. **useMemo** memoizes values and expensive computations
5. **Split contexts** to prevent cascading re-renders
6. **Move state down** to isolate frequent updates
7. **Virtualize long lists** for massive performance gains
8. **Code splitting** reduces initial bundle size

---

## What's Next?

Tomorrow we'll learn about **Error Boundaries** - handling errors gracefully in React applications!
