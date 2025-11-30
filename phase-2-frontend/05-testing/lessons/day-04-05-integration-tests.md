# Days 4-5: Integration Testing & API Mocking

## Introduction

Integration tests verify that multiple parts of your application work together correctly. For frontend applications, this means testing components that fetch data, interact with APIs, and coordinate state across the application.

## Learning Objectives

By the end of these lessons, you will:
- Understand integration testing strategies
- Mock APIs with MSW (Mock Service Worker)
- Test components that fetch data
- Test complex user flows
- Handle authentication in tests
- Build a comprehensive test suite

---

## Integration Testing Strategy

### What to Test

```
Unit Tests        Integration Tests       E2E Tests
(isolated)        (connected)             (full system)
    │                  │                      │
    ▼                  ▼                      ▼
┌─────────┐      ┌─────────────┐       ┌──────────────┐
│ Button  │      │ Form + API  │       │ Full Checkout│
│ Input   │      │ + Validation │       │   Flow       │
│ Utils   │      │ + Navigation│       │              │
└─────────┘      └─────────────┘       └──────────────┘
```

### Integration Test Scenarios

1. **Component + API** - Form submission, data fetching
2. **Component + State** - Context updates, store changes
3. **Component + Router** - Navigation, URL params
4. **Multiple Components** - Parent-child interactions

---

## MSW (Mock Service Worker)

MSW intercepts network requests at the service worker level, allowing you to mock APIs without changing your application code.

### Installation

```bash
npm install -D msw
```

### Setup

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // GET request
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'John Doe', email: 'john@example.com' },
      { id: '2', name: 'Jane Doe', email: 'jane@example.com' },
    ]);
  }),

  // GET with params
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params;
    return HttpResponse.json({
      id,
      name: 'John Doe',
      email: 'john@example.com',
    });
  }),

  // POST request
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      { id: '3', ...body },
      { status: 201 }
    );
  }),

  // PUT request
  http.put('/api/users/:id', async ({ params, request }) => {
    const { id } = params;
    const body = await request.json();
    return HttpResponse.json({ id, ...body });
  }),

  // DELETE request
  http.delete('/api/users/:id', () => {
    return new HttpResponse(null, { status: 204 });
  }),
];
```

### Server Setup for Tests

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### Configure Test Environment

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { server } from '../mocks/server';
import { beforeAll, afterEach, afterAll } from 'vitest';

// Start server before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset handlers after each test
afterEach(() => server.resetHandlers());

// Close server after all tests
afterAll(() => server.close());
```

### Using MSW in Tests

```typescript
import { server } from '../mocks/server';
import { http, HttpResponse } from 'msw';

describe('UserList', () => {
  test('displays users from API', async () => {
    render(<UserList />);

    // Wait for data to load
    await screen.findByText('John Doe');
    expect(screen.getByText('Jane Doe')).toBeInTheDocument();
  });

  test('handles API error', async () => {
    // Override handler for this test
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json(
          { message: 'Internal Server Error' },
          { status: 500 }
        );
      })
    );

    render(<UserList />);

    await screen.findByText(/error/i);
  });

  test('handles empty response', async () => {
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json([]);
      })
    );

    render(<UserList />);

    await screen.findByText('No users found');
  });
});
```

---

## Testing Data Fetching

### Component with useEffect Fetch

