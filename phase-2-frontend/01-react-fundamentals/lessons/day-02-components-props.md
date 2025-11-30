# Day 2: Components & Props

## Understanding Components

Components are the building blocks of React applications. They're reusable, self-contained pieces of UI.

### Component Hierarchy

```
App
├── Header
│   ├── Logo
│   └── Navigation
│       ├── NavLink
│       ├── NavLink
│       └── NavLink
├── Main
│   ├── Sidebar
│   └── Content
│       ├── ArticleCard
│       ├── ArticleCard
│       └── ArticleCard
└── Footer
    ├── FooterLinks
    └── Copyright
```

### Creating Components

```jsx
// Function component (preferred)
function Welcome() {
    return <h1>Welcome to React!</h1>;
}

// Arrow function component
const Welcome = () => {
    return <h1>Welcome to React!</h1>;
};

// With TypeScript
interface WelcomeProps {
    name: string;
}

const Welcome: React.FC<WelcomeProps> = ({ name }) => {
    return <h1>Welcome, {name}!</h1>;
};

// Or without React.FC
function Welcome({ name }: WelcomeProps) {
    return <h1>Welcome, {name}!</h1>;
}
```

---

## Understanding Props

Props (properties) are how you pass data from parent to child components. They are **read-only**.

### Basic Props

```jsx
// Parent component
function App() {
    return (
        <div>
            <Greeting name="Alice" />
            <Greeting name="Bob" />
            <Greeting name="Charlie" />
        </div>
    );
}

// Child component receiving props
function Greeting(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Destructuring props (preferred)
function Greeting({ name }) {
    return <h1>Hello, {name}!</h1>;
}
```

### Multiple Props

```jsx
function UserCard({ name, email, avatar, isOnline }) {
    return (
        <div className="user-card">
            <img src={avatar} alt={name} />
            <h2>{name}</h2>
            <p>{email}</p>
            <span className={isOnline ? 'online' : 'offline'}>
                {isOnline ? '🟢 Online' : '⚫ Offline'}
            </span>
        </div>
    );
}

// Usage
function App() {
    return (
        <UserCard
            name="John Doe"
            email="john@example.com"
            avatar="https://example.com/john.jpg"
            isOnline={true}
        />
    );
}
```

### Props with Different Data Types

```jsx
function DataDisplay({
    text,           // string
    count,          // number
    isActive,       // boolean
    items,          // array
    user,           // object
    onClick,        // function
    children        // React nodes
}) {
    return (
        <div>
            <p>Text: {text}</p>
            <p>Count: {count}</p>
            <p>Active: {isActive ? 'Yes' : 'No'}</p>
            <p>Items: {items.join(', ')}</p>
            <p>User: {user.name}</p>
            <button onClick={onClick}>Click Me</button>
            <div>{children}</div>
        </div>
    );
}

// Usage
function App() {
    const handleClick = () => console.log('Clicked!');

    return (
        <DataDisplay
            text="Hello"
            count={42}
            isActive={true}
            items={['apple', 'banana', 'cherry']}
            user={{ name: 'John', age: 30 }}
            onClick={handleClick}
        >
            <p>This is children content</p>
        </DataDisplay>
    );
}
```

---

## Default Props

### Using Default Parameters

```jsx
// Modern approach - default parameters
function Button({
    label = 'Click Me',
    variant = 'primary',
    size = 'medium',
    disabled = false
}) {
    return (
        <button
            className={`btn btn-${variant} btn-${size}`}
            disabled={disabled}
        >
            {label}
        </button>
    );
}

// Usage
<Button />                              // Uses all defaults
<Button label="Submit" />               // Custom label
<Button variant="secondary" size="large" />  // Custom variant and size
```

### Using defaultProps (Older Pattern)

```jsx
function Button({ label, variant, size, disabled }) {
    return (
        <button
            className={`btn btn-${variant} btn-${size}`}
            disabled={disabled}
        >
            {label}
        </button>
    );
}

Button.defaultProps = {
    label: 'Click Me',
    variant: 'primary',
    size: 'medium',
    disabled: false
};
```

