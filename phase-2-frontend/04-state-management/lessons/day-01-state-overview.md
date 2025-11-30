# Day 1: State Management Overview - Choosing the Right Solution

## Introduction

State management is one of the most important decisions in React application architecture. Understanding when to use local state, context, or external libraries helps build maintainable and performant applications.

## Learning Objectives

By the end of this lesson, you will:
- Understand different types of state
- Know when to use each state management solution
- Compare popular state management libraries
- Choose the right tool for your project

---

## Types of State

### 1. Local/Component State

State that belongs to a single component:

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Use for:**
- Form inputs
- UI toggles (modals, dropdowns)
- Component-specific data

### 2. Shared/Lifted State

State shared between sibling components:

```tsx
function Parent() {
  const [selectedId, setSelectedId] = useState(null);

  return (
    <>
      <List items={items} onSelect={setSelectedId} />
      <Details id={selectedId} />
    </>
  );
}
```

### 3. Global State

State needed across many components:

```tsx
// Theme, user authentication, cart items, etc.
<ThemeProvider value={theme}>
  <UserProvider value={user}>
    <App />
  </UserProvider>
</ThemeProvider>
```

### 4. Server State

Data from external sources (APIs, databases):

```tsx
// Caching, synchronization, background updates
const { data, isLoading } = useQuery(['users'], fetchUsers);
```

### 5. URL State

State stored in the URL:

```tsx
// Search params, pagination, filters
const searchParams = useSearchParams();
const page = searchParams.get('page');
```

---

## State Management Solutions

### Built-in React

| Solution | Best For |
|----------|----------|
| `useState` | Local component state |
| `useReducer` | Complex state logic |
| `Context` | Infrequent global updates |
| URL params | Shareable/bookmarkable state |

### External Libraries

| Library | Best For |
|---------|----------|
| **Zustand** | Simple global state |
| **Jotai** | Atomic state management |
| **Redux Toolkit** | Complex enterprise apps |
| **React Query** | Server state/caching |
| **SWR** | Simple data fetching |

---

## Decision Tree

```
Need to manage state?
│
├─ Is it server data?
│  └─ YES → React Query or SWR
│
├─ Is it form data?
│  └─ YES → React Hook Form or local state
│
├─ Is it shareable via URL?
│  └─ YES → URL search params
│
├─ Does it need to be global?
│  │
│  ├─ NO → useState or useReducer
│  │
│  └─ YES → Does it change frequently?
│           │
│           ├─ NO → React Context
│           │
│           └─ YES → Zustand or Redux
```

---

## When to Use What

### useState

```tsx
// ✅ Perfect for:
const [isOpen, setIsOpen] = useState(false);      // UI toggles
const [inputValue, setInputValue] = useState(''); // Form inputs
const [activeTab, setActiveTab] = useState(0);    // Local selection
```

### useReducer

```tsx
// ✅ Perfect for:
// - Complex state with multiple sub-values
// - State transitions that depend on previous state
// - When you want to centralize state logic

const [state, dispatch] = useReducer(reducer, {
  items: [],
  filter: 'all',
  sortBy: 'date',
  page: 1,
});
```

### Context

```tsx
// ✅ Good for:
// - Theme
// - User authentication
// - Locale/language
// - Feature flags
// Data that doesn't change often!

// ❌ Bad for:
// - Frequently updating values
// - Large amounts of data
// - Performance-critical state
```

### Zustand

```tsx
// ✅ Perfect for:
// - Global UI state
// - Shopping carts
// - App-wide settings
// - When Context causes too many re-renders

const useStore = create((set) => ({
  cart: [],
  addItem: (item) => set((state) => ({
    cart: [...state.cart, item]
  })),
}));
```

### React Query / SWR

```tsx
// ✅ Perfect for:
// - API data fetching
// - Caching and synchronization
// - Optimistic updates
// - Background refetching
// - Pagination/infinite scroll

const { data, isLoading, error } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
});
```

---

## Common Patterns

### Pattern 1: Colocation

Keep state close to where it's used:

```tsx
// ❌ Bad - state too high
function App() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <div>
      <Header />
      <SearchResults query={searchQuery} />
      <Footer />
    </div>
  );
}

// ✅ Good - state colocated
function App() {
  return (
    <div>
      <Header />
      <Search /> {/* State lives here */}
      <Footer />
    </div>
  );
}

function Search() {
  const [query, setQuery] = useState('');
  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <SearchResults query={query} />
    </>
  );
}
```

### Pattern 2: State Composition

Compose multiple state sources:

```tsx
function ProductPage({ productId }) {
  // Server state
  const { data: product } = useQuery(['product', productId], () =>
    fetchProduct(productId)
  );

  // Local UI state
  const [selectedVariant, setSelectedVariant] = useState(null);

  // Global state
  const { addToCart } = useCartStore();

  // URL state
  const searchParams = useSearchParams();
  const tab = searchParams.get('tab') || 'details';

  return (
    <div>
      <ProductInfo product={product} />
      <VariantSelector
        variants={product?.variants}
        selected={selectedVariant}
        onSelect={setSelectedVariant}
      />
      <Tabs activeTab={tab} />
      <AddToCartButton onClick={() => addToCart(product, selectedVariant)} />
    </div>
  );
}
```

