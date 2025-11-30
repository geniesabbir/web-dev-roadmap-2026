# React Fundamentals

**Duration:** 3-4 weeks

## Learning Objectives

By the end of this section, you will:
- Understand React's component-based architecture
- Build interactive UIs with components
- Manage state with useState
- Handle side effects with useEffect
- Work with forms and user input
- Think in React

---

## Week 1: Core Concepts

### Day 1: Introduction to React
**Topics:**
- What is React and why use it?
- Virtual DOM concept
- Setting up with Vite
- Project structure
- JSX basics

**Setup:**
```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

**Exercises:**
```tsx
// exercises/01-intro/
// 1. Create a React project with Vite
// 2. Explore the file structure
// 3. Write JSX with different element types
// 4. Understand the difference between JSX and HTML
```

### Day 2: Components
**Topics:**
- Functional components
- Component naming conventions
- Importing and exporting components
- Component composition
- Organizing components

**Exercises:**
```tsx
// exercises/02-components/

// 1. Create a Header component
// 2. Create a Footer component
// 3. Create a Card component
// 4. Compose them in App.tsx
```

### Day 3: Props
**Topics:**
- Passing props to components
- Props with TypeScript
- Destructuring props
- Default props
- Children prop
- Prop drilling (and its problems)

**Exercises:**
```tsx
// exercises/03-props/

// 1. Create a UserCard that accepts name, email, avatar
// 2. Create a Button with variant and size props
// 3. Create a Layout with children
// 4. Type all props with TypeScript
```

### Day 4: Conditional Rendering
**Topics:**
- `if` statements
- Ternary operator
- Logical && operator
- Multiple conditions
- Rendering nothing (null)

**Exercises:**
```tsx
// exercises/04-conditional/

// 1. Show/hide a message based on state
// 2. Render different components based on user role
// 3. Show loading, error, or data states
// 4. Toggle component visibility
```

### Day 5: Lists and Keys
**Topics:**
- Rendering arrays with `map()`
- The importance of keys
- Choosing good keys
- Rendering nested lists
- Filtering and sorting before rendering

**Exercises:**
```tsx
// exercises/05-lists/

// 1. Render a list of products
// 2. Filter products by category
// 3. Sort products by price
// 4. Render a nested comment thread
```

---

## Week 2: State and Events

### Day 1: useState Basics
**Topics:**
- What is state?
- `useState` hook
- State with TypeScript
- Updating state
- State is immutable
- Multiple state variables

**Exercises:**
```tsx
// exercises/06-usestate-basics/

// 1. Create a counter with increment/decrement
// 2. Build a toggle switch
// 3. Create a text input with state
// 4. Build a color picker
```

### Day 2: useState with Objects and Arrays
**Topics:**
- Updating objects in state
- Updating arrays in state
- Spread operator for updates
- Nested state updates
- When to use multiple useState vs one object

**Exercises:**
```tsx
// exercises/07-usestate-complex/

// 1. Build a form with object state
// 2. Create a todo list (add, toggle, delete)
// 3. Build a shopping cart
// 4. Manage a list of editable items
```

### Day 3: Event Handling
**Topics:**
- Event handlers in React
- Event object
- Passing arguments to handlers
- Preventing default behavior
- Event types in TypeScript

**Exercises:**
```tsx
// exercises/08-events/

// 1. Handle click events
// 2. Handle form submission
// 3. Handle keyboard events
// 4. Build a drag and drop interface (basic)
```

### Day 4: Forms
**Topics:**
- Controlled components
- Uncontrolled components
- Form validation
- Multiple inputs
- Select and textarea
- Form libraries introduction

**Exercises:**
```tsx
// exercises/09-forms/

// 1. Build a login form
// 2. Build a registration form with validation
// 3. Create a multi-step form
// 4. Handle file uploads
```

### Day 5: Lifting State Up
**Topics:**
- When to lift state
- Sharing state between components
- Single source of truth
- Inverse data flow (child to parent)

**Exercises:**
```tsx
// exercises/10-lifting-state/

// 1. Temperature converter (Celsius/Fahrenheit)
// 2. Filter component that controls a list
// 3. Accordion where only one item is open
// 4. Tab component with shared state
```

---

## Week 3: Effects and Refs

### Day 1: useEffect Basics
**Topics:**
- What are side effects?
- `useEffect` syntax
- Dependency array
- Cleanup function
- Common use cases

**Exercises:**
```tsx
// exercises/11-useeffect-basics/

