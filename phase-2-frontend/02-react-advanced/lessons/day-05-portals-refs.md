# Day 5: Portals and Refs - Breaking Out of the Box

## Introduction

React Portals and Refs are powerful features that let you break React's normal component boundaries. Portals render children outside their parent DOM hierarchy, while Refs provide direct access to DOM elements. Both are essential for building modals, tooltips, focus management, and animations.

## Learning Objectives

By the end of this lesson, you will:
- Create portals for modals and tooltips
- Use useRef for DOM access
- Implement forward refs for component composition
- Build focus management systems
- Understand when and why to use these features

---

## Part 1: React Portals

### What Are Portals?

Portals let you render children into a DOM node that exists outside the parent component's DOM hierarchy:

```jsx
import { createPortal } from 'react-dom';

function Modal({ children }) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root') // Render outside parent
  );
}
```

### Why Use Portals?

1. **CSS stacking contexts** - Avoid z-index issues
2. **Overflow hidden** - Escape parent's overflow:hidden
3. **Semantic HTML** - Keep modals at body level
4. **Accessibility** - Proper focus management

### Setting Up Portal Root

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>
    <div id="modal-root"></div>
    <div id="tooltip-root"></div>
  </body>
</html>
```

---

## Building a Modal with Portals

### Basic Modal Component

```jsx
import { createPortal } from 'react-dom';
import { useEffect, useRef } from 'react';

function Modal({ isOpen, onClose, title, children }) {
  const previousFocus = useRef(null);

  useEffect(() => {
    if (isOpen) {
      // Save currently focused element
      previousFocus.current = document.activeElement;

      // Prevent body scroll
      document.body.style.overflow = 'hidden';
    }

    return () => {
      // Restore body scroll
      document.body.style.overflow = '';

      // Restore focus
      if (previousFocus.current) {
        previousFocus.current.focus();
      }
    };
  }, [isOpen]);

  // Handle escape key
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape' && isOpen) {
        onClose();
      }
    };

    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close modal"
            className="modal-close"
          >
            ×
          </button>
        </header>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>,
    document.getElementById('modal-root')
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <button onClick={() => setShowModal(true)}>Open Modal</button>

      <Modal
        isOpen={showModal}
        onClose={() => setShowModal(false)}
        title="Confirmation"
      >
        <p>Are you sure you want to proceed?</p>
        <div className="modal-actions">
          <button onClick={() => setShowModal(false)}>Cancel</button>
          <button onClick={() => {
            // Handle confirm
            setShowModal(false);
          }}>
            Confirm
          </button>
        </div>
      </Modal>
    </div>
  );
}
```

### Modal CSS

```css
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #eee;
}

.modal-body {
  padding: 20px;
}

.modal-close {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  padding: 0;
  line-height: 1;
}
```

---

## Tooltip with Portal

```jsx
import { createPortal } from 'react-dom';
import { useState, useRef, useEffect } from 'react';

