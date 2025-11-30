# Day 10: TypeScript with React

## Setting Up React with TypeScript

### Create React App

```bash
# Create new React TypeScript project
npx create-react-app my-app --template typescript

# Or with Vite (recommended for modern projects)
npm create vite@latest my-app -- --template react-ts

# Project structure
my-app/
├── src/
│   ├── App.tsx          # .tsx for JSX files
│   ├── App.css
│   ├── index.tsx
│   ├── components/
│   │   └── Button.tsx
│   ├── hooks/
│   │   └── useAuth.ts   # .ts for non-JSX files
│   ├── types/
│   │   └── index.ts
│   └── utils/
│       └── helpers.ts
├── tsconfig.json
└── package.json
```

### tsconfig.json for React

```json
{
    "compilerOptions": {
        "target": "ES2020",
        "lib": ["DOM", "DOM.Iterable", "ESNext"],
        "module": "ESNext",
        "moduleResolution": "bundler",
        "jsx": "react-jsx",
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,
        "isolatedModules": true,
        "baseUrl": ".",
        "paths": {
            "@/*": ["src/*"],
            "@components/*": ["src/components/*"],
            "@hooks/*": ["src/hooks/*"],
            "@types/*": ["src/types/*"]
        }
    },
    "include": ["src"],
    "references": [{ "path": "./tsconfig.node.json" }]
}
```

---

## Component Types

### Functional Components

```tsx
// Basic component with no props
const HelloWorld = () => {
    return <h1>Hello, World!</h1>;
};

// With explicit return type
const HelloWorld2 = (): JSX.Element => {
    return <h1>Hello, World!</h1>;
};

// Using React.FC (optional, less common now)
const HelloWorld3: React.FC = () => {
    return <h1>Hello, World!</h1>;
};


// Component with props
interface GreetingProps {
    name: string;
    age?: number;  // Optional prop
}

const Greeting = ({ name, age }: GreetingProps) => {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            {age && <p>You are {age} years old.</p>}
        </div>
    );
};

// Usage
<Greeting name="John" />
<Greeting name="Jane" age={25} />


// Component with children
interface CardProps {
    title: string;
    children: React.ReactNode;  // Most flexible for children
}

const Card = ({ title, children }: CardProps) => {
    return (
        <div className="card">
            <h2>{title}</h2>
            <div className="card-content">{children}</div>
        </div>
    );
};

// Usage
<Card title="Welcome">
    <p>This is the card content</p>
    <button>Click me</button>
</Card>


// More specific children types
interface ListProps {
    children: React.ReactElement | React.ReactElement[];  // Only React elements
}

interface TextOnlyProps {
    children: string;  // Only text
}

interface SingleChildProps {
    children: React.ReactElement;  // Exactly one element
}
```

### Props with Default Values

```tsx
interface ButtonProps {
    label: string;
    variant?: "primary" | "secondary" | "danger";
    size?: "small" | "medium" | "large";
    disabled?: boolean;
    onClick?: () => void;
}

const Button = ({
    label,
    variant = "primary",
    size = "medium",
    disabled = false,
    onClick
}: ButtonProps) => {
    return (
        <button
            className={`btn btn-${variant} btn-${size}`}
            disabled={disabled}
            onClick={onClick}
        >
            {label}
        </button>
    );
};

// Alternative: defaultProps (older pattern)
const Button2 = (props: ButtonProps) => {
    const { label, variant, size, disabled, onClick } = props;
    // ...
};

Button2.defaultProps = {
    variant: "primary",
    size: "medium",
    disabled: false
};
```

### Component Props Patterns

```tsx
// Extending HTML element props
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
    variant?: "primary" | "secondary";
    isLoading?: boolean;
}

const Button = ({ variant = "primary", isLoading, children, ...rest }: ButtonProps) => {
    return (
        <button
            className={`btn btn-${variant}`}
            disabled={isLoading || rest.disabled}
            {...rest}
        >
            {isLoading ? "Loading..." : children}
        </button>
    );
};

// Usage - all HTML button props are available
<Button type="submit" onClick={() => {}} variant="primary">
    Submit
</Button>


// Extending input props
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
    label: string;
    error?: string;
}

const Input = ({ label, error, ...rest }: InputProps) => {
    return (
        <div className="input-group">
            <label>{label}</label>
            <input {...rest} className={error ? "error" : ""} />
            {error && <span className="error-message">{error}</span>}
        </div>
    );
};


// Polymorphic component (as prop)
type PolymorphicProps<E extends React.ElementType> = {
    as?: E;
    children: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<E>, "as" | "children">;

const Box = <E extends React.ElementType = "div">({
    as,
    children,
    ...rest
}: PolymorphicProps<E>) => {
    const Component = as || "div";
    return <Component {...rest}>{children}</Component>;
};

// Usage
<Box>Default div</Box>
<Box as="section">As section</Box>
<Box as="a" href="/home">As anchor</Box>
<Box as="button" onClick={() => {}}>As button</Box>
```

