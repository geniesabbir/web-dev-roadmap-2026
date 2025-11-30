# Day 3: State & Hooks

## Understanding State

State is data that changes over time and affects what your component renders. Unlike props, state is:
- **Owned by the component** - Created and managed internally
- **Mutable** - Can be updated (through setState)
- **Triggers re-render** - UI updates when state changes

### State vs Props

```jsx
// Props - passed from parent, read-only
function Greeting({ name }) {
    // name = "New Name";  ❌ Can't modify props
    return <h1>Hello, {name}</h1>;
}

// State - owned by component, can change
function Counter() {
    const [count, setCount] = useState(0);

    // setCount(count + 1);  ✅ Can update state
    return <p>Count: {count}</p>;
}
```

---

## The useState Hook

### Basic useState

```jsx
import { useState } from 'react';

function Counter() {
    // useState returns [currentValue, setterFunction]
    const [count, setCount] = useState(0);  // 0 is initial value

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </div>
    );
}
```

### useState with Different Types

```jsx
function StateExamples() {
    // Number
    const [count, setCount] = useState(0);

    // String
    const [name, setName] = useState('');

    // Boolean
    const [isOpen, setIsOpen] = useState(false);

    // Array
    const [items, setItems] = useState([]);

    // Object
    const [user, setUser] = useState({
        name: '',
        email: '',
        age: 0
    });

    // null/undefined for loading states
    const [data, setData] = useState(null);

    return <div>...</div>;
}
```

