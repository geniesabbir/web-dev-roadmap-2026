# Day 10: Mini Project - Task Manager Application

## Introduction

Today we'll build a complete Task Manager application that combines everything you've learned in React Fundamentals. This project will reinforce your understanding of components, props, state, events, conditional rendering, lists, forms, effects, and custom hooks.

## Project Overview

We'll build a feature-rich task manager with:
- Task creation with title, description, and priority
- Task filtering (all, active, completed)
- Task editing and deletion
- Local storage persistence
- Search functionality
- Statistics dashboard
- Responsive design

---

## Project Setup

```bash
# Create new Vite project
npm create vite@latest task-manager -- --template react

# Navigate and install dependencies
cd task-manager
npm install

# Start development server
npm run dev
```

### Project Structure

```
task-manager/
├── src/
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── TaskForm.jsx
│   │   ├── TaskList.jsx
│   │   ├── TaskItem.jsx
│   │   ├── TaskFilters.jsx
│   │   ├── SearchBar.jsx
│   │   └── Stats.jsx
│   ├── hooks/
│   │   ├── useLocalStorage.js
│   │   ├── useTasks.js
│   │   └── useDebounce.js
│   ├── App.jsx
│   ├── App.css
│   └── main.jsx
└── package.json
```

---

## Step 1: Custom Hooks

Let's start by creating the reusable hooks our app needs.

### useLocalStorage.js

```jsx
// src/hooks/useLocalStorage.js
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initialValue) {
  // Get initial value from localStorage or use provided initial value
  const [value, setValue] = useState(() => {
    try {
      const storedValue = localStorage.getItem(key);
      return storedValue ? JSON.parse(storedValue) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Update localStorage whenever value changes
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, value]);

  return [value, setValue];
}
```

### useDebounce.js

```jsx
// src/hooks/useDebounce.js
import { useState, useEffect } from 'react';

export function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timeoutId);
  }, [value, delay]);

  return debouncedValue;
}
```

### useTasks.js

```jsx
// src/hooks/useTasks.js
import { useState, useMemo, useCallback } from 'react';
import { useLocalStorage } from './useLocalStorage';

export function useTasks() {
  const [tasks, setTasks] = useLocalStorage('tasks', []);
  const [filter, setFilter] = useState('all'); // 'all', 'active', 'completed'
  const [searchQuery, setSearchQuery] = useState('');

  // Add a new task
  const addTask = useCallback((taskData) => {
    const newTask = {
      id: Date.now(),
      title: taskData.title,
      description: taskData.description || '',
      priority: taskData.priority || 'medium',
      completed: false,
      createdAt: new Date().toISOString()
    };

    setTasks(prev => [newTask, ...prev]);
  }, [setTasks]);

  // Toggle task completion
  const toggleTask = useCallback((id) => {
    setTasks(prev =>
      prev.map(task =>
        task.id === id ? { ...task, completed: !task.completed } : task
      )
    );
  }, [setTasks]);

  // Update a task
  const updateTask = useCallback((id, updates) => {
    setTasks(prev =>
      prev.map(task =>
        task.id === id ? { ...task, ...updates } : task
      )
    );
  }, [setTasks]);

  // Delete a task
  const deleteTask = useCallback((id) => {
    setTasks(prev => prev.filter(task => task.id !== id));
  }, [setTasks]);

  // Clear completed tasks
  const clearCompleted = useCallback(() => {
    setTasks(prev => prev.filter(task => !task.completed));
  }, [setTasks]);

  // Filter and search tasks
  const filteredTasks = useMemo(() => {
    return tasks
      .filter(task => {
        // Apply status filter
        if (filter === 'active') return !task.completed;
        if (filter === 'completed') return task.completed;
        return true;
      })
      .filter(task => {
        // Apply search filter
        if (!searchQuery) return true;
        const query = searchQuery.toLowerCase();
        return (
          task.title.toLowerCase().includes(query) ||
          task.description.toLowerCase().includes(query)
        );
      });
  }, [tasks, filter, searchQuery]);

  // Statistics
  const stats = useMemo(() => ({
    total: tasks.length,
    active: tasks.filter(t => !t.completed).length,
    completed: tasks.filter(t => t.completed).length,
    highPriority: tasks.filter(t => t.priority === 'high' && !t.completed).length
  }), [tasks]);

  return {
    tasks: filteredTasks,
    allTasks: tasks,
    filter,
    setFilter,
    searchQuery,
    setSearchQuery,
    addTask,
    toggleTask,
    updateTask,
    deleteTask,
    clearCompleted,
    stats
  };
}
```