```tsx
// UserProfile.tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch user');
        return res.json();
      })
      .then(setUser)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div role="status">Loading...</div>;
  if (error) return <div role="alert">Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

```tsx
// UserProfile.test.tsx
describe('UserProfile', () => {
  test('shows loading state initially', () => {
    render(<UserProfile userId="1" />);
    expect(screen.getByRole('status')).toHaveTextContent('Loading...');
  });

  test('displays user data after loading', async () => {
    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.queryByRole('status')).not.toBeInTheDocument();
    });

    expect(screen.getByRole('heading')).toHaveTextContent('John Doe');
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  test('displays error on fetch failure', async () => {
    server.use(
      http.get('/api/users/:id', () => {
        return HttpResponse.json(
          { error: 'Not found' },
          { status: 404 }
        );
      })
    );

    render(<UserProfile userId="999" />);

    await screen.findByRole('alert');
    expect(screen.getByRole('alert')).toHaveTextContent('Error');
  });

  test('refetches when userId changes', async () => {
    const { rerender } = render(<UserProfile userId="1" />);

    await screen.findByText('John Doe');

    // Update handler for user 2
    server.use(
      http.get('/api/users/2', () => {
        return HttpResponse.json({
          id: '2',
          name: 'Jane Doe',
          email: 'jane@example.com',
        });
      })
    );

    rerender(<UserProfile userId="2" />);

    await screen.findByText('Jane Doe');
  });
});
```

### Testing with React Query

```tsx
// Using React Query for data fetching
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json()),
  });
}

function UserCard({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
    </div>
  );
}
```

```tsx
// Test setup with React Query
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        gcTime: 0,
      },
    },
  });
}

function renderWithQueryClient(ui: React.ReactElement) {
  const queryClient = createTestQueryClient();
  return render(
    <QueryClientProvider client={queryClient}>
      {ui}
    </QueryClientProvider>
  );
}

