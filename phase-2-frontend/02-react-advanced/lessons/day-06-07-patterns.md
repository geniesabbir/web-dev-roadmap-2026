# Days 6-7: Advanced React Patterns - Building Flexible Components

## Introduction

React patterns are proven solutions to common design problems. They help create components that are flexible, reusable, and maintainable. In this two-day lesson, we'll master the most important patterns used in modern React applications.

## Learning Objectives

By the end of this lesson, you will:
- Master Compound Components pattern
- Understand Render Props pattern
- Create Higher-Order Components (HOCs)
- Implement Custom Hook patterns
- Use the Provider pattern effectively
- Know when to apply each pattern

---

## Pattern 1: Compound Components

Compound components are a set of components that work together to form a complete UI. Think of HTML's `<select>` and `<option>` - they're designed to work together.

### The Problem

```jsx
// ❌ Inflexible - all props at top level
<Tabs
  tabs={['Profile', 'Settings', 'Notifications']}
  contents={[<Profile />, <Settings />, <Notifications />]}
  activeIndex={0}
  onChange={handleChange}
/>

// Hard to customize individual tabs
// Can't add icons, badges, or conditional tabs easily
```

### The Solution: Compound Components

```jsx
// ✅ Flexible compound components
<Tabs defaultIndex={0} onChange={handleChange}>
  <TabList>
    <Tab>Profile</Tab>
    <Tab>Settings</Tab>
    <Tab disabled>Notifications</Tab>
  </TabList>
  <TabPanels>
    <TabPanel><Profile /></TabPanel>
    <TabPanel><Settings /></TabPanel>
    <TabPanel><Notifications /></TabPanel>
  </TabPanels>
</Tabs>
```

### Complete Implementation

```jsx
import { createContext, useContext, useState } from 'react';

// Context for sharing state
const TabsContext = createContext(null);

// Main Tabs component
function Tabs({ children, defaultIndex = 0, onChange }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);

  const handleChange = (index) => {
    setActiveIndex(index);
    onChange?.(index);
  };

  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex: handleChange }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Container for tab buttons
function TabList({ children }) {
  return (
    <div className="tab-list" role="tablist">
      {children}
    </div>
  );
}

// Individual tab button
function Tab({ children, index, disabled = false }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);

  return (
    <button
      role="tab"
      aria-selected={activeIndex === index}
      disabled={disabled}
      className={`tab ${activeIndex === index ? 'active' : ''}`}
      onClick={() => !disabled && setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

// Container for tab panels
function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

// Individual tab panel
function TabPanel({ children, index }) {
  const { activeIndex } = useContext(TabsContext);

  if (activeIndex !== index) return null;

  return (
    <div role="tabpanel" className="tab-panel">
      {children}
    </div>
  );
}

// Auto-assign indices to children
function TabsWrapper({ children, defaultIndex = 0, onChange }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);

  const handleChange = (index) => {
    setActiveIndex(index);
    onChange?.(index);
  };

  // Clone children and inject index props
  const enhanceChildren = (children, type, startIndex = 0) => {
    let index = startIndex;
    return React.Children.map(children, (child) => {
      if (!React.isValidElement(child)) return child;

      if (child.type === type) {
        return React.cloneElement(child, { index: index++ });
      }

      return child;
    });
  };

  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex: handleChange }}>
      <div className="tabs">
        {React.Children.map(children, (child) => {
          if (child.type === TabList) {
            return React.cloneElement(child, {
              children: enhanceChildren(child.props.children, Tab)
            });
          }
          if (child.type === TabPanels) {
            return React.cloneElement(child, {
              children: enhanceChildren(child.props.children, TabPanel)
            });
          }
          return child;
        })}
      </div>
    </TabsContext.Provider>
  );
}

// Export components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panels = TabPanels;
Tabs.Panel = TabPanel;

export default Tabs;
```

### Usage

```jsx
function Settings() {
  return (
    <Tabs defaultIndex={0} onChange={(i) => console.log('Tab:', i)}>
      <Tabs.List>
        <Tabs.Tab>General</Tabs.Tab>
        <Tabs.Tab>Security</Tabs.Tab>
        <Tabs.Tab>Notifications</Tabs.Tab>
      </Tabs.List>
      <Tabs.Panels>
        <Tabs.Panel>
          <GeneralSettings />
        </Tabs.Panel>
        <Tabs.Panel>
          <SecuritySettings />
        </Tabs.Panel>
        <Tabs.Panel>
          <NotificationSettings />
        </Tabs.Panel>
      </Tabs.Panels>
    </Tabs>
  );
}
```

