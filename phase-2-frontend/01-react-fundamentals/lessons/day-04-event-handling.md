# Day 4: Event Handling

## Events in React

React events work similarly to DOM events but with some differences:
- Named using camelCase (onClick, not onclick)
- Pass functions, not strings
- Use synthetic events (cross-browser wrapper)

### Basic Event Handling

```jsx
// DOM event handling
<button onclick="handleClick()">Click</button>

// React event handling
<button onClick={handleClick}>Click</button>
```

---

## Common Event Types

### Mouse Events

```jsx
function MouseEvents() {
    const handleClick = (e) => {
        console.log('Clicked!', e.clientX, e.clientY);
    };

    const handleDoubleClick = () => {
        console.log('Double clicked!');
    };

    const handleMouseEnter = () => {
        console.log('Mouse entered');
    };

    const handleMouseLeave = () => {
        console.log('Mouse left');
    };

    const handleMouseMove = (e) => {
        console.log('Position:', e.clientX, e.clientY);
    };

    const handleContextMenu = (e) => {
        e.preventDefault();
        console.log('Right clicked!');
    };

    return (
        <div
            onClick={handleClick}
            onDoubleClick={handleDoubleClick}
            onMouseEnter={handleMouseEnter}
            onMouseLeave={handleMouseLeave}
            onMouseMove={handleMouseMove}
            onContextMenu={handleContextMenu}
            style={{
                padding: '50px',
                backgroundColor: '#f0f0f0'
            }}
        >
            Interact with me
        </div>
    );
}
```

### Keyboard Events

```jsx
function KeyboardEvents() {
    const handleKeyDown = (e) => {
        console.log('Key down:', e.key);

        // Check for specific keys
        if (e.key === 'Enter') {
            console.log('Enter pressed!');
        }

        if (e.key === 'Escape') {
            console.log('Escape pressed!');
        }

        // Check for modifier keys
        if (e.ctrlKey && e.key === 's') {
            e.preventDefault();
            console.log('Ctrl+S pressed!');
        }

        if (e.shiftKey && e.key === 'Tab') {
            console.log('Shift+Tab pressed!');
        }
    };

    const handleKeyUp = (e) => {
        console.log('Key up:', e.key);
    };

    const handleKeyPress = (e) => {
        // Deprecated - use onKeyDown instead
        console.log('Key pressed:', e.key);
    };

    return (
        <input
            onKeyDown={handleKeyDown}
            onKeyUp={handleKeyUp}
            placeholder="Press any key"
        />
    );
}
```

### Form Events

```jsx
function FormEvents() {
    const [value, setValue] = useState('');

    const handleChange = (e) => {
        setValue(e.target.value);
    };

    const handleSubmit = (e) => {
        e.preventDefault();  // Prevent page reload
        console.log('Submitted:', value);
    };

    const handleFocus = () => {
        console.log('Input focused');
    };

    const handleBlur = () => {
        console.log('Input blurred');
    };

    const handleSelect = (e) => {
        console.log('Selected text:', window.getSelection().toString());
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                value={value}
                onChange={handleChange}
                onFocus={handleFocus}
                onBlur={handleBlur}
                onSelect={handleSelect}
            />
            <button type="submit">Submit</button>
        </form>
    );
}
```

### Focus Events

```jsx
function FocusEvents() {
    const handleFocus = (e) => {
        console.log('Focused:', e.target.name);
        e.target.style.borderColor = 'blue';
    };

    const handleBlur = (e) => {
        console.log('Blurred:', e.target.name);
        e.target.style.borderColor = '#ccc';
    };

    // Focus/blur on container (bubbles)
    const handleContainerFocus = () => {
        console.log('Something inside focused');
    };

    return (
        <div onFocus={handleContainerFocus}>
            <input name="first" onFocus={handleFocus} onBlur={handleBlur} />
            <input name="second" onFocus={handleFocus} onBlur={handleBlur} />
        </div>
    );
}
```

---

## The Event Object

React wraps native events in SyntheticEvent for cross-browser compatibility.

### Common Event Properties

