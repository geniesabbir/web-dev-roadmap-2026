# Day 1: React Context API - Global State Management

## Introduction

The Context API is React's built-in solution for sharing data across multiple components without explicitly passing props through every level of the component tree (known as "prop drilling"). It's perfect for themes, user authentication, language preferences, and other globally needed data.

## Learning Objectives

By the end of this lesson, you will:
- Understand what Context is and when to use it
- Create and provide context values
- Consume context in components
- Update context values
- Build practical context patterns

---

## The Problem: Prop Drilling

Without Context, passing data through many levels looks like this:

```jsx
// ❌ Prop drilling - passing user through every component
function App() {
  const [user, setUser] = useState({ name: 'John', role: 'admin' });

  return <Layout user={user} />;
}

function Layout({ user }) {
  return (
    <div>
      <Header user={user} />
      <Main user={user} />
      <Footer user={user} />
    </div>
  );
}

function Header({ user }) {
  return (
    <header>
      <Logo />
      <Navigation user={user} />
    </header>
  );
}

function Navigation({ user }) {
  return (
    <nav>
      <UserMenu user={user} /> {/* Finally uses the user! */}
    </nav>
  );
}
```

**Problems with prop drilling:**
- Verbose and repetitive code
- Intermediate components receive props they don't use
- Hard to refactor
- Difficult to maintain

---

## The Solution: Context API

Context provides a way to pass data through the component tree without passing props manually at every level.

### Three Steps to Using Context

1. **Create** the context
2. **Provide** the context value
3. **Consume** the context in components

---

## Step 1: Creating Context

```jsx
import { createContext } from 'react';

// Create context with an optional default value
const UserContext = createContext(null);

// Or with a default value
const ThemeContext = createContext('light');

export { UserContext, ThemeContext };
```

### Best Practice: Separate File

```jsx
// src/context/UserContext.js
import { createContext } from 'react';

export const UserContext = createContext(null);
```

---

## Step 2: Providing Context

Wrap your component tree with the Provider:

```jsx
import { useState } from 'react';
import { UserContext } from './context/UserContext';

function App() {
  const [user, setUser] = useState({
    name: 'John',
    email: 'john@example.com',
    role: 'admin'
  });

  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}
```

### The Provider Component

- Provider wraps components that need access to the context
- All descendants can access the provided value
- Multiple components can consume the same context

```jsx
// Nested providers are valid
function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={theme}>
        <Layout />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

---

## Step 3: Consuming Context

### Using useContext Hook (Recommended)

```jsx
import { useContext } from 'react';
import { UserContext } from './context/UserContext';

function UserProfile() {
  const user = useContext(UserContext);

  if (!user) {
    return <p>Please log in</p>;
  }

  return (
    <div className="profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <span className="role">{user.role}</span>
    </div>
  );
}
```

### Using Consumer Component (Legacy/Class Components)

```jsx
import { UserContext } from './context/UserContext';

function UserGreeting() {
  return (
    <UserContext.Consumer>
      {user => (
        <h1>Hello, {user?.name || 'Guest'}!</h1>
      )}
    </UserContext.Consumer>
  );
}
```

---

## Complete Example: User Authentication Context

### Creating the Auth Context

```jsx
// src/context/AuthContext.jsx
import { createContext, useContext, useState, useCallback } from 'react';

// Create context
const AuthContext = createContext(null);

// Custom hook for using auth context
export function useAuth() {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }

  return context;
}

// Provider component
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const login = useCallback(async (email, password) => {
    setLoading(true);
    setError(null);

    try {
      // Simulated API call
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });

      if (!response.ok) {
        throw new Error('Invalid credentials');
      }

      const userData = await response.json();
      setUser(userData);

      // Store token
      localStorage.setItem('token', userData.token);

      return userData;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem('token');
  }, []);

  const value = {
    user,
    loading,
    error,
    isAuthenticated: !!user,
    login,
    logout
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
```

### Using the Auth Context

```jsx
// src/App.jsx
import { AuthProvider } from './context/AuthContext';
import { Router } from './Router';

function App() {
  return (
    <AuthProvider>
      <Router />
    </AuthProvider>
  );
}

// src/components/LoginForm.jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { login, loading, error } = useAuth();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(email, password);
      // Redirect on success
    } catch (err) {
      // Error is already in context
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}

      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        disabled={loading}
      />

      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        disabled={loading}
      />

      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

// src/components/UserMenu.jsx
import { useAuth } from '../context/AuthContext';

