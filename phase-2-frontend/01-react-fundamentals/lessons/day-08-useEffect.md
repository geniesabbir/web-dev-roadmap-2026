# Day 8: useEffect Hook - Side Effects in React

## Introduction

The `useEffect` hook is React's way of handling side effects - operations that interact with the outside world or affect something outside the scope of the current function. This includes data fetching, subscriptions, timers, and DOM manipulation.

## Learning Objectives

By the end of this lesson, you will:
- Understand what side effects are and why they need special handling
- Master the useEffect hook syntax and dependency array
- Learn cleanup functions and when to use them
- Handle data fetching properly
- Avoid common useEffect pitfalls

---

## What Are Side Effects?

Side effects are operations that:
- **Fetch data** from APIs
- **Subscribe** to external data sources
- **Manually modify** the DOM
- **Set up timers** (setTimeout, setInterval)
- **Log** to console or analytics
- **Store data** in localStorage

```jsx
// These are ALL side effects:
fetch('/api/users')           // Network request
document.title = 'New Title'  // DOM manipulation
localStorage.setItem(...)     // Browser storage
setInterval(...)              // Timer
console.log(...)              // Logging
```

**Why special handling?** React components should be pure during rendering. Side effects need to happen AFTER rendering completes.

---

## Basic useEffect Syntax

```jsx
import { useEffect } from 'react';

function Component() {
  useEffect(() => {
    // Side effect code runs AFTER render
    console.log('Component rendered!');
  });

  return <div>Hello</div>;
}
```

### The Three Parts of useEffect

```jsx
useEffect(
  () => {
    // 1. EFFECT: Code that runs after render
    console.log('Effect ran');

    // 3. CLEANUP (optional): Returned function
    return () => {
      console.log('Cleanup ran');
    };
  },
  [dependencies] // 2. DEPENDENCY ARRAY: When to re-run
);
```

---

## Dependency Array Deep Dive

The dependency array controls WHEN the effect runs:

### 1. No Dependency Array - Runs After EVERY Render

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Runs after EVERY render (initial + all updates)
  useEffect(() => {
    console.log('Effect ran - count or name changed');
  });
  // ⚠️ Usually NOT what you want - can cause performance issues

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <input value={name} onChange={e => setName(e.target.value)} />
    </div>
  );
}
```

### 2. Empty Dependency Array - Runs ONCE on Mount

```jsx
function UserProfile() {
  const [user, setUser] = useState(null);

  // Runs ONLY on mount (component first appears)
  useEffect(() => {
    console.log('Component mounted - fetching user');
    fetchUser().then(setUser);
  }, []); // Empty array = run once

  return <div>{user?.name}</div>;
}
```

### 3. With Dependencies - Runs When Dependencies Change

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  // Runs on mount AND whenever userId changes
  useEffect(() => {
    console.log(`Fetching user ${userId}`);
    fetchUser(userId).then(setUser);
  }, [userId]); // Re-run when userId changes

  return <div>{user?.name}</div>;
}
```

### Dependency Array Rules

```jsx
function Example({ propValue }) {
  const [stateValue, setStateValue] = useState(0);

  useEffect(() => {
    // Any value used inside the effect...
    console.log(propValue, stateValue);

    // ...MUST be in the dependency array
  }, [propValue, stateValue]); // ✅ Both included

  // ❌ WRONG - missing dependency
  useEffect(() => {
    console.log(propValue);
  }, []); // propValue is used but not listed!
}
```

---

## Document Title Example

A classic useEffect example - updating the browser tab title:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  // Update document title whenever count changes
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### With Cleanup

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const previousTitle = document.title;
    document.title = `Count: ${count}`;

    // Cleanup: Restore original title when component unmounts
    return () => {
      document.title = previousTitle;
    };
  }, [count]);

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

## Cleanup Functions

Cleanup functions prevent memory leaks and stale behavior:

### Timer Cleanup

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // Set up interval
    const intervalId = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup: Clear interval when component unmounts
    return () => {
      clearInterval(intervalId);
    };
  }, []); // Empty array - set up once

  return <p>Seconds: {seconds}</p>;
}
```

### Event Listener Cleanup

```jsx
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // Add event listener
    window.addEventListener('resize', handleResize);

    // Cleanup: Remove event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <p>{size.width} x {size.height}</p>;
}
```

### Subscription Cleanup

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Subscribe to chat room
    const connection = chatApi.connect(roomId);

    connection.on('message', (msg) => {
      setMessages(prev => [...prev, msg]);
    });

    // Cleanup: Disconnect when roomId changes or unmount
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Re-connect if roomId changes

  return (
    <ul>
      {messages.map((msg, i) => (
        <li key={i}>{msg}</li>
      ))}
    </ul>
  );
}
```