function Tooltip({ children, content, position = 'top' }) {
  const [isVisible, setIsVisible] = useState(false);
  const [coords, setCoords] = useState({ top: 0, left: 0 });
  const triggerRef = useRef(null);
  const tooltipRef = useRef(null);

  useEffect(() => {
    if (isVisible && triggerRef.current) {
      const rect = triggerRef.current.getBoundingClientRect();
      const tooltipRect = tooltipRef.current?.getBoundingClientRect();

      let top, left;

      switch (position) {
        case 'top':
          top = rect.top - (tooltipRect?.height || 0) - 8;
          left = rect.left + rect.width / 2;
          break;
        case 'bottom':
          top = rect.bottom + 8;
          left = rect.left + rect.width / 2;
          break;
        case 'left':
          top = rect.top + rect.height / 2;
          left = rect.left - (tooltipRect?.width || 0) - 8;
          break;
        case 'right':
          top = rect.top + rect.height / 2;
          left = rect.right + 8;
          break;
        default:
          top = rect.top - (tooltipRect?.height || 0) - 8;
          left = rect.left + rect.width / 2;
      }

      setCoords({ top, left });
    }
  }, [isVisible, position]);

  return (
    <>
      <span
        ref={triggerRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
        onFocus={() => setIsVisible(true)}
        onBlur={() => setIsVisible(false)}
      >
        {children}
      </span>

      {isVisible && createPortal(
        <div
          ref={tooltipRef}
          className={`tooltip tooltip-${position}`}
          style={{
            position: 'fixed',
            top: coords.top,
            left: coords.left,
            transform: position === 'top' || position === 'bottom'
              ? 'translateX(-50%)'
              : 'translateY(-50%)'
          }}
          role="tooltip"
        >
          {content}
        </div>,
        document.getElementById('tooltip-root')
      )}
    </>
  );
}

// Usage
<Tooltip content="This is helpful information" position="top">
  <button>Hover me</button>
</Tooltip>
```

---

## Part 2: Refs in React

### What Are Refs?

Refs provide a way to access DOM nodes or store mutable values that don't trigger re-renders:

```jsx
import { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

### useRef Basics

```jsx
const myRef = useRef(initialValue);

// Access the current value
console.log(myRef.current);

// Update (doesn't cause re-render!)
myRef.current = newValue;
```

---

## Common useRef Use Cases

### 1. Focus Management

```jsx
function LoginForm() {
  const emailRef = useRef(null);
  const passwordRef = useRef(null);

  useEffect(() => {
    // Focus email input on mount
    emailRef.current?.focus();
  }, []);

  const handleEmailKeyDown = (e) => {
    if (e.key === 'Enter') {
      passwordRef.current?.focus();
    }
  };

  return (
    <form>
      <input
        ref={emailRef}
        type="email"
        placeholder="Email"
        onKeyDown={handleEmailKeyDown}
      />
      <input
        ref={passwordRef}
        type="password"
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

### 2. Storing Previous Values

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  });

  const prevCount = prevCountRef.current;

  return (
    <div>
      <p>Now: {count}, before: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

### 3. Storing Timeout/Interval IDs

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  const startTimer = () => {
    if (intervalRef.current) return; // Prevent multiple intervals

    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  // Cleanup on unmount
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

### 4. Tracking Component Mount State

```jsx
function AsyncComponent() {
  const [data, setData] = useState(null);
  const isMountedRef = useRef(true);

  useEffect(() => {
    async function fetchData() {
      const result = await fetch('/api/data');
      const json = await result.json();

      // Only update if still mounted
      if (isMountedRef.current) {
        setData(json);
      }
    }

    fetchData();

    return () => {
      isMountedRef.current = false;
    };
  }, []);

  return data ? <div>{data.name}</div> : <div>Loading...</div>;
}
```

### 5. Measuring DOM Elements

```jsx
function MeasuredBox() {
  const boxRef = useRef(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useEffect(() => {
    if (boxRef.current) {
      const { width, height } = boxRef.current.getBoundingClientRect();
      setDimensions({ width, height });
    }
  }, []);

  // Measure on resize
  useEffect(() => {
    const handleResize = () => {
      if (boxRef.current) {
        const { width, height } = boxRef.current.getBoundingClientRect();
        setDimensions({ width, height });
      }
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return (
    <div ref={boxRef} className="measured-box">
      <p>Width: {dimensions.width}px</p>
      <p>Height: {dimensions.height}px</p>
    </div>
  );
}
```

---

## forwardRef - Passing Refs to Child Components

When you need to pass a ref to a custom component:

### The Problem

```jsx
// ❌ This doesn't work - ref is not a prop!
function CustomInput({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

function Form() {
  const inputRef = useRef(null);
  return <CustomInput ref={inputRef} />; // ref won't be passed
}
```

### The Solution: forwardRef

```jsx
import { forwardRef, useRef } from 'react';

// ✅ Use forwardRef
const CustomInput = forwardRef(function CustomInput(props, ref) {
  return (
    <input
      ref={ref}
      className="custom-input"
      {...props}
    />
  );
});

// Usage
function Form() {
  const inputRef = useRef(null);

  const handleFocus = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Enter text" />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
}
```

### forwardRef with TypeScript

```tsx
import { forwardRef, InputHTMLAttributes } from 'react';

interface CustomInputProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string;
}

const CustomInput = forwardRef<HTMLInputElement, CustomInputProps>(
  function CustomInput({ label, ...props }, ref) {
    return (
      <div className="input-wrapper">
        <label>{label}</label>
        <input ref={ref} {...props} />
      </div>
    );
  }
);
```

---

## useImperativeHandle - Customizing Ref Values

Control what values are exposed via ref:

```jsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef(null);

  // Customize what's exposed to parent
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current?.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    },
    getValue: () => {
      return inputRef.current?.value;
    },
    // Don't expose the actual DOM node
  }), []);

  return <input ref={inputRef} {...props} />;
});

// Usage
function Form() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    const value = inputRef.current?.getValue();
    console.log('Value:', value);
    inputRef.current?.clear();
    inputRef.current?.focus();
  };

  return (
    <div>
      <FancyInput ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

### Complex Component with useImperativeHandle

```jsx
const VideoPlayer = forwardRef(function VideoPlayer({ src }, ref) {
  const videoRef = useRef(null);

  useImperativeHandle(ref, () => ({
    play: () => videoRef.current?.play(),
    pause: () => videoRef.current?.pause(),
    stop: () => {
      videoRef.current?.pause();
      videoRef.current.currentTime = 0;
    },
    seek: (time) => {
      videoRef.current.currentTime = time;
    },
    getDuration: () => videoRef.current?.duration,
    getCurrentTime: () => videoRef.current?.currentTime,
    setVolume: (volume) => {
      videoRef.current.volume = Math.max(0, Math.min(1, volume));
    }
  }), []);

  return (
    <video ref={videoRef} src={src}>
      Your browser does not support video.
    </video>
  );
});

// Usage
function VideoController() {
  const playerRef = useRef(null);

  return (
    <div>
      <VideoPlayer ref={playerRef} src="/video.mp4" />
      <div className="controls">
        <button onClick={() => playerRef.current?.play()}>Play</button>
        <button onClick={() => playerRef.current?.pause()}>Pause</button>
        <button onClick={() => playerRef.current?.stop()}>Stop</button>
        <button onClick={() => playerRef.current?.seek(0)}>Restart</button>
      </div>
    </div>
  );
}
```

---

## Callback Refs

For more control over when refs are set:

```jsx
function MeasureOnRender() {
  const [height, setHeight] = useState(0);

  // Callback ref - called when element is mounted/unmounted
  const measuredRef = useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <div ref={measuredRef}>
      <p>This content has dynamic height.</p>
      <p>Measured height: {height}px</p>
    </div>
  );
}
```

### Tracking Multiple Refs

```jsx
function DynamicList({ items }) {
  const itemRefs = useRef(new Map());

  const scrollToItem = (id) => {
    const element = itemRefs.current.get(id);
    element?.scrollIntoView({ behavior: 'smooth' });
  };

  return (
    <div>
      <nav>
        {items.map(item => (
          <button key={item.id} onClick={() => scrollToItem(item.id)}>
            Go to {item.name}
          </button>
        ))}
      </nav>

      <ul>
        {items.map(item => (
          <li
            key={item.id}
            ref={(node) => {
              if (node) {
                itemRefs.current.set(item.id, node);
              } else {
                itemRefs.current.delete(item.id);
              }
            }}
          >
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Exercises

### Exercise 1: Dropdown with Portal

Create a dropdown menu using portals:

```jsx
function Dropdown({ trigger, children }) {
  // TODO:
  // - Position dropdown relative to trigger
  // - Use portal to render menu
  // - Close on click outside
  // - Close on Escape key
}
```

### Exercise 2: Auto-focus Form

Create a form that:
- Auto-focuses first empty field on mount
- Moves focus to next field on Enter
- Focuses first error field on validation failure

---

## Solutions

### Solution 1: Dropdown with Portal

```jsx
import { createPortal } from 'react-dom';
import { useState, useRef, useEffect } from 'react';

function Dropdown({ trigger, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const triggerRef = useRef(null);
  const menuRef = useRef(null);

  // Position dropdown
  useEffect(() => {
    if (isOpen && triggerRef.current) {
      const rect = triggerRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY + 4,
        left: rect.left + window.scrollX
      });
    }
  }, [isOpen]);

  // Close on click outside
  useEffect(() => {
    const handleClickOutside = (e) => {
      if (
        isOpen &&
        !triggerRef.current?.contains(e.target) &&
        !menuRef.current?.contains(e.target)
      ) {
        setIsOpen(false);
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [isOpen]);

  // Close on Escape
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape' && isOpen) {
        setIsOpen(false);
        triggerRef.current?.focus();
      }
    };

    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen]);

  return (
    <>
      <div
        ref={triggerRef}
        onClick={() => setIsOpen(!isOpen)}
        aria-haspopup="true"
        aria-expanded={isOpen}
      >
        {trigger}
      </div>

      {isOpen && createPortal(
        <div
          ref={menuRef}
          className="dropdown-menu"
          style={{
            position: 'absolute',
            top: position.top,
            left: position.left
          }}
          role="menu"
        >
          {children}
        </div>,
        document.body
      )}
    </>
  );
}