### More Compound Component Examples

#### Accordion

```jsx
const AccordionContext = createContext(null);

function Accordion({ children, multiple = false }) {
  const [openItems, setOpenItems] = useState(new Set());

  const toggle = (id) => {
    setOpenItems((prev) => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        if (!multiple) next.clear();
        next.add(id);
      }
      return next;
    });
  };

  const isOpen = (id) => openItems.has(id);

  return (
    <AccordionContext.Provider value={{ toggle, isOpen }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ children, id }) {
  return (
    <div className="accordion-item" data-id={id}>
      {React.Children.map(children, (child) =>
        React.cloneElement(child, { itemId: id })
      )}
    </div>
  );
}

function AccordionHeader({ children, itemId }) {
  const { toggle, isOpen } = useContext(AccordionContext);

  return (
    <button
      className={`accordion-header ${isOpen(itemId) ? 'open' : ''}`}
      onClick={() => toggle(itemId)}
      aria-expanded={isOpen(itemId)}
    >
      {children}
      <span className="icon">{isOpen(itemId) ? '−' : '+'}</span>
    </button>
  );
}

function AccordionContent({ children, itemId }) {
  const { isOpen } = useContext(AccordionContext);

  if (!isOpen(itemId)) return null;

  return <div className="accordion-content">{children}</div>;
}

// Usage
<Accordion multiple>
  <AccordionItem id="item1">
    <AccordionHeader>What is React?</AccordionHeader>
    <AccordionContent>React is a JavaScript library...</AccordionContent>
  </AccordionItem>
  <AccordionItem id="item2">
    <AccordionHeader>What are hooks?</AccordionHeader>
    <AccordionContent>Hooks are functions that...</AccordionContent>
  </AccordionItem>
</Accordion>
```

---

## Pattern 2: Render Props

Render props is a pattern where a component receives a function as a prop that returns React elements. It enables sharing code between components.

### Basic Render Props

```jsx
// Component with render prop
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return render(position);
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <p>Mouse position: {x}, {y}</p>
          <div
            style={{
              position: 'fixed',
              left: x - 10,
              top: y - 10,
              width: 20,
              height: 20,
              background: 'red',
              borderRadius: '50%',
              pointerEvents: 'none'
            }}
          />
        </div>
      )}
    />
  );
}
```

### Children as Render Prop

```jsx
// Using children instead of render prop
function Toggle({ children }) {
  const [on, setOn] = useState(false);

  const toggle = () => setOn((prev) => !prev);

  return children({ on, toggle });
}

// Usage
<Toggle>
  {({ on, toggle }) => (
    <div>
      <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>
      {on && <p>The toggle is on!</p>}
    </div>
  )}
</Toggle>
```

### Data Fetching with Render Props

```jsx
function FetchData({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch');
        const json = await response.json();

        if (!cancelled) {
          setData(json);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url]);

  return children({ data, loading, error });
}

// Usage
<FetchData url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error.message}</p>;
    return (
      <ul>
        {data.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    );
  }}
</FetchData>
```

### Modern Alternative: Custom Hooks

Render props have largely been replaced by custom hooks:

```jsx
// Instead of render props
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return position;
}

// Usage
function App() {
  const { x, y } = useMousePosition();

  return <p>Mouse: {x}, {y}</p>;
}
```

---

## Pattern 3: Higher-Order Components (HOCs)

A Higher-Order Component is a function that takes a component and returns a new component with enhanced functionality.

### Basic HOC

```jsx
// HOC that adds loading state
function withLoading(WrappedComponent) {
  return function WithLoading({ isLoading, ...props }) {
    if (isLoading) {
      return <div className="loading">Loading...</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

// Usage
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

// In component
<UserListWithLoading isLoading={loading} users={users} />
```

### HOC with Configuration

