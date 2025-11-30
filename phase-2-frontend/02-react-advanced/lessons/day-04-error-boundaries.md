# Day 4: Error Boundaries - Handling Errors Gracefully

## Introduction

Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of crashing the entire application. They're essential for building robust, production-ready React applications.

## Learning Objectives

By the end of this lesson, you will:
- Understand what errors error boundaries catch
- Create error boundary components
- Implement fallback UIs for different scenarios
- Handle errors in event handlers and async code
- Build a comprehensive error handling strategy

---

## The Problem: Unhandled Errors

Without error boundaries, a single error crashes the entire React app:

```jsx
function BuggyComponent() {
  // This will crash the entire app!
  throw new Error('Something went wrong!');
  return <div>Hello</div>;
}

function App() {
  return (
    <div>
      <Header />
      <BuggyComponent /> {/* Error here crashes everything */}
      <Footer />
    </div>
  );
}
```

**Result:** White screen of death, terrible user experience.

---

## Creating an Error Boundary

Error boundaries must be class components (as of React 18):

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  // Update state when error occurs
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  // Log error information
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error);
    console.error('Error info:', errorInfo);

    // Log to error reporting service
    // logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h1>Something went wrong</h1>
          <p>We're sorry for the inconvenience. Please try refreshing the page.</p>
          <button onClick={() => window.location.reload()}>
            Refresh Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### Using the Error Boundary

```jsx
function App() {
  return (
    <ErrorBoundary>
      <Header />
      <main>
        <BuggyComponent /> {/* Error here shows fallback UI */}
      </main>
      <Footer />
    </ErrorBoundary>
  );
}
```

---

## Granular Error Boundaries

Wrap different sections separately for better UX:

```jsx
function App() {
  return (
    <div className="app">
      <ErrorBoundary fallback={<HeaderError />}>
        <Header />
      </ErrorBoundary>

      <main>
        <ErrorBoundary fallback={<SidebarError />}>
          <Sidebar />
        </ErrorBoundary>

        <ErrorBoundary fallback={<ContentError />}>
          <MainContent />
        </ErrorBoundary>
      </main>

      <ErrorBoundary fallback={<FooterError />}>
        <Footer />
      </ErrorBoundary>
    </div>
  );
}
```

Now if `MainContent` crashes, `Header`, `Sidebar`, and `Footer` still work!

---

## Customizable Error Boundary

Create a reusable error boundary with custom fallback:

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error service
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  resetError = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      // Use custom fallback if provided
      if (this.props.fallback) {
        return typeof this.props.fallback === 'function'
          ? this.props.fallback({
              error: this.state.error,
              resetError: this.resetError
            })
          : this.props.fallback;
      }

      // Default fallback
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### Usage with Custom Fallback

```jsx
// Simple fallback component
<ErrorBoundary fallback={<p>Error loading widget</p>}>
  <Widget />
</ErrorBoundary>

// Fallback with reset functionality
<ErrorBoundary
  fallback={({ error, resetError }) => (
    <div className="error-card">
      <h3>Widget Error</h3>
      <p>{error.message}</p>
      <button onClick={resetError}>Retry</button>
    </div>
  )}
>
  <Widget />
</ErrorBoundary>

// With error logging
<ErrorBoundary
  onError={(error, errorInfo) => {
    sendToErrorTracking(error, errorInfo);
  }}
  fallback={<WidgetError />}
>
  <Widget />
</ErrorBoundary>
```

---

## Error Boundary with Reset on Navigation

Reset error state when route changes:

```jsx
import { Component } from 'react';
import { useLocation } from 'react-router-dom';

class ErrorBoundaryClass extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidUpdate(prevProps) {
    // Reset error when location changes
    if (this.state.hasError && prevProps.location !== this.props.location) {
      this.setState({ hasError: false, error: null });
    }
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <DefaultError />;
    }
    return this.props.children;
  }
}

// Wrapper to use hooks with class component
function ErrorBoundary(props) {
  const location = useLocation();
  return <ErrorBoundaryClass {...props} location={location} />;
}

export default ErrorBoundary;
```

---

## What Error Boundaries DON'T Catch

Error boundaries do NOT catch errors in:

### 1. Event Handlers

```jsx
function Button() {
  const handleClick = () => {
    throw new Error('Click error!'); // NOT caught by error boundary
  };

  return <button onClick={handleClick}>Click me</button>;
}

// Solution: Use try-catch in event handlers
function Button() {
  const [error, setError] = useState(null);

  const handleClick = () => {
    try {
      riskyOperation();
    } catch (err) {
      setError(err);
      // or re-throw to let it bubble up
    }
  };

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

### 2. Asynchronous Code

```jsx
function AsyncComponent() {
  useEffect(() => {
    async function fetchData() {
      throw new Error('Async error!'); // NOT caught by error boundary
    }
    fetchData();
  }, []);

  return <div>Loading...</div>;
}

