# Advanced State Management

> Advanced / Senior Level (3-5 years)

---

## Questions

111. How do you implement optimistic updates?
112. What is TanStack Query (React Query) and its benefits?
113. What is SWR library and how does it work?
114. How do you handle caching in React applications?
115. What is RTK Query and how does it compare to React Query?
116. How do you implement real-time updates with WebSockets?
117. What is the difference between local and global state?
118. How do you handle state synchronization across tabs?
119. What is Redux Saga and when do you use it?
120. What is Redux Thunk vs Redux Saga?

---

## Detailed Answers

### 111. How do you implement optimistic updates?

<details>
<summary>View Answer</summary>

**Optimistic Updates** is a pattern where the UI is updated immediately (optimistically) before the server confirms the change, providing instant feedback to users. If the server request fails, the change is rolled back.

#### What are Optimistic Updates?

Optimistic updates:
- **Update UI immediately** (don't wait for server)
- **Assume success** (optimistic assumption)
- **Better UX** - instant feedback, feels faster
- **Rollback on failure** - revert if server rejects
- **Show pending state** (optional)

**Flow:**
```
1. User action (e.g., click "Like")
2. Update UI immediately (show liked)
3. Send request to server
4. If success: Keep UI as is
5. If failure: Rollback UI (show unliked)
```

---

#### Basic Example: Like Button

```tsx
import { useState } from 'react';

interface Post {
  id: string;
  likes: number;
  isLiked: boolean;
}

function LikeButton({ post }: { post: Post }) {
  const [likes, setLikes] = useState(post.likes);
  const [isLiked, setIsLiked] = useState(post.isLiked);
  const [isLoading, setIsLoading] = useState(false);

  const handleLike = async () => {
    // Store previous state for rollback
    const previousLikes = likes;
    const previousIsLiked = isLiked;

    // 1. Optimistic update - immediate UI change
    setIsLiked(!isLiked);
    setLikes(isLiked ? likes - 1 : likes + 1);
    setIsLoading(true);

    try {
      // 2. Send request to server
      const response = await fetch(`/api/posts/${post.id}/like`, {
        method: 'POST',
        body: JSON.stringify({ liked: !isLiked }),
      });

      if (!response.ok) throw new Error('Failed to update');

      // 3. Success - keep optimistic state
      const data = await response.json();
      setLikes(data.likes);
    } catch (error) {
      // 4. Failure - rollback to previous state
      setIsLiked(previousIsLiked);
      setLikes(previousLikes);
      console.error('Failed to like post:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <button onClick={handleLike} disabled={isLoading}>
      {isLiked ? '❤️' : '🤍'} {likes}
      {isLoading && ' ...'}
    </button>
  );
}
```

---

#### Advanced Example: Todo List

```tsx
import { useState } from 'react';
import { v4 as uuidv4 } from 'uuid';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
  isPending?: boolean;  // Optimistic state
}

function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);

  // Add todo optimistically
  const addTodo = async (text: string) => {
    const tempId = uuidv4();
    const optimisticTodo: Todo = {
      id: tempId,
      text,
      completed: false,
      isPending: true,  // Mark as pending
    };

    // 1. Add to UI immediately
    setTodos((prev) => [...prev, optimisticTodo]);

    try {
      // 2. Send to server
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text }),
      });

      if (!response.ok) throw new Error('Failed to create todo');

      const serverTodo = await response.json();

      // 3. Replace temp todo with server todo
      setTodos((prev) =>
        prev.map((todo) =>
          todo.id === tempId
            ? { ...serverTodo, isPending: false }
            : todo
        )
      );
    } catch (error) {
      // 4. Remove optimistic todo on failure
      setTodos((prev) => prev.filter((todo) => todo.id !== tempId));
      alert('Failed to add todo');
    }
  };

  // Delete todo optimistically
  const deleteTodo = async (id: string) => {
    // Store for rollback
    const todoToDelete = todos.find((todo) => todo.id === id);
    if (!todoToDelete) return;

    // 1. Remove from UI immediately
    setTodos((prev) => prev.filter((todo) => todo.id !== id));

    try {
      // 2. Send delete request
      const response = await fetch(`/api/todos/${id}`, {
        method: 'DELETE',
      });

      if (!response.ok) throw new Error('Failed to delete');

      // 3. Success - already removed from UI
    } catch (error) {
      // 4. Rollback - restore deleted todo
      setTodos((prev) => [...prev, todoToDelete]);
      alert('Failed to delete todo');
    }
  };

  // Toggle todo optimistically
  const toggleTodo = async (id: string) => {
    // Store previous state
    const previousTodos = [...todos];

    // 1. Toggle immediately
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id
          ? { ...todo, completed: !todo.completed, isPending: true }
          : todo
      )
    );

    try {
      // 2. Send update
      const todo = todos.find((t) => t.id === id);
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todo?.completed }),
      });

      if (!response.ok) throw new Error('Failed to update');

      // 3. Remove pending state
      setTodos((prev) =>
        prev.map((todo) =>
          todo.id === id ? { ...todo, isPending: false } : todo
        )
      );
    } catch (error) {
      // 4. Rollback to previous state
      setTodos(previousTodos);
      alert('Failed to update todo');
    }
  };

  return (
    <div>
      <button onClick={() => addTodo('New Todo')}>Add Todo</button>
      <ul>
        {todos.map((todo) => (
          <li
            key={todo.id}
            style={{ opacity: todo.isPending ? 0.5 : 1 }}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
              disabled={todo.isPending}
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              disabled={todo.isPending}
            >
              Delete
            </button>
            {todo.isPending && <span> (saving...)</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

#### Optimistic Updates with React Query

```tsx
import { useMutation, useQueryClient, useQuery } from '@tanstack/react-query';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

function TodoListWithReactQuery() {
  const queryClient = useQueryClient();

  // Fetch todos
  const { data: todos = [] } = useQuery<Todo[]>({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then((res) => res.json()),
  });

  // Mutation with optimistic update
  const toggleMutation = useMutation({
    mutationFn: async ({ id, completed }: { id: string; completed: boolean }) => {
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed }),
      });
      return response.json();
    },
    // Optimistic update
    onMutate: async ({ id, completed }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot previous value
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);

      // Optimistically update
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((todo) =>
          todo.id === id ? { ...todo, completed } : todo
        ) || []
      );

      // Return context with snapshot
      return { previousTodos };
    },
    // If mutation fails, rollback
    onError: (err, variables, context) => {
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
    },
    // Always refetch after success or error
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  // Delete mutation with optimistic update
  const deleteMutation = useMutation({
    mutationFn: async (id: string) => {
      const response = await fetch(`/api/todos/${id}`, {
        method: 'DELETE',
      });
      return response.json();
    },
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);

      // Optimistically remove
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.filter((todo) => todo.id !== id) || []
      );

      return { previousTodos };
    },
    onError: (err, variables, context) => {
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() =>
              toggleMutation.mutate({ id: todo.id, completed: !todo.completed })
            }
          />
          <span>{todo.text}</span>
          <button onClick={() => deleteMutation.mutate(todo.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

---

#### Optimistic Updates with useOptimistic (React 19)

```tsx
import { useOptimistic, useState } from 'react';

interface Message {
  id: string;
  text: string;
  sending?: boolean;
}

function MessageThread() {
  const [messages, setMessages] = useState<Message[]>([]);
  
  // useOptimistic hook for optimistic state
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: Message) => [...state, { ...newMessage, sending: true }]
  );

  const sendMessage = async (text: string) => {
    const tempMessage: Message = {
      id: crypto.randomUUID(),
      text,
    };

    // Add optimistic message
    addOptimisticMessage(tempMessage);

    try {
      // Send to server
      const response = await fetch('/api/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text }),
      });

      const serverMessage = await response.json();
      
      // Add real message
      setMessages((prev) => [...prev, serverMessage]);
    } catch (error) {
      console.error('Failed to send message:', error);
      // Optimistic message will be removed automatically
    }
  };

  return (
    <div>
      <ul>
        {optimisticMessages.map((message) => (
          <li key={message.id} style={{ opacity: message.sending ? 0.5 : 1 }}>
            {message.text}
            {message.sending && ' (sending...)'}
          </li>
        ))}
      </ul>
      <button onClick={() => sendMessage('Hello!')}>Send</button>
    </div>
  );
}
```

---

#### Best Practices

**1. Always Store Previous State:**
```tsx
const handleUpdate = async () => {
  // ✅ Store for rollback
  const previousState = currentState;
  
  // Update optimistically
  setState(newState);
  
  try {
    await api.update(newState);
  } catch {
    setState(previousState);  // Rollback
  }
};
```

**2. Show Pending State:**
```tsx
interface Item {
  id: string;
  name: string;
  isPending?: boolean;  // Visual indicator
}

<li style={{ opacity: item.isPending ? 0.5 : 1 }}>
  {item.name}
  {item.isPending && ' (saving...)'}
</li>
```

**3. Cancel Pending Requests:**
```tsx
const handleUpdate = async () => {
  // Cancel previous request if still pending
  abortController?.abort();
  
  const controller = new AbortController();
  
  try {
    await fetch('/api/update', {
      signal: controller.signal,
    });
  } catch (error) {
    if (error.name === 'AbortError') return;
    // Handle error
  }
};
```

**4. Queue Multiple Updates:**
```tsx
const [updateQueue, setUpdateQueue] = useState<Update[]>([]);

const handleUpdate = (update: Update) => {
  // Add to queue
  setUpdateQueue((prev) => [...prev, update]);
  
  // Process queue
  processQueue();
};

const processQueue = async () => {
  while (updateQueue.length > 0) {
    const update = updateQueue[0];
    try {
      await api.update(update);
      setUpdateQueue((prev) => prev.slice(1));
    } catch {
      // Handle error, retry, etc.
      break;
    }
  }
};
```

**5. Use Temporary IDs:**
```tsx
const addItem = async (text: string) => {
  const tempId = `temp-${Date.now()}`;
  
  // Add with temp ID
  setItems((prev) => [...prev, { id: tempId, text }]);
  
  try {
    const response = await api.create({ text });
    
    // Replace temp ID with server ID
    setItems((prev) =>
      prev.map((item) =>
        item.id === tempId ? response : item
      )
    );
  } catch {
    // Remove item with temp ID
    setItems((prev) => prev.filter((item) => item.id !== tempId));
  }
};
```

---

#### When to Use Optimistic Updates

**✅ Good Use Cases:**

1. **Like/Favorite buttons** - instant feedback
2. **Todo items** - check/uncheck, add/delete
3. **Form submissions** - save drafts
4. **Social interactions** - follow, comment
5. **Real-time editing** - collaborative docs
6. **Settings changes** - toggle switches

**❌ When Not to Use:**

1. **Payment transactions** - must confirm
2. **Critical operations** - account deletion
3. **Complex validation** - server-side rules
4. **High failure rate** - unstable network
5. **Security operations** - password changes

---

#### Common Patterns

**Pattern 1: Optimistic + Rollback:**
```tsx
// Update immediately, rollback on error
const [state, setState] = useState(initial);

const update = async (newValue) => {
  const prev = state;
  setState(newValue);
  
  try {
    await api.update(newValue);
  } catch {
    setState(prev);
  }
};
```

**Pattern 2: Optimistic + Confirm:**
```tsx
// Update immediately, replace with server response
const update = async (newValue) => {
  setState(newValue);  // Optimistic
  
  try {
    const response = await api.update(newValue);
    setState(response);  // Replace with server data
  } catch {
    setState(prevValue);
  }
};
```

**Pattern 3: Pending State:**
```tsx
// Show pending indicator
const update = async (newValue) => {
  setState({ ...newValue, pending: true });
  
  try {
    await api.update(newValue);
    setState({ ...newValue, pending: false });
  } catch {
    setState(prevValue);
  }
};
```

---

#### Interview Tips

✅ **Key Points:**
- Optimistic updates = update UI before server confirms
- Provides instant feedback (better UX)
- Must store previous state for rollback
- Rollback if server request fails
- Show pending state for clarity

✅ **When to Mention:**
- Discussing UX improvements
- Performance optimization
- State management patterns
- Real-time features
- React Query/SWR usage

✅ **Common Follow-ups:**
- "How do you handle failures?"
- "What if multiple updates happen?"
- "How does React Query help?"
- "When should you avoid this?"
- "Show me a real example"

✅ **Perfect Answer Structure:**
1. Define: Update UI before server confirms
2. Why: Instant feedback, better UX
3. How: Store previous state, update immediately, rollback on error
4. Example: Like button or todo list
5. Tools: React Query, useOptimistic (React 19)
6. When to use: Social interactions, simple updates
7. When to avoid: Critical operations, payments

</details>

---

### 112. What is TanStack Query (React Query) and its benefits?

<details>
<summary>View Answer</summary>

**TanStack Query** (formerly React Query) is a powerful data-fetching and state management library that handles server state, caching, synchronization, and more with minimal code.

#### What is TanStack Query?

TanStack Query provides:
- **Automatic caching** - intelligent cache management
- **Background refetching** - keep data fresh
- **Automatic retries** - resilient to failures
- **Pagination & infinite scroll** - built-in support
- **Optimistic updates** - instant UI feedback
- **Request deduplication** - avoid duplicate requests
- **Server state management** - separate from client state

---

#### Basic Setup

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// 1. Create a client
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      cacheTime: 5 * 60 * 1000, // 5 minutes
      refetchOnWindowFocus: true,
      retry: 3,
    },
  },
});

// 2. Wrap app with provider
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  );
}
```

---

#### Basic Query (Fetching Data)

```tsx
import { useQuery } from '@tanstack/react-query';

interface User {
  id: string;
  name: string;
  email: string;
}

function UserProfile({ userId }: { userId: string }) {
  const {
    data,
    isLoading,
    isError,
    error,
    refetch,
  } = useQuery<User>({
    queryKey: ['user', userId],  // Unique key for caching
    queryFn: async () => {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error('Failed to fetch user');
      return response.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.email}</p>
      <button onClick={() => refetch()}>Refresh</button>
    </div>
  );
}
```

---

#### Mutations (Updating Data)

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

interface CreateUserData {
  name: string;
  email: string;
}

function CreateUserForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (userData: CreateUserData) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });
      return response.json();
    },
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
      {mutation.isError && <div>Error: {mutation.error.message}</div>}
      {mutation.isSuccess && <div>User created!</div>}
    </form>
  );
}
```

---

#### Real-World Example: Todo App

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

// Fetch todos
function useTodos() {
  return useQuery<Todo[]>({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('/api/todos');
      return response.json();
    },
  });
}

// Add todo mutation
function useAddTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (text: string) => {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text }),
      });
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}

// Toggle todo mutation
function useToggleTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ id, completed }: { id: string; completed: boolean }) => {
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed }),
      });
      return response.json();
    },
    // Optimistic update
    onMutate: async ({ id, completed }) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);

      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((todo) =>
          todo.id === id ? { ...todo, completed } : todo
        ) || []
      );

      return { previousTodos };
    },
    onError: (err, variables, context) => {
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}

// Delete todo mutation
function useDeleteTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (id: string) => {
      await fetch(`/api/todos/${id}`, { method: 'DELETE' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}

// Component
function TodoApp() {
  const { data: todos, isLoading } = useTodos();
  const addTodo = useAddTodo();
  const toggleTodo = useToggleTodo();
  const deleteTodo = useDeleteTodo();

  const handleAdd = () => {
    const text = prompt('Enter todo:');
    if (text) addTodo.mutate(text);
  };

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <button onClick={handleAdd}>Add Todo</button>
      <ul>
        {todos?.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() =>
                toggleTodo.mutate({ id: todo.id, completed: !todo.completed })
              }
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo.mutate(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

#### Advanced Features

**1. Pagination:**
```tsx
function UserList() {
  const [page, setPage] = useState(1);

  const { data, isLoading, isPlaceholderData } = useQuery({
    queryKey: ['users', page],
    queryFn: () => fetch(`/api/users?page=${page}`).then((res) => res.json()),
    placeholderData: (previousData) => previousData, // Keep previous data while fetching
  });

  return (
    <div>
      <ul>
        {data?.users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      <button onClick={() => setPage((p) => Math.max(1, p - 1))}>Previous</button>
      <button onClick={() => setPage((p) => p + 1)}>Next</button>
      {isPlaceholderData && <span> Loading...</span>}
    </div>
  );
}
```

**2. Infinite Scroll:**
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

function InfiniteUserList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['users'],
    queryFn: ({ pageParam = 1 }) =>
      fetch(`/api/users?page=${pageParam}`).then((res) => res.json()),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
    initialPageParam: 1,
  });

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.users.map((user) => (
            <div key={user.id}>{user.name}</div>
          ))}
        </div>
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading more...' : 'Load More'}
      </button>
    </div>
  );
}
```

**3. Dependent Queries:**
```tsx
function UserDetails({ userId }: { userId: string }) {
  // First query
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then((res) => res.json()),
  });

  // Second query depends on first
  const { data: posts } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetch(`/api/posts?userId=${user.id}`).then((res) => res.json()),
    enabled: !!user, // Only run if user exists
  });

  return (
    <div>
      <h2>{user?.name}</h2>
      <h3>Posts:</h3>
      {posts?.map((post) => <div key={post.id}>{post.title}</div>)}
    </div>
  );
}
```

**4. Parallel Queries:**
```tsx
function Dashboard() {
  const users = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
  const posts = useQuery({ queryKey: ['posts'], queryFn: fetchPosts });
  const stats = useQuery({ queryKey: ['stats'], queryFn: fetchStats });

  if (users.isLoading || posts.isLoading || stats.isLoading) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <div>Users: {users.data?.length}</div>
      <div>Posts: {posts.data?.length}</div>
      <div>Views: {stats.data?.views}</div>
    </div>
  );
}

