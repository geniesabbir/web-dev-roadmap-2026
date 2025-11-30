# Day 3: React Query (TanStack Query) - Server State Management

## Introduction

React Query (now TanStack Query) is the gold standard for managing server state in React applications. It handles caching, background updates, stale data, pagination, and more - solving problems that are difficult to handle with traditional state management.

## Learning Objectives

By the end of this lesson, you will:
- Set up React Query in your application
- Fetch and cache data effectively
- Handle mutations with optimistic updates
- Implement pagination and infinite scroll
- Master cache invalidation strategies

---

## Installation & Setup

```bash
npm install @tanstack/react-query
npm install @tanstack/react-query-devtools
```

### Provider Setup

```tsx
// app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

// app/layout.tsx
import { QueryProvider } from './providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <QueryProvider>{children}</QueryProvider>
      </body>
    </html>
  );
}
```

---

## Basic Queries

### Simple Query

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchUsers() {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('Failed to fetch users');
  return response.json();
}

function UserList() {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      <ul>
        {data.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Query with Parameters

```tsx
async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}

function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId, // Only fetch when userId exists
  });

  if (isLoading) return <UserSkeleton />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

---

## Query Keys

Query keys uniquely identify cached data:

```tsx
// Simple key
useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

// Key with parameters
useQuery({ queryKey: ['todo', todoId], queryFn: () => fetchTodo(todoId) });

// Key with multiple parameters
useQuery({
  queryKey: ['todos', { status, page }],
  queryFn: () => fetchTodos(status, page),
});

// Key with nested objects
useQuery({
  queryKey: ['users', userId, 'posts', { sort, limit }],
  queryFn: () => fetchUserPosts(userId, { sort, limit }),
});
```

---

## Query Configuration

```tsx
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,

  // Timing
  staleTime: 5 * 60 * 1000,      // Data fresh for 5 minutes
  gcTime: 30 * 60 * 1000,        // Cache garbage collected after 30 min
  refetchInterval: 30 * 1000,     // Refetch every 30 seconds

  // Behavior
  enabled: isReady,               // Conditional fetching
  refetchOnWindowFocus: true,     // Refetch when window gains focus
  refetchOnMount: true,           // Refetch when component mounts
  refetchOnReconnect: true,       // Refetch on network reconnect
  retry: 3,                       // Retry failed requests 3 times
  retryDelay: (attemptIndex) =>   // Exponential backoff
    Math.min(1000 * 2 ** attemptIndex, 30000),

  // Callbacks
  onSuccess: (data) => console.log('Fetched:', data),
  onError: (error) => console.error('Error:', error),
  onSettled: () => console.log('Query settled'),

  // Data transformation
  select: (data) => data.filter((item) => item.active),

  // Placeholder data while loading
  placeholderData: [],
  // Or use previous data
  placeholderData: (previousData) => previousData,
});
```

---

## Mutations

### Basic Mutation

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

async function createUser(userData: CreateUserData) {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  });
  if (!response.ok) throw new Error('Failed to create user');
  return response.json();
}

function CreateUserForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      // Invalidate and refetch users list
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    mutation.mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create User'}
      </button>
      {mutation.isError && <p className="error">{mutation.error.message}</p>}
      {mutation.isSuccess && <p className="success">User created!</p>}
    </form>
  );
}
```

### Optimistic Updates

```tsx
function TodoItem({ todo }: { todo: Todo }) {
  const queryClient = useQueryClient();

  const toggleMutation = useMutation({
    mutationFn: (completed: boolean) =>
      fetch(`/api/todos/${todo.id}`, {
        method: 'PATCH',
        body: JSON.stringify({ completed }),
      }),

    // Optimistic update
    onMutate: async (newCompleted) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot previous value
      const previousTodos = queryClient.getQueryData(['todos']);

      // Optimistically update
      queryClient.setQueryData(['todos'], (old: Todo[]) =>
        old.map((t) =>
          t.id === todo.id ? { ...t, completed: newCompleted } : t
        )
      );

      // Return context for rollback
      return { previousTodos };
    },

    // Rollback on error
    onError: (err, newCompleted, context) => {
      queryClient.setQueryData(['todos'], context?.previousTodos);
    },

    // Refetch after success or error
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleMutation.mutate(!todo.completed)}
        disabled={toggleMutation.isPending}
      />
      <span>{todo.title}</span>
    </li>
  );
}
```

---

## Pagination

### Offset-Based Pagination

```tsx
function PaginatedUsers() {
  const [page, setPage] = useState(1);

  const { data, isLoading, isFetching, isPlaceholderData } = useQuery({
    queryKey: ['users', page],
    queryFn: () => fetchUsers(page),
    placeholderData: (previousData) => previousData, // Keep previous data while fetching
  });

  return (
    <div>
      {isLoading ? (
        <div>Loading...</div>
      ) : (
        <>
          <ul>
            {data.users.map((user) => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>

          <div className="pagination">
            <button
              onClick={() => setPage((p) => Math.max(p - 1, 1))}
              disabled={page === 1}
            >
              Previous
            </button>

            <span>Page {page}</span>

            <button
              onClick={() => setPage((p) => p + 1)}
              disabled={isPlaceholderData || !data.hasMore}
            >
              Next
            </button>
          </div>

          {isFetching && <span>Updating...</span>}
        </>
      )}
    </div>
  );
}
```

### Infinite Scroll

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';

async function fetchPosts({ pageParam = 1 }) {
  const response = await fetch(`/api/posts?page=${pageParam}&limit=10`);
  return response.json();
}

