# Day 2: useReducer Hook - Managing Complex State

## Introduction

`useReducer` is React's alternative to `useState` for managing complex state logic. It's inspired by Redux and follows the same pattern: dispatching actions to a reducer function that determines how state should change. When state has multiple sub-values or complex update logic, useReducer is often cleaner than useState.

## Learning Objectives

By the end of this lesson, you will:
- Understand when to use useReducer vs useState
- Master the reducer pattern
- Handle complex state updates cleanly
- Combine useReducer with Context
- Build scalable state management solutions

---

## useState vs useReducer

### When to Use useState

```jsx
// ✅ useState is great for simple state
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isOpen, setIsOpen] = useState(false);
```

### When to Use useReducer

```jsx
// ✅ useReducer is better when:
// 1. State has multiple sub-values
// 2. Next state depends on previous state
// 3. Complex state transitions
// 4. You want predictable state updates

const initialState = {
  items: [],
  loading: false,
  error: null,
  filter: 'all',
  sortBy: 'date'
};
```

---

## Basic useReducer Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer`: Function that takes state and action, returns new state
- `initialState`: Initial state value
- `state`: Current state value
- `dispatch`: Function to send actions to the reducer

### Simple Counter Example

```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'SET':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// Initial state
const initialState = { count: 0 };

// Component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'SET', payload: 100 })}>
        Set to 100
      </button>
    </div>
  );
}
```

---

## The Reducer Pattern

### Anatomy of a Reducer

```jsx
function reducer(state, action) {
  // state: Current state
  // action: Object with type and optional payload

  switch (action.type) {
    case 'ACTION_TYPE':
      // Return NEW state (never mutate!)
      return { ...state, property: newValue };

    default:
      // Either return state unchanged or throw
      return state;
  }
}
```

### Rules for Reducers

1. **Pure function** - Same inputs always produce same outputs
2. **No side effects** - No API calls, timers, or random values
3. **Never mutate** - Always return new state objects
4. **Handle all action types** - Include a default case

```jsx
// ❌ BAD - Mutating state
function badReducer(state, action) {
  state.count = state.count + 1; // DON'T DO THIS!
  return state;
}

// ✅ GOOD - Returning new state
function goodReducer(state, action) {
  return {
    ...state,
    count: state.count + 1
  };
}
```

---

## Todo List Example

A complete example showing useReducer for managing a todo list:

```jsx
import { useReducer } from 'react';

// Action types (optional but helpful)
const ACTIONS = {
  ADD_TODO: 'ADD_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
  DELETE_TODO: 'DELETE_TODO',
  EDIT_TODO: 'EDIT_TODO',
  SET_FILTER: 'SET_FILTER',
  CLEAR_COMPLETED: 'CLEAR_COMPLETED'
};

// Reducer
function todoReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false
          }
        ]
      };

    case ACTIONS.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };

    case ACTIONS.DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };

    case ACTIONS.EDIT_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        )
      };

    case ACTIONS.SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };

    case ACTIONS.CLEAR_COMPLETED:
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };

    default:
      return state;
  }
}

// Initial state
const initialState = {
  todos: [],
  filter: 'all' // 'all', 'active', 'completed'
};

// Component
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');

  // Filter todos based on current filter
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!input.trim()) return;
    dispatch({ type: ACTIONS.ADD_TODO, payload: input });
    setInput('');
  };

  return (
    <div className="todo-app">
      <h1>Todo List</h1>

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Add todo..."
        />
        <button type="submit">Add</button>
      </form>

      <div className="filters">
        {['all', 'active', 'completed'].map(filter => (
          <button
            key={filter}
            className={state.filter === filter ? 'active' : ''}
            onClick={() => dispatch({ type: ACTIONS.SET_FILTER, payload: filter })}
          >
            {filter}
          </button>
        ))}
      </div>

      <ul className="todo-list">
        {filteredTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            dispatch={dispatch}
          />
        ))}
      </ul>

      <div className="actions">
        <span>{state.todos.filter(t => !t.completed).length} items left</span>
        <button onClick={() => dispatch({ type: ACTIONS.CLEAR_COMPLETED })}>
          Clear Completed
        </button>
      </div>
    </div>
  );
}

function TodoItem({ todo, dispatch }) {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleSave = () => {
    if (editText.trim()) {
      dispatch({
        type: ACTIONS.EDIT_TODO,
        payload: { id: todo.id, text: editText }
      });
    }
    setIsEditing(false);
  };

  if (isEditing) {
    return (
      <li>
        <input
          value={editText}
          onChange={(e) => setEditText(e.target.value)}
          onBlur={handleSave}
          onKeyDown={(e) => e.key === 'Enter' && handleSave()}
          autoFocus
        />
      </li>
    );
  }

  return (
    <li className={todo.completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => dispatch({ type: ACTIONS.TOGGLE_TODO, payload: todo.id })}
      />
      <span onDoubleClick={() => setIsEditing(true)}>{todo.text}</span>
      <button onClick={() => dispatch({ type: ACTIONS.DELETE_TODO, payload: todo.id })}>
        ×
      </button>
    </li>
  );
}
```