### Updating State

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    // Direct value
    const reset = () => setCount(0);

    // Functional update (use previous state)
    const increment = () => setCount(prev => prev + 1);
    const decrement = () => setCount(prev => prev - 1);

    // ⚠️ Why functional updates matter:
    const incrementThree = () => {
        // ❌ Wrong - all three use same stale count value
        setCount(count + 1);
        setCount(count + 1);
        setCount(count + 1);
        // Result: only increments by 1!

        // ✅ Correct - each uses latest value
        setCount(prev => prev + 1);
        setCount(prev => prev + 1);
        setCount(prev => prev + 1);
        // Result: increments by 3
    };

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>+</button>
            <button onClick={decrement}>-</button>
            <button onClick={reset}>Reset</button>
        </div>
    );
}
```

### Updating Objects in State

```jsx
function UserForm() {
    const [user, setUser] = useState({
        name: '',
        email: '',
        address: {
            city: '',
            country: ''
        }
    });

    // ❌ Wrong - mutating state directly
    const updateNameWrong = (name) => {
        user.name = name;  // Don't do this!
        setUser(user);     // Won't trigger re-render
    };

    // ✅ Correct - create new object
    const updateName = (name) => {
        setUser({ ...user, name });
    };

    // ✅ Updating nested object
    const updateCity = (city) => {
        setUser({
            ...user,
            address: {
                ...user.address,
                city
            }
        });
    };

    return (
        <form>
            <input
                value={user.name}
                onChange={(e) => updateName(e.target.value)}
                placeholder="Name"
            />
            <input
                value={user.email}
                onChange={(e) => setUser({ ...user, email: e.target.value })}
                placeholder="Email"
            />
            <input
                value={user.address.city}
                onChange={(e) => updateCity(e.target.value)}
                placeholder="City"
            />
        </form>
    );
}
```

### Updating Arrays in State

```jsx
function TodoList() {
    const [todos, setTodos] = useState([
        { id: 1, text: 'Learn React', done: false },
        { id: 2, text: 'Build project', done: false }
    ]);

    // Add item
    const addTodo = (text) => {
        const newTodo = {
            id: Date.now(),
            text,
            done: false
        };
        setTodos([...todos, newTodo]);  // Add at end
        // setTodos([newTodo, ...todos]);  // Add at beginning
    };

    // Remove item
    const removeTodo = (id) => {
        setTodos(todos.filter(todo => todo.id !== id));
    };

    // Update item
    const toggleTodo = (id) => {
        setTodos(todos.map(todo =>
            todo.id === id
                ? { ...todo, done: !todo.done }
                : todo
        ));
    };

    // Replace item
    const updateTodoText = (id, newText) => {
        setTodos(todos.map(todo =>
            todo.id === id
                ? { ...todo, text: newText }
                : todo
        ));
    };

    // Insert at index
    const insertAt = (index, newTodo) => {
        setTodos([
            ...todos.slice(0, index),
            newTodo,
            ...todos.slice(index)
        ]);
    };

    return (
        <ul>
            {todos.map(todo => (
                <li key={todo.id}>
                    <input
                        type="checkbox"
                        checked={todo.done}
                        onChange={() => toggleTodo(todo.id)}
                    />
                    <span style={{
                        textDecoration: todo.done ? 'line-through' : 'none'
                    }}>
                        {todo.text}
                    </span>
                    <button onClick={() => removeTodo(todo.id)}>
                        Delete
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

---

## Lazy Initialization

For expensive initial values, use a function:

```jsx
// ❌ This runs on every render
function ExpensiveComponent() {
    const [data, setData] = useState(
        expensiveComputation()  // Runs every time component renders
    );
}

// ✅ This runs only once on mount
function ExpensiveComponent() {
    const [data, setData] = useState(() => {
        return expensiveComputation();  // Only runs on initial render
    });
}

// Common use case: localStorage
function useLocalStorageState(key, initialValue) {
    const [value, setValue] = useState(() => {
        const stored = localStorage.getItem(key);
        return stored ? JSON.parse(stored) : initialValue;
    });

    useEffect(() => {
        localStorage.setItem(key, JSON.stringify(value));
    }, [key, value]);

    return [value, setValue];
}
```

---

## Multiple State Variables

### Separate useState Calls (Preferred)

```jsx
function UserProfile() {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    const [age, setAge] = useState(0);
    const [isEditing, setIsEditing] = useState(false);

    // Each can be updated independently
    return (
        <div>
            <input value={name} onChange={e => setName(e.target.value)} />
            <input value={email} onChange={e => setEmail(e.target.value)} />
            <input
                type="number"
                value={age}
                onChange={e => setAge(Number(e.target.value))}
            />
            <button onClick={() => setIsEditing(!isEditing)}>
                {isEditing ? 'Save' : 'Edit'}
            </button>
        </div>
    );
}
```

### Single Object State

```jsx
function UserProfile() {
    const [form, setForm] = useState({
        name: '',
        email: '',
        age: 0
    });

    // Generic handler
    const handleChange = (e) => {
        const { name, value, type } = e.target;
        setForm(prev => ({
            ...prev,
            [name]: type === 'number' ? Number(value) : value
        }));
    };

    return (
        <div>
            <input
                name="name"
                value={form.name}
                onChange={handleChange}
            />
            <input
                name="email"
                value={form.email}
                onChange={handleChange}
            />
            <input
                name="age"
                type="number"
                value={form.age}
                onChange={handleChange}
            />
        </div>
    );
}
```

### When to Group State

```jsx
// ✅ Group related values that change together
const [position, setPosition] = useState({ x: 0, y: 0 });

// ✅ Separate unrelated values
const [name, setName] = useState('');
const [theme, setTheme] = useState('light');

// ✅ Group form fields
const [formData, setFormData] = useState({
    username: '',
    password: '',
    rememberMe: false
});

// ❌ Don't over-group unrelated things
const [everything, setEverything] = useState({
    user: null,
    theme: 'light',
    sidebarOpen: false,
    notifications: []
});  // Hard to manage!
```

---

## State Lifting

When multiple components need to share state, "lift" it to their common parent.

```jsx
// Before: Each component has its own state
function TemperatureInput() {
    const [temp, setTemp] = useState('');
    return <input value={temp} onChange={e => setTemp(e.target.value)} />;
}

// Problem: Celsius and Fahrenheit can't sync!
function Calculator() {
    return (
        <div>
            <TemperatureInput label="Celsius" />
            <TemperatureInput label="Fahrenheit" />
        </div>
    );
}
```

```jsx
// After: State lifted to parent
function TemperatureInput({ temp, onTempChange, label }) {
    return (
        <div>
            <label>{label}</label>
            <input
                value={temp}
                onChange={e => onTempChange(e.target.value)}
            />
        </div>
    );
}

function Calculator() {
    const [celsius, setCelsius] = useState('');

    const fahrenheit = celsius
        ? ((parseFloat(celsius) * 9/5) + 32).toFixed(2)
        : '';

    const handleCelsiusChange = (value) => {
        setCelsius(value);
    };

    const handleFahrenheitChange = (value) => {
        const c = ((parseFloat(value) - 32) * 5/9).toFixed(2);
        setCelsius(isNaN(c) ? '' : c);
    };

    return (
        <div>
            <TemperatureInput
                label="Celsius"
                temp={celsius}
                onTempChange={handleCelsiusChange}
            />
            <TemperatureInput
                label="Fahrenheit"
                temp={fahrenheit}
                onTempChange={handleFahrenheitChange}
            />
        </div>
    );
}
```

---

## Controlled vs Uncontrolled Components

### Controlled Component

React controls the input value through state.

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

// Benefits:
// - Single source of truth
// - Instant access to value
// - Can validate/transform on every keystroke
// - Can programmatically set value
```

### Uncontrolled Component

DOM handles the input value, React reads it when needed.

```jsx
function UncontrolledInput() {
    const inputRef = useRef(null);

    const handleSubmit = (e) => {
        e.preventDefault();
        console.log(inputRef.current.value);
    };

    return (
        <form onSubmit={handleSubmit}>
            <input ref={inputRef} defaultValue="initial" />
            <button type="submit">Submit</button>
        </form>
    );
}

// Use cases:
// - File inputs (always uncontrolled)
// - Integration with non-React libraries
// - Simple forms without validation
```

### File Inputs (Always Uncontrolled)

```jsx
function FileUpload() {
    const fileInputRef = useRef(null);
    const [selectedFile, setSelectedFile] = useState(null);

    const handleFileChange = (e) => {
        const file = e.target.files[0];
        setSelectedFile(file);
    };

    return (
        <div>
            <input
                type="file"
                ref={fileInputRef}
                onChange={handleFileChange}
            />
            {selectedFile && (
                <p>Selected: {selectedFile.name}</p>
            )}
        </div>
    );
}
```

---

## Common Patterns

### Toggle State

```jsx
function Toggle() {
    const [isOn, setIsOn] = useState(false);

    const toggle = () => setIsOn(prev => !prev);

    return (
        <button onClick={toggle}>
            {isOn ? 'ON' : 'OFF'}
        </button>
    );
}
```

### Counter with Min/Max

```jsx
function Counter({ min = 0, max = 100, step = 1 }) {
    const [count, setCount] = useState(min);

    const increment = () => {
        setCount(prev => Math.min(prev + step, max));
    };

    const decrement = () => {
        setCount(prev => Math.max(prev - step, min));
    };

    return (
        <div>
            <button onClick={decrement} disabled={count <= min}>-</button>
            <span>{count}</span>
            <button onClick={increment} disabled={count >= max}>+</button>
        </div>
    );
}
```

### Form with Validation

```jsx
function SignupForm() {
    const [formData, setFormData] = useState({
        email: '',
        password: '',
        confirmPassword: ''
    });

    const [errors, setErrors] = useState({});
    const [isSubmitting, setIsSubmitting] = useState(false);

    const validate = () => {
        const newErrors = {};

        if (!formData.email) {
            newErrors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
            newErrors.email = 'Invalid email format';
        }

        if (!formData.password) {
            newErrors.password = 'Password is required';
        } else if (formData.password.length < 8) {
            newErrors.password = 'Password must be at least 8 characters';
        }

        if (formData.password !== formData.confirmPassword) {
            newErrors.confirmPassword = 'Passwords do not match';
        }

        setErrors(newErrors);
        return Object.keys(newErrors).length === 0;
    };

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({ ...prev, [name]: value }));
        // Clear error when user starts typing
        if (errors[name]) {
            setErrors(prev => ({ ...prev, [name]: '' }));
        }
    };

    const handleSubmit = async (e) => {
        e.preventDefault();

        if (!validate()) return;

        setIsSubmitting(true);
        try {
            // Submit form...
            console.log('Form submitted:', formData);
        } finally {
            setIsSubmitting(false);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <div>
                <input
                    name="email"
                    type="email"
                    value={formData.email}
                    onChange={handleChange}
                    placeholder="Email"
                />
                {errors.email && <span className="error">{errors.email}</span>}
            </div>

            <div>
                <input
                    name="password"
                    type="password"
                    value={formData.password}
                    onChange={handleChange}
                    placeholder="Password"
                />
                {errors.password && <span className="error">{errors.password}</span>}
            </div>

            <div>
                <input
                    name="confirmPassword"
                    type="password"
                    value={formData.confirmPassword}
                    onChange={handleChange}
                    placeholder="Confirm Password"
                />
                {errors.confirmPassword && (
                    <span className="error">{errors.confirmPassword}</span>
                )}
            </div>

            <button type="submit" disabled={isSubmitting}>
                {isSubmitting ? 'Signing up...' : 'Sign Up'}
            </button>
        </form>
    );
}
```

---

## Practice Exercises

### Exercise 1: Shopping Cart

```jsx
// Create a shopping cart with:
// - List of products
// - Add to cart button
// - Cart showing items and total
// - Remove from cart
// - Update quantity

// Your code here
```

**Solution:**
```jsx
function ShoppingCart() {
    const products = [
        { id: 1, name: 'Laptop', price: 999 },
        { id: 2, name: 'Phone', price: 699 },
        { id: 3, name: 'Headphones', price: 199 },
        { id: 4, name: 'Watch', price: 299 }
    ];

    const [cart, setCart] = useState([]);

    const addToCart = (product) => {
        setCart(prev => {
            const existing = prev.find(item => item.id === product.id);
            if (existing) {
                return prev.map(item =>
                    item.id === product.id
                        ? { ...item, quantity: item.quantity + 1 }
                        : item
                );
            }
            return [...prev, { ...product, quantity: 1 }];
        });
    };

    const removeFromCart = (productId) => {
        setCart(prev => prev.filter(item => item.id !== productId));
    };

    const updateQuantity = (productId, quantity) => {
        if (quantity < 1) {
            removeFromCart(productId);
            return;
        }
        setCart(prev => prev.map(item =>
            item.id === productId
                ? { ...item, quantity }
                : item
        ));
    };

    const total = cart.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
    );

    return (
        <div style={{ display: 'flex', gap: '40px' }}>
            <div>
                <h2>Products</h2>
                {products.map(product => (
                    <div key={product.id} style={{
                        padding: '10px',
                        marginBottom: '10px',
                        border: '1px solid #ddd',
                        borderRadius: '8px'
                    }}>
                        <span>{product.name} - ${product.price}</span>
                        <button
                            onClick={() => addToCart(product)}
                            style={{ marginLeft: '10px' }}
                        >
                            Add to Cart
                        </button>
                    </div>
                ))}
            </div>

            <div>
                <h2>Cart ({cart.length} items)</h2>
                {cart.length === 0 ? (
                    <p>Cart is empty</p>
                ) : (
                    <>
                        {cart.map(item => (
                            <div key={item.id} style={{
                                padding: '10px',
                                marginBottom: '10px',
                                backgroundColor: '#f5f5f5',
                                borderRadius: '8px'
                            }}>
                                <div>{item.name} - ${item.price}</div>
                                <div style={{ display: 'flex', alignItems: 'center', gap: '10px' }}>
                                    <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>
                                        -
                                    </button>
                                    <span>{item.quantity}</span>
                                    <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                                        +
                                    </button>
                                    <span>= ${item.price * item.quantity}</span>
                                    <button onClick={() => removeFromCart(item.id)}>
                                        Remove
                                    </button>
                                </div>
                            </div>
                        ))}
                        <h3>Total: ${total}</h3>
                    </>
                )}
            </div>
        </div>
    );
}
```

### Exercise 2: Multi-Step Form

```jsx
// Create a multi-step form with:
// - Step 1: Personal Info (name, email)
// - Step 2: Address (city, country)
// - Step 3: Review and Submit
// - Navigation between steps

// Your code here
```

**Solution:**
```jsx
function MultiStepForm() {
    const [step, setStep] = useState(1);
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        city: '',
        country: ''
    });

    const updateField = (field, value) => {
        setFormData(prev => ({ ...prev, [field]: value }));
    };

    const nextStep = () => setStep(prev => Math.min(prev + 1, 3));
    const prevStep = () => setStep(prev => Math.max(prev - 1, 1));

    const handleSubmit = () => {
        console.log('Submitted:', formData);
        alert('Form submitted!');
    };

    return (
        <div style={{ maxWidth: '400px', margin: '0 auto' }}>
            <div style={{ marginBottom: '20px' }}>
                <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                    {[1, 2, 3].map(s => (
                        <div
                            key={s}
                            style={{
                                width: '30px',
                                height: '30px',
                                borderRadius: '50%',
                                backgroundColor: step >= s ? '#0066cc' : '#ddd',
                                color: step >= s ? 'white' : '#666',
                                display: 'flex',
                                alignItems: 'center',
                                justifyContent: 'center'
                            }}
                        >
                            {s}
                        </div>
                    ))}
                </div>
            </div>

            {step === 1 && (
                <div>
                    <h2>Personal Information</h2>
                    <input
                        placeholder="Name"
                        value={formData.name}
                        onChange={e => updateField('name', e.target.value)}
                        style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
                    />
                    <input
                        placeholder="Email"
                        type="email"
                        value={formData.email}
                        onChange={e => updateField('email', e.target.value)}
                        style={{ width: '100%', padding: '10px' }}
                    />
                </div>
            )}

            {step === 2 && (
                <div>
                    <h2>Address</h2>
                    <input
                        placeholder="City"
                        value={formData.city}
                        onChange={e => updateField('city', e.target.value)}
                        style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
                    />
                    <input
                        placeholder="Country"
                        value={formData.country}
                        onChange={e => updateField('country', e.target.value)}
                        style={{ width: '100%', padding: '10px' }}
                    />
                </div>
            )}

            {step === 3 && (
                <div>
                    <h2>Review</h2>
                    <div style={{ backgroundColor: '#f5f5f5', padding: '20px', borderRadius: '8px' }}>
                        <p><strong>Name:</strong> {formData.name}</p>
                        <p><strong>Email:</strong> {formData.email}</p>
                        <p><strong>City:</strong> {formData.city}</p>
                        <p><strong>Country:</strong> {formData.country}</p>
                    </div>
                </div>
            )}

            <div style={{ marginTop: '20px', display: 'flex', justifyContent: 'space-between' }}>
                <button
                    onClick={prevStep}
                    disabled={step === 1}
                    style={{ padding: '10px 20px' }}
                >
                    Previous
                </button>
                {step < 3 ? (
                    <button onClick={nextStep} style={{ padding: '10px 20px' }}>
                        Next
                    </button>
                ) : (
                    <button onClick={handleSubmit} style={{ padding: '10px 20px' }}>
                        Submit
                    </button>
                )}
            </div>
        </div>
    );
}
```

---

## Key Takeaways

1. **useState returns [value, setter]** - Destructure both
2. **State updates trigger re-render** - React updates the UI
3. **Never mutate state directly** - Always create new objects/arrays
4. **Use functional updates** - When new state depends on old state
5. **Lift state up** - For shared state between components
6. **Controlled components** - React manages form inputs
7. **Group related state** - But don't over-group

---

## Self-Check Questions

1. What's the difference between props and state?
2. Why should you never mutate state directly?
3. When should you use functional updates?
4. How do you update a nested object in state?
5. What is state lifting and when would you use it?
6. What's the difference between controlled and uncontrolled components?

---

**Next Lesson:** [Day 4 - Event Handling](./day-04-event-handling.md)
