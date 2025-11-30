# Days 4-5: State Management Project - Building a Complete Application

## Introduction

In this project, we'll build a complete task management application combining everything we've learned: Zustand for client state, React Query for server state, and proper state architecture patterns.

## Project Overview

**TaskFlow** - A collaborative task management app with:
- User authentication state (Zustand)
- Task CRUD operations (React Query)
- Real-time UI state (Zustand)
- Offline support with optimistic updates
- Proper caching and invalidation

---

## Project Structure

```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── providers.tsx
│   └── (dashboard)/
│       ├── layout.tsx
│       └── page.tsx
├── components/
│   ├── tasks/
│   │   ├── TaskList.tsx
│   │   ├── TaskItem.tsx
│   │   ├── TaskForm.tsx
│   │   └── TaskFilters.tsx
│   └── ui/
│       ├── Modal.tsx
│       ├── Sidebar.tsx
│       └── Toast.tsx
├── hooks/
│   ├── useTasks.ts
│   ├── useProjects.ts
│   └── useAuth.ts
├── stores/
│   ├── uiStore.ts
│   └── authStore.ts
├── lib/
│   ├── api.ts
│   └── utils.ts
└── types/
    └── index.ts
```

---

## Types Definition

```typescript
// types/index.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

export interface Project {
  id: string;
  name: string;
  color: string;
  userId: string;
}

export interface Task {
  id: string;
  title: string;
  description?: string;
  completed: boolean;
  priority: 'low' | 'medium' | 'high';
  dueDate?: string;
  projectId?: string;
  userId: string;
  createdAt: string;
  updatedAt: string;
}

export type TaskFilter = 'all' | 'today' | 'upcoming' | 'completed';
export type SortBy = 'dueDate' | 'priority' | 'createdAt';
```

---

## Client State with Zustand

### Auth Store

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { User } from '@/types';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;

  setAuth: (user: User, token: string) => void;
  logout: () => void;
  updateUser: (updates: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      setAuth: (user, token) =>
        set({
          user,
          token,
          isAuthenticated: true,
        }),

      logout: () =>
        set({
          user: null,
          token: null,
          isAuthenticated: false,
        }),

      updateUser: (updates) =>
        set((state) => ({
          user: state.user ? { ...state.user, ...updates } : null,
        })),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

### UI Store

```typescript
// stores/uiStore.ts
import { create } from 'zustand';
import { TaskFilter, SortBy } from '@/types';

interface UIState {
  // Sidebar
  sidebarOpen: boolean;
  toggleSidebar: () => void;
  setSidebarOpen: (open: boolean) => void;

  // Task filters
  filter: TaskFilter;
  sortBy: SortBy;
  searchQuery: string;
  selectedProjectId: string | null;
  setFilter: (filter: TaskFilter) => void;
  setSortBy: (sortBy: SortBy) => void;
  setSearchQuery: (query: string) => void;
  setSelectedProject: (projectId: string | null) => void;

  // Modals
  isTaskModalOpen: boolean;
  editingTaskId: string | null;
  openTaskModal: (taskId?: string) => void;
  closeTaskModal: () => void;

  // Toast notifications
  toasts: Toast[];
  addToast: (toast: Omit<Toast, 'id'>) => void;
  removeToast: (id: string) => void;
}

interface Toast {
  id: string;
  type: 'success' | 'error' | 'info';
  message: string;
}

export const useUIStore = create<UIState>((set) => ({
  // Sidebar
  sidebarOpen: true,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  setSidebarOpen: (open) => set({ sidebarOpen: open }),

  // Task filters
  filter: 'all',
  sortBy: 'dueDate',
  searchQuery: '',
  selectedProjectId: null,
  setFilter: (filter) => set({ filter }),
  setSortBy: (sortBy) => set({ sortBy }),
  setSearchQuery: (searchQuery) => set({ searchQuery }),
  setSelectedProject: (selectedProjectId) => set({ selectedProjectId }),

  // Modals
  isTaskModalOpen: false,
  editingTaskId: null,
  openTaskModal: (taskId) =>
    set({ isTaskModalOpen: true, editingTaskId: taskId || null }),
  closeTaskModal: () => set({ isTaskModalOpen: false, editingTaskId: null }),

  // Toasts
  toasts: [],
  addToast: (toast) =>
    set((s) => ({
      toasts: [...s.toasts, { ...toast, id: crypto.randomUUID() }],
    })),
  removeToast: (id) =>
    set((s) => ({ toasts: s.toasts.filter((t) => t.id !== id) })),
}));
```

---

## Server State with React Query

### API Functions

```typescript
// lib/api.ts
import { Task, Project } from '@/types';
import { useAuthStore } from '@/stores/authStore';

const API_URL = process.env.NEXT_PUBLIC_API_URL || '/api';

async function fetchWithAuth(url: string, options: RequestInit = {}) {
  const token = useAuthStore.getState().token;

  const response = await fetch(`${API_URL}${url}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      Authorization: token ? `Bearer ${token}` : '',
      ...options.headers,
    },
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new Error(error.message || 'Request failed');
  }

  return response.json();
}