---

## The Children Prop

The `children` prop allows you to pass content between component tags.

### Basic Children

```jsx
function Card({ children }) {
    return (
        <div className="card">
            {children}
        </div>
    );
}

// Usage
function App() {
    return (
        <Card>
            <h2>Card Title</h2>
            <p>This is card content.</p>
            <button>Learn More</button>
        </Card>
    );
}
```

### Children with Other Props

```jsx
function Modal({ title, children, onClose }) {
    return (
        <div className="modal-overlay">
            <div className="modal">
                <div className="modal-header">
                    <h2>{title}</h2>
                    <button onClick={onClose}>×</button>
                </div>
                <div className="modal-body">
                    {children}
                </div>
            </div>
        </div>
    );
}

// Usage
function App() {
    return (
        <Modal title="Welcome" onClose={() => console.log('Close')}>
            <p>Welcome to our app!</p>
            <button>Get Started</button>
        </Modal>
    );
}
```

### Render Props Pattern

```jsx
function DataFetcher({ url, render }) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetch(url)
            .then(res => res.json())
            .then(data => {
                setData(data);
                setLoading(false);
            });
    }, [url]);

    return render({ data, loading });
}

// Usage
function App() {
    return (
        <DataFetcher
            url="/api/users"
            render={({ data, loading }) => {
                if (loading) return <p>Loading...</p>;
                return (
                    <ul>
                        {data.map(user => (
                            <li key={user.id}>{user.name}</li>
                        ))}
                    </ul>
                );
            }}
        />
    );
}
```

---

## Prop Validation with PropTypes

```jsx
import PropTypes from 'prop-types';

function UserProfile({ name, age, email, avatar, hobbies, onUpdate }) {
    return (
        <div>
            <img src={avatar} alt={name} />
            <h2>{name}</h2>
            <p>Age: {age}</p>
            <p>Email: {email}</p>
            <ul>
                {hobbies.map((hobby, i) => (
                    <li key={i}>{hobby}</li>
                ))}
            </ul>
            <button onClick={onUpdate}>Update</button>
        </div>
    );
}

UserProfile.propTypes = {
    // Required string
    name: PropTypes.string.isRequired,

    // Optional number
    age: PropTypes.number,

    // Required string matching pattern
    email: PropTypes.string.isRequired,

    // Optional string with default
    avatar: PropTypes.string,

    // Array of strings
    hobbies: PropTypes.arrayOf(PropTypes.string),

    // Required function
    onUpdate: PropTypes.func.isRequired,

    // Object with specific shape
    address: PropTypes.shape({
        street: PropTypes.string,
        city: PropTypes.string,
        zip: PropTypes.string
    }),

    // One of specific values
    status: PropTypes.oneOf(['active', 'inactive', 'pending']),

    // One of specific types
    id: PropTypes.oneOfType([
        PropTypes.string,
        PropTypes.number
    ]),

    // Any renderable content
    children: PropTypes.node,

    // React element
    icon: PropTypes.element
};

UserProfile.defaultProps = {
    age: 0,
    avatar: '/default-avatar.png',
    hobbies: []
};
```

### Common PropTypes

```jsx
PropTypes.string          // String
PropTypes.number          // Number
PropTypes.bool            // Boolean
PropTypes.array           // Array
PropTypes.object          // Object
PropTypes.func            // Function
PropTypes.node            // Anything renderable
PropTypes.element         // React element
PropTypes.symbol          // Symbol
PropTypes.any             // Any type

// Modifiers
PropTypes.string.isRequired   // Required
PropTypes.arrayOf(PropTypes.number)   // Array of numbers
PropTypes.objectOf(PropTypes.string)  // Object with string values
PropTypes.shape({ ... })      // Object with specific shape
PropTypes.exact({ ... })      // Object with exact shape
PropTypes.oneOf(['a', 'b'])   // One of values
PropTypes.oneOfType([...])    // One of types
PropTypes.instanceOf(Class)   // Instance of class
```