---

## Typing Hooks

### useState

```tsx
// Inferred type from initial value
const [count, setCount] = useState(0);  // number
const [name, setName] = useState("");   // string
const [active, setActive] = useState(false);  // boolean

// Explicit type for complex state
interface User {
    id: number;
    name: string;
    email: string;
}

const [user, setUser] = useState<User | null>(null);

// Array state
const [items, setItems] = useState<string[]>([]);
const [users, setUsers] = useState<User[]>([]);

// Union types
type Status = "idle" | "loading" | "success" | "error";
const [status, setStatus] = useState<Status>("idle");

// Object state
interface FormState {
    email: string;
    password: string;
    rememberMe: boolean;
}

const [form, setForm] = useState<FormState>({
    email: "",
    password: "",
    rememberMe: false
});

// Update patterns
setCount(10);                    // Direct value
setCount(prev => prev + 1);      // Callback

setUser({ id: 1, name: "John", email: "john@example.com" });
setUser(null);

setItems(prev => [...prev, "new item"]);

setForm(prev => ({ ...prev, email: "test@example.com" }));
```

### useReducer

```tsx
// State and action types
interface CounterState {
    count: number;
    step: number;
}

type CounterAction =
    | { type: "INCREMENT" }
    | { type: "DECREMENT" }
    | { type: "RESET" }
    | { type: "SET_STEP"; payload: number }
    | { type: "SET_COUNT"; payload: number };

// Reducer function
const counterReducer = (
    state: CounterState,
    action: CounterAction
): CounterState => {
    switch (action.type) {
        case "INCREMENT":
            return { ...state, count: state.count + state.step };
        case "DECREMENT":
            return { ...state, count: state.count - state.step };
        case "RESET":
            return { ...state, count: 0 };
        case "SET_STEP":
            return { ...state, step: action.payload };
        case "SET_COUNT":
            return { ...state, count: action.payload };
        default:
            return state;
    }
};

// Component using reducer
const Counter = () => {
    const [state, dispatch] = useReducer(counterReducer, {
        count: 0,
        step: 1
    });

    return (
        <div>
            <p>Count: {state.count}</p>
            <p>Step: {state.step}</p>
            <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
            <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
            <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
            <button onClick={() => dispatch({ type: "SET_STEP", payload: 5 })}>
                Step = 5
            </button>
        </div>
    );
};


// More complex example with async actions
interface FetchState<T> {
    data: T | null;
    loading: boolean;
    error: string | null;
}

type FetchAction<T> =
    | { type: "FETCH_START" }
    | { type: "FETCH_SUCCESS"; payload: T }
    | { type: "FETCH_ERROR"; payload: string };

function fetchReducer<T>(
    state: FetchState<T>,
    action: FetchAction<T>
): FetchState<T> {
    switch (action.type) {
        case "FETCH_START":
            return { ...state, loading: true, error: null };
        case "FETCH_SUCCESS":
            return { data: action.payload, loading: false, error: null };
        case "FETCH_ERROR":
            return { ...state, loading: false, error: action.payload };
        default:
            return state;
    }
}

// Usage with typed data
interface User {
    id: number;
    name: string;
}

const [state, dispatch] = useReducer(
    fetchReducer<User[]>,
    { data: null, loading: false, error: null }
);
```

### useRef