```jsx
// HOC factory
function withErrorBoundary(WrappedComponent, ErrorComponent) {
  return class WithErrorBoundary extends React.Component {
    state = { hasError: false };

    static getDerivedStateFromError() {
      return { hasError: true };
    }

    componentDidCatch(error, errorInfo) {
      console.error('Error:', error, errorInfo);
    }

    render() {
      if (this.state.hasError) {
        return ErrorComponent ? (
          <ErrorComponent />
        ) : (
          <div>Something went wrong</div>
        );
      }

      return <WrappedComponent {...this.props} />;
    }
  };
}

// Usage
const SafeComponent = withErrorBoundary(
  DangerousComponent,
  () => <p>Failed to load</p>
);
```

### Authentication HOC

```jsx
function withAuth(WrappedComponent) {
  return function WithAuth(props) {
    const { user, loading } = useAuth();

    if (loading) {
      return <LoadingSpinner />;
    }

    if (!user) {
      return <Navigate to="/login" />;
    }

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

// In router
<Route path="/dashboard" element={<ProtectedDashboard />} />
```

### HOC Best Practices

```jsx
// ✅ Good HOC practices
function withUser(WrappedComponent) {
  // Display name for debugging
  const displayName =
    WrappedComponent.displayName || WrappedComponent.name || 'Component';

  function WithUser(props) {
    const user = useUser();
    return <WrappedComponent {...props} user={user} />;
  }

  // Set display name
  WithUser.displayName = `WithUser(${displayName})`;

  // Copy static methods
  hoistNonReactStatics(WithUser, WrappedComponent);

  return WithUser;
}
```

---

## Pattern 4: Controlled vs Uncontrolled Components

### Controlled Components

React state drives the component:

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

### Uncontrolled Components