---

## Step 2: Components

### Header.jsx

```jsx
// src/components/Header.jsx
export function Header() {
  return (
    <header className="header">
      <h1>Task Manager</h1>
      <p>Organize your tasks efficiently</p>
    </header>
  );
}
```

### Stats.jsx

```jsx
// src/components/Stats.jsx
export function Stats({ stats }) {
  return (
    <div className="stats">
      <div className="stat-card">
        <span className="stat-value">{stats.total}</span>
        <span className="stat-label">Total</span>
      </div>
      <div className="stat-card active">
        <span className="stat-value">{stats.active}</span>
        <span className="stat-label">Active</span>
      </div>
      <div className="stat-card completed">
        <span className="stat-value">{stats.completed}</span>
        <span className="stat-label">Completed</span>
      </div>
      <div className="stat-card high-priority">
        <span className="stat-value">{stats.highPriority}</span>
        <span className="stat-label">High Priority</span>
      </div>
    </div>
  );
}
```

### SearchBar.jsx

```jsx
// src/components/SearchBar.jsx
export function SearchBar({ value, onChange }) {
  return (
    <div className="search-bar">
      <input
        type="text"
        placeholder="Search tasks..."
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="search-input"
      />
      {value && (
        <button
          className="clear-search"
          onClick={() => onChange('')}
          aria-label="Clear search"
        >
          ×
        </button>
      )}
    </div>
  );
}
```

### TaskFilters.jsx

```jsx
// src/components/TaskFilters.jsx
export function TaskFilters({ filter, onFilterChange, onClearCompleted, hasCompleted }) {
  const filters = [
    { value: 'all', label: 'All' },
    { value: 'active', label: 'Active' },
    { value: 'completed', label: 'Completed' }
  ];

  return (
    <div className="task-filters">
      <div className="filter-buttons">
        {filters.map(f => (
          <button
            key={f.value}
            className={`filter-btn ${filter === f.value ? 'active' : ''}`}
            onClick={() => onFilterChange(f.value)}
          >
            {f.label}
          </button>
        ))}
      </div>

      {hasCompleted && (
        <button
          className="clear-completed-btn"
          onClick={onClearCompleted}
        >
          Clear Completed
        </button>
      )}
    </div>
  );
}
```

### TaskForm.jsx

```jsx
// src/components/TaskForm.jsx
import { useState } from 'react';

export function TaskForm({ onSubmit }) {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [priority, setPriority] = useState('medium');
  const [isExpanded, setIsExpanded] = useState(false);

  const handleSubmit = (e) => {
    e.preventDefault();

    if (!title.trim()) return;

    onSubmit({
      title: title.trim(),
      description: description.trim(),
      priority
    });

    // Reset form
    setTitle('');
    setDescription('');
    setPriority('medium');
    setIsExpanded(false);
  };

  return (
    <form className="task-form" onSubmit={handleSubmit}>
      <div className="form-main">
        <input
          type="text"
          placeholder="What needs to be done?"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          className="task-input"
          onFocus={() => setIsExpanded(true)}
        />
        <button type="submit" className="add-btn" disabled={!title.trim()}>
          Add Task
        </button>
      </div>

      {isExpanded && (
        <div className="form-details">
          <textarea
            placeholder="Add a description (optional)"
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            className="description-input"
            rows={2}
          />

          <div className="priority-select">
            <label>Priority:</label>
            <select
              value={priority}
              onChange={(e) => setPriority(e.target.value)}
            >
              <option value="low">Low</option>
              <option value="medium">Medium</option>
              <option value="high">High</option>
            </select>
          </div>

          <button
            type="button"
            className="collapse-btn"
            onClick={() => setIsExpanded(false)}
          >
            Collapse
          </button>
        </div>
      )}
    </form>
  );
}
```

### TaskItem.jsx