describe('UserCard', () => {
  test('fetches and displays user', async () => {
    renderWithQueryClient(<UserCard userId="1" />);

    // Shows loading skeleton
    expect(screen.getByTestId('skeleton')).toBeInTheDocument();

    // Shows user after loading
    await screen.findByRole('heading', { name: 'John Doe' });
    expect(screen.getByAltText('John Doe')).toBeInTheDocument();
  });
});
```

---

## Testing Form Submissions

### Form with API Call

```tsx
// CreateUserForm.tsx
function CreateUserForm({ onSuccess }: { onSuccess: (user: User) => void }) {
  const [formData, setFormData] = useState({ name: '', email: '' });
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitting(true);
    setError(null);

    try {
      const res = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (!res.ok) {
        const data = await res.json();
        throw new Error(data.message || 'Failed to create user');
      }

      const user = await res.json();
      onSuccess(user);
    } catch (e) {
      setError(e instanceof Error ? e.message : 'Unknown error');
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          value={formData.name}
          onChange={e => setFormData(d => ({ ...d, name: e.target.value }))}
          required
        />
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={e => setFormData(d => ({ ...d, email: e.target.value }))}
          required
        />
      </div>

      {error && <div role="alert">{error}</div>}

      <button type="submit" disabled={submitting}>
        {submitting ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

```tsx
// CreateUserForm.test.tsx
describe('CreateUserForm', () => {
  const user = userEvent.setup();

  test('submits form with valid data', async () => {
    const onSuccess = vi.fn();
    render(<CreateUserForm onSuccess={onSuccess} />);

    await user.type(screen.getByLabelText('Name'), 'New User');
    await user.type(screen.getByLabelText('Email'), 'new@example.com');
    await user.click(screen.getByRole('button', { name: 'Create User' }));

    // Button shows loading state
    expect(screen.getByRole('button')).toHaveTextContent('Creating...');
    expect(screen.getByRole('button')).toBeDisabled();

    // Wait for success
    await waitFor(() => {
      expect(onSuccess).toHaveBeenCalledWith(
        expect.objectContaining({
          name: 'New User',
          email: 'new@example.com',
        })
      );
    });
  });

  test('shows validation error from API', async () => {
    server.use(
      http.post('/api/users', () => {
        return HttpResponse.json(
          { message: 'Email already exists' },
          { status: 400 }
        );
      })
    );

    render(<CreateUserForm onSuccess={vi.fn()} />);

    await user.type(screen.getByLabelText('Name'), 'Test');
    await user.type(screen.getByLabelText('Email'), 'existing@example.com');
    await user.click(screen.getByRole('button', { name: 'Create User' }));

    await screen.findByRole('alert');
    expect(screen.getByRole('alert')).toHaveTextContent('Email already exists');
  });

  test('handles network error', async () => {
    server.use(
      http.post('/api/users', () => {
        return HttpResponse.error();
      })
    );

    render(<CreateUserForm onSuccess={vi.fn()} />);

    await user.type(screen.getByLabelText('Name'), 'Test');
    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.click(screen.getByRole('button', { name: 'Create User' }));

    await screen.findByRole('alert');
  });
});
```

---

## Testing Authentication Flows

### Auth Context

```tsx
// AuthContext.tsx
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Check for existing session
    fetch('/api/auth/me')
      .then(res => res.ok ? res.json() : null)
      .then(setUser)
      .finally(() => setIsLoading(false));
  }, []);

  const login = async (email: string, password: string) => {
    const res = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });

    if (!res.ok) {
      const data = await res.json();
      throw new Error(data.message || 'Login failed');
    }

    const user = await res.json();
    setUser(user);
  };

  const logout = async () => {
    await fetch('/api/auth/logout', { method: 'POST' });
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

### MSW Handlers for Auth

```typescript
// mocks/handlers.ts
let currentUser: User | null = null;

export const authHandlers = [
  http.get('/api/auth/me', () => {
    if (currentUser) {
      return HttpResponse.json(currentUser);
    }
    return HttpResponse.json({ message: 'Not authenticated' }, { status: 401 });
  }),

  http.post('/api/auth/login', async ({ request }) => {
    const { email, password } = await request.json();

    if (email === 'test@example.com' && password === 'password') {
      currentUser = { id: '1', name: 'Test User', email };
      return HttpResponse.json(currentUser);
    }

    return HttpResponse.json(
      { message: 'Invalid credentials' },
      { status: 401 }
    );
  }),

  http.post('/api/auth/logout', () => {
    currentUser = null;
    return new HttpResponse(null, { status: 204 });
  }),
];

// Reset auth state between tests
export function resetAuthState() {
  currentUser = null;
}
```

### Testing Auth Flow

```tsx
// LoginPage.test.tsx
describe('LoginPage', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    resetAuthState();
  });

  test('logs in with valid credentials', async () => {
    render(
      <AuthProvider>
        <LoginPage />
      </AuthProvider>
    );

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'password');
    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    await waitFor(() => {
      expect(screen.getByText('Welcome, Test User')).toBeInTheDocument();
    });
  });

  test('shows error with invalid credentials', async () => {
    render(
      <AuthProvider>
        <LoginPage />
      </AuthProvider>
    );

    await user.type(screen.getByLabelText('Email'), 'wrong@example.com');
    await user.type(screen.getByLabelText('Password'), 'wrongpassword');
    await user.click(screen.getByRole('button', { name: 'Sign In' }));

    await screen.findByText('Invalid credentials');
  });
});
```

### Testing Protected Routes

```tsx
// ProtectedRoute.tsx
function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, isLoading } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isLoading && !user) {
      navigate('/login');
    }
  }, [user, isLoading, navigate]);

  if (isLoading) return <div>Loading...</div>;
  if (!user) return null;

  return <>{children}</>;
}
```

```tsx
// ProtectedRoute.test.tsx
import { MemoryRouter, Routes, Route } from 'react-router-dom';

function renderWithRouter(
  ui: React.ReactElement,
  { route = '/' } = {}
) {
  return render(
    <MemoryRouter initialEntries={[route]}>
      <AuthProvider>
        {ui}
      </AuthProvider>
    </MemoryRouter>
  );
}