DOM holds the state:

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    console.log('Value:', inputRef.current.value);
  };

  return (
    <div>
      <input ref={inputRef} defaultValue="" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

### Hybrid: Controlled with Initial Value

```jsx
function HybridInput({ initialValue = '', onChange }) {
  const [value, setValue] = useState(initialValue);

  const handleChange = (e) => {
    setValue(e.target.value);
    onChange?.(e.target.value);
  };

  return <input value={value} onChange={handleChange} />;
}
```

### State Reducer Pattern

Give consumers control over state changes:

```jsx
function useToggle(initialState = false, reducer = defaultReducer) {
  const [on, dispatch] = useReducer(reducer, initialState);

  const toggle = () => dispatch({ type: 'TOGGLE' });
  const setOn = () => dispatch({ type: 'ON' });
  const setOff = () => dispatch({ type: 'OFF' });

  return { on, toggle, setOn, setOff };
}

function defaultReducer(state, action) {
  switch (action.type) {
    case 'TOGGLE':
      return !state;
    case 'ON':
      return true;
    case 'OFF':
      return false;
    default:
      return state;
  }
}

// Usage with custom reducer
function App() {
  const { on, toggle } = useToggle(false, (state, action) => {
    // Custom logic: can only turn off, not on
    if (action.type === 'ON') {
      return state; // Prevent turning on
    }
    return defaultReducer(state, action);
  });

  return <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>;
}
```

---

## Pattern 5: Composition vs Inheritance

React favors composition over inheritance:

### Specialization Pattern

```jsx
// Generic component
function Card({ children, className = '', ...props }) {
  return (
    <div className={`card ${className}`} {...props}>
      {children}
    </div>
  );
}

// Specialized components using composition
function InfoCard({ title, children }) {
  return (
    <Card className="info-card">
      <h3>{title}</h3>
      <div className="info-content">{children}</div>
    </Card>
  );
}

function WarningCard({ children }) {
  return (
    <Card className="warning-card">
      <span className="warning-icon">⚠️</span>
      {children}
    </Card>
  );
}

function UserCard({ user }) {
  return (
    <Card className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h4>{user.name}</h4>
      <p>{user.email}</p>
    </Card>
  );
}
```

### Containment Pattern

```jsx
// Layout components
function Page({ children }) {
  return <div className="page">{children}</div>;
}

function Header({ children }) {
  return <header className="header">{children}</header>;
}

function Main({ children }) {
  return <main className="main">{children}</main>;
}

function Sidebar({ children }) {
  return <aside className="sidebar">{children}</aside>;
}

// Usage - compose layout
function DashboardLayout({ user }) {
  return (
    <Page>
      <Header>
        <Logo />
        <Nav />
        <UserMenu user={user} />
      </Header>
      <div className="content-area">
        <Sidebar>
          <SideNav />
        </Sidebar>
        <Main>
          <Outlet /> {/* Router outlet for nested routes */}
        </Main>
      </div>
    </Page>
  );
}
```

### Slot Pattern

```jsx
function Dialog({ header, footer, children }) {
  return (
    <div className="dialog">
      {header && <div className="dialog-header">{header}</div>}
      <div className="dialog-body">{children}</div>
      {footer && <div className="dialog-footer">{footer}</div>}
    </div>
  );
}

// Usage
<Dialog
  header={<h2>Confirm Action</h2>}
  footer={
    <>
      <button onClick={onCancel}>Cancel</button>
      <button onClick={onConfirm}>Confirm</button>
    </>
  }
>
  <p>Are you sure you want to proceed?</p>
</Dialog>
```

---

## Pattern 6: Provider Pattern

Centralized state management using Context:

```jsx
// Feature-specific provider
function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);

  const addNotification = useCallback((notification) => {
    const id = Date.now();
    setNotifications((prev) => [...prev, { ...notification, id }]);

    // Auto-remove after duration
    if (notification.duration !== 0) {
      setTimeout(() => {
        removeNotification(id);
      }, notification.duration || 5000);
    }

    return id;
  }, []);

  const removeNotification = useCallback((id) => {
    setNotifications((prev) => prev.filter((n) => n.id !== id));
  }, []);

  const value = useMemo(
    () => ({
      notifications,
      addNotification,
      removeNotification,
      success: (message) => addNotification({ type: 'success', message }),
      error: (message) => addNotification({ type: 'error', message, duration: 0 }),
      info: (message) => addNotification({ type: 'info', message }),
      warning: (message) => addNotification({ type: 'warning', message })
    }),
    [notifications, addNotification, removeNotification]
  );

  return (
    <NotificationContext.Provider value={value}>
      {children}
      <NotificationContainer notifications={notifications} onDismiss={removeNotification} />
    </NotificationContext.Provider>
  );
}

// Hook
function useNotifications() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotifications must be used within NotificationProvider');
  }
  return context;
}

// Usage
function SaveButton() {
  const { success, error } = useNotifications();

  const handleSave = async () => {
    try {
      await saveData();
      success('Data saved successfully!');
    } catch (err) {
      error('Failed to save data');
    }
  };

  return <button onClick={handleSave}>Save</button>;
}
```

---

## Exercises

### Exercise 1: Compound Menu Component

Create a compound component for a dropdown menu:

```jsx
// Target API
<Menu>
  <Menu.Button>Options</Menu.Button>
  <Menu.Items>
    <Menu.Item onClick={() => console.log('Edit')}>Edit</Menu.Item>
    <Menu.Item onClick={() => console.log('Delete')}>Delete</Menu.Item>
    <Menu.Divider />
    <Menu.Item disabled>Archive</Menu.Item>
  </Menu.Items>
</Menu>
```

### Exercise 2: Form Field HOC

Create an HOC that adds validation and error display to any input component:

```jsx
const ValidatedInput = withValidation(Input, {
  required: true,
  minLength: 3,
  pattern: /^[a-zA-Z]+$/
});
```

---

## Solutions

### Solution 1: Compound Menu

```jsx
import { createContext, useContext, useState, useRef, useEffect } from 'react';

const MenuContext = createContext(null);

function Menu({ children }) {
  const [isOpen, setIsOpen] = useState(false);
  const menuRef = useRef(null);

  const toggle = () => setIsOpen((prev) => !prev);
  const close = () => setIsOpen(false);

  // Close on outside click
  useEffect(() => {
    const handleClickOutside = (e) => {
      if (menuRef.current && !menuRef.current.contains(e.target)) {
        close();
      }
    };

    if (isOpen) {
      document.addEventListener('mousedown', handleClickOutside);
    }

    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [isOpen]);

  // Close on Escape
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') close();
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
    }

    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen]);

  return (
    <MenuContext.Provider value={{ isOpen, toggle, close }}>
      <div className="menu" ref={menuRef}>
        {children}
      </div>
    </MenuContext.Provider>
  );
}

function MenuButton({ children }) {
  const { isOpen, toggle } = useContext(MenuContext);

  return (
    <button
      className="menu-button"
      onClick={toggle}
      aria-expanded={isOpen}
      aria-haspopup="true"
    >
      {children}
    </button>
  );
}

function MenuItems({ children }) {
  const { isOpen } = useContext(MenuContext);

  if (!isOpen) return null;

  return (
    <div className="menu-items" role="menu">
      {children}
    </div>
  );
}

function MenuItem({ children, onClick, disabled = false }) {
  const { close } = useContext(MenuContext);

  const handleClick = () => {
    if (!disabled) {
      onClick?.();
      close();
    }
  };

  return (
    <button
      className={`menu-item ${disabled ? 'disabled' : ''}`}
      onClick={handleClick}
      disabled={disabled}
      role="menuitem"
    >
      {children}
    </button>
  );
}

function MenuDivider() {
  return <hr className="menu-divider" />;
}

Menu.Button = MenuButton;
Menu.Items = MenuItems;
Menu.Item = MenuItem;
Menu.Divider = MenuDivider;

export default Menu;
```

### Solution 2: Form Validation HOC

```jsx
import { useState, forwardRef } from 'react';

function withValidation(InputComponent, rules = {}) {
  return forwardRef(function ValidatedInput({ onChange, onBlur, ...props }, ref) {
    const [error, setError] = useState(null);
    const [touched, setTouched] = useState(false);

    const validate = (value) => {
      if (rules.required && !value) {
        return 'This field is required';
      }

      if (rules.minLength && value.length < rules.minLength) {
        return `Must be at least ${rules.minLength} characters`;
      }

      if (rules.maxLength && value.length > rules.maxLength) {
        return `Must be no more than ${rules.maxLength} characters`;
      }

      if (rules.pattern && !rules.pattern.test(value)) {
        return rules.patternMessage || 'Invalid format';
      }

      if (rules.validate) {
        return rules.validate(value);
      }

      return null;
    };

    const handleChange = (e) => {
      const value = e.target.value;
      const validationError = validate(value);
      setError(validationError);
      onChange?.(e);
    };

    const handleBlur = (e) => {
      setTouched(true);
      const value = e.target.value;
      const validationError = validate(value);
      setError(validationError);
      onBlur?.(e);
    };

    const showError = touched && error;

    return (
      <div className="validated-field">
        <InputComponent
          ref={ref}
          {...props}
          onChange={handleChange}
          onBlur={handleBlur}
          aria-invalid={showError}
          className={`${props.className || ''} ${showError ? 'error' : ''}`}
        />
        {showError && <span className="error-message">{error}</span>}
      </div>
    );
  });
}

// Usage
const BasicInput = forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));

const ValidatedNameInput = withValidation(BasicInput, {
  required: true,
  minLength: 2,
  maxLength: 50,
  pattern: /^[a-zA-Z\s]+$/,
  patternMessage: 'Name can only contain letters'
});

const ValidatedEmailInput = withValidation(BasicInput, {
  required: true,
  pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  patternMessage: 'Please enter a valid email'
});

// In form
function RegistrationForm() {
  return (
    <form>
      <ValidatedNameInput placeholder="Full Name" />
      <ValidatedEmailInput type="email" placeholder="Email" />
    </form>
  );
}
```

---

## Pattern Selection Guide

| Pattern | Use When |
|---------|----------|
| **Compound Components** | Building UI with multiple related parts (tabs, accordions, menus) |
| **Render Props** | Sharing behavior between components (legacy code) |
| **Custom Hooks** | Sharing stateful logic (modern approach) |
| **HOCs** | Cross-cutting concerns (auth, error boundaries) |
| **Provider Pattern** | App-wide state (themes, auth, notifications) |
| **Composition** | Building specialized components from generic ones |
| **State Reducer** | Giving consumers control over internal state |

---

## Key Takeaways

1. **Compound Components** create flexible, declarative APIs
2. **Custom Hooks** have largely replaced Render Props
3. **HOCs** are still useful for cross-cutting concerns
4. **Composition over inheritance** is the React way
5. **Provider Pattern** centralizes shared state
6. **Choose patterns based on needs** - don't over-engineer
7. **Combine patterns** as needed

---

## What's Next?

Congratulations on completing React Advanced! Next up is **Next.js** - the React framework for production that adds routing, server-side rendering, and much more!