// Or use useQueries for dynamic parallel queries
import { useQueries } from '@tanstack/react-query';

function MultiUserProfiles({ userIds }: { userIds: string[] }) {
  const results = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['user', id],
      queryFn: () => fetch(`/api/users/${id}`).then((res) => res.json()),
    })),
  });

  return (
    <div>
      {results.map((result, i) => (
        <div key={userIds[i]}>
          {result.isLoading ? 'Loading...' : result.data?.name}
        </div>
      ))}
    </div>
  );
}
```

---

#### Benefits of TanStack Query

**1. Automatic Caching:**
```tsx
// Data is cached automatically by query key
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });

// Same query elsewhere - uses cache!
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
```

**2. Background Refetching:**
```tsx
// Automatically refetches when:
// - Window refocuses
// - Network reconnects
// - Configured interval

const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  refetchOnWindowFocus: true,
  refetchOnReconnect: true,
  refetchInterval: 30000, // Every 30 seconds
});
```

**3. Request Deduplication:**
```tsx
// Multiple components request same data
function Component1() {
  const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
}

function Component2() {
  const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
}

// Only ONE request is made!
```

**4. Stale While Revalidate:**
```tsx
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  staleTime: 60000, // Data is fresh for 1 minute
});

// Shows cached data immediately, refetches in background if stale
```

**5. Built-in Retry Logic:**
```tsx
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  retry: 3, // Retry 3 times on failure
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
});
```

---

#### Comparison: Before vs After

**Before (Manual State Management):**
```tsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err);
        setLoading(false);
      });
  }, []);

  // Manual refetching
  const refetch = () => {
    setLoading(true);
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setLoading(false);
      });
  };

  // No caching, no deduplication, no background refresh
  // Need to manage all state manually

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {users.map((user) => <div key={user.id}>{user.name}</div>)}
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}
```

**After (TanStack Query):**
```tsx
function UserList() {
  const { data: users, isLoading, error, refetch } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
  });

  // Automatic caching, deduplication, background refresh
  // Automatic retries, refetch on window focus
  // All handled by React Query!

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {users.map((user) => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => refetch()}>Refresh</button>
    </div>
  );
}
```

---

#### Best Practices

**1. Use Query Keys Wisely:**
```tsx
// ✅ Good - hierarchical keys
queryKey: ['users']                    // All users
queryKey: ['users', userId]            // Specific user
queryKey: ['users', userId, 'posts']   // User's posts

// ❌ Bad - flat keys
queryKey: ['allUsers']
queryKey: ['userById123']
queryKey: ['postsForUser123']
```

**2. Separate Client and Server State:**
```tsx
// ✅ Server state - use React Query
const { data: users } = useQuery({ queryKey: ['users'], ... });

// ✅ Client state - use useState
const [isModalOpen, setIsModalOpen] = useState(false);
```

**3. Handle Loading and Error States:**
```tsx
const { data, isLoading, isError, error } = useQuery(...);

if (isLoading) return <Spinner />;
if (isError) return <ErrorMessage error={error} />;
if (!data) return <Empty />;

return <DataDisplay data={data} />;
```

**4. Use DevTools:**
```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

---

#### Interview Tips

✅ **Key Points:**
- TanStack Query = powerful data fetching library
- Automatic caching, background refetching, retries
- Separates server state from client state
- Reduces boilerplate code significantly
- Built-in support for pagination, infinite scroll, optimistic updates

✅ **When to Mention:**
- Discussing state management
- Data fetching strategies
- Performance optimization
- Caching strategies
- Real-world app architecture

✅ **Common Follow-ups:**
- "How does caching work?"
- "What's the difference vs Redux?"
- "How do you handle mutations?"
- "What are query keys?"
- "How does it compare to SWR?"

✅ **Perfect Answer Structure:**
1. Define: Data fetching & caching library
2. Key features: Automatic caching, refetching, retries
3. Benefits: Less code, better UX, automatic optimizations
4. Example: Basic query and mutation
5. Advanced: Pagination, infinite scroll, optimistic updates
6. When to use: Server state management

</details>

---

### 113. What is SWR library and how does it work?

<details>
<summary>View Answer</summary>

**SWR (Stale-While-Revalidate)** is a React Hooks library for data fetching created by Vercel. It implements the stale-while-revalidate caching strategy, providing instant page loads with cached data while revalidating in the background.

#### What is SWR?

SWR stands for **Stale-While-Revalidate**:
- **Stale** - Return cached data immediately (may be outdated)
- **While** - At the same time
- **Revalidate** - Fetch fresh data in background

Features:
- **Fast page loads** - instant cached data
- **Automatic revalidation** - keeps data fresh
- **Built-in cache** - smart caching
- **Real-time** - WebSocket/polling support
- **Pagination** - built-in
- **Lightweight** - smaller than React Query

---

#### Basic Setup

```bash
npm install swr
```

```tsx
import useSWR from 'swr';

// Fetcher function
const fetcher = (url: string) => fetch(url).then((res) => res.json());

// Basic usage
function UserProfile({ userId }: { userId: string }) {
  const { data, error, isLoading } = useSWR(
    `/api/users/${userId}`,
    fetcher
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.email}</p>
    </div>
  );
}
```

---

#### How SWR Works

**The Stale-While-Revalidate Strategy:**

```
1. User requests data
2. SWR checks cache
3. If cached: Return stale data immediately (fast!)
4. Then: Fetch fresh data in background
5. When fresh data arrives: Update UI
6. If no cache: Fetch and show loading state
```

**Visual Flow:**
```tsx
function Example() {
  const { data } = useSWR('/api/data', fetcher);

  // First render: data = undefined (loading)
  // Second render: data = cached (stale but instant)
  // Third render: data = fresh (revalidated)

  return <div>{data?.value}</div>;
}
```

---

#### Global Configuration

```tsx
import { SWRConfig } from 'swr';

function App() {
  return (
    <SWRConfig
      value={{
        fetcher: (url: string) => fetch(url).then((res) => res.json()),
        refreshInterval: 3000, // Refresh every 3 seconds
        revalidateOnFocus: true, // Revalidate when window regains focus
        revalidateOnReconnect: true, // Revalidate on reconnect
        dedupingInterval: 2000, // Dedupe requests within 2 seconds
      }}
    >
      <YourApp />
    </SWRConfig>
  );
}
```

---

#### Real-World Example: Todo App

```tsx
import useSWR, { mutate } from 'swr';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

const fetcher = (url: string) => fetch(url).then((res) => res.json());

// Custom hook for todos
function useTodos() {
  const { data, error, isLoading, mutate } = useSWR<Todo[]>(
    '/api/todos',
    fetcher
  );

  return {
    todos: data,
    isLoading,
    isError: error,
    mutate,
  };
}

function TodoApp() {
  const { todos, isLoading, mutate } = useTodos();

  const addTodo = async (text: string) => {
    // Optimistic update
    const newTodo = { id: Date.now().toString(), text, completed: false };
    mutate([...todos!, newTodo], false); // Update without revalidation

    // Send to server
    await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text }),
    });

    // Revalidate
    mutate();
  };

  const toggleTodo = async (id: string) => {
    const todo = todos?.find((t) => t.id === id);
    if (!todo) return;

    // Optimistic update
    mutate(
      todos?.map((t) =>
        t.id === id ? { ...t, completed: !t.completed } : t
      ),
      false
    );

    // Send to server
    await fetch(`/api/todos/${id}`, {
      method: 'PATCH',
      body: JSON.stringify({ completed: !todo.completed }),
    });

    // Revalidate
    mutate();
  };

  const deleteTodo = async (id: string) => {
    // Optimistic update
    mutate(
      todos?.filter((t) => t.id !== id),
      false
    );

    // Send to server
    await fetch(`/api/todos/${id}`, { method: 'DELETE' });

    // Revalidate
    mutate();
  };

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <button onClick={() => addTodo('New Todo')}>Add Todo</button>
      <ul>
        {todos?.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

#### Advanced Features

**1. Conditional Fetching:**
```tsx
function UserProfile({ userId }: { userId: string | null }) {
  // Only fetch if userId exists
  const { data } = useSWR(
    userId ? `/api/users/${userId}` : null,
    fetcher
  );

  return <div>{data?.name}</div>;
}
```

**2. Dependent Requests:**
```tsx
function UserPosts({ userId }: { userId: string }) {
  // First request
  const { data: user } = useSWR(`/api/users/${userId}`, fetcher);

  // Second request depends on first
  const { data: posts } = useSWR(
    user ? `/api/posts?userId=${user.id}` : null,
    fetcher
  );

  return (
    <div>
      <h2>{user?.name}</h2>
      {posts?.map((post) => <div key={post.id}>{post.title}</div>)}
    </div>
  );
}
```

**3. Pagination:**
```tsx
import useSWR from 'swr';

function UserList() {
  const [page, setPage] = useState(1);

  const { data, error } = useSWR(
    `/api/users?page=${page}`,
    fetcher,
    {
      keepPreviousData: true, // Keep previous page while loading
    }
  );

  return (
    <div>
      {data?.users.map((user) => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => setPage(page - 1)}>Previous</button>
      <button onClick={() => setPage(page + 1)}>Next</button>
    </div>
  );
}
```

**4. Infinite Loading:**
```tsx
import useSWRInfinite from 'swr/infinite';

function InfiniteUserList() {
  const getKey = (pageIndex: number, previousPageData: any) => {
    // Reached the end
    if (previousPageData && !previousPageData.hasMore) return null;

    // First page
    return `/api/users?page=${pageIndex + 1}`;
  };

  const { data, size, setSize, isLoading } = useSWRInfinite(
    getKey,
    fetcher
  );

  const users = data ? data.flatMap((page) => page.users) : [];
  const hasMore = data?.[data.length - 1]?.hasMore;

  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
      {hasMore && (
        <button onClick={() => setSize(size + 1)}>
          {isLoading ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

**5. Mutation and Revalidation:**
```tsx
import { mutate } from 'swr';

// Revalidate specific key
mutate('/api/users');

// Revalidate with new data
mutate('/api/users', newData);

// Revalidate without refetch
mutate('/api/users', newData, false);

// Revalidate all keys matching pattern
mutate(
  (key) => typeof key === 'string' && key.startsWith('/api/users'),
);
```

**6. Optimistic Updates:**
```tsx
function LikeButton({ postId }: { postId: string }) {
  const { data: post, mutate } = useSWR(`/api/posts/${postId}`, fetcher);

  const handleLike = async () => {
    // Optimistic update
    mutate(
      { ...post, likes: post.likes + 1, isLiked: true },
      false // Don't revalidate immediately
    );

    try {
      // Send to server
      await fetch(`/api/posts/${postId}/like`, { method: 'POST' });

      // Revalidate to get accurate data
      mutate();
    } catch (error) {
      // Rollback on error
      mutate();
    }
  };

  return (
    <button onClick={handleLike}>
      {post?.isLiked ? '❤️' : '🤍'} {post?.likes}
    </button>
  );
}
```

**7. Prefetching:**
```tsx
import { mutate } from 'swr';

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher);

  // Prefetch user data on hover
  const prefetchUser = (userId: string) => {
    mutate(
      `/api/users/${userId}`,
      fetch(`/api/users/${userId}`).then((res) => res.json())
    );
  };

  return (
    <ul>
      {users?.map((user) => (
        <li key={user.id} onMouseEnter={() => prefetchUser(user.id)}>
          <Link to={`/users/${user.id}`}>{user.name}</Link>
        </li>
      ))}
    </ul>
  );
}
```

---

#### SWR vs React Query

| Feature | SWR | React Query |
|---------|-----|-------------|
| **Size** | Smaller (~5KB) | Larger (~13KB) |
| **API** | Simpler | More comprehensive |
| **Caching** | Built-in | Built-in |
| **Devtools** | No | Yes |
| **Mutations** | Manual | Built-in useMutation |
| **Infinite Scroll** | useSWRInfinite | useInfiniteQuery |
| **Learning Curve** | Easier | Steeper |
| **Use Case** | Simple apps, Next.js | Complex apps |

---

#### Real-World: Live Data

```tsx
import useSWR from 'swr';

function LiveStockPrice({ symbol }: { symbol: string }) {
  const { data } = useSWR(
    `/api/stock/${symbol}`,
    fetcher,
    {
      refreshInterval: 1000, // Refresh every second
      dedupingInterval: 500, // Dedupe within 500ms
    }
  );

  return (
    <div>
      <h2>{symbol}</h2>
      <p>Price: ${data?.price}</p>
      <p style={{ color: data?.change > 0 ? 'green' : 'red' }}>
        Change: {data?.change}%
      </p>
    </div>
  );
}
```

---

#### Best Practices

**1. Use Global Fetcher:**
```tsx
// ✅ Good - global fetcher
<SWRConfig value={{ fetcher: globalFetcher }}>
  <App />
</SWRConfig>

// Components don't need to specify fetcher
const { data } = useSWR('/api/users');
```

**2. Error Handling:**
```tsx
const { data, error } = useSWR('/api/users', fetcher, {
  onError: (err) => {
    console.error('SWR Error:', err);
    // Send to error tracking service
  },
});

if (error) return <ErrorBoundary error={error} />;
```

**3. Type Safety:**
```tsx
interface User {
  id: string;
  name: string;
}

const { data } = useSWR<User[]>('/api/users', fetcher);
// data is typed as User[] | undefined
```

**4. Dedupe Requests:**
```tsx
// Multiple components use same key
function Component1() {
  const { data } = useSWR('/api/users', fetcher);
}

function Component2() {
  const { data } = useSWR('/api/users', fetcher);
}

// Only ONE request is made (within dedupingInterval)
```

---

#### When to Use SWR

**✅ Use SWR When:**

1. **Building with Next.js** - created by Vercel
2. **Simple data fetching** - straightforward use cases
3. **Real-time data** - polling, live updates
4. **Want lightweight** - smaller bundle size
5. **Prefer simplicity** - easier API

**✅ Use React Query When:**

1. **Complex mutations** - need useMutation
2. **Advanced caching** - more control
3. **DevTools needed** - debugging
4. **Large applications** - comprehensive features
5. **Not using Next.js** - framework agnostic

---

#### Interview Tips

✅ **Key Points:**
- SWR = Stale-While-Revalidate strategy
- Returns cached data instantly, revalidates in background
- Created by Vercel, great with Next.js
- Smaller and simpler than React Query
- Built-in caching, revalidation, polling

✅ **When to Mention:**
- Discussing Next.js projects
- Data fetching strategies
- Caching patterns
- Performance optimization
- Comparing state management solutions

✅ **Common Follow-ups:**
- "How is it different from React Query?"
- "What does stale-while-revalidate mean?"
- "How do you handle mutations?"
- "When would you use SWR vs React Query?"
- "How does caching work?"

✅ **Perfect Answer Structure:**
1. Define: Stale-While-Revalidate = return cache, refetch in background
2. Benefits: Fast loads, automatic revalidation, simple API
3. Example: Basic usage with useSWR
4. Features: Polling, pagination, optimistic updates
5. vs React Query: Lighter, simpler, good for Next.js
6. When to use: Simple apps, real-time data, Next.js projects

</details>

---

### 114. How do you handle caching in React applications?

<details>
<summary>View Answer</summary>

**Caching** in React applications stores data temporarily to avoid unnecessary network requests, improve performance, and provide instant data access. There are multiple caching strategies and levels.

#### Types of Caching in React

**1. Memory Cache** - Store data in component state or global state
**2. Browser Cache** - Use browser APIs (localStorage, IndexedDB)
**3. HTTP Cache** - Leverage browser HTTP caching
**4. Service Worker Cache** - Offline-first caching
**5. Library Cache** - React Query, SWR, etc.

---

#### 1. Memory Cache with useState

```tsx
import { useState, useEffect } from 'react';

interface User {
  id: string;
  name: string;
}

// Simple in-memory cache
const cache = new Map<string, any>();

function useUserWithCache(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const cacheKey = `user-${userId}`;

    // Check cache first
    if (cache.has(cacheKey)) {
      setUser(cache.get(cacheKey));
      setLoading(false);
      return;
    }

    // Fetch if not cached
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        cache.set(cacheKey, data); // Store in cache
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  return { user, loading };
}
```

---

#### 2. Cache with Expiration

```tsx
import { useState, useEffect } from 'react';

interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

class CacheManager {
  private cache = new Map<string, CacheEntry<any>>();
  private ttl: number; // Time to live in milliseconds

  constructor(ttl: number = 5 * 60 * 1000) {
    // Default 5 minutes
    this.ttl = ttl;
  }

  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    if (!entry) return null;

    // Check if expired
    const now = Date.now();
    if (now - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return entry.data;
  }

  set<T>(key: string, data: T): void {
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
    });
  }

  clear(): void {
    this.cache.clear();
  }

  delete(key: string): void {
    this.cache.delete(key);
  }

  has(key: string): boolean {
    const entry = this.cache.get(key);
    if (!entry) return false;

    // Check expiration
    const now = Date.now();
    if (now - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return false;
    }

    return true;
  }
}

// Global cache instance
const apiCache = new CacheManager(5 * 60 * 1000); // 5 minutes TTL

// Hook using cache manager
function useApiData<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    // Check cache
    const cached = apiCache.get<T>(url);
    if (cached) {
      setData(cached);
      setLoading(false);
      return;
    }

    // Fetch if not cached
    setLoading(true);
    fetch(url)
      .then((res) => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then((data) => {
        apiCache.set(url, data);
        setData(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading } = useApiData<User>(`/api/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

---

#### 3. localStorage Cache

```tsx
import { useState, useEffect } from 'react';

interface CacheOptions {
  ttl?: number; // Time to live in ms
  key: string;
}

function useLocalStorageCache<T>(options: CacheOptions) {
  const { key, ttl = 5 * 60 * 1000 } = options;

  const get = (): T | null => {
    try {
      const item = localStorage.getItem(key);
      if (!item) return null;

      const parsed = JSON.parse(item);
      const now = Date.now();

      // Check expiration
      if (ttl && now - parsed.timestamp > ttl) {
        localStorage.removeItem(key);
        return null;
      }

      return parsed.data;
    } catch {
      return null;
    }
  };

  const set = (data: T) => {
    try {
      const item = {
        data,
        timestamp: Date.now(),
      };
      localStorage.setItem(key, JSON.stringify(item));
    } catch (error) {
      console.error('Failed to cache data:', error);
    }
  };

  const remove = () => {
    localStorage.removeItem(key);
  };

  return { get, set, remove };
}

// Usage
function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const cache = useLocalStorageCache<User[]>({
    key: 'users-list',
    ttl: 10 * 60 * 1000, // 10 minutes
  });

  useEffect(() => {
    // Try cache first
    const cached = cache.get();
    if (cached) {
      setUsers(cached);
      return;
    }

    // Fetch if not cached
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        cache.set(data); // Store in localStorage
      });
  }, []);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

#### 4. React Query Cache (Recommended)

```tsx
import { useQuery, useQueryClient } from '@tanstack/react-query';

function UserProfile({ userId }: { userId: string }) {
  // Automatic caching with React Query
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then((res) => res.json()),
    staleTime: 5 * 60 * 1000, // Consider data fresh for 5 minutes
    cacheTime: 10 * 60 * 1000, // Keep in cache for 10 minutes
  });

  return <div>{user?.name}</div>;
}

// Programmatic cache manipulation
function UserActions({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  // Update cache directly
  const updateUserCache = (newData: User) => {
    queryClient.setQueryData(['user', userId], newData);
  };

  // Invalidate cache (force refetch)
  const invalidateUser = () => {
    queryClient.invalidateQueries({ queryKey: ['user', userId] });
  };

  // Prefetch user data
  const prefetchUser = () => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetch(`/api/users/${userId}`).then((res) => res.json()),
    });
  };

  return (
    <div>
      <button onClick={invalidateUser}>Refresh</button>
      <button onClick={prefetchUser}>Prefetch</button>
    </div>
  );
}
```

---

#### 5. SWR Cache

```tsx
import useSWR, { mutate } from 'swr';

const fetcher = (url: string) => fetch(url).then((res) => res.json());

function UserProfile({ userId }: { userId: string }) {
  // Automatic caching with SWR
  const { data: user } = useSWR(`/api/users/${userId}`, fetcher, {
    dedupingInterval: 2000, // Dedupe requests within 2 seconds
    revalidateOnFocus: false, // Don't revalidate on window focus
    revalidateOnReconnect: true, // Revalidate on reconnect
  });

  return <div>{user?.name}</div>;
}

// Programmatic cache manipulation
function updateUserCache(userId: string, newData: User) {
  // Update cache without revalidation
  mutate(`/api/users/${userId}`, newData, false);
}

// Revalidate cache
function revalidateUser(userId: string) {
  mutate(`/api/users/${userId}`);
}

// Clear cache
function clearUserCache(userId: string) {
  mutate(`/api/users/${userId}`, undefined, false);
}
```

---

#### 6. IndexedDB Cache (Large Data)

```tsx
import { openDB, DBSchema, IDBPDatabase } from 'idb';

interface CacheDB extends DBSchema {
  cache: {
    key: string;
    value: {
      data: any;
      timestamp: number;
    };
  };
}

class IndexedDBCache {
  private db: IDBPDatabase<CacheDB> | null = null;
  private readonly dbName = 'app-cache';
  private readonly storeName = 'cache';

  async init() {
    this.db = await openDB<CacheDB>(this.dbName, 1, {
      upgrade(db) {
        if (!db.objectStoreNames.contains('cache')) {
          db.createObjectStore('cache');
        }
      },
    });
  }

  async get<T>(key: string, ttl?: number): Promise<T | null> {
    if (!this.db) await this.init();

    const entry = await this.db!.get(this.storeName, key);
    if (!entry) return null;

    // Check expiration
    if (ttl) {
      const now = Date.now();
      if (now - entry.timestamp > ttl) {
        await this.delete(key);
        return null;
      }
    }

    return entry.data;
  }

  async set<T>(key: string, data: T): Promise<void> {
    if (!this.db) await this.init();

    await this.db!.put(this.storeName, {
      data,
      timestamp: Date.now(),
    }, key);
  }

  async delete(key: string): Promise<void> {
    if (!this.db) await this.init();
    await this.db!.delete(this.storeName, key);
  }

  async clear(): Promise<void> {
    if (!this.db) await this.init();
    await this.db!.clear(this.storeName);
  }
}

// Usage
const dbCache = new IndexedDBCache();

function useLargeDataCache<T>(key: string, fetcher: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadData = async () => {
      // Try cache first
      const cached = await dbCache.get<T>(key, 60 * 60 * 1000); // 1 hour TTL
      if (cached) {
        setData(cached);
        setLoading(false);
        return;
      }

      // Fetch if not cached
      const fetchedData = await fetcher();
      await dbCache.set(key, fetchedData);
      setData(fetchedData);
      setLoading(false);
    };

    loadData();
  }, [key]);

  return { data, loading };
}
```

---

#### 7. HTTP Cache Headers

```tsx
// Server-side: Set cache headers
// This is typically done on your API/server