describe('ProtectedRoute', () => {
  test('redirects to login when not authenticated', async () => {
    renderWithRouter(
      <Routes>
        <Route path="/login" element={<div>Login Page</div>} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <div>Dashboard</div>
            </ProtectedRoute>
          }
        />
      </Routes>,
      { route: '/dashboard' }
    );

    await screen.findByText('Login Page');
  });

  test('shows content when authenticated', async () => {
    // Set up authenticated state
    server.use(
      http.get('/api/auth/me', () => {
        return HttpResponse.json({
          id: '1',
          name: 'Test User',
          email: 'test@example.com',
        });
      })
    );

    renderWithRouter(
      <Routes>
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <div>Dashboard</div>
            </ProtectedRoute>
          }
        />
      </Routes>,
      { route: '/dashboard' }
    );

    await screen.findByText('Dashboard');
  });
});
```

---

## Testing Complex User Flows

### E-Commerce Checkout Flow

```tsx
// CheckoutPage.tsx
function CheckoutPage() {
  const [step, setStep] = useState(1);
  const { items, total, clearCart } = useCart();
  const [orderDetails, setOrderDetails] = useState<OrderDetails | null>(null);

  const handleShippingSubmit = (data: ShippingData) => {
    setOrderDetails(prev => ({ ...prev, shipping: data }));
    setStep(2);
  };

  const handlePaymentSubmit = async (data: PaymentData) => {
    const order = await createOrder({
      items,
      shipping: orderDetails!.shipping,
      payment: data,
    });

    clearCart();
    setOrderDetails(order);
    setStep(3);
  };

  return (
    <div>
      <h1>Checkout</h1>

      {step === 1 && (
        <ShippingForm onSubmit={handleShippingSubmit} />
      )}

      {step === 2 && (
        <PaymentForm onSubmit={handlePaymentSubmit} />
      )}

      {step === 3 && (
        <OrderConfirmation order={orderDetails} />
      )}

      <OrderSummary items={items} total={total} />
    </div>
  );
}
```

```tsx
// CheckoutPage.test.tsx
describe('CheckoutPage', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    // Set up cart with items
    server.use(
      http.get('/api/cart', () => {
        return HttpResponse.json({
          items: [
            { id: '1', name: 'Product 1', price: 29.99, quantity: 2 },
            { id: '2', name: 'Product 2', price: 49.99, quantity: 1 },
          ],
          total: 109.97,
        });
      }),
      http.post('/api/orders', async ({ request }) => {
        const body = await request.json();
        return HttpResponse.json({
          id: 'order-123',
          ...body,
          status: 'confirmed',
        });
      })
    );
  });

  test('completes full checkout flow', async () => {
    render(
      <CartProvider>
        <CheckoutPage />
      </CartProvider>
    );

    // Step 1: Shipping
    await screen.findByText('Shipping Information');

    await user.type(screen.getByLabelText('Full Name'), 'John Doe');
    await user.type(screen.getByLabelText('Address'), '123 Main St');
    await user.type(screen.getByLabelText('City'), 'New York');
    await user.type(screen.getByLabelText('Zip Code'), '10001');
    await user.click(screen.getByRole('button', { name: 'Continue to Payment' }));

    // Step 2: Payment
    await screen.findByText('Payment Information');

    await user.type(screen.getByLabelText('Card Number'), '4242424242424242');
    await user.type(screen.getByLabelText('Expiry'), '12/25');
    await user.type(screen.getByLabelText('CVC'), '123');
    await user.click(screen.getByRole('button', { name: 'Place Order' }));

    // Step 3: Confirmation
    await screen.findByText('Order Confirmed!');
    expect(screen.getByText('Order #order-123')).toBeInTheDocument();
    expect(screen.getByText('Thank you for your purchase')).toBeInTheDocument();
  });

  test('shows order summary throughout checkout', async () => {
    render(
      <CartProvider>
        <CheckoutPage />
      </CartProvider>
    );

    // Order summary visible on all steps
    await screen.findByText('Product 1');
    expect(screen.getByText('Product 2')).toBeInTheDocument();
    expect(screen.getByText('$109.97')).toBeInTheDocument();
  });

  test('handles payment failure', async () => {
    server.use(
      http.post('/api/orders', () => {
        return HttpResponse.json(
          { message: 'Payment declined' },
          { status: 400 }
        );
      })
    );

    render(
      <CartProvider>
        <CheckoutPage />
      </CartProvider>
    );

    // Complete shipping
    await screen.findByText('Shipping Information');
    await user.type(screen.getByLabelText('Full Name'), 'John Doe');
    await user.type(screen.getByLabelText('Address'), '123 Main St');
    await user.type(screen.getByLabelText('City'), 'New York');
    await user.type(screen.getByLabelText('Zip Code'), '10001');
    await user.click(screen.getByRole('button', { name: 'Continue to Payment' }));

    // Try payment
    await screen.findByText('Payment Information');
    await user.type(screen.getByLabelText('Card Number'), '4000000000000002');
    await user.type(screen.getByLabelText('Expiry'), '12/25');
    await user.type(screen.getByLabelText('CVC'), '123');
    await user.click(screen.getByRole('button', { name: 'Place Order' }));

    // Error shown, still on payment step
    await screen.findByText('Payment declined');
    expect(screen.getByText('Payment Information')).toBeInTheDocument();
  });
});
```

---

## Testing with Router

### Setup Router for Tests

```tsx
import { MemoryRouter, Routes, Route } from 'react-router-dom';

