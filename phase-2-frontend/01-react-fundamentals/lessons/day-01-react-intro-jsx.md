# Day 1: React Introduction & JSX

## What is React?

React is a **JavaScript library for building user interfaces**, created by Facebook (Meta). It's the most popular frontend library in the world.

### Why React?

1. **Component-Based** - Build encapsulated components that manage their own state
2. **Declarative** - Describe what UI should look like, React handles the DOM updates
3. **Virtual DOM** - Efficient updates by minimizing actual DOM manipulation
4. **Huge Ecosystem** - Thousands of libraries, tools, and community support
5. **Job Market** - Most in-demand frontend skill

### React vs Vanilla JavaScript

```javascript
// Vanilla JavaScript - Imperative (how to do it)
const button = document.createElement('button');
button.textContent = 'Count: 0';
button.addEventListener('click', () => {
    const count = parseInt(button.textContent.split(': ')[1]) + 1;
    button.textContent = `Count: ${count}`;
});
document.body.appendChild(button);

// React - Declarative (what we want)
function Counter() {
    const [count, setCount] = useState(0);
    return (
        <button onClick={() => setCount(count + 1)}>
            Count: {count}
        </button>
    );
}
```

---

## Setting Up a React Project

### Using Vite (Recommended)

```bash
# Create new React project with Vite
npm create vite@latest my-react-app -- --template react

# With TypeScript
npm create vite@latest my-react-app -- --template react-ts

# Navigate and install
cd my-react-app
npm install

# Start development server
npm run dev
```

### Using Create React App (Legacy)

```bash
# Create React App (slower, more config)
npx create-react-app my-react-app

# With TypeScript
npx create-react-app my-react-app --template typescript

cd my-react-app
npm start
```

### Project Structure (Vite)

```
my-react-app/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.css
│   ├── App.jsx          # Main component
│   ├── index.css
│   └── main.jsx         # Entry point
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

### Entry Point (main.jsx)

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

---

## Understanding JSX

JSX (JavaScript XML) is a syntax extension that lets you write HTML-like code in JavaScript.

### JSX is NOT HTML

```jsx
// JSX - looks like HTML but it's JavaScript
const element = <h1>Hello, World!</h1>;

// Compiles to:
const element = React.createElement('h1', null, 'Hello, World!');
```

### JSX Rules

#### 1. Return a Single Root Element

```jsx
// ❌ Wrong - multiple root elements
function App() {
    return (
        <h1>Title</h1>
        <p>Paragraph</p>
    );
}

// ✅ Correct - wrapped in div
function App() {
    return (
        <div>
            <h1>Title</h1>
            <p>Paragraph</p>
        </div>
    );
}

// ✅ Better - use Fragment (no extra DOM node)
function App() {
    return (
        <>
            <h1>Title</h1>
            <p>Paragraph</p>
        </>
    );
}

// ✅ Also valid - explicit Fragment
import { Fragment } from 'react';

function App() {
    return (
        <Fragment>
            <h1>Title</h1>
            <p>Paragraph</p>
        </Fragment>
    );
}
```

#### 2. Close All Tags

```jsx
// ❌ Wrong
<img src="photo.jpg">
<input type="text">
<br>

// ✅ Correct - self-closing tags
<img src="photo.jpg" />
<input type="text" />
<br />
```

#### 3. Use camelCase for Attributes

```jsx
// HTML attributes become camelCase
<div className="container">          {/* class → className */}
<label htmlFor="name">               {/* for → htmlFor */}
<button onClick={handleClick}>       {/* onclick → onClick */}
<input onChange={handleChange} />    {/* onchange → onChange */}
<div tabIndex={0}>                   {/* tabindex → tabIndex */}
<svg strokeWidth={2}>                {/* stroke-width → strokeWidth */}
```

#### 4. JavaScript Expressions in Curly Braces

```jsx
function Greeting() {
    const name = "John";
    const age = 25;
    const isLoggedIn = true;

    return (
        <div>
            {/* Variables */}
            <h1>Hello, {name}!</h1>

            {/* Expressions */}
            <p>Age: {age}</p>
            <p>Next year: {age + 1}</p>

            {/* Function calls */}
            <p>Uppercase: {name.toUpperCase()}</p>

            {/* Ternary operator */}
            <p>{isLoggedIn ? 'Welcome back!' : 'Please log in'}</p>

            {/* Template literals */}
            <p>{`${name} is ${age} years old`}</p>
        </div>
    );
}
```

#### 5. Inline Styles as Objects

```jsx
// ❌ Wrong - string style like HTML
<div style="color: red; font-size: 16px;">

