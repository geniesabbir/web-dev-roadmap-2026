# Day 2: Zustand - Simple, Scalable State Management

## Introduction

Zustand is a small, fast, and scalable state management solution for React. With its minimal API and excellent TypeScript support, it's become one of the most popular alternatives to Redux.

## Learning Objectives

By the end of this lesson, you will:
- Set up Zustand in a React application
- Create stores with actions and selectors
- Implement persistence and middleware
- Handle async operations
- Structure stores for scalability

---

## Getting Started

### Installation

```bash
npm install zustand
```

### Basic Store

```tsx
// stores/counterStore.ts
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

### Using the Store

```tsx
function Counter() {
  const { count, increment, decrement, reset } = useCounterStore();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Or select specific values
function CountDisplay() {
  const count = useCounterStore((state) => state.count);
  return <p>Count: {count}</p>;
}

function CounterButtons() {
  const increment = useCounterStore((state) => state.increment);
  const decrement = useCounterStore((state) => state.decrement);

  return (
    <>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </>
  );
}
```

---

## Store Patterns

### Complete Todo Store

```tsx
// stores/todoStore.ts
import { create } from 'zustand';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';

  // Actions
  addTodo: (text: string) => void;
  toggleTodo: (id: string) => void;
  deleteTodo: (id: string) => void;
  setFilter: (filter: TodoState['filter']) => void;
  clearCompleted: () => void;

  // Computed (not stored, but defined in store)
  getFilteredTodos: () => Todo[];
  getStats: () => { total: number; active: number; completed: number };
}

export const useTodoStore = create<TodoState>((set, get) => ({
  todos: [],
  filter: 'all',

  addTodo: (text) =>
    set((state) => ({
      todos: [
        ...state.todos,
        {
          id: crypto.randomUUID(),
          text,
          completed: false,
          createdAt: new Date(),
        },
      ],
    })),

  toggleTodo: (id) =>
    set((state) => ({
      todos: state.todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      ),
    })),

  deleteTodo: (id) =>
    set((state) => ({
      todos: state.todos.filter((todo) => todo.id !== id),
    })),

  setFilter: (filter) => set({ filter }),

  clearCompleted: () =>
    set((state) => ({
      todos: state.todos.filter((todo) => !todo.completed),
    })),

  getFilteredTodos: () => {
    const { todos, filter } = get();
    switch (filter) {
      case 'active':
        return todos.filter((t) => !t.completed);
      case 'completed':
        return todos.filter((t) => t.completed);
      default:
        return todos;
    }
  },

  getStats: () => {
    const { todos } = get();
    return {
      total: todos.length,
      active: todos.filter((t) => !t.completed).length,
      completed: todos.filter((t) => t.completed).length,
    };
  },
}));
```

### Using the Todo Store

```tsx
function TodoApp() {
  return (
    <div>
      <TodoForm />
      <TodoFilters />
      <TodoList />
      <TodoStats />
    </div>
  );
}

function TodoForm() {
  const [text, setText] = useState('');
  const addTodo = useTodoStore((state) => state.addTodo);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text.trim());
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add todo..."
      />
      <button type="submit">Add</button>
    </form>
  );
}

function TodoFilters() {
  const { filter, setFilter } = useTodoStore((state) => ({
    filter: state.filter,
    setFilter: state.setFilter,
  }));

  return (
    <div>
      {(['all', 'active', 'completed'] as const).map((f) => (
        <button
          key={f}
          onClick={() => setFilter(f)}
          className={filter === f ? 'active' : ''}
        >
          {f}
        </button>
      ))}
    </div>
  );
}

function TodoList() {
  const getFilteredTodos = useTodoStore((state) => state.getFilteredTodos);
  const toggleTodo = useTodoStore((state) => state.toggleTodo);
  const deleteTodo = useTodoStore((state) => state.deleteTodo);

  const todos = getFilteredTodos();

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span className={todo.completed ? 'completed' : ''}>{todo.text}</span>
          <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

function TodoStats() {
  const getStats = useTodoStore((state) => state.getStats);
  const stats = getStats();

  return (
    <div>
      <span>Total: {stats.total}</span>
      <span>Active: {stats.active}</span>
      <span>Completed: {stats.completed}</span>
    </div>
  );
}
```

---

## Persistence

### With localStorage

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface SettingsState {
  theme: 'light' | 'dark';
  language: string;
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (language: string) => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: 'settings-storage', // localStorage key
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        // Only persist these fields
        theme: state.theme,
        language: state.language,
      }),
    }
  )
);
```

### With sessionStorage

```tsx
export const useSessionStore = create<State>()(
  persist(
    (set) => ({
      // ...
    }),
    {
      name: 'session-storage',
      storage: createJSONStorage(() => sessionStorage),
    }
  )
);
```

### Hydration Handling

```tsx
// For SSR/Next.js apps
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [isHydrated, setIsHydrated] = useState(false);
  const theme = useSettingsStore((state) => state.theme);

  useEffect(() => {
    setIsHydrated(true);
  }, []);

  if (!isHydrated) {
    return <div className="loading">Loading...</div>;
  }

  return <div data-theme={theme}>{children}</div>;
}
```

---

## Async Actions

### Basic Async

```tsx
interface UserState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  fetchUser: (id: string) => Promise<void>;
  updateUser: (data: Partial<User>) => Promise<void>;
}

export const useUserStore = create<UserState>((set, get) => ({
  user: null,
  isLoading: false,
  error: null,

  fetchUser: async (id) => {
    set({ isLoading: true, error: null });

    try {
      const response = await fetch(`/api/users/${id}`);
      if (!response.ok) throw new Error('Failed to fetch user');
      const user = await response.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false,
      });
    }
  },

  updateUser: async (data) => {
    const { user } = get();
    if (!user) return;

    set({ isLoading: true, error: null });

    try {
      const response = await fetch(`/api/users/${user.id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) throw new Error('Failed to update user');

      const updatedUser = await response.json();
      set({ user: updatedUser, isLoading: false });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false,
      });
    }
  },
}));
```

### Optimistic Updates

```tsx
interface CartState {
  items: CartItem[];
  addItem: (item: CartItem) => Promise<void>;
  removeItem: (id: string) => Promise<void>;
}