---

## Data Fetching with useEffect

### Basic Pattern

```jsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchUsers() {
      try {
        setLoading(true);
        const response = await fetch('/api/users');

        if (!response.ok) {
          throw new Error('Failed to fetch');
        }

        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Fetching with Parameters

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isCancelled = false;

    async function fetchUser() {
      setLoading(true);

      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();

        // Only update state if not cancelled
        if (!isCancelled) {
          setUser(data);
          setLoading(false);
        }
      } catch (err) {
        if (!isCancelled) {
          console.error(err);
          setLoading(false);
        }
      }
    }

    fetchUser();

    // Cleanup: Mark as cancelled if userId changes before fetch completes
    return () => {
      isCancelled = true;
    };
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (!user) return <p>User not found</p>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Using AbortController (Modern Approach)

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }

    const abortController = new AbortController();

    async function search() {
      setLoading(true);

      try {
        const response = await fetch(
          `/api/search?q=${encodeURIComponent(query)}`,
          { signal: abortController.signal }
        );
        const data = await response.json();
        setResults(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          console.error('Search failed:', err);
        }
      } finally {
        setLoading(false);
      }
    }

    search();

    // Cleanup: Abort fetch if query changes
    return () => {
      abortController.abort();
    };
  }, [query]);

  return (
    <div>
      {loading && <p>Searching...</p>}
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Multiple useEffect Hooks

You can (and should) use multiple useEffect hooks to separate concerns:

```jsx
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);

  // Effect 1: Fetch user data
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Effect 2: Fetch user's posts
  useEffect(() => {
    fetchPosts(userId).then(setPosts);
  }, [userId]);

  // Effect 3: Subscribe to notifications
  useEffect(() => {
    const unsubscribe = subscribeToNotifications(userId, (notif) => {
      setNotifications(prev => [notif, ...prev]);
    });

    return () => unsubscribe();
  }, [userId]);

  // Effect 4: Update document title (independent)
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Dashboard`;
    }
  }, [user]);

  return (
    <div>
      <h1>{user?.name}</h1>
      <PostList posts={posts} />
      <NotificationBell count={notifications.length} />
    </div>
  );
}
```

---

## Common Patterns

### Debounced Search

```jsx
function SearchInput() {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  // Debounce the query
  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedQuery(query);
    }, 300); // Wait 300ms after user stops typing

    return () => clearTimeout(timeoutId);
  }, [query]);

  // Fetch when debounced query changes
  useEffect(() => {
    if (debouncedQuery) {
      console.log('Searching for:', debouncedQuery);
      // fetch(`/api/search?q=${debouncedQuery}`)
    }
  }, [debouncedQuery]);

  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### LocalStorage Sync

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  // Sync to localStorage whenever value changes
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  return (
    <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

### Online Status Detection

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

function App() {
  const isOnline = useOnlineStatus();

  return (
    <div>
      {!isOnline && (
        <div className="offline-banner">
          You are offline. Some features may be unavailable.
        </div>
      )}
      {/* Rest of app */}
    </div>
  );
}
```

---

## Common Mistakes and Fixes

### 1. Missing Dependencies

```jsx
// ❌ BAD - count not in dependencies
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // Always uses initial count (0)
    }, 1000);
    return () => clearInterval(id);
  }, []); // Missing count dependency!
}

// ✅ GOOD - Use functional update
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // Uses previous value
    }, 1000);
    return () => clearInterval(id);
  }, []); // No dependency needed!
}
```

### 2. Object/Array Dependencies

```jsx
// ❌ BAD - Object created every render
function UserList({ filters }) {
  useEffect(() => {
    fetchUsers(filters);
  }, [filters]); // Runs every render because {} !== {}
}

// ✅ GOOD - Destructure or use specific values
function UserList({ filters }) {
  const { status, role } = filters;

  useEffect(() => {
    fetchUsers({ status, role });
  }, [status, role]); // Only runs when actual values change
}
```

### 3. Async Functions Directly in useEffect