function UserMenu() {
  const { user, isAuthenticated, logout } = useAuth();

  if (!isAuthenticated) {
    return <a href="/login">Login</a>;
  }

  return (
    <div className="user-menu">
      <span>Hello, {user.name}</span>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// src/components/ProtectedRoute.jsx
import { useAuth } from '../context/AuthContext';
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

---

## Theme Context Example

```jsx
// src/context/ThemeContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';

const ThemeContext = createContext(null);

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

export function ThemeProvider({ children }) {
  // Initialize from localStorage or system preference
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('theme');
    if (saved) return saved;

    return window.matchMedia('(prefers-color-scheme: dark)').matches
      ? 'dark'
      : 'light';
  });

  // Apply theme to document
  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    setTheme,
    toggleTheme,
    isDark: theme === 'dark'
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage
function ThemeToggle() {
  const { theme, toggleTheme, isDark } = useTheme();

  return (
    <button onClick={toggleTheme} aria-label="Toggle theme">
      {isDark ? '☀️' : '🌙'}
    </button>
  );
}

function Card({ children }) {
  const { isDark } = useTheme();

  return (
    <div className={`card ${isDark ? 'card-dark' : 'card-light'}`}>
      {children}
    </div>
  );
}
```

---

## Multiple Contexts

### Combining Multiple Providers

```jsx
// src/context/index.jsx
import { AuthProvider } from './AuthContext';
import { ThemeProvider } from './ThemeContext';
import { NotificationProvider } from './NotificationContext';

export function AppProviders({ children }) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// src/App.jsx
import { AppProviders } from './context';

function App() {
  return (
    <AppProviders>
      <Router />
    </AppProviders>
  );
}
```

---

## Context with TypeScript

```tsx
// src/context/UserContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Define types
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface UserContextType {
  user: User | null;
  setUser: (user: User | null) => void;
  isAdmin: boolean;
}

// Create context with proper typing
const UserContext = createContext<UserContextType | null>(null);

// Custom hook with type checking
export function useUser(): UserContextType {
  const context = useContext(UserContext);

  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }

  return context;
}

// Provider props type
interface UserProviderProps {
  children: ReactNode;
}

// Provider component
export function UserProvider({ children }: UserProviderProps) {
  const [user, setUser] = useState<User | null>(null);

  const value: UserContextType = {
    user,
    setUser,
    isAdmin: user?.role === 'admin'
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

---

## When to Use Context

### ✅ Good Use Cases

1. **Theme data** (colors, fonts, dark mode)
2. **User authentication** state
3. **Locale/language** preferences
4. **UI state** (sidebar open, modal state)
5. **Feature flags**

### ❌ Avoid Context For

1. **Frequently changing data** (every keystroke)
2. **Large amounts of data**
3. **Data only needed by a few components**
4. **Complex state logic** (use useReducer or state libraries)

```jsx
// ❌ BAD - Too frequent updates
const MouseContext = createContext({ x: 0, y: 0 });

function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  // Updates on every mouse move - causes all consumers to re-render!
  useEffect(() => {
    const handler = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);

  return (
    <MouseContext.Provider value={position}>
      <App />
    </MouseContext.Provider>
  );
}
```

---

## Performance Considerations

### Problem: Unnecessary Re-renders

Every time context value changes, ALL consumers re-render:

```jsx
// ❌ Creates new object every render
function Provider({ children }) {
  const [user, setUser] = useState(null);

  // This object is new every render!
  const value = { user, setUser };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

### Solution: Memoize Context Value

```jsx
import { useMemo, useCallback } from 'react';

function Provider({ children }) {
  const [user, setUser] = useState(null);

  // Memoize the value object
  const value = useMemo(() => ({
    user,
    setUser
  }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

### Solution: Split Contexts

```jsx
// Split state and dispatch into separate contexts
const UserStateContext = createContext(null);
const UserDispatchContext = createContext(null);

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={setUser}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}

// Components can subscribe to only what they need
function UserDisplay() {
  const user = useContext(UserStateContext); // Only re-renders when user changes
  return <span>{user?.name}</span>;
}

function UserActions() {
  const setUser = useContext(UserDispatchContext); // Never re-renders from user changes
  return <button onClick={() => setUser(null)}>Logout</button>;
}
```

---

## Exercises

### Exercise 1: Notification Context

Create a notification system context:

```jsx
// Requirements:
// - Add notifications with type (success, error, info)
// - Auto-remove notifications after 5 seconds
// - Manually dismiss notifications
// - Display multiple notifications

function useNotifications() {
  // TODO: Implement
}

// Usage:
function App() {
  const { addNotification } = useNotifications();

  const handleSave = () => {
    addNotification({
      type: 'success',
      message: 'Saved successfully!'
    });
  };
}
```

### Exercise 2: Shopping Cart Context

Create a shopping cart context:

```jsx
// Requirements:
// - Add items to cart
// - Remove items from cart
// - Update item quantity
// - Calculate total
// - Persist to localStorage

function useCart() {
  // TODO: Implement
}
```

---

## Solutions

### Solution 1: Notification Context

```jsx
// src/context/NotificationContext.jsx
import { createContext, useContext, useState, useCallback } from 'react';

const NotificationContext = createContext(null);

export function useNotifications() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotifications must be used within NotificationProvider');
  }
  return context;
}