export const useCartStore = create<CartState>((set, get) => ({
  items: [],

  addItem: async (item) => {
    const previousItems = get().items;

    // Optimistic update
    set((state) => ({
      items: [...state.items, item],
    }));

    try {
      await fetch('/api/cart', {
        method: 'POST',
        body: JSON.stringify(item),
      });
    } catch (error) {
      // Rollback on error
      set({ items: previousItems });
      throw error;
    }
  },

  removeItem: async (id) => {
    const previousItems = get().items;

    // Optimistic update
    set((state) => ({
      items: state.items.filter((item) => item.id !== id),
    }));

    try {
      await fetch(`/api/cart/${id}`, { method: 'DELETE' });
    } catch (error) {
      // Rollback on error
      set({ items: previousItems });
      throw error;
    }
  },
}));
```

---

## Middleware

### Logging Middleware

```tsx
import { create, StateCreator } from 'zustand';

const log = <T extends object>(
  config: StateCreator<T>
): StateCreator<T> => (set, get, api) =>
  config(
    (...args) => {
      console.log('  applying', args);
      set(...args);
      console.log('  new state', get());
    },
    get,
    api
  );

export const useStore = create<State>()(
  log((set) => ({
    // ...store
  }))
);
```

### Immer Middleware (Mutable Updates)

```bash
npm install immer
```

```tsx
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface State {
  users: User[];
  updateUser: (id: string, data: Partial<User>) => void;
  addNestedItem: (userId: string, item: Item) => void;
}

export const useStore = create<State>()(
  immer((set) => ({
    users: [],

    // Can mutate directly with immer
    updateUser: (id, data) =>
      set((state) => {
        const user = state.users.find((u) => u.id === id);
        if (user) {
          Object.assign(user, data);
        }
      }),

    addNestedItem: (userId, item) =>
      set((state) => {
        const user = state.users.find((u) => u.id === userId);
        if (user) {
          user.items.push(item);
        }
      }),
  }))
);
```

### DevTools

```tsx
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

export const useStore = create<State>()(
  devtools(
    (set) => ({
      count: 0,
      increment: () =>
        set(
          (state) => ({ count: state.count + 1 }),
          false, // replace: false
          'increment' // action name for devtools
        ),
    }),
    { name: 'Counter Store' }
  )
);
```

### Combining Middleware

```tsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export const useStore = create<State>()(
  devtools(
    persist(
      immer((set) => ({
        // store
      })),
      { name: 'app-storage' }
    ),
    { name: 'App Store' }
  )
);
```

---

## Store Slices

### Splitting Large Stores

```tsx
// stores/slices/userSlice.ts
import { StateCreator } from 'zustand';

export interface UserSlice {
  user: User | null;
  setUser: (user: User | null) => void;
}

export const createUserSlice: StateCreator<
  UserSlice & CartSlice,
  [],
  [],
  UserSlice
> = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
});

// stores/slices/cartSlice.ts
export interface CartSlice {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
}

export const createCartSlice: StateCreator<
  UserSlice & CartSlice,
  [],
  [],
  CartSlice
> = (set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) =>
    set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
});

// stores/index.ts
import { create } from 'zustand';
import { createUserSlice, UserSlice } from './slices/userSlice';
import { createCartSlice, CartSlice } from './slices/cartSlice';

export const useStore = create<UserSlice & CartSlice>()((...a) => ({
  ...createUserSlice(...a),
  ...createCartSlice(...a),
}));
```

---

## Best Practices

### 1. Use Selectors

```tsx
// ❌ Bad - subscribes to entire store
function Component() {
  const store = useStore();
  return <div>{store.count}</div>;
}

// ✅ Good - subscribes only to count
function Component() {
  const count = useStore((state) => state.count);
  return <div>{count}</div>;
}
```

### 2. Memoize Selectors for Objects

```tsx
import { useShallow } from 'zustand/react/shallow';