```jsx
// src/components/TaskItem.jsx
import { useState } from 'react';

export function TaskItem({ task, onToggle, onUpdate, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  const [editTitle, setEditTitle] = useState(task.title);
  const [editDescription, setEditDescription] = useState(task.description);

  const handleSave = () => {
    if (!editTitle.trim()) return;

    onUpdate(task.id, {
      title: editTitle.trim(),
      description: editDescription.trim()
    });
    setIsEditing(false);
  };

  const handleCancel = () => {
    setEditTitle(task.title);
    setEditDescription(task.description);
    setIsEditing(false);
  };

  const handleKeyDown = (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSave();
    }
    if (e.key === 'Escape') {
      handleCancel();
    }
  };

  const priorityClass = `priority-${task.priority}`;
  const completedClass = task.completed ? 'completed' : '';

  if (isEditing) {
    return (
      <div className={`task-item editing ${priorityClass}`}>
        <input
          type="text"
          value={editTitle}
          onChange={(e) => setEditTitle(e.target.value)}
          onKeyDown={handleKeyDown}
          className="edit-title-input"
          autoFocus
        />
        <textarea
          value={editDescription}
          onChange={(e) => setEditDescription(e.target.value)}
          onKeyDown={handleKeyDown}
          className="edit-description-input"
          placeholder="Description"
          rows={2}
        />
        <div className="edit-actions">
          <button className="save-btn" onClick={handleSave}>
            Save
          </button>
          <button className="cancel-btn" onClick={handleCancel}>
            Cancel
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className={`task-item ${priorityClass} ${completedClass}`}>
      <div className="task-checkbox">
        <input
          type="checkbox"
          checked={task.completed}
          onChange={() => onToggle(task.id)}
          id={`task-${task.id}`}
        />
        <label htmlFor={`task-${task.id}`} className="checkbox-label" />
      </div>

      <div className="task-content">
        <h3 className="task-title">{task.title}</h3>
        {task.description && (
          <p className="task-description">{task.description}</p>
        )}
        <div className="task-meta">
          <span className={`priority-badge ${priorityClass}`}>
            {task.priority}
          </span>
          <span className="created-date">
            {new Date(task.createdAt).toLocaleDateString()}
          </span>
        </div>
      </div>

      <div className="task-actions">
        <button
          className="edit-btn"
          onClick={() => setIsEditing(true)}
          aria-label="Edit task"
        >
          Edit
        </button>
        <button
          className="delete-btn"
          onClick={() => onDelete(task.id)}
          aria-label="Delete task"
        >
          Delete
        </button>
      </div>
    </div>
  );
}
```

### TaskList.jsx

```jsx
// src/components/TaskList.jsx
import { TaskItem } from './TaskItem';

export function TaskList({ tasks, onToggle, onUpdate, onDelete }) {
  if (tasks.length === 0) {
    return (
      <div className="empty-state">
        <p>No tasks found</p>
        <span>Add a new task or adjust your filters</span>
      </div>
    );
  }

  return (
    <div className="task-list">
      {tasks.map(task => (
        <TaskItem
          key={task.id}
          task={task}
          onToggle={onToggle}
          onUpdate={onUpdate}
          onDelete={onDelete}
        />
      ))}
    </div>
  );
}
```

---

## Step 3: Main App Component

### App.jsx

```jsx
// src/App.jsx
import { Header } from './components/Header';
import { Stats } from './components/Stats';
import { SearchBar } from './components/SearchBar';
import { TaskFilters } from './components/TaskFilters';
import { TaskForm } from './components/TaskForm';
import { TaskList } from './components/TaskList';
import { useTasks } from './hooks/useTasks';
import { useDebounce } from './hooks/useDebounce';
import './App.css';

function App() {
  const {
    tasks,
    filter,
    setFilter,
    searchQuery,
    setSearchQuery,
    addTask,
    toggleTask,
    updateTask,
    deleteTask,
    clearCompleted,
    stats
  } = useTasks();

  // Debounce search for better performance
  const debouncedSearch = useDebounce(searchQuery, 300);

  return (
    <div className="app">
      <Header />

      <main className="main-content">
        <Stats stats={stats} />

        <TaskForm onSubmit={addTask} />

        <div className="task-controls">
          <SearchBar
            value={searchQuery}
            onChange={setSearchQuery}
          />
          <TaskFilters
            filter={filter}
            onFilterChange={setFilter}
            onClearCompleted={clearCompleted}
            hasCompleted={stats.completed > 0}
          />
        </div>

        <TaskList
          tasks={tasks}
          onToggle={toggleTask}
          onUpdate={updateTask}
          onDelete={deleteTask}
        />
      </main>

      <footer className="footer">
        <p>Built with React - Task Manager v1.0</p>
      </footer>
    </div>
  );
}

export default App;
```