```jsx
function EventProperties() {
    const handleClick = (e) => {
        // Event object properties
        console.log('Type:', e.type);           // 'click'
        console.log('Target:', e.target);       // Element that triggered
        console.log('Current Target:', e.currentTarget);  // Element with handler

        // Mouse position
        console.log('Client X/Y:', e.clientX, e.clientY);
        console.log('Page X/Y:', e.pageX, e.pageY);
        console.log('Screen X/Y:', e.screenX, e.screenY);

        // Modifier keys
        console.log('Ctrl:', e.ctrlKey);
        console.log('Shift:', e.shiftKey);
        console.log('Alt:', e.altKey);
        console.log('Meta (Cmd):', e.metaKey);

        // Access native event
        console.log('Native event:', e.nativeEvent);
    };

    return <button onClick={handleClick}>Click Me</button>;
}
```

### Preventing Default Behavior

```jsx
function PreventDefault() {
    const handleSubmit = (e) => {
        e.preventDefault();  // Prevent form submission/page reload
        console.log('Form submitted via JavaScript');
    };

    const handleLinkClick = (e) => {
        e.preventDefault();  // Prevent navigation
        console.log('Link clicked, but not navigating');
    };

    const handleContextMenu = (e) => {
        e.preventDefault();  // Prevent right-click menu
        console.log('Custom context menu');
    };

    return (
        <div>
            <form onSubmit={handleSubmit}>
                <button type="submit">Submit</button>
            </form>

            <a href="https://example.com" onClick={handleLinkClick}>
                Click me
            </a>

            <div onContextMenu={handleContextMenu}>
                Right-click me
            </div>
        </div>
    );
}
```

### Stopping Event Propagation

```jsx
function EventPropagation() {
    const handleParentClick = () => {
        console.log('Parent clicked');
    };

    const handleChildClick = (e) => {
        e.stopPropagation();  // Stop bubbling to parent
        console.log('Child clicked');
    };

    return (
        <div onClick={handleParentClick} style={{ padding: '20px', background: '#ddd' }}>
            Parent
            <button onClick={handleChildClick}>
                Child (click won't bubble)
            </button>
        </div>
    );
}
```

---

## Event Handler Patterns

### Inline Handlers

```jsx
function InlineHandlers() {
    const [count, setCount] = useState(0);

    return (
        <div>
            {/* Simple inline handler */}
            <button onClick={() => setCount(count + 1)}>
                Count: {count}
            </button>

            {/* Inline with event object */}
            <input onChange={(e) => console.log(e.target.value)} />

            {/* Inline calling a function with arguments */}
            <button onClick={() => alert('Hello!')}>
                Alert
            </button>
        </div>
    );
}
```

### Named Handlers

```jsx
function NamedHandlers() {
    const [count, setCount] = useState(0);

    const handleIncrement = () => {
        setCount(prev => prev + 1);
    };

    const handleDecrement = () => {
        setCount(prev => prev - 1);
    };

    const handleReset = () => {
        setCount(0);
    };

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={handleIncrement}>+</button>
            <button onClick={handleDecrement}>-</button>
            <button onClick={handleReset}>Reset</button>
        </div>
    );
}
```

### Passing Arguments to Handlers

