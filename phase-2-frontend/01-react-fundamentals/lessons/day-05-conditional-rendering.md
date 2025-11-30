# Day 5: Conditional Rendering

## Introduction to Conditional Rendering

Conditional rendering in React works the same way conditions work in JavaScript. Use JavaScript operators to create elements representing the current state.

---

## Basic Conditional Patterns

### Using if/else Statements

```jsx
function Greeting({ isLoggedIn }) {
    if (isLoggedIn) {
        return <h1>Welcome back!</h1>;
    }
    return <h1>Please sign in</h1>;
}

// With more complex logic
function Dashboard({ user }) {
    if (!user) {
        return <LoginPage />;
    }

    if (user.role === 'admin') {
        return <AdminDashboard />;
    }

    if (user.role === 'moderator') {
        return <ModeratorDashboard />;
    }

    return <UserDashboard />;
}
```

### Early Return Pattern

```jsx
function UserProfile({ user, loading, error }) {
    // Handle loading first
    if (loading) {
        return <LoadingSpinner />;
    }

    // Handle error
    if (error) {
        return <ErrorMessage error={error} />;
    }

    // Handle no user
    if (!user) {
        return <NotFound />;
    }

    // Render main content
    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
}
```

---

## Ternary Operator

Best for simple if/else choices:

```jsx
function Button({ isLoading }) {
    return (
        <button disabled={isLoading}>
            {isLoading ? 'Loading...' : 'Submit'}
        </button>
    );
}

function StatusBadge({ status }) {
    return (
        <span className={status === 'active' ? 'badge-green' : 'badge-red'}>
            {status === 'active' ? 'Active' : 'Inactive'}
        </span>
    );
}

// Nested ternary (use sparingly)
function Message({ type }) {
    return (
        <div className={
            type === 'success' ? 'green' :
            type === 'error' ? 'red' :
            type === 'warning' ? 'yellow' :
            'gray'
        }>
            Message content
        </div>
    );
}

// Better alternative for complex conditions
function Message({ type }) {
    const getClassName = () => {
        switch (type) {
            case 'success': return 'green';
            case 'error': return 'red';
            case 'warning': return 'yellow';
            default: return 'gray';
        }
    };

    return <div className={getClassName()}>Message content</div>;
}
```

---

## Logical && Operator

Renders element only when condition is true:

```jsx
function Notifications({ messages }) {
    return (
        <div>
            <h1>Dashboard</h1>
            {messages.length > 0 && (
                <div className="notification">
                    You have {messages.length} new messages
                </div>
            )}
        </div>
    );
}

function AdminPanel({ user }) {
    return (
        <div>
            <h1>Welcome, {user.name}</h1>
            {user.isAdmin && <AdminTools />}
            {user.canEdit && <EditButton />}
            {user.posts.length > 0 && (
                <PostList posts={user.posts} />
            )}
        </div>
    );
}
```

### Avoiding && Gotchas

```jsx
// ⚠️ Watch out for falsy values

function Counter({ count }) {
    // ❌ This renders "0" when count is 0
    return <div>{count && <span>Count: {count}</span>}</div>;
}

function Counter({ count }) {
    // ✅ This works correctly
    return <div>{count > 0 && <span>Count: {count}</span>}</div>;
}

function Counter({ count }) {
    // ✅ Also works
    return <div>{count !== 0 && <span>Count: {count}</span>}</div>;
}

function Counter({ count }) {
    // ✅ Using ternary is always safe
    return <div>{count ? <span>Count: {count}</span> : null}</div>;
}
```

---

## Logical || Operator (Default Values)

```jsx
function Avatar({ imageUrl, name }) {
    return (
        <div>
            {/* Show placeholder if no image */}
            <img src={imageUrl || '/default-avatar.png'} alt={name} />

            {/* Show 'Anonymous' if no name */}
            <span>{name || 'Anonymous'}</span>
        </div>
    );
}

// Nullish coalescing (??) is even better
function Avatar({ imageUrl, name }) {
    return (
        <div>
            {/* Only fallback for null/undefined, not empty string */}
            <img src={imageUrl ?? '/default-avatar.png'} alt={name} />
            <span>{name ?? 'Anonymous'}</span>
        </div>
    );
}
```

---

## Conditional Component Rendering

### Assigning to Variables