```jsx
// ❌ BAD - useEffect can't be async
useEffect(async () => {
  const data = await fetch('/api/data');
  setData(data);
}, []);

// ✅ GOOD - Define async function inside
useEffect(() => {
  async function fetchData() {
    const data = await fetch('/api/data');
    setData(data);
  }

  fetchData();
}, []);
```

### 4. Race Conditions

```jsx
// ❌ BAD - Race condition possible
useEffect(() => {
  fetch(`/api/user/${userId}`)
    .then(res => res.json())
    .then(setUser); // Might set stale data
}, [userId]);

// ✅ GOOD - Use cleanup flag
useEffect(() => {
  let cancelled = false;

  fetch(`/api/user/${userId}`)
    .then(res => res.json())
    .then(data => {
      if (!cancelled) setUser(data);
    });

  return () => { cancelled = true; };
}, [userId]);
```

---

## useEffect vs Event Handlers

```jsx
function Form() {
  const [value, setValue] = useState('');

  // ❌ WRONG - Don't use effect for user actions
  useEffect(() => {
    if (value) {
      submitForm(value);
    }
  }, [value]);

  // ✅ RIGHT - Use event handler for user actions
  const handleSubmit = () => {
    submitForm(value);
  };

  return (
    <form onSubmit={e => { e.preventDefault(); handleSubmit(); }}>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Rule of thumb:**
- **useEffect**: Synchronize with external systems
- **Event handlers**: Respond to user actions

---

## Exercises

### Exercise 1: Auto-Save Draft

Create a component that auto-saves a draft to localStorage:

```jsx
function AutoSaveDraft() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [lastSaved, setLastSaved] = useState(null);

  // TODO: Load draft from localStorage on mount

  // TODO: Auto-save to localStorage after 1 second of no typing

  return (
    <div>
      <input
        placeholder="Title"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <textarea
        placeholder="Content"
        value={content}
        onChange={e => setContent(e.target.value)}
      />
      {lastSaved && <p>Last saved: {lastSaved}</p>}
    </div>
  );
}
```

### Exercise 2: Clock with Timezone

```jsx
function WorldClock({ timezone }) {
  // TODO: Display current time that updates every second
  // TODO: Format time according to timezone prop
  // TODO: Clean up interval on unmount or timezone change

  return <div>{/* Display time */}</div>;
}
```

### Exercise 3: Infinite Scroll

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);

  // TODO: Fetch initial items

  // TODO: Set up scroll listener to detect when near bottom

  // TODO: Load more items when scrolling near bottom

  return (
    <div className="scroll-container">
      {items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      {loading && <p>Loading more...</p>}
    </div>
  );
}
```

---

## Solutions

### Solution 1: Auto-Save Draft

```jsx
function AutoSaveDraft() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [lastSaved, setLastSaved] = useState(null);

  // Load draft from localStorage on mount
  useEffect(() => {
    const savedDraft = localStorage.getItem('draft');
    if (savedDraft) {
      const { title, content, savedAt } = JSON.parse(savedDraft);
      setTitle(title);
      setContent(content);
      setLastSaved(savedAt);
    }
  }, []);

  // Auto-save after 1 second of no typing
  useEffect(() => {
    if (!title && !content) return;

    const timeoutId = setTimeout(() => {
      const savedAt = new Date().toLocaleTimeString();
      localStorage.setItem('draft', JSON.stringify({
        title,
        content,
        savedAt
      }));
      setLastSaved(savedAt);
    }, 1000);

    return () => clearTimeout(timeoutId);
  }, [title, content]);

  const clearDraft = () => {
    localStorage.removeItem('draft');
    setTitle('');
    setContent('');
    setLastSaved(null);
  };

  return (
    <div style={{ maxWidth: '600px', margin: '0 auto' }}>
      <input
        placeholder="Title"
        value={title}
        onChange={e => setTitle(e.target.value)}
        style={{ width: '100%', padding: '8px', marginBottom: '8px' }}
      />
      <textarea
        placeholder="Content"
        value={content}
        onChange={e => setContent(e.target.value)}
        rows={10}
        style={{ width: '100%', padding: '8px' }}
      />
      <div style={{ display: 'flex', justifyContent: 'space-between' }}>
        {lastSaved && <p>Last saved: {lastSaved}</p>}
        <button onClick={clearDraft}>Clear Draft</button>
      </div>
    </div>
  );
}
```

### Solution 2: Clock with Timezone