```jsx
function HandlerArguments() {
    const handleClick = (id, name) => {
        console.log(`Clicked item ${id}: ${name}`);
    };

    const items = [
        { id: 1, name: 'Apple' },
        { id: 2, name: 'Banana' },
        { id: 3, name: 'Cherry' }
    ];

    return (
        <ul>
            {items.map(item => (
                <li key={item.id}>
                    {/* Method 1: Arrow function */}
                    <button onClick={() => handleClick(item.id, item.name)}>
                        {item.name}
                    </button>

                    {/* Method 2: bind (less common) */}
                    <button onClick={handleClick.bind(null, item.id, item.name)}>
                        {item.name}
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

### Handlers with Event and Arguments

```jsx
function EventAndArguments() {
    const handleItemClick = (itemId, e) => {
        console.log('Item ID:', itemId);
        console.log('Event target:', e.target);
    };

    const items = [1, 2, 3];

    return (
        <ul>
            {items.map(id => (
                <li key={id}>
                    <button onClick={(e) => handleItemClick(id, e)}>
                        Item {id}
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

---

## Handling Form Inputs

### Text Inputs

```jsx
function TextInputs() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: ''
    });

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    return (
        <form>
            <input
                name="username"
                type="text"
                value={formData.username}
                onChange={handleChange}
                placeholder="Username"
            />
            <input
                name="email"
                type="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
            />
            <input
                name="password"
                type="password"
                value={formData.password}
                onChange={handleChange}
                placeholder="Password"
            />
        </form>
    );
}
```

### Checkboxes

```jsx
function Checkboxes() {
    const [preferences, setPreferences] = useState({
        newsletter: false,
        notifications: true,
        marketing: false
    });

    const handleCheckboxChange = (e) => {
        const { name, checked } = e.target;
        setPreferences(prev => ({
            ...prev,
            [name]: checked
        }));
    };

    return (
        <div>
            <label>
                <input
                    type="checkbox"
                    name="newsletter"
                    checked={preferences.newsletter}
                    onChange={handleCheckboxChange}
                />
                Subscribe to newsletter
            </label>

            <label>
                <input
                    type="checkbox"
                    name="notifications"
                    checked={preferences.notifications}
                    onChange={handleCheckboxChange}
                />
                Enable notifications
            </label>

            <label>
                <input
                    type="checkbox"
                    name="marketing"
                    checked={preferences.marketing}
                    onChange={handleCheckboxChange}
                />
                Receive marketing emails
            </label>
        </div>
    );
}
```

### Radio Buttons

```jsx
function RadioButtons() {
    const [plan, setPlan] = useState('basic');

    const handlePlanChange = (e) => {
        setPlan(e.target.value);
    };

    return (
        <div>
            <p>Select a plan:</p>

            <label>
                <input
                    type="radio"
                    name="plan"
                    value="basic"
                    checked={plan === 'basic'}
                    onChange={handlePlanChange}
                />
                Basic ($9/month)
            </label>

            <label>
                <input
                    type="radio"
                    name="plan"
                    value="pro"
                    checked={plan === 'pro'}
                    onChange={handlePlanChange}
                />
                Pro ($19/month)
            </label>

            <label>
                <input
                    type="radio"
                    name="plan"
                    value="enterprise"
                    checked={plan === 'enterprise'}
                    onChange={handlePlanChange}
                />
                Enterprise ($49/month)
            </label>

            <p>Selected: {plan}</p>
        </div>
    );
}
```

### Select Dropdowns

```jsx
function SelectDropdowns() {
    const [country, setCountry] = useState('');
    const [skills, setSkills] = useState([]);

    // Single select
    const handleCountryChange = (e) => {
        setCountry(e.target.value);
    };

    // Multiple select
    const handleSkillsChange = (e) => {
        const options = e.target.options;
        const selected = [];
        for (let i = 0; i < options.length; i++) {
            if (options[i].selected) {
                selected.push(options[i].value);
            }
        }
        setSkills(selected);
    };

    return (
        <div>
            <select value={country} onChange={handleCountryChange}>
                <option value="">Select a country</option>
                <option value="us">United States</option>
                <option value="uk">United Kingdom</option>
                <option value="ca">Canada</option>
                <option value="au">Australia</option>
            </select>

            <select
                multiple
                value={skills}
                onChange={handleSkillsChange}
                style={{ height: '100px' }}
            >
                <option value="javascript">JavaScript</option>
                <option value="react">React</option>
                <option value="node">Node.js</option>
                <option value="python">Python</option>
            </select>

            <p>Country: {country}</p>
            <p>Skills: {skills.join(', ')}</p>
        </div>
    );
}
```

---

## Advanced Patterns

### Debounced Input

```jsx
function DebouncedSearch() {
    const [query, setQuery] = useState('');
    const [debouncedQuery, setDebouncedQuery] = useState('');

    useEffect(() => {
        const timer = setTimeout(() => {
            setDebouncedQuery(query);
        }, 500);

        return () => clearTimeout(timer);
    }, [query]);

    useEffect(() => {
        if (debouncedQuery) {
            console.log('Searching for:', debouncedQuery);
            // Perform search
        }
    }, [debouncedQuery]);

    return (
        <input
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="Search..."
        />
    );
}
```

### Drag and Drop

```jsx
function DragAndDrop() {
    const [items, setItems] = useState(['Item 1', 'Item 2', 'Item 3']);
    const [draggedIndex, setDraggedIndex] = useState(null);

    const handleDragStart = (e, index) => {
        setDraggedIndex(index);
        e.dataTransfer.effectAllowed = 'move';
    };

    const handleDragOver = (e, index) => {
        e.preventDefault();
        if (draggedIndex === index) return;

        const newItems = [...items];
        const draggedItem = newItems[draggedIndex];
        newItems.splice(draggedIndex, 1);
        newItems.splice(index, 0, draggedItem);

        setItems(newItems);
        setDraggedIndex(index);
    };

    const handleDragEnd = () => {
        setDraggedIndex(null);
    };

    return (
        <ul style={{ listStyle: 'none', padding: 0 }}>
            {items.map((item, index) => (
                <li
                    key={item}
                    draggable
                    onDragStart={(e) => handleDragStart(e, index)}
                    onDragOver={(e) => handleDragOver(e, index)}
                    onDragEnd={handleDragEnd}
                    style={{
                        padding: '10px',
                        margin: '5px 0',
                        backgroundColor: draggedIndex === index ? '#e0e0e0' : '#f5f5f5',
                        borderRadius: '4px',
                        cursor: 'grab'
                    }}
                >
                    {item}
                </li>
            ))}
        </ul>
    );
}
```

### Keyboard Shortcuts

```jsx
function KeyboardShortcuts() {
    const [isModalOpen, setIsModalOpen] = useState(false);
    const [isSaved, setIsSaved] = useState(false);

    useEffect(() => {
        const handleKeyDown = (e) => {
            // Ctrl/Cmd + S to save
            if ((e.ctrlKey || e.metaKey) && e.key === 's') {
                e.preventDefault();
                setIsSaved(true);
                setTimeout(() => setIsSaved(false), 2000);
            }

            // Escape to close modal
            if (e.key === 'Escape' && isModalOpen) {
                setIsModalOpen(false);
            }

            // Ctrl/Cmd + K to open modal
            if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
                e.preventDefault();
                setIsModalOpen(true);
            }
        };

        window.addEventListener('keydown', handleKeyDown);
        return () => window.removeEventListener('keydown', handleKeyDown);
    }, [isModalOpen]);

    return (
        <div>
            <p>Press Ctrl+S to save, Ctrl+K to open modal, Escape to close</p>
            {isSaved && <p style={{ color: 'green' }}>Saved!</p>}
            {isModalOpen && (
                <div style={{
                    position: 'fixed',
                    top: '50%',
                    left: '50%',
                    transform: 'translate(-50%, -50%)',
                    padding: '20px',
                    backgroundColor: 'white',
                    boxShadow: '0 2px 10px rgba(0,0,0,0.2)'
                }}>
                    <p>Modal content</p>
                    <button onClick={() => setIsModalOpen(false)}>Close</button>
                </div>
            )}
        </div>
    );
}
```

---

## Practice Exercises

### Exercise 1: Interactive Counter

```jsx
// Create a counter with:
// - Increment/decrement buttons
// - Double-click to reset
// - Hold button to increment/decrement continuously
// - Keyboard arrows to increment/decrement

// Your code here
```

**Solution:**
```jsx
function InteractiveCounter() {
    const [count, setCount] = useState(0);
    const intervalRef = useRef(null);

    const startIncrement = () => {
        intervalRef.current = setInterval(() => {
            setCount(prev => prev + 1);
        }, 100);
    };

    const startDecrement = () => {
        intervalRef.current = setInterval(() => {
            setCount(prev => prev - 1);
        }, 100);
    };

    const stopContinuous = () => {
        if (intervalRef.current) {
            clearInterval(intervalRef.current);
        }
    };

    const handleKeyDown = (e) => {
        if (e.key === 'ArrowUp') {
            setCount(prev => prev + 1);
        } else if (e.key === 'ArrowDown') {
            setCount(prev => prev - 1);
        }
    };

    useEffect(() => {
        return () => {
            if (intervalRef.current) {
                clearInterval(intervalRef.current);
            }
        };
    }, []);

    return (
        <div
            tabIndex={0}
            onKeyDown={handleKeyDown}
            style={{ outline: 'none', padding: '20px' }}
        >
            <p style={{ fontSize: '48px', textAlign: 'center' }}>{count}</p>
            <div style={{ display: 'flex', gap: '10px', justifyContent: 'center' }}>
                <button
                    onClick={() => setCount(prev => prev - 1)}
                    onMouseDown={startDecrement}
                    onMouseUp={stopContinuous}
                    onMouseLeave={stopContinuous}
                >
                    -
                </button>
                <button onDoubleClick={() => setCount(0)}>
                    Reset (double-click)
                </button>
                <button
                    onClick={() => setCount(prev => prev + 1)}
                    onMouseDown={startIncrement}
                    onMouseUp={stopContinuous}
                    onMouseLeave={stopContinuous}
                >
                    +
                </button>
            </div>
            <p style={{ textAlign: 'center', color: '#666' }}>
                Use arrow keys or hold buttons
            </p>
        </div>
    );
}
```

### Exercise 2: Form with Real-time Validation

```jsx
// Create a signup form with:
// - Email validation (real-time)
// - Password strength indicator
// - Form submission handling
// - Error display

// Your code here
```

**Solution:**
```jsx
function SignupFormValidation() {
    const [formData, setFormData] = useState({
        email: '',
        password: ''
    });
    const [errors, setErrors] = useState({});
    const [passwordStrength, setPasswordStrength] = useState(0);

    const validateEmail = (email) => {
        if (!email) return 'Email is required';
        if (!/\S+@\S+\.\S+/.test(email)) return 'Invalid email format';
        return '';
    };

    const calculatePasswordStrength = (password) => {
        let strength = 0;
        if (password.length >= 8) strength += 25;
        if (/[A-Z]/.test(password)) strength += 25;
        if (/[a-z]/.test(password)) strength += 25;
        if (/[0-9]/.test(password)) strength += 15;
        if (/[^A-Za-z0-9]/.test(password)) strength += 10;
        return Math.min(strength, 100);
    };

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({ ...prev, [name]: value }));

        if (name === 'email') {
            setErrors(prev => ({ ...prev, email: validateEmail(value) }));
        }

        if (name === 'password') {
            setPasswordStrength(calculatePasswordStrength(value));
            if (value.length < 8) {
                setErrors(prev => ({
                    ...prev,
                    password: 'Password must be at least 8 characters'
                }));
            } else {
                setErrors(prev => ({ ...prev, password: '' }));
            }
        }
    };

    const handleSubmit = (e) => {
        e.preventDefault();

        const emailError = validateEmail(formData.email);
        const passwordError = formData.password.length < 8
            ? 'Password must be at least 8 characters'
            : '';

        if (emailError || passwordError) {
            setErrors({ email: emailError, password: passwordError });
            return;
        }

        console.log('Submitted:', formData);
        alert('Form submitted successfully!');
    };

    const getStrengthColor = () => {
        if (passwordStrength < 30) return '#ff4444';
        if (passwordStrength < 60) return '#ffaa00';
        return '#00aa00';
    };

    return (
        <form onSubmit={handleSubmit} style={{ maxWidth: '300px' }}>
            <div style={{ marginBottom: '15px' }}>
                <input
                    name="email"
                    type="email"
                    value={formData.email}
                    onChange={handleChange}
                    onBlur={handleChange}
                    placeholder="Email"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.email ? 'red' : '#ccc'
                    }}
                />
                {errors.email && (
                    <span style={{ color: 'red', fontSize: '12px' }}>
                        {errors.email}
                    </span>
                )}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <input
                    name="password"
                    type="password"
                    value={formData.password}
                    onChange={handleChange}
                    placeholder="Password"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.password ? 'red' : '#ccc'
                    }}
                />
                {formData.password && (
                    <div style={{ marginTop: '5px' }}>
                        <div style={{
                            height: '5px',
                            backgroundColor: '#eee',
                            borderRadius: '3px'
                        }}>
                            <div style={{
                                height: '100%',
                                width: `${passwordStrength}%`,
                                backgroundColor: getStrengthColor(),
                                borderRadius: '3px',
                                transition: 'width 0.3s'
                            }} />
                        </div>
                        <span style={{ fontSize: '12px' }}>
                            Strength: {passwordStrength}%
                        </span>
                    </div>
                )}
                {errors.password && (
                    <span style={{ color: 'red', fontSize: '12px' }}>
                        {errors.password}
                    </span>
                )}
            </div>

            <button type="submit" style={{ width: '100%', padding: '10px' }}>
                Sign Up
            </button>
        </form>
    );
}
```

---

## Key Takeaways

1. **camelCase event names** - onClick, onChange, onSubmit
2. **Pass functions** - Not function calls (onClick={fn} not onClick={fn()})
3. **SyntheticEvent** - Cross-browser event wrapper
4. **preventDefault()** - Stop default browser behavior
5. **stopPropagation()** - Stop event bubbling
6. **Controlled inputs** - React manages form values via state
7. **Event delegation** - Attach one handler to parent for many children

---

## Self-Check Questions

1. What's the difference between onClick={fn} and onClick={fn()}?
2. When should you use preventDefault()?
3. How do you pass arguments to event handlers?
4. What's the difference between target and currentTarget?
5. How do you handle keyboard shortcuts in React?
6. What are controlled vs uncontrolled inputs?

---

**Next Lesson:** [Day 5 - Conditional Rendering](./day-05-conditional-rendering.md)