```tsx
// DOM element ref
const inputRef = useRef<HTMLInputElement>(null);
const buttonRef = useRef<HTMLButtonElement>(null);
const divRef = useRef<HTMLDivElement>(null);

const focusInput = () => {
    inputRef.current?.focus();
};

// In JSX
<input ref={inputRef} type="text" />
<button ref={buttonRef}>Click</button>


// Mutable ref (storing values)
const countRef = useRef<number>(0);
const timerRef = useRef<NodeJS.Timeout | null>(null);

const startTimer = () => {
    timerRef.current = setInterval(() => {
        countRef.current += 1;
        console.log(countRef.current);
    }, 1000);
};

const stopTimer = () => {
    if (timerRef.current) {
        clearInterval(timerRef.current);
    }
};


// Previous value pattern
function usePrevious<T>(value: T): T | undefined {
    const ref = useRef<T>();

    useEffect(() => {
        ref.current = value;
    }, [value]);

    return ref.current;
}

// Usage
const [count, setCount] = useState(0);
const prevCount = usePrevious(count);
```

### useCallback and useMemo

```tsx
// useCallback - memoize functions
interface ItemProps {
    id: number;
    onDelete: (id: number) => void;
}

const ParentComponent = () => {
    const [items, setItems] = useState<number[]>([1, 2, 3]);

    // Memoized callback
    const handleDelete = useCallback((id: number) => {
        setItems(prev => prev.filter(item => item !== id));
    }, []);  // No dependencies

    // Callback with dependencies
    const handleAdd = useCallback((newItem: number) => {
        setItems(prev => [...prev, newItem]);
    }, []);

    return (
        <div>
            {items.map(id => (
                <Item key={id} id={id} onDelete={handleDelete} />
            ))}
        </div>
    );
};


// useMemo - memoize values
interface User {
    id: number;
    name: string;
    role: "admin" | "user";
}

const UserList = ({ users, filterRole }: {
    users: User[];
    filterRole: "admin" | "user" | "all";
}) => {
    // Memoized computed value
    const filteredUsers = useMemo(() => {
        if (filterRole === "all") return users;
        return users.filter(user => user.role === filterRole);
    }, [users, filterRole]);

    const userCount = useMemo(() => filteredUsers.length, [filteredUsers]);

    // Memoized complex computation
    const sortedUsers = useMemo(() => {
        return [...filteredUsers].sort((a, b) => a.name.localeCompare(b.name));
    }, [filteredUsers]);

    return (
        <div>
            <p>Showing {userCount} users</p>
            {sortedUsers.map(user => (
                <div key={user.id}>{user.name}</div>
            ))}
        </div>
    );
};
```

### useEffect

```tsx
// Basic effect with cleanup
useEffect(() => {
    const subscription = eventEmitter.subscribe((event) => {
        console.log(event);
    });

    // Cleanup function
    return () => {
        subscription.unsubscribe();
    };
}, []);

// Effect with typed async function
useEffect(() => {
    const fetchData = async () => {
        try {
            const response = await fetch("/api/users");
            const data: User[] = await response.json();
            setUsers(data);
        } catch (error) {
            setError(error instanceof Error ? error.message : "Unknown error");
        }
    };

    fetchData();
}, []);

// Effect with abort controller
useEffect(() => {
    const controller = new AbortController();

    const fetchData = async () => {
        try {
            const response = await fetch("/api/users", {
                signal: controller.signal
            });
            const data: User[] = await response.json();
            setUsers(data);
        } catch (error) {
            if (error instanceof Error && error.name !== "AbortError") {
                setError(error.message);
            }
        }
    };

    fetchData();

    return () => {
        controller.abort();
    };
}, []);
```

---

## Context API with TypeScript

### Creating Typed Context