// Express.js example:
app.get('/api/users/:id', (req, res) => {
  // Cache for 5 minutes
  res.set('Cache-Control', 'public, max-age=300');
  res.json(userData);
});

// Or immutable resources:
app.get('/api/data/:id', (req, res) => {
  // Cache indefinitely (immutable)
  res.set('Cache-Control', 'public, max-age=31536000, immutable');
  res.json(data);
});

// React fetch with cache:
fetch('/api/users/123', {
  cache: 'force-cache', // Use cached response if available
});

fetch('/api/users/123', {
  cache: 'no-store', // Always fetch fresh
});

fetch('/api/users/123', {
  cache: 'reload', // Always fetch, update cache
});
```

---

#### 8. Service Worker Cache

```tsx
// service-worker.js
const CACHE_NAME = 'app-cache-v1';
const urlsToCache = [
  '/',
  '/static/css/main.css',
  '/static/js/main.js',
];

// Install - cache resources
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(urlsToCache);
    })
  );
});

// Fetch - serve from cache, fallback to network
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      // Cache hit - return response
      if (response) {
        return response;
      }

      // Clone request
      const fetchRequest = event.request.clone();

      // Fetch from network
      return fetch(fetchRequest).then((response) => {
        // Check if valid response
        if (!response || response.status !== 200 || response.type !== 'basic') {
          return response;
        }

        // Clone response
        const responseToCache = response.clone();

        // Cache the response
        caches.open(CACHE_NAME).then((cache) => {
          cache.put(event.request, responseToCache);
        });

        return response;
      });
    })
  );
});

// In React app - register service worker
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
      .then((registration) => {
        console.log('SW registered:', registration);
      })
      .catch((error) => {
        console.log('SW registration failed:', error);
      });
  });
}
```

---

#### 9. Cache Invalidation Strategies

```tsx
import { useQueryClient } from '@tanstack/react-query';

function CacheInvalidationExample() {
  const queryClient = useQueryClient();

  // Strategy 1: Time-based invalidation
  useEffect(() => {
    const interval = setInterval(() => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }, 5 * 60 * 1000); // Every 5 minutes

    return () => clearInterval(interval);
  }, [queryClient]);

  // Strategy 2: Event-based invalidation
  const handleUserUpdate = async (userId: string, data: Partial<User>) => {
    await fetch(`/api/users/${userId}`, {
      method: 'PATCH',
      body: JSON.stringify(data),
    });

    // Invalidate related caches
    queryClient.invalidateQueries({ queryKey: ['users'] });
    queryClient.invalidateQueries({ queryKey: ['user', userId] });
  };

  // Strategy 3: Mutation-based invalidation
  const mutation = useMutation({
    mutationFn: updateUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  // Strategy 4: Pattern-based invalidation
  const invalidateAllUserData = () => {
    queryClient.invalidateQueries({
      predicate: (query) =>
        Array.isArray(query.queryKey) &&
        query.queryKey[0] === 'user',
    });
  };

  return <div>Cache Invalidation Example</div>;
}
```

---

#### Best Practices

**1. Choose Appropriate Cache Storage:**
```tsx
// Small data (< 5MB) - use Memory/localStorage
const cache = new Map();

// Medium data (5-50MB) - use IndexedDB
const dbCache = new IndexedDBCache();

// Large data or offline-first - use Service Workers
```

**2. Set Appropriate TTL:**
```tsx
// Frequently changing data - short TTL
staleTime: 30 * 1000, // 30 seconds

// Rarely changing data - long TTL
staleTime: 60 * 60 * 1000, // 1 hour

// Static data - very long TTL
staleTime: Infinity,
```

**3. Implement Cache Versioning:**
```tsx
const CACHE_VERSION = 'v1';
const cacheKey = `${CACHE_VERSION}-users-${userId}`;

// When cache structure changes, increment version
// Old cache entries are automatically invalidated
```

**4. Handle Cache Errors:**
```tsx
try {
  const cached = await cache.get(key);
  if (cached) return cached;
} catch (error) {
  console.error('Cache error:', error);
  // Fallback to network fetch
}
```

**5. Monitor Cache Size:**
```tsx
class CacheManager {
  private maxSize = 100;

  set(key: string, data: any) {
    if (this.cache.size >= this.maxSize) {
      // Remove oldest entry (LRU)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, data);
  }
}
```

---

#### Cache Strategy Comparison

| Strategy | Use Case | Pros | Cons |
|----------|----------|------|------|
| **Memory Cache** | Small, temporary data | Fast, simple | Lost on reload |
| **localStorage** | Persistent, < 5MB | Persistent, simple | Size limit, synchronous |
| **IndexedDB** | Large data | No size limit, async | Complex API |
| **HTTP Cache** | Static assets | Native browser support | Less control |
| **Service Worker** | Offline-first | Full control, offline | Complex setup |
| **React Query/SWR** | API data | Automatic management | Library dependency |

---

#### Interview Tips

✅ **Key Points:**
- Caching improves performance by avoiding redundant requests
- Multiple levels: memory, localStorage, IndexedDB, HTTP, Service Workers
- Libraries like React Query/SWR handle caching automatically
- Implement TTL (time to live) for cache expiration
- Invalidate cache when data changes

✅ **When to Mention:**
- Discussing performance optimization
- Data fetching strategies
- Offline-first applications
- State management patterns
- User experience improvements

✅ **Common Follow-ups:**
- "What cache storage should you use?"
- "How do you handle cache invalidation?"
- "What's the difference between stale-while-revalidate?"
- "How does React Query caching work?"
- "When would you use Service Workers?"

✅ **Perfect Answer Structure:**
1. Define: Caching stores data to avoid repeated requests
2. Types: Memory, localStorage, IndexedDB, HTTP, Service Workers
3. Libraries: React Query, SWR handle automatically
4. Example: Show cache with TTL implementation
5. Invalidation: Time-based, event-based strategies
6. Best practices: Appropriate TTL, cache versioning, size limits

</details>

---

### 115. What is RTK Query and how does it compare to React Query?

<details>
<summary>View Answer</summary>

**RTK Query** is a data fetching and caching tool built into Redux Toolkit. It's designed to simplify common data fetching patterns in Redux applications with automatic cache management, request deduplication, and polling support.

#### What is RTK Query?

RTK Query provides:
- **Built into Redux Toolkit** - part of @reduxjs/toolkit
- **Automatic caching** - intelligent cache management
- **Redux integration** - works seamlessly with Redux
- **TypeScript support** - excellent type inference
- **Code generation** - auto-generate hooks
- **Endpoints definition** - centralized API definitions
- **Tag-based invalidation** - smart cache invalidation

---

#### Basic RTK Query Setup

```tsx
// src/services/api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface Post {
  id: string;
  title: string;
  userId: string;
}

// Define API slice
export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User', 'Post'], // For cache invalidation
  endpoints: (builder) => ({
    // Query endpoint (GET)
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      providesTags: ['User'], // Tag for cache invalidation
    }),

    // Query with parameter
    getUser: builder.query<User, string>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),

    // Mutation endpoint (POST/PUT/DELETE)
    createUser: builder.mutation<User, Partial<User>>({
      query: (body) => ({
        url: '/users',
        method: 'POST',
        body,
      }),
      invalidatesTags: ['User'], // Invalidate user cache
    }),

    updateUser: builder.mutation<User, { id: string; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),

    deleteUser: builder.mutation<void, string>({
      query: (id) => ({
        url: `/users/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['User'],
    }),
  }),
});

// Export auto-generated hooks
export const {
  useGetUsersQuery,
  useGetUserQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
} = api;
```

**Configure Store:**
```tsx
// src/store.ts
import { configureStore } from '@reduxjs/toolkit';
import { api } from './services/api';

export const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});
```

**Setup Provider:**
```tsx
// src/App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

function App() {
  return (
    <Provider store={store}>
      <YourApp />
    </Provider>
  );
}
```

---

#### Using RTK Query in Components

```tsx
import {
  useGetUsersQuery,
  useGetUserQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
} from './services/api';

// Query example
function UserList() {
  const { data: users, isLoading, error, refetch } = useGetUsersQuery();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading users</div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      <ul>
        {users?.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Query with parameter
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useGetUserQuery(userId);

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user?.name}</h2>
      <p>{user?.email}</p>
    </div>
  );
}

// Mutation example
function CreateUserForm() {
  const [createUser, { isLoading, error }] = useCreateUserMutation();

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    try {
      await createUser({
        name: formData.get('name') as string,
        email: formData.get('email') as string,
      }).unwrap();
      alert('User created!');
    } catch (err) {
      console.error('Failed to create user:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Creating...' : 'Create User'}
      </button>
      {error && <div>Error: {error.toString()}</div>}
    </form>
  );
}
```

---

#### RTK Query Advanced Features

**1. Polling (Auto-refresh):**
```tsx
function LiveUserList() {
  const { data: users } = useGetUsersQuery(undefined, {
    pollingInterval: 3000, // Refetch every 3 seconds
  });

  return (
    <ul>
      {users?.map((user) => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**2. Conditional Fetching:**
```tsx
function UserProfile({ userId }: { userId: string | null }) {
  const { data: user } = useGetUserQuery(userId!, {
    skip: !userId, // Skip query if userId is null
  });

  return <div>{user?.name}</div>;
}
```

**3. Optimistic Updates:**
```tsx
export const api = createApi({
  // ... other config
  endpoints: (builder) => ({
    updateUser: builder.mutation<User, { id: string; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: data,
      }),
      // Optimistic update
      async onQueryStarted({ id, data }, { dispatch, queryFulfilled }) {
        // Update cache optimistically
        const patchResult = dispatch(
          api.util.updateQueryData('getUser', id, (draft) => {
            Object.assign(draft, data);
          })
        );

        try {
          await queryFulfilled;
        } catch {
          // Rollback on error
          patchResult.undo();
        }
      },
    }),
  }),
});
```

**4. Prefetching:**
```tsx
function UserListItem({ user }: { user: User }) {
  const [prefetchUser] = api.usePrefetch('getUser');

  return (
    <li
      onMouseEnter={() => prefetchUser(user.id)}
      onClick={() => navigate(`/users/${user.id}`)}
    >
      {user.name}
    </li>
  );
}
```

**5. Custom Base Query:**
```tsx
import { fetchBaseQuery } from '@reduxjs/toolkit/query';
import type { BaseQueryFn } from '@reduxjs/toolkit/query';

const baseQuery = fetchBaseQuery({
  baseUrl: '/api',
  prepareHeaders: (headers, { getState }) => {
    const token = (getState() as RootState).auth.token;
    if (token) {
      headers.set('authorization', `Bearer ${token}`);
    }
    return headers;
  },
});

// With retry and error handling
const baseQueryWithRetry: BaseQueryFn = async (args, api, extraOptions) => {
  let result = await baseQuery(args, api, extraOptions);

  // Retry on 429 (rate limit)
  if (result.error && result.error.status === 429) {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    result = await baseQuery(args, api, extraOptions);
  }

  return result;
};
```

---

#### React Query Setup (Comparison)

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query
function UserList() {
  const { data: users, isLoading, error, refetch } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading users</div>;

  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      <ul>
        {users?.map((user: User) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Mutation
function CreateUserForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (userData: Partial<User>) =>
      fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      }).then((res) => res.json()),
    onSuccess: () => {
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
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

---

#### RTK Query vs React Query Comparison

| Feature | RTK Query | React Query |
|---------|-----------|-------------|
| **Bundle Size** | ~9KB (with Redux) | ~13KB (standalone) |
| **Redux Integration** | ✅ Native | ❌ Separate |
| **Setup Complexity** | More (Redux needed) | Less (standalone) |
| **API Definition** | Centralized (slice) | Decentralized (hooks) |
| **TypeScript** | Excellent | Excellent |
| **DevTools** | Redux DevTools | React Query DevTools |
| **Cache Invalidation** | Tag-based | Key-based |
| **Optimistic Updates** | Built-in | Built-in |
| **Polling** | ✅ Built-in | ✅ Built-in |
| **Prefetching** | ✅ Built-in | ✅ Built-in |
| **Offline Support** | Limited | Better |
| **Framework Agnostic** | ❌ Redux only | ✅ React, Vue, Svelte |
| **Learning Curve** | Steeper (Redux) | Gentler |
| **Community** | Redux community | Large, active |
| **Best For** | Existing Redux apps | New projects |

---

#### Key Differences

**1. API Definition:**

**RTK Query - Centralized:**
```tsx
// All endpoints in one place
export const api = createApi({
  endpoints: (builder) => ({
    getUsers: builder.query(...),
    getUser: builder.query(...),
    createUser: builder.mutation(...),
  }),
});
```

**React Query - Decentralized:**
```tsx
// Hooks defined where needed
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
const { data } = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
const mutation = useMutation({ mutationFn: createUser });
```

**2. Cache Invalidation:**

**RTK Query - Tag-based:**
```tsx
export const api = createApi({
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      providesTags: ['User'],
    }),
    createUser: builder.mutation({
      invalidatesTags: ['User'], // Invalidates all 'User' queries
    }),
  }),
});
```

**React Query - Key-based:**
```tsx
const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
});
```

**3. Redux Integration:**

**RTK Query:**
- Data stored in Redux store
- Access via Redux DevTools
- Use Redux selectors
- Integrates with existing Redux code

**React Query:**
- Separate cache (not in Redux)
- Own DevTools
- No Redux needed
- Fully independent

---

#### When to Use Each

**Use RTK Query When:**

✅ Already using Redux Toolkit
✅ Want centralized API definition
✅ Need Redux integration
✅ Prefer Redux DevTools
✅ Team familiar with Redux

**Use React Query When:**

✅ Starting new project
✅ Don't need Redux
✅ Want simpler setup
✅ Need framework flexibility
✅ Want better DevTools
✅ Prefer decentralized approach

---

#### Migration Example

**From RTK Query to React Query:**

```tsx
// RTK Query
const { data, isLoading } = useGetUsersQuery();
const [createUser] = useCreateUserMutation();