```jsx
function LoginControl({ isLoggedIn }) {
    let button;
    let content;

    if (isLoggedIn) {
        button = <LogoutButton />;
        content = <UserDashboard />;
    } else {
        button = <LoginButton />;
        content = <GuestContent />;
    }

    return (
        <div>
            <header>{button}</header>
            <main>{content}</main>
        </div>
    );
}
```

### Using Object Lookup

```jsx
function StatusMessage({ status }) {
    const messages = {
        loading: <LoadingSpinner />,
        error: <ErrorMessage />,
        success: <SuccessMessage />,
        empty: <EmptyState />,
        default: null
    };

    return messages[status] || messages.default;
}

function Icon({ type }) {
    const icons = {
        success: '✅',
        error: '❌',
        warning: '⚠️',
        info: 'ℹ️'
    };

    return <span>{icons[type] || '❓'}</span>;
}
```

### Component Mapping

```jsx
function FormField({ type, ...props }) {
    const components = {
        text: TextInput,
        email: EmailInput,
        password: PasswordInput,
        textarea: TextAreaInput,
        select: SelectInput,
        checkbox: CheckboxInput
    };

    const Component = components[type] || TextInput;

    return <Component {...props} />;
}

// Usage
function Form() {
    return (
        <form>
            <FormField type="text" name="username" />
            <FormField type="email" name="email" />
            <FormField type="password" name="password" />
            <FormField type="select" name="country" options={countries} />
        </form>
    );
}
```

---

## Conditional Attributes

### Conditional className

```jsx
function Button({ variant, size, disabled }) {
    // Template literal approach
    const className = `
        btn
        btn-${variant || 'primary'}
        btn-${size || 'medium'}
        ${disabled ? 'btn-disabled' : ''}
    `.trim();

    return <button className={className} disabled={disabled}>Click</button>;
}

// Array join approach
function Button({ variant, size, disabled, active }) {
    const classes = [
        'btn',
        `btn-${variant || 'primary'}`,
        `btn-${size || 'medium'}`,
        disabled && 'btn-disabled',
        active && 'btn-active'
    ].filter(Boolean).join(' ');

    return <button className={classes}>Click</button>;
}

// With a helper library (classnames/clsx)
import clsx from 'clsx';

function Button({ variant, size, disabled, active }) {
    return (
        <button
            className={clsx(
                'btn',
                `btn-${variant || 'primary'}`,
                `btn-${size || 'medium'}`,
                {
                    'btn-disabled': disabled,
                    'btn-active': active
                }
            )}
        >
            Click
        </button>
    );
}
```

### Conditional Styles

```jsx
function Card({ isHighlighted, isError }) {
    const style = {
        padding: '20px',
        borderRadius: '8px',
        ...(isHighlighted && {
            backgroundColor: '#fffde7',
            border: '2px solid #ffc107'
        }),
        ...(isError && {
            backgroundColor: '#ffebee',
            border: '2px solid #f44336'
        })
    };

    return <div style={style}>Card content</div>;
}
```

### Conditional Props

```jsx
function Link({ href, external, children }) {
    const props = {
        href,
        ...(external && {
            target: '_blank',
            rel: 'noopener noreferrer'
        })
    };

    return <a {...props}>{children}</a>;
}

// Usage
<Link href="/about">Internal Link</Link>
<Link href="https://example.com" external>External Link</Link>
```

---

## Rendering Nothing

```jsx
// Return null to render nothing
function WarningBanner({ warn }) {
    if (!warn) {
        return null;
    }

    return <div className="warning">Warning!</div>;
}

// In JSX
function Profile({ showBio, bio }) {
    return (
        <div>
            <h1>John Doe</h1>
            {showBio ? <p>{bio}</p> : null}
        </div>
    );
}

// Undefined also renders nothing
function OptionalContent({ content }) {
    return (
        <div>
            <h1>Title</h1>
            {content}  {/* If undefined, renders nothing */}
        </div>
    );
}
```

---

## Show/Hide Pattern