---

## Step 4: Styling

### App.css

```css
/* src/App.css */

/* Reset and Base Styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --primary: #6366f1;
  --primary-dark: #4f46e5;
  --success: #22c55e;
  --warning: #f59e0b;
  --danger: #ef4444;
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-400: #9ca3af;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;
  --shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --radius: 8px;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--gray-100);
  color: var(--gray-800);
  line-height: 1.5;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header */
.header {
  background: linear-gradient(135deg, var(--primary), var(--primary-dark));
  color: white;
  padding: 2rem;
  text-align: center;
}

.header h1 {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.header p {
  opacity: 0.9;
}

/* Main Content */
.main-content {
  flex: 1;
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  width: 100%;
}

/* Stats */
.stats {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: white;
  padding: 1rem;
  border-radius: var(--radius);
  text-align: center;
  box-shadow: var(--shadow);
}

.stat-value {
  display: block;
  font-size: 2rem;
  font-weight: bold;
  color: var(--gray-800);
}

.stat-label {
  font-size: 0.875rem;
  color: var(--gray-500);
}

.stat-card.active .stat-value {
  color: var(--primary);
}

.stat-card.completed .stat-value {
  color: var(--success);
}

.stat-card.high-priority .stat-value {
  color: var(--danger);
}

/* Task Form */
.task-form {
  background: white;
  border-radius: var(--radius);
  padding: 1rem;
  box-shadow: var(--shadow);
  margin-bottom: 1.5rem;
}

.form-main {
  display: flex;
  gap: 0.5rem;
}

.task-input {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 2px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 1rem;
  transition: border-color 0.2s;
}

.task-input:focus {
  outline: none;
  border-color: var(--primary);
}

.add-btn {
  padding: 0.75rem 1.5rem;
  background: var(--primary);
  color: white;
  border: none;
  border-radius: var(--radius);
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.add-btn:hover:not(:disabled) {
  background: var(--primary-dark);
}

.add-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.form-details {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid var(--gray-200);
}

.description-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 0.875rem;
  resize: vertical;
  font-family: inherit;
}

.description-input:focus {
  outline: none;
  border-color: var(--primary);
}

.priority-select {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

.priority-select select {
  padding: 0.5rem;
  border: 2px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 0.875rem;
}

.collapse-btn {
  margin-top: 0.75rem;
  padding: 0.5rem 1rem;
  background: var(--gray-100);
  border: none;
  border-radius: var(--radius);
  cursor: pointer;
  font-size: 0.875rem;
}

/* Task Controls */
.task-controls {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

/* Search Bar */
.search-bar {
  position: relative;
}

.search-input {
  width: 100%;
  padding: 0.75rem 1rem;
  padding-right: 2.5rem;
  border: 2px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 1rem;
  background: white;
}

.search-input:focus {
  outline: none;
  border-color: var(--primary);
}

.clear-search {
  position: absolute;
  right: 0.5rem;
  top: 50%;
  transform: translateY(-50%);
  background: var(--gray-200);
  border: none;
  width: 1.5rem;
  height: 1.5rem;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1rem;
  line-height: 1;
}

/* Task Filters */
.task-filters {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.filter-buttons {
  display: flex;
  gap: 0.25rem;
  background: white;
  padding: 0.25rem;
  border-radius: var(--radius);
  box-shadow: var(--shadow);
}

.filter-btn {
  padding: 0.5rem 1rem;
  border: none;
  background: transparent;
  border-radius: calc(var(--radius) - 2px);
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.filter-btn:hover {
  background: var(--gray-100);
}

.filter-btn.active {
  background: var(--primary);
  color: white;
}

.clear-completed-btn {
  padding: 0.5rem 1rem;
  background: white;
  border: 1px solid var(--gray-300);
  border-radius: var(--radius);
  cursor: pointer;
  font-size: 0.875rem;
  color: var(--gray-600);
}

.clear-completed-btn:hover {
  background: var(--gray-50);
  border-color: var(--danger);
  color: var(--danger);
}

/* Task List */
.task-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.empty-state {
  text-align: center;
  padding: 3rem;
  background: white;
  border-radius: var(--radius);
  box-shadow: var(--shadow);
}

.empty-state p {
  font-size: 1.125rem;
  color: var(--gray-600);
  margin-bottom: 0.25rem;
}

.empty-state span {
  font-size: 0.875rem;
  color: var(--gray-400);
}

/* Task Item */
.task-item {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  padding: 1rem;
  background: white;
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  border-left: 4px solid var(--gray-300);
  transition: all 0.2s;
}

.task-item:hover {
  box-shadow: var(--shadow-lg);
}

.task-item.priority-low {
  border-left-color: var(--success);
}

.task-item.priority-medium {
  border-left-color: var(--warning);
}

.task-item.priority-high {
  border-left-color: var(--danger);
}

.task-item.completed {
  opacity: 0.7;
  background: var(--gray-50);
}

.task-item.completed .task-title {
  text-decoration: line-through;
  color: var(--gray-400);
}

/* Checkbox */
.task-checkbox {
  flex-shrink: 0;
  position: relative;
}

.task-checkbox input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

.checkbox-label {
  display: block;
  width: 1.5rem;
  height: 1.5rem;
  border: 2px solid var(--gray-300);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s;
}

.task-checkbox input:checked + .checkbox-label {
  background: var(--success);
  border-color: var(--success);
}

.task-checkbox input:checked + .checkbox-label::after {
  content: '✓';
  color: white;
  font-size: 0.875rem;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Task Content */
.task-content {
  flex: 1;
  min-width: 0;
}

.task-title {
  font-size: 1rem;
  font-weight: 500;
  margin-bottom: 0.25rem;
  word-break: break-word;
}

.task-description {
  font-size: 0.875rem;
  color: var(--gray-500);
  margin-bottom: 0.5rem;
  word-break: break-word;
}

.task-meta {
  display: flex;
  gap: 0.75rem;
  font-size: 0.75rem;
}

.priority-badge {
  padding: 0.125rem 0.5rem;
  border-radius: 999px;
  font-weight: 500;
  text-transform: capitalize;
}

.priority-badge.priority-low {
  background: #dcfce7;
  color: #166534;
}

.priority-badge.priority-medium {
  background: #fef3c7;
  color: #92400e;
}

.priority-badge.priority-high {
  background: #fee2e2;
  color: #991b1b;
}

.created-date {
  color: var(--gray-400);
}

/* Task Actions */
.task-actions {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;
}

.edit-btn,
.delete-btn {
  padding: 0.375rem 0.75rem;
  border: none;
  border-radius: var(--radius);
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.edit-btn {
  background: var(--gray-100);
  color: var(--gray-600);
}

.edit-btn:hover {
  background: var(--primary);
  color: white;
}

.delete-btn {
  background: var(--gray-100);
  color: var(--gray-600);
}

.delete-btn:hover {
  background: var(--danger);
  color: white;
}

/* Editing State */
.task-item.editing {
  flex-direction: column;
}

.edit-title-input,
.edit-description-input {
  width: 100%;
  padding: 0.5rem;
  border: 2px solid var(--primary);
  border-radius: var(--radius);
  font-size: 1rem;
  font-family: inherit;
}

.edit-title-input {
  margin-bottom: 0.5rem;
}

.edit-description-input {
  resize: vertical;
  font-size: 0.875rem;
}

.edit-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

.save-btn,
.cancel-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: var(--radius);
  cursor: pointer;
  font-size: 0.875rem;
}

.save-btn {
  background: var(--primary);
  color: white;
}

.cancel-btn {
  background: var(--gray-200);
}

/* Footer */
.footer {
  text-align: center;
  padding: 1.5rem;
  background: white;
  color: var(--gray-500);
  font-size: 0.875rem;
  border-top: 1px solid var(--gray-200);
}

/* Responsive */
@media (max-width: 640px) {
  .stats {
    grid-template-columns: repeat(2, 1fr);
  }

  .main-content {
    padding: 1rem;
  }

  .form-main {
    flex-direction: column;
  }

  .task-filters {
    flex-direction: column;
    align-items: stretch;
  }

  .filter-buttons {
    justify-content: center;
  }

  .task-item {
    flex-wrap: wrap;
  }

  .task-actions {
    width: 100%;
    justify-content: flex-end;
    margin-top: 0.5rem;
  }
}
```