function InfinitePostList() {
  const { ref, inView } = useInView();

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    error,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    initialPageParam: 1,
    getNextPageParam: (lastPage, pages) =>
      lastPage.hasMore ? pages.length + 1 : undefined,
  });

  // Auto-fetch when bottom is visible
  useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, fetchNextPage]);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {data.pages.map((page, i) => (
        <React.Fragment key={i}>
          {page.posts.map((post) => (
            <PostCard key={post.id} post={post} />
          ))}
        </React.Fragment>
      ))}

      <div ref={ref}>
        {isFetchingNextPage ? (
          <div>Loading more...</div>
        ) : hasNextPage ? (
          <div>Scroll for more</div>
        ) : (
          <div>No more posts</div>
        )}
      </div>
    </div>
  );
}
```

---

## Cache Invalidation

```tsx
const queryClient = useQueryClient();

// Invalidate all queries
queryClient.invalidateQueries();

// Invalidate specific query
queryClient.invalidateQueries({ queryKey: ['users'] });

// Invalidate queries starting with key
queryClient.invalidateQueries({ queryKey: ['users'] }); // Matches ['users'], ['users', 1], etc.

// Invalidate exact query
queryClient.invalidateQueries({
  queryKey: ['users', userId],
  exact: true,
});

// Remove from cache entirely
queryClient.removeQueries({ queryKey: ['users', userId] });

// Set data directly
queryClient.setQueryData(['user', userId], newUserData);

// Get cached data
const cachedUser = queryClient.getQueryData(['user', userId]);
```

---

## Prefetching

```tsx
const queryClient = useQueryClient();

// Prefetch for hover/focus
function UserLink({ userId }: { userId: string }) {
  const prefetchUser = () => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 5 * 60 * 1000,
    });
  };

  return (
    <Link
      href={`/users/${userId}`}
      onMouseEnter={prefetchUser}
      onFocus={prefetchUser}
    >
      View User
    </Link>
  );
}

// Prefetch on page load (Next.js)
export async function getServerSideProps() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
}
```

---

## Custom Hooks

### Reusable Query Hook

```tsx
// hooks/useUsers.ts
export function useUsers(options = {}) {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000,
    ...options,
  });
}

export function useUser(userId: string, options = {}) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId,
    ...options,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<User> }) =>
      updateUser(id, data),
    onSuccess: (data, variables) => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

### Usage

```tsx
function UserManagement() {
  const { data: users, isLoading } = useUsers();
  const createUser = useCreateUser();
  const updateUser = useUpdateUser();
  const deleteUser = useDeleteUser();

  // Component logic...
}
```

---

## Parallel & Dependent Queries

### Parallel Queries

```tsx
function Dashboard() {
  // These run in parallel
  const usersQuery = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
  const postsQuery = useQuery({ queryKey: ['posts'], queryFn: fetchPosts });
  const statsQuery = useQuery({ queryKey: ['stats'], queryFn: fetchStats });

  // Or use useQueries for dynamic parallel queries
  const results = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),
    })),
  });
}
```

### Dependent Queries

```tsx
function UserPosts({ userId }: { userId: string }) {
  // First query
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  // Dependent query - only runs when user is available
  const { data: posts } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetchUserPosts(user!.id),
    enabled: !!user?.id, // Only fetch when user exists
  });

  return (
    <div>
      <h1>{user?.name}'s Posts</h1>
      {posts?.map((post) => <PostCard key={post.id} post={post} />)}
    </div>
  );
}
```

---

## Exercise: Build a Data Dashboard

Create a dashboard with:
1. User list with pagination
2. User detail view with posts
3. Create/Edit user forms with mutations
4. Optimistic updates for favorites

---

## Solution Summary

```tsx
// hooks/api.ts
export function useUsers(page: number) {
  return useQuery({
    queryKey: ['users', { page }],
    queryFn: () => fetchUsers(page),
    placeholderData: (prev) => prev,
  });
}

export function useUserWithPosts(userId: string) {
  const userQuery = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId,
  });

  const postsQuery = useQuery({
    queryKey: ['user', userId, 'posts'],
    queryFn: () => fetchUserPosts(userId),
    enabled: !!userId,
  });

  return {
    user: userQuery.data,
    posts: postsQuery.data,
    isLoading: userQuery.isLoading || postsQuery.isLoading,
  };
}

export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ userId, favorite }: { userId: string; favorite: boolean }) =>
      updateUserFavorite(userId, favorite),

    onMutate: async ({ userId, favorite }) => {
      await queryClient.cancelQueries({ queryKey: ['users'] });
      const previous = queryClient.getQueryData(['users']);

      queryClient.setQueryData(['users'], (old: any) => ({
        ...old,
        users: old.users.map((u: User) =>
          u.id === userId ? { ...u, favorite } : u
        ),
      }));

      return { previous };
    },

    onError: (err, variables, context) => {
      queryClient.setQueryData(['users'], context?.previous);
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

---

## Key Takeaways

1. **React Query handles server state** - caching, syncing, background updates
2. **Query keys** uniquely identify cached data
3. **Mutations** for create/update/delete operations
4. **Optimistic updates** for instant UI feedback
5. **Invalidation** keeps data fresh
6. **Custom hooks** encapsulate reusable logic
7. **DevTools** for debugging cache state

---

## What's Next?

Tomorrow we'll build a **Complete Project** combining Zustand for client state and React Query for server state!
