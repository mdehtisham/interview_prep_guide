# React 19 Features

> Advanced / Senior Level (3-5 years)

---

## Questions

91. What are React Server Actions?
92. What is the use hook in React 19?
93. What are Asset Loading improvements in React 19?
94. What is the new useFormStatus hook?
95. What is useOptimistic hook?
96. What are Document Metadata features in React 19?
97. What are Actions in React 19?
98. What is the ref as prop feature in React 19?
99. How does React 19 improve hydration errors?
100. What are the Web Components improvements in React 19?

---

## Detailed Answers

### 91. What are React Server Actions?

<details>
<summary>View Answer</summary>

**React Server Actions**

A React 19 feature that allows you to define server-side functions that can be called directly from Client Components, enabling seamless client-server interactions without manual API routes.

---

## What are Server Actions?

**Server Actions = Functions that run on the server, callable from client**

### Key Characteristics

1. **Run on server only**
2. **Called from client components**
3. **No API routes needed**
4. **Type-safe by default**
5. **Built-in form integration**
6. **Progressive enhancement**
7. **Automatic serialization**

---

## Basic Example

### Defining a Server Action

```jsx
// actions.js
'use server';

import { db } from '@/lib/database';

export async function createPost(formData) {
  // This runs on the server!
  const title = formData.get('title');
  const content = formData.get('content');
  
  await db.post.create({
    data: { title, content }
  });
  
  return { success: true };
}
```

---

### Using in a Client Component

```jsx
// PostForm.jsx
'use client';

import { createPost } from './actions';

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

**That's it! No API routes, no fetch, no loading states!**

---

## The 'use server' Directive

### File-Level Directive

```jsx
// actions.js
'use server';

// All exports are Server Actions
export async function createUser(data) {
  // Runs on server
}

export async function deleteUser(id) {
  // Runs on server
}
```

---

### Function-Level Directive

```jsx
// Component.jsx
import { db } from '@/lib/database';

export default function Component() {
  // Single Server Action in file
  async function handleSubmit() {
    'use server';
    
    await db.user.create({ ... });
  }
  
  return <form action={handleSubmit}>...</form>;
}
```

---

## With Forms

### Traditional Approach

```jsx
// Old way - lots of boilerplate
'use client';

import { useState } from 'react';

function CreatePost() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  async function handleSubmit(e) {
    e.preventDefault();
    setLoading(true);
    setError(null);
    
    try {
      const formData = new FormData(e.target);
      const data = {
        title: formData.get('title'),
        content: formData.get('content')
      };
      
      const res = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      
      if (!res.ok) throw new Error('Failed');
      
      const result = await res.json();
      // Handle success
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="title" />
      <textarea name="content" />
      <button disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      {error && <div>{error}</div>}
    </form>
  );
}

// Also need API route:
// app/api/posts/route.js
export async function POST(req) {
  const data = await req.json();
  await db.post.create({ data });
  return Response.json({ success: true });
}
```

---

### With Server Actions

```jsx
// New way - much simpler!
'use client';

import { createPost } from './actions';

function CreatePost() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit">Create Post</button>
    </form>
  );
}

// actions.js
'use server';

export async function createPost(formData) {
  await db.post.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content')
    }
  });
}
```

**90% less code!**

---

## With useActionState Hook

### Getting Action State

```jsx
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

function PostForm() {
  const [state, action, isPending] = useActionState(
    createPost,
    { success: false, message: '' }
  );
  
  return (
    <form action={action}>
      <input name="title" />
      <textarea name="content" />
      
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Post'}
      </button>
      
      {state.success && (
        <div className="success">{state.message}</div>
      )}
    </form>
  );
}

// actions.js
'use server';

export async function createPost(prevState, formData) {
  const title = formData.get('title');
  const content = formData.get('content');
  
  await db.post.create({
    data: { title, content }
  });
  
  return {
    success: true,
    message: 'Post created successfully!'
  };
}
```

---

## Returning Data

### Server Action can return data

```jsx
// actions.js
'use server';

export async function createPost(formData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content')
    }
  });
  
  // Return data to client
  return {
    id: post.id,
    createdAt: post.createdAt
  };
}
```

---

### Using returned data

```jsx
'use client';

import { createPost } from './actions';
import { useActionState } from 'react';

function PostForm() {
  const [state, action] = useActionState(
    createPost,
    null
  );
  
  return (
    <div>
      <form action={action}>
        <input name="title" />
        <textarea name="content" />
        <button>Create</button>
      </form>
      
      {state && (
        <div>
          Post created with ID: {state.id}
          At: {state.createdAt}
        </div>
      )}
    </div>
  );
}
```

---

## Validation

### Server-Side Validation

```jsx
// actions.js
'use server';

import { z } from 'zod';

const PostSchema = z.object({
  title: z.string().min(3).max(100),
  content: z.string().min(10)
});

export async function createPost(prevState, formData) {
  // Validate on server
  const result = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content')
  });
  
  if (!result.success) {
    return {
      errors: result.error.flatten().fieldErrors,
      success: false
    };
  }
  
  // Create post
  await db.post.create({
    data: result.data
  });
  
  return {
    success: true,
    message: 'Post created!'
  };
}
```

---

### Displaying Validation Errors

```jsx
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

function PostForm() {
  const [state, action, isPending] = useActionState(
    createPost,
    { errors: {}, success: false }
  );
  
  return (
    <form action={action}>
      <div>
        <input name="title" />
        {state.errors?.title && (
          <span className="error">{state.errors.title}</span>
        )}
      </div>
      
      <div>
        <textarea name="content" />
        {state.errors?.content && (
          <span className="error">{state.errors.content}</span>
        )}
      </div>
      
      <button disabled={isPending}>Submit</button>
    </form>
  );
}
```

---

## Progressive Enhancement

**Server Actions work even without JavaScript!**

```jsx
// This form works with JavaScript disabled
<form action={createPost}>
  <input name="title" />
  <button>Submit</button>
</form>

// With JS: Submits via JavaScript
// Without JS: Traditional form submission
```

---

## Error Handling

### Throwing Errors

```jsx
'use server';

export async function createPost(formData) {
  const title = formData.get('title');
  
  if (!title) {
    throw new Error('Title is required');
  }
  
  await db.post.create({ data: { title } });
}
```

---

### Catching Errors

```jsx
'use client';

import { useState } from 'react';
import { createPost } from './actions';

function PostForm() {
  const [error, setError] = useState(null);
  
  async function handleSubmit(formData) {
    try {
      await createPost(formData);
      setError(null);
    } catch (err) {
      setError(err.message);
    }
  }
  
  return (
    <form action={handleSubmit}>
      <input name="title" />
      <button>Submit</button>
      {error && <div className="error">{error}</div>}
    </form>
  );
}
```

---

## Revalidation

### Revalidate Cache After Mutation

```jsx
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData) {
  await db.post.create({
    data: {
      title: formData.get('title')
    }
  });
  
  // Revalidate posts page
  revalidatePath('/posts');
}
```

---

### Redirect After Action

```jsx
'use server';

import { redirect } from 'next/navigation';

export async function createPost(formData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title')
    }
  });
  
  // Redirect to new post
  redirect(`/posts/${post.id}`);
}
```

---

## Calling from Event Handlers

### Outside Forms

```jsx
'use client';

import { useState } from 'react';
import { deletePost } from './actions';

function Post({ post }) {
  const [isPending, setIsPending] = useState(false);
  
  async function handleDelete() {
    setIsPending(true);
    await deletePost(post.id);
    setIsPending(false);
  }
  
  return (
    <div>
      <h3>{post.title}</h3>
      <button onClick={handleDelete} disabled={isPending}>
        {isPending ? 'Deleting...' : 'Delete'}
      </button>
    </div>
  );
}

// actions.js
'use server';

export async function deletePost(id) {
  await db.post.delete({ where: { id } });
  revalidatePath('/posts');
}
```

---

## Real-World Example: Blog Post Editor

### Server Actions File

```jsx
// app/actions.js
'use server';

import { db } from '@/lib/database';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const PostSchema = z.object({
  title: z.string().min(3).max(100),
  content: z.string().min(10),
  published: z.boolean().default(false)
});

export async function createPost(prevState, formData) {
  // Validate
  const result = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    published: formData.get('published') === 'on'
  });
  
  if (!result.success) {
    return {
      errors: result.error.flatten().fieldErrors,
      success: false
    };
  }
  
  // Create
  const post = await db.post.create({
    data: result.data
  });
  
  // Revalidate and redirect
  revalidatePath('/posts');
  redirect(`/posts/${post.id}`);
}

export async function updatePost(id, prevState, formData) {
  const result = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    published: formData.get('published') === 'on'
  });
  
  if (!result.success) {
    return {
      errors: result.error.flatten().fieldErrors,
      success: false
    };
  }
  
  await db.post.update({
    where: { id },
    data: result.data
  });
  
  revalidatePath(`/posts/${id}`);
  
  return {
    success: true,
    message: 'Post updated!'
  };
}

export async function deletePost(id) {
  await db.post.delete({ where: { id } });
  revalidatePath('/posts');
  redirect('/posts');
}

export async function togglePublished(id) {
  const post = await db.post.findUnique({ where: { id } });
  
  await db.post.update({
    where: { id },
    data: { published: !post.published }
  });
  
  revalidatePath(`/posts/${id}`);
}
```

---

### Create Post Form

```jsx
// app/posts/new/page.jsx
'use client';

import { useActionState } from 'react';
import { createPost } from '@/app/actions';

function NewPostPage() {
  const [state, action, isPending] = useActionState(
    createPost,
    { errors: {}, success: false }
  );
  
  return (
    <div className="container">
      <h1>Create New Post</h1>
      
      <form action={action} className="post-form">
        <div className="field">
          <label htmlFor="title">Title</label>
          <input
            id="title"
            name="title"
            type="text"
            required
          />
          {state.errors?.title && (
            <span className="error">{state.errors.title[0]}</span>
          )}
        </div>
        
        <div className="field">
          <label htmlFor="content">Content</label>
          <textarea
            id="content"
            name="content"
            rows={10}
            required
          />
          {state.errors?.content && (
            <span className="error">{state.errors.content[0]}</span>
          )}
        </div>
        
        <div className="field">
          <label>
            <input type="checkbox" name="published" />
            Publish immediately
          </label>
        </div>
        
        <button
          type="submit"
          disabled={isPending}
          className="btn-primary"
        >
          {isPending ? 'Creating...' : 'Create Post'}
        </button>
      </form>
    </div>
  );
}

export default NewPostPage;
```

---

### Edit Post Form

```jsx
// app/posts/[id]/edit/page.jsx
'use client';

import { useActionState } from 'react';
import { updatePost } from '@/app/actions';

function EditPostPage({ post }) {
  const updatePostWithId = updatePost.bind(null, post.id);
  
  const [state, action, isPending] = useActionState(
    updatePostWithId,
    { errors: {}, success: false }
  );
  
  return (
    <form action={action}>
      <input
        name="title"
        defaultValue={post.title}
      />
      {state.errors?.title && <span>{state.errors.title[0]}</span>}
      
      <textarea
        name="content"
        defaultValue={post.content}
      />
      {state.errors?.content && <span>{state.errors.content[0]}</span>}
      
      <label>
        <input
          type="checkbox"
          name="published"
          defaultChecked={post.published}
        />
        Published
      </label>
      
      <button disabled={isPending}>
        {isPending ? 'Saving...' : 'Save Changes'}
      </button>
      
      {state.success && (
        <div className="success">{state.message}</div>
      )}
    </form>
  );
}

export default EditPostPage;
```

---

### Post Actions Component

```jsx
// components/PostActions.jsx
'use client';

import { useState } from 'react';
import { deletePost, togglePublished } from '@/app/actions';

function PostActions({ post }) {
  const [isDeleting, setIsDeleting] = useState(false);
  const [isToggling, setIsToggling] = useState(false);
  
  async function handleDelete() {
    if (!confirm('Delete this post?')) return;
    
    setIsDeleting(true);
    await deletePost(post.id);
    // Redirects, so no need to set false
  }
  
  async function handleToggle() {
    setIsToggling(true);
    await togglePublished(post.id);
    setIsToggling(false);
  }
  
  return (
    <div className="post-actions">
      <button
        onClick={handleToggle}
        disabled={isToggling}
        className="btn-secondary"
      >
        {isToggling ? 'Updating...' : (
          post.published ? 'Unpublish' : 'Publish'
        )}
      </button>
      
      <button
        onClick={handleDelete}
        disabled={isDeleting}
        className="btn-danger"
      >
        {isDeleting ? 'Deleting...' : 'Delete'}
      </button>
    </div>
  );
}

export default PostActions;
```

---

## Optimistic Updates

### With useOptimistic

```jsx
'use client';

import { useOptimistic } from 'react';
import { likePost } from './actions';

function Post({ post }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (current, amount) => current + amount
  );
  
  async function handleLike() {
    // Immediately update UI
    addOptimisticLike(1);
    
    // Then update server
    await likePost(post.id);
  }
  
  return (
    <div>
      <h3>{post.title}</h3>
      <button onClick={handleLike}>
        ❤️ {optimisticLikes} likes
      </button>
    </div>
  );
}

// actions.js
'use server';

export async function likePost(id) {
  await db.post.update({
    where: { id },
    data: { likes: { increment: 1 } }
  });
  
  revalidatePath(`/posts/${id}`);
}
```

---

## Benefits

### 1. No API Routes Needed

**Before:**
- Create API route file
- Handle request/response
- Parse body
- Return JSON

**After:**
- Write function
- Done!

---

### 2. Type Safety

```typescript
// actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  //    ^ TypeScript knows the types
  
  return { id: 123 };  // Return type inferred
}

// Component.tsx
import { createPost } from './actions';

const result = await createPost(formData);
//    ^ TypeScript knows result has { id: number }
```

---

### 3. Automatic Serialization

```jsx
'use server';

export async function getUser(id) {
  const user = await db.user.findUnique({ where: { id } });
  
  // Return complex objects - automatically serialized
  return {
    ...user,
    createdAt: user.createdAt,  // Date serialized automatically
    settings: { ... }            // Nested objects work
  };
}
```

---

### 4. Progressive Enhancement

Forms work even without JavaScript enabled.

---

### 5. Built-in Form Integration

Direct integration with `<form>` element.

---

### 6. Simplified Error Handling

Errors automatically caught and handled.

---

## Security

### Server Actions are secure by default

```jsx
'use server';

import { cookies } from 'next/headers';

export async function deletePost(id) {
  // Check authentication
  const session = cookies().get('session');
  if (!session) {
    throw new Error('Unauthorized');
  }
  
  // Check authorization
  const post = await db.post.findUnique({ where: { id } });
  if (post.authorId !== session.userId) {
    throw new Error('Forbidden');
  }
  
  // Proceed with delete
  await db.post.delete({ where: { id } });
}
```

**Always validate on server!**

---

## Comparison

### Without Server Actions

```jsx
// 1. API Route
export async function POST(req) {
  const data = await req.json();
  const post = await db.post.create({ data });
  return Response.json(post);
}

// 2. Client Component
'use client';