---

## Step 5: Testing Your Application

### Manual Testing Checklist

Test each feature:

1. **Adding Tasks**
   - [ ] Add a task with just a title
   - [ ] Add a task with title and description
   - [ ] Add tasks with different priorities
   - [ ] Verify tasks appear at the top of the list

2. **Completing Tasks**
   - [ ] Toggle task completion
   - [ ] Verify completed tasks show strikethrough
   - [ ] Verify stats update correctly

3. **Editing Tasks**
   - [ ] Click edit and modify title
   - [ ] Click edit and modify description
   - [ ] Press Enter to save
   - [ ] Press Escape to cancel

4. **Deleting Tasks**
   - [ ] Delete individual tasks
   - [ ] Clear all completed tasks

5. **Filtering**
   - [ ] Filter by "All"
   - [ ] Filter by "Active"
   - [ ] Filter by "Completed"

6. **Searching**
   - [ ] Search by title
   - [ ] Search by description
   - [ ] Clear search

7. **Persistence**
   - [ ] Refresh page and verify tasks persist
   - [ ] Close and reopen browser

---

## Bonus Features to Add

### 1. Due Dates

```jsx
// Add to TaskForm and TaskItem
const [dueDate, setDueDate] = useState('');

// In form
<input
  type="date"
  value={dueDate}
  onChange={(e) => setDueDate(e.target.value)}
  min={new Date().toISOString().split('T')[0]}
/>

// In TaskItem - show if overdue
const isOverdue = task.dueDate && new Date(task.dueDate) < new Date();
```