// ✅ Correct - object with camelCase properties
<div style={{ color: 'red', fontSize: '16px' }}>

// ✅ Better - extract to variable
function StyledComponent() {
    const styles = {
        container: {
            backgroundColor: '#f0f0f0',
            padding: '20px',
            borderRadius: '8px'
        },
        title: {
            color: '#333',
            fontSize: '24px',
            marginBottom: '10px'
        }
    };

    return (
        <div style={styles.container}>
            <h1 style={styles.title}>Styled Title</h1>
        </div>
    );
}
```

---

## Creating Components

### Function Components

```jsx
// Basic function component
function Welcome() {
    return <h1>Welcome to React!</h1>;
}

// Arrow function component
const Welcome = () => {
    return <h1>Welcome to React!</h1>;
};

// Arrow function with implicit return
const Welcome = () => <h1>Welcome to React!</h1>;
```

### Component Naming Rules

```jsx
// ✅ Components must start with uppercase
function MyComponent() { ... }
function Header() { ... }
function UserProfile() { ... }

// ❌ Lowercase is treated as HTML element
function myComponent() { ... }  // Won't work as component
```

### Using Components

```jsx
// App.jsx
import Header from './Header';
import Footer from './Footer';

function App() {
    return (
        <div>
            <Header />
            <main>
                <h1>Main Content</h1>
            </main>
            <Footer />
        </div>
    );
}

export default App;
```

### Component File Structure

```jsx
// Header.jsx
function Header() {
    return (
        <header>
            <nav>
                <a href="/">Home</a>
                <a href="/about">About</a>
                <a href="/contact">Contact</a>
            </nav>
        </header>
    );
}

export default Header;

// Or named export
export function Header() {
    return (
        <header>...</header>
    );
}

// Import named export
import { Header } from './Header';
```

---

## Rendering Lists

### Using map()

```jsx
function FruitList() {
    const fruits = ['Apple', 'Banana', 'Cherry', 'Date'];

    return (
        <ul>
            {fruits.map((fruit, index) => (
                <li key={index}>{fruit}</li>
            ))}
        </ul>
    );
}
```

### The key Prop

```jsx
// ❌ Using index as key (can cause issues with reordering)
{items.map((item, index) => (
    <li key={index}>{item.name}</li>
))}