async function handleSubmit(e) {
  e.preventDefault();
  const res = await fetch('/api/posts', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  const result = await res.json();
}

// Two files, more code, manual error handling
```

---

### With Server Actions

```jsx
// 1. Server Action
'use server';

export async function createPost(formData) {
  return await db.post.create({
    data: {
      title: formData.get('title')
    }
  });
}

// 2. Client Component
'use client';

import { createPost } from './actions';

<form action={createPost}>
  <input name="title" />
  <button>Submit</button>
</form>

// Simpler, cleaner, type-safe
```

---

## Best Practices

**1. Always validate on server**
```jsx
'use server';

export async function createPost(formData) {
  // Validate everything!
  const result = schema.safeParse(data);
  if (!result.success) return { errors: ... };
}
```

**2. Check authentication/authorization**
```jsx
'use server';

export async function deletePost(id) {
  const user = await getCurrentUser();
  if (!user) throw new Error('Unauthorized');
}
```

**3. Return useful error messages**
```jsx
return {
  success: false,
  errors: { title: 'Title is required' }
};
```

**4. Revalidate cache after mutations**
```jsx
revalidatePath('/posts');
```

**5. Use TypeScript for type safety**
```typescript
export async function createPost(formData: FormData): Promise<Result> {
  // ...
}
```

---

**Interview Tips:**
- **Server Actions** = server-side functions callable from client
- **'use server' directive** = marks function as Server Action
- **No API routes** = direct server calls
- **Works with forms** = `<form action={serverAction}>`
- **useActionState** = get action state and pending status
- **Progressive enhancement** = works without JavaScript
- **Type-safe** = TypeScript support built-in
- **Automatic serialization** = data automatically serialized
- **FormData** = receive form data directly
- **Return data** = can return data to client
- **Error handling** = throw errors or return error state
- **Validation** = server-side validation
- **Revalidation** = revalidatePath/revalidateTag after mutations
- **Redirect** = redirect() after actions
- **Security** = always validate and check auth
- **Optimistic updates** = combine with useOptimistic
- **Event handlers** = call from onClick, not just forms
- **React 19+** = new feature in React 19
- **Next.js** = fully supported in Next.js 14+
- **Reduces boilerplate** = 70-90% less code than API routes
- **Better DX** = simpler developer experience
- **File-level or function-level** = 'use server' placement
- **Client-server boundary** = clear separation
- **No manual fetch** = no need for fetch() calls
- **Pending states** = built-in with useActionState

</details>

---

### 92. What is the use hook in React 19?

<details>
<summary>View Answer</summary>

**The `use` Hook**

A new React 19 hook that reads the value of a resource like a Promise or Context during render, suspending the component if the resource isn't ready yet.

---

## What is the `use` Hook?

**`use` = Read Promises and Context in render**

### Key Features

1. **Read Promises** - await promises in render
2. **Read Context** - alternative to useContext
3. **Can be conditional** - unlike other hooks
4. **Can be in loops** - unlike other hooks
5. **Suspends component** - while promise resolves
6. **No effect needed** - read data directly

---

## Basic Syntax

```jsx
import { use } from 'react';

// With Promises
function Component() {
  const data = use(fetchDataPromise);
  // Component suspends until promise resolves
  return <div>{data}</div>;
}

// With Context
function Component() {
  const theme = use(ThemeContext);
  return <div>{theme}</div>;
}
```

---

## Using with Promises

### Traditional Approach (React 18)

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(data => {
        if (!cancelled) {
          setUser(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });
    
    return () => { cancelled = true; };
  }, [userId]);
  
  if (loading) return <Spinner />;
  if (error) return <Error error={error} />;
  
  return <div>{user.name}</div>;
}
```

---

### With `use` Hook (React 19)

```jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  // Component suspends until promise resolves
  const user = use(userPromise);
  
  return <div>{user.name}</div>;
}

// Usage
function App() {
  const userPromise = fetch('/api/users/1').then(r => r.json());
  
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**Much simpler! No loading states, no effects, no cleanup!**

---

## Key Difference: Promise as Prop

```jsx
// DON'T: Create promise in component
function Component() {
  const data = use(fetch('/api/data'));  // ❌ Creates new promise every render!
  return <div>{data}</div>;
}

// DO: Pass promise as prop
function Component({ dataPromise }) {
  const data = use(dataPromise);  // ✅ Reads the promise
  return <div>{data}</div>;
}

function Parent() {
  const dataPromise = fetch('/api/data');  // Create once
  
  return (
    <Suspense fallback={<Loading />}>
      <Component dataPromise={dataPromise} />
    </Suspense>
  );
}
```

---

## Using with Context

### Traditional useContext

```jsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

function Button() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

---

### With `use` Hook

```jsx
import { use } from 'react';
import { ThemeContext } from './ThemeContext';

function Button() {
  const theme = use(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**Same result, but `use` can be conditional!**

---

## Conditional Usage (Revolutionary!)

### Hooks Rules (Old)

```jsx
// ❌ Can't use hooks conditionally
function Component({ useTheme }) {
  if (useTheme) {
    const theme = useContext(ThemeContext);  // Error!
  }
}

// ❌ Can't use hooks in loops
function Component({ contexts }) {
  const values = contexts.map(ctx => useContext(ctx));  // Error!
}
```

---

### `use` Hook (New)

```jsx
import { use } from 'react';

// ✅ Conditional use is allowed!
function Component({ useTheme, ThemeContext }) {
  let theme;
  if (useTheme) {
    theme = use(ThemeContext);  // Works!
  }
  
  return <div>{theme || 'default'}</div>;
}

// ✅ use in loops is allowed!
function Component({ contexts }) {
  const values = contexts.map(ctx => use(ctx));  // Works!
  return <div>{values.join(', ')}</div>;
}
```

**This is a game changer!**

---

## Complete Example: Data Fetching

### Data Fetching Utilities

```jsx
// lib/fetch.js

// Cache for promises
const cache = new Map();

export function fetchUser(id) {
  const key = `user-${id}`;
  
  if (!cache.has(key)) {
    cache.set(
      key,
      fetch(`/api/users/${id}`).then(r => r.json())
    );
  }
  
  return cache.get(key);
}

export function fetchPosts(userId) {
  const key = `posts-${userId}`;
  
  if (!cache.has(key)) {
    cache.set(
      key,
      fetch(`/api/posts?userId=${userId}`).then(r => r.json())
    );
  }
  
  return cache.get(key);
}
```

---

### Components Using `use`

```jsx
// components/UserProfile.jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);
  
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}

// components/PostList.jsx
import { use } from 'react';

function PostList({ postsPromise }) {
  const posts = use(postsPromise);
  
  return (
    <div className="posts">
      {posts.map(post => (
        <article key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// app/users/[id]/page.jsx
import { Suspense } from 'react';
import { fetchUser, fetchPosts } from '@/lib/fetch';
import UserProfile from '@/components/UserProfile';
import PostList from '@/components/PostList';

function UserPage({ params }) {
  const userPromise = fetchUser(params.id);
  const postsPromise = fetchPosts(params.id);
  
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
      
      <Suspense fallback={<PostsSkeleton />}>
        <PostList postsPromise={postsPromise} />
      </Suspense>
    </div>
  );
}

export default UserPage;
```

**Data fetching is now declarative and composable!**

---

## Parallel Data Fetching

### Waterfall (Bad)

```jsx
import { use } from 'react';

function Component() {
  // Wait for user
  const user = use(fetchUser());
  
  // Then wait for posts (waterfall!)
  const posts = use(fetchPosts(user.id));
  
  return <div>...</div>;
}
```

---

### Parallel (Good)

```jsx
import { use } from 'react';

function Component({ userId }) {
  // Start both fetches immediately
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId);
  
  // Read them (happens in parallel)
  const user = use(userPromise);
  const posts = use(postsPromise);
  
  return <div>...</div>;
}
```

---

## Error Handling

### With Error Boundaries

```jsx
import { use } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function UserProfile({ userPromise }) {
  const user = use(userPromise);  // Throws if promise rejects
  return <div>{user.name}</div>;
}

function App() {
  const userPromise = fetchUser(1);
  
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Loading />}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

### Handling Errors Manually

```jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  let user;
  
  try {
    user = use(userPromise);
  } catch (error) {
    if (error instanceof Error) {
      return <div>Error: {error.message}</div>;
    }
    throw error;  // Re-throw if it's Suspense
  }
  
  return <div>{user.name}</div>;
}
```

---

## Conditional Context Reading

### Real-World Example

```jsx
import { use } from 'react';
import { UserContext, GuestContext } from './contexts';

function Greeting({ isLoggedIn }) {
  // Read different contexts based on condition!
  const user = isLoggedIn
    ? use(UserContext)
    : use(GuestContext);
  
  return <h1>Hello, {user.name}!</h1>;
}
```

**Not possible with `useContext`!**

---

## Reading Multiple Contexts in Loop

```jsx
import { use } from 'react';

function MultiThemeComponent({ themes }) {
  // Read multiple theme contexts
  const themeValues = themes.map(ThemeContext => use(ThemeContext));
  
  return (
    <div>
      {themeValues.map((theme, i) => (
        <div key={i} style={{ color: theme.color }}>
          Theme {i + 1}
        </div>
      ))}
    </div>
  );
}
```

---

## Comparison: use vs useEffect + useState

### With useEffect + useState

```jsx
import { useState, useEffect } from 'react';

function User({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}

// 3 state variables
// 1 effect
// Manual loading/error handling
// ~20 lines of code
```

---

### With `use` Hook

```jsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function User({ userPromise }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}

function App({ userId }) {
  const userPromise = fetchUser(userId);
  
  return (
    <ErrorBoundary fallback={<div>Error!</div>}>
      <Suspense fallback={<div>Loading...</div>}>
        <User userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}

// No state
// No effects
// Automatic loading/error handling
// ~5 lines per component
```

**4x less code!**

---

## Advanced: Conditional Data Fetching

```jsx
import { use } from 'react';

function UserDashboard({ userId, showPosts, showComments }) {
  // Always fetch user
  const user = use(fetchUser(userId));
  
  // Conditionally fetch posts
  let posts = null;
  if (showPosts) {
    posts = use(fetchPosts(userId));
  }
  
  // Conditionally fetch comments
  let comments = null;
  if (showComments) {
    comments = use(fetchComments(userId));
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      {posts && <PostList posts={posts} />}
      {comments && <CommentList comments={comments} />}
    </div>
  );
}
```

**Dynamic data fetching based on props!**

---

## Streaming with `use`

```jsx
import { use, Suspense } from 'react';

function Page() {
  return (
    <div>
      {/* Header loads first */}
      <header>My Site</header>
      
      {/* User info streams when ready */}
      <Suspense fallback={<UserSkeleton />}>
        <User userPromise={fetchUser(1)} />
      </Suspense>
      
      {/* Posts stream when ready */}
      <Suspense fallback={<PostsSkeleton />}>
        <Posts postsPromise={fetchPosts()} />
      </Suspense>
      
      {/* Comments stream when ready */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments commentsPromise={fetchComments()} />
      </Suspense>
    </div>
  );
}

function User({ userPromise }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}

function Posts({ postsPromise }) {
  const posts = use(postsPromise);
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}

function Comments({ commentsPromise }) {
  const comments = use(commentsPromise);
  return <ul>{comments.map(c => <li key={c.id}>{c.text}</li>)}</ul>;
}
```

**Progressive rendering with streaming!**

---

## Best Practices

**1. Pass promises as props, don't create in component**
```jsx
// ✅ Good
function Parent() {
  const promise = fetchData();  // Create here
  return <Child dataPromise={promise} />;
}

// ❌ Bad
function Child() {
  const data = use(fetchData());  // Creates new promise every render
}
```

**2. Wrap with Suspense**
```jsx
<Suspense fallback={<Loading />}>
  <Component dataPromise={promise} />
</Suspense>
```

**3. Use Error Boundaries**
```jsx
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Loading />}>
    <Component dataPromise={promise} />
  </Suspense>
</ErrorBoundary>
```

**4. Cache promises**
```jsx
const cache = new Map();

function fetchData(id) {
  if (!cache.has(id)) {
    cache.set(id, fetch(`/api/${id}`).then(r => r.json()));
  }
  return cache.get(id);
}
```

**5. Start fetching early**
```jsx
// Start fetching before rendering
const promise = fetchData();

function Component() {
  const data = use(promise);  // Already fetching
}
```

---

## Common Patterns

### Pattern 1: Parallel Fetching

```jsx
function Page({ userId }) {
  // Start all fetches immediately
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId);
  const followersPromise = fetchFollowers(userId);
  
  return (
    <div>
      <Suspense fallback={<Skeleton />}>
        <User userPromise={userPromise} />
      </Suspense>
      
      <Suspense fallback={<Skeleton />}>
        <Posts postsPromise={postsPromise} />
      </Suspense>
      
      <Suspense fallback={<Skeleton />}>
        <Followers followersPromise={followersPromise} />
      </Suspense>
    </div>
  );
}
```

---

### Pattern 2: Nested Suspense

```jsx
function Page() {
  const dataPromise = fetchData();
  
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Content dataPromise={dataPromise} />
    </Suspense>
  );
}

function Content({ dataPromise }) {
  const data = use(dataPromise);
  
  return (
    <div>
      <h1>{data.title}</h1>
      
      {/* Nested Suspense for slower data */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments commentsPromise={fetchComments(data.id)} />
      </Suspense>
    </div>
  );
}
```

---

### Pattern 3: Conditional Context

```jsx
function Component({ useTheme, ThemeContext, fallbackTheme }) {
  const theme = useTheme ? use(ThemeContext) : fallbackTheme;
  
  return <div style={{ color: theme.color }}>...</div>;
}
```

---

## Limitations

**1. Must be in component or custom hook**
```jsx
// ❌ Can't use in regular function
function helper() {
  const data = use(promise);  // Error!
}

// ✅ Use in component
function Component() {
  const data = use(promise);  // OK
}
```

**2. Can't use in try/catch for Suspense**
```jsx
try {
  const data = use(promise);  // This will suspend, not throw
} catch (e) {
  // Won't catch Suspense
}
```

**3. Promise must be stable**
```jsx
// ❌ New promise every render
function Component() {
  const data = use(fetch('/api/data'));  // New fetch every render!
}

// ✅ Stable promise
function Component({ dataPromise }) {
  const data = use(dataPromise);  // Same promise
}
```

---

## use vs Other Hooks

| Feature | useEffect | useContext | use |
|---------|-----------|------------|-----|
| Conditional | ❌ No | ❌ No | ✅ Yes |
| Loops | ❌ No | ❌ No | ✅ Yes |
| Async data | ✅ Yes | ❌ No | ✅ Yes |
| Context | ❌ No | ✅ Yes | ✅ Yes |
| Suspense | ❌ No | ❌ No | ✅ Yes |
| Rules of Hooks | ✅ Must follow | ✅ Must follow | ⚠️ More flexible |

---

**Interview Tips:**
- **`use` hook** = read Promises and Context during render
- **Suspends component** = while promise resolves
- **No useEffect needed** = direct data reading
- **Pass promise as prop** = don't create in component
- **Conditional usage** = unlike other hooks, can be conditional
- **Can be in loops** = map over contexts/promises
- **Works with Context** = alternative to useContext
- **Requires Suspense** = wrap component in Suspense
- **Error boundaries** = handle promise rejections
- **Parallel fetching** = start multiple fetches simultaneously
- **Cache promises** = prevent duplicate fetches
- **Declarative** = more declarative than useEffect
- **Less boilerplate** = 70-80% less code
- **Type-safe** = TypeScript support
- **React 19+ only** = new in React 19
- **Streaming SSR** = works great with streaming
- **Server Components** = complements Server Components
- **No loading states** = Suspense handles loading
- **No cleanup** = no cleanup needed
- **Game changer** = revolutionizes async data in React
- **Flexible hook rules** = breaks traditional hook constraints
- **Better composition** = easier to compose async logic
- **Simplifies patterns** = simplifies many common patterns

</details>

---

### 93. What are Asset Loading improvements in React 19?

<details>
<summary>View Answer</summary>

**Asset Loading Improvements in React 19**

React 19 introduces automatic asset management with new APIs like `preload`, `prefetch`, and improved support for stylesheets, scripts, and fonts, with better integration with Suspense.

---

## What's New?

**React 19 automatically manages loading of:**
- **Stylesheets** - automatically deduped and inserted
- **Scripts** - automatic insertion and deduplication
- **Fonts** - automatic preloading
- **Images** - integrated with Suspense
- **Preloading** - explicit resource preloading
- **Prefetching** - loading resources in advance

---

## Stylesheet Loading

### Old Way (React 18)

```jsx
import { useEffect } from 'react';

function Component() {
  useEffect(() => {
    // Manually load stylesheet
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = '/styles/component.css';
    document.head.appendChild(link);
    
    return () => {
      document.head.removeChild(link);
    };
  }, []);
  
  return <div className="component">Content</div>;
}

// Problems:
// 1. Manual DOM manipulation
// 2. No deduplication
// 3. Race conditions
// 4. Component renders before styles load (FOUC)
```

---

### New Way (React 19)

```jsx
import { Suspense } from 'react';

function Component() {
  return (
    <div className="component">
      {/* React automatically manages this */}
      <link rel="stylesheet" href="/styles/component.css" precedence="default" />
      Content
    </div>
  );
}

// Benefits:
// 1. Automatic insertion
// 2. Automatic deduplication
// 3. Suspense integration
// 4. No FOUC (Flash of Unstyled Content)
```

**React suspends until stylesheet loads!**

---

## Precedence for Stylesheets

### Controlling Load Order

```jsx
function App() {
  return (
    <div>
      {/* Loads first */}
      <link
        rel="stylesheet"
        href="/reset.css"
        precedence="reset"
      />
      
      {/* Loads second */}
      <link
        rel="stylesheet"
        href="/theme.css"
        precedence="default"
      />
      
      {/* Loads last */}
      <link
        rel="stylesheet"
        href="/component.css"
        precedence="high"
      />
      
      <MyComponent />
    </div>
  );
}
```

**Precedence values:** `reset` < `low` < `default` < `high`

---

## Automatic Deduplication

```jsx
function ComponentA() {
  return (
    <div>
      <link rel="stylesheet" href="/shared.css" precedence="default" />
      Component A
    </div>
  );
}

function ComponentB() {
  return (
    <div>
      {/* React deduplicates - only loaded once! */}
      <link rel="stylesheet" href="/shared.css" precedence="default" />
      Component B
    </div>
  );
}

function App() {
  return (
    <>
      <ComponentA />
      <ComponentB />
    </>
  );
}

// /shared.css loads only once, even though declared in both components!
```

---

## Script Loading

### Async Scripts

```jsx
function Component() {
  return (
    <div>
      {/* React manages this script */}
      <script async src="/analytics.js" />
      Content
    </div>
  );
}

// Benefits:
// 1. Automatic deduplication
// 2. Proper insertion order
// 3. No duplicate loads
```

---

### Script Deduplication

```jsx
function Header() {
  return (
    <header>
      <script async src="/tracking.js" />
    </header>
  );
}

function Footer() {
  return (
    <footer>
      {/* Same script - only loads once! */}
      <script async src="/tracking.js" />
    </footer>
  );
}
```

---

## Preloading Resources

### ReactDOM.preload()

```jsx
import { preload } from 'react-dom';

// Preload before component renders
function App() {
  // Start loading immediately
  preload('/fonts/custom.woff2', { as: 'font', type: 'font/woff2' });
  preload('/critical.css', { as: 'style' });
  preload('/hero.jpg', { as: 'image' });
  
  return (
    <div>
      <Hero />  {/* Assets already loading */}
    </div>
  );
}
```

---

### Preload Types

```jsx
import { preload } from 'react-dom';

// Font preloading
preload('/fonts/Inter.woff2', {
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous'
});

// Stylesheet preloading
preload('/critical.css', {
  as: 'style'
});

// Script preloading
preload('/analytics.js', {
  as: 'script'
});

// Image preloading
preload('/hero.jpg', {
  as: 'image',
  fetchPriority: 'high'
});

// Fetch preloading
preload('/api/data', {
  as: 'fetch',
  crossOrigin: 'anonymous'
});
```

---

## Prefetching Resources

### ReactDOM.prefetch()

```jsx
import { prefetch } from 'react-dom';

function ProductList() {
  return (
    <div>
      {products.map(product => (
        <div
          key={product.id}
          onMouseEnter={() => {
            // Prefetch product page when user hovers
            prefetch(`/products/${product.id}`, { as: 'document' });
          }}
        >
          {product.name}
        </div>
      ))}
    </div>
  );
}
```

---

### Prefetch vs Preload

```jsx
import { preload, prefetch } from 'react-dom';

// PRELOAD = Need soon (high priority)
preload('/critical.css', { as: 'style' });
// Browser loads immediately with high priority

// PREFETCH = Might need later (low priority)
prefetch('/next-page.html', { as: 'document' });
// Browser loads when idle with low priority
```

| | preload | prefetch |
|---------|---------|----------|
| **Priority** | High | Low |
| **When** | Need now | Might need later |
| **Use for** | Current page assets | Next page assets |
| **Timing** | Immediate | When idle |

---

## Font Loading

### Automatic Font Preloading

```jsx
function App() {
  return (
    <html>
      <head>
        {/* React automatically preloads fonts referenced in CSS */}
        <link rel="stylesheet" href="/fonts.css" precedence="default" />
      </head>
      <body>
        <div style={{ fontFamily: 'CustomFont' }}>
          Text
        </div>
      </body>
    </html>
  );
}

// fonts.css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  // React automatically preloads this!
}
```

---

### Manual Font Preloading

```jsx
import { preload } from 'react-dom';

function App() {
  // Explicitly preload font
  preload('/fonts/Inter-Bold.woff2', {
    as: 'font',
    type: 'font/woff2',
    crossOrigin: 'anonymous'  // Required for fonts!
  });
  
  return <div>Content</div>;
}
```

---

## Suspense Integration

### Suspend Until Assets Load

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Component />
    </Suspense>
  );
}

function Component() {
  return (
    <div>
      {/* Component suspends until stylesheet loads */}
      <link rel="stylesheet" href="/component.css" precedence="default" />
      
      {/* No FOUC - styles guaranteed to be loaded! */}
      <div className="styled-content">
        Content with styles
      </div>
    </div>
  );
}
```

**No more Flash of Unstyled Content!**

---

## Real-World Example: E-commerce Product Page

```jsx
import { Suspense } from 'react';
import { preload, prefetch } from 'react-dom';

function ProductPage({ product }) {
  // Preload critical assets
  preload(product.imageUrl, { as: 'image', fetchPriority: 'high' });
  preload('/fonts/Product-Title.woff2', { as: 'font', type: 'font/woff2' });
  
  // Prefetch related products (low priority)
  product.relatedIds.forEach(id => {
    prefetch(`/products/${id}`, { as: 'document' });
  });
  
  return (
    <Suspense fallback={<ProductSkeleton />}>
      <div className="product-page">
        {/* Stylesheet with precedence */}
        <link
          rel="stylesheet"
          href="/styles/product.css"
          precedence="default"
        />
        
        {/* Product content */}
        <img src={product.imageUrl} alt={product.name} />
        <h1>{product.name}</h1>
        <p>{product.description}</p>
        
        {/* Third-party script */}
        <script async src="/reviews-widget.js" />
        <div id="reviews"></div>
      </div>
    </Suspense>
  );
}
```

---

## Component-Scoped Styles

```jsx
function Card() {
  return (
    <div className="card">
      {/* Styles scoped to this component */}
      <link rel="stylesheet" href="/styles/card.css" precedence="default" />
      
      <h3>Card Title</h3>
      <p>Card content</p>
    </div>
  );
}

function App() {
  return (
    <div>
      <Card />  {/* Loads card.css */}
      <Card />  {/* Reuses same card.css */}
      <Card />  {/* Reuses same card.css */}
    </div>
  );
}

// card.css only loads once!
```

---

## Conditional Asset Loading

```jsx
function ThemeComponent({ darkMode }) {
  return (
    <div>
      {/* Load different stylesheet based on theme */}
      {darkMode ? (
        <link rel="stylesheet" href="/dark.css" precedence="theme" />
      ) : (
        <link rel="stylesheet" href="/light.css" precedence="theme" />
      )}
      
      <Content />
    </div>
  );
}
```

---

## Route-Based Code Splitting

```jsx
import { lazy, Suspense } from 'react';
import { prefetch } from 'react-dom';

const HomePage = lazy(() => import('./pages/Home'));
const AboutPage = lazy(() => import('./pages/About'));

function App() {
  return (
    <div>
      <nav>
        <Link
          to="/"
          onMouseEnter={() => {
            // Prefetch home page on hover
            prefetch('/', { as: 'document' });
          }}
        >
          Home
        </Link>
        
        <Link
          to="/about"
          onMouseEnter={() => {
            // Prefetch about page on hover
            prefetch('/about', { as: 'document' });
          }}
        >
          About
        </Link>
      </nav>
      
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/about" element={<AboutPage />} />
        </Routes>
      </Suspense>
    </div>
  );
}
```

---

## Optimizing Initial Load

```jsx
import { preload } from 'react-dom';

function App() {
  // Preload critical resources
  preload('/fonts/Inter-Regular.woff2', {
    as: 'font',
    type: 'font/woff2',
    crossOrigin: 'anonymous'
  });
  
  preload('/critical.css', { as: 'style' });
  preload('/hero.jpg', { as: 'image', fetchPriority: 'high' });
  
  return (
    <Suspense fallback={<Skeleton />}>
      <div>
        <link rel="stylesheet" href="/critical.css" precedence="high" />
        <img src="/hero.jpg" alt="Hero" />
        <h1>Welcome</h1>
      </div>
    </Suspense>
  );
}
```

---

## Preloading API Data

```jsx
import { preload } from 'react-dom';

function ProductPage({ productId }) {
  // Preload API data
  preload(`/api/products/${productId}`, {
    as: 'fetch',
    crossOrigin: 'anonymous'
  });
  
  // Component can now fetch immediately
  // (browser already started loading)
  const product = use(fetch(`/api/products/${productId}`));
  
  return <div>{product.name}</div>;
}
```

---

## Benefits

### 1. No More Manual Asset Management

**Before:**
```jsx
useEffect(() => {
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = '/styles.css';
  document.head.appendChild(link);
  return () => document.head.removeChild(link);
}, []);
```

**After:**
```jsx
<link rel="stylesheet" href="/styles.css" precedence="default" />
```

---

### 2. Automatic Deduplication

```jsx
// Both components reference same stylesheet
// Only loads once!
<ComponentA />  {/* Declares /shared.css */}
<ComponentB />  {/* Also declares /shared.css */}

// React automatically deduplicates
```

---

### 3. No FOUC (Flash of Unstyled Content)

```jsx
<Suspense fallback={<Skeleton />}>
  <Component />
</Suspense>

// Component suspends until styles load
// User never sees unstyled content!
```

---

### 4. Better Performance

- **Preload critical assets** - load in parallel
- **Prefetch next pages** - instant navigation
- **Automatic prioritization** - browser optimizes loading

---

## Comparison

### React 18 (Manual)

```jsx
import { useEffect } from 'react';

function Component() {
  const [loaded, setLoaded] = useState(false);
  
  useEffect(() => {
    // Manually create link
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = '/component.css';
    link.onload = () => setLoaded(true);
    document.head.appendChild(link);
    
    // Manually create script
    const script = document.createElement('script');
    script.src = '/widget.js';
    script.async = true;
    document.body.appendChild(script);
    
    return () => {
      document.head.removeChild(link);
      document.body.removeChild(script);
    };
  }, []);
  
  if (!loaded) return <div>Loading...</div>;
  
  return <div className="styled">Content</div>;
}

// Problems:
// - Manual DOM manipulation
// - Race conditions
// - FOUC
// - Complex cleanup
```

---

### React 19 (Automatic)

```jsx
import { Suspense } from 'react';

function Component() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <div className="styled">
        <link rel="stylesheet" href="/component.css" precedence="default" />
        <script async src="/widget.js" />
        Content
      </div>
    </Suspense>
  );
}