---

## Props with TypeScript

```tsx
// Interface for props
interface ButtonProps {
    label: string;
    variant?: 'primary' | 'secondary' | 'danger';
    size?: 'small' | 'medium' | 'large';
    disabled?: boolean;
    onClick?: () => void;
}

function Button({
    label,
    variant = 'primary',
    size = 'medium',
    disabled = false,
    onClick
}: ButtonProps) {
    return (
        <button
            className={`btn btn-${variant} btn-${size}`}
            disabled={disabled}
            onClick={onClick}
        >
            {label}
        </button>
    );
}

// With children
interface CardProps {
    title: string;
    children: React.ReactNode;
    footer?: React.ReactNode;
}

function Card({ title, children, footer }: CardProps) {
    return (
        <div className="card">
            <h2>{title}</h2>
            <div className="card-body">{children}</div>
            {footer && <div className="card-footer">{footer}</div>}
        </div>
    );
}

// Generic props
interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
    return (
        <ul>
            {items.map((item, index) => (
                <li key={index}>{renderItem(item)}</li>
            ))}
        </ul>
    );
}

// Usage
<List
    items={['Apple', 'Banana', 'Cherry']}
    renderItem={(item) => <span>{item}</span>}
/>
```

---

## Component Composition

### Basic Composition

```jsx
function Page() {
    return (
        <Layout>
            <Header />
            <Sidebar />
            <Content>
                <Article title="Hello World" />
                <Article title="React is Great" />
            </Content>
            <Footer />
        </Layout>
    );
}

function Layout({ children }) {
    return <div className="layout">{children}</div>;
}

function Content({ children }) {
    return <main className="content">{children}</main>;
}
```

### Specialized Components

```jsx
// Generic Dialog component
function Dialog({ title, message, children }) {
    return (
        <div className="dialog">
            <h2>{title}</h2>
            <p>{message}</p>
            {children}
        </div>
    );
}

// Specialized Welcome Dialog
function WelcomeDialog() {
    return (
        <Dialog
            title="Welcome!"
            message="Thank you for visiting our site."
        >
            <button>Get Started</button>
        </Dialog>
    );
}

// Specialized Error Dialog
function ErrorDialog({ error, onRetry }) {
    return (
        <Dialog
            title="Error"
            message={error.message}
        >
            <button onClick={onRetry}>Retry</button>
        </Dialog>
    );
}
```

### Containment Pattern

```jsx
function SplitPane({ left, right }) {
    return (
        <div className="split-pane">
            <div className="left-pane">{left}</div>
            <div className="right-pane">{right}</div>
        </div>
    );
}

// Usage
function App() {
    return (
        <SplitPane
            left={<ContactList />}
            right={<ChatWindow />}
        />
    );
}
```

---

## Passing Props Down (Prop Drilling)

```jsx
// Prop drilling example - passing data through multiple levels
function App() {
    const user = { name: 'John', role: 'admin' };

    return (
        <Dashboard user={user} />
    );
}

function Dashboard({ user }) {
    return (
        <div>
            <Header user={user} />
            <Content user={user} />
        </div>
    );
}

function Header({ user }) {
    return (
        <header>
            <UserMenu user={user} />
        </header>
    );
}

function UserMenu({ user }) {
    return (
        <div>
            <span>{user.name}</span>
            {user.role === 'admin' && <span>👑</span>}
        </div>
    );
}
```

### Avoiding Prop Drilling

```jsx
// Solution 1: Component composition
function App() {
    const user = { name: 'John', role: 'admin' };

    return (
        <Dashboard>
            <Header>
                <UserMenu user={user} />
            </Header>
            <Content />
        </Dashboard>
    );
}

// Solution 2: Context API (covered later)
// Solution 3: State management library
```

---

## Spreading Props