// React Query equivalent
const { data, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: () => fetch('/api/users').then((res) => res.json()),
});
const mutation = useMutation({
  mutationFn: (user) => fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify(user),
  }).then((res) => res.json()),
});
```

**From React Query to RTK Query:**

```tsx
// React Query
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
const mutation = useMutation({ mutationFn: createUser });

// RTK Query equivalent
const { data } = useGetUsersQuery();
const [createUser] = useCreateUserMutation();
```

---

#### Best Practices

**RTK Query:**

1. **Use Tag Types for Invalidation:**
```tsx
tagTypes: ['User', 'Post'],
providesTags: (result) => 
  result
    ? [...result.map(({ id }) => ({ type: 'User' as const, id })), 'User']
    : ['User'],
```

2. **Centralize API Logic:**
```tsx
// Keep all API endpoints in one file
export const api = createApi({
  endpoints: (builder) => ({
    // All endpoints here
  }),
});
```

3. **Use Optimistic Updates:**
```tsx
onQueryStarted: async (arg, { dispatch, queryFulfilled }) => {
  const patch = dispatch(api.util.updateQueryData(...));
  try {
    await queryFulfilled;
  } catch {
    patch.undo();
  }
},
```

**React Query:**

1. **Use Query Keys Consistently:**
```tsx
queryKey: ['users']                    // All users
queryKey: ['users', userId]            // Specific user
queryKey: ['users', userId, 'posts']   // User's posts
```

2. **Extract to Custom Hooks:**
```tsx
function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
}
```

3. **Handle Cache Invalidation:**
```tsx
queryClient.invalidateQueries({ queryKey: ['users'] });
```

---

#### Interview Tips

✅ **Key Points:**
- RTK Query is part of Redux Toolkit for data fetching
- React Query is standalone, more flexible
- RTK Query: centralized API, tag-based invalidation, Redux integration
- React Query: decentralized, key-based, no Redux needed
- Both support caching, polling, optimistic updates

✅ **When to Mention:**
- Discussing Redux vs other state management
- Data fetching strategies
- Cache management
- Team already using Redux
- Comparing state management libraries

✅ **Common Follow-ups:**
- "Which one should you use?"
- "What are the main differences?"
- "Can you use both together?"
- "How does RTK Query compare to Redux Thunk?"
- "What about bundle size?"

✅ **Perfect Answer Structure:**
1. Define both: RTK Query (Redux), React Query (standalone)
2. Key difference: Centralized vs decentralized API definition
3. RTK Query: For Redux apps, tag-based invalidation
4. React Query: For new projects, simpler setup
5. Comparison table: Size, setup, integration
6. When to use each: Redux = RTK Query, New = React Query

</details>

---

### 116. How do you implement real-time updates with WebSockets?

<details>
<summary>View Answer</summary>

**WebSockets** provide full-duplex, bidirectional communication between client and server, enabling real-time updates without polling. Perfect for chat applications, live notifications, collaborative editing, and real-time dashboards.

#### What are WebSockets?

- **Persistent connection** - stays open for continuous communication
- **Low latency** - instant message delivery
- **Bidirectional** - both client and server can send messages
- **Efficient** - no HTTP overhead per message
- **Real-time** - instant updates without polling

**vs. HTTP Polling:**
- HTTP: Client repeatedly asks "any updates?" (wasteful)
- WebSocket: Server pushes updates when available (efficient)

---

#### 1. Basic WebSocket Implementation

```tsx
import { useEffect, useState } from 'react';

interface Message {
  id: string;
  text: string;
  timestamp: number;
}

function ChatRoom() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [ws, setWs] = useState<WebSocket | null>(null);
  const [connectionStatus, setConnectionStatus] = useState<'connecting' | 'connected' | 'disconnected'>('disconnected');

  useEffect(() => {
    // Create WebSocket connection
    const websocket = new WebSocket('ws://localhost:8080');

    websocket.onopen = () => {
      console.log('WebSocket connected');
      setConnectionStatus('connected');
    };

    websocket.onmessage = (event) => {
      const message: Message = JSON.parse(event.data);
      setMessages((prev) => [...prev, message]);
    };

    websocket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    websocket.onclose = () => {
      console.log('WebSocket disconnected');
      setConnectionStatus('disconnected');
    };

    setWs(websocket);

    // Cleanup on unmount
    return () => {
      websocket.close();
    };
  }, []);

  const sendMessage = (text: string) => {
    if (ws && ws.readyState === WebSocket.OPEN) {
      const message: Message = {
        id: Date.now().toString(),
        text,
        timestamp: Date.now(),
      };
      ws.send(JSON.stringify(message));
    }
  };

  return (
    <div>
      <div>Status: {connectionStatus}</div>
      <div>
        {messages.map((msg) => (
          <div key={msg.id}>{msg.text}</div>
        ))}
      </div>
      <input
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            sendMessage(e.currentTarget.value);
            e.currentTarget.value = '';
          }
        }}
      />
    </div>
  );
}
```

---

#### 2. Custom WebSocket Hook

```tsx
import { useEffect, useRef, useState, useCallback } from 'react';

type ReadyState = 'connecting' | 'open' | 'closing' | 'closed';

interface UseWebSocketOptions {
  onOpen?: (event: Event) => void;
  onClose?: (event: CloseEvent) => void;
  onMessage?: (event: MessageEvent) => void;
  onError?: (event: Event) => void;
  reconnect?: boolean;
  reconnectInterval?: number;
  reconnectAttempts?: number;
}

function useWebSocket(url: string, options: UseWebSocketOptions = {}) {
  const {
    onOpen,
    onClose,
    onMessage,
    onError,
    reconnect = true,
    reconnectInterval = 3000,
    reconnectAttempts = 5,
  } = options;

  const [readyState, setReadyState] = useState<ReadyState>('connecting');
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectCountRef = useRef(0);
  const reconnectTimerRef = useRef<NodeJS.Timeout>();

  const connect = useCallback(() => {
    try {
      const ws = new WebSocket(url);

      ws.onopen = (event) => {
        setReadyState('open');
        reconnectCountRef.current = 0;
        onOpen?.(event);
      };

      ws.onclose = (event) => {
        setReadyState('closed');
        onClose?.(event);

        // Attempt reconnect
        if (reconnect && reconnectCountRef.current < reconnectAttempts) {
          reconnectCountRef.current++;
          reconnectTimerRef.current = setTimeout(() => {
            connect();
          }, reconnectInterval);
        }
      };

      ws.onmessage = (event) => {
        onMessage?.(event);
      };

      ws.onerror = (event) => {
        setReadyState('closed');
        onError?.(event);
      };

      wsRef.current = ws;
    } catch (error) {
      console.error('WebSocket connection error:', error);
    }
  }, [url, onOpen, onClose, onMessage, onError, reconnect, reconnectInterval, reconnectAttempts]);

  useEffect(() => {
    connect();

    return () => {
      if (reconnectTimerRef.current) {
        clearTimeout(reconnectTimerRef.current);
      }
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, [connect]);

  const send = useCallback((data: string | ArrayBufferLike | Blob | ArrayBufferView) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(data);
    } else {
      console.warn('WebSocket is not open. ReadyState:', wsRef.current?.readyState);
    }
  }, []);

  const close = useCallback(() => {
    if (wsRef.current) {
      wsRef.current.close();
    }
  }, []);

  return {
    send,
    close,
    readyState,
    connect,
  };
}

// Usage
function ChatApp() {
  const [messages, setMessages] = useState<Message[]>([]);

  const { send, readyState } = useWebSocket('ws://localhost:8080', {
    onMessage: (event) => {
      const message = JSON.parse(event.data);
      setMessages((prev) => [...prev, message]);
    },
    onOpen: () => console.log('Connected'),
    onClose: () => console.log('Disconnected'),
    reconnect: true,
    reconnectAttempts: 5,
  });

  const sendMessage = (text: string) => {
    send(JSON.stringify({ text, timestamp: Date.now() }));
  };

  return (
    <div>
      <div>Status: {readyState}</div>
      <MessageList messages={messages} />
      <MessageInput onSend={sendMessage} disabled={readyState !== 'open'} />
    </div>
  );
}
```

---

#### 3. WebSocket with React Context

```tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';

interface WebSocketContextValue {
  send: (data: string) => void;
  lastMessage: MessageEvent | null;
  readyState: ReadyState;
}

const WebSocketContext = createContext<WebSocketContextValue | null>(null);

export function WebSocketProvider({
  url,
  children,
}: {
  url: string;
  children: ReactNode;
}) {
  const [ws, setWs] = useState<WebSocket | null>(null);
  const [lastMessage, setLastMessage] = useState<MessageEvent | null>(null);
  const [readyState, setReadyState] = useState<ReadyState>('connecting');

  useEffect(() => {
    const websocket = new WebSocket(url);

    websocket.onopen = () => setReadyState('open');
    websocket.onclose = () => setReadyState('closed');
    websocket.onmessage = (event) => setLastMessage(event);
    websocket.onerror = () => setReadyState('closed');

    setWs(websocket);

    return () => websocket.close();
  }, [url]);

  const send = (data: string) => {
    if (ws?.readyState === WebSocket.OPEN) {
      ws.send(data);
    }
  };

  return (
    <WebSocketContext.Provider value={{ send, lastMessage, readyState }}>
      {children}
    </WebSocketContext.Provider>
  );
}

export function useWebSocketContext() {
  const context = useContext(WebSocketContext);
  if (!context) {
    throw new Error('useWebSocketContext must be used within WebSocketProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <WebSocketProvider url="ws://localhost:8080">
      <ChatRoom />
      <NotificationPanel />
    </WebSocketProvider>
  );
}

function ChatRoom() {
  const { send, lastMessage, readyState } = useWebSocketContext();
  const [messages, setMessages] = useState<any[]>([]);

  useEffect(() => {
    if (lastMessage) {
      const message = JSON.parse(lastMessage.data);
      setMessages((prev) => [...prev, message]);
    }
  }, [lastMessage]);

  return (
    <div>
      <div>Status: {readyState}</div>
      {messages.map((msg, i) => (
        <div key={i}>{msg.text}</div>
      ))}
      <button onClick={() => send(JSON.stringify({ text: 'Hello' }))}>
        Send
      </button>
    </div>
  );
}
```

---

#### 4. WebSocket with State Management (Zustand)

```tsx
import { create } from 'zustand';

interface Message {
  id: string;
  text: string;
  userId: string;
  timestamp: number;
}

interface WebSocketStore {
  ws: WebSocket | null;
  messages: Message[];
  connectionStatus: 'connecting' | 'connected' | 'disconnected';
  connect: (url: string) => void;
  disconnect: () => void;
  sendMessage: (text: string) => void;
  addMessage: (message: Message) => void;
}

export const useWebSocketStore = create<WebSocketStore>((set, get) => ({
  ws: null,
  messages: [],
  connectionStatus: 'disconnected',

  connect: (url: string) => {
    const ws = new WebSocket(url);

    ws.onopen = () => {
      set({ connectionStatus: 'connected' });
    };

    ws.onmessage = (event) => {
      const message: Message = JSON.parse(event.data);
      get().addMessage(message);
    };

    ws.onclose = () => {
      set({ connectionStatus: 'disconnected' });
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
      set({ connectionStatus: 'disconnected' });
    };

    set({ ws, connectionStatus: 'connecting' });
  },

  disconnect: () => {
    const { ws } = get();
    if (ws) {
      ws.close();
      set({ ws: null, connectionStatus: 'disconnected' });
    }
  },

  sendMessage: (text: string) => {
    const { ws } = get();
    if (ws?.readyState === WebSocket.OPEN) {
      const message: Message = {
        id: Date.now().toString(),
        text,
        userId: 'current-user',
        timestamp: Date.now(),
      };
      ws.send(JSON.stringify(message));
    }
  },

  addMessage: (message: Message) => {
    set((state) => ({
      messages: [...state.messages, message],
    }));
  },
}));

// Usage
function ChatComponent() {
  const { messages, connectionStatus, connect, sendMessage } = useWebSocketStore();
  const [input, setInput] = useState('');

  useEffect(() => {
    connect('ws://localhost:8080');
  }, [connect]);

  const handleSend = () => {
    if (input.trim()) {
      sendMessage(input);
      setInput('');
    }
  };

  return (
    <div>
      <div>Status: {connectionStatus}</div>
      <div>
        {messages.map((msg) => (
          <div key={msg.id}>
            <strong>{msg.userId}:</strong> {msg.text}
          </div>
        ))}
      </div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleSend()}
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
}
```

---

#### 5. Socket.IO Integration (Enhanced WebSockets)

```tsx
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

// Custom hook for Socket.IO
function useSocketIO(url: string) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const socketInstance = io(url, {
      transports: ['websocket'],
      autoConnect: true,
    });

    socketInstance.on('connect', () => {
      console.log('Socket.IO connected');
      setConnected(true);
    });

    socketInstance.on('disconnect', () => {
      console.log('Socket.IO disconnected');
      setConnected(false);
    });

    setSocket(socketInstance);

    return () => {
      socketInstance.disconnect();
    };
  }, [url]);

  return { socket, connected };
}

// Real-time chat with Socket.IO
function ChatWithSocketIO() {
  const { socket, connected } = useSocketIO('http://localhost:3001');
  const [messages, setMessages] = useState<Message[]>([]);
  const [typingUsers, setTypingUsers] = useState<string[]>([]);

  useEffect(() => {
    if (!socket) return;

    // Listen for messages
    socket.on('message', (message: Message) => {
      setMessages((prev) => [...prev, message]);
    });

    // Listen for typing indicator
    socket.on('user-typing', (userId: string) => {
      setTypingUsers((prev) => [...prev, userId]);
    });

    socket.on('user-stopped-typing', (userId: string) => {
      setTypingUsers((prev) => prev.filter((id) => id !== userId));
    });

    return () => {
      socket.off('message');
      socket.off('user-typing');
      socket.off('user-stopped-typing');
    };
  }, [socket]);

  const sendMessage = (text: string) => {
    if (socket && connected) {
      socket.emit('message', {
        text,
        timestamp: Date.now(),
      });
    }
  };

  const handleTyping = () => {
    socket?.emit('typing');
  };

  return (
    <div>
      <div>Status: {connected ? 'Connected' : 'Disconnected'}</div>
      <MessageList messages={messages} />
      {typingUsers.length > 0 && (
        <div>{typingUsers.join(', ')} is typing...</div>
      )}
      <MessageInput
        onSend={sendMessage}
        onTyping={handleTyping}
        disabled={!connected}
      />
    </div>
  );
}
```

---

#### 6. Real-Time Notifications

```tsx
import { useEffect, useState } from 'react';