function renderWithRouter(
  ui: React.ReactElement,
  {
    route = '/',
    routes = [],
  }: {
    route?: string;
    routes?: { path: string; element: React.ReactElement }[];
  } = {}
) {
  return render(
    <MemoryRouter initialEntries={[route]}>
      <Routes>
        {routes.map(({ path, element }) => (
          <Route key={path} path={path} element={element} />
        ))}
        <Route path="*" element={ui} />
      </Routes>
    </MemoryRouter>
  );
}
```

### Testing Navigation

```tsx
// Navigation.test.tsx
describe('Navigation', () => {
  const user = userEvent.setup();

  test('navigates between pages', async () => {
    renderWithRouter(<App />, {
      route: '/',
      routes: [
        { path: '/', element: <HomePage /> },
        { path: '/about', element: <AboutPage /> },
        { path: '/contact', element: <ContactPage /> },
      ],
    });

    // Start on home
    expect(screen.getByText('Welcome Home')).toBeInTheDocument();

    // Navigate to about
    await user.click(screen.getByRole('link', { name: 'About' }));
    expect(screen.getByText('About Us')).toBeInTheDocument();

    // Navigate to contact
    await user.click(screen.getByRole('link', { name: 'Contact' }));
    expect(screen.getByText('Contact Us')).toBeInTheDocument();
  });
});
```

### Testing URL Parameters

```tsx
// ProductPage.tsx
function ProductPage() {
  const { productId } = useParams();
  const { data: product } = useQuery(['product', productId], () =>
    fetchProduct(productId)
  );

  return <div>{product?.name}</div>;
}
```

```tsx
// ProductPage.test.tsx
describe('ProductPage', () => {
  test('loads product based on URL param', async () => {
    server.use(
      http.get('/api/products/:id', ({ params }) => {
        return HttpResponse.json({
          id: params.id,
          name: `Product ${params.id}`,
        });
      })
    );

    renderWithRouter(<ProductPage />, {
      route: '/products/123',
      routes: [{ path: '/products/:productId', element: <ProductPage /> }],
    });

    await screen.findByText('Product 123');
  });
});
```

---

## Testing with Zustand

```tsx
// Store
import { create } from 'zustand';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) =>
    set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) =>
    set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
  clearCart: () => set({ items: [] }),
}));
```

```tsx
// Reset store between tests
import { useCartStore } from './store';

beforeEach(() => {
  useCartStore.setState({ items: [] });
});