```jsx
// Spread operator for passing props
function Button(props) {
    return <button {...props} />;
}

// Usage
<Button onClick={handleClick} disabled={true} className="primary">
    Click Me
</Button>

// Extracting some props, spreading rest
function CustomInput({ label, error, ...inputProps }) {
    return (
        <div className="form-field">
            <label>{label}</label>
            <input {...inputProps} />
            {error && <span className="error">{error}</span>}
        </div>
    );
}

// Usage
<CustomInput
    label="Email"
    type="email"
    placeholder="Enter email"
    value={email}
    onChange={handleChange}
    error={emailError}
/>
```

---

## Practice Exercises

### Exercise 1: Reusable Button Component

```jsx
// Create a Button component with these props:
// - variant: 'primary' | 'secondary' | 'danger' | 'ghost'
// - size: 'small' | 'medium' | 'large'
// - disabled: boolean
// - loading: boolean
// - leftIcon: React node
// - rightIcon: React node
// - children: button text

// Your code here
```

**Solution:**
```jsx
function Button({
    variant = 'primary',
    size = 'medium',
    disabled = false,
    loading = false,
    leftIcon,
    rightIcon,
    children,
    onClick
}) {
    const baseStyles = {
        display: 'inline-flex',
        alignItems: 'center',
        justifyContent: 'center',
        gap: '8px',
        border: 'none',
        borderRadius: '6px',
        cursor: disabled || loading ? 'not-allowed' : 'pointer',
        opacity: disabled ? 0.5 : 1,
        fontWeight: '500',
        transition: 'all 0.2s'
    };

    const sizeStyles = {
        small: { padding: '6px 12px', fontSize: '12px' },
        medium: { padding: '10px 20px', fontSize: '14px' },
        large: { padding: '14px 28px', fontSize: '16px' }
    };

    const variantStyles = {
        primary: { backgroundColor: '#0066cc', color: 'white' },
        secondary: { backgroundColor: '#e0e0e0', color: '#333' },
        danger: { backgroundColor: '#dc3545', color: 'white' },
        ghost: { backgroundColor: 'transparent', color: '#0066cc', border: '1px solid #0066cc' }
    };

    const style = {
        ...baseStyles,
        ...sizeStyles[size],
        ...variantStyles[variant]
    };

    return (
        <button
            style={style}
            disabled={disabled || loading}
            onClick={onClick}
        >
            {loading ? (
                <span>Loading...</span>
            ) : (
                <>
                    {leftIcon && <span>{leftIcon}</span>}
                    {children}
                    {rightIcon && <span>{rightIcon}</span>}
                </>
            )}
        </button>
    );
}

// Usage
function App() {
    return (
        <div style={{ display: 'flex', gap: '10px', flexWrap: 'wrap' }}>
            <Button>Default</Button>
            <Button variant="secondary">Secondary</Button>
            <Button variant="danger" size="large">Delete</Button>
            <Button variant="ghost" size="small">Cancel</Button>
            <Button loading>Loading</Button>
            <Button disabled>Disabled</Button>
            <Button leftIcon="📧">Email</Button>
            <Button rightIcon="→">Next</Button>
        </div>
    );
}
```

### Exercise 2: Card Component with Slots

```jsx
// Create a Card component with named slots:
// - header
// - children (body)
// - footer
// - image (optional)

// Your code here
```