```tsx
// types/auth.ts
interface User {
    id: string;
    email: string;
    name: string;
}

interface AuthContextType {
    user: User | null;
    isAuthenticated: boolean;
    login: (email: string, password: string) => Promise<void>;
    logout: () => void;
    loading: boolean;
}

// context/AuthContext.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from "react";

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Custom hook for using context
export const useAuth = (): AuthContextType => {
    const context = useContext(AuthContext);
    if (context === undefined) {
        throw new Error("useAuth must be used within an AuthProvider");
    }
    return context;
};

// Provider component
interface AuthProviderProps {
    children: ReactNode;
}

export const AuthProvider = ({ children }: AuthProviderProps) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(false);

    const login = useCallback(async (email: string, password: string) => {
        setLoading(true);
        try {
            const response = await fetch("/api/auth/login", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ email, password })
            });

            if (!response.ok) {
                throw new Error("Login failed");
            }

            const userData: User = await response.json();
            setUser(userData);
        } finally {
            setLoading(false);
        }
    }, []);

    const logout = useCallback(() => {
        setUser(null);
    }, []);

    const value: AuthContextType = {
        user,
        isAuthenticated: user !== null,
        login,
        logout,
        loading
    };

    return (
        <AuthContext.Provider value={value}>
            {children}
        </AuthContext.Provider>
    );
};

// Usage in components
const LoginButton = () => {
    const { isAuthenticated, login, logout, loading } = useAuth();

    if (loading) return <button disabled>Loading...</button>;

    if (isAuthenticated) {
        return <button onClick={logout}>Logout</button>;
    }

    return (
        <button onClick={() => login("user@example.com", "password")}>
            Login
        </button>
    );
};

const UserProfile = () => {
    const { user, isAuthenticated } = useAuth();

    if (!isAuthenticated || !user) {
        return <p>Please log in</p>;
    }

    return (
        <div>
            <h2>{user.name}</h2>
            <p>{user.email}</p>
        </div>
    );
};
```

### Generic Context Pattern

```tsx
// Generic context factory
function createContext<T>() {
    const Context = React.createContext<T | undefined>(undefined);

    function useContextValue(): T {
        const context = React.useContext(Context);
        if (context === undefined) {
            throw new Error("useContext must be used within a Provider");
        }
        return context;
    }

    return [Context.Provider, useContextValue] as const;
}

// Usage
interface ThemeContextType {
    theme: "light" | "dark";
    toggleTheme: () => void;
}

const [ThemeProvider, useTheme] = createContext<ThemeContextType>();

// Provider implementation
const ThemeProviderWrapper = ({ children }: { children: ReactNode }) => {
    const [theme, setTheme] = useState<"light" | "dark">("light");

    const toggleTheme = useCallback(() => {
        setTheme(prev => prev === "light" ? "dark" : "light");
    }, []);

    return (
        <ThemeProvider value={{ theme, toggleTheme }}>
            {children}
        </ThemeProvider>
    );
};

// Usage in component
const ThemeToggle = () => {
    const { theme, toggleTheme } = useTheme();

    return (
        <button onClick={toggleTheme}>
            Current: {theme}
        </button>
    );
};
```

---

## Custom Hooks

### Typed Custom Hooks

```tsx
// Simple data fetching hook
interface UseFetchResult<T> {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<Error | null>(null);

    const fetchData = useCallback(async () => {
        setLoading(true);
        setError(null);

        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            const result: T = await response.json();
            setData(result);
        } catch (e) {
            setError(e instanceof Error ? e : new Error("Unknown error"));
        } finally {
            setLoading(false);
        }
    }, [url]);

    useEffect(() => {
        fetchData();
    }, [fetchData]);

    return { data, loading, error, refetch: fetchData };
}

// Usage
interface Post {
    id: number;
    title: string;
    body: string;
}

const PostList = () => {
    const { data: posts, loading, error, refetch } = useFetch<Post[]>(
        "https://jsonplaceholder.typicode.com/posts"
    );

    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error.message}</p>;
    if (!posts) return null;

    return (
        <div>
            <button onClick={refetch}>Refresh</button>
            {posts.map(post => (
                <article key={post.id}>
                    <h2>{post.title}</h2>
                    <p>{post.body}</p>
                </article>
            ))}
        </div>
    );
};


// Local storage hook
function useLocalStorage<T>(
    key: string,
    initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
    const [storedValue, setStoredValue] = useState<T>(() => {
        try {
            const item = localStorage.getItem(key);
            return item ? (JSON.parse(item) as T) : initialValue;
        } catch {
            return initialValue;
        }
    });

    const setValue = useCallback(
        (value: T | ((prev: T) => T)) => {
            try {
                const valueToStore = value instanceof Function
                    ? value(storedValue)
                    : value;

                setStoredValue(valueToStore);
                localStorage.setItem(key, JSON.stringify(valueToStore));
            } catch (error) {
                console.error("Error saving to localStorage:", error);
            }
        },
        [key, storedValue]
    );

    return [storedValue, setValue];
}

// Usage
const Settings = () => {
    const [theme, setTheme] = useLocalStorage<"light" | "dark">("theme", "light");
    const [user, setUser] = useLocalStorage<User | null>("user", null);

    return (
        <div>
            <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
                Toggle theme: {theme}
            </button>
        </div>
    );
};


// Debounce hook
function useDebounce<T>(value: T, delay: number): T {
    const [debouncedValue, setDebouncedValue] = useState<T>(value);

    useEffect(() => {
        const handler = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);

        return () => {
            clearTimeout(handler);
        };
    }, [value, delay]);

    return debouncedValue;
}

// Usage
const SearchComponent = () => {
    const [query, setQuery] = useState("");
    const debouncedQuery = useDebounce(query, 500);

    useEffect(() => {
        if (debouncedQuery) {
            // Perform search
            console.log("Searching for:", debouncedQuery);
        }
    }, [debouncedQuery]);

    return (
        <input
            value={query}
            onChange={e => setQuery(e.target.value)}
            placeholder="Search..."
        />
    );
};
```