// Benefits:
// - Automatic management
// - No race conditions
// - No FOUC
// - No cleanup needed
```

---

## Best Practices

**1. Use precedence for stylesheets**
```jsx
<link rel="stylesheet" href="/critical.css" precedence="high" />
```

**2. Preload critical assets**
```jsx
preload('/hero.jpg', { as: 'image', fetchPriority: 'high' });
```

**3. Prefetch next pages**
```jsx
prefetch('/next-page', { as: 'document' });
```

**4. Use Suspense with stylesheets**
```jsx
<Suspense fallback={<Skeleton />}>
  <ComponentWithStyles />
</Suspense>
```

**5. Specify crossOrigin for fonts**
```jsx
preload('/font.woff2', {
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous'  // Required!
});
```

**6. Use fetchPriority for images**
```jsx
preload('/hero.jpg', {
  as: 'image',
  fetchPriority: 'high'  // For above-the-fold images
});
```

---

**Interview Tips:**
- **Asset loading** = React 19 automatically manages assets
- **Stylesheets** = automatic insertion and deduplication
- **precedence attribute** = control stylesheet order
- **Scripts** = automatic deduplication
- **Suspense integration** = suspend until assets load
- **No FOUC** = styles guaranteed before render
- **preload()** = load critical assets with high priority
- **prefetch()** = load future assets with low priority
- **Font preloading** = automatic for CSS fonts
- **Deduplication** = same asset only loads once
- **Component-scoped** = declare assets in components
- **Conditional loading** = load different assets based on state
- **Type safety** = TypeScript support for preload/prefetch
- **No manual DOM** = no createElement/appendChild needed
- **Better performance** = parallel loading, prioritization
- **Simpler code** = 70% less boilerplate
- **crossOrigin required** = for fonts and external resources
- **fetchPriority** = hint browser priority
- **React 19+ only** = new feature
- **Works with SSR** = server-side rendering compatible
- **Backward compatible** = graceful degradation

</details>

---

### 94. What is the new useFormStatus hook?

<details>
<summary>View Answer</summary>

**The `useFormStatus` Hook**

A React 19 hook that provides status information about the last form submission, allowing you to show pending states, disable buttons, and display feedback during form submission.

---

## What is useFormStatus?

**`useFormStatus` = Get form submission status**

### Returns

```jsx
import { useFormStatus } from 'react-dom';

const { pending, data, method, action } = useFormStatus();

// pending: boolean - is form currently submitting?
// data: FormData | null - form data being submitted
// method: string - HTTP method (POST, GET, etc.)
// action: string | function - form action
```

---

## Basic Example

### Simple Submit Button

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function ContactForm() {
  async function handleSubmit(formData) {
    'use server';
    await sendEmail(formData);
  }
  
  return (
    <form action={handleSubmit}>
      <input name="email" type="email" />
      <textarea name="message" />
      <SubmitButton />  {/* Uses useFormStatus */}
    </form>
  );
}
```

**Button automatically shows loading state!**

---

## Key Point: Must Be Child of Form

```jsx
import { useFormStatus } from 'react-dom';

// ❌ WRONG - useFormStatus outside form
function WrongExample() {
  const { pending } = useFormStatus();  // Won't work!
  
  return (
    <form>
      <button disabled={pending}>Submit</button>
    </form>
  );
}

// ✅ CORRECT - useFormStatus in child component
function SubmitButton() {
  const { pending } = useFormStatus();  // Works!
  return <button disabled={pending}>Submit</button>;
}

function CorrectExample() {
  return (
    <form>
      <SubmitButton />  {/* Child component */}
    </form>
  );
}
```

**Important: `useFormStatus` must be called in a component that's a child of the `<form>`**

---

## The `pending` Property

### Disable Button During Submit

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating post...' : 'Create Post'}
    </button>
  );
}

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <SubmitButton />
    </form>
  );
}
```

---

### Show Spinner

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending && <Spinner />}
      {pending ? 'Saving...' : 'Save'}
    </button>
  );
}
```

---

### Disable All Form Inputs

```jsx
import { useFormStatus } from 'react-dom';

function FormFields() {
  const { pending } = useFormStatus();
  
  return (
    <>
      <input
        name="title"
        disabled={pending}
        placeholder="Title"
      />
      
      <textarea
        name="content"
        disabled={pending}
        placeholder="Content"
      />
      
      <button type="submit" disabled={pending}>
        {pending ? 'Submitting...' : 'Submit'}
      </button>
    </>
  );
}

function Form() {
  return (
    <form action={submitAction}>
      <FormFields />  {/* All fields auto-disable */}
    </form>
  );
}
```

---

## The `data` Property

### Access Form Data Being Submitted

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <span>
          Submitting {data?.get('title')}...
        </span>
      ) : (
        'Submit'
      )}
    </button>
  );
}

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Post title" />
      <SubmitButton />
    </form>
  );
}

// Button shows: "Submitting My Post Title..." during submit
```

---

## The `method` Property

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, method } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? `${method}ing...` : 'Submit'}
    </button>
  );
}

function Form() {
  return (
    <form action="/api/posts" method="POST">
      <input name="title" />
      <SubmitButton />  {/* Shows "POSTing..." */}
    </form>
  );
}
```

---

## Real-World Example: Blog Post Form

### Server Action

```jsx
// actions.js
'use server';

import { db } from '@/lib/database';
import { revalidatePath } from 'next/cache';

export async function createPost(formData) {
  const title = formData.get('title');
  const content = formData.get('content');
  
  // Simulate delay
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  await db.post.create({
    data: { title, content }
  });
  
  revalidatePath('/posts');
}
```

---

### Form Components

```jsx
import { useFormStatus } from 'react-dom';
import { createPost } from './actions';

// Submit button component
function SubmitButton() {
  const { pending, data } = useFormStatus();
  
  return (
    <button
      type="submit"
      disabled={pending}
      className="btn-primary"
    >
      {pending ? (
        <>
          <Spinner />
          <span>Creating "{data?.get('title')}"...</span>
        </>
      ) : (
        'Create Post'
      )}
    </button>
  );
}

// Form inputs component
function FormInputs() {
  const { pending } = useFormStatus();
  
  return (
    <>
      <div className="field">
        <label htmlFor="title">Title</label>
        <input
          id="title"
          name="title"
          type="text"
          disabled={pending}
          required
        />
      </div>
      
      <div className="field">
        <label htmlFor="content">Content</label>
        <textarea
          id="content"
          name="content"
          rows={10}
          disabled={pending}
          required
        />
      </div>
    </>
  );
}

// Main form
function PostForm() {
  return (
    <form action={createPost} className="post-form">
      <h1>Create New Post</h1>
      
      <FormInputs />
      
      <div className="actions">
        <SubmitButton />
      </div>
    </form>
  );
}

export default PostForm;
```

---

## Progress Indicator

```jsx
import { useFormStatus } from 'react-dom';

function FormProgress() {
  const { pending } = useFormStatus();
  
  if (!pending) return null;
  
  return (
    <div className="progress-bar">
      <div className="progress-fill" />
      <span>Uploading...</span>
    </div>
  );
}

function UploadForm() {
  return (
    <form action={uploadFile}>
      <input type="file" name="file" />
      
      <FormProgress />  {/* Shows when submitting */}
      
      <button type="submit">Upload</button>
    </form>
  );
}
```

---

## Multiple Submit Buttons