// Or create a helper
function resetStores() {
  useCartStore.setState({ items: [] });
  useUserStore.setState({ user: null });
}
```

```tsx
// CartPage.test.tsx
describe('CartPage', () => {
  beforeEach(() => {
    useCartStore.setState({ items: [] });
  });

  test('displays cart items', () => {
    useCartStore.setState({
      items: [
        { id: '1', name: 'Product 1', price: 10, quantity: 2 },
        { id: '2', name: 'Product 2', price: 20, quantity: 1 },
      ],
    });

    render(<CartPage />);

    expect(screen.getByText('Product 1')).toBeInTheDocument();
    expect(screen.getByText('Product 2')).toBeInTheDocument();
  });

  test('removes item from cart', async () => {
    const user = userEvent.setup();

    useCartStore.setState({
      items: [{ id: '1', name: 'Product 1', price: 10, quantity: 1 }],
    });

    render(<CartPage />);

    await user.click(screen.getByRole('button', { name: /remove/i }));

    expect(screen.queryByText('Product 1')).not.toBeInTheDocument();
    expect(useCartStore.getState().items).toHaveLength(0);
  });
});
```

---

## Building a Test Suite

### Project Structure

```
src/
├── __tests__/           # Integration tests
│   ├── auth.test.tsx
│   ├── checkout.test.tsx
│   └── search.test.tsx
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   └── Button.test.tsx    # Unit test
│   └── UserCard/
│       ├── UserCard.tsx
│       └── UserCard.test.tsx  # Component test
├── mocks/
│   ├── handlers.ts
│   ├── server.ts
│   └── fixtures/
│       ├── users.ts
│       └── products.ts
├── test/
│   ├── setup.ts
│   └── utils.tsx              # Test utilities
└── pages/
    └── Dashboard/
        ├── Dashboard.tsx
        └── Dashboard.test.tsx
```

### Test Utilities

```tsx
// test/utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { MemoryRouter } from 'react-router-dom';
import { AuthProvider } from '../contexts/AuthContext';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  route?: string;
  preloadedState?: Record<string, unknown>;
}

function AllTheProviders({ children }: { children: React.ReactNode }) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return (
    <QueryClientProvider client={queryClient}>
      <MemoryRouter>
        <AuthProvider>
          {children}
        </AuthProvider>
      </MemoryRouter>
    </QueryClientProvider>
  );
}

function customRender(
  ui: React.ReactElement,
  options?: CustomRenderOptions
) {
  return render(ui, { wrapper: AllTheProviders, ...options });
}

export * from '@testing-library/react';
export { customRender as render };
```

### Test Fixtures

```tsx
// mocks/fixtures/users.ts
export const mockUsers = {
  admin: {
    id: '1',
    name: 'Admin User',
    email: 'admin@example.com',
    role: 'admin',
  },
  regular: {
    id: '2',
    name: 'Regular User',
    email: 'user@example.com',
    role: 'user',
  },
};

export const mockUserList = Object.values(mockUsers);

// mocks/fixtures/products.ts
export const mockProducts = [
  { id: '1', name: 'Product 1', price: 29.99, stock: 10 },
  { id: '2', name: 'Product 2', price: 49.99, stock: 5 },
  { id: '3', name: 'Product 3', price: 99.99, stock: 0 },
];
```

---

## Exercise: Build Integration Tests for a Blog App

Create integration tests for this blog application:

```tsx
// BlogApp.tsx
function BlogApp() {
  return (
    <Routes>
      <Route path="/" element={<PostList />} />
      <Route path="/posts/:id" element={<PostDetail />} />
      <Route path="/posts/new" element={<CreatePost />} />
    </Routes>
  );
}