// Usage
function App() {
  return (
    <Dropdown trigger={<button>Options ▼</button>}>
      <ul className="menu-list">
        <li role="menuitem" onClick={() => console.log('Edit')}>Edit</li>
        <li role="menuitem" onClick={() => console.log('Delete')}>Delete</li>
        <li role="menuitem" onClick={() => console.log('Share')}>Share</li>
      </ul>
    </Dropdown>
  );
}
```

### Solution 2: Auto-focus Form

```jsx
import { useRef, useEffect, useState } from 'react';

function AutoFocusForm() {
  const [values, setValues] = useState({
    name: '',
    email: '',
    phone: ''
  });
  const [errors, setErrors] = useState({});

  const nameRef = useRef(null);
  const emailRef = useRef(null);
  const phoneRef = useRef(null);

  const fieldRefs = {
    name: nameRef,
    email: emailRef,
    phone: phoneRef
  };

  const fieldOrder = ['name', 'email', 'phone'];

  // Auto-focus first empty field on mount
  useEffect(() => {
    for (const field of fieldOrder) {
      if (!values[field]) {
        fieldRefs[field].current?.focus();
        break;
      }
    }
  }, []);

  // Focus first error field when errors change
  useEffect(() => {
    const errorFields = Object.keys(errors);
    if (errorFields.length > 0) {
      const firstErrorField = fieldOrder.find(f => errors[f]);
      if (firstErrorField) {
        fieldRefs[firstErrorField].current?.focus();
      }
    }
  }, [errors]);

  const handleKeyDown = (e, currentField) => {
    if (e.key === 'Enter') {
      e.preventDefault();
      const currentIndex = fieldOrder.indexOf(currentField);
      const nextField = fieldOrder[currentIndex + 1];

      if (nextField) {
        fieldRefs[nextField].current?.focus();
      } else {
        handleSubmit(e);
      }
    }
  };

  const handleChange = (field) => (e) => {
    setValues(prev => ({ ...prev, [field]: e.target.value }));
    // Clear error when user types
    if (errors[field]) {
      setErrors(prev => {
        const { [field]: _, ...rest } = prev;
        return rest;
      });
    }
  };

  const validate = () => {
    const newErrors = {};

    if (!values.name.trim()) {
      newErrors.name = 'Name is required';
    }
    if (!values.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      newErrors.email = 'Invalid email format';
    }
    if (!values.phone.trim()) {
      newErrors.phone = 'Phone is required';
    }

    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    console.log('Form submitted:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="field">
        <label htmlFor="name">Name</label>
        <input
          ref={nameRef}
          id="name"
          value={values.name}
          onChange={handleChange('name')}
          onKeyDown={(e) => handleKeyDown(e, 'name')}
          aria-invalid={!!errors.name}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>

      <div className="field">
        <label htmlFor="email">Email</label>
        <input
          ref={emailRef}
          id="email"
          type="email"
          value={values.email}
          onChange={handleChange('email')}
          onKeyDown={(e) => handleKeyDown(e, 'email')}
          aria-invalid={!!errors.email}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div className="field">
        <label htmlFor="phone">Phone</label>
        <input
          ref={phoneRef}
          id="phone"
          type="tel"
          value={values.phone}
          onChange={handleChange('phone')}
          onKeyDown={(e) => handleKeyDown(e, 'phone')}
          aria-invalid={!!errors.phone}
        />
        {errors.phone && <span className="error">{errors.phone}</span>}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Key Takeaways

1. **Portals** render children outside parent DOM hierarchy
2. **Use portals for** modals, tooltips, dropdowns, notifications
3. **useRef** stores mutable values without causing re-renders
4. **Refs access DOM** elements for focus, measurements, animations
5. **forwardRef** passes refs to custom components
6. **useImperativeHandle** customizes what refs expose
7. **Callback refs** give control over when refs are set
8. **Events still bubble** through portals to React tree

---

## What's Next?

Tomorrow we'll explore **Advanced React Patterns** - Compound Components, Render Props, Higher-Order Components, and more design patterns for building flexible, reusable components!