```jsx
import { useFormStatus } from 'react-dom';

function SaveButton() {
  const { pending, data } = useFormStatus();
  const isSaving = pending && data?.get('action') === 'save';
  
  return (
    <button
      type="submit"
      name="action"
      value="save"
      disabled={pending}
    >
      {isSaving ? 'Saving...' : 'Save Draft'}
    </button>
  );
}

function PublishButton() {
  const { pending, data } = useFormStatus();
  const isPublishing = pending && data?.get('action') === 'publish';
  
  return (
    <button
      type="submit"
      name="action"
      value="publish"
      disabled={pending}
      className="btn-primary"
    >
      {isPublishing ? 'Publishing...' : 'Publish'}
    </button>
  );
}

function PostForm() {
  async function handleSubmit(formData) {
    'use server';
    
    const action = formData.get('action');
    
    if (action === 'save') {
      await saveAsDraft(formData);
    } else {
      await publishPost(formData);
    }
  }
  
  return (
    <form action={handleSubmit}>
      <input name="title" />
      <textarea name="content" />
      
      <div className="actions">
        <SaveButton />     {/* Shows "Saving..." */}
        <PublishButton />  {/* Shows "Publishing..." */}
      </div>
    </form>
  );
}
```

---

## Optimistic UI with useFormStatus

```jsx
import { useFormStatus } from 'react-dom';
import { useOptimistic } from 'react';

function Comment({ comment }) {
  const { pending } = useFormStatus();
  
  return (
    <div className={pending ? 'pending' : ''}>
      <p>{comment.text}</p>
      {pending && <span className="badge">Sending...</span>}
    </div>
  );
}

function CommentForm({ postId, comments }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (current, newComment) => [...current, newComment]
  );
  
  async function handleSubmit(formData) {
    'use server';
    
    const text = formData.get('text');
    addOptimisticComment({ text, id: Date.now() });
    
    await createComment({ postId, text });
  }
  
  return (
    <div>
      {optimisticComments.map(comment => (
        <Comment key={comment.id} comment={comment} />
      ))}
      
      <form action={handleSubmit}>
        <textarea name="text" />
        <SubmitButton />
      </form>
    </div>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending}>
      {pending ? 'Posting...' : 'Post Comment'}
    </button>
  );
}
```

---

## Custom Loading Component

```jsx
import { useFormStatus } from 'react-dom';

function LoadingOverlay() {
  const { pending } = useFormStatus();
  
  if (!pending) return null;
  
  return (
    <div className="overlay">
      <div className="spinner-large" />
      <p>Processing your request...</p>
    </div>
  );
}

function Form() {
  return (
    <form action={submitAction}>
      <input name="field1" />
      <input name="field2" />
      
      <LoadingOverlay />  {/* Shows during submit */}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Validation Feedback

```jsx
import { useFormStatus } from 'react-dom';
import { useActionState } from 'react';

function FormFields() {
  const { pending } = useFormStatus();
  
  return (
    <>
      <input
        name="email"
        type="email"
        disabled={pending}
        aria-busy={pending}
      />
      <button type="submit" disabled={pending}>
        {pending ? 'Validating...' : 'Submit'}
      </button>
    </>
  );
}

function RegistrationForm() {
  const [state, action] = useActionState(registerUser, null);
  
  return (
    <form action={action}>
      <FormFields />
      
      {state?.errors && (
        <div className="errors">
          {Object.values(state.errors).map(err => (
            <p key={err}>{err}</p>
          ))}
        </div>
      )}
    </form>
  );
}
```

---

## File Upload with Progress

```jsx
import { useFormStatus } from 'react-dom';

function UploadStatus() {
  const { pending, data } = useFormStatus();
  
  if (!pending) return null;
  
  const fileName = data?.get('file')?.name;
  
  return (
    <div className="upload-status">
      <div className="spinner" />
      <span>Uploading {fileName}...</span>
      <div className="progress-bar">
        <div className="progress" />
      </div>
    </div>
  );
}

function FileUploadForm() {
  return (
    <form action={uploadFile}>
      <input type="file" name="file" accept="image/*" />
      
      <UploadStatus />
      
      <button type="submit">Upload</button>
    </form>
  );
}
```

---

## Comparison: Before and After

### Without useFormStatus

```jsx
import { useState } from 'react';