// PostList.tsx
function PostList() {
  const { data: posts, isLoading } = useQuery(['posts'], fetchPosts);

  if (isLoading) return <LoadingSpinner />;

  return (
    <div>
      <h1>Blog Posts</h1>
      <Link to="/posts/new">Create Post</Link>
      {posts?.map(post => (
        <article key={post.id}>
          <h2><Link to={`/posts/${post.id}`}>{post.title}</Link></h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// PostDetail.tsx
function PostDetail() {
  const { id } = useParams();
  const navigate = useNavigate();
  const { data: post, isLoading } = useQuery(['post', id], () => fetchPost(id));

  const deleteMutation = useMutation(deletePost, {
    onSuccess: () => navigate('/'),
  });

  if (isLoading) return <LoadingSpinner />;
  if (!post) return <div>Post not found</div>;

  return (
    <article>
      <h1>{post.title}</h1>
      <p>By {post.author}</p>
      <div>{post.content}</div>
      <button onClick={() => deleteMutation.mutate(post.id)}>Delete</button>
    </article>
  );
}

// CreatePost.tsx
function CreatePost() {
  const navigate = useNavigate();
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const mutation = useMutation(createPost, {
    onSuccess: (post) => navigate(`/posts/${post.id}`),
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    mutation.mutate({ title, content });
  };

  return (
    <form onSubmit={handleSubmit}>
      <h1>Create Post</h1>
      <input
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="Title"
        aria-label="Title"
      />
      <textarea
        value={content}
        onChange={e => setContent(e.target.value)}
        placeholder="Content"
        aria-label="Content"
      />
      <button type="submit" disabled={mutation.isLoading}>
        {mutation.isLoading ? 'Publishing...' : 'Publish'}
      </button>
      {mutation.error && <div role="alert">Failed to create post</div>}
    </form>
  );
}
```

---

## Solution

```tsx
// BlogApp.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, waitFor } from './test/utils';
import userEvent from '@testing-library/user-event';
import { server } from './mocks/server';
import { http, HttpResponse } from 'msw';
import { BlogApp } from './BlogApp';

const mockPosts = [
  { id: '1', title: 'First Post', excerpt: 'This is the first post', author: 'John', content: 'Full content here' },
  { id: '2', title: 'Second Post', excerpt: 'This is the second post', author: 'Jane', content: 'More content' },
];

describe('BlogApp Integration', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    server.use(
      http.get('/api/posts', () => {
        return HttpResponse.json(mockPosts);
      }),
      http.get('/api/posts/:id', ({ params }) => {
        const post = mockPosts.find(p => p.id === params.id);
        if (!post) {
          return HttpResponse.json({ message: 'Not found' }, { status: 404 });
        }
        return HttpResponse.json(post);
      }),
      http.post('/api/posts', async ({ request }) => {
        const body = await request.json();
        return HttpResponse.json({ id: '3', ...body }, { status: 201 });
      }),
      http.delete('/api/posts/:id', () => {
        return new HttpResponse(null, { status: 204 });
      })
    );
  });

  describe('PostList', () => {
    it('displays list of posts', async () => {
      render(<BlogApp />, { route: '/' });

      await screen.findByRole('heading', { name: 'Blog Posts' });

      expect(screen.getByText('First Post')).toBeInTheDocument();
      expect(screen.getByText('Second Post')).toBeInTheDocument();
    });

    it('shows loading state', () => {
      render(<BlogApp />, { route: '/' });

      expect(screen.getByRole('status')).toBeInTheDocument();
    });

    it('links to create post page', async () => {
      render(<BlogApp />, { route: '/' });

      await screen.findByRole('heading', { name: 'Blog Posts' });

      await user.click(screen.getByRole('link', { name: 'Create Post' }));

      expect(screen.getByRole('heading', { name: 'Create Post' })).toBeInTheDocument();
    });
  });

  describe('PostDetail', () => {
    it('displays post content', async () => {
      render(<BlogApp />, { route: '/posts/1' });

      await screen.findByRole('heading', { name: 'First Post' });

      expect(screen.getByText('By John')).toBeInTheDocument();
      expect(screen.getByText('Full content here')).toBeInTheDocument();
    });

    it('handles post not found', async () => {
      server.use(
        http.get('/api/posts/:id', () => {
          return HttpResponse.json({ message: 'Not found' }, { status: 404 });
        })
      );

      render(<BlogApp />, { route: '/posts/999' });

      await screen.findByText('Post not found');
    });

    it('deletes post and navigates to home', async () => {
      render(<BlogApp />, { route: '/posts/1' });

      await screen.findByRole('heading', { name: 'First Post' });

      await user.click(screen.getByRole('button', { name: 'Delete' }));

      await screen.findByRole('heading', { name: 'Blog Posts' });
    });
  });

  describe('CreatePost', () => {
    it('creates new post and navigates to it', async () => {
      render(<BlogApp />, { route: '/posts/new' });

      await user.type(screen.getByLabelText('Title'), 'New Post Title');
      await user.type(screen.getByLabelText('Content'), 'New post content here');
      await user.click(screen.getByRole('button', { name: 'Publish' }));

      // Should navigate to new post
      await waitFor(() => {
        expect(screen.getByRole('heading', { name: 'New Post Title' })).toBeInTheDocument();
      });
    });

    it('shows loading state during submission', async () => {
      render(<BlogApp />, { route: '/posts/new' });

      await user.type(screen.getByLabelText('Title'), 'Test');
      await user.type(screen.getByLabelText('Content'), 'Content');
      await user.click(screen.getByRole('button', { name: 'Publish' }));

      expect(screen.getByRole('button', { name: 'Publishing...' })).toBeDisabled();
    });

    it('shows error on failure', async () => {
      server.use(
        http.post('/api/posts', () => {
          return HttpResponse.json({ message: 'Failed' }, { status: 500 });
        })
      );

      render(<BlogApp />, { route: '/posts/new' });

      await user.type(screen.getByLabelText('Title'), 'Test');
      await user.type(screen.getByLabelText('Content'), 'Content');
      await user.click(screen.getByRole('button', { name: 'Publish' }));

      await screen.findByRole('alert');
      expect(screen.getByRole('alert')).toHaveTextContent('Failed to create post');
    });
  });

  describe('Navigation Flow', () => {
    it('completes full user journey', async () => {
      render(<BlogApp />, { route: '/' });

      // View post list
      await screen.findByText('First Post');

      // Navigate to post detail
      await user.click(screen.getByRole('link', { name: 'First Post' }));
      await screen.findByText('Full content here');

      // Go back (using browser back button simulation)
      // In real app, would test back button or breadcrumb

      // Navigate to create post
      render(<BlogApp />, { route: '/' });
      await screen.findByText('First Post');
      await user.click(screen.getByRole('link', { name: 'Create Post' }));

      // Create new post
      await user.type(screen.getByLabelText('Title'), 'My New Post');
      await user.type(screen.getByLabelText('Content'), 'Amazing content');
      await user.click(screen.getByRole('button', { name: 'Publish' }));

      // Verify navigation to new post
      await screen.findByRole('heading', { name: 'My New Post' });
    });
  });
});
```

---

## Key Takeaways

1. **MSW is the standard** - Mock at the network level, not in components
2. **Test user flows** - Integration tests should mirror real usage
3. **Reset state between tests** - Use beforeEach to ensure isolation
4. **Custom render for providers** - Wrap with all necessary context
5. **Test error states** - Override handlers to test failures
6. **Test loading states** - Verify UI during async operations
7. **Combine with unit tests** - Integration tests complement, not replace

---

## What's Next?

Congratulations! You've completed the Frontend Testing section. You now have the skills to:
- Write unit tests with Jest/Vitest
- Test React components with Testing Library
- Mock APIs with MSW
- Write comprehensive integration tests

Next up: **Phase 3 - Backend Development** with Node.js and Express!