### Hook with Generic Event Handlers

```tsx
// Form handling hook
interface UseFormOptions<T> {
    initialValues: T;
    onSubmit: (values: T) => void | Promise<void>;
    validate?: (values: T) => Partial<Record<keyof T, string>>;
}

interface UseFormResult<T> {
    values: T;
    errors: Partial<Record<keyof T, string>>;
    touched: Partial<Record<keyof T, boolean>>;
    isSubmitting: boolean;
    handleChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
    handleBlur: (e: React.FocusEvent<HTMLInputElement>) => void;
    handleSubmit: (e: React.FormEvent) => void;
    setFieldValue: <K extends keyof T>(field: K, value: T[K]) => void;
    resetForm: () => void;
}

function useForm<T extends Record<string, unknown>>({
    initialValues,
    onSubmit,
    validate
}: UseFormOptions<T>): UseFormResult<T> {
    const [values, setValues] = useState<T>(initialValues);
    const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
    const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
    const [isSubmitting, setIsSubmitting] = useState(false);

    const handleChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
        const { name, value, type, checked } = e.target;
        setValues(prev => ({
            ...prev,
            [name]: type === "checkbox" ? checked : value
        }));
    }, []);

    const handleBlur = useCallback((e: React.FocusEvent<HTMLInputElement>) => {
        const { name } = e.target;
        setTouched(prev => ({ ...prev, [name]: true }));

        if (validate) {
            const validationErrors = validate(values);
            setErrors(validationErrors);
        }
    }, [values, validate]);

    const handleSubmit = useCallback(async (e: React.FormEvent) => {
        e.preventDefault();

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
    }, [values, onSubmit, validate]);

    const setFieldValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
        setValues(prev => ({ ...prev, [field]: value }));
    }, []);

    const resetForm = useCallback(() => {
        setValues(initialValues);
        setErrors({});
        setTouched({});
    }, [initialValues]);

    return {
        values,
        errors,
        touched,
        isSubmitting,
        handleChange,
        handleBlur,
        handleSubmit,
        setFieldValue,
        resetForm
    };
}

// Usage
interface LoginFormValues {
    email: string;
    password: string;
    rememberMe: boolean;
}

const LoginForm = () => {
    const form = useForm<LoginFormValues>({
        initialValues: {
            email: "",
            password: "",
            rememberMe: false
        },
        validate: (values) => {
            const errors: Partial<Record<keyof LoginFormValues, string>> = {};

            if (!values.email) {
                errors.email = "Email is required";
            } else if (!values.email.includes("@")) {
                errors.email = "Invalid email format";
            }

            if (!values.password) {
                errors.password = "Password is required";
            } else if (values.password.length < 8) {
                errors.password = "Password must be at least 8 characters";
            }

            return errors;
        },
        onSubmit: async (values) => {
            console.log("Submitting:", values);
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
    });

    return (
        <form onSubmit={form.handleSubmit}>
            <div>
                <input
                    name="email"
                    type="email"
                    value={form.values.email}
                    onChange={form.handleChange}
                    onBlur={form.handleBlur}
                    placeholder="Email"
                />
                {form.touched.email && form.errors.email && (
                    <span className="error">{form.errors.email}</span>
                )}
            </div>

            <div>
                <input
                    name="password"
                    type="password"
                    value={form.values.password}
                    onChange={form.handleChange}
                    onBlur={form.handleBlur}
                    placeholder="Password"
                />
                {form.touched.password && form.errors.password && (
                    <span className="error">{form.errors.password}</span>
                )}
            </div>

            <label>
                <input
                    name="rememberMe"
                    type="checkbox"
                    checked={form.values.rememberMe}
                    onChange={form.handleChange}
                />
                Remember me
            </label>

            <button type="submit" disabled={form.isSubmitting}>
                {form.isSubmitting ? "Logging in..." : "Login"}
            </button>
        </form>
    );
};
```