function Form() {
  const [isPending, setIsPending] = useState(false);
  
  async function handleSubmit(e) {
    e.preventDefault();
    setIsPending(true);
    
    try {
      const formData = new FormData(e.target);
      await submitForm(formData);
    } finally {
      setIsPending(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="field" disabled={isPending} />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

// Manual state management
// Must prevent default
// Manual FormData creation
// Manual error handling
```

---

### With useFormStatus

```jsx
import { useFormStatus } from 'react-dom';

function FormFields() {
  const { pending } = useFormStatus();
  
  return (
    <>
      <input name="field" disabled={pending} />
      <button disabled={pending}>
        {pending ? 'Submitting...' : 'Submit'}
      </button>
    </>
  );
}

function Form() {
  return (
    <form action={submitForm}>
      <FormFields />
    </form>
  );
}

// Automatic state management
// No preventDefault needed
// Automatic FormData handling
// Built-in error handling
```

**60% less code!**

---

## Best Practices

**1. Call in child component**
```jsx
// ✅ Good
function SubmitButton() {
  const { pending } = useFormStatus();  // In child
  return <button disabled={pending}>Submit</button>;
}

function Form() {
  return (
    <form>
      <SubmitButton />
    </form>
  );
}
```

**2. Disable inputs during submit**
```jsx
const { pending } = useFormStatus();
return <input disabled={pending} />;
```

**3. Show meaningful loading states**
```jsx
const { pending, data } = useFormStatus();
return pending ? `Saving ${data?.get('title')}...` : 'Save';
```

**4. Use with Server Actions**
```jsx
<form action={serverAction}>
  <SubmitButtonWithStatus />
</form>
```

**5. Combine with useActionState**
```jsx
const [state, action] = useActionState(...);
return <form action={action}>...</form>;
```

---

**Interview Tips:**
- **useFormStatus** = get form submission status
- **pending property** = is form currently submitting?
- **data property** = FormData being submitted
- **method property** = HTTP method
- **action property** = form action
- **Must be child of form** = called in component inside `<form>`
- **Automatic state** = no manual useState needed
- **Disable button** = prevent double submissions
- **Show spinner** = loading indicators
- **Access form data** = get values being submitted
- **Multiple buttons** = different states per button
- **No preventDefault** = works with native forms
- **Server Actions** = designed for Server Actions
- **Progressive enhancement** = works without JavaScript
- **Type-safe** = TypeScript support
- **React 19+ only** = new hook in React 19
- **Works with useActionState** = complementary hooks
- **Optimistic UI** = combine with useOptimistic
- **Less boilerplate** = 60-70% less code
- **Better UX** = immediate feedback
- **No manual state** = React handles state
- **Component pattern** = extract to reusable components
- **Accessibility** = use aria-busy attribute
- **File uploads** = show upload progress

</details>

---

### 95. What is useOptimistic hook?

<details>
<summary>View Answer</summary>

**The `useOptimistic` Hook**

A React 19 hook that allows you to show optimistic UI updates immediately while an async action is pending, then automatically revert to actual data when the action completes.

---

## What is useOptimistic?

**`useOptimistic` = Show optimistic state while async action pending**

### Problem It Solves

**Without optimistic updates:**
```
User clicks "Like" button
  ↓
Wait for server response (500ms)
  ↓
UI updates

User experience: Laggy, unresponsive
```

**With optimistic updates:**
```
User clicks "Like" button
  ↓
UI updates IMMEDIATELY ✨
  ↓
Server processes in background
  ↓
UI stays updated (or reverts if error)

User experience: Instant, responsive
```

---

## Basic Syntax

```jsx
import { useOptimistic } from 'react';

const [optimisticState, addOptimistic] = useOptimistic(
  currentState,
  updateFn
);

// currentState: actual current state
// updateFn: (state, optimisticValue) => newState
// optimisticState: state to display (includes optimistic updates)
// addOptimistic: function to add optimistic update
```

---

## Simple Example: Like Button

```jsx
import { useOptimistic } from 'react';
import { likePost } from './actions';

function Post({ post }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (currentLikes, amount) => currentLikes + amount
  );
  
  async function handleLike() {
    // Update UI immediately
    addOptimisticLike(1);
    
    // Then update server
    await likePost(post.id);
  }
  
  return (
    <div>
      <h3>{post.title}</h3>
      <button onClick={handleLike}>
        ❤️ {optimisticLikes} likes
      </button>
    </div>
  );
}

// actions.js
'use server';

export async function likePost(postId) {
  await db.post.update({
    where: { id: postId },
    data: { likes: { increment: 1 } }
  });
  revalidatePath(`/posts/${postId}`);
}
```

**Like count updates instantly, even though server takes 500ms!**

---

## How It Works

### Timeline

```
1. User clicks "Like"
   likes: 42

2. addOptimisticLike(1) called
   optimisticLikes: 43 ← UI shows this immediately!
   
3. likePost() runs in background (500ms)
   optimisticLikes: 43 ← Still showing optimistic value
   
4. Server responds, component re-renders with new data
   post.likes: 43 ← Real data arrives
   optimisticLikes: 43 ← Matches real data
```

---

## Complete Example: Todo List

```jsx
import { useOptimistic } from 'react';
import { addTodo, toggleTodo } from './actions';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => {
      if (newTodo.type === 'add') {
        return [...currentTodos, newTodo.todo];
      }
      if (newTodo.type === 'toggle') {
        return currentTodos.map(todo =>
          todo.id === newTodo.id
            ? { ...todo, completed: !todo.completed }
            : todo
        );
      }
      return currentTodos;
    }
  );
  
  async function handleAdd(formData) {
    const text = formData.get('text');
    const newTodo = {
      id: Date.now(),
      text,
      completed: false
    };
    
    // Optimistically add
    addOptimisticTodo({ type: 'add', todo: newTodo });
    
    // Then save to server
    await addTodo(text);
  }
  
  async function handleToggle(id) {
    // Optimistically toggle
    addOptimisticTodo({ type: 'toggle', id });
    
    // Then update server
    await toggleTodo(id);
  }
  
  return (
    <div>
      <form action={handleAdd}>
        <input name="text" placeholder="New todo" />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li
            key={todo.id}
            style={{
              textDecoration: todo.completed ? 'line-through' : 'none',
              opacity: todo.pending ? 0.5 : 1
            }}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggle(todo.id)}
            />
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## With Forms

```jsx
import { useOptimistic } from 'react';
import { useActionState } from 'react';
import { createComment } from './actions';

function CommentList({ comments }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (current, newComment) => [...current, newComment]
  );
  
  const [state, formAction] = useActionState(
    async (prevState, formData) => {
      const text = formData.get('text');
      
      // Add optimistic comment
      addOptimisticComment({
        id: Date.now(),
        text,
        author: 'You',
        pending: true  // Mark as pending
      });
      
      // Create on server
      await createComment(text);
      
      return { success: true };
    },
    null
  );
  
  return (
    <div>
      {optimisticComments.map(comment => (
        <div
          key={comment.id}
          className={comment.pending ? 'pending' : ''}
        >
          <strong>{comment.author}</strong>
          <p>{comment.text}</p>
          {comment.pending && <span className="badge">Sending...</span>}
        </div>
      ))}
      
      <form action={formAction}>
        <textarea name="text" placeholder="Add comment" />
        <button type="submit">Post</button>
      </form>
    </div>
  );
}
```

---

## Marking Pending Items

```jsx
import { useOptimistic } from 'react';

function MessageList({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (current, newMessage) => [
      ...current,
      { ...newMessage, pending: true }  // Add pending flag
    ]
  );
  
  async function sendMessage(text) {
    // Add optimistically with pending flag
    addOptimisticMessage({
      id: Date.now(),
      text,
      sender: 'me'
    });
    
    // Send to server
    await fetch('/api/messages', {
      method: 'POST',
      body: JSON.stringify({ text })
    });
  }
  
  return (
    <ul>
      {optimisticMessages.map(msg => (
        <li
          key={msg.id}
          className={msg.pending ? 'pending' : 'sent'}
        >
          {msg.text}
          {msg.pending && <Spinner />}
        </li>
      ))}
    </ul>
  );
}
```

---

## Error Handling

```jsx
import { useOptimistic, useState } from 'react';

function LikeButton({ post }) {
  const [error, setError] = useState(null);
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (current, amount) => current + amount
  );
  
  async function handleLike() {
    try {
      setError(null);
      
      // Optimistically increment
      addOptimisticLike(1);
      
      // Try to save
      const result = await likePost(post.id);
      
      if (!result.success) {
        throw new Error('Failed to like');
      }
    } catch (err) {
      // Optimistic update automatically reverts on error!
      setError(err.message);
    }
  }
  
  return (
    <div>
      <button onClick={handleLike}>
        ❤️ {optimisticLikes}
      </button>
      {error && <div className="error">{error}</div>}
    </div>
  );
}
```

**Optimistic state automatically reverts if the action fails!**

---

## Advanced: Multiple Optimistic Updates

```jsx
import { useOptimistic } from 'react';

function ShoppingCart({ items }) {
  const [optimisticItems, updateOptimisticItems] = useOptimistic(
    items,
    (current, action) => {
      switch (action.type) {
        case 'add':
          return [...current, action.item];
        
        case 'remove':
          return current.filter(item => item.id !== action.id);
        
        case 'increment':
          return current.map(item =>
            item.id === action.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          );
        
        case 'decrement':
          return current.map(item =>
            item.id === action.id
              ? { ...item, quantity: Math.max(0, item.quantity - 1) }
              : item
          );
        
        default:
          return current;
      }
    }
  );
  
  async function addItem(item) {
    updateOptimisticItems({ type: 'add', item });
    await fetch('/api/cart/add', {
      method: 'POST',
      body: JSON.stringify(item)
    });
  }
  
  async function removeItem(id) {
    updateOptimisticItems({ type: 'remove', id });
    await fetch(`/api/cart/${id}`, { method: 'DELETE' });
  }
  
  async function incrementQuantity(id) {
    updateOptimisticItems({ type: 'increment', id });
    await fetch(`/api/cart/${id}/increment`, { method: 'POST' });
  }
  
  async function decrementQuantity(id) {
    updateOptimisticItems({ type: 'decrement', id });
    await fetch(`/api/cart/${id}/decrement`, { method: 'POST' });
  }
  
  return (
    <div className="cart">
      {optimisticItems.map(item => (
        <div key={item.id} className="cart-item">
          <span>{item.name}</span>
          <div className="quantity-controls">
            <button onClick={() => decrementQuantity(item.id)}>-</button>
            <span>{item.quantity}</span>
            <button onClick={() => incrementQuantity(item.id)}>+</button>
          </div>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      
      <div className="total">
        Total: ${optimisticItems.reduce((sum, item) => 
          sum + item.price * item.quantity, 0
        )}
      </div>
    </div>
  );
}
```

---

## Real-World Example: Social Media Post

```jsx
import { useOptimistic } from 'react';
import { likePost, bookmarkPost, sharePost } from './actions';

function SocialPost({ post }) {
  // Optimistic likes
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (current, amount) => current + amount
  );
  
  // Optimistic bookmarked state
  const [optimisticBookmarked, setOptimisticBookmarked] = useOptimistic(
    post.isBookmarked,
    (current, newValue) => newValue
  );
  
  // Optimistic shares
  const [optimisticShares, addOptimisticShare] = useOptimistic(
    post.shares,
    (current, amount) => current + amount
  );
  
  async function handleLike() {
    addOptimisticLike(post.isLiked ? -1 : 1);
    await likePost(post.id, !post.isLiked);
  }
  
  async function handleBookmark() {
    setOptimisticBookmarked(!post.isBookmarked);
    await bookmarkPost(post.id, !post.isBookmarked);
  }
  
  async function handleShare() {
    addOptimisticShare(1);
    await sharePost(post.id);
  }
  
  return (
    <article className="post">
      <div className="post-header">
        <img src={post.author.avatar} alt={post.author.name} />
        <div>
          <strong>{post.author.name}</strong>
          <time>{post.createdAt}</time>
        </div>
      </div>
      
      <div className="post-content">
        <p>{post.text}</p>
        {post.image && <img src={post.image} alt="" />}
      </div>
      
      <div className="post-actions">
        <button
          onClick={handleLike}
          className={post.isLiked ? 'active' : ''}
        >
          ❤️ {optimisticLikes}
        </button>
        
        <button
          onClick={handleBookmark}
          className={optimisticBookmarked ? 'active' : ''}
        >
          🔖 {optimisticBookmarked ? 'Saved' : 'Save'}
        </button>
        
        <button onClick={handleShare}>
          🔗 {optimisticShares} shares
        </button>
      </div>
    </article>
  );
}
```

---

## With Server Actions

```jsx
import { useOptimistic } from 'react';

function CommentSection({ postId, comments }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (current, newComment) => [...current, newComment]
  );
  
  async function addComment(formData) {
    'use server';
    
    const text = formData.get('text');
    
    // This runs on client automatically
    addOptimisticComment({
      id: Date.now(),
      text,
      author: 'You',
      createdAt: new Date(),
      pending: true
    });
    
    // This runs on server
    await db.comment.create({
      data: {
        postId,
        text,
        authorId: session.userId
      }
    });
    
    revalidatePath(`/posts/${postId}`);
  }
  
  return (
    <div>
      <h3>Comments</h3>
      
      {optimisticComments.map(comment => (
        <div
          key={comment.id}
          className={comment.pending ? 'pending' : ''}
        >
          <strong>{comment.author}</strong>
          <p>{comment.text}</p>
          <time>{comment.createdAt.toLocaleString()}</time>
        </div>
      ))}
      
      <form action={addComment}>
        <textarea name="text" required />
        <button type="submit">Post Comment</button>
      </form>
    </div>
  );
}
```

---

## Comparison: Before and After

### Without useOptimistic

```jsx
import { useState } from 'react';

function LikeButton({ post }) {
  const [likes, setLikes] = useState(post.likes);
  const [isPending, setIsPending] = useState(false);
  
  async function handleLike() {
    setIsPending(true);
    
    try {
      const result = await likePost(post.id);
      setLikes(result.likes);  // Wait for server response
    } finally {
      setIsPending(false);
    }
  }
  
  return (
    <button onClick={handleLike} disabled={isPending}>
      {isPending ? 'Liking...' : `❤️ ${likes}`}
    </button>
  );
}

// Problem: Button disabled while pending
// User must wait for server response
// Feels slow and unresponsive
```

---

### With useOptimistic

```jsx
import { useOptimistic } from 'react';

function LikeButton({ post }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (current, amount) => current + amount
  );
  
  async function handleLike() {
    addOptimisticLike(1);  // Update UI immediately!
    await likePost(post.id);  // Then update server
  }
  
  return (
    <button onClick={handleLike}>
      ❤️ {optimisticLikes}
    </button>
  );
}

// Benefit: Button always enabled
// UI updates instantly
// Feels fast and responsive
```

---

## Visual Feedback for Pending State

```jsx
import { useOptimistic } from 'react';

function TodoItem({ todo, onToggle }) {
  const [optimisticCompleted, setOptimisticCompleted] = useOptimistic(
    todo.completed,
    (current, newValue) => newValue
  );
  
  async function handleToggle() {
    setOptimisticCompleted(!todo.completed);
    await onToggle(todo.id);
  }
  
  return (
    <li
      style={{
        opacity: optimisticCompleted !== todo.completed ? 0.5 : 1,
        textDecoration: optimisticCompleted ? 'line-through' : 'none'
      }}
    >
      <input
        type="checkbox"
        checked={optimisticCompleted}
        onChange={handleToggle}
      />
      {todo.text}
      {optimisticCompleted !== todo.completed && (
        <Spinner size="small" />
      )}
    </li>
  );
}
```

---

## Best Practices

**1. Mark optimistic items as pending**
```jsx
addOptimisticItem({
  ...item,
  pending: true  // Visual indicator
});
```

**2. Keep optimistic logic simple**
```jsx
// ✅ Good: Simple update
(current, amount) => current + amount

// ❌ Bad: Complex logic that might fail
(current, item) => {
  // Complex calculations...
  // Multiple conditions...
  // Could cause bugs
}
```

**3. Handle errors gracefully**
```jsx
try {
  addOptimistic(...);
  await serverAction();
} catch (error) {
  // Optimistic state reverts automatically
  showError(error.message);
}
```

**4. Use for user actions only**
```jsx
// ✅ Good: User clicks button
addOptimisticLike(1);

// ❌ Bad: Background updates
// Don't use optimistic updates for automatic/background updates
```

**5. Provide visual feedback**
```jsx
<div className={item.pending ? 'opacity-50' : ''}>
  {item.text}
  {item.pending && <Spinner />}
</div>
```

---

## When to Use useOptimistic

### ✅ Good Use Cases

- Like/unlike buttons
- Follow/unfollow buttons
- Adding items to cart
- Toggling todos
- Posting comments
- Sending messages
- Bookmarking items
- Rating items
- Any user-initiated action

### ❌ Not Good Use Cases

- Loading initial data
- Background syncs
- Webhook responses
- Real-time updates from server
- Complex multi-step operations
- Operations that might fail validation

---

## Common Patterns

### Pattern 1: Toggle Boolean

```jsx
const [optimistic, setOptimistic] = useOptimistic(
  value,
  (current, newValue) => newValue
);

setOptimistic(!value);
```

### Pattern 2: Increment Counter

```jsx
const [optimistic, addOptimistic] = useOptimistic(
  count,
  (current, amount) => current + amount
);

addOptimistic(1);
```

### Pattern 3: Add to List

```jsx
const [optimistic, addOptimistic] = useOptimistic(
  items,
  (current, newItem) => [...current, newItem]
);

addOptimistic({ id: Date.now(), text: 'New item' });
```

### Pattern 4: Update in List

```jsx
const [optimistic, updateOptimistic] = useOptimistic(
  items,
  (current, update) => current.map(item =>
    item.id === update.id ? { ...item, ...update.data } : item
  )
);

updateOptimistic({ id: 123, data: { completed: true } });
```

### Pattern 5: Remove from List

```jsx
const [optimistic, removeOptimistic] = useOptimistic(
  items,
  (current, idToRemove) => current.filter(item => item.id !== idToRemove)
);

removeOptimistic(123);
```

---

**Interview Tips:**
- **useOptimistic** = show optimistic UI while async action pending
- **Instant feedback** = UI updates immediately
- **Automatic revert** = reverts if action fails
- **Better UX** = feels faster and more responsive
- **Two parameters** = currentState and updateFn
- **Returns two values** = optimisticState and addOptimistic
- **Update function** = (currentState, optimisticValue) => newState
- **Mark pending** = add pending flag to optimistic items
- **Visual feedback** = show spinner or reduce opacity
- **Error handling** = automatically reverts on error
- **Use with Server Actions** = designed for Server Actions
- **User actions only** = not for background updates
- **Simple updates** = keep update logic simple
- **Multiple updates** = can have multiple useOptimistic calls
- **Reducer pattern** = can use like reducer with action types
- **React 19+ only** = new hook in React 19
- **No manual revert** = React handles revert automatically
- **Type-safe** = TypeScript support
- **Works with forms** = combine with useActionState
- **Performance** = no extra renders
- **Progressive enhancement** = works without JavaScript
- **Real-time feel** = makes apps feel instant
- **Trust the user** = assume action will succeed

</details>

---

### 96. What are Document Metadata features in React 19?

<details>
<summary>View Answer</summary>

**Document Metadata in React 19**

React 19 allows you to render document metadata tags like `<title>`, `<meta>`, and `<link>` directly in components, and React automatically hoists them to the document `<head>`, eliminating the need for third-party libraries.

---

## What's New?

**Before React 19:**
```jsx
import Head from 'next/head';  // Need external library

function Page() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="..." />
      </Head>
      <div>Content</div>
    </>
  );
}
```

---

**React 19:**
```jsx
function Page() {
  return (
    <>
      {/* Automatically hoisted to <head> */}
      <title>My Page</title>
      <meta name="description" content="..." />
      <div>Content</div>
    </>
  );
}
```

**No library needed! React handles it automatically!**

---

## Supported Tags

### 1. `<title>`

```jsx
function ProductPage({ product }) {
  return (
    <div>
      <title>{product.name} - My Store</title>
      <h1>{product.name}</h1>
    </div>
  );
}

// Renders in <head>:
// <title>iPhone 15 - My Store</title>
```

---

### 2. `<meta>` Tags

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="description" content={post.excerpt} />
      <meta name="author" content={post.author} />
      <meta name="keywords" content={post.tags.join(', ')} />
      
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

---

### 3. `<link>` Tags

```jsx
function Page() {
  return (
    <div>
      {/* Canonical URL */}
      <link rel="canonical" href="https://example.com/page" />
      
      {/* Alternate languages */}
      <link rel="alternate" hrefLang="es" href="https://example.com/es/page" />
      
      {/* RSS feed */}
      <link rel="alternate" type="application/rss+xml" href="/feed.xml" />
      
      <h1>Page Content</h1>
    </div>
  );
}
```

---

## SEO Meta Tags

```jsx
function ProductPage({ product }) {
  return (
    <div>
      {/* Basic meta tags */}
      <title>{product.name} | Buy Online</title>
      <meta name="description" content={product.description} />
      
      {/* Open Graph (Facebook) */}
      <meta property="og:title" content={product.name} />
      <meta property="og:description" content={product.description} />
      <meta property="og:image" content={product.image} />
      <meta property="og:type" content="product" />
      <meta property="og:url" content={`https://example.com/products/${product.id}`} />
      
      {/* Twitter Card */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={product.name} />
      <meta name="twitter:description" content={product.description} />
      <meta name="twitter:image" content={product.image} />
      
      {/* Product info */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <span>${product.price}</span>
    </div>
  );
}
```

---

## Dynamic Metadata

```jsx
function BlogPost({ post }) {
  return (
    <article>
      {/* Dynamic based on data */}
      <title>{post.title} - Blog</title>
      <meta name="description" content={post.excerpt} />
      <meta name="author" content={post.author.name} />
      <meta name="publish-date" content={post.publishedAt} />
      
      {/* Conditional tags */}
      {post.featured && (
        <meta name="robots" content="index, follow" />
      )}
      
      {post.noIndex && (
        <meta name="robots" content="noindex, nofollow" />
      )}
      
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

---

## Component-Level Metadata

```jsx
// Each component can define its own metadata

function HomePage() {
  return (
    <div>
      <title>Home - My Site</title>
      <meta name="description" content="Welcome to my site" />
      <h1>Home</h1>
    </div>
  );
}

function AboutPage() {
  return (
    <div>
      <title>About - My Site</title>
      <meta name="description" content="Learn about us" />
      <h1>About</h1>
    </div>
  );
}

// Title and meta tags automatically update when navigating!
```

---

## Nested Components

```jsx
function Layout({ children }) {
  return (
    <div>
      {/* Default metadata */}
      <title>My Site</title>
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      
      <header>...</header>
      {children}
      <footer>...</footer>
    </div>
  );
}

function BlogPost({ post }) {
  return (
    <article>
      {/* Overrides Layout's title */}
      <title>{post.title} - My Site</title>
      <meta name="description" content={post.excerpt} />
      
      <h1>{post.title}</h1>
    </article>
  );
}

function App() {
  return (
    <Layout>
      <BlogPost post={post} />  {/* BlogPost's title wins */}
    </Layout>
  );
}
```

**Last title wins! Child components override parent metadata.**

---

## Precedence Rules

```jsx
function App() {
  return (
    <div>
      {/* First title */}
      <title>Title 1</title>
      
      <Component1 />
    </div>
  );
}

function Component1() {
  return (
    <div>
      {/* Overrides Title 1 */}
      <title>Title 2</title>
      
      <Component2 />
    </div>
  );
}

function Component2() {
  return (
    <div>
      {/* Overrides Title 2 - This one wins! */}
      <title>Title 3</title>
      
      <h1>Content</h1>
    </div>
  );
}

// Final result: <title>Title 3</title>
```

**Deepest/last title in render tree wins.**

---

## Real-World Example: E-commerce

```jsx
function ProductPage({ product }) {
  return (
    <div className="product-page">
      {/* SEO */}
      <title>{product.name} - ${product.price} | Store Name</title>
      <meta
        name="description"
        content={`Buy ${product.name} for $${product.price}. ${product.description}`}
      />
      <meta name="keywords" content={product.keywords.join(', ')} />
      
      {/* Open Graph */}
      <meta property="og:title" content={product.name} />
      <meta property="og:description" content={product.description} />
      <meta property="og:image" content={product.images[0]} />
      <meta property="og:type" content="product" />
      <meta property="og:price:amount" content={product.price} />
      <meta property="og:price:currency" content="USD" />
      
      {/* Twitter */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={product.name} />
      <meta name="twitter:description" content={product.description} />
      <meta name="twitter:image" content={product.images[0]} />
      
      {/* Canonical */}
      <link rel="canonical" href={`https://store.com/products/${product.slug}`} />
      
      {/* Product Schema */}
      <script type="application/ld+json">
        {JSON.stringify({
          "@context": "https://schema.org",
          "@type": "Product",
          "name": product.name,
          "description": product.description,
          "image": product.images,
          "offers": {
            "@type": "Offer",
            "price": product.price,
            "priceCurrency": "USD"
          }
        })}
      </script>
      
      {/* Page content */}
      <div className="product-images">
        {product.images.map(img => <img key={img} src={img} alt={product.name} />)}
      </div>
      
      <div className="product-info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        <p>{product.description}</p>
        <button>Add to Cart</button>
      </div>
    </div>
  );
}
```

---

## Internationalization (i18n)

```jsx
function BlogPost({ post, locale }) {
  return (
    <article lang={locale}>
      <title>{post.title[locale]}</title>
      <meta name="description" content={post.description[locale]} />
      
      {/* Alternate language versions */}
      <link rel="alternate" hrefLang="en" href={`/en/blog/${post.slug}`} />
      <link rel="alternate" hrefLang="es" href={`/es/blog/${post.slug}`} />
      <link rel="alternate" hrefLang="fr" href={`/fr/blog/${post.slug}`} />
      <link rel="alternate" hrefLang="x-default" href={`/blog/${post.slug}`} />
      
      <h1>{post.title[locale]}</h1>
      <div>{post.content[locale]}</div>
    </article>
  );
}
```

---

## Conditional Metadata

```jsx
function Page({ page, user }) {
  return (
    <div>
      <title>{page.title}</title>
      
      {/* Only for published pages */}
      {page.published && (
        <meta name="robots" content="index, follow" />
      )}
      
      {/* For draft/private pages */}
      {!page.published && (
        <meta name="robots" content="noindex, nofollow" />
      )}
      
      {/* For premium content */}
      {page.premium && (
        <meta name="article:content_tier" content="premium" />
      )}
      
      {/* Author info if available */}
      {user && (
        <>
          <meta name="author" content={user.name} />
          <link rel="author" href={`/users/${user.id}`} />
        </>
      )}
      
      <h1>{page.title}</h1>
      <div>{page.content}</div>
    </div>
  );
}
```

---

## Benefits

### 1. No External Library

**Before:**
```jsx
import { Helmet } from 'react-helmet';
import Head from 'next/head';
// Need external dependency
```

**After:**
```jsx
// Built into React 19!
<title>My Page</title>
```

---

### 2. Component-Scoped

```jsx
function BlogPost({ post }) {
  return (
    <article>
      {/* Metadata lives with component */}
      <title>{post.title}</title>
      <meta name="description" content={post.excerpt} />
      
      <h1>{post.title}</h1>
    </article>
  );
}

// Metadata and content colocated!
```

---

### 3. Automatic Updates

```jsx
function Page({ data }) {
  return (
    <div>
      {/* Updates automatically when data changes */}
      <title>{data.title}</title>
      <meta name="description" content={data.description} />
      
      <h1>{data.title}</h1>
    </div>
  );
}

// Title updates in <head> when data changes!
```

---

### 4. SSR Compatible

```jsx
// Works perfectly with Server Components and SSR
async function ProductPage({ params }) {
  const product = await db.product.findUnique({ where: { id: params.id } });
  
  return (
    <div>
      <title>{product.name}</title>
      <meta name="description" content={product.description} />
      
      <h1>{product.name}</h1>
    </div>
  );
}

// Metadata included in initial HTML!
```

---

## Comparison

### React 18 with react-helmet

```jsx
import { Helmet } from 'react-helmet';

function Page({ post }) {
  return (
    <div>
      <Helmet>
        <title>{post.title}</title>
        <meta name="description" content={post.excerpt} />
      </Helmet>
      
      <h1>{post.title}</h1>
    </div>
  );
}

// Requires:
// 1. Install react-helmet
// 2. Import Helmet component
// 3. Wrap metadata in Helmet
// 4. Extra div in render tree
```

---

### React 19 (Native)

```jsx
function Page({ post }) {
  return (
    <div>
      <title>{post.title}</title>
      <meta name="description" content={post.excerpt} />
      
      <h1>{post.title}</h1>
    </div>
  );
}

// Built-in:
// 1. No installation
// 2. No imports
// 3. Direct usage
// 4. Cleaner render tree
```

---

## Best Practices

**1. Include in every page component**
```jsx
function Page() {
  return (
    <div>
      <title>Page Title</title>
      <meta name="description" content="..." />
      {/* content */}
    </div>
  );
}
```

**2. Use descriptive titles**
```jsx
// ✅ Good
<title>iPhone 15 Pro - Smartphones - TechStore</title>

// ❌ Bad
<title>Product</title>
```

**3. Include Open Graph tags**
```jsx
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:image" content={image} />
```

**4. Add canonical URLs**
```jsx
<link rel="canonical" href={canonicalUrl} />
```

**5. Handle i18n properly**
```jsx
<link rel="alternate" hrefLang="en" href="/en/page" />
<link rel="alternate" hrefLang="es" href="/es/page" />
```

**6. Use robots meta for non-public pages**
```jsx
{!published && (
  <meta name="robots" content="noindex, nofollow" />
)}
```

---

## Migration from react-helmet

### Before

```jsx
import { Helmet } from 'react-helmet';

function Page({ data }) {
  return (
    <div>
      <Helmet>
        <title>{data.title}</title>
        <meta name="description" content={data.description} />
        <meta property="og:title" content={data.title} />
      </Helmet>
      
      <h1>{data.title}</h1>
    </div>
  );
}
```

---

### After

```jsx
// Remove import
// import { Helmet } from 'react-helmet';

function Page({ data }) {
  return (
    <div>
      {/* Remove Helmet wrapper */}
      <title>{data.title}</title>
      <meta name="description" content={data.description} />
      <meta property="og:title" content={data.title} />
      
      <h1>{data.title}</h1>
    </div>
  );
}

// 1. Remove react-helmet dependency
// 2. Remove Helmet import
// 3. Remove Helmet wrapper
// 4. Done!
```

---

**Interview Tips:**
- **Document metadata** = `<title>`, `<meta>`, `<link>` tags
- **Automatic hoisting** = React moves to `<head>` automatically
- **No library needed** = built into React 19
- **Component-scoped** = define metadata in components
- **Last wins** = deepest/last tag takes precedence
- **Dynamic** = updates when props/state change
- **SSR compatible** = works with server-side rendering
- **SEO friendly** = proper meta tags for search engines
- **Open Graph** = social media preview tags
- **Twitter Cards** = Twitter-specific meta tags
- **Canonical URLs** = avoid duplicate content
- **i18n support** = alternate language links
- **Conditional metadata** = based on props/state
- **Nested components** = child overrides parent
- **Type-safe** = TypeScript support
- **No wrapper needed** = direct tag usage
- **Simpler migration** = easy to migrate from react-helmet
- **Better DX** = cleaner, more intuitive API
- **Performance** = no extra library overhead
- **React 19+ only** = new feature
- **Replaces react-helmet** = no need for third-party
- **Replaces next/head** = built into React itself
- **Colocated** = metadata with component
- **Automatic cleanup** = removed when component unmounts

</details>

---

### 97. What are Actions in React 19?

<details>
<summary>View Answer</summary>

**Actions in React 19**

Actions are a new convention in React 19 for handling async transitions. They're functions passed to form `action` props or event handlers that automatically manage pending states, errors, and optimistic updates.

---

## What are Actions?

**Actions = Async functions that manage transitions automatically**

### Key Characteristics

1. **Async by default** - handle promises automatically
2. **Automatic pending state** - track loading status
3. **Error handling** - built-in error management
4. **Optimistic updates** - UI updates immediately
5. **Form integration** - work with `<form action={...}>`
6. **Progressive enhancement** - work without JavaScript
7. **Type-safe** - TypeScript support

---

## Basic Example

```jsx
function TodoForm() {
  // This is an Action
  async function addTodo(formData) {
    'use server';
    const text = formData.get('text');
    await db.todo.create({ data: { text } });
  }
  
  return (
    <form action={addTodo}>
      <input name="text" />
      <button type="submit">Add</button>
    </form>
  );
}
```

**React automatically handles:**
- Pending state
- Error handling
- Form reset
- Optimistic updates

---

## Actions vs Regular Functions

### Regular Async Function

```jsx
function Component() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  async function handleSubmit(e) {
    e.preventDefault();
    setLoading(true);
    setError(null);
    
    try {
      const formData = new FormData(e.target);
      await submitData(formData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="field" />
      <button disabled={loading}>
        {loading ? 'Submitting...' : 'Submit'}
      </button>
      {error && <div>{error}</div>}
    </form>
  );
}

// Manual:
// - State management
// - preventDefault
// - Loading state
// - Error handling
// - FormData creation
```

---

### Action

```jsx
import { useActionState } from 'react';

function Component() {
  async function submitAction(prevState, formData) {
    'use server';
    await submitData(formData);
    return { success: true };
  }
  
  const [state, action, isPending] = useActionState(
    submitAction,
    null
  );
  
  return (
    <form action={action}>
      <input name="field" />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

// Automatic:
// - State management
// - No preventDefault needed
// - Built-in pending state
// - Automatic error handling
// - Direct FormData access
```

**70% less code!**

---

## With useActionState Hook

```jsx
import { useActionState } from 'react';

function ContactForm() {
  async function submitContact(prevState, formData) {
    'use server';
    
    const email = formData.get('email');
    const message = formData.get('message');
    
    // Validate
    if (!email || !message) {
      return { error: 'All fields required' };
    }
    
    // Send email
    await sendEmail({ email, message });
    
    return { success: true, message: 'Email sent!' };
  }
  
  const [state, action, isPending] = useActionState(
    submitContact,
    { error: null, success: false }
  );
  
  return (
    <form action={action}>
      <input
        name="email"
        type="email"
        disabled={isPending}
      />
      
      <textarea
        name="message"
        disabled={isPending}
      />
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Sending...' : 'Send'}
      </button>
      
      {state.error && (
        <div className="error">{state.error}</div>
      )}
      
      {state.success && (
        <div className="success">{state.message}</div>
      )}
    </form>
  );
}
```

---

## With useFormStatus

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Saving...' : 'Save'}
    </button>
  );
}

function Form() {
  async function saveAction(formData) {
    'use server';
    await db.save(formData);
  }
  
  return (
    <form action={saveAction}>
      <input name="title" />
      <SubmitButton />  {/* Automatically knows pending state */}
    </form>
  );
}
```

---

## With useOptimistic

```jsx
import { useOptimistic } from 'react';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (current, newTodo) => [...current, newTodo]
  );
  
  async function addTodoAction(formData) {
    'use server';
    
    const text = formData.get('text');
    
    // Add optimistically
    addOptimisticTodo({
      id: Date.now(),
      text,
      completed: false
    });
    
    // Then save to server
    await db.todo.create({ data: { text } });
  }
  
  return (
    <div>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      
      <form action={addTodoAction}>
        <input name="text" />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```

---

## Action with Validation

```jsx
import { useActionState } from 'react';
import { z } from 'zod';

const schema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  password: z.string().min(8)
});

function RegistrationForm() {
  async function registerAction(prevState, formData) {
    'use server';
    
    // Validate
    const result = schema.safeParse({
      username: formData.get('username'),
      email: formData.get('email'),
      password: formData.get('password')
    });
    
    if (!result.success) {
      return {
        errors: result.error.flatten().fieldErrors
      };
    }
    
    // Create user
    await db.user.create({ data: result.data });
    
    return { success: true };
  }
  
  const [state, action, isPending] = useActionState(
    registerAction,
    { errors: {} }
  );
  
  return (
    <form action={action}>
      <div>
        <input name="username" />
        {state.errors?.username && (
          <span className="error">{state.errors.username}</span>
        )}
      </div>
      
      <div>
        <input name="email" type="email" />
        {state.errors?.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>
      
      <div>
        <input name="password" type="password" />
        {state.errors?.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>
      
      <button disabled={isPending}>Register</button>
    </form>
  );
}
```

---

## Action with Redirect

```jsx
import { redirect } from 'next/navigation';

async function createPostAction(formData) {
  'use server';
  
  const title = formData.get('title');
  const content = formData.get('content');
  
  const post = await db.post.create({
    data: { title, content }
  });
  
  // Redirect after success
  redirect(`/posts/${post.id}`);
}

function NewPostForm() {
  return (
    <form action={createPostAction}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

---

## Action with Revalidation

```jsx
import { revalidatePath } from 'next/cache';

async function updatePostAction(postId, formData) {
  'use server';
  
  const title = formData.get('title');
  const content = formData.get('content');
  
  await db.post.update({
    where: { id: postId },
    data: { title, content }
  });
  
  // Revalidate cache
  revalidatePath(`/posts/${postId}`);
}

function EditPostForm({ post }) {
  const updateWithId = updatePostAction.bind(null, post.id);
  
  return (
    <form action={updateWithId}>
      <input name="title" defaultValue={post.title} />
      <textarea name="content" defaultValue={post.content} />
      <button type="submit">Update</button>
    </form>
  );
}
```

---

## Action in Event Handler

```jsx
import { useState, useTransition } from 'react';

function DeleteButton({ postId }) {
  const [isPending, startTransition] = useTransition();
  
  async function deleteAction() {
    'use server';
    await db.post.delete({ where: { id: postId } });
    revalidatePath('/posts');
  }
  
  function handleDelete() {
    if (!confirm('Delete this post?')) return;
    
    startTransition(async () => {
      await deleteAction();
    });
  }
  
  return (
    <button
      onClick={handleDelete}
      disabled={isPending}
    >
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

---

## Multiple Actions in One Form

```jsx
function PostForm({ post }) {
  async function saveAction(formData) {
    'use server';
    const action = formData.get('action');
    
    if (action === 'save') {
      await db.post.update({
        where: { id: post.id },
        data: {
          title: formData.get('title'),
          published: false
        }
      });
    } else if (action === 'publish') {
      await db.post.update({
        where: { id: post.id },
        data: {
          title: formData.get('title'),
          published: true
        }
      });
    }
    
    revalidatePath(`/posts/${post.id}`);
  }
  
  return (
    <form action={saveAction}>
      <input name="title" defaultValue={post.title} />
      
      <button name="action" value="save" type="submit">
        Save Draft
      </button>
      
      <button name="action" value="publish" type="submit">
        Publish
      </button>
    </form>
  );
}
```

---

## Real-World Example: Comment System

```jsx
import { useActionState, useOptimistic } from 'react';

function CommentSection({ postId, comments }) {
  // Optimistic updates
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (current, newComment) => [...current, newComment]
  );
  
  // Create action
  async function createCommentAction(prevState, formData) {
    'use server';
    
    const text = formData.get('text');
    
    if (!text || text.length < 3) {
      return { error: 'Comment too short' };
    }
    
    // Add optimistically
    const optimisticComment = {
      id: Date.now(),
      text,
      author: 'You',
      createdAt: new Date(),
      pending: true
    };
    addOptimisticComment(optimisticComment);
    
    // Save to database
    await db.comment.create({
      data: {
        postId,
        text,
        authorId: session.userId
      }
    });
    
    revalidatePath(`/posts/${postId}`);
    
    return { success: true };
  }
  
  // Delete action
  async function deleteCommentAction(commentId) {
    'use server';
    await db.comment.delete({ where: { id: commentId } });
    revalidatePath(`/posts/${postId}`);
  }
  
  const [state, formAction, isPending] = useActionState(
    createCommentAction,
    { error: null }
  );
  
  return (
    <div className="comments">
      <h3>Comments ({optimisticComments.length})</h3>
      
      {optimisticComments.map(comment => (
        <div
          key={comment.id}
          className={comment.pending ? 'pending' : ''}
        >
          <div className="comment-header">
            <strong>{comment.author}</strong>
            <time>{comment.createdAt.toLocaleString()}</time>
          </div>
          <p>{comment.text}</p>
          
          {!comment.pending && (
            <form action={deleteCommentAction.bind(null, comment.id)}>
              <button type="submit" className="btn-link">
                Delete
              </button>
            </form>
          )}
        </div>
      ))}
      
      <form action={formAction} className="comment-form">
        <textarea
          name="text"
          placeholder="Add a comment..."
          disabled={isPending}
          required
        />
        
        <button type="submit" disabled={isPending}>
          {isPending ? 'Posting...' : 'Post Comment'}
        </button>
        
        {state.error && (
          <div className="error">{state.error}</div>
        )}
      </form>
    </div>
  );
}
```

---

## Progressive Enhancement

```jsx
// This form works even without JavaScript!
function ContactForm() {
  async function submitAction(formData) {
    'use server';
    
    const email = formData.get('email');
    const message = formData.get('message');
    
    await sendEmail({ email, message });
    
    // Redirect to thank you page
    redirect('/thank-you');
  }
  
  return (
    <form action={submitAction}>
      <input name="email" type="email" required />
      <textarea name="message" required />
      <button type="submit">Send</button>
    </form>
  );
}

// With JS: Ajax submission, pending states, optimistic updates
// Without JS: Traditional form submission to server
// Both work!
```

---

## Error Handling

```jsx
import { useActionState } from 'react';

function Form() {
  async function submitAction(prevState, formData) {
    'use server';
    
    try {
      const data = await processData(formData);
      return { success: true, data };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
  
  const [state, action, isPending] = useActionState(
    submitAction,
    { success: false, error: null }
  );
  
  return (
    <form action={action}>
      <input name="field" />
      <button disabled={isPending}>Submit</button>
      
      {state.error && (
        <div className="error">
          Error: {state.error}
        </div>
      )}
      
      {state.success && (
        <div className="success">
          Success!
        </div>
      )}
    </form>
  );
}
```

---

## Benefits

### 1. Automatic Pending State

```jsx
// No manual useState for loading!
const [state, action, isPending] = useActionState(...);
```

### 2. Automatic Error Handling

```jsx
// Errors automatically caught and passed to state
return { error: 'Something went wrong' };
```

### 3. Progressive Enhancement

```jsx
// Works without JavaScript
<form action={serverAction}>...</form>
```

### 4. Optimistic Updates

```jsx
// Built-in support for optimistic UI
useOptimistic + Actions = instant feedback
```

### 5. Form Integration

```jsx
// Direct form action support
<form action={action}>...</form>
```

### 6. Type Safety

```jsx
// TypeScript knows the action signature
async function myAction(formData: FormData): Promise<State>
```

---

## Best Practices

**1. Use Server Actions for mutations**
```jsx
async function createPost(formData) {
  'use server';
  await db.post.create(...);
}
```

**2. Validate on server**
```jsx
if (!email) {
  return { error: 'Email required' };
}
```

**3. Return useful state**
```jsx
return {
  success: true,
  message: 'Post created!',
  postId: post.id
};
```

**4. Revalidate after mutations**
```jsx
revalidatePath('/posts');
```

**5. Use with useOptimistic for instant feedback**
```jsx
addOptimistic(...);
await serverAction();
```

**6. Handle errors gracefully**
```jsx
try {
  await riskyOperation();
} catch (error) {
  return { error: error.message };
}
```

---

**Interview Tips:**
- **Actions** = async functions for transitions
- **Automatic pending** = built-in loading state
- **Form integration** = use with `<form action={...}>`
- **useActionState** = manage action state
- **useFormStatus** = access form pending state
- **useOptimistic** = optimistic updates with actions
- **Server Actions** = actions that run on server
- **Progressive enhancement** = work without JavaScript
- **Error handling** = automatic error management
- **Validation** = server-side validation
- **Revalidation** = update cache after mutations
- **Redirect** = navigate after success
- **Type-safe** = TypeScript support
- **Less boilerplate** = 70% less code
- **Better UX** = automatic pending states
- **FormData** = direct access to form data
- **Multiple actions** = multiple buttons in one form
- **Event handlers** = use in onClick too
- **React 19+ only** = new convention
- **Replaces manual state** = no useState for loading
- **Replaces manual errors** = automatic error handling
- **Built-in support** = first-class React feature
- **Composable** = work with all React 19 hooks

</details>

---

### 98. What is the ref as prop feature in React 19?

<details>
<summary>View Answer</summary>

**Ref as Prop in React 19**

React 19 allows you to pass `ref` as a regular prop instead of using `forwardRef`, simplifying component APIs and making refs work like any other prop.

---

## What Changed?

### React 18 (Old Way)

```jsx
import { forwardRef } from 'react';

// Must use forwardRef to accept ref
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// Usage
function Form() {
  const inputRef = useRef();
  return <Input ref={inputRef} />;
}
```

---

### React 19 (New Way)

```jsx
// ref is just a regular prop!
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// Usage (same)
function Form() {
  const inputRef = useRef();
  return <Input ref={inputRef} />;
}
```

**No `forwardRef` needed!**

---

## Simple Examples

### Before (React 18)

```jsx
import { forwardRef, useRef } from 'react';

// 1. Wrap in forwardRef
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} className="btn">
      {props.children}
    </button>
  );
});

// 2. Use it
function App() {
  const buttonRef = useRef();
  
  return <Button ref={buttonRef}>Click</Button>;
}
```

---

### After (React 19)

```jsx
import { useRef } from 'react';

// 1. Just accept ref as prop
function Button({ ref, children }) {
  return (
    <button ref={ref} className="btn">
      {children}
    </button>
  );
}

// 2. Use it (same)
function App() {
  const buttonRef = useRef();
  
  return <Button ref={buttonRef}>Click</Button>;
}
```

---

## With TypeScript

### Before (React 18)

```tsx
import { forwardRef, ButtonHTMLAttributes } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
}

// Complex type with forwardRef
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={`btn btn-${variant}`}
        {...props}
      />
    );
  }
);

export default Button;
```

---

### After (React 19)

```tsx
import { ButtonHTMLAttributes } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  ref?: React.Ref<HTMLButtonElement>;  // Just add ref to props
}

// Simple function component
function Button({ variant = 'primary', ref, ...props }: ButtonProps) {
  return (
    <button
      ref={ref}
      className={`btn btn-${variant}`}
      {...props}
    />
  );
}

export default Button;
```

**Simpler types!**

---

## Real-World Example: Input Component

### Before

```jsx
import { forwardRef, useState } from 'react';

const Input = forwardRef((
  { label, error, ...props },
  ref
) => {
  const [focused, setFocused] = useState(false);
  
  return (
    <div className="input-group">
      <label>{label}</label>
      <input
        ref={ref}
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        className={focused ? 'focused' : ''}
        {...props}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
});

export default Input;
```

---

### After

```jsx
import { useState } from 'react';

function Input({ label, error, ref, ...props }) {
  const [focused, setFocused] = useState(false);
  
  return (
    <div className="input-group">
      <label>{label}</label>
      <input
        ref={ref}
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        className={focused ? 'focused' : ''}
        {...props}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

export default Input;
```

---

## With useImperativeHandle

### Before

```jsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; }
  }));
  
  return <input ref={inputRef} {...props} />;
});
```

---

### After

```jsx
import { useImperativeHandle, useRef } from 'react';

function FancyInput({ ref, ...props }) {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; }
  }));
  
  return <input ref={inputRef} {...props} />;
}
```

---

## Complex Example: Modal Component

### Before (React 18)

```jsx
import { forwardRef, useImperativeHandle, useState } from 'react';

const Modal = forwardRef(({ title, children }, ref) => {
  const [isOpen, setIsOpen] = useState(false);
  
  useImperativeHandle(ref, () => ({
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
    toggle: () => setIsOpen(prev => !prev)
  }));
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={() => setIsOpen(false)}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={() => setIsOpen(false)}>×</button>
        </div>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
});

// Usage
function App() {
  const modalRef = useRef();
  
  return (
    <div>
      <button onClick={() => modalRef.current.open()}>
        Open Modal
      </button>
      <Modal ref={modalRef} title="My Modal">
        <p>Modal content</p>
      </Modal>
    </div>
  );
}
```

---

### After (React 19)

```jsx
import { useImperativeHandle, useState } from 'react';

function Modal({ ref, title, children }) {
  const [isOpen, setIsOpen] = useState(false);
  
  useImperativeHandle(ref, () => ({
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
    toggle: () => setIsOpen(prev => !prev)
  }));
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={() => setIsOpen(false)}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={() => setIsOpen(false)}>×</button>
        </div>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
}

// Usage (same)
function App() {
  const modalRef = useRef();
  
  return (
    <div>
      <button onClick={() => modalRef.current.open()}>
        Open Modal
      </button>
      <Modal ref={modalRef} title="My Modal">
        <p>Modal content</p>
      </Modal>
    </div>
  );
}
```

---

## Destructuring Ref

```jsx
// You can destructure ref like any prop
function Input({ ref, label, type = 'text', ...props }) {
  return (
    <div>
      <label>{label}</label>
      <input ref={ref} type={type} {...props} />
    </div>
  );
}

// Or use rest/spread
function Button({ ref, children, ...props }) {
  return (
    <button ref={ref} {...props}>
      {children}
    </button>
  );
}
```

---

## Multiple Refs

```jsx
import { useRef } from 'react';

function Form() {
  const emailRef = useRef();
  const passwordRef = useRef();
  const submitRef = useRef();
  
  function handleSubmit(e) {
    e.preventDefault();
    console.log(emailRef.current.value);
    console.log(passwordRef.current.value);
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <Input
        ref={emailRef}
        name="email"
        type="email"
        label="Email"
      />
      
      <Input
        ref={passwordRef}
        name="password"
        type="password"
        label="Password"
      />
      
      <Button ref={submitRef} type="submit">
        Login
      </Button>
    </form>
  );
}

function Input({ ref, label, ...props }) {
  return (
    <div>
      <label>{label}</label>
      <input ref={ref} {...props} />
    </div>
  );
}

function Button({ ref, children, ...props }) {
  return <button ref={ref} {...props}>{children}</button>;
}
```

---

## Ref Callbacks

```jsx
// Ref callbacks work the same
function Component({ ref }) {
  return (
    <input
      ref={node => {
        // Called when element mounts
        if (node) {
          node.focus();
        }
        
        // Pass to parent ref if exists
        if (typeof ref === 'function') {
          ref(node);
        } else if (ref) {
          ref.current = node;
        }
      }}
    />
  );
}
```

---

## Benefits

### 1. Simpler Component API

**Before:**
```jsx
const Component = forwardRef((props, ref) => { ... });
```

**After:**
```jsx
function Component({ ref, ...props }) { ... }
```

---

### 2. Easier TypeScript

**Before:**
```tsx
const Component = forwardRef<HTMLElement, Props>(...);
```

**After:**
```tsx
function Component({ ref, ...props }: Props & { ref?: Ref }) { ... }
```

---

### 3. No Wrapper Function

**Before:**
```jsx
export default forwardRef((props, ref) => <Component {...props} ref={ref} />);
```

**After:**
```jsx
export default function Component({ ref, ...props }) { ... }
```

---

### 4. Destructuring Works

```jsx
// Can destructure ref with other props
function Component({ ref, className, onClick, ...props }) {
  return <button ref={ref} className={className} onClick={onClick} {...props} />;
}
```

---

### 5. Better DevTools

Components show up with their actual names instead of `ForwardRef(Component)`.

---

## Migration Guide

### Step 1: Remove forwardRef import

```jsx
// Before
import { forwardRef } from 'react';

// After
// (no import needed)
```

---

### Step 2: Change function signature

```jsx
// Before
const Component = forwardRef((props, ref) => {
  return <div ref={ref}>{props.children}</div>;
});

// After
function Component({ ref, children }) {
  return <div ref={ref}>{children}</div>;
}
```

---

### Step 3: Update TypeScript types

```tsx
// Before
interface Props {
  name: string;
}
const Component = forwardRef<HTMLDivElement, Props>(
  ({ name }, ref) => <div ref={ref}>{name}</div>
);

// After
interface Props {
  name: string;
  ref?: React.Ref<HTMLDivElement>;
}
function Component({ name, ref }: Props) {
  return <div ref={ref}>{name}</div>;
}
```

---

## Backward Compatibility

```jsx
// forwardRef still works in React 19
const OldComponent = forwardRef((props, ref) => {
  return <div ref={ref}>Still works!</div>;
});

// But prefer the new way
function NewComponent({ ref }) {
  return <div ref={ref}>Better!</div>;
}
```

---

## Common Patterns

### Pattern 1: Simple Forwarding

```jsx
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

### Pattern 2: With Internal Ref

```jsx
import { useRef, useEffect } from 'react';

function Input({ ref, autofocus, ...props }) {
  const internalRef = useRef();
  
  useEffect(() => {
    if (autofocus) {
      internalRef.current?.focus();
    }
  }, [autofocus]);
  
  return <input ref={ref || internalRef} {...props} />;
}
```

### Pattern 3: With useImperativeHandle

```jsx
import { useImperativeHandle, useRef } from 'react';

function Input({ ref, ...props }) {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    value: () => inputRef.current.value
  }));
  
  return <input ref={inputRef} {...props} />;
}
```

### Pattern 4: Conditional Ref

```jsx
function Component({ ref, shouldAttachRef, ...props }) {
  return <div ref={shouldAttachRef ? ref : null} {...props} />;
}
```

---

## Best Practices

**1. Accept ref as a regular prop**
```jsx
function Component({ ref, ...props }) { ... }
```

**2. Document ref in TypeScript**
```tsx
interface Props {
  ref?: React.Ref<HTMLElement>;
}
```

**3. Use useImperativeHandle for custom methods**
```jsx
useImperativeHandle(ref, () => ({
  customMethod: () => { ... }
}));
```

**4. Forward ref even if not used immediately**
```jsx
// Future-proof
function Component({ ref, ...props }) {
  return <div ref={ref} {...props} />;
}
```

**5. Remove forwardRef in new code**
```jsx
// Prefer this
function Component({ ref }) { ... }

// Over this
const Component = forwardRef((props, ref) => { ... });
```

---

**Interview Tips:**
- **ref as prop** = ref is now a regular prop
- **No forwardRef** = don't need forwardRef wrapper
- **Simpler API** = cleaner component definition
- **Destructuring** = can destructure ref with other props
- **TypeScript** = simpler types without forwardRef
- **Better DevTools** = proper component names
- **Backward compatible** = forwardRef still works
- **useImperativeHandle** = still works with ref prop
- **Migration** = easy to migrate from forwardRef
- **React 19+ only** = new feature
- **Function components** = only for function components
- **Class components** = still need React.createRef
- **Ref callbacks** = still supported
- **Multiple refs** = can pass multiple refs to different elements
- **Conditional refs** = can conditionally attach refs
- **Less boilerplate** = no wrapper function needed
- **Cleaner code** = more readable
- **Standard prop** = treated like any other prop
- **No special handling** = ref is no longer special
- **Future-proof** = recommended way going forward
- **Libraries** = component libraries should adopt this
- **Performance** = same performance as forwardRef
- **Best practice** = prefer ref as prop in React 19+

</details>

---

### 99. How does React 19 improve hydration errors?

<details>
<summary>View Answer</summary>

**Hydration Errors Improvements in React 19**

React 19 dramatically improves hydration error messages with better diffs, more context, and actionable guidance, making it much easier to debug mismatches between server and client rendering.

---

## What are Hydration Errors?

**Hydration = Attaching React to server-rendered HTML**

When HTML from server doesn't match what React expects on client, you get a **hydration mismatch error**.

### Common Causes

1. **Different data** - Server and client have different data
2. **Browser-only APIs** - Using `window`, `localStorage` during render
3. **Random values** - Using `Math.random()`, `Date.now()` in render
4. **Conditional rendering** - Different conditions on server vs client
5. **Third-party scripts** - Scripts modifying DOM before hydration

---

## React 18 Error Messages (Before)

### Old Error (Vague)

```
Warning: Text content did not match. Server: "42" Client: "43"

Warning: An error occurred during hydration. The server HTML was 
replaced with client content in <div>.
```

**Problems:**
- Doesn't show WHERE the error is
- Doesn't show the actual diff
- Doesn't suggest how to fix
- Hard to find the problematic component

---

## React 19 Error Messages (After)

### New Error (Detailed)

```
Hydration failed because the server rendered HTML didn't match the client.

As a result this tree will be regenerated on the client. This can happen
if a SSR-ed Client Component used:

- A server/client branch `if (typeof window !== 'undefined')`
- Variable input such as `Date.now()` or `Math.random()`
- Date formatting in a user's locale
- External changing data without sending a snapshot of it
- Invalid HTML tag nesting

It can also happen if the client has a browser extension modifying the HTML.

https://react.dev/link/hydration-mismatch

  <App>
    <Page>
      <Layout>
+       <div>Client value: 43</div>
-       <div>Server value: 42</div>
```

**Improvements:**
- Shows exact location in component tree
- Shows actual diff with + and -
- Lists common causes
- Provides documentation link
- Suggests solutions

---

## Example: Date Formatting Error

### Code That Causes Error

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      {/* This causes hydration error! */}
      <time>{new Date(post.publishedAt).toLocaleDateString()}</time>
      <div>{post.content}</div>
    </article>
  );
}
```

---

### React 18 Error

```
Warning: Text content did not match.
Server: "12/25/2024" Client: "25/12/2024"
```

---

### React 19 Error

```
Hydration failed because the server rendered HTML didn't match the client.

This can happen if you're using date formatting that depends on the user's
locale. The server might be in a different timezone or locale than the client.

Solution: Format the date on the client only, or use the same locale on 
both server and client.

  <BlogPost>
+   <time>25/12/2024</time>
-   <time>12/25/2024</time>

See: https://react.dev/link/hydration-mismatch
```

**Much clearer!**

---

## Example: Browser API Error

### Code That Causes Error

```jsx
function UserGreeting() {
  // This causes hydration error!
  const username = localStorage.getItem('username') || 'Guest';
  
  return <div>Welcome, {username}!</div>;
}
```

---

### React 18 Error

```
Warning: Text content did not match.
Server: "Welcome, Guest!" Client: "Welcome, John!"
```

---

### React 19 Error

```
Hydration failed because the server rendered HTML didn't match the client.

The component used browser-only APIs during render:
- localStorage is not available on the server

Solution: Use useEffect to read from localStorage after hydration:

  useEffect(() => {
    const username = localStorage.getItem('username');
    setUsername(username);
  }, []);

  <UserGreeting>
+   <div>Welcome, John!</div>
-   <div>Welcome, Guest!</div>

See: https://react.dev/link/hydration-mismatch
```

---

## Example: Random Values Error

### Code That Causes Error

```jsx
function Widget() {
  // This causes hydration error!
  const id = Math.random().toString(36);
  
  return <div id={id}>Content</div>;
}
```

---

### React 18 Error

```
Warning: Prop `id` did not match.
Server: "abc123" Client: "xyz789"
```

---

### React 19 Error

```
Hydration failed because the server rendered HTML didn't match the client.

The component uses random values during render:
- Math.random() generates different values on server and client

Solution: Generate the ID once and pass it as a prop, or use useId():

  const id = useId(); // React's stable ID generator

  <Widget>
-   <div id="abc123">Content</div>
+   <div id="xyz789">Content</div>

See: https://react.dev/link/hydration-mismatch
```

---

## Visual Diff Display

### Text Content Mismatch

```jsx
function Counter({ count }) {
  return <div>Count: {count}</div>;
}
```

**React 19 Error:**
```
Hydration failed:

  <Counter>
    <div>
-     Count: 42
+     Count: 43
    </div>
```

---

### Attribute Mismatch

```jsx
function Button() {
  const theme = typeof window !== 'undefined' ? 'dark' : 'light';
  return <button className={theme}>Click</button>;
}
```

**React 19 Error:**
```
Hydration failed:

  <Button>
-   <button className="light">Click</button>
+   <button className="dark">Click</button>
```

---

### Element Type Mismatch

```jsx
function Heading({ level }) {
  const Tag = level === 1 ? 'h1' : 'h2';
  return <Tag>Title</Tag>;
}
```

**React 19 Error:**
```
Hydration failed:

  <Heading>
-   <h1>Title</h1>
+   <h2>Title</h2>
```

---

## Component Stack Trace

**React 19 shows full component tree:**

```
Hydration error in:

  App
    ↓
  Layout
    ↓
  Sidebar
    ↓
  UserWidget
    ↓
  UserGreeting  ← Error here!

The error occurred in the <UserGreeting> component.
```

---

## Common Fixes

### Fix 1: Use useEffect for Browser APIs

```jsx
// ❌ Bad - Causes hydration error
function Component() {
  const value = localStorage.getItem('key');
  return <div>{value}</div>;
}

// ✅ Good - Hydrates correctly
function Component() {
  const [value, setValue] = useState(null);
  
  useEffect(() => {
    setValue(localStorage.getItem('key'));
  }, []);
  
  return <div>{value || 'Loading...'}</div>;
}
```

---

### Fix 2: Use useId for Stable IDs

```jsx
// ❌ Bad - Random ID
function Component() {
  const id = Math.random().toString(36);
  return <div id={id}>Content</div>;
}

// ✅ Good - Stable ID
function Component() {
  const id = useId();
  return <div id={id}>Content</div>;
}
```

---

### Fix 3: Suppress Hydration Warning (When Intentional)

```jsx
function Clock() {
  const time = new Date().toLocaleTimeString();
  
  return (
    <time suppressHydrationWarning>
      {time}
    </time>
  );
}

// Use sparingly! Only when mismatch is intentional and harmless.
```

---

### Fix 4: Client-Only Rendering

```jsx
import dynamic from 'next/dynamic';

// Don't render on server
const ClientOnlyComponent = dynamic(
  () => import('./ClientOnlyComponent'),
  { ssr: false }
);

function Page() {
  return (
    <div>
      <ServerSafeContent />
      <ClientOnlyComponent />  {/* Only renders on client */}
    </div>
  );
}
```

---

### Fix 5: Same Data on Server and Client

```jsx
// ❌ Bad - Different data
function Component() {
  const timestamp = Date.now();  // Different on server/client
  return <div>{timestamp}</div>;
}

// ✅ Good - Pass as prop
function Component({ timestamp }) {
  return <div>{timestamp}</div>;
}

// Server generates timestamp once
const timestamp = Date.now();
<Component timestamp={timestamp} />
```

---

## Real-World Example: Theme Mismatch

### Problem Code

```jsx
function App() {
  // Causes hydration error!
  const theme = localStorage.getItem('theme') || 'light';
  
  return (
    <div className={theme}>
      <Header />
      <Content />
    </div>
  );
}
```

---

### React 19 Error Message

```
Hydration failed because the server rendered HTML didn't match the client.

Cause: localStorage is only available on the client.

On server: theme = "light" (default)
On client: theme = "dark" (from localStorage)

Solution: Use a two-pass render:

1. First render: Use default theme (matches server)
2. After hydration: Update to user's theme

  <App>
-   <div className="light">
+   <div className="dark">

See: https://react.dev/link/hydration-mismatch
```

---

### Fixed Code

```jsx
function App() {
  const [theme, setTheme] = useState('light');  // Default matches server
  const [isHydrated, setIsHydrated] = useState(false);
  
  useEffect(() => {
    // After hydration, load user's theme
    setIsHydrated(true);
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      setTheme(savedTheme);
    }
  }, []);
  
  return (
    <div className={theme}>
      <Header />
      <Content />
      {!isHydrated && <style>{/* Prevent flash */}</style>}
    </div>
  );
}
```

---

## Debugging Tips from React 19

### Tip 1: Check Component Tree

```
Error shows:
  App > Layout > Sidebar > UserCard
                              ^
                              Look here!
```

---

### Tip 2: Look for Browser APIs

```
Common culprits:
- localStorage
- sessionStorage
- window.location
- document.cookie
- navigator.userAgent
- matchMedia
```

---

### Tip 3: Check for Random Values

```
Avoid in render:
- Math.random()
- Date.now()
- new Date().toLocaleString()
- UUID generators

Use instead:
- Props passed from parent
- useId() for React IDs
- Server-generated IDs
```

---

### Tip 4: Check Third-Party Scripts

```
Browser extensions and scripts can modify DOM:
- Ad blockers
- Translation tools
- Accessibility tools
- Analytics scripts

Solution: Load scripts after hydration
```

---

## suppressHydrationWarning Attribute

### When to Use

```jsx
function Clock() {
  const time = new Date().toLocaleTimeString();
  
  return (
    <div suppressHydrationWarning>
      Current time: {time}
    </div>
  );
}

// Use when:
// - Mismatch is intentional
// - Mismatch is harmless (e.g., timestamps)
// - No better solution
```

---

### Don't Overuse

```jsx
// ❌ Bad - Hiding real problems
function Component() {
  return (
    <div suppressHydrationWarning>
      {/* Complex component with actual bugs */}
    </div>
  );
}

// ✅ Good - Fix the root cause
function Component() {
  const [data, setData] = useState(serverData);
  
  useEffect(() => {
    setData(clientData);
  }, []);
  
  return <div>{data}</div>;
}
```

---

## Comparison

### React 18

```
Pros:
- Shows that error occurred

Cons:
- Vague error messages
- No visual diff
- No component stack
- No suggested fixes
- Hard to debug
```

---

### React 19

```
Pros:
- Detailed error messages
- Visual diff with +/-
- Full component stack
- Lists common causes
- Suggests specific fixes
- Links to documentation
- Much easier to debug

Cons:
- (None - pure improvement!)
```

---

## Best Practices

**1. Avoid browser APIs in render**
```jsx
// Use useEffect instead
useEffect(() => {
  const value = localStorage.getItem('key');
}, []);
```

**2. Use useId for stable IDs**
```jsx
const id = useId();
```

**3. Pass timestamps as props**
```jsx
<Component timestamp={serverTimestamp} />
```

**4. Use same locale on server and client**
```jsx
const locale = 'en-US';
new Date().toLocaleDateString(locale);
```

**5. Test SSR thoroughly**
```jsx
// Test both server and client rendering
```

**6. Use suppressHydrationWarning sparingly**
```jsx
// Only for intentional, harmless mismatches
```

---

**Interview Tips:**
- **Hydration** = attaching React to server-rendered HTML
- **Hydration error** = mismatch between server and client HTML
- **React 19 improvements** = better error messages with diffs
- **Visual diff** = shows + and - for differences
- **Component stack** = shows where error occurred
- **Common causes** = browser APIs, random values, dates
- **Suggested fixes** = React 19 suggests solutions
- **Documentation links** = links to relevant docs
- **useEffect solution** = use for browser APIs
- **useId hook** = stable IDs across server/client
- **suppressHydrationWarning** = suppress intentional mismatches
- **Client-only components** = ssr: false option
- **Two-pass render** = match server, then update
- **localStorage problem** = not available on server
- **Date formatting** = different locales cause issues
- **Math.random() problem** = different values each time
- **Third-party scripts** = can modify DOM before hydration
- **Debug faster** = React 19 makes debugging much easier
- **Better DX** = improved developer experience
- **Actionable errors** = tells you exactly what to fix
- **Real diffs** = see actual differences
- **Component location** = know exactly where error is
- **Best practice** = avoid browser APIs in render

</details>

---

### 100. What are the Web Components improvements in React 19?

<details>
<summary>View Answer</summary>

**Web Components Improvements in React 19**

React 19 finally provides full, seamless support for Web Components with proper prop passing, event handling, and two-way integration, making it much easier to use custom elements in React apps.

---

## What's New?

### The Problem (React 18)

```jsx
// Web Component: <my-button color="red" count="5">

function App() {
  return (
    <my-button
      color="red"      // ❌ Passed as attribute (string only)
      count={5}        // ❌ Lost - primitives only
      onClick={fn}     // ❌ Lost - no event handler
    />
  );
}

// Only strings work, everything else ignored!
```

---

### The Solution (React 19)

```jsx
// Web Component: <my-button color="red" count="5">

function App() {
  return (
    <my-button
      color="red"      // ✅ Passed as property
      count={5}        // ✅ Passed as property (number)
      onClick={fn}     // ✅ Works!
    />
  );
}

// Everything works!
```

---

## Key Improvements

### 1. Properties vs Attributes

**React 18 (Old):**
```jsx
// Everything becomes attributes (strings)
<my-element
  title="Hello"           // attribute: "Hello"
  count={42}              // attribute: "42" (string!)
  data={{ x: 1 }}         // attribute: "[object Object]" (broken!)
  onClick={handler}       // ignored!
/>
```

---

**React 19 (New):**
```jsx
// Passed as properties (correct types)
<my-element
  title="Hello"           // property: "Hello"
  count={42}              // property: 42 (number!)
  data={{ x: 1 }}         // property: { x: 1 } (object!)
  onClick={handler}       // works!
/>
```

---

### 2. Complex Data Types

**React 18:**
```jsx
const data = { name: 'John', age: 30 };

// Doesn't work - becomes "[object Object]"
<user-card user={data} />

// Had to use ref workaround
const ref = useRef();
useEffect(() => {
  ref.current.user = data;  // Manual assignment
}, [data]);

<user-card ref={ref} />
```

---

**React 19:**
```jsx
const data = { name: 'John', age: 30 };

// Just works!
<user-card user={data} />
```

---

### 3. Event Handling

**React 18:**
```jsx
// Custom events don't work
<my-slider
  onChange={handleChange}  // Ignored!
/>

// Had to use ref + addEventListener
const ref = useRef();
useEffect(() => {
  const element = ref.current;
  element.addEventListener('change', handleChange);
  return () => element.removeEventListener('change', handleChange);
}, []);

<my-slider ref={ref} />
```

---

**React 19:**
```jsx
// Custom events just work!
<my-slider
  onChange={handleChange}  // Works!
/>
```

---

## Real-World Examples

### Example 1: Using a Web Component

```jsx
// Define Web Component
class MyCounter extends HTMLElement {
  constructor() {
    super();
    this._count = 0;
    this.attachShadow({ mode: 'open' });
  }
  
  set count(value) {
    this._count = value;
    this.render();
  }
  
  get count() {
    return this._count;
  }
  
  render() {
    this.shadowRoot.innerHTML = `
      <div>
        <p>Count: ${this._count}</p>
        <button id="inc">+</button>
        <button id="dec">-</button>
      </div>
    `;
    
    this.shadowRoot.getElementById('inc').onclick = () => {
      this.dispatchEvent(new CustomEvent('increment'));
    };
    
    this.shadowRoot.getElementById('dec').onclick = () => {
      this.dispatchEvent(new CustomEvent('decrement'));
    };
  }
  
  connectedCallback() {
    this.render();
  }
}

customElements.define('my-counter', MyCounter);
```

---

### Using in React 18 (Old Way)

```jsx
import { useRef, useEffect, useState } from 'react';

function App() {
  const [count, setCount] = useState(0);
  const counterRef = useRef();
  
  // Manual property assignment
  useEffect(() => {
    counterRef.current.count = count;
  }, [count]);
  
  // Manual event listeners
  useEffect(() => {
    const element = counterRef.current;
    
    const handleIncrement = () => setCount(c => c + 1);
    const handleDecrement = () => setCount(c => c - 1);
    
    element.addEventListener('increment', handleIncrement);
    element.addEventListener('decrement', handleDecrement);
    
    return () => {
      element.removeEventListener('increment', handleIncrement);
      element.removeEventListener('decrement', handleDecrement);
    };
  }, []);
  
  return <my-counter ref={counterRef} />;
}

// Lots of boilerplate!
```

---

### Using in React 19 (New Way)

```jsx
import { useState } from 'react';

function App() {
  const [count, setCount] = useState(0);
  
  return (
    <my-counter
      count={count}
      onIncrement={() => setCount(c => c + 1)}
      onDecrement={() => setCount(c => c - 1)}
    />
  );
}

// So much simpler!
```

---

### Example 2: Google Maps Web Component

```jsx
// React 18 - Complex
function Map() {
  const mapRef = useRef();
  const center = { lat: 37.7749, lng: -122.4194 };
  
  useEffect(() => {
    // Manual property assignment
    mapRef.current.center = center;
    mapRef.current.zoom = 12;
    
    // Manual event listener
    const handleMarkerClick = (e) => {
      console.log('Marker clicked:', e.detail);
    };
    
    mapRef.current.addEventListener('marker-click', handleMarkerClick);
    
    return () => {
      mapRef.current.removeEventListener('marker-click', handleMarkerClick);
    };
  }, [center]);
  
  return <google-map ref={mapRef} />;
}
```

---

```jsx
// React 19 - Simple
function Map() {
  const center = { lat: 37.7749, lng: -122.4194 };
  
  return (
    <google-map
      center={center}
      zoom={12}
      onMarkerClick={(e) => console.log('Marker clicked:', e.detail)}
    />
  );
}
```

---

### Example 3: Third-Party Component Library

```jsx
// Shoelace UI components
import '@shoelace-style/shoelace/dist/components/button/button.js';
import '@shoelace-style/shoelace/dist/components/input/input.js';
import '@shoelace-style/shoelace/dist/components/dialog/dialog.js';

function App() {
  const [open, setOpen] = useState(false);
  const [value, setValue] = useState('');
  
  return (
    <div>
      {/* React 19 - Props work! */}
      <sl-button
        variant="primary"
        size="large"
        onClick={() => setOpen(true)}
      >
        Open Dialog
      </sl-button>
      
      <sl-dialog
        label="Enter your name"
        open={open}
        onSlRequestClose={() => setOpen(false)}
      >
        <sl-input
          value={value}
          onSlInput={(e) => setValue(e.target.value)}
          placeholder="Your name"
        />
        
        <sl-button
          slot="footer"
          variant="primary"
          onClick={() => {
            console.log('Name:', value);
            setOpen(false);
          }}
        >
          Submit
        </sl-button>
      </sl-dialog>
    </div>
  );
}
```

---

## Property Types Support

### Strings

```jsx
<my-element title="Hello" />  // Works in both React 18 & 19
```

### Numbers

```jsx
<my-element count={42} />  // React 18: "42", React 19: 42
```

### Booleans

```jsx
<my-element disabled={true} />  // React 18: attribute, React 19: property
```

### Objects

```jsx
<my-element data={{ x: 1, y: 2 }} />  // React 18: broken, React 19: works!
```

### Arrays

```jsx
<my-element items={[1, 2, 3]} />  // React 18: broken, React 19: works!
```

### Functions

```jsx
<my-element onClick={handler} />  // React 18: ignored, React 19: works!
```

---

## Event Naming Convention

```jsx
// Web Component dispatches: CustomEvent('change')
// React handler: onChange

<my-slider
  onChange={handleChange}  // React 19 maps this correctly
/>

// Web Component dispatches: CustomEvent('value-changed')
// React handler: onValueChanged (camelCase)

<my-input
  onValueChanged={handleChange}  // React 19 converts to 'value-changed'
/>
```

---

## Refs Still Work

```jsx
function App() {
  const elementRef = useRef();
  
  function handleClick() {
    // Can still access element directly
    elementRef.current.customMethod();
    console.log(elementRef.current.someProperty);
  }
  
  return (
    <div>
      <my-element ref={elementRef} />
      <button onClick={handleClick}>Call Method</button>
    </div>
  );
}
```

---

## TypeScript Support

### Declare Web Component Types

```tsx
// types/web-components.d.ts
import 'react';

declare global {
  namespace JSX {
    interface IntrinsicElements {
      'my-counter': {
        count?: number;
        onIncrement?: () => void;
        onDecrement?: () => void;
        ref?: React.Ref<HTMLElement>;
      };
      
      'my-slider': {
        value?: number;
        min?: number;
        max?: number;
        onChange?: (e: CustomEvent<number>) => void;
        ref?: React.Ref<HTMLElement>;
      };
    }
  }
}
```

---

### Use with Type Safety

```tsx
function App() {
  const [count, setCount] = useState(0);
  
  return (
    <my-counter
      count={count}  // Type-checked!
      onIncrement={() => setCount(c => c + 1)}  // Type-checked!
      // @ts-expect-error - Property doesn't exist
      invalidProp="value"
    />
  );
}
```

---

## Comparison

### React 18

```
Pros:
- Web Components work (basic)

Cons:
- Only strings work reliably
- Objects/arrays become "[object Object]"
- Custom events need manual addEventListener
- Requires ref workarounds
- Lots of boilerplate
- Poor developer experience
```

---

### React 19

```
Pros:
- All data types work
- Objects/arrays passed correctly
- Custom events work with onEventName
- No ref workarounds needed
- Clean, simple code
- Great developer experience
- First-class support

Cons:
- (None - pure improvement!)
```

---

## Migration Guide

### Before (React 18)

```jsx
import { useRef, useEffect } from 'react';

function Component() {
  const ref = useRef();
  const data = { name: 'John' };
  
  useEffect(() => {
    ref.current.data = data;
    
    const handler = (e) => console.log(e.detail);
    ref.current.addEventListener('change', handler);
    
    return () => {
      ref.current.removeEventListener('change', handler);
    };
  }, [data]);
  
  return <my-element ref={ref} />;
}
```

---

### After (React 19)

```jsx
function Component() {
  const data = { name: 'John' };
  
  return (
    <my-element
      data={data}
      onChange={(e) => console.log(e.detail)}
    />
  );
}
```

**Remove:**
- useRef (if only for props)
- useEffect for property assignment
- useEffect for event listeners
- addEventListener/removeEventListener

**Add:**
- Props directly on element
- Event handlers as props

---

## Best Practices

**1. Use props directly**
```jsx
<my-element count={5} data={obj} />
```

**2. Use camelCase for event handlers**
```jsx
<my-element onChange={handler} onValueChanged={handler} />
```

**3. Declare TypeScript types**
```tsx
interface IntrinsicElements {
  'my-element': { ... };
}
```

**4. Use ref only when needed**
```jsx
// For calling methods
const ref = useRef();
ref.current.customMethod();
```

**5. Test with different data types**
```jsx
// Strings, numbers, objects, arrays, functions
```

---

## Real-World Use Cases

### 1. Design System Integration

```jsx
import '@company/design-system';

function App() {
  return (
    <ds-layout>
      <ds-header title="My App" user={currentUser} />
      <ds-sidebar items={menuItems} onItemClick={handleNavigation} />
      <ds-content>
        <ds-card data={cardData} onAction={handleAction} />
      </ds-content>
    </ds-layout>
  );
}
```

### 2. Third-Party Widgets

```jsx
<stripe-payment-element
  amount={1000}
  currency="usd"
  clientSecret={secret}
  onSuccess={handleSuccess}
  onError={handleError}
/>
```

### 3. Micro-Frontends

```jsx
<micro-frontend
  name="shopping-cart"
  config={cartConfig}
  user={currentUser}
  onCheckout={handleCheckout}
/>
```

---

**Interview Tips:**
- **Web Components** = custom HTML elements
- **React 19 support** = full, seamless integration
- **Properties vs attributes** = React 19 uses properties (correct types)
- **Complex data** = objects, arrays now work
- **Event handling** = custom events work with onEventName
- **No ref workarounds** = don't need ref for props anymore
- **Less boilerplate** = 70% less code
- **Type safety** = declare types for TypeScript
- **camelCase events** = onChange, onValueChanged
- **All types supported** = strings, numbers, objects, arrays, functions
- **Better DX** = much better developer experience
- **Third-party libs** = easier to use Shoelace, Material, etc.
- **Design systems** = integrate company design systems
- **Micro-frontends** = better support for micro-frontend architecture
- **React 18 problems** = only strings worked reliably
- **Migration easy** = remove useEffect workarounds
- **First-class support** = Web Components are first-class citizens
- **Custom Elements API** = standard Web Components API
- **Shadow DOM** = works correctly
- **Events bubble** = custom events bubble properly
- **Refs optional** = only need for imperative methods
- **Game changer** = makes Web Components practical in React
- **Industry standard** = aligns React with web standards

</details>