// ✅ Using unique ID as key
{items.map(item => (
    <li key={item.id}>{item.name}</li>
))}
```

### Rendering Objects

```jsx
function UserList() {
    const users = [
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' },
        { id: 3, name: 'Bob', email: 'bob@example.com' }
    ];

    return (
        <div>
            <h2>Users</h2>
            <ul>
                {users.map(user => (
                    <li key={user.id}>
                        <strong>{user.name}</strong>
                        <span> - {user.email}</span>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

### Conditional Rendering in Lists

```jsx
function TaskList() {
    const tasks = [
        { id: 1, text: 'Learn React', completed: true },
        { id: 2, text: 'Build project', completed: false },
        { id: 3, text: 'Deploy app', completed: false }
    ];

    return (
        <ul>
            {tasks.map(task => (
                <li
                    key={task.id}
                    style={{
                        textDecoration: task.completed ? 'line-through' : 'none'
                    }}
                >
                    {task.text}
                </li>
            ))}
        </ul>
    );
}
```

---

## Conditional Rendering

### Using Ternary Operator

```jsx
function Greeting({ isLoggedIn }) {
    return (
        <div>
            {isLoggedIn ? (
                <h1>Welcome back!</h1>
            ) : (
                <h1>Please sign in</h1>
            )}
        </div>
    );
}
```

### Using && (Logical AND)

```jsx
function Notification({ messages }) {
    return (
        <div>
            <h1>Messages</h1>
            {messages.length > 0 && (
                <p>You have {messages.length} new messages</p>
            )}
        </div>
    );
}

// ⚠️ Watch out for falsy values
function Counter({ count }) {
    return (
        <div>
            {/* ❌ Will render "0" when count is 0 */}
            {count && <p>Count: {count}</p>}

            {/* ✅ Correct - explicit boolean check */}
            {count > 0 && <p>Count: {count}</p>}
        </div>
    );
}
```

### Using if/else (Outside JSX)

```jsx
function StatusBadge({ status }) {
    let badge;

    if (status === 'active') {
        badge = <span className="badge green">Active</span>;
    } else if (status === 'pending') {
        badge = <span className="badge yellow">Pending</span>;
    } else {
        badge = <span className="badge red">Inactive</span>;
    }

    return <div>{badge}</div>;
}
```

### Switch-like Patterns

```jsx
function Icon({ type }) {
    const icons = {
        success: '✅',
        error: '❌',
        warning: '⚠️',
        info: 'ℹ️'
    };

    return <span>{icons[type] || '❓'}</span>;
}

// Or with a function
function getStatusComponent(status) {
    switch (status) {
        case 'loading':
            return <LoadingSpinner />;
        case 'error':
            return <ErrorMessage />;
        case 'success':
            return <SuccessMessage />;
        default:
            return null;
    }
}

function Status({ status }) {
    return <div>{getStatusComponent(status)}</div>;
}
```

---

## Comments in JSX

```jsx
function App() {
    return (
        <div>
            {/* This is a JSX comment */}
            <h1>Hello</h1>

            {/*
                Multi-line
                JSX comment
            */}
            <p>World</p>

            {/* You can also comment out elements */}
            {/* <OldComponent /> */}
        </div>
    );
}

// Regular JS comments work outside JSX
// This is a regular comment
function MyComponent() {
    // This works too
    const x = 5; // And inline comments

    return <div>{x}</div>;
}
```

---

## Practice Exercises

### Exercise 1: Profile Card

```jsx
// Create a ProfileCard component that displays:
// - Name
// - Avatar image
// - Bio
// - Social links

function ProfileCard() {
    // Your code here
}
```

**Solution:**
```jsx
function ProfileCard() {
    const user = {
        name: 'Jane Developer',
        avatar: 'https://via.placeholder.com/150',
        bio: 'Full-stack developer passionate about React and TypeScript.',
        social: {
            twitter: 'https://twitter.com/jane',
            github: 'https://github.com/jane',
            linkedin: 'https://linkedin.com/in/jane'
        }
    };

    const styles = {
        card: {
            maxWidth: '300px',
            padding: '20px',
            borderRadius: '12px',
            boxShadow: '0 4px 6px rgba(0, 0, 0, 0.1)',
            textAlign: 'center'
        },
        avatar: {
            width: '100px',
            height: '100px',
            borderRadius: '50%',
            marginBottom: '15px'
        },
        name: {
            margin: '0 0 10px',
            fontSize: '1.5rem'
        },
        bio: {
            color: '#666',
            marginBottom: '15px'
        },
        links: {
            display: 'flex',
            justifyContent: 'center',
            gap: '15px'
        },
        link: {
            color: '#0066cc',
            textDecoration: 'none'
        }
    };

    return (
        <div style={styles.card}>
            <img
                src={user.avatar}
                alt={user.name}
                style={styles.avatar}
            />
            <h2 style={styles.name}>{user.name}</h2>
            <p style={styles.bio}>{user.bio}</p>
            <div style={styles.links}>
                <a href={user.social.twitter} style={styles.link}>Twitter</a>
                <a href={user.social.github} style={styles.link}>GitHub</a>
                <a href={user.social.linkedin} style={styles.link}>LinkedIn</a>
            </div>
        </div>
    );
}
```

### Exercise 2: Product List

```jsx
// Create a component that displays a list of products
// Each product should show: name, price, whether it's in stock

function ProductList() {
    const products = [
        { id: 1, name: 'Laptop', price: 999, inStock: true },
        { id: 2, name: 'Phone', price: 699, inStock: true },
        { id: 3, name: 'Tablet', price: 499, inStock: false },
        { id: 4, name: 'Watch', price: 299, inStock: true }
    ];

    // Your code here
}
```

**Solution:**
```jsx
function ProductList() {
    const products = [
        { id: 1, name: 'Laptop', price: 999, inStock: true },
        { id: 2, name: 'Phone', price: 699, inStock: true },
        { id: 3, name: 'Tablet', price: 499, inStock: false },
        { id: 4, name: 'Watch', price: 299, inStock: true }
    ];

    return (
        <div>
            <h1>Products</h1>
            <ul style={{ listStyle: 'none', padding: 0 }}>
                {products.map(product => (
                    <li
                        key={product.id}
                        style={{
                            padding: '15px',
                            marginBottom: '10px',
                            backgroundColor: '#f5f5f5',
                            borderRadius: '8px',
                            display: 'flex',
                            justifyContent: 'space-between',
                            alignItems: 'center',
                            opacity: product.inStock ? 1 : 0.5
                        }}
                    >
                        <span style={{ fontWeight: 'bold' }}>
                            {product.name}
                        </span>
                        <span>
                            ${product.price}
                        </span>
                        <span style={{
                            color: product.inStock ? 'green' : 'red'
                        }}>
                            {product.inStock ? 'In Stock' : 'Out of Stock'}
                        </span>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

### Exercise 3: Conditional Welcome

```jsx
// Create a component that shows different content based on user role
// - Admin sees admin dashboard link
// - User sees profile link
// - Guest sees login button

function WelcomeMessage({ user }) {
    // user can be: { name: 'John', role: 'admin' }
    // or: { name: 'Jane', role: 'user' }
    // or: null (guest)

    // Your code here
}
```

**Solution:**
```jsx
function WelcomeMessage({ user }) {
    if (!user) {
        return (
            <div>
                <h1>Welcome, Guest!</h1>
                <button>Log In</button>
                <button>Sign Up</button>
            </div>
        );
    }

    return (
        <div>
            <h1>Welcome, {user.name}!</h1>

            {user.role === 'admin' && (
                <nav>
                    <a href="/dashboard">Admin Dashboard</a>
                    <a href="/users">Manage Users</a>
                    <a href="/settings">Settings</a>
                </nav>
            )}

            {user.role === 'user' && (
                <nav>
                    <a href="/profile">My Profile</a>
                    <a href="/orders">My Orders</a>
                </nav>
            )}

            <button>Log Out</button>
        </div>
    );
}

// Usage
function App() {
    const admin = { name: 'John', role: 'admin' };
    const user = { name: 'Jane', role: 'user' };

    return (
        <div>
            <WelcomeMessage user={admin} />
            <WelcomeMessage user={user} />
            <WelcomeMessage user={null} />
        </div>
    );
}
```

---

## Key Takeaways

1. **React is declarative** - Describe what UI should look like
2. **JSX is JavaScript** - HTML-like syntax that compiles to JS
3. **Components are functions** - Return JSX, start with uppercase
4. **Single root element** - Use Fragment if needed
5. **camelCase attributes** - className, onClick, htmlFor
6. **Curly braces for JS** - Expressions, not statements
7. **Keys for lists** - Use unique IDs, not indexes

---

## Self-Check Questions

1. What is JSX and how does it differ from HTML?
2. Why must component names start with uppercase?
3. What's the difference between `class` and `className`?
4. How do you embed JavaScript expressions in JSX?
5. Why are keys important when rendering lists?
6. What are Fragments and when would you use them?

---

**Next Lesson:** [Day 2 - Components & Props](./day-02-components-props.md)