interface Notification {
  id: string;
  type: 'info' | 'warning' | 'error' | 'success';
  message: string;
  timestamp: number;
}

function NotificationSystem() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [ws, setWs] = useState<WebSocket | null>(null);

  useEffect(() => {
    const websocket = new WebSocket('ws://localhost:8080/notifications');

    websocket.onmessage = (event) => {
      const notification: Notification = JSON.parse(event.data);
      setNotifications((prev) => [notification, ...prev]);

      // Auto-dismiss after 5 seconds
      setTimeout(() => {
        setNotifications((prev) =>
          prev.filter((n) => n.id !== notification.id)
        );
      }, 5000);
    };

    setWs(websocket);

    return () => websocket.close();
  }, []);

  return (
    <div className="notification-container">
      {notifications.map((notification) => (
        <div key={notification.id} className={`notification ${notification.type}`}>
          <span>{notification.message}</span>
          <button
            onClick={() =>
              setNotifications((prev) =>
                prev.filter((n) => n.id !== notification.id)
              )
            }
          >
            ×
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

#### 7. Live Data Dashboard

```tsx
import { useEffect, useState } from 'react';
import { Line } from 'react-chartjs-2';

interface DataPoint {
  timestamp: number;
  value: number;
}

function LiveDashboard() {
  const [dataPoints, setDataPoints] = useState<DataPoint[]>([]);
  const maxPoints = 50; // Keep last 50 points

  useEffect(() => {
    const ws = new WebSocket('ws://localhost:8080/metrics');

    ws.onmessage = (event) => {
      const dataPoint: DataPoint = JSON.parse(event.data);
      setDataPoints((prev) => {
        const updated = [...prev, dataPoint];
        // Keep only last maxPoints
        return updated.slice(-maxPoints);
      });
    };

    return () => ws.close();
  }, []);

  const chartData = {
    labels: dataPoints.map((p) => new Date(p.timestamp).toLocaleTimeString()),
    datasets: [
      {
        label: 'Real-time Metrics',
        data: dataPoints.map((p) => p.value),
        borderColor: 'rgb(75, 192, 192)',
        tension: 0.1,
      },
    ],
  };

  return (
    <div>
      <h2>Live Dashboard</h2>
      <div>Latest: {dataPoints[dataPoints.length - 1]?.value}</div>
      <Line data={chartData} />
    </div>
  );
}
```

---

#### 8. Collaborative Editing (Cursor Tracking)

```tsx
import { useEffect, useState } from 'react';

interface Cursor {
  userId: string;
  x: number;
  y: number;
  color: string;
}

function CollaborativeCanvas() {
  const [cursors, setCursors] = useState<Map<string, Cursor>>(new Map());
  const [ws, setWs] = useState<WebSocket | null>(null);

  useEffect(() => {
    const websocket = new WebSocket('ws://localhost:8080/collaborate');

    websocket.onmessage = (event) => {
      const { type, userId, x, y, color } = JSON.parse(event.data);

      if (type === 'cursor-move') {
        setCursors((prev) => {
          const updated = new Map(prev);
          updated.set(userId, { userId, x, y, color });
          return updated;
        });
      } else if (type === 'cursor-leave') {
        setCursors((prev) => {
          const updated = new Map(prev);
          updated.delete(userId);
          return updated;
        });
      }
    };

    setWs(websocket);

    return () => websocket.close();
  }, []);

  const handleMouseMove = (e: React.MouseEvent) => {
    if (ws?.readyState === WebSocket.OPEN) {
      ws.send(
        JSON.stringify({
          type: 'cursor-move',
          x: e.clientX,
          y: e.clientY,
        })
      );
    }
  };

  return (
    <div
      onMouseMove={handleMouseMove}
      style={{ width: '100vw', height: '100vh', position: 'relative' }}
    >
      {Array.from(cursors.values()).map((cursor) => (
        <div
          key={cursor.userId}
          style={{
            position: 'absolute',
            left: cursor.x,
            top: cursor.y,
            width: 10,
            height: 10,
            borderRadius: '50%',
            backgroundColor: cursor.color,
            pointerEvents: 'none',
          }}
        />
      ))}
    </div>
  );
}
```

---

#### Best Practices

**1. Handle Reconnection:**
```tsx
function reconnectWebSocket(url: string, attempts = 5) {
  let reconnectCount = 0;

  const connect = () => {
    const ws = new WebSocket(url);

    ws.onclose = () => {
      if (reconnectCount < attempts) {
        reconnectCount++;
        setTimeout(() => connect(), 3000);
      }
    };

    return ws;
  };

  return connect();
}
```

**2. Heartbeat/Ping-Pong:**
```tsx
useEffect(() => {
  if (!ws) return;

  const interval = setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify({ type: 'ping' }));
    }
  }, 30000); // Every 30 seconds

  return () => clearInterval(interval);
}, [ws]);
```

**3. Handle Authentication:**
```tsx
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  // Send auth token on connect
  ws.send(
    JSON.stringify({
      type: 'auth',
      token: getAuthToken(),
    })
  );
};
```

**4. Message Queue (Offline Support):**
```tsx
const messageQueue: string[] = [];

function sendMessage(data: string) {
  if (ws?.readyState === WebSocket.OPEN) {
    ws.send(data);
  } else {
    messageQueue.push(data);
  }
}

ws.onopen = () => {
  // Send queued messages
  while (messageQueue.length > 0) {
    const message = messageQueue.shift();
    ws.send(message!);
  }
};
```

**5. Error Handling:**
```tsx
ws.onerror = (error) => {
  console.error('WebSocket error:', error);
  // Show user-friendly error
  setError('Connection error. Retrying...');
};

ws.onclose = (event) => {
  if (!event.wasClean) {
    console.error('Connection closed unexpectedly');
  }
};
```

---

#### WebSocket vs Other Real-Time Options

| Approach | Latency | Efficiency | Use Case |
|----------|---------|------------|----------|
| **WebSocket** | Very low | High | Chat, live data, collaboration |
| **Server-Sent Events (SSE)** | Low | Medium | One-way updates, notifications |
| **Long Polling** | Medium | Low | Legacy browser support |
| **HTTP/2 Push** | Medium | Medium | Resource preloading |
| **Short Polling** | High | Very low | Avoid if possible |

---

#### Interview Tips

✅ **Key Points:**
- WebSockets provide persistent, bidirectional connections
- Use custom hooks to manage WebSocket lifecycle
- Implement reconnection logic for resilience
- Handle authentication, heartbeats, and error cases
- Consider Socket.IO for enhanced features

✅ **When to Mention:**
- Real-time features (chat, notifications, live updates)
- Performance optimization vs polling
- Collaborative applications
- Live dashboards and metrics
- Discussing state synchronization

✅ **Common Follow-ups:**
- "How do you handle reconnection?"
- "What's the difference between WebSocket and HTTP?"
- "When would you use Server-Sent Events instead?"
- "How do you secure WebSocket connections?"
- "What is Socket.IO and why use it?"

✅ **Perfect Answer Structure:**
1. Define: WebSocket = persistent, bidirectional connection
2. Benefits: Low latency, efficient, real-time updates
3. Custom hook: Show reusable WebSocket hook with reconnection
4. Use cases: Chat, notifications, live data, collaboration
5. Best practices: Reconnection, heartbeat, authentication, error handling
6. vs Alternatives: Compare with SSE, polling

</details>

---

### 117. What is the difference between local and global state?

<details>
<summary>View Answer</summary>

**Local state** is data managed within a single component, while **global state** is shared data accessible across multiple components throughout the application. Choosing between them is crucial for maintainability and performance.

#### Key Differences

| Aspect | Local State | Global State |
|--------|-------------|-------------|
| **Scope** | Single component | Entire application |
| **Access** | Only within component | Anywhere in app |
| **Tools** | `useState`, `useReducer` | Context, Redux, Zustand |
| **Performance** | No re-renders outside | Can cause unnecessary re-renders |
| **Complexity** | Simple | More complex |
| **Use Case** | Form inputs, toggles | User auth, theme, cart |
| **Lifetime** | Component lifecycle | Application lifecycle |
| **Testing** | Easier | Requires setup |

---

#### 1. Local State Examples

**Form Input (Local):**
```tsx
import { useState } from 'react';

function LoginForm() {
  // Local state - only this component needs it
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Submit logic
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type={showPassword ? 'text' : 'password'}
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="button" onClick={() => setShowPassword(!showPassword)}>
        {showPassword ? 'Hide' : 'Show'}
      </button>
      <button type="submit">Login</button>
    </form>
  );
}
```

**Toggle/Accordion (Local):**
```tsx
function Accordion({ title, children }: { title: string; children: ReactNode }) {
  // Local state - only this accordion needs to know if it's open
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {title}
        {isOpen ? '▼' : '▶'}
      </button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}
```

**Modal (Local):**
```tsx
function ProductCard({ product }: { product: Product }) {
  // Local state - only this card needs to manage its modal
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => setIsModalOpen(true)}>View Details</button>

      {isModalOpen && (
        <Modal onClose={() => setIsModalOpen(false)}>
          <ProductDetails product={product} />
        </Modal>
      )}
    </div>
  );
}
```

---

#### 2. Global State Examples

**User Authentication (Global):**
```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextValue | null>(null);

// Global state - user info needed throughout app
export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    const userData = await response.json();
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        login,
        logout,
        isAuthenticated: !!user,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}

// Usage across multiple components
function Header() {
  const { user, logout } = useAuth();
  return (
    <header>
      <span>Welcome, {user?.name}</span>
      <button onClick={logout}>Logout</button>
    </header>
  );
}

function UserProfile() {
  const { user } = useAuth();
  return <div>Email: {user?.email}</div>;
}
```

**Theme (Global):**
```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

// Global state - theme affects entire app
export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <div className={`app-${theme}`}>{children}</div>
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

// Usage
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

function Component() {
  const { theme } = useTheme();
  return <div>Current theme: {theme}</div>;
}
```

**Shopping Cart (Global):**
```tsx
import { create } from 'zustand';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  total: number;
}

// Global state - cart accessible from anywhere
export const useCartStore = create<CartStore>((set, get) => ({
  items: [],

  addItem: (item) => {
    set((state) => {
      const existing = state.items.find((i) => i.id === item.id);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
        };
      }
      return { items: [...state.items, { ...item, quantity: 1 }] };
    });
  },

  removeItem: (id) => {
    set((state) => ({
      items: state.items.filter((item) => item.id !== id),
    }));
  },

  updateQuantity: (id, quantity) => {
    set((state) => ({
      items: state.items.map((item) =>
        item.id === id ? { ...item, quantity } : item
      ),
    }));
  },

  clearCart: () => set({ items: [] }),

  get total() {
    return get().items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  },
}));

// Usage in multiple components
function ProductCard({ product }: { product: Product }) {
  const addItem = useCartStore((state) => state.addItem);

  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>Add to Cart</button>
    </div>
  );
}

function CartSummary() {
  const { items, total } = useCartStore();

  return (
    <div>
      <h3>Cart ({items.length} items)</h3>
      <p>Total: ${total}</p>
    </div>
  );
}

function Header() {
  const itemCount = useCartStore((state) => state.items.length);

  return (
    <header>
      <CartIcon count={itemCount} />
    </header>
  );
}
```

---

#### 3. When to Use Local vs Global State

**Use Local State When:**

✅ **UI State (Component-Specific):**
```tsx
// Form input values
const [email, setEmail] = useState('');

// Toggle states
const [isOpen, setIsOpen] = useState(false);

// Loading/error states for component-specific operations
const [isLoading, setIsLoading] = useState(false);
```

✅ **Temporary Data:**
```tsx
// Temporary search query
const [searchQuery, setSearchQuery] = useState('');

// Hover state
const [isHovered, setIsHovered] = useState(false);

// Focus state
const [isFocused, setIsFocused] = useState(false);
```

✅ **Single Component Concerns:**
```tsx
// Modal visibility for this component
const [showModal, setShowModal] = useState(false);

// Dropdown open state
const [isDropdownOpen, setIsDropdownOpen] = useState(false);
```

**Use Global State When:**

✅ **Cross-Component Data:**
```tsx
// User authentication - needed everywhere
const { user, isAuthenticated } = useAuth();

// App theme - affects entire UI
const { theme } = useTheme();

// Shopping cart - accessed from multiple pages
const { items, total } = useCart();
```

✅ **Persistent Data:**
```tsx
// User preferences
const { language, currency } = useSettings();

// App configuration
const { apiUrl, features } = useConfig();
```

✅ **Shared Business Logic:**
```tsx
// Multi-step form data shared across steps
const { formData, updateStep } = useCheckout();

// Real-time notifications
const { notifications } = useNotifications();
```

---

#### 4. Common Mistakes

**❌ Making Everything Global:**
```tsx
// BAD - form state doesn't need to be global
const { email, setEmail } = useGlobalStore();

return <input value={email} onChange={(e) => setEmail(e.target.value)} />;
```

**✅ Keep Form State Local:**
```tsx
// GOOD - form state is local
const [email, setEmail] = useState('');

return <input value={email} onChange={(e) => setEmail(e.target.value)} />;
```

**❌ Prop Drilling Instead of Global State:**
```tsx
// BAD - passing user through many levels
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user}><Page user={user} /></Layout>;
}

function Page({ user }) {
  return <Component user={user} />;
}

function Component({ user }) {
  return <div>{user.name}</div>;
}
```

**✅ Use Global State for Widely-Used Data:**
```tsx
// GOOD - user in global state, no prop drilling
function App() {
  return (
    <AuthProvider>
      <Layout><Page /></Layout>
    </AuthProvider>
  );
}

function Component() {
  const { user } = useAuth();
  return <div>{user.name}</div>;
}
```

**❌ Not Lifting State When Needed:**
```tsx
// BAD - siblings need to share, but state is local to one
function Parent() {
  return (
    <>
      <ComponentA /> {/* Has state */}
      <ComponentB /> {/* Needs same state */}
    </>
  );
}
```

**✅ Lift State to Common Parent:**
```tsx
// GOOD - lift state to parent
function Parent() {
  const [sharedData, setSharedData] = useState('');
  return (
    <>
      <ComponentA data={sharedData} onChange={setSharedData} />
      <ComponentB data={sharedData} />
    </>
  );
}
```

---

#### 5. Hybrid Approach (Server State vs Client State)

**Server State (Global):**
```tsx
import { useQuery } from '@tanstack/react-query';

// Server state - globally cached
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
}

// Multiple components can use same server state
function UserProfile({ userId }: { userId: string }) {
  const { data: user } = useUser(userId);
  return <div>{user?.name}</div>;
}

function UserAvatar({ userId }: { userId: string }) {
  const { data: user } = useUser(userId); // Same cache
  return <img src={user?.avatar} />;
}
```

**Client State (Local):**
```tsx
function SearchBar() {
  // Client state - search input, local to this component
  const [query, setQuery] = useState('');

  // Server state - search results, globally cached
  const { data: results } = useQuery({
    queryKey: ['search', query],
    queryFn: () => searchProducts(query),
    enabled: query.length > 0,
  });

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {results?.map((result) => (
        <div key={result.id}>{result.name}</div>
      ))}
    </div>
  );
}
```

---

#### 6. Decision Tree

```
Do multiple components need this data?
├─ No → Use Local State (useState)
└─ Yes
   ├─ Is it from server/API?
   │  └─ Yes → Use React Query/SWR (global cache)
   └─ No (client-side only)
      ├─ Is it UI state (theme, locale, auth)?
      │  └─ Yes → Use Context/Zustand (global state)
      └─ Can you lift state to common parent?
         ├─ Yes → Lift state
         └─ No → Use Context/Global state
```

---

#### 7. Performance Considerations

**Local State - Better Performance:**
```tsx
// Only this component re-renders when state changes
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Global State - Potential Performance Issues:**
```tsx
// Every component using this context re-renders
const AppContext = createContext();

function App() {
  const [state, setState] = useState({ user: null, theme: 'light', cart: [] });
  return <AppContext.Provider value={state}>{children}</AppContext.Provider>;
}

// This re-renders even if only user changes, not cart
function CartCount() {
  const { cart } = useContext(AppContext);
  return <div>{cart.length}</div>;
}
```

**Optimized Global State:**
```tsx
// Split contexts by concern
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Now only re-renders when cart changes
function CartCount() {
  const { items } = useCart();
  return <div>{items.length}</div>;
}

// Or use Zustand with selectors
const itemCount = useCartStore((state) => state.items.length);
// Only re-renders when items.length changes
```

---

#### Best Practices

**1. Start with Local State:**
```tsx
// Start simple
const [value, setValue] = useState('');

// Only lift to global if multiple components need it
```

**2. Split Global State by Domain:**
```tsx
// GOOD - separate concerns
<AuthProvider>
  <ThemeProvider>
    <CartProvider>
      <App />
    </CartProvider>
  </ThemeProvider>
</AuthProvider>

// BAD - everything in one context
<AppProvider value={{ user, theme, cart, settings, ...}}>
```

**3. Use Selectors for Performance:**
```tsx
// Only subscribe to what you need
const userName = useStore((state) => state.user.name);
// Not: const { user } = useStore();
```

**4. Consider Server State Separate:**
```tsx
// Server data - React Query
const { data: user } = useQuery(['user'], fetchUser);

// Client data - Local/Global state
const [isModalOpen, setIsModalOpen] = useState(false);
```

---

#### Interview Tips

✅ **Key Points:**
- Local state: Single component, useState/useReducer
- Global state: Multiple components, Context/Redux/Zustand
- Start with local, lift to global when needed
- Consider server state separately (React Query)
- Performance: Local state doesn't trigger re-renders outside component

✅ **When to Mention:**
- State management discussions
- Component architecture
- Performance optimization
- Prop drilling problems
- When to use Context vs Redux

✅ **Common Follow-ups:**
- "When should you lift state up?"
- "What's prop drilling and how to avoid it?"
- "How do you prevent unnecessary re-renders with global state?"
- "What's the difference between client and server state?"
- "When would you use Redux over Context?"

✅ **Perfect Answer Structure:**
1. Define both: Local (single component) vs Global (app-wide)
2. Local examples: Form inputs, toggles, temporary UI state
3. Global examples: Auth, theme, cart, settings
4. When to use each: Start local, lift when multiple components need it
5. Performance: Local better, optimize global with selectors
6. Server state: Consider React Query separately from client state

</details>

---

### 118. How do you handle state synchronization across tabs?

<details>
<summary>View Answer</summary>

**Cross-tab state synchronization** allows multiple browser tabs/windows of the same application to share and synchronize state in real-time. This is essential for multi-tab applications where users expect consistent data across all instances.

#### Why Synchronize Across Tabs?

- **Consistent user experience** - Same data in all tabs
- **Real-time updates** - Changes in one tab reflect in others
- **Authentication sync** - Logout in one tab logs out all
- **Shopping cart** - Cart updates across tabs
- **Notifications** - Dismiss in one, dismissed in all

---

#### Methods for Cross-Tab Synchronization

1. **localStorage + storage event**
2. **BroadcastChannel API**
3. **SharedWorker**
4. **IndexedDB + storage events**
5. **WebSocket/Server-side sync**
6. **Service Worker**

---

#### 1. localStorage with Storage Event (Most Common)

```tsx
import { useEffect, useState } from 'react';

// Simple cross-tab sync with localStorage
function useSharedState<T>(key: string, initialValue: T) {
  // Get initial value from localStorage
  const [state, setState] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // Update localStorage when state changes
  const setSharedState = (value: T | ((prev: T) => T)) => {
    setState((prev) => {
      const newValue = value instanceof Function ? value(prev) : value;
      localStorage.setItem(key, JSON.stringify(newValue));
      return newValue;
    });
  };

  // Listen for changes from other tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key && e.newValue) {
        try {
          setState(JSON.parse(e.newValue));
        } catch (error) {
          console.error('Failed to parse storage value:', error);
        }
      }
    };

    // Storage event only fires on OTHER tabs, not the current one
    window.addEventListener('storage', handleStorageChange);

    return () => {
      window.removeEventListener('storage', handleStorageChange);
    };
  }, [key]);

  return [state, setSharedState] as const;
}