```jsx
function ToggleContent() {
    const [isVisible, setIsVisible] = useState(false);

    return (
        <div>
            <button onClick={() => setIsVisible(!isVisible)}>
                {isVisible ? 'Hide' : 'Show'} Content
            </button>

            {isVisible && (
                <div className="content">
                    <p>This content can be toggled</p>
                </div>
            )}
        </div>
    );
}

// With CSS (keeps in DOM)
function ToggleWithCSS() {
    const [isVisible, setIsVisible] = useState(false);

    return (
        <div>
            <button onClick={() => setIsVisible(!isVisible)}>
                Toggle
            </button>

            <div style={{ display: isVisible ? 'block' : 'none' }}>
                <p>Hidden with CSS</p>
            </div>
        </div>
    );
}

// When to use each:
// - Conditional rendering: Content is expensive to render
// - CSS display: Content needs to maintain state when hidden
```

---

## Loading States

```jsx
function DataLoader({ isLoading, error, data, children }) {
    if (isLoading) {
        return (
            <div className="loader">
                <Spinner />
                <p>Loading...</p>
            </div>
        );
    }

    if (error) {
        return (
            <div className="error">
                <p>Error: {error.message}</p>
                <button onClick={() => window.location.reload()}>
                    Retry
                </button>
            </div>
        );
    }

    if (!data) {
        return (
            <div className="empty">
                <p>No data available</p>
            </div>
        );
    }

    return children(data);
}

// Usage
function UserList() {
    const { data, isLoading, error } = useFetchUsers();

    return (
        <DataLoader isLoading={isLoading} error={error} data={data}>
            {(users) => (
                <ul>
                    {users.map(user => (
                        <li key={user.id}>{user.name}</li>
                    ))}
                </ul>
            )}
        </DataLoader>
    );
}
```

---

## Authentication Patterns

```jsx
function ProtectedRoute({ user, children }) {
    if (!user) {
        return <Navigate to="/login" />;
    }

    return children;
}

function RoleBasedAccess({ user, requiredRole, children }) {
    if (!user) {
        return <LoginPrompt />;
    }

    if (user.role !== requiredRole) {
        return <AccessDenied />;
    }

    return children;
}

// Usage
function App() {
    const { user } = useAuth();

    return (
        <Routes>
            <Route path="/login" element={<LoginPage />} />

            <Route
                path="/dashboard"
                element={
                    <ProtectedRoute user={user}>
                        <Dashboard />
                    </ProtectedRoute>
                }
            />

            <Route
                path="/admin"
                element={
                    <RoleBasedAccess user={user} requiredRole="admin">
                        <AdminPanel />
                    </RoleBasedAccess>
                }
            />
        </Routes>
    );
}
```

---

## Practice Exercises

### Exercise 1: Notification Component

```jsx
// Create a Notification component that:
// - Shows different icons based on type (success, error, warning, info)
// - Can be dismissed
// - Auto-dismisses after a timeout (optional)

// Your code here
```

**Solution:**
```jsx
function Notification({ type, message, onDismiss, autoDissmiss = 0 }) {
    const [isVisible, setIsVisible] = useState(true);

    useEffect(() => {
        if (autoDissmiss > 0) {
            const timer = setTimeout(() => {
                setIsVisible(false);
                onDismiss?.();
            }, autoDissmiss);
            return () => clearTimeout(timer);
        }
    }, [autoDissmiss, onDismiss]);

    if (!isVisible) return null;

    const config = {
        success: { icon: '✅', bg: '#d4edda', border: '#28a745' },
        error: { icon: '❌', bg: '#f8d7da', border: '#dc3545' },
        warning: { icon: '⚠️', bg: '#fff3cd', border: '#ffc107' },
        info: { icon: 'ℹ️', bg: '#d1ecf1', border: '#17a2b8' }
    };

    const { icon, bg, border } = config[type] || config.info;

    return (
        <div style={{
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'space-between',
            padding: '12px 16px',
            backgroundColor: bg,
            border: `1px solid ${border}`,
            borderRadius: '8px',
            marginBottom: '10px'
        }}>
            <div style={{ display: 'flex', alignItems: 'center', gap: '10px' }}>
                <span style={{ fontSize: '20px' }}>{icon}</span>
                <span>{message}</span>
            </div>
            <button
                onClick={() => {
                    setIsVisible(false);
                    onDismiss?.();
                }}
                style={{
                    background: 'none',
                    border: 'none',
                    cursor: 'pointer',
                    fontSize: '18px'
                }}
            >
                ×
            </button>
        </div>
    );
}

// Usage
function App() {
    const [notifications, setNotifications] = useState([
        { id: 1, type: 'success', message: 'File uploaded successfully!' },
        { id: 2, type: 'error', message: 'Failed to save changes.' },
        { id: 3, type: 'warning', message: 'Your session will expire soon.' }
    ]);

    const dismiss = (id) => {
        setNotifications(prev => prev.filter(n => n.id !== id));
    };

    return (
        <div style={{ maxWidth: '400px', padding: '20px' }}>
            {notifications.map(n => (
                <Notification
                    key={n.id}
                    type={n.type}
                    message={n.message}
                    onDismiss={() => dismiss(n.id)}
                    autoDissmiss={5000}
                />
            ))}
        </div>
    );
}
```