---

## Form State with useReducer

Managing complex form state:

```jsx
const initialFormState = {
  values: {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  },
  errors: {},
  touched: {},
  isSubmitting: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: {
          ...state.values,
          [action.field]: action.value
        }
      };

    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.error
        }
      };

    case 'SET_ERRORS':
      return {
        ...state,
        errors: action.errors
      };

    case 'SET_TOUCHED':
      return {
        ...state,
        touched: {
          ...state.touched,
          [action.field]: true
        }
      };

    case 'SET_SUBMITTING':
      return {
        ...state,
        isSubmitting: action.value
      };

    case 'RESET':
      return initialFormState;

    default:
      return state;
  }
}

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialFormState);

  const validate = () => {
    const errors = {};

    if (!state.values.username) {
      errors.username = 'Username is required';
    }

    if (!state.values.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(state.values.email)) {
      errors.email = 'Invalid email format';
    }

    if (!state.values.password) {
      errors.password = 'Password is required';
    } else if (state.values.password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }

    if (state.values.password !== state.values.confirmPassword) {
      errors.confirmPassword = 'Passwords do not match';
    }

    return errors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    dispatch({ type: 'SET_FIELD', field: name, value });
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    dispatch({ type: 'SET_TOUCHED', field: name });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const errors = validate();
    dispatch({ type: 'SET_ERRORS', errors });

    if (Object.keys(errors).length > 0) {
      return;
    }

    dispatch({ type: 'SET_SUBMITTING', value: true });

    try {
      await submitForm(state.values);
      dispatch({ type: 'RESET' });
      alert('Registration successful!');
    } catch (error) {
      dispatch({ type: 'SET_ERROR', field: 'submit', error: error.message });
    } finally {
      dispatch({ type: 'SET_SUBMITTING', value: false });
    }
  };

  const getFieldError = (field) => {
    return state.touched[field] && state.errors[field];
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="field">
        <input
          name="username"
          value={state.values.username}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Username"
        />
        {getFieldError('username') && (
          <span className="error">{state.errors.username}</span>
        )}
      </div>

      <div className="field">
        <input
          name="email"
          type="email"
          value={state.values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {getFieldError('email') && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>

      <div className="field">
        <input
          name="password"
          type="password"
          value={state.values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {getFieldError('password') && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>

      <div className="field">
        <input
          name="confirmPassword"
          type="password"
          value={state.values.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Confirm Password"
        />
        {getFieldError('confirmPassword') && (
          <span className="error">{state.errors.confirmPassword}</span>
        )}
      </div>

      {state.errors.submit && (
        <div className="error">{state.errors.submit}</div>
      )}

      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

---

## Async Actions with useReducer

Handling async operations:

```jsx
const initialState = {
  data: null,
  loading: false,
  error: null
};

function dataReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };

    case 'FETCH_SUCCESS':
      return { ...state, loading: false, data: action.payload };

    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };

    default:
      return state;
  }
}

function UserProfile({ userId }) {
  const [state, dispatch] = useReducer(dataReducer, initialState);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      dispatch({ type: 'FETCH_START' });

      try {
        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }

        const data = await response.json();

        if (!cancelled) {
          dispatch({ type: 'FETCH_SUCCESS', payload: data });
        }
      } catch (error) {
        if (!cancelled) {
          dispatch({ type: 'FETCH_ERROR', payload: error.message });
        }
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (state.loading) return <p>Loading...</p>;
  if (state.error) return <p>Error: {state.error}</p>;
  if (!state.data) return null;

  return (
    <div>
      <h1>{state.data.name}</h1>
      <p>{state.data.email}</p>
    </div>
  );
}
```

---

## useReducer + Context

The powerful combination for global state management:

```jsx
// src/context/TodoContext.jsx
import { createContext, useContext, useReducer } from 'react';

// Context
const TodoStateContext = createContext(null);
const TodoDispatchContext = createContext(null);

// Hooks
export function useTodoState() {
  const context = useContext(TodoStateContext);
  if (!context) {
    throw new Error('useTodoState must be used within TodoProvider');
  }
  return context;
}

export function useTodoDispatch() {
  const context = useContext(TodoDispatchContext);
  if (!context) {
    throw new Error('useTodoDispatch must be used within TodoProvider');
  }
  return context;
}

// Combined hook
export function useTodos() {
  return [useTodoState(), useTodoDispatch()];
}

// Action types
export const TODO_ACTIONS = {
  ADD: 'ADD',
  TOGGLE: 'TOGGLE',
  DELETE: 'DELETE',
  EDIT: 'EDIT'
};

// Reducer
function todoReducer(state, action) {
  switch (action.type) {
    case TODO_ACTIONS.ADD:
      return [
        ...state,
        { id: Date.now(), text: action.payload, completed: false }
      ];

    case TODO_ACTIONS.TOGGLE:
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );

    case TODO_ACTIONS.DELETE:
      return state.filter(todo => todo.id !== action.payload);

    case TODO_ACTIONS.EDIT:
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, text: action.payload.text }
          : todo
      );

    default:
      return state;
  }
}

// Provider
export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, []);

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        {children}
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}

// Action creators (optional helpers)
export const todoActions = {
  add: (text) => ({ type: TODO_ACTIONS.ADD, payload: text }),
  toggle: (id) => ({ type: TODO_ACTIONS.TOGGLE, payload: id }),
  delete: (id) => ({ type: TODO_ACTIONS.DELETE, payload: id }),
  edit: (id, text) => ({ type: TODO_ACTIONS.EDIT, payload: { id, text } })
};
```

### Using the Context

```jsx
// src/App.jsx
import { TodoProvider } from './context/TodoContext';
import { TodoList } from './components/TodoList';
import { AddTodo } from './components/AddTodo';

function App() {
  return (
    <TodoProvider>
      <h1>Todo App</h1>
      <AddTodo />
      <TodoList />
    </TodoProvider>
  );
}

// src/components/AddTodo.jsx
import { useState } from 'react';
import { useTodoDispatch, todoActions } from '../context/TodoContext';