export function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);

  const addNotification = useCallback(({ type = 'info', message, duration = 5000 }) => {
    const id = Date.now();

    setNotifications(prev => [...prev, { id, type, message }]);

    // Auto-remove after duration
    if (duration > 0) {
      setTimeout(() => {
        removeNotification(id);
      }, duration);
    }

    return id;
  }, []);

  const removeNotification = useCallback((id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  }, []);

  const clearAll = useCallback(() => {
    setNotifications([]);
  }, []);

  const value = {
    notifications,
    addNotification,
    removeNotification,
    clearAll
  };

  return (
    <NotificationContext.Provider value={value}>
      {children}
      <NotificationContainer />
    </NotificationContext.Provider>
  );
}

function NotificationContainer() {
  const { notifications, removeNotification } = useNotifications();

  return (
    <div className="notification-container">
      {notifications.map(notification => (
        <div
          key={notification.id}
          className={`notification notification-${notification.type}`}
        >
          <span>{notification.message}</span>
          <button onClick={() => removeNotification(notification.id)}>
            ×
          </button>
        </div>
      ))}
    </div>
  );
}

// Usage
function SaveButton() {
  const { addNotification } = useNotifications();

  const handleSave = async () => {
    try {
      await saveData();
      addNotification({
        type: 'success',
        message: 'Data saved successfully!'
      });
    } catch (error) {
      addNotification({
        type: 'error',
        message: 'Failed to save data',
        duration: 0 // Don't auto-remove errors
      });
    }
  };

  return <button onClick={handleSave}>Save</button>;
}
```

### Solution 2: Shopping Cart Context

```jsx
// src/context/CartContext.jsx
import { createContext, useContext, useReducer, useEffect, useMemo } from 'react';

const CartContext = createContext(null);

// Actions
const ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',
  CLEAR_CART: 'CLEAR_CART',
  LOAD_CART: 'LOAD_CART'
};

// Reducer
function cartReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_ITEM: {
      const existingIndex = state.items.findIndex(
        item => item.id === action.payload.id
      );

      if (existingIndex >= 0) {
        const newItems = [...state.items];
        newItems[existingIndex].quantity += action.payload.quantity || 1;
        return { ...state, items: newItems };
      }

      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: action.payload.quantity || 1 }]
      };
    }

    case ACTIONS.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };

    case ACTIONS.UPDATE_QUANTITY: {
      const { id, quantity } = action.payload;

      if (quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(item => item.id !== id)
        };
      }

      return {
        ...state,
        items: state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        )
      };
    }

    case ACTIONS.CLEAR_CART:
      return { ...state, items: [] };

    case ACTIONS.LOAD_CART:
      return { ...state, items: action.payload };

    default:
      return state;
  }
}

// Initial state
const initialState = {
  items: []
};

export function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  return context;
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  // Load from localStorage on mount
  useEffect(() => {
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      try {
        const items = JSON.parse(savedCart);
        dispatch({ type: ACTIONS.LOAD_CART, payload: items });
      } catch (error) {
        console.error('Failed to load cart:', error);
      }
    }
  }, []);

  // Save to localStorage on changes
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(state.items));
  }, [state.items]);

  // Actions
  const addItem = (item) => {
    dispatch({ type: ACTIONS.ADD_ITEM, payload: item });
  };

  const removeItem = (id) => {
    dispatch({ type: ACTIONS.REMOVE_ITEM, payload: id });
  };

  const updateQuantity = (id, quantity) => {
    dispatch({ type: ACTIONS.UPDATE_QUANTITY, payload: { id, quantity } });
  };

  const clearCart = () => {
    dispatch({ type: ACTIONS.CLEAR_CART });
  };

  // Computed values
  const itemCount = useMemo(
    () => state.items.reduce((total, item) => total + item.quantity, 0),
    [state.items]
  );

  const subtotal = useMemo(
    () => state.items.reduce((total, item) => total + item.price * item.quantity, 0),
    [state.items]
  );

  const value = {
    items: state.items,
    itemCount,
    subtotal,
    addItem,
    removeItem,
    updateQuantity,
    clearCart
  };

  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// Usage
function ProductCard({ product }) {
  const { addItem } = useCart();

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { items, itemCount, subtotal, removeItem, updateQuantity, clearCart } = useCart();

  return (
    <div className="cart">
      <h2>Cart ({itemCount} items)</h2>

      {items.map(item => (
        <div key={item.id} className="cart-item">
          <span>{item.name}</span>
          <input
            type="number"
            value={item.quantity}
            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
            min="0"
          />
          <span>${(item.price * item.quantity).toFixed(2)}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}

      <div className="cart-total">
        <strong>Subtotal: ${subtotal.toFixed(2)}</strong>
      </div>

      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}
```

---

## Key Takeaways

1. **Context solves prop drilling** - share data without passing props through every level
2. **Three steps**: Create, Provide, Consume
3. **Custom hooks** make consuming context cleaner and type-safe
4. **Separate concerns** - create different contexts for different types of data
5. **Memoize values** to prevent unnecessary re-renders
6. **Split contexts** if you need to optimize performance
7. **Don't overuse** - Context isn't a replacement for all state management

---

## What's Next?

Tomorrow we'll learn about **useReducer** - a more powerful way to manage complex state that works great with Context for building scalable applications!