### Pattern 3: Derived State

Calculate from existing state instead of storing:

```tsx
// ❌ Bad - redundant state
const [items, setItems] = useState([]);
const [filteredItems, setFilteredItems] = useState([]);
const [totalPrice, setTotalPrice] = useState(0);

// ✅ Good - derived values
const [items, setItems] = useState([]);
const [filter, setFilter] = useState('all');

const filteredItems = useMemo(
  () => items.filter(item => filter === 'all' || item.category === filter),
  [items, filter]
);

const totalPrice = useMemo(
  () => items.reduce((sum, item) => sum + item.price, 0),
  [items]
);
```

---

## Anti-Patterns to Avoid

### 1. Putting Everything in Global State

```tsx
// ❌ Bad
const useStore = create((set) => ({
  isModalOpen: false,          // Should be local
  inputValue: '',              // Should be local
  isLoading: false,            // Part of server state
  users: [],                   // Server state
  currentPage: 1,              // URL state
}));

// ✅ Good - separate concerns
// Local state for modal
// React Query for users
// URL for pagination
// Zustand only for truly global state
```

### 2. Prop Drilling vs Over-Globalizing

```tsx
// ❌ Bad - too much prop drilling
<App>
  <Layout user={user}>
    <Header user={user}>
      <Nav user={user}>
        <UserMenu user={user} />
      </Nav>
    </Header>
  </Layout>
</App>

// ❌ Also bad - everything global
useStore.setState({ temporaryFormValue: 'x' });

// ✅ Good - Context for user, local state for forms
<UserProvider value={user}>
  <App />
</UserProvider>
```

### 3. Synchronizing State

```tsx
// ❌ Bad - keeping state in sync
const [users, setUsers] = useState([]);
const [selectedUser, setSelectedUser] = useState(null);

// When users change, selectedUser might be stale!

// ✅ Good - store only the ID
const [users, setUsers] = useState([]);
const [selectedUserId, setSelectedUserId] = useState(null);

const selectedUser = users.find(u => u.id === selectedUserId);
```

---

## Comparison Summary

| Feature | Context | Zustand | Redux | React Query |
|---------|---------|---------|-------|-------------|
| Learning Curve | Low | Low | High | Medium |
| Boilerplate | Low | Very Low | High | Low |
| DevTools | React DevTools | Yes | Excellent | Yes |
| Async | Manual | Built-in | Middleware | Built-in |
| Performance | Requires splitting | Good | Good | Excellent |
| Server State | No | No | With middleware | Yes |
| Bundle Size | 0 | ~1KB | ~7KB | ~13KB |

---

## Exercise: State Audit

Analyze this component and identify the best state management approach:

```tsx
function EcommercePage() {
  // What type of state is each of these?
  // What solution would you use?

  const [products, setProducts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [cart, setCart] = useState([]);
  const [user, setUser] = useState(null);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [isCartOpen, setIsCartOpen] = useState(false);
  const [sortBy, setSortBy] = useState('price');

  // ...
}
```

---

## Solution

```tsx
function EcommercePage() {
  // 1. products, isLoading → SERVER STATE
  // Use: React Query
  const { data: products, isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });

  // 2. cart → GLOBAL STATE (persisted, used across pages)
  // Use: Zustand with persistence
  const { cart, addToCart } = useCartStore();

  // 3. user → GLOBAL STATE (auth)
  // Use: Auth Context or Auth library
  const { user } = useAuth();

  // 4. searchQuery, selectedCategory, sortBy → URL STATE
  // Use: URL search params (shareable, bookmarkable)
  const searchParams = useSearchParams();
  const searchQuery = searchParams.get('q') || '';
  const selectedCategory = searchParams.get('category') || 'all';
  const sortBy = searchParams.get('sort') || 'price';

  // 5. isCartOpen → LOCAL STATE
  // Use: useState (only this component needs it)
  const [isCartOpen, setIsCartOpen] = useState(false);

  return (
    <div>
      <SearchBar query={searchQuery} />
      <CategoryFilter category={selectedCategory} />
      <SortSelect value={sortBy} />
      <ProductGrid products={products} isLoading={isLoading} />
      <CartButton onClick={() => setIsCartOpen(true)} count={cart.length} />
      {isCartOpen && <CartModal onClose={() => setIsCartOpen(false)} />}
    </div>
  );
}
```

---

## Key Takeaways

1. **Start simple** - Use useState until you need something else
2. **Colocate state** - Keep it close to where it's used
3. **Separate concerns** - Different types of state need different solutions
4. **Server state is special** - Use React Query or SWR
5. **URL state is shareable** - Use for filters, pagination, tabs
6. **Global state sparingly** - Only for truly app-wide data
7. **Derive when possible** - Calculate instead of storing

---

## What's Next?

Tomorrow we'll dive deep into **Zustand** - a minimal, flexible state management library that's perfect for most applications!