function AddTodo() {
  const [text, setText] = useState('');
  const dispatch = useTodoDispatch();

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch(todoActions.add(text));
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

// src/components/TodoList.jsx
import { useTodoState, useTodoDispatch, todoActions } from '../context/TodoContext';

function TodoList() {
  const todos = useTodoState();
  const dispatch = useTodoDispatch();

  if (todos.length === 0) {
    return <p>No todos yet!</p>;
  }

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => dispatch(todoActions.toggle(todo.id))}
          />
          <span style={{
            textDecoration: todo.completed ? 'line-through' : 'none'
          }}>
            {todo.text}
          </span>
          <button onClick={() => dispatch(todoActions.delete(todo.id))}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

---

## TypeScript with useReducer

```tsx
// Types
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

type TodoAction =
  | { type: 'ADD'; payload: string }
  | { type: 'TOGGLE'; payload: number }
  | { type: 'DELETE'; payload: number }
  | { type: 'EDIT'; payload: { id: number; text: string } };

// Reducer with type safety
function todoReducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'ADD':
      return [
        ...state,
        { id: Date.now(), text: action.payload, completed: false }
      ];

    case 'TOGGLE':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );

    case 'DELETE':
      return state.filter(todo => todo.id !== action.payload);

    case 'EDIT':
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, text: action.payload.text }
          : todo
      );

    default:
      // TypeScript will error if we miss a case
      const _exhaustiveCheck: never = action;
      return state;
  }
}

// Usage
function TodoApp() {
  const [todos, dispatch] = useReducer(todoReducer, []);

  // TypeScript ensures correct action structure
  dispatch({ type: 'ADD', payload: 'New todo' }); // ✅
  dispatch({ type: 'TOGGLE', payload: 1 }); // ✅
  // dispatch({ type: 'ADD', payload: 123 }); // ❌ Type error
  // dispatch({ type: 'UNKNOWN' }); // ❌ Type error
}
```

---

## Lazy Initialization

For expensive initial state calculations:

```jsx
// Without lazy init - runs every render
const [state, dispatch] = useReducer(reducer, createExpensiveState());

// With lazy init - runs only once
function init(initialArg) {
  // Expensive computation
  return {
    ...initialArg,
    processedData: heavyProcessing(initialArg.rawData)
  };
}

const [state, dispatch] = useReducer(reducer, initialArg, init);
```

### Practical Example

```jsx
function init(savedData) {
  if (savedData) {
    return JSON.parse(savedData);
  }
  return { items: [], filter: 'all' };
}

function App() {
  const savedData = localStorage.getItem('appState');
  const [state, dispatch] = useReducer(
    appReducer,
    savedData,
    init
  );

  // ...
}
```

---

## Exercises

### Exercise 1: Shopping Cart Reducer

Create a shopping cart with:
- Add item (with quantity)
- Remove item
- Update quantity
- Apply discount code
- Calculate totals

```jsx
const initialState = {
  items: [],
  discountCode: null,
  discountPercent: 0
};

function cartReducer(state, action) {
  // TODO: Implement
}
```

### Exercise 2: Multi-Step Form

Create a multi-step form reducer:

```jsx
const initialState = {
  step: 1,
  totalSteps: 3,
  data: {
    step1: {},
    step2: {},
    step3: {}
  },
  errors: {},
  completed: false
};

function formReducer(state, action) {
  // TODO: Implement NEXT_STEP, PREV_STEP, UPDATE_DATA, SET_ERRORS, SUBMIT
}
```

---

## Solutions

### Solution 1: Shopping Cart Reducer

```jsx
const initialState = {
  items: [],
  discountCode: null,
  discountPercent: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingIndex = state.items.findIndex(
        item => item.id === action.payload.id
      );

      if (existingIndex >= 0) {
        const newItems = [...state.items];
        newItems[existingIndex] = {
          ...newItems[existingIndex],
          quantity: newItems[existingIndex].quantity + (action.payload.quantity || 1)
        };
        return { ...state, items: newItems };
      }

      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: action.payload.quantity || 1 }]
      };
    }

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };

    case 'UPDATE_QUANTITY': {
      const { id, quantity } = action.payload;
      if (quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(item => item.id !== id)
        };
      }
      return {
        ...state,
        items: state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        )
      };
    }

    case 'APPLY_DISCOUNT': {
      const discounts = {
        'SAVE10': 10,
        'SAVE20': 20,
        'HALF': 50
      };
      const percent = discounts[action.payload] || 0;
      return {
        ...state,
        discountCode: percent > 0 ? action.payload : null,
        discountPercent: percent
      };
    }

    case 'REMOVE_DISCOUNT':
      return {
        ...state,
        discountCode: null,
        discountPercent: 0
      };

    case 'CLEAR_CART':
      return initialState;

    default:
      return state;
  }
}

// Helper hook with computed values
function useCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const subtotal = state.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  const discount = subtotal * (state.discountPercent / 100);
  const total = subtotal - discount;
  const itemCount = state.items.reduce((sum, item) => sum + item.quantity, 0);

  return {
    ...state,
    subtotal,
    discount,
    total,
    itemCount,
    dispatch
  };
}
```

### Solution 2: Multi-Step Form

```jsx
const initialState = {
  step: 1,
  totalSteps: 3,
  data: {
    step1: { name: '', email: '' },
    step2: { address: '', city: '', zip: '' },
    step3: { cardNumber: '', expiry: '' }
  },
  errors: {},
  completed: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'NEXT_STEP':
      if (state.step < state.totalSteps) {
        return { ...state, step: state.step + 1, errors: {} };
      }
      return state;

    case 'PREV_STEP':
      if (state.step > 1) {
        return { ...state, step: state.step - 1, errors: {} };
      }
      return state;

    case 'GO_TO_STEP':
      if (action.payload >= 1 && action.payload <= state.totalSteps) {
        return { ...state, step: action.payload, errors: {} };
      }
      return state;

    case 'UPDATE_DATA':
      return {
        ...state,
        data: {
          ...state.data,
          [`step${state.step}`]: {
            ...state.data[`step${state.step}`],
            ...action.payload
          }
        }
      };

    case 'SET_ERRORS':
      return { ...state, errors: action.payload };

    case 'SUBMIT':
      return { ...state, completed: true };

    case 'RESET':
      return initialState;

    default:
      return state;
  }
}

function MultiStepForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const validateStep = () => {
    const currentData = state.data[`step${state.step}`];
    const errors = {};

    // Validation logic per step
    if (state.step === 1) {
      if (!currentData.name) errors.name = 'Name is required';
      if (!currentData.email) errors.email = 'Email is required';
    }
    // ... more validation

    return errors;
  };

  const handleNext = () => {
    const errors = validateStep();
    if (Object.keys(errors).length > 0) {
      dispatch({ type: 'SET_ERRORS', payload: errors });
      return;
    }

    if (state.step === state.totalSteps) {
      dispatch({ type: 'SUBMIT' });
    } else {
      dispatch({ type: 'NEXT_STEP' });
    }
  };

  if (state.completed) {
    return <div>Form submitted successfully!</div>;
  }

  return (
    <div>
      <div className="steps">
        {[1, 2, 3].map(step => (
          <button
            key={step}
            className={state.step === step ? 'active' : ''}
            onClick={() => dispatch({ type: 'GO_TO_STEP', payload: step })}
          >
            Step {step}
          </button>
        ))}
      </div>

      <StepContent
        step={state.step}
        data={state.data[`step${state.step}`]}
        errors={state.errors}
        onChange={(field, value) =>
          dispatch({ type: 'UPDATE_DATA', payload: { [field]: value } })
        }
      />

      <div className="navigation">
        {state.step > 1 && (
          <button onClick={() => dispatch({ type: 'PREV_STEP' })}>
            Previous
          </button>
        )}
        <button onClick={handleNext}>
          {state.step === state.totalSteps ? 'Submit' : 'Next'}
        </button>
      </div>
    </div>
  );
}
```

---

## Key Takeaways

1. **useReducer for complex state** - Multiple related values or complex update logic
2. **Reducer must be pure** - No side effects, no mutations
3. **Actions describe what happened** - Not how state should change
4. **Combine with Context** - For global state management
5. **TypeScript helps** - Ensures correct action types and payloads
6. **Lazy initialization** - For expensive initial state computation
7. **Split contexts** - Separate state and dispatch for performance

---

## What's Next?

Tomorrow we'll learn about **Performance Optimization** - useMemo, useCallback, React.memo, and strategies to keep your React apps fast!