// Tasks API
export const tasksApi = {
  getAll: (): Promise<Task[]> => fetchWithAuth('/tasks'),

  getById: (id: string): Promise<Task> => fetchWithAuth(`/tasks/${id}`),

  create: (data: Partial<Task>): Promise<Task> =>
    fetchWithAuth('/tasks', {
      method: 'POST',
      body: JSON.stringify(data),
    }),

  update: (id: string, data: Partial<Task>): Promise<Task> =>
    fetchWithAuth(`/tasks/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(data),
    }),

  delete: (id: string): Promise<void> =>
    fetchWithAuth(`/tasks/${id}`, { method: 'DELETE' }),
};

// Projects API
export const projectsApi = {
  getAll: (): Promise<Project[]> => fetchWithAuth('/projects'),

  create: (data: Partial<Project>): Promise<Project> =>
    fetchWithAuth('/projects', {
      method: 'POST',
      body: JSON.stringify(data),
    }),

  delete: (id: string): Promise<void> =>
    fetchWithAuth(`/projects/${id}`, { method: 'DELETE' }),
};
```

### Task Hooks

```typescript
// hooks/useTasks.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { tasksApi } from '@/lib/api';
import { Task } from '@/types';
import { useUIStore } from '@/stores/uiStore';

export function useTasks() {
  const { filter, sortBy, searchQuery, selectedProjectId } = useUIStore();

  const query = useQuery({
    queryKey: ['tasks'],
    queryFn: tasksApi.getAll,
    staleTime: 5 * 60 * 1000,
  });

  // Filter and sort tasks
  const filteredTasks = query.data
    ?.filter((task) => {
      // Search filter
      if (searchQuery) {
        const q = searchQuery.toLowerCase();
        if (!task.title.toLowerCase().includes(q)) return false;
      }

      // Project filter
      if (selectedProjectId && task.projectId !== selectedProjectId) {
        return false;
      }

      // Status filter
      switch (filter) {
        case 'today':
          return isToday(task.dueDate);
        case 'upcoming':
          return isFuture(task.dueDate) && !task.completed;
        case 'completed':
          return task.completed;
        default:
          return true;
      }
    })
    .sort((a, b) => {
      switch (sortBy) {
        case 'priority':
          return priorityOrder[b.priority] - priorityOrder[a.priority];
        case 'dueDate':
          return new Date(a.dueDate || '').getTime() -
                 new Date(b.dueDate || '').getTime();
        default:
          return new Date(b.createdAt).getTime() -
                 new Date(a.createdAt).getTime();
      }
    });

  return {
    ...query,
    tasks: filteredTasks || [],
  };
}

const priorityOrder = { low: 0, medium: 1, high: 2 };

export function useTask(id: string) {
  return useQuery({
    queryKey: ['tasks', id],
    queryFn: () => tasksApi.getById(id),
    enabled: !!id,
  });
}

export function useCreateTask() {
  const queryClient = useQueryClient();
  const { addToast, closeTaskModal } = useUIStore();

  return useMutation({
    mutationFn: tasksApi.create,

    onMutate: async (newTask) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previousTasks = queryClient.getQueryData(['tasks']);

      const optimisticTask: Task = {
        id: `temp-${Date.now()}`,
        title: newTask.title || '',
        completed: false,
        priority: newTask.priority || 'medium',
        userId: '',
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        ...newTask,
      };

      queryClient.setQueryData(['tasks'], (old: Task[] = []) => [
        optimisticTask,
        ...old,
      ]);

      return { previousTasks };
    },

    onError: (err, newTask, context) => {
      queryClient.setQueryData(['tasks'], context?.previousTasks);
      addToast({ type: 'error', message: 'Failed to create task' });
    },

    onSuccess: () => {
      addToast({ type: 'success', message: 'Task created!' });
      closeTaskModal();
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}

export function useUpdateTask() {
  const queryClient = useQueryClient();
  const { addToast } = useUIStore();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Task> }) =>
      tasksApi.update(id, data),

    onMutate: async ({ id, data }) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previousTasks = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[] = []) =>
        old.map((task) => (task.id === id ? { ...task, ...data } : task))
      );

      return { previousTasks };
    },

    onError: (err, variables, context) => {
      queryClient.setQueryData(['tasks'], context?.previousTasks);
      addToast({ type: 'error', message: 'Failed to update task' });
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}

export function useDeleteTask() {
  const queryClient = useQueryClient();
  const { addToast } = useUIStore();

  return useMutation({
    mutationFn: tasksApi.delete,

    onMutate: async (deletedId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previousTasks = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[] = []) =>
        old.filter((task) => task.id !== deletedId)
      );

      return { previousTasks };
    },

    onError: (err, deletedId, context) => {
      queryClient.setQueryData(['tasks'], context?.previousTasks);
      addToast({ type: 'error', message: 'Failed to delete task' });
    },

    onSuccess: () => {
      addToast({ type: 'success', message: 'Task deleted' });
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}

export function useToggleTask() {
  const updateTask = useUpdateTask();

  return (task: Task) => {
    updateTask.mutate({
      id: task.id,
      data: { completed: !task.completed },
    });
  };
}
```

---

## Components

### TaskList Component

```tsx
// components/tasks/TaskList.tsx
'use client';

import { useTasks, useToggleTask, useDeleteTask } from '@/hooks/useTasks';
import { TaskItem } from './TaskItem';
import { TaskFilters } from './TaskFilters';
import { useUIStore } from '@/stores/uiStore';

export function TaskList() {
  const { tasks, isLoading, error } = useTasks();
  const toggleTask = useToggleTask();
  const deleteTask = useDeleteTask();
  const { openTaskModal } = useUIStore();

  if (isLoading) {
    return <TaskListSkeleton />;
  }

  if (error) {
    return (
      <div className="text-center py-10">
        <p className="text-red-500">Error loading tasks</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Tasks</h1>
        <button
          onClick={() => openTaskModal()}
          className="btn btn-primary"
        >
          Add Task
        </button>
      </div>

      <TaskFilters />

      {tasks.length === 0 ? (
        <div className="text-center py-10 text-gray-500">
          <p>No tasks found</p>
          <button
            onClick={() => openTaskModal()}
            className="text-blue-500 mt-2"
          >
            Create your first task
          </button>
        </div>
      ) : (
        <ul className="space-y-2">
          {tasks.map((task) => (
            <TaskItem
              key={task.id}
              task={task}
              onToggle={() => toggleTask(task)}
              onEdit={() => openTaskModal(task.id)}
              onDelete={() => deleteTask.mutate(task.id)}
            />
          ))}
        </ul>
      )}
    </div>
  );
}
```

### TaskItem Component

```tsx
// components/tasks/TaskItem.tsx
'use client';

import { Task } from '@/types';
import { formatDate, cn } from '@/lib/utils';

interface TaskItemProps {
  task: Task;
  onToggle: () => void;
  onEdit: () => void;
  onDelete: () => void;
}

export function TaskItem({ task, onToggle, onEdit, onDelete }: TaskItemProps) {
  const priorityColors = {
    low: 'border-l-green-500',
    medium: 'border-l-yellow-500',
    high: 'border-l-red-500',
  };

  return (
    <li
      className={cn(
        'flex items-center gap-4 p-4 bg-white rounded-lg shadow-sm',
        'border-l-4',
        priorityColors[task.priority],
        task.completed && 'opacity-60'
      )}
    >
      <input
        type="checkbox"
        checked={task.completed}
        onChange={onToggle}
        className="w-5 h-5 rounded"
      />

      <div className="flex-1 min-w-0">
        <p
          className={cn(
            'font-medium truncate',
            task.completed && 'line-through text-gray-500'
          )}
        >
          {task.title}
        </p>
        {task.dueDate && (
          <p className="text-sm text-gray-500">
            Due: {formatDate(task.dueDate)}
          </p>
        )}
      </div>

      <div className="flex gap-2">
        <button
          onClick={onEdit}
          className="p-2 text-gray-500 hover:text-blue-500"
        >
          Edit
        </button>
        <button
          onClick={onDelete}
          className="p-2 text-gray-500 hover:text-red-500"
        >
          Delete
        </button>
      </div>
    </li>
  );
}
```

### TaskForm Component

```tsx
// components/tasks/TaskForm.tsx
'use client';

import { useState, useEffect } from 'react';
import { useCreateTask, useUpdateTask, useTask } from '@/hooks/useTasks';
import { useUIStore } from '@/stores/uiStore';
import { useProjects } from '@/hooks/useProjects';

export function TaskForm() {
  const { editingTaskId, closeTaskModal } = useUIStore();
  const { data: existingTask } = useTask(editingTaskId || '');
  const { data: projects } = useProjects();
  const createTask = useCreateTask();
  const updateTask = useUpdateTask();

  const [formData, setFormData] = useState({
    title: '',
    description: '',
    priority: 'medium' as const,
    dueDate: '',
    projectId: '',
  });

  useEffect(() => {
    if (existingTask) {
      setFormData({
        title: existingTask.title,
        description: existingTask.description || '',
        priority: existingTask.priority,
        dueDate: existingTask.dueDate || '',
        projectId: existingTask.projectId || '',
      });
    }
  }, [existingTask]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    if (editingTaskId) {
      updateTask.mutate({
        id: editingTaskId,
        data: formData,
      });
    } else {
      createTask.mutate(formData);
    }
  };

  const isSubmitting = createTask.isPending || updateTask.isPending;

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Title</label>
        <input
          type="text"
          value={formData.title}
          onChange={(e) => setFormData({ ...formData, title: e.target.value })}
          className="w-full p-2 border rounded"
          required
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Description</label>
        <textarea
          value={formData.description}
          onChange={(e) =>
            setFormData({ ...formData, description: e.target.value })
          }
          className="w-full p-2 border rounded"
          rows={3}
        />
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium mb-1">Priority</label>
          <select
            value={formData.priority}
            onChange={(e) =>
              setFormData({
                ...formData,
                priority: e.target.value as 'low' | 'medium' | 'high',
              })
            }
            className="w-full p-2 border rounded"
          >
            <option value="low">Low</option>
            <option value="medium">Medium</option>
            <option value="high">High</option>
          </select>
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">Due Date</label>
          <input
            type="date"
            value={formData.dueDate}
            onChange={(e) =>
              setFormData({ ...formData, dueDate: e.target.value })
            }
            className="w-full p-2 border rounded"
          />
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Project</label>
        <select
          value={formData.projectId}
          onChange={(e) =>
            setFormData({ ...formData, projectId: e.target.value })
          }
          className="w-full p-2 border rounded"
        >
          <option value="">No Project</option>
          {projects?.map((project) => (
            <option key={project.id} value={project.id}>
              {project.name}
            </option>
          ))}
        </select>
      </div>

      <div className="flex gap-2 justify-end">
        <button
          type="button"
          onClick={closeTaskModal}
          className="px-4 py-2 border rounded"
        >
          Cancel
        </button>
        <button
          type="submit"
          disabled={isSubmitting}
          className="px-4 py-2 bg-blue-500 text-white rounded"
        >
          {isSubmitting
            ? 'Saving...'
            : editingTaskId
            ? 'Update Task'
            : 'Create Task'}
        </button>
      </div>
    </form>
  );
}
```

### TaskFilters Component

```tsx
// components/tasks/TaskFilters.tsx
'use client';

import { useUIStore } from '@/stores/uiStore';
import { TaskFilter, SortBy } from '@/types';

export function TaskFilters() {
  const { filter, sortBy, searchQuery, setFilter, setSortBy, setSearchQuery } =
    useUIStore();

  const filters: { value: TaskFilter; label: string }[] = [
    { value: 'all', label: 'All' },
    { value: 'today', label: 'Today' },
    { value: 'upcoming', label: 'Upcoming' },
    { value: 'completed', label: 'Completed' },
  ];

  return (
    <div className="flex flex-wrap gap-4 mb-6">
      <input
        type="search"
        placeholder="Search tasks..."
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        className="flex-1 min-w-[200px] p-2 border rounded"
      />

      <div className="flex gap-1 bg-gray-100 p-1 rounded">
        {filters.map((f) => (
          <button
            key={f.value}
            onClick={() => setFilter(f.value)}
            className={`px-3 py-1 rounded ${
              filter === f.value
                ? 'bg-white shadow'
                : 'text-gray-600 hover:bg-gray-200'
            }`}
          >
            {f.label}
          </button>
        ))}
      </div>

      <select
        value={sortBy}
        onChange={(e) => setSortBy(e.target.value as SortBy)}
        className="p-2 border rounded"
      >
        <option value="dueDate">Sort by Due Date</option>
        <option value="priority">Sort by Priority</option>
        <option value="createdAt">Sort by Created</option>
      </select>
    </div>
  );
}
```

---

## Main Dashboard Page

```tsx
// app/(dashboard)/page.tsx
'use client';

import { TaskList } from '@/components/tasks/TaskList';
import { TaskForm } from '@/components/tasks/TaskForm';
import { Modal } from '@/components/ui/Modal';
import { Toasts } from '@/components/ui/Toasts';
import { useUIStore } from '@/stores/uiStore';

export default function DashboardPage() {
  const { isTaskModalOpen, closeTaskModal, editingTaskId } = useUIStore();

  return (
    <div className="container mx-auto px-4 py-8">
      <TaskList />

      <Modal
        isOpen={isTaskModalOpen}
        onClose={closeTaskModal}
        title={editingTaskId ? 'Edit Task' : 'New Task'}
      >
        <TaskForm />
      </Modal>

      <Toasts />
    </div>
  );
}
```

---

## Key Takeaways

1. **Separate client and server state** - Zustand for UI, React Query for API data
2. **Optimistic updates** improve perceived performance
3. **Custom hooks** encapsulate complex logic
4. **Proper typing** prevents bugs and improves DX
5. **Query invalidation** keeps data fresh
6. **Toast notifications** provide user feedback
7. **Filter and sort in hooks** keeps components simple

---

## Congratulations!

You've completed the State Management module! You now understand how to:
- Choose the right state management solution
- Build stores with Zustand
- Manage server state with React Query
- Combine both for a complete application

**Next up: Frontend Testing!**