// Usage
function App() {
  const [count, setCount] = useSharedState('app-count', 0);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <p>Open this app in multiple tabs to see sync!</p>
    </div>
  );
}
```

---

#### 2. Shopping Cart Synchronization

```tsx
import { useEffect, useState } from 'react';

interface CartItem {
  id: string;
  name: string;
  quantity: number;
  price: number;
}

function useSyncedCart() {
  const CART_KEY = 'shopping-cart';
  
  const [cart, setCart] = useState<CartItem[]>(() => {
    const stored = localStorage.getItem(CART_KEY);
    return stored ? JSON.parse(stored) : [];
  });

  // Sync to localStorage
  useEffect(() => {
    localStorage.setItem(CART_KEY, JSON.stringify(cart));
  }, [cart]);

  // Listen for changes from other tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === CART_KEY && e.newValue) {
        setCart(JSON.parse(e.newValue));
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, []);

  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    setCart((prev) => {
      const existing = prev.find((i) => i.id === item.id);
      if (existing) {
        return prev.map((i) =>
          i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
        );
      }
      return [...prev, { ...item, quantity: 1 }];
    });
  };

  const removeItem = (id: string) => {
    setCart((prev) => prev.filter((item) => item.id !== id));
  };

  const clearCart = () => {
    setCart([]);
  };

  return { cart, addItem, removeItem, clearCart };
}