---

## Event Handling

### Typed Event Handlers

```tsx
// Form events
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
};

const handleTextAreaChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    console.log(e.target.value);
};

const handleSelectChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    console.log(e.target.value);
};

const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Handle form submission
};

// Mouse events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log(e.clientX, e.clientY);
};

const handleMouseEnter = (e: React.MouseEvent<HTMLDivElement>) => {
    console.log("Mouse entered");
};

// Keyboard events
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === "Enter") {
        console.log("Enter pressed");
    }
};

const handleKeyPress = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (e.key === "Enter" && e.ctrlKey) {
        console.log("Ctrl+Enter pressed");
    }
};

// Focus events
const handleFocus = (e: React.FocusEvent<HTMLInputElement>) => {
    console.log("Input focused");
};

const handleBlur = (e: React.FocusEvent<HTMLInputElement>) => {
    console.log("Input blurred");
};

// Drag events
const handleDragStart = (e: React.DragEvent<HTMLDivElement>) => {
    e.dataTransfer.setData("text/plain", "dragged item");
};

const handleDrop = (e: React.DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    const data = e.dataTransfer.getData("text/plain");
    console.log("Dropped:", data);
};
```

### Generic Event Handlers

```tsx
// Creating reusable event handlers
interface Item {
    id: number;
    name: string;
}

interface ItemListProps {
    items: Item[];
    onItemClick: (item: Item) => void;
    onItemDelete: (id: number) => void;
}

const ItemList = ({ items, onItemClick, onItemDelete }: ItemListProps) => {
    // Create handler factory
    const createClickHandler = (item: Item) => () => {
        onItemClick(item);
    };

    const createDeleteHandler = (id: number) => (e: React.MouseEvent) => {
        e.stopPropagation();  // Prevent item click
        onItemDelete(id);
    };

    return (
        <ul>
            {items.map(item => (
                <li key={item.id} onClick={createClickHandler(item)}>
                    {item.name}
                    <button onClick={createDeleteHandler(item.id)}>
                        Delete
                    </button>
                </li>
            ))}
        </ul>
    );
};

// Type-safe event delegation
const handleListClick = (e: React.MouseEvent<HTMLUListElement>) => {
    const target = e.target as HTMLElement;
    const itemId = target.dataset.itemId;

    if (itemId) {
        console.log("Clicked item:", itemId);
    }
};
```

---

## Practice Exercises

### Exercise 1: Typed Todo App

```tsx
// Create a fully typed Todo application with:
// - Add, toggle, delete todos
// - Filter by status (all, active, completed)
// - Persist to localStorage

// Your code here
```

