# State Management

**Duration:** 1-2 weeks

## Learning Objectives

By the end of this section, you will:
- Understand different types of state
- Choose the right tool for each situation
- Implement Zustand for client state
- Use React Query for server state
- Know when you don't need a state library

---

## State Categories

### 1. Local State
- Single component state
- Use: `useState`, `useReducer`

### 2. Lifted State
- Shared between few components
- Use: Props, Context

### 3. Global Client State
- App-wide UI state (theme, sidebar, modals)
- Use: Zustand, Redux, Context

### 4. Server State
- Data from APIs
- Use: React Query, SWR

### 5. URL State
- Filters, pagination, search
- Use: searchParams, nuqs

### 6. Form State
- Form inputs and validation
- Use: React Hook Form, useActionState

---

## Week 1: Client State

### Day 1: When You Don't Need State Libraries
**Topics:**
- React's built-in state is powerful
- Prop drilling isn't always bad
- Context for simple global state
- Server Components reduce client state
- URL as state

**Exercise:**
Refactor an over-engineered app to use simpler patterns.

### Day 2-3: Zustand
**Topics:**
- Why Zustand?
- Creating stores
- Actions and selectors
- TypeScript integration
- Devtools
- Persistence middleware
- Slices pattern

**Setup:**
```bash
npm install zustand
```

**Basic Store:**
```tsx
// stores/useCartStore.ts
import { create } from 'zustand'

interface CartItem {
  id: string
  name: string
  price: number
  quantity: number
}

interface CartStore {
  items: CartItem[]
  addItem: (item: Omit<CartItem, 'quantity'>) => void
  removeItem: (id: string) => void
  updateQuantity: (id: string, quantity: number) => void
  clearCart: () => void
  totalItems: () => number
  totalPrice: () => number
}

export const useCartStore = create<CartStore>((set, get) => ({
  items: [],

  addItem: (item) =>
    set((state) => {
      const existing = state.items.find((i) => i.id === item.id)
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
        }
      }
      return { items: [...state.items, { ...item, quantity: 1 }] }
    }),

  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter((i) => i.id !== id),
    })),

  updateQuantity: (id, quantity) =>
    set((state) => ({
      items: state.items.map((i) =>
        i.id === id ? { ...i, quantity } : i
      ),
    })),

  clearCart: () => set({ items: [] }),

  totalItems: () => get().items.reduce((acc, item) => acc + item.quantity, 0),

  totalPrice: () =>
    get().items.reduce((acc, item) => acc + item.price * item.quantity, 0),
}))
```

**Using the Store:**
```tsx
// components/Cart.tsx
import { useCartStore } from '@/stores/useCartStore'

export function Cart() {
  const items = useCartStore((state) => state.items)
  const totalPrice = useCartStore((state) => state.totalPrice())
  const removeItem = useCartStore((state) => state.removeItem)

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>
          <span>{item.name}</span>
          <span>${item.price} x {item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <div>Total: ${totalPrice}</div>
    </div>
  )
}
```

**With Persistence:**
```tsx
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      // ... same as above
    }),
    {
      name: 'cart-storage',
    }
  )
)
```

**Exercises:**
```tsx
// exercises/zustand/

// 1. Build a theme store (light/dark/system)
// 2. Build a sidebar store (open/closed, collapsed)
// 3. Build a notification store (add, remove, auto-dismiss)
// 4. Add persistence to theme store
```

### Day 4-5: Redux Toolkit (Optional)
**Topics:**
- When Redux makes sense
- Redux Toolkit basics
- Slices
- RTK Query
- Comparison with Zustand

**Note:** Redux is more complex but valuable for:
- Very large applications
- Teams with Redux experience
- Complex state dependencies
- When you need RTK Query

**Basic Redux Toolkit:**
```tsx
// store/slices/counterSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

interface CounterState {
  value: number
}

const initialState: CounterState = { value: 0 }

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1
    },
    decrement: (state) => {
      state.value -= 1
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload
    },
  },
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions
export default counterSlice.reducer
```