// Usage
function ShoppingCart() {
  const { cart, addItem, removeItem, clearCart } = useSyncedCart();

  const total = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

  return (
    <div>
      <h2>Shopping Cart (Synced Across Tabs)</h2>
      {cart.map((item) => (
        <div key={item.id}>
          <span>{item.name} x {item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}
```

---

#### 3. Authentication Synchronization

```tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextValue {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

const AUTH_KEY = 'auth-user';
const AUTH_EVENT = 'auth-change';

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(() => {
    const stored = localStorage.getItem(AUTH_KEY);
    return stored ? JSON.parse(stored) : null;
  });

  const login = (userData: User) => {
    setUser(userData);
    localStorage.setItem(AUTH_KEY, JSON.stringify(userData));
    // Manually trigger storage event for current tab
    window.dispatchEvent(new Event(AUTH_EVENT));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem(AUTH_KEY);
    window.dispatchEvent(new Event(AUTH_EVENT));
  };

  // Listen for auth changes from other tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === AUTH_KEY) {
        setUser(e.newValue ? JSON.parse(e.newValue) : null);
      }
    };

    // Also listen to custom event for same-tab updates
    const handleAuthChange = () => {
      const stored = localStorage.getItem(AUTH_KEY);
      setUser(stored ? JSON.parse(stored) : null);
    };

    window.addEventListener('storage', handleStorageChange);
    window.addEventListener(AUTH_EVENT, handleAuthChange);

    return () => {
      window.removeEventListener('storage', handleStorageChange);
      window.removeEventListener(AUTH_EVENT, handleAuthChange);
    };
  }, []);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}

// Usage
function Header() {
  const { user, logout } = useAuth();

  return (
    <header>
      {user ? (
        <>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout (All Tabs)</button>
        </>
      ) : (
        <span>Not logged in</span>
      )}
    </header>
  );
}
```

---

#### 4. BroadcastChannel API (Modern Approach)

```tsx
import { useEffect, useState, useRef } from 'react';

function useBroadcastChannel<T>(channelName: string, initialValue: T) {
  const [state, setState] = useState<T>(initialValue);
  const channelRef = useRef<BroadcastChannel | null>(null);

  useEffect(() => {
    // Create broadcast channel
    const channel = new BroadcastChannel(channelName);
    channelRef.current = channel;

    // Listen for messages from other tabs
    channel.onmessage = (event) => {
      setState(event.data);
    };

    return () => {
      channel.close();
    };
  }, [channelName]);

  const broadcast = (value: T) => {
    setState(value);
    channelRef.current?.postMessage(value);
  };

  return [state, broadcast] as const;
}

// Usage
function NotificationSystem() {
  const [notifications, setNotifications] = useBroadcastChannel<string[]>(
    'notifications',
    []
  );

  const addNotification = (message: string) => {
    const updated = [...notifications, message];
    setNotifications(updated);
  };

  const clearNotifications = () => {
    setNotifications([]);
  };

  return (
    <div>
      <h2>Notifications (Synced via BroadcastChannel)</h2>
      {notifications.map((msg, i) => (
        <div key={i}>{msg}</div>
      ))}
      <button onClick={() => addNotification('New notification')}>
        Add Notification
      </button>
      <button onClick={clearNotifications}>Clear All</button>
    </div>
  );
}
```

---

#### 5. Advanced Zustand with Cross-Tab Sync

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface AppState {
  count: number;
  user: User | null;
  theme: 'light' | 'dark';
  increment: () => void;
  decrement: () => void;
  setUser: (user: User | null) => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

// Zustand with persistence automatically syncs across tabs
export const useAppStore = create<AppState>()(n  persist(
    (set) => ({
      count: 0,
      user: null,
      theme: 'light',

      increment: () => set((state) => ({ count: state.count + 1 })),
      decrement: () => set((state) => ({ count: state.count - 1 })),
      setUser: (user) => set({ user }),
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'app-storage', // localStorage key
      storage: createJSONStorage(() => localStorage),
    }
  )
);

// Listen for changes from other tabs
if (typeof window !== 'undefined') {
  window.addEventListener('storage', (e) => {
    if (e.key === 'app-storage' && e.newValue) {
      // Zustand will automatically sync
      useAppStore.persist.rehydrate();
    }
  });
}

// Usage
function Counter() {
  const { count, increment, decrement } = useAppStore();

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <p>Changes sync across all tabs!</p>
    </div>
  );
}
```

---

#### 6. Redux with Cross-Tab Sync

```tsx
import { configureStore } from '@reduxjs/toolkit';
import { useEffect } from 'react';

// Middleware for cross-tab sync
const crossTabSyncMiddleware = (store: any) => (next: any) => (action: any) => {
  const result = next(action);

  // Broadcast action to other tabs
  if (action.meta?.broadcast !== false) {
    const channel = new BroadcastChannel('redux-sync');
    channel.postMessage({
      type: action.type,
      payload: action.payload,
    });
    channel.close();
  }

  return result;
};

// Create store with middleware
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(crossTabSyncMiddleware),
});

// Listen for actions from other tabs
if (typeof window !== 'undefined') {
  const channel = new BroadcastChannel('redux-sync');
  
  channel.onmessage = (event) => {
    // Dispatch received action with broadcast: false to avoid loop
    store.dispatch({
      ...event.data,
      meta: { broadcast: false },
    });
  };
}

export default store;
```

---

#### 7. Custom Tab Sync Manager

```tsx
type SyncCallback<T> = (data: T) => void;

class TabSyncManager<T> {
  private channel: BroadcastChannel | null = null;
  private storageKey: string;
  private callbacks: Set<SyncCallback<T>> = new Set();

  constructor(key: string, useBroadcast = true) {
    this.storageKey = key;

    if (useBroadcast && 'BroadcastChannel' in window) {
      this.channel = new BroadcastChannel(key);
      this.channel.onmessage = (event) => {
        this.notifyCallbacks(event.data);
      };
    } else {
      // Fallback to localStorage events
      window.addEventListener('storage', this.handleStorageChange);
    }
  }

  private handleStorageChange = (e: StorageEvent) => {
    if (e.key === this.storageKey && e.newValue) {
      try {
        const data = JSON.parse(e.newValue);
        this.notifyCallbacks(data);
      } catch (error) {
        console.error('Failed to parse synced data:', error);
      }
    }
  };

  private notifyCallbacks(data: T) {
    this.callbacks.forEach((callback) => callback(data));
  }

  subscribe(callback: SyncCallback<T>) {
    this.callbacks.add(callback);
    return () => {
      this.callbacks.delete(callback);
    };
  }

  broadcast(data: T) {
    if (this.channel) {
      this.channel.postMessage(data);
    }
    // Also update localStorage for fallback
    localStorage.setItem(this.storageKey, JSON.stringify(data));
  }

  get(): T | null {
    const stored = localStorage.getItem(this.storageKey);
    return stored ? JSON.parse(stored) : null;
  }

  destroy() {
    this.channel?.close();
    window.removeEventListener('storage', this.handleStorageChange);
    this.callbacks.clear();
  }
}

// Usage
const cartSync = new TabSyncManager<CartItem[]>('cart');

function useCartSync() {
  const [cart, setCart] = useState<CartItem[]>(() => cartSync.get() || []);

  useEffect(() => {
    const unsubscribe = cartSync.subscribe((newCart) => {
      setCart(newCart);
    });

    return unsubscribe;
  }, []);

  const updateCart = (newCart: CartItem[]) => {
    setCart(newCart);
    cartSync.broadcast(newCart);
  };

  return [cart, updateCart] as const;
}
```

---

#### 8. Real-Time Tab Sync with WebSocket

```tsx
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

function useRealtimeSync<T>(key: string, initialValue: T) {
  const [state, setState] = useState<T>(initialValue);
  const [socket, setSocket] = useState<any>(null);

  useEffect(() => {
    const socketInstance = io('http://localhost:3001');

    socketInstance.on('connect', () => {
      // Subscribe to updates for this key
      socketInstance.emit('subscribe', key);
    });

    socketInstance.on('update', (data: { key: string; value: T }) => {
      if (data.key === key) {
        setState(data.value);
      }
    });

    setSocket(socketInstance);

    return () => {
      socketInstance.disconnect();
    };
  }, [key]);

  const update = (value: T) => {
    setState(value);
    socket?.emit('update', { key, value });
  };

  return [state, update] as const;
}

// Usage
function CollaborativeCounter() {
  const [count, setCount] = useRealtimeSync('shared-count', 0);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <p>Synced across all tabs AND all users in real-time!</p>
    </div>
  );
}
```

---

#### Comparison of Methods

| Method | Browser Support | Performance | Use Case |
|--------|----------------|-------------|----------|
| **localStorage + storage event** | Excellent | Good | Simple sync, <5MB data |
| **BroadcastChannel** | Modern browsers | Excellent | Fast, message-based sync |
| **SharedWorker** | Limited | Good | Complex shared logic |
| **IndexedDB** | Excellent | Good | Large data sync |
| **WebSocket** | Excellent | Real-time | Multi-user sync |
| **Service Worker** | Modern browsers | Good | Offline-first apps |

---

#### Best Practices

**1. Handle Race Conditions:**
```tsx
// Add timestamps to resolve conflicts
interface SyncData<T> {
  value: T;
  timestamp: number;
}

function syncWithConflictResolution<T>(key: string, value: T) {
  const data: SyncData<T> = {
    value,
    timestamp: Date.now(),
  };

  const existing = localStorage.getItem(key);
  if (existing) {
    const existingData: SyncData<T> = JSON.parse(existing);
    // Only update if newer
    if (data.timestamp > existingData.timestamp) {
      localStorage.setItem(key, JSON.stringify(data));
    }
  } else {
    localStorage.setItem(key, JSON.stringify(data));
  }
}
```

**2. Debounce Frequent Updates:**
```tsx
import { debounce } from 'lodash';

const debouncedSync = debounce((key: string, value: any) => {
  localStorage.setItem(key, JSON.stringify(value));
}, 300);
```

**3. Validate Synced Data:**
```tsx
function validateAndSync<T>(key: string, value: T, schema: any) {
  try {
    const validated = schema.parse(value);
    localStorage.setItem(key, JSON.stringify(validated));
  } catch (error) {
    console.error('Invalid data for sync:', error);
  }
}
```

**4. Handle Storage Quota:**
```tsx
try {
  localStorage.setItem(key, JSON.stringify(value));
} catch (error) {
  if (error.name === 'QuotaExceededError') {
    // Clear old data or use IndexedDB
    console.error('Storage quota exceeded');
  }
}
```

**5. Clean Up on Unmount:**
```tsx
useEffect(() => {
  const channel = new BroadcastChannel('sync');
  
  return () => {
    channel.close(); // Important!
  };
}, []);
```

---

#### Common Pitfalls

**❌ Storage Event Doesn't Fire in Same Tab:**
```tsx
// Storage event only fires in OTHER tabs
window.addEventListener('storage', handler); // Won't fire for same tab changes
```

**✅ Solution - Manual Event Dispatch:**
```tsx
const updateAndSync = (value: any) => {
  localStorage.setItem(key, JSON.stringify(value));
  // Manually trigger for same tab
  window.dispatchEvent(new Event('local-storage-change'));
};
```

**❌ Not Handling JSON Errors:**
```tsx
const data = JSON.parse(e.newValue); // Can throw
```

**✅ Solution - Try-Catch:**
```tsx
try {
  const data = JSON.parse(e.newValue);
  setState(data);
} catch (error) {
  console.error('Failed to parse:', error);
}
```

---

#### Interview Tips

✅ **Key Points:**
- Cross-tab sync keeps multiple tabs in sync
- localStorage + storage event is most common method
- BroadcastChannel API is modern, efficient alternative
- Storage event doesn't fire in same tab (need workaround)
- Important for auth, cart, notifications

✅ **When to Mention:**
- Multi-tab applications
- Authentication systems
- Shopping carts
- Real-time collaboration
- Offline-first applications

✅ **Common Follow-ups:**
- "What are the limitations of localStorage?"
- "How do you handle race conditions?"
- "What's BroadcastChannel and when to use it?"
- "How do you sync across devices, not just tabs?"
- "What about SharedWorker?"

✅ **Perfect Answer Structure:**
1. Define: Cross-tab sync keeps state consistent across tabs
2. Why: Auth, cart, notifications need to sync
3. Methods: localStorage + storage event (common), BroadcastChannel (modern)
4. Example: Show cart sync with storage event listener
5. Gotcha: Storage event doesn't fire in same tab
6. Alternative: BroadcastChannel for better performance

</details>

---

### 119. What is Redux Saga and when do you use it?

<details>
<summary>View Answer</summary>

**Redux Saga** is a Redux middleware library that makes handling side effects (async operations, API calls, etc.) easier using ES6 Generators. It provides a powerful, testable way to manage complex async flows, race conditions, and cancellations.

#### What is Redux Saga?

- **Generator-based** - uses `function*` and `yield`
- **Side effects management** - handles async operations
- **Declarative** - effects are plain objects
- **Testable** - easy to test without mocking
- **Powerful patterns** - race, debounce, retry, cancel
- **Centralized logic** - keeps components clean

**vs. Redux Thunk:**
- Thunk: Simple, callback-based
- Saga: Advanced, generator-based, more powerful

---

#### 1. Basic Redux Saga Setup

**Install:**
```bash
npm install redux-saga
```

**Setup:**
```tsx
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import rootSaga from './sagas';
import rootReducer from './reducers';

// Create saga middleware
const sagaMiddleware = createSagaMiddleware();

// Create store with saga middleware
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ thunk: false }).concat(sagaMiddleware),
});

// Run root saga
sagaMiddleware.run(rootSaga);

export default store;
```

---

#### 2. Basic Saga Example

**Actions:**
```tsx
// actions/users.ts
export const FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST';
export const FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS';
export const FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE';

export const fetchUsersRequest = () => ({
  type: FETCH_USERS_REQUEST,
});

export const fetchUsersSuccess = (users: User[]) => ({
  type: FETCH_USERS_SUCCESS,
  payload: users,
});

export const fetchUsersFailure = (error: string) => ({
  type: FETCH_USERS_FAILURE,
  payload: error,
});
```

**Saga:**
```tsx
// sagas/userSaga.ts
import { call, put, takeLatest } from 'redux-saga/effects';
import {
  FETCH_USERS_REQUEST,
  fetchUsersSuccess,
  fetchUsersFailure,
} from '../actions/users';
import api from '../services/api';

interface User {
  id: string;
  name: string;
  email: string;
}

// Worker saga: handles the async operation
function* fetchUsersSaga() {
  try {
    // call() makes the API request
    const users: User[] = yield call(api.getUsers);
    
    // put() dispatches an action
    yield put(fetchUsersSuccess(users));
  } catch (error) {
    yield put(fetchUsersFailure(error.message));
  }
}

// Watcher saga: watches for FETCH_USERS_REQUEST action
export function* watchFetchUsers() {
  // takeLatest cancels previous request if new one starts
  yield takeLatest(FETCH_USERS_REQUEST, fetchUsersSaga);
}
```

**Root Saga:**
```tsx
// sagas/index.ts
import { all, fork } from 'redux-saga/effects';
import { watchFetchUsers } from './userSaga';
import { watchCreateUser } from './createUserSaga';

export default function* rootSaga() {
  yield all([
    fork(watchFetchUsers),
    fork(watchCreateUser),
  ]);
}
```

**Reducer:**
```tsx
// reducers/usersReducer.ts
import {
  FETCH_USERS_REQUEST,
  FETCH_USERS_SUCCESS,
  FETCH_USERS_FAILURE,
} from '../actions/users';

interface UsersState {
  data: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UsersState = {
  data: [],
  loading: false,
  error: null,
};

export default function usersReducer(state = initialState, action: any) {
  switch (action.type) {
    case FETCH_USERS_REQUEST:
      return { ...state, loading: true, error: null };
    case FETCH_USERS_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_USERS_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}
```

**Component:**
```tsx
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsersRequest } from './actions/users';

function UserList() {
  const dispatch = useDispatch();
  const { data: users, loading, error } = useSelector(
    (state: RootState) => state.users
  );

  useEffect(() => {
    dispatch(fetchUsersRequest());
  }, [dispatch]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

#### 3. Advanced Saga Patterns

**Debouncing (Search Input):**
```tsx
import { call, put, debounce } from 'redux-saga/effects';
import { SEARCH_REQUEST, searchSuccess } from '../actions/search';

function* searchSaga(action: { type: string; payload: string }) {
  try {
    const results = yield call(api.search, action.payload);
    yield put(searchSuccess(results));
  } catch (error) {
    // Handle error
  }
}

export function* watchSearch() {
  // Debounce search by 500ms
  yield debounce(500, SEARCH_REQUEST, searchSaga);
}

// Usage in component:
// dispatch(searchRequest(query)); // Only sends request after 500ms of no typing
```

**Throttling:**
```tsx
import { call, put, throttle } from 'redux-saga/effects';
import { SAVE_DRAFT } from '../actions/draft';

function* saveDraftSaga(action: any) {
  yield call(api.saveDraft, action.payload);
}

export function* watchSaveDraft() {
  // Throttle to once per second
  yield throttle(1000, SAVE_DRAFT, saveDraftSaga);
}
```

**Race Condition (Timeout):**
```tsx
import { call, put, race, delay } from 'redux-saga/effects';

function* fetchWithTimeoutSaga() {
  try {
    // Race between API call and timeout
    const { response, timeout } = yield race({
      response: call(api.getUsers),
      timeout: delay(5000), // 5 second timeout
    });

    if (response) {
      yield put(fetchUsersSuccess(response));
    } else {
      yield put(fetchUsersFailure('Request timeout'));
    }
  } catch (error) {
    yield put(fetchUsersFailure(error.message));
  }
}
```

**Polling:**
```tsx
import { call, put, delay, cancel, fork, take } from 'redux-saga/effects';
import { START_POLLING, STOP_POLLING } from '../actions/polling';

function* pollDataSaga() {
  while (true) {
    try {
      const data = yield call(api.getData);
      yield put({ type: 'DATA_RECEIVED', payload: data });
      yield delay(3000); // Poll every 3 seconds
    } catch (error) {
      // Handle error
    }
  }
}

export function* watchPolling() {
  while (true) {
    yield take(START_POLLING);
    const pollTask = yield fork(pollDataSaga);
    yield take(STOP_POLLING);
    yield cancel(pollTask); // Cancel polling
  }
}
```

**Retry Logic:**
```tsx
import { call, put, retry } from 'redux-saga/effects';

function* fetchWithRetrySaga() {
  try {
    // Retry up to 3 times with 2 second delay
    const users = yield retry(3, 2000, api.getUsers);
    yield put(fetchUsersSuccess(users));
  } catch (error) {
    yield put(fetchUsersFailure('Failed after 3 retries'));
  }
}
```

**Cancellation:**
```tsx
import { call, put, take, cancel, fork } from 'redux-saga/effects';
import { FETCH_START, FETCH_CANCEL } from '../actions';

function* fetchSaga() {
  try {
    const data = yield call(api.fetchData);
    yield put({ type: 'FETCH_SUCCESS', payload: data });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}

export function* watchFetchWithCancel() {
  while (true) {
    yield take(FETCH_START);
    const task = yield fork(fetchSaga);
    yield take(FETCH_CANCEL);
    yield cancel(task); // Cancel ongoing request
  }
}
```

**Parallel Requests:**
```tsx
import { call, put, all } from 'redux-saga/effects';

function* fetchAllDataSaga() {
  try {
    // Run multiple requests in parallel
    const [users, posts, comments] = yield all([
      call(api.getUsers),
      call(api.getPosts),
      call(api.getComments),
    ]);

    yield put({
      type: 'FETCH_ALL_SUCCESS',
      payload: { users, posts, comments },
    });
  } catch (error) {
    yield put({ type: 'FETCH_ALL_FAILURE', payload: error.message });
  }
}
```

**Sequential Requests (Dependent):**
```tsx
function* fetchUserAndPostsSaga(action: { payload: string }) {
  try {
    // First get user
    const user = yield call(api.getUser, action.payload);
    yield put({ type: 'USER_SUCCESS', payload: user });

    // Then get user's posts (depends on user ID)
    const posts = yield call(api.getUserPosts, user.id);
    yield put({ type: 'POSTS_SUCCESS', payload: posts });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}
```

---

#### 4. Real-World Example: Login Flow

```tsx
import { call, put, takeLatest } from 'redux-saga/effects';
import { push } from 'connected-react-router';
import {
  LOGIN_REQUEST,
  loginSuccess,
  loginFailure,
} from '../actions/auth';
import api from '../services/api';

function* loginSaga(action: { payload: { email: string; password: string } }) {
  try {
    // Call login API
    const response = yield call(api.login, action.payload);

    // Store token
    localStorage.setItem('token', response.token);

    // Dispatch success
    yield put(loginSuccess(response.user));

    // Redirect to dashboard
    yield put(push('/dashboard'));

    // Show success notification
    yield put({
      type: 'SHOW_NOTIFICATION',
      payload: { message: 'Login successful', type: 'success' },
    });
  } catch (error) {
    yield put(loginFailure(error.message));
    yield put({
      type: 'SHOW_NOTIFICATION',
      payload: { message: 'Login failed', type: 'error' },
    });
  }
}

export function* watchLogin() {
  yield takeLatest(LOGIN_REQUEST, loginSaga);
}
```

---

#### 5. Testing Sagas

```tsx
import { call, put } from 'redux-saga/effects';
import { fetchUsersSaga } from './userSaga';
import { fetchUsersSuccess, fetchUsersFailure } from '../actions/users';
import api from '../services/api';

describe('fetchUsersSaga', () => {
  it('should fetch users successfully', () => {
    const generator = fetchUsersSaga();

    // Test call effect
    expect(generator.next().value).toEqual(call(api.getUsers));

    // Mock API response
    const mockUsers = [{ id: '1', name: 'John' }];

    // Test put effect
    expect(generator.next(mockUsers).value).toEqual(
      put(fetchUsersSuccess(mockUsers))
    );

    // Test completion
    expect(generator.next().done).toBe(true);
  });

  it('should handle errors', () => {
    const generator = fetchUsersSaga();

    generator.next();

    // Test error handling
    const error = new Error('API Error');
    expect(generator.throw(error).value).toEqual(
      put(fetchUsersFailure('API Error'))
    );
  });
});
```

---

#### When to Use Redux Saga

**✅ Use Redux Saga When:**

1. **Complex Async Flows:**
   - Multiple dependent API calls
   - Complex error handling and retries
   - Need to cancel ongoing requests

2. **Advanced Patterns:**
   - Debouncing/throttling
   - Polling
   - Race conditions
   - Optimistic updates with rollback

3. **Side Effects Coordination:**
   - Need to listen to multiple actions
   - Complex business logic
   - Centralized side effects management

4. **Testability:**
   - Need pure, easily testable side effects
   - No mocking required

**❌ Don't Use Redux Saga When:**

1. **Simple Use Cases:**
   - Basic API calls
   - Simple async operations
   - Small applications

2. **Team Unfamiliarity:**
   - Team not comfortable with generators
   - Steeper learning curve

3. **Modern Alternatives:**
   - RTK Query handles your use case
   - React Query is sufficient
   - Redux Toolkit's `createAsyncThunk` is enough

---

#### Redux Saga Effects Reference

| Effect | Description | Example |
|--------|-------------|---------|
| **call** | Call function | `yield call(api.getUsers)` |
| **put** | Dispatch action | `yield put(action)` |
| **take** | Wait for action | `yield take(ACTION_TYPE)` |
| **takeLatest** | Cancel previous, run latest | `yield takeLatest(TYPE, saga)` |
| **takeEvery** | Run for every action | `yield takeEvery(TYPE, saga)` |
| **fork** | Non-blocking call | `yield fork(saga)` |
| **spawn** | Detached fork | `yield spawn(saga)` |
| **cancel** | Cancel task | `yield cancel(task)` |
| **select** | Get state | `yield select(selector)` |
| **all** | Run parallel | `yield all([saga1, saga2])` |
| **race** | Race conditions | `yield race({ a: saga1, b: saga2 })` |
| **delay** | Delay execution | `yield delay(1000)` |
| **debounce** | Debounce calls | `yield debounce(500, TYPE, saga)` |
| **throttle** | Throttle calls | `yield throttle(1000, TYPE, saga)` |
| **retry** | Retry on failure | `yield retry(3, 2000, api.call)` |

---

#### Saga vs Thunk Comparison

**Redux Thunk:**
```tsx
export const fetchUsers = () => async (dispatch: Dispatch) => {
  dispatch({ type: 'FETCH_REQUEST' });
  try {
    const users = await api.getUsers();
    dispatch({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    dispatch({ type: 'FETCH_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
function* fetchUsersSaga() {
  yield put({ type: 'FETCH_REQUEST' });
  try {
    const users = yield call(api.getUsers);
    yield put({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}

function* watchFetchUsers() {
  yield takeLatest('FETCH_USERS', fetchUsersSaga);
}
```

| Feature | Redux Thunk | Redux Saga |
|---------|-------------|------------|
| **Syntax** | async/await | Generators |
| **Complexity** | Simple | Advanced |
| **Testing** | Requires mocking | No mocking needed |
| **Cancellation** | Manual | Built-in |
| **Debounce/Throttle** | Manual | Built-in |
| **Race Conditions** | Manual | Built-in |
| **Learning Curve** | Easy | Steep |
| **Bundle Size** | Tiny | Larger |
| **Use Case** | Simple async | Complex flows |

---

#### Best Practices

**1. Keep Sagas Pure:**
```tsx
// GOOD - use call() for side effects
function* saga() {
  const data = yield call(api.getData);
}

// BAD - direct API call
function* saga() {
  const data = await api.getData(); // Don't use await in sagas
}
```

**2. Use Typed Effects:**
```tsx
import { CallEffect, PutEffect } from 'redux-saga/effects';

function* saga(): Generator<
  CallEffect | PutEffect,
  void,
  User[]
> {
  const users = yield call(api.getUsers);
  yield put(fetchUsersSuccess(users));
}
```

**3. Handle Errors Properly:**
```tsx
function* saga() {
  try {
    const data = yield call(api.getData);
    yield put(success(data));
  } catch (error) {
    yield put(failure(error.message));
    // Log to error tracking
    yield call(errorTracker.log, error);
  }
}
```

**4. Use Select for State:**
```tsx
import { select } from 'redux-saga/effects';

function* saga() {
  const token = yield select((state: RootState) => state.auth.token);
  const data = yield call(api.getData, token);
}
```

---

#### Interview Tips

✅ **Key Points:**
- Redux Saga uses generators for side effects
- More powerful than Redux Thunk for complex async
- Built-in patterns: debounce, throttle, race, cancel, retry
- Easier to test (no mocking needed)
- Steeper learning curve

✅ **When to Mention:**
- Complex async flows
- Need cancellation or debouncing
- Testing side effects
- Coordinating multiple async operations
- Comparing Redux middleware options

✅ **Common Follow-ups:**
- "Saga vs Thunk - which to use?"
- "How do you test sagas?"
- "What are generator functions?"
- "When would you use takeLatest vs takeEvery?"
- "How do you handle cancellation?"

✅ **Perfect Answer Structure:**
1. Define: Redux Saga = generator-based middleware for side effects
2. Why: Better than Thunk for complex async flows
3. Features: Debounce, cancel, retry, race built-in
4. Example: Show debounced search or cancellation
5. When to use: Complex flows, need advanced patterns
6. vs Thunk: Saga more powerful but steeper learning curve

</details>

---

### 120. What is Redux Thunk vs Redux Saga?

<details>
<summary>View Answer</summary>

**Redux Thunk** and **Redux Saga** are both Redux middleware for handling side effects (async operations), but they differ significantly in approach, complexity, and capabilities. Thunk is simpler and uses promises/async-await, while Saga is more powerful and uses ES6 generators.

#### Quick Comparison

| Feature | Redux Thunk | Redux Saga |
|---------|-------------|------------|
| **Syntax** | async/await, Promises | Generators (`function*`) |
| **Complexity** | Simple | Complex |
| **Learning Curve** | Easy | Steep |
| **Testing** | Requires mocking | Declarative, no mocking |
| **Cancellation** | Manual | Built-in |
| **Debouncing** | Manual (lodash) | Built-in |
| **Throttling** | Manual | Built-in |
| **Race Conditions** | Manual handling | Built-in `race()` |
| **Parallel Requests** | `Promise.all()` | `all()` effect |
| **Sequential Requests** | `await` | `yield` |
| **Bundle Size** | ~1KB | ~8KB |
| **Best For** | Simple async | Complex flows |
| **Code Location** | Action creators | Separate sagas |
| **Dispatch** | Return function | Yield `put()` |

---

#### 1. Redux Thunk - The Basics

**What is Redux Thunk?**
- Middleware that lets action creators return **functions** instead of objects
- Uses standard JavaScript async/await or Promises
- Simplest way to handle async in Redux

**Installation:**
```bash
npm install redux-thunk
```

**Setup:**
```tsx
import { configureStore } from '@reduxjs/toolkit';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware(), // thunk included by default
});
```

**Basic Example:**
```tsx
// Action types
const FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST';
const FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS';
const FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE';

// Thunk action creator
export const fetchUsers = () => {
  return async (dispatch: Dispatch) => {
    dispatch({ type: FETCH_USERS_REQUEST });
    
    try {
      const response = await fetch('/api/users');
      const users = await response.json();
      dispatch({ type: FETCH_USERS_SUCCESS, payload: users });
    } catch (error) {
      dispatch({ type: FETCH_USERS_FAILURE, payload: error.message });
    }
  };
};

// Usage in component
function UserList() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUsers());
  }, [dispatch]);
}
```

---

#### 2. Redux Saga - The Basics

**What is Redux Saga?**
- Middleware that uses **generators** to handle side effects
- More powerful and testable than Thunk
- Declarative approach to async operations

**Installation:**
```bash
npm install redux-saga
```

**Setup:**
```tsx
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import rootSaga from './sagas';
import rootReducer from './reducers';

const sagaMiddleware = createSagaMiddleware();

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ thunk: false }).concat(sagaMiddleware),
});

sagaMiddleware.run(rootSaga);
```

**Basic Example:**
```tsx
import { call, put, takeLatest } from 'redux-saga/effects';

// Action types
const FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST';
const FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS';
const FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE';

// Worker saga
function* fetchUsersSaga() {
  try {
    const users = yield call(fetch, '/api/users');
    const data = yield call([users, 'json']);
    yield put({ type: FETCH_USERS_SUCCESS, payload: data });
  } catch (error) {
    yield put({ type: FETCH_USERS_FAILURE, payload: error.message });
  }
}

// Watcher saga
function* watchFetchUsers() {
  yield takeLatest(FETCH_USERS_REQUEST, fetchUsersSaga);
}

// Root saga
export default function* rootSaga() {
  yield all([
    watchFetchUsers(),
  ]);
}

// Usage in component (same as Thunk)
function UserList() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch({ type: FETCH_USERS_REQUEST });
  }, [dispatch]);
}
```

---

#### 3. Side-by-Side Comparison: Real Examples

**Example 1: Simple API Call**

**Redux Thunk:**
```tsx
export const fetchUser = (userId: string) => async (dispatch: Dispatch) => {
  dispatch({ type: 'FETCH_USER_REQUEST' });
  
  try {
    const response = await fetch(`/api/users/${userId}`);
    const user = await response.json();
    dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    dispatch({ type: 'FETCH_USER_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
function* fetchUserSaga(action: { payload: string }) {
  try {
    const response = yield call(fetch, `/api/users/${action.payload}`);
    const user = yield call([response, 'json']);
    yield put({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    yield put({ type: 'FETCH_USER_FAILURE', payload: error.message });
  }
}

function* watchFetchUser() {
  yield takeLatest('FETCH_USER_REQUEST', fetchUserSaga);
}
```

---

**Example 2: Sequential API Calls (Dependent Requests)**

**Redux Thunk:**
```tsx
export const fetchUserAndPosts = (userId: string) => async (dispatch: Dispatch) => {
  try {
    // First request
    const userResponse = await fetch(`/api/users/${userId}`);
    const user = await userResponse.json();
    dispatch({ type: 'USER_SUCCESS', payload: user });
    
    // Second request depends on first
    const postsResponse = await fetch(`/api/users/${user.id}/posts`);
    const posts = await postsResponse.json();
    dispatch({ type: 'POSTS_SUCCESS', payload: posts });
  } catch (error) {
    dispatch({ type: 'FETCH_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
function* fetchUserAndPostsSaga(action: { payload: string }) {
  try {
    // First request
    const userResponse = yield call(fetch, `/api/users/${action.payload}`);
    const user = yield call([userResponse, 'json']);
    yield put({ type: 'USER_SUCCESS', payload: user });
    
    // Second request depends on first
    const postsResponse = yield call(fetch, `/api/users/${user.id}/posts`);
    const posts = yield call([postsResponse, 'json']);
    yield put({ type: 'POSTS_SUCCESS', payload: posts });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}
```

---

**Example 3: Parallel API Calls**

**Redux Thunk:**
```tsx
export const fetchAllData = () => async (dispatch: Dispatch) => {
  try {
    dispatch({ type: 'FETCH_ALL_REQUEST' });
    
    // Run in parallel
    const [users, posts, comments] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json()),
      fetch('/api/comments').then(r => r.json()),
    ]);
    
    dispatch({
      type: 'FETCH_ALL_SUCCESS',
      payload: { users, posts, comments },
    });
  } catch (error) {
    dispatch({ type: 'FETCH_ALL_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
function* fetchAllDataSaga() {
  try {
    yield put({ type: 'FETCH_ALL_REQUEST' });
    
    // Run in parallel
    const [usersResponse, postsResponse, commentsResponse] = yield all([
      call(fetch, '/api/users'),
      call(fetch, '/api/posts'),
      call(fetch, '/api/comments'),
    ]);
    
    const [users, posts, comments] = yield all([
      call([usersResponse, 'json']),
      call([postsResponse, 'json']),
      call([commentsResponse, 'json']),
    ]);
    
    yield put({
      type: 'FETCH_ALL_SUCCESS',
      payload: { users, posts, comments },
    });
  } catch (error) {
    yield put({ type: 'FETCH_ALL_FAILURE', payload: error.message });
  }
}
```

---

**Example 4: Debouncing (Search)**

**Redux Thunk (Manual Debounce):**
```tsx
import debounce from 'lodash/debounce';

// Create debounced function outside component
const debouncedSearch = debounce(async (dispatch: Dispatch, query: string) => {
  try {
    const response = await fetch(`/api/search?q=${query}`);
    const results = await response.json();
    dispatch({ type: 'SEARCH_SUCCESS', payload: results });
  } catch (error) {
    dispatch({ type: 'SEARCH_FAILURE', payload: error.message });
  }
}, 500);

export const searchProducts = (query: string) => (dispatch: Dispatch) => {
  dispatch({ type: 'SEARCH_REQUEST' });
  debouncedSearch(dispatch, query);
};
```

**Redux Saga (Built-in Debounce):**
```tsx
import { call, put, debounce } from 'redux-saga/effects';

function* searchSaga(action: { payload: string }) {
  try {
    const response = yield call(fetch, `/api/search?q=${action.payload}`);
    const results = yield call([response, 'json']);
    yield put({ type: 'SEARCH_SUCCESS', payload: results });
  } catch (error) {
    yield put({ type: 'SEARCH_FAILURE', payload: error.message });
  }
}

function* watchSearch() {
  // Automatically debounces by 500ms
  yield debounce(500, 'SEARCH_REQUEST', searchSaga);
}
```

---

**Example 5: Request Cancellation**

**Redux Thunk (Manual with AbortController):**
```tsx
let abortController: AbortController | null = null;

export const fetchUsers = () => async (dispatch: Dispatch) => {
  // Cancel previous request
  if (abortController) {
    abortController.abort();
  }
  
  abortController = new AbortController();
  
  try {
    dispatch({ type: 'FETCH_REQUEST' });
    
    const response = await fetch('/api/users', {
      signal: abortController.signal,
    });
    const users = await response.json();
    
    dispatch({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    if (error.name !== 'AbortError') {
      dispatch({ type: 'FETCH_FAILURE', payload: error.message });
    }
  }
};
```

**Redux Saga (Built-in Cancellation):**
```tsx
import { call, put, takeLatest } from 'redux-saga/effects';

function* fetchUsersSaga() {
  try {
    yield put({ type: 'FETCH_REQUEST' });
    const response = yield call(fetch, '/api/users');
    const users = yield call([response, 'json']);
    yield put({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}

function* watchFetchUsers() {
  // takeLatest automatically cancels previous request
  yield takeLatest('FETCH_USERS', fetchUsersSaga);
}
```

---

**Example 6: Race Condition (Timeout)**

**Redux Thunk:**
```tsx
const timeout = (ms: number) => new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Timeout')), ms)
);

export const fetchWithTimeout = () => async (dispatch: Dispatch) => {
  try {
    const response = await Promise.race([
      fetch('/api/users'),
      timeout(5000),
    ]);
    const users = await response.json();
    dispatch({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    dispatch({ type: 'FETCH_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
import { call, put, race, delay } from 'redux-saga/effects';

function* fetchWithTimeoutSaga() {
  try {
    const { response, timeout } = yield race({
      response: call(fetch, '/api/users'),
      timeout: delay(5000),
    });
    
    if (response) {
      const users = yield call([response, 'json']);
      yield put({ type: 'FETCH_SUCCESS', payload: users });
    } else {
      yield put({ type: 'FETCH_FAILURE', payload: 'Timeout' });
    }
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: error.message });
  }
}
```

---

**Example 7: Polling**

**Redux Thunk:**
```tsx
let pollingInterval: NodeJS.Timeout | null = null;

export const startPolling = () => async (dispatch: Dispatch) => {
  const poll = async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      dispatch({ type: 'DATA_RECEIVED', payload: data });
    } catch (error) {
      console.error('Polling error:', error);
    }
  };
  
  await poll(); // Initial fetch
  pollingInterval = setInterval(poll, 3000);
};

export const stopPolling = () => (dispatch: Dispatch) => {
  if (pollingInterval) {
    clearInterval(pollingInterval);
    pollingInterval = null;
  }
};
```

**Redux Saga:**
```tsx
import { call, put, delay, cancel, fork, take } from 'redux-saga/effects';

function* pollDataSaga() {
  while (true) {
    try {
      const response = yield call(fetch, '/api/data');
      const data = yield call([response, 'json']);
      yield put({ type: 'DATA_RECEIVED', payload: data });
      yield delay(3000);
    } catch (error) {
      console.error('Polling error:', error);
    }
  }
}

function* watchPolling() {
  while (true) {
    yield take('START_POLLING');
    const pollTask = yield fork(pollDataSaga);
    yield take('STOP_POLLING');
    yield cancel(pollTask);
  }
}
```

---

**Example 8: Retry Logic**

**Redux Thunk:**
```tsx
const retry = async <T,>(
  fn: () => Promise<T>,
  retriesLeft = 3,
  interval = 1000
): Promise<T> => {
  try {
    return await fn();
  } catch (error) {
    if (retriesLeft === 0) throw error;
    await new Promise(resolve => setTimeout(resolve, interval));
    return retry(fn, retriesLeft - 1, interval);
  }
};

export const fetchWithRetry = () => async (dispatch: Dispatch) => {
  try {
    const response = await retry(
      () => fetch('/api/users').then(r => r.json()),
      3,
      2000
    );
    dispatch({ type: 'FETCH_SUCCESS', payload: response });
  } catch (error) {
    dispatch({ type: 'FETCH_FAILURE', payload: error.message });
  }
};
```

**Redux Saga:**
```tsx
import { call, put, retry } from 'redux-saga/effects';

function* fetchWithRetrySaga() {
  try {
    // Retry 3 times with 2 second delay
    const response = yield retry(
      3,
      2000,
      fetch,
      '/api/users'
    );
    const users = yield call([response, 'json']);
    yield put({ type: 'FETCH_SUCCESS', payload: users });
  } catch (error) {
    yield put({ type: 'FETCH_FAILURE', payload: 'Failed after 3 retries' });
  }
}
```

---

#### 4. Testing Comparison

**Testing Redux Thunk:**
```tsx
import configureMockStore from 'redux-mock-store';
import thunk from 'redux-thunk';
import { fetchUsers } from './actions';

const mockStore = configureMockStore([thunk]);

describe('fetchUsers thunk', () => {
  it('should fetch users successfully', async () => {
    // Mock fetch
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve([{ id: 1, name: 'John' }]),
      })
    ) as jest.Mock;

    const store = mockStore({});
    await store.dispatch(fetchUsers());

    const actions = store.getActions();
    expect(actions[0]).toEqual({ type: 'FETCH_USERS_REQUEST' });
    expect(actions[1]).toEqual({
      type: 'FETCH_USERS_SUCCESS',
      payload: [{ id: 1, name: 'John' }],
    });
  });
});
```

**Testing Redux Saga (No Mocking!):**
```tsx
import { call, put } from 'redux-saga/effects';
import { fetchUsersSaga } from './sagas';

describe('fetchUsersSaga', () => {
  it('should fetch users successfully', () => {
    const generator = fetchUsersSaga();

    // Test call effect
    expect(generator.next().value).toEqual(call(fetch, '/api/users'));

    // Mock response
    const mockResponse = { json: () => [{ id: 1, name: 'John' }] };
    expect(generator.next(mockResponse).value).toEqual(
      call([mockResponse, 'json'])
    );

    // Test put effect
    const mockUsers = [{ id: 1, name: 'John' }];
    expect(generator.next(mockUsers).value).toEqual(
      put({ type: 'FETCH_USERS_SUCCESS', payload: mockUsers })
    );

    // Test done
    expect(generator.next().done).toBe(true);
  });
});
```

**Winner: Saga** - Testing is much easier without mocking!

---

#### 5. When to Use Each

**Use Redux Thunk When:**

✅ **Simple async operations**
- Basic API calls
- Straightforward error handling
- No complex flows

✅ **Small applications**
- Quick prototypes
- MVPs
- Small teams

✅ **Team familiarity**
- Team knows async/await
- Don't want to learn generators
- Want minimal setup

✅ **Bundle size matters**
- Every KB counts
- Simple use cases

**Use Redux Saga When:**

✅ **Complex async flows**
- Multiple dependent requests
- Complex error handling
- Need cancellation

✅ **Advanced patterns needed**
- Debouncing/throttling
- Polling
- Race conditions
- Retry logic

✅ **Testability is critical**
- Need pure, declarative tests
- Don't want to mock everything
- Complex business logic

✅ **Large applications**
- Many async operations
- Complex side effects
- Need centralized logic

---

#### 6. Migration from Thunk to Saga

**Before (Thunk):**
```tsx
export const fetchUser = (id: string) => async (dispatch: Dispatch) => {
  dispatch({ type: 'FETCH_USER_REQUEST' });
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    dispatch({ type: 'FETCH_USER_FAILURE', payload: error.message });
  }
};

// Component
dispatch(fetchUser('123'));
```

**After (Saga):**
```tsx
// Saga
function* fetchUserSaga(action: { payload: string }) {
  yield put({ type: 'FETCH_USER_REQUEST' });
  try {
    const response = yield call(fetch, `/api/users/${action.payload}`);
    const user = yield call([response, 'json']);
    yield put({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    yield put({ type: 'FETCH_USER_FAILURE', payload: error.message });
  }
}

function* watchFetchUser() {
  yield takeLatest('FETCH_USER', fetchUserSaga);
}

// Component
dispatch({ type: 'FETCH_USER', payload: '123' });
```

---

#### 7. Modern Alternative: RTK Query

**Instead of Thunk or Saga, consider RTK Query:**

```tsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      query: () => '/users',
    }),
    getUser: builder.query<User, string>({
      query: (id) => `/users/${id}`,
    }),
  }),
});

// Auto-generated hooks
export const { useGetUsersQuery, useGetUserQuery } = api;

// Component - much simpler!
function UserList() {
  const { data: users, isLoading } = useGetUsersQuery();
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <ul>
      {users?.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**Benefits:**
- No need to write thunks or sagas
- Automatic caching
- Auto-generated hooks
- Less boilerplate

---

#### Best Practices

**Redux Thunk:**
1. Keep thunks simple and focused
2. Extract reusable async logic
3. Use TypeScript for type safety
4. Handle errors consistently
5. Consider migration to RTK Query

**Redux Saga:**
1. Keep sagas pure (use `call` for side effects)
2. Use typed effects for TypeScript
3. Handle errors in try-catch
4. Use `select` to access state
5. Document complex flows

---

#### Summary

**Redux Thunk:**
- ✅ Simple, easy to learn
- ✅ Uses familiar async/await
- ✅ Small bundle size
- ❌ Manual patterns (debounce, cancel)
- ❌ Testing requires mocking
- ❌ Less powerful for complex flows

**Redux Saga:**
- ✅ Powerful for complex async
- ✅ Built-in patterns (debounce, race, retry)
- ✅ Easy to test (no mocking)
- ✅ Better for large apps
- ❌ Steeper learning curve
- ❌ Larger bundle size
- ❌ Generators less familiar

**Modern Choice:**
- **RTK Query** - Best for most new projects
- **Thunk** - Simple use cases, quick setup
- **Saga** - Complex flows, need advanced patterns

---

#### Interview Tips

✅ **Key Points:**
- Thunk: Simple, async/await, good for basic use cases
- Saga: Complex, generators, advanced patterns built-in
- Thunk easier to learn, Saga more powerful
- Saga better for testing (no mocking)
- Both handle async, different approaches

✅ **When to Mention:**
- Redux middleware comparison
- Handling async in Redux
- Complex async flows
- Testing side effects
- Migration strategies

✅ **Common Follow-ups:**
- "When would you choose Saga over Thunk?"
- "What are generators in JavaScript?"
- "How do you test each?"
- "What about RTK Query?"
- "Can you use both together?"

✅ **Perfect Answer Structure:**
1. Define both: Thunk (async/await) vs Saga (generators)
2. Key difference: Simple vs powerful
3. Thunk: Good for basic API calls, easy to learn
4. Saga: Advanced patterns, better testing, complex flows
5. Example: Show debouncing (manual vs built-in)
6. When to use: Thunk for simple, Saga for complex
7. Modern: Consider RTK Query instead

</details>