function Component() {
  // ✅ Prevents re-renders when other state changes
  const { name, email } = useStore(
    useShallow((state) => ({
      name: state.user?.name,
      email: state.user?.email,
    }))
  );
}
```

### 3. Keep Actions in Store

```tsx
// ❌ Bad - action outside store
function Component() {
  const setCount = useStore((state) => state.setCount);

  const increment = () => {
    setCount(useStore.getState().count + 1);
  };
}

// ✅ Good - action in store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));
```

### 4. Type Everything

```tsx
interface State {
  count: number;
  user: User | null;
  increment: () => void;
  setUser: (user: User | null) => void;
}

export const useStore = create<State>()((set) => ({
  // TypeScript will enforce correct types
}));
```

---

## Exercise: Shopping Cart Store

Create a complete shopping cart store with:
- Add/remove items
- Update quantities
- Persistence
- Total calculation
- Coupon codes

---

## Solution

```tsx
// stores/cartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
}

interface Coupon {
  code: string;
  discount: number; // percentage
}

interface CartState {
  items: CartItem[];
  coupon: Coupon | null;

  // Actions
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  applyCoupon: (code: string) => Promise<boolean>;
  removeCoupon: () => void;
  clearCart: () => void;

  // Computed
  getSubtotal: () => number;
  getDiscount: () => number;
  getTotal: () => number;
  getItemCount: () => number;
}

const COUPONS: Record<string, number> = {
  SAVE10: 10,
  SAVE20: 20,
  HALF: 50,
};

export const useCartStore = create<CartState>()(
  persist(
    immer((set, get) => ({
      items: [],
      coupon: null,

      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.id === item.id);
          if (existing) {
            existing.quantity += 1;
          } else {
            state.items.push({ ...item, quantity: 1 });
          }
        }),

      removeItem: (id) =>
        set((state) => {
          state.items = state.items.filter((item) => item.id !== id);
        }),

      updateQuantity: (id, quantity) =>
        set((state) => {
          if (quantity <= 0) {
            state.items = state.items.filter((item) => item.id !== id);
            return;
          }
          const item = state.items.find((i) => i.id === id);
          if (item) {
            item.quantity = quantity;
          }
        }),

      applyCoupon: async (code) => {
        // Simulate API call
        await new Promise((r) => setTimeout(r, 500));

        const discount = COUPONS[code.toUpperCase()];
        if (discount) {
          set({ coupon: { code: code.toUpperCase(), discount } });
          return true;
        }
        return false;
      },

      removeCoupon: () => set({ coupon: null }),

      clearCart: () => set({ items: [], coupon: null }),

      getSubtotal: () => {
        return get().items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        );
      },

      getDiscount: () => {
        const { coupon } = get();
        if (!coupon) return 0;
        return get().getSubtotal() * (coupon.discount / 100);
      },

      getTotal: () => {
        return get().getSubtotal() - get().getDiscount();
      },

      getItemCount: () => {
        return get().items.reduce((sum, item) => sum + item.quantity, 0);
      },
    })),
    {
      name: 'cart-storage',
      partialize: (state) => ({
        items: state.items,
        coupon: state.coupon,
      }),
    }
  )
);

// Usage
function Cart() {
  const { items, removeItem, updateQuantity, clearCart } = useCartStore();
  const subtotal = useCartStore((s) => s.getSubtotal());
  const discount = useCartStore((s) => s.getDiscount());
  const total = useCartStore((s) => s.getTotal());

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>
          <img src={item.image} alt={item.name} />
          <span>{item.name}</span>
          <span>${item.price}</span>
          <input
            type="number"
            value={item.quantity}
            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
            min="0"
          />
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}

      <CouponForm />

      <div>
        <p>Subtotal: ${subtotal.toFixed(2)}</p>
        {discount > 0 && <p>Discount: -${discount.toFixed(2)}</p>}
        <p>Total: ${total.toFixed(2)}</p>
      </div>

      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}

function CouponForm() {
  const [code, setCode] = useState('');
  const [error, setError] = useState('');
  const { coupon, applyCoupon, removeCoupon } = useCartStore();

  const handleApply = async () => {
    setError('');
    const success = await applyCoupon(code);
    if (success) {
      setCode('');
    } else {
      setError('Invalid coupon code');
    }
  };

  if (coupon) {
    return (
      <div>
        <span>Coupon: {coupon.code} ({coupon.discount}% off)</span>
        <button onClick={removeCoupon}>Remove</button>
      </div>
    );
  }

  return (
    <div>
      <input
        value={code}
        onChange={(e) => setCode(e.target.value)}
        placeholder="Coupon code"
      />
      <button onClick={handleApply}>Apply</button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

---

## Key Takeaways

1. **Zustand is simple** - minimal API, no providers needed
2. **Use selectors** to prevent unnecessary re-renders
3. **Persist middleware** for localStorage/sessionStorage
4. **Immer middleware** for mutable-style updates
5. **Slice pattern** for large stores
6. **DevTools** for debugging
7. **Type everything** for better DX

---

## What's Next?

Tomorrow we'll learn **React Query** - the ultimate solution for server state management!