---

## Week 2: Server State

### Day 1-3: React Query (TanStack Query)
**Topics:**
- Why React Query?
- Queries and mutations
- Caching and stale time
- Background refetching
- Optimistic updates
- Infinite queries
- Prefetching

**Setup:**
```bash
npm install @tanstack/react-query
```

**Provider Setup:**
```tsx
// app/providers.tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { useState } from 'react'

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000,
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

**Basic Query:**
```tsx
// hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query'

interface User {
  id: number
  name: string
  email: string
}

async function fetchUsers(): Promise<User[]> {
  const res = await fetch('/api/users')
  if (!res.ok) throw new Error('Failed to fetch users')
  return res.json()
}

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  })
}

// Usage
function UserList() {
  const { data: users, isLoading, error } = useUsers()

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <ul>
      {users?.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

**Mutation with Optimistic Update:**
```tsx
// hooks/useTodos.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'

interface Todo {
  id: string
  title: string
  completed: boolean
}

export function useToggleTodo() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (todo: Todo) => {
      const res = await fetch(`/api/todos/${todo.id}`, {
        method: 'PATCH',
        body: JSON.stringify({ completed: !todo.completed }),
      })
      return res.json()
    },
    onMutate: async (todo) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] })

      // Snapshot previous value
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos'])

      // Optimistically update
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((t) =>
          t.id === todo.id ? { ...t, completed: !t.completed } : t
        )
      )

      return { previousTodos }
    },
    onError: (err, todo, context) => {
      // Rollback on error
      queryClient.setQueryData(['todos'], context?.previousTodos)
    },
    onSettled: () => {
      // Refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })
}
```

**Infinite Query:**
```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

export function useInfinitePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: async ({ pageParam = 1 }) => {
      const res = await fetch(`/api/posts?page=${pageParam}`)
      return res.json()
    },
    getNextPageParam: (lastPage) => lastPage.nextPage,
    initialPageParam: 1,
  })
}
```

**Exercises:**
```tsx
// exercises/react-query/

// 1. Fetch and display a list with loading/error states
// 2. Implement search with debounced queries
// 3. Add mutations with optimistic updates
// 4. Build infinite scroll pagination
// 5. Prefetch data on hover
```

### Day 4: Form State
**Topics:**
- React Hook Form
- Zod integration
- Server Actions with forms
- Form validation patterns

**Setup:**
```bash
npm install react-hook-form @hookform/resolvers zod
```

**Form Example:**
```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
})

type FormData = z.infer<typeof schema>

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  })

  const onSubmit = async (data: FormData) => {
    await login(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      <div>
        <input {...register('password')} type="password" placeholder="Password" />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      <button disabled={isSubmitting}>
        {isSubmitting ? 'Loading...' : 'Login'}
      </button>
    </form>
  )
}
```

### Day 5: Putting It All Together

**Decision Tree:**
```
Is it UI state (theme, sidebar, modal)?
  → Zustand or Context

Is it from an API?
  → React Query

Is it in the URL (filters, pagination)?
  → searchParams / nuqs

Is it form data?
  → React Hook Form

Is it single component?
  → useState

Is it few components?
  → Lift state + props
```

**Exercise: Build a Complete App**
- Products list (React Query)
- Shopping cart (Zustand with persistence)
- Filters in URL (searchParams)
- Checkout form (React Hook Form)

---

## Resources

### Documentation
- [Zustand](https://zustand-demo.pmnd.rs/)
- [TanStack Query](https://tanstack.com/query)
- [React Hook Form](https://react-hook-form.com/)

---

## Checklist Before Moving On

- [ ] Understand when to use each type of state
- [ ] Can implement Zustand stores
- [ ] Can use React Query for data fetching
- [ ] Know how to handle optimistic updates
- [ ] Can build forms with React Hook Form
- [ ] Completed the shopping app exercise

---

**Next:** [Testing](../05-testing/README.md)