**Solution:**
```jsx
function Card({
    header,
    children,
    footer,
    image,
    variant = 'elevated'
}) {
    const cardStyles = {
        elevated: {
            boxShadow: '0 4px 6px rgba(0,0,0,0.1)',
            border: 'none'
        },
        outlined: {
            boxShadow: 'none',
            border: '1px solid #e0e0e0'
        },
        flat: {
            boxShadow: 'none',
            border: 'none',
            backgroundColor: '#f5f5f5'
        }
    };

    const styles = {
        card: {
            borderRadius: '12px',
            overflow: 'hidden',
            backgroundColor: 'white',
            ...cardStyles[variant]
        },
        image: {
            width: '100%',
            height: '200px',
            objectFit: 'cover'
        },
        header: {
            padding: '16px',
            borderBottom: '1px solid #eee'
        },
        body: {
            padding: '16px'
        },
        footer: {
            padding: '16px',
            borderTop: '1px solid #eee',
            backgroundColor: '#fafafa'
        }
    };

    return (
        <div style={styles.card}>
            {image && (
                <img src={image} alt="" style={styles.image} />
            )}
            {header && (
                <div style={styles.header}>{header}</div>
            )}
            <div style={styles.body}>{children}</div>
            {footer && (
                <div style={styles.footer}>{footer}</div>
            )}
        </div>
    );
}

// Usage
function App() {
    return (
        <div style={{ maxWidth: '400px', margin: '20px' }}>
            <Card
                image="https://via.placeholder.com/400x200"
                header={<h3 style={{ margin: 0 }}>Card Title</h3>}
                footer={
                    <div style={{ display: 'flex', gap: '10px' }}>
                        <button>Cancel</button>
                        <button>Confirm</button>
                    </div>
                }
            >
                <p>This is the card body content. It can contain any React elements.</p>
            </Card>
        </div>
    );
}
```

### Exercise 3: List Component

```jsx
// Create a generic List component that:
// - Accepts items array
// - Has renderItem function for custom rendering
// - Shows empty state when no items
// - Has optional loading state

// Your code here
```

**Solution:**
```jsx
function List({
    items = [],
    renderItem,
    loading = false,
    emptyMessage = 'No items to display',
    keyExtractor = (item, index) => index
}) {
    if (loading) {
        return (
            <div style={{ padding: '40px', textAlign: 'center' }}>
                <div>Loading...</div>
            </div>
        );
    }

    if (items.length === 0) {
        return (
            <div style={{
                padding: '40px',
                textAlign: 'center',
                color: '#666'
            }}>
                {emptyMessage}
            </div>
        );
    }

    return (
        <ul style={{ listStyle: 'none', padding: 0, margin: 0 }}>
            {items.map((item, index) => (
                <li key={keyExtractor(item, index)}>
                    {renderItem(item, index)}
                </li>
            ))}
        </ul>
    );
}

// Usage examples
function App() {
    const users = [
        { id: 1, name: 'Alice', email: 'alice@example.com' },
        { id: 2, name: 'Bob', email: 'bob@example.com' },
        { id: 3, name: 'Charlie', email: 'charlie@example.com' }
    ];

    return (
        <div>
            <h2>Users</h2>
            <List
                items={users}
                keyExtractor={(user) => user.id}
                renderItem={(user) => (
                    <div style={{
                        padding: '12px',
                        marginBottom: '8px',
                        backgroundColor: '#f5f5f5',
                        borderRadius: '8px'
                    }}>
                        <strong>{user.name}</strong>
                        <span style={{ marginLeft: '10px', color: '#666' }}>
                            {user.email}
                        </span>
                    </div>
                )}
            />

            <h2>Empty List</h2>
            <List
                items={[]}
                emptyMessage="No users found"
                renderItem={(item) => <div>{item}</div>}
            />

            <h2>Loading List</h2>
            <List
                items={[]}
                loading={true}
                renderItem={(item) => <div>{item}</div>}
            />
        </div>
    );
}
```

---

## Key Takeaways

1. **Props are read-only** - Never modify props directly
2. **Destructure props** - Cleaner and easier to read
3. **Use default values** - Provide sensible defaults
4. **Children is special** - Content between tags
5. **Validate with PropTypes** - Or use TypeScript
6. **Composition over inheritance** - Build complex UIs from simple pieces
7. **Spread carefully** - Useful but can pass unwanted props

---

## Self-Check Questions

1. What are props and how are they different from state?
2. How do you set default values for props?
3. What is the children prop used for?
4. How do you validate props in React?
5. What is prop drilling and how can you avoid it?
6. What does the spread operator do when used with props?

---

**Next Lesson:** [Day 3 - State & Hooks](./day-03-state-hooks.md)