### Exercise 2: Accordion Component

```jsx
// Create an Accordion component with:
// - Multiple sections
// - Only one section open at a time (optional)
// - Smooth animations
// - Keyboard navigation

// Your code here
```

**Solution:**
```jsx
function Accordion({ items, allowMultiple = false }) {
    const [openItems, setOpenItems] = useState([]);

    const toggleItem = (index) => {
        if (allowMultiple) {
            setOpenItems(prev =>
                prev.includes(index)
                    ? prev.filter(i => i !== index)
                    : [...prev, index]
            );
        } else {
            setOpenItems(prev =>
                prev.includes(index) ? [] : [index]
            );
        }
    };

    const handleKeyDown = (e, index) => {
        if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            toggleItem(index);
        }
    };

    return (
        <div className="accordion">
            {items.map((item, index) => {
                const isOpen = openItems.includes(index);

                return (
                    <div key={index} style={{ marginBottom: '5px' }}>
                        <button
                            onClick={() => toggleItem(index)}
                            onKeyDown={(e) => handleKeyDown(e, index)}
                            style={{
                                width: '100%',
                                padding: '15px',
                                display: 'flex',
                                justifyContent: 'space-between',
                                alignItems: 'center',
                                backgroundColor: isOpen ? '#e3f2fd' : '#f5f5f5',
                                border: '1px solid #ddd',
                                borderRadius: isOpen ? '8px 8px 0 0' : '8px',
                                cursor: 'pointer',
                                fontWeight: 'bold'
                            }}
                            aria-expanded={isOpen}
                        >
                            {item.title}
                            <span style={{
                                transform: isOpen ? 'rotate(180deg)' : 'rotate(0)',
                                transition: 'transform 0.3s'
                            }}>
                                ▼
                            </span>
                        </button>
                        <div
                            style={{
                                maxHeight: isOpen ? '500px' : '0',
                                overflow: 'hidden',
                                transition: 'max-height 0.3s ease-in-out',
                                backgroundColor: 'white',
                                border: isOpen ? '1px solid #ddd' : 'none',
                                borderTop: 'none',
                                borderRadius: '0 0 8px 8px'
                            }}
                        >
                            <div style={{ padding: '15px' }}>
                                {item.content}
                            </div>
                        </div>
                    </div>
                );
            })}
        </div>
    );
}

// Usage
function App() {
    const faqItems = [
        {
            title: 'What is React?',
            content: 'React is a JavaScript library for building user interfaces.'
        },
        {
            title: 'What are components?',
            content: 'Components are reusable pieces of UI that can manage their own state.'
        },
        {
            title: 'What is JSX?',
            content: 'JSX is a syntax extension that allows you to write HTML-like code in JavaScript.'
        }
    ];

    return (
        <div style={{ maxWidth: '600px', padding: '20px' }}>
            <h2>FAQ</h2>
            <Accordion items={faqItems} />

            <h2 style={{ marginTop: '30px' }}>FAQ (Multiple Open)</h2>
            <Accordion items={faqItems} allowMultiple />
        </div>
    );
}
```

---

## Key Takeaways

1. **if/else outside JSX** - For complex logic or early returns
2. **Ternary for simple choices** - Inline if/else
3. **&& for conditional render** - When you only have one condition
4. **Avoid nested ternaries** - Use variables or functions instead
5. **Object lookup** - Clean way to map values to components
6. **Return null** - To render nothing
7. **CSS vs conditional** - Choose based on use case

---

## Self-Check Questions

1. When should you use if/else vs ternary operator?
2. What's the gotcha with using && for conditional rendering?
3. How do you conditionally add className?
4. When would you use CSS display:none vs conditional rendering?
5. How do you render nothing in React?
6. What's the object lookup pattern for conditional rendering?

---

**Next Lesson:** [Day 6 - Lists & Keys](./day-06-lists-keys.md)