```jsx
function WorldClock({ timezone = 'UTC' }) {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // Timer doesn't depend on timezone

  const formattedTime = time.toLocaleTimeString('en-US', {
    timeZone: timezone,
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: true
  });

  const formattedDate = time.toLocaleDateString('en-US', {
    timeZone: timezone,
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });

  return (
    <div style={{ textAlign: 'center', padding: '20px' }}>
      <h2>{timezone}</h2>
      <p style={{ fontSize: '48px', fontFamily: 'monospace' }}>
        {formattedTime}
      </p>
      <p>{formattedDate}</p>
    </div>
  );
}

// Usage
function App() {
  return (
    <div style={{ display: 'flex', gap: '20px' }}>
      <WorldClock timezone="America/New_York" />
      <WorldClock timezone="Europe/London" />
      <WorldClock timezone="Asia/Tokyo" />
    </div>
  );
}
```

### Solution 3: Infinite Scroll

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  // Simulated fetch function
  const fetchItems = async (pageNum) => {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 500));

    // Simulate 100 total items, 10 per page
    const start = (pageNum - 1) * 10;
    const newItems = Array.from({ length: 10 }, (_, i) => ({
      id: start + i + 1,
      name: `Item ${start + i + 1}`
    }));

    return {
      items: newItems,
      hasMore: pageNum < 10 // 10 pages total
    };
  };

  // Fetch initial items
  useEffect(() => {
    async function loadInitial() {
      setLoading(true);
      const data = await fetchItems(1);
      setItems(data.items);
      setHasMore(data.hasMore);
      setLoading(false);
    }

    loadInitial();
  }, []);

  // Load more items when page changes (but not on initial load)
  useEffect(() => {
    if (page === 1) return;

    async function loadMore() {
      setLoading(true);
      const data = await fetchItems(page);
      setItems(prev => [...prev, ...data.items]);
      setHasMore(data.hasMore);
      setLoading(false);
    }

    loadMore();
  }, [page]);

  // Set up scroll listener
  useEffect(() => {
    const handleScroll = () => {
      if (loading || !hasMore) return;

      const scrollTop = document.documentElement.scrollTop;
      const scrollHeight = document.documentElement.scrollHeight;
      const clientHeight = document.documentElement.clientHeight;

      // Load more when 100px from bottom
      if (scrollTop + clientHeight >= scrollHeight - 100) {
        setPage(p => p + 1);
      }
    };

    window.addEventListener('scroll', handleScroll);

    return () => window.removeEventListener('scroll', handleScroll);
  }, [loading, hasMore]);

  return (
    <div style={{ maxWidth: '600px', margin: '0 auto', padding: '20px' }}>
      <h1>Infinite Scroll Demo</h1>
      {items.map(item => (
        <div
          key={item.id}
          style={{
            padding: '20px',
            margin: '10px 0',
            background: '#f0f0f0',
            borderRadius: '8px'
          }}
        >
          {item.name}
        </div>
      ))}
      {loading && <p style={{ textAlign: 'center' }}>Loading more...</p>}
      {!hasMore && (
        <p style={{ textAlign: 'center', color: '#666' }}>
          No more items to load
        </p>
      )}
    </div>
  );
}
```

---

## When NOT to Use useEffect

React recommends avoiding useEffect when possible:

```jsx
// ❌ DON'T: Transform data in effect
useEffect(() => {
  setFilteredList(items.filter(i => i.active));
}, [items]);

// ✅ DO: Calculate during render
const filteredList = items.filter(i => i.active);

// ❌ DON'T: Reset state in effect
useEffect(() => {
  setSelection(null);
}, [items]);

// ✅ DO: Use key to reset component
<ItemList items={items} key={categoryId} />

// ❌ DON'T: Submit form in effect
useEffect(() => {
  if (submitted) {
    sendToServer(formData);
  }
}, [submitted, formData]);

// ✅ DO: Use event handler
const handleSubmit = () => {
  sendToServer(formData);
};
```

---

## Key Takeaways

1. **useEffect is for side effects** - interactions with the outside world
2. **Dependency array controls when effects run** - empty = once, with deps = when deps change
3. **Always clean up** subscriptions, timers, and event listeners
4. **Handle race conditions** when fetching data with changing dependencies
5. **Separate concerns** with multiple useEffect hooks
6. **Don't overuse effects** - many things can be done during render or in event handlers
7. **Use functional updates** in setters to avoid stale closures

---

## What's Next?

Tomorrow we'll learn about **Custom Hooks** - how to extract and reuse stateful logic across components. We've already seen a preview with `useLocalStorage` and `useOnlineStatus`!