// Solution: Handle in the async function
function AsyncComponent() {
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const result = await riskyAsyncOperation();
        setData(result);
      } catch (err) {
        setError(err);
      }
    }
    fetchData();
  }, []);

  if (error) return <p>Error: {error.message}</p>;
  if (!data) return <p>Loading...</p>;
  return <div>{data}</div>;
}
```

### 3. Server-Side Rendering

```jsx
// Errors during SSR need different handling
// Use try-catch on the server
```

### 4. Errors in the Error Boundary Itself

```jsx
// If the error boundary throws, it can't catch its own error
// Always keep error boundaries simple and error-free
```

---

## react-error-boundary Library

A popular library that provides a flexible error boundary:

```bash
npm install react-error-boundary
```

### Basic Usage

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset application state here
      }}
      onError={(error, errorInfo) => {
        // Log error to service
        logError(error, errorInfo);
      }}
    >
      <MyApp />
    </ErrorBoundary>
  );
}
```

### With Reset Keys

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function UserProfile({ userId }) {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      resetKeys={[userId]} // Reset when userId changes
    >
      <Profile userId={userId} />
    </ErrorBoundary>
  );
}
```

### useErrorBoundary Hook

```jsx
import { useErrorBoundary } from 'react-error-boundary';

function DataLoader() {
  const { showBoundary } = useErrorBoundary();

  const loadData = async () => {
    try {
      const data = await fetchData();
      // use data
    } catch (error) {
      showBoundary(error); // Propagate to nearest error boundary
    }
  };

  return <button onClick={loadData}>Load Data</button>;
}
```

### withErrorBoundary HOC

```jsx
import { withErrorBoundary } from 'react-error-boundary';

function MyComponent() {
  // component code
}

export default withErrorBoundary(MyComponent, {
  fallback: <ErrorFallback />,
  onError: logError
});
```

---

## Building a Comprehensive Error Handling Strategy

### 1. Global Error Boundary

```jsx
// src/components/GlobalErrorBoundary.jsx
import { Component } from 'react';

class GlobalErrorBoundary extends Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error tracking service
    console.error('Global error:', error);

    // In production, send to service like Sentry
    if (process.env.NODE_ENV === 'production') {
      // Sentry.captureException(error, { extra: errorInfo });
    }
  }

  handleReload = () => {
    window.location.reload();
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="global-error">
          <h1>Oops! Something went wrong</h1>
          <p>We've been notified and are working on a fix.</p>
          <button onClick={this.handleReload}>
            Reload Application
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default GlobalErrorBoundary;
```

### 2. Feature-Level Error Boundaries

```jsx
// src/components/FeatureErrorBoundary.jsx
import { Component } from 'react';

class FeatureErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error(`Error in ${this.props.featureName}:`, error);
  }

  handleRetry = () => {
    this.setState({ hasError: false });
    if (this.props.onRetry) {
      this.props.onRetry();
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="feature-error">
          <p>Unable to load {this.props.featureName}</p>
          <button onClick={this.handleRetry}>Retry</button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<FeatureErrorBoundary featureName="Dashboard Widget">
  <DashboardWidget />
</FeatureErrorBoundary>
```

### 3. Async Error Handler Hook

```jsx
// src/hooks/useAsyncError.js
import { useState, useCallback } from 'react';

export function useAsyncError() {
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  const execute = useCallback(async (asyncFn) => {
    setLoading(true);
    setError(null);

    try {
      const result = await asyncFn();
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  const clearError = useCallback(() => {
    setError(null);
  }, []);

  return { error, loading, execute, clearError };
}

// Usage
function DataComponent() {
  const { error, loading, execute, clearError } = useAsyncError();
  const [data, setData] = useState(null);

  useEffect(() => {
    execute(async () => {
      const result = await fetchData();
      setData(result);
    });
  }, [execute]);

  if (loading) return <Loading />;

  if (error) {
    return (
      <div>
        <p>Error: {error.message}</p>
        <button onClick={() => {
          clearError();
          execute(async () => {
            const result = await fetchData();
            setData(result);
          });
        }}>
          Retry
        </button>
      </div>
    );
  }

  return <DisplayData data={data} />;
}
```

### 4. Complete App Structure

```jsx
// src/App.jsx
import { BrowserRouter } from 'react-router-dom';
import GlobalErrorBoundary from './components/GlobalErrorBoundary';
import { AppProviders } from './context';
import Router from './Router';

function App() {
  return (
    <GlobalErrorBoundary>
      <BrowserRouter>
        <AppProviders>
          <Router />
        </AppProviders>
      </BrowserRouter>
    </GlobalErrorBoundary>
  );
}

// src/Router.jsx
import { Routes, Route } from 'react-router-dom';
import FeatureErrorBoundary from './components/FeatureErrorBoundary';

function Router() {
  return (
    <Routes>
      <Route
        path="/dashboard"
        element={
          <FeatureErrorBoundary featureName="Dashboard">
            <Dashboard />
          </FeatureErrorBoundary>
        }
      />
      <Route
        path="/settings"
        element={
          <FeatureErrorBoundary featureName="Settings">
            <Settings />
          </FeatureErrorBoundary>
        }
      />
    </Routes>
  );
}
```

---

## Error Reporting Services

### Sentry Integration

```bash
npm install @sentry/react
```

```jsx
import * as Sentry from '@sentry/react';

// Initialize Sentry
Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  environment: process.env.NODE_ENV,
  integrations: [
    new Sentry.BrowserTracing(),
  ],
  tracesSampleRate: 0.1,
});

// Use Sentry's error boundary
function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={<ErrorFallback />}
      showDialog // Shows user feedback dialog
    >
      <MyApp />
    </Sentry.ErrorBoundary>
  );
}

// Manual error reporting
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { feature: 'checkout' },
    extra: { userId: user.id }
  });
}
```

---

## Exercises

### Exercise 1: Create a Retry Error Boundary

Create an error boundary that:
- Shows the error message
- Provides a "Retry" button
- Limits retries to 3 attempts
- Shows different UI after max retries

### Exercise 2: Error Boundary with Logging

Create an error boundary that:
- Logs errors to console in development
- Sends errors to an API endpoint in production
- Includes component stack trace
- Includes user information if available

---

## Solutions

### Solution 1: Retry Error Boundary

```jsx
import { Component } from 'react';

class RetryErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      retryCount: 0
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
  }

  handleRetry = () => {
    this.setState(prev => ({
      hasError: false,
      error: null,
      retryCount: prev.retryCount + 1
    }));
  };

  render() {
    const maxRetries = this.props.maxRetries || 3;

    if (this.state.hasError) {
      if (this.state.retryCount >= maxRetries) {
        return (
          <div className="error-boundary max-retries">
            <h2>Something went wrong</h2>
            <p>We've tried {maxRetries} times but couldn't recover.</p>
            <p>Error: {this.state.error?.message}</p>
            <button onClick={() => window.location.reload()}>
              Reload Page
            </button>
          </div>
        );
      }

      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>Error: {this.state.error?.message}</p>
          <p>Attempt {this.state.retryCount + 1} of {maxRetries}</p>
          <button onClick={this.handleRetry}>
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default RetryErrorBoundary;

// Usage
<RetryErrorBoundary maxRetries={3}>
  <UnstableComponent />
</RetryErrorBoundary>
```

### Solution 2: Logging Error Boundary

```jsx
import { Component, createContext, useContext } from 'react';

// User context for getting user info
const UserContext = createContext(null);

class LoggingErrorBoundary extends Component {
  static contextType = UserContext;

  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorId: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    const user = this.context;
    const errorId = `err_${Date.now()}`;

    const errorReport = {
      errorId,
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      user: user ? { id: user.id, email: user.email } : null
    };

    if (process.env.NODE_ENV === 'development') {
      console.group('Error Boundary Caught Error');
      console.error('Error:', error);
      console.error('Component Stack:', errorInfo.componentStack);
      console.log('Error Report:', errorReport);
      console.groupEnd();
    } else {
      // Send to error logging API
      this.logErrorToService(errorReport);
    }

    this.setState({ errorId });
  }

  logErrorToService = async (errorReport) => {
    try {
      await fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorReport)
      });
    } catch (err) {
      console.error('Failed to log error:', err);
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>We've logged this error and will look into it.</p>
          {this.state.errorId && (
            <p className="error-id">
              Error ID: <code>{this.state.errorId}</code>
            </p>
          )}
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default LoggingErrorBoundary;
```

---

## Key Takeaways

1. **Error boundaries prevent crashes** - Show fallback UI instead of white screen
2. **Use class components** - Error boundaries require lifecycle methods
3. **Be granular** - Wrap different features separately
4. **Handle async errors separately** - Use try-catch in async code
5. **Event handler errors** - Need manual handling with try-catch
6. **Use libraries** - `react-error-boundary` provides useful features
7. **Log errors** - Send to services like Sentry in production
8. **Provide recovery options** - Let users retry or navigate away

---

## What's Next?

Tomorrow we'll learn about **Portals and Refs** - rendering outside the DOM hierarchy and accessing DOM elements directly!