### 2. Drag and Drop Reordering

```jsx
// Install react-beautiful-dnd
// npm install react-beautiful-dnd

import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

// Wrap TaskList with DragDropContext
```

### 3. Categories/Tags

```jsx
// Add categories array to task
const [categories, setCategories] = useState([]);

// Display as color-coded tags
<div className="tags">
  {task.categories.map(cat => (
    <span key={cat} className={`tag tag-${cat}`}>{cat}</span>
  ))}
</div>
```

### 4. Keyboard Shortcuts

```jsx
// Add useKeyboardShortcuts hook
useEffect(() => {
  const handleKeyDown = (e) => {
    if (e.key === 'n' && (e.metaKey || e.ctrlKey)) {
      e.preventDefault();
      // Focus the new task input
    }
  };

  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, []);
```

---

## Project Recap

### What We Built

A fully functional task manager with:
- CRUD operations (Create, Read, Update, Delete)
- Filtering and searching
- Local storage persistence
- Custom hooks for reusable logic
- Responsive design
- Accessible UI elements

### Concepts Applied

| Concept | Where Used |
|---------|------------|
| Components | Header, TaskForm, TaskList, TaskItem, etc. |
| Props | Passing data and callbacks between components |
| State | Form inputs, tasks array, filter, search |
| useEffect | Local storage sync |
| Custom Hooks | useTasks, useLocalStorage, useDebounce |
| Conditional Rendering | Empty state, edit mode, completed style |
| Lists & Keys | Rendering tasks with unique keys |
| Forms | Task creation and editing |
| Events | Click, change, submit, keyboard events |

---

## Key Takeaways

1. **Plan before coding** - Design your data structure and component hierarchy first
2. **Extract reusable logic** into custom hooks early
3. **Keep components focused** - single responsibility principle
4. **Lift state up** when multiple components need the same data
5. **Use TypeScript** in real projects for better developer experience
6. **Test incrementally** - verify each feature as you build it
7. **Style matters** - a polished UI makes your app feel professional

---

## Congratulations!

You've completed the React Fundamentals module! You now have a solid foundation in:
- Creating and composing React components
- Managing state with useState and useEffect
- Handling user interactions
- Building forms
- Working with lists
- Creating custom hooks

**Next up: Advanced React Patterns** - where you'll learn about Context, Reducers, Performance Optimization, and more!