// 1. Update document title
// 2. Log state changes
// 3. Set up and clean up event listeners
// 4. Sync with localStorage
```

### Day 2: useEffect Data Fetching
**Topics:**
- Fetching data in useEffect
- Loading states
- Error handling
- Race conditions
- Abort controllers
- Why you might not need useEffect for data

**Exercises:**
```tsx
// exercises/12-useeffect-fetch/

// 1. Fetch and display a list of users
// 2. Fetch with loading and error states
// 3. Fetch with search/filter parameters
// 4. Handle race conditions properly
```

### Day 3: useRef
**Topics:**
- What are refs?
- Accessing DOM elements
- Storing mutable values
- Refs vs state
- `forwardRef`

**Exercises:**
```tsx
// exercises/13-useref/

// 1. Focus an input on mount
// 2. Scroll to an element
// 3. Store previous state value
// 4. Build a stopwatch with refs
```

### Day 4: Component Lifecycle
**Topics:**
- Mounting, updating, unmounting
- useEffect for lifecycle
- Cleanup importance
- Strict mode and double effects
- Common patterns

**Exercises:**
```tsx
// exercises/14-lifecycle/

// 1. Track component mount/unmount
// 2. Set up subscriptions with cleanup
// 3. Handle window resize events
// 4. Implement an intersection observer
```

### Day 5: Mini Project Day
Build a complete mini application:

**Project: Movie Search App**
- Search input with debouncing
- Fetch movies from OMDB API
- Display results in a grid
- Show loading and error states
- Click to see movie details
- Save favorites to localStorage

---

## Week 4: Styling and Polish

### Day 1: Styling in React
**Topics:**
- CSS Modules
- Inline styles
- CSS-in-JS overview
- Tailwind CSS setup
- Conditional classes

**Exercises:**
```tsx
// exercises/15-styling/

// 1. Style components with CSS Modules
// 2. Set up and use Tailwind CSS
// 3. Create a theme with CSS variables
// 4. Build a styled component library
```

### Day 2: Tailwind CSS Deep Dive
**Topics:**
- Utility classes
- Responsive design
- Dark mode
- Custom configuration
- Component extraction

**Exercises:**
```tsx
// exercises/16-tailwind/

// 1. Build a responsive navbar
// 2. Create a card component
// 3. Implement dark mode toggle
// 4. Build a complete landing page
```

### Day 3-4: Project Work
Build the **Task Management App**:
- Create, read, update, delete tasks
- Mark tasks complete
- Filter by status (all, active, completed)
- Sort by date or priority
- Persist to localStorage
- Full TypeScript
- Styled with Tailwind

### Day 5: Code Review & Refactoring
- Review your code
- Extract reusable components
- Add TypeScript improvements
- Write cleaner code
- Document your components

---

## Common Patterns

### Component Template
```tsx
import { useState } from 'react';

interface Props {
  title: string;
  onAction?: () => void;
}

export function MyComponent({ title, onAction }: Props) {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(prev => prev + 1);
    onAction?.();
  };

  return (
    <div>
      <h1>{title}</h1>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

### Data Fetching Pattern
```tsx
import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
}

export function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUsers() {
      try {
        const res = await fetch('/api/users', { signal: controller.signal });
        if (!res.ok) throw new Error('Failed to fetch');
        const data = await res.json();
        setUsers(data);
      } catch (e) {
        if (e instanceof Error && e.name !== 'AbortError') {
          setError(e.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();

    return () => controller.abort();
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## Resources

### Documentation
- [React Documentation](https://react.dev/)
- [TypeScript + React Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

### Practice
- [React Tutorial](https://react.dev/learn)
- [React Challenges](https://reactchallenges.com/)

### Tools
- [React DevTools](https://react.dev/learn/react-developer-tools)
- [Vite](https://vitejs.dev/)

---

## Checklist Before Moving On

- [ ] Can create components with TypeScript
- [ ] Understand props and prop types
- [ ] Can manage state with useState
- [ ] Handle arrays and objects in state correctly
- [ ] Can handle events
- [ ] Understand useEffect and cleanup
- [ ] Can fetch data and handle loading/error states
- [ ] Know when to use useRef
- [ ] Completed Movie Search App
- [ ] Completed Task Management App

---

**Next:** [React Advanced Patterns](../02-react-advanced/README.md)