**Solution:**
```tsx
// types.ts
interface Todo {
    id: string;
    text: string;
    completed: boolean;
    createdAt: Date;
}

type FilterStatus = "all" | "active" | "completed";

// hooks/useTodos.ts
interface UseTodosResult {
    todos: Todo[];
    filteredTodos: Todo[];
    filter: FilterStatus;
    addTodo: (text: string) => void;
    toggleTodo: (id: string) => void;
    deleteTodo: (id: string) => void;
    setFilter: (filter: FilterStatus) => void;
    clearCompleted: () => void;
}

function useTodos(): UseTodosResult {
    const [todos, setTodos] = useLocalStorage<Todo[]>("todos", []);
    const [filter, setFilter] = useState<FilterStatus>("all");

    const addTodo = useCallback((text: string) => {
        const newTodo: Todo = {
            id: crypto.randomUUID(),
            text: text.trim(),
            completed: false,
            createdAt: new Date()
        };
        setTodos(prev => [...prev, newTodo]);
    }, [setTodos]);

    const toggleTodo = useCallback((id: string) => {
        setTodos(prev =>
            prev.map(todo =>
                todo.id === id
                    ? { ...todo, completed: !todo.completed }
                    : todo
            )
        );
    }, [setTodos]);

    const deleteTodo = useCallback((id: string) => {
        setTodos(prev => prev.filter(todo => todo.id !== id));
    }, [setTodos]);

    const clearCompleted = useCallback(() => {
        setTodos(prev => prev.filter(todo => !todo.completed));
    }, [setTodos]);

    const filteredTodos = useMemo(() => {
        switch (filter) {
            case "active":
                return todos.filter(todo => !todo.completed);
            case "completed":
                return todos.filter(todo => todo.completed);
            default:
                return todos;
        }
    }, [todos, filter]);

    return {
        todos,
        filteredTodos,
        filter,
        addTodo,
        toggleTodo,
        deleteTodo,
        setFilter,
        clearCompleted
    };
}

// components/TodoItem.tsx
interface TodoItemProps {
    todo: Todo;
    onToggle: (id: string) => void;
    onDelete: (id: string) => void;
}

const TodoItem = ({ todo, onToggle, onDelete }: TodoItemProps) => {
    return (
        <li className={todo.completed ? "completed" : ""}>
            <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => onToggle(todo.id)}
            />
            <span>{todo.text}</span>
            <button onClick={() => onDelete(todo.id)}>×</button>
        </li>
    );
};

// components/TodoInput.tsx
interface TodoInputProps {
    onAdd: (text: string) => void;
}

const TodoInput = ({ onAdd }: TodoInputProps) => {
    const [text, setText] = useState("");

    const handleSubmit = (e: React.FormEvent) => {
        e.preventDefault();
        if (text.trim()) {
            onAdd(text);
            setText("");
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                value={text}
                onChange={e => setText(e.target.value)}
                placeholder="What needs to be done?"
            />
            <button type="submit">Add</button>
        </form>
    );
};

// components/TodoFilter.tsx
interface TodoFilterProps {
    current: FilterStatus;
    onChange: (filter: FilterStatus) => void;
    activeCount: number;
}

const TodoFilter = ({ current, onChange, activeCount }: TodoFilterProps) => {
    const filters: FilterStatus[] = ["all", "active", "completed"];

    return (
        <div className="filters">
            <span>{activeCount} items left</span>
            {filters.map(filter => (
                <button
                    key={filter}
                    className={current === filter ? "active" : ""}
                    onClick={() => onChange(filter)}
                >
                    {filter.charAt(0).toUpperCase() + filter.slice(1)}
                </button>
            ))}
        </div>
    );
};

// App.tsx
const TodoApp = () => {
    const {
        todos,
        filteredTodos,
        filter,
        addTodo,
        toggleTodo,
        deleteTodo,
        setFilter,
        clearCompleted
    } = useTodos();

    const activeCount = todos.filter(t => !t.completed).length;
    const hasCompleted = todos.some(t => t.completed);

    return (
        <div className="todo-app">
            <h1>Todos</h1>
            <TodoInput onAdd={addTodo} />

            <ul className="todo-list">
                {filteredTodos.map(todo => (
                    <TodoItem
                        key={todo.id}
                        todo={todo}
                        onToggle={toggleTodo}
                        onDelete={deleteTodo}
                    />
                ))}
            </ul>

            {todos.length > 0 && (
                <>
                    <TodoFilter
                        current={filter}
                        onChange={setFilter}
                        activeCount={activeCount}
                    />
                    {hasCompleted && (
                        <button onClick={clearCompleted}>
                            Clear completed
                        </button>
                    )}
                </>
            )}
        </div>
    );
};
```

### Exercise 2: Typed Data Table Component

```tsx
// Create a generic, typed DataTable component that:
// - Accepts any data type
// - Has typed columns with custom renderers
// - Supports sorting

// Your code here
```

**Solution:**
```tsx
// types
interface Column<T> {
    key: keyof T;
    header: string;
    render?: (value: T[keyof T], row: T) => React.ReactNode;
    sortable?: boolean;
}

interface DataTableProps<T extends { id: string | number }> {
    data: T[];
    columns: Column<T>[];
    onRowClick?: (row: T) => void;
}

type SortDirection = "asc" | "desc" | null;

interface SortConfig<T> {
    key: keyof T | null;
    direction: SortDirection;
}

// DataTable component
function DataTable<T extends { id: string | number }>({
    data,
    columns,
    onRowClick
}: DataTableProps<T>) {
    const [sortConfig, setSortConfig] = useState<SortConfig<T>>({
        key: null,
        direction: null
    });

    const handleSort = (key: keyof T) => {
        let direction: SortDirection = "asc";

        if (sortConfig.key === key) {
            if (sortConfig.direction === "asc") {
                direction = "desc";
            } else if (sortConfig.direction === "desc") {
                direction = null;
            }
        }

        setSortConfig({ key: direction ? key : null, direction });
    };

    const sortedData = useMemo(() => {
        if (!sortConfig.key || !sortConfig.direction) {
            return data;
        }

        return [...data].sort((a, b) => {
            const aValue = a[sortConfig.key!];
            const bValue = b[sortConfig.key!];

            if (aValue < bValue) {
                return sortConfig.direction === "asc" ? -1 : 1;
            }
            if (aValue > bValue) {
                return sortConfig.direction === "asc" ? 1 : -1;
            }
            return 0;
        });
    }, [data, sortConfig]);

    const getSortIndicator = (key: keyof T): string => {
        if (sortConfig.key !== key) return "";
        if (sortConfig.direction === "asc") return " ↑";
        if (sortConfig.direction === "desc") return " ↓";
        return "";
    };

    return (
        <table className="data-table">
            <thead>
                <tr>
                    {columns.map(column => (
                        <th
                            key={String(column.key)}
                            onClick={() => column.sortable && handleSort(column.key)}
                            style={{ cursor: column.sortable ? "pointer" : "default" }}
                        >
                            {column.header}
                            {column.sortable && getSortIndicator(column.key)}
                        </th>
                    ))}
                </tr>
            </thead>
            <tbody>
                {sortedData.map(row => (
                    <tr
                        key={row.id}
                        onClick={() => onRowClick?.(row)}
                        style={{ cursor: onRowClick ? "pointer" : "default" }}
                    >
                        {columns.map(column => (
                            <td key={String(column.key)}>
                                {column.render
                                    ? column.render(row[column.key], row)
                                    : String(row[column.key])}
                            </td>
                        ))}
                    </tr>
                ))}
            </tbody>
        </table>
    );
}

// Usage
interface User {
    id: number;
    name: string;
    email: string;
    role: "admin" | "user";
    createdAt: string;
}

const UserTable = () => {
    const users: User[] = [
        { id: 1, name: "John", email: "john@example.com", role: "admin", createdAt: "2024-01-15" },
        { id: 2, name: "Jane", email: "jane@example.com", role: "user", createdAt: "2024-02-20" },
    ];

    const columns: Column<User>[] = [
        { key: "id", header: "ID", sortable: true },
        { key: "name", header: "Name", sortable: true },
        { key: "email", header: "Email", sortable: true },
        {
            key: "role",
            header: "Role",
            sortable: true,
            render: (value) => (
                <span className={`badge badge-${value}`}>
                    {String(value).toUpperCase()}
                </span>
            )
        },
        {
            key: "createdAt",
            header: "Created",
            sortable: true,
            render: (value) => new Date(String(value)).toLocaleDateString()
        }
    ];

    return (
        <DataTable
            data={users}
            columns={columns}
            onRowClick={(user) => console.log("Selected:", user)}
        />
    );
};
```

---

## Key Takeaways

1. **Component props** - Use interfaces for props, extend HTML attributes
2. **useState** - Type inference or explicit generic for complex types
3. **useReducer** - Discriminated union for actions
4. **useRef** - Specify element type for DOM refs
5. **Context** - Create typed context with custom hook
6. **Custom hooks** - Return typed objects or tuples
7. **Events** - Use specific React event types

---

## Self-Check Questions

1. When should you use `React.FC` vs regular function?
2. How do you type optional children?
3. What's the difference between `ReactNode` and `ReactElement`?
4. How do you extend HTML element props?
5. When should you use generics in hooks?
6. How do you create a type-safe context?

---

## TypeScript + React Complete!

You've completed the TypeScript section of Phase 1. You now know:
- TypeScript fundamentals and setup
- Basic and advanced types
- Generics and utility types
- Classes and modules
- Practical TypeScript patterns
- TypeScript with React

**Next:** Continue to [Git & GitHub](../../04-git/lessons/day-01-git-basics.md) to learn version control.
