# GraphQL & Data Fetching

> Expert / Architect Level (5+ years)

---

## Questions

221. What is GraphQL and how does it integrate with React?
222. What is Apollo Client and how do you use it?
223. What is Relay framework?
224. How do you implement GraphQL subscriptions in React?
225. What is normalized caching in GraphQL clients?b 
226. How do you handle GraphQL errors in React?
227. What is the difference between REST and GraphQL?
228. How do you implement optimistic UI with GraphQL?
229. What is code generation with GraphQL?
230. How do you implement pagination with GraphQL?

---

## Detailed Answers

### 221. What is GraphQL and how does it integrate with React?

<details>
<summary>View Answer</summary>

**GraphQL** is a **query language for APIs** and a **runtime for fulfilling queries** with your existing data. Developed by Facebook in 2012 and open-sourced in 2015, GraphQL provides a more efficient, powerful, and flexible alternative to REST APIs. It integrates seamlessly with React through client libraries that provide hooks and components for data fetching.

---

#### **What is GraphQL?**

GraphQL is a **specification** that defines:

1. **Query Language**: A syntax for requesting specific data
2. **Schema Definition Language (SDL)**: A type system for defining your API
3. **Runtime**: Server-side execution engine for processing queries

**Core Concepts:**

```graphql
# Schema Definition (Server-Side)
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
}

type Query {
  user(id: ID!): User
  users: [User!]!
  post(id: ID!): Post
  posts(published: Boolean): [Post!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String): User!
  deleteUser(id: ID!): Boolean!
  createPost(title: String!, content: String!, authorId: ID!): Post!
}

type Subscription {
  postCreated: Post!
  userUpdated(userId: ID!): User!
}
```

**Query Example (Client-Side):**

```graphql
# Request exactly what you need
query GetUserWithPosts {
  user(id: "123") {
    id
    name
    email
    posts {
      id
      title
      published
    }
  }
}

# Response - shape matches query structure
{
  "data": {
    "user": {
      "id": "123",
      "name": "John Doe",
      "email": "john@example.com",
      "posts": [
        {
          "id": "1",
          "title": "GraphQL Introduction",
          "published": true
        },
        {
          "id": "2",
          "title": "React Hooks",
          "published": false
        }
      ]
    }
  }
}
```

---

#### **GraphQL vs REST**

| Feature | REST | GraphQL |
|---------|------|---------|
| **Endpoints** | Multiple endpoints (`/users`, `/posts`) | Single endpoint (`/graphql`) |
| **Data Fetching** | Fixed data structure per endpoint | Request exactly what you need |
| **Over-fetching** | Common (get more data than needed) | ❌ None (specify exact fields) |
| **Under-fetching** | Common (need multiple requests) | ❌ None (fetch related data in one query) |
| **Versioning** | Needs v1, v2, etc. | No versioning needed (add fields, deprecate old ones) |
| **Type System** | No built-in types | ✅ Strong type system |
| **Documentation** | Manual (Swagger, etc.) | ✅ Auto-generated from schema |
| **Caching** | Easy (HTTP caching) | Complex (needs normalized cache) |

**Example - REST vs GraphQL:**

```javascript
// REST Approach
// Problem: Need 3 requests to get user with posts and comments

// Request 1: Get user
GET /api/users/123
{
  "id": "123",
  "name": "John",
  "email": "john@example.com",
  "profileImage": "...", // Don't need this
  "bio": "...",          // Don't need this
  "settings": {...}       // Don't need this
}

// Request 2: Get user's posts
GET /api/users/123/posts
[
  {
    "id": "1",
    "title": "Post 1",
    "content": "...",    // Don't need this
    "createdAt": "...",  // Don't need this
    "updatedAt": "..."   // Don't need this
  }
]

// Request 3: Get comments for each post
GET /api/posts/1/comments
// ... more requests

// GraphQL Approach
// Solution: One request, exact data needed

query GetUserDashboard {
  user(id: "123") {
    name
    email
    posts {
      id
      title
      comments {
        id
        text
        author {
          name
        }
      }
    }
  }
}

// Single response with exact data structure needed
```

---

#### **How GraphQL Integrates with React**

GraphQL integrates with React through **client libraries** that provide:

1. **Query hooks** for data fetching
2. **Mutation hooks** for data updates
3. **Caching** for performance
4. **Type safety** with TypeScript
5. **Developer tools** for debugging

**Popular GraphQL Client Libraries:**

| Library | Best For | Pros | Cons |
|---------|----------|------|------|
| **Apollo Client** | Most projects | Full-featured, large ecosystem, great DevTools | Bundle size (33KB) |
| **Relay** | Large apps | Optimal performance, powerful caching | Steep learning curve, complex setup |
| **urql** | Smaller apps | Lightweight (7KB), simple API | Fewer features than Apollo |
| **graphql-request** | Simple needs | Minimal (5KB), no caching | No hooks, manual management |

---

#### **Integration Method 1: Apollo Client (Most Popular)**

**Installation:**

```bash
npm install @apollo/client graphql
```

**Setup:**

```typescript
// src/apolloClient.ts
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';

const httpLink = createHttpLink({
  uri: 'https://api.example.com/graphql',
});

export const apolloClient = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          // Cache configuration
        },
      },
    },
  }),
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network',
    },
  },
});
```

```typescript
// src/App.tsx
import { ApolloProvider } from '@apollo/client';
import { apolloClient } from './apolloClient';

function App() {
  return (
    <ApolloProvider client={apolloClient}>
      <YourComponents />
    </ApolloProvider>
  );
}
```

**Using Queries in React:**

```typescript
// src/graphql/queries.ts
import { gql } from '@apollo/client';

export const GET_USER = gql`
  query GetUser($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      posts {
        id
        title
        published
        createdAt
      }
    }
  }
`;

export const GET_POSTS = gql`
  query GetPosts($published: Boolean) {
    posts(published: $published) {
      id
      title
      content
      author {
        id
        name
      }
    }
  }
`;
```

```typescript
// src/components/UserProfile.tsx
import { useQuery } from '@apollo/client';
import { GET_USER } from '../graphql/queries';

interface User {
  id: string;
  name: string;
  email: string;
  posts: Post[];
}

interface Post {
  id: string;
  title: string;
  published: boolean;
  createdAt: string;
}

function UserProfile({ userId }: { userId: string }) {
  const { loading, error, data, refetch } = useQuery<{ user: User }>(
    GET_USER,
    {
      variables: { userId },
      // Optional configurations
      fetchPolicy: 'cache-first', // cache-first, network-only, cache-and-network
      pollInterval: 5000, // Poll every 5 seconds
      skip: !userId, // Skip query if no userId
    }
  );

  if (loading) return <div>Loading user...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data?.user) return <div>User not found</div>;

  const { user } = data;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      
      <button onClick={() => refetch()}>
        Refresh
      </button>

      <h2>Posts</h2>
      {user.posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <span>{post.published ? '✅ Published' : '📝 Draft'}</span>
          <time>{new Date(post.createdAt).toLocaleDateString()}</time>
        </div>
      ))}
    </div>
  );
}
```

**Using Mutations:**

```typescript
// src/graphql/mutations.ts
import { gql } from '@apollo/client';

export const CREATE_POST = gql`
  mutation CreatePost($title: String!, $content: String!, $authorId: ID!) {
    createPost(title: $title, content: $content, authorId: $authorId) {
      id
      title
      content
      published
      author {
        id
        name
      }
    }
  }
`;

export const UPDATE_USER = gql`
  mutation UpdateUser($id: ID!, $name: String!) {
    updateUser(id: $id, name: $name) {
      id
      name
      email
    }
  }
`;
```

```typescript
// src/components/CreatePostForm.tsx
import { useMutation } from '@apollo/client';
import { CREATE_POST } from '../graphql/mutations';
import { GET_POSTS } from '../graphql/queries';

function CreatePostForm({ authorId }: { authorId: string }) {
  const [title, setTitle] = React.useState('');
  const [content, setContent] = React.useState('');

  const [createPost, { loading, error, data }] = useMutation(CREATE_POST, {
    // Refetch queries after mutation
    refetchQueries: [
      { query: GET_POSTS, variables: { published: false } }
    ],
    
    // Or update cache manually for better performance
    update(cache, { data }) {
      const newPost = data?.createPost;
      if (!newPost) return;

      cache.modify({
        fields: {
          posts(existingPosts = []) {
            const newPostRef = cache.writeFragment({
              data: newPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                  content
                }
              `,
            });
            return [...existingPosts, newPostRef];
          },
        },
      });
    },

    // Optimistic response (update UI immediately)
    optimisticResponse: {
      createPost: {
        __typename: 'Post',
        id: 'temp-id',
        title,
        content,
        published: false,
        author: {
          __typename: 'User',
          id: authorId,
          name: 'Current User',
        },
      },
    },

    onCompleted: (data) => {
      console.log('Post created:', data.createPost);
      setTitle('');
      setContent('');
    },

    onError: (error) => {
      console.error('Failed to create post:', error);
    },
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    try {
      await createPost({
        variables: {
          title,
          content,
          authorId,
        },
      });
    } catch (err) {
      // Error handled by onError callback
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        required
      />
      
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Post content"
        required
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      
      {error && <div>Error: {error.message}</div>}
      {data && <div>Post created successfully!</div>}
    </form>
  );
}
```

---

#### **Integration Method 2: urql (Lightweight Alternative)**

**Installation:**

```bash
npm install urql graphql
```

**Setup:**

```typescript
// src/App.tsx
import { createClient, Provider } from 'urql';

const client = createClient({
  url: 'https://api.example.com/graphql',
  fetchOptions: {
    credentials: 'include',
  },
});

function App() {
  return (
    <Provider value={client}>
      <YourComponents />
    </Provider>
  );
}
```

**Using Queries:**

```typescript
import { useQuery } from 'urql';

const GET_USERS = `
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

function UserList() {
  const [result, reexecuteQuery] = useQuery({ query: GET_USERS });

  const { data, fetching, error } = result;

  if (fetching) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button onClick={() => reexecuteQuery({ requestPolicy: 'network-only' })}>
        Refresh
      </button>
      
      {data.users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

#### **Integration Method 3: Manual Fetch (No Library)**

For simple use cases, you can use native `fetch`:

```typescript
// src/utils/graphqlClient.ts
export async function graphqlRequest<T>(
  query: string,
  variables?: Record<string, any>
): Promise<T> {
  const response = await fetch('https://api.example.com/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, variables }),
  });

  const { data, errors } = await response.json();

  if (errors) {
    throw new Error(errors[0].message);
  }

  return data;
}
```

```typescript
// src/components/UserProfile.tsx
import { useState, useEffect } from 'react';
import { graphqlRequest } from '../utils/graphqlClient';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const query = `
      query GetUser($userId: ID!) {
        user(id: $userId) {
          id
          name
          email
        }
      }
    `;

    graphqlRequest(query, { userId })
      .then(data => setUser(data.user))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>User not found</div>;

  return <div>{user.name}</div>;
}
```

---

#### **Real-World Example: Social Media Feed**

```typescript
// src/graphql/schema.graphql
type Query {
  feed(limit: Int, offset: Int): FeedConnection!
  post(id: ID!): Post
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  likePost(postId: ID!): Post!
  addComment(postId: ID!, text: String!): Comment!
}

type Subscription {
  newPost: Post!
  postLiked(postId: ID!): Post!
}

type FeedConnection {
  posts: [Post!]!
  pageInfo: PageInfo!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  likes: Int!
  comments: [Comment!]!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  createdAt: DateTime!
}

type User {
  id: ID!
  name: String!
  avatar: String
}
```

```typescript
// src/components/SocialFeed.tsx
import { useQuery, useMutation } from '@apollo/client';
import { gql } from '@apollo/client';

const GET_FEED = gql`
  query GetFeed($limit: Int!, $offset: Int!) {
    feed(limit: $limit, offset: $offset) {
      posts {
        id
        title
        content
        likes
        createdAt
        author {
          id
          name
          avatar
        }
        comments {
          id
          text
          author {
            name
          }
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(postId: $postId) {
      id
      likes
    }
  }
`;

const ADD_COMMENT = gql`
  mutation AddComment($postId: ID!, $text: String!) {
    addComment(postId: $postId, text: $text) {
      id
      text
      author {
        name
      }
    }
  }
`;

function SocialFeed() {
  const [limit] = useState(10);
  const [offset, setOffset] = useState(0);

  const { loading, error, data, fetchMore } = useQuery(GET_FEED, {
    variables: { limit, offset },
  });

  const [likePost] = useMutation(LIKE_POST, {
    optimisticResponse: (vars) => ({
      likePost: {
        __typename: 'Post',
        id: vars.postId,
        likes: -1, // Will be updated from server
      },
    }),
  });

  const [addComment] = useMutation(ADD_COMMENT, {
    update(cache, { data: { addComment } }) {
      cache.modify({
        id: cache.identify({ __typename: 'Post', id: addComment.postId }),
        fields: {
          comments(existingComments = []) {
            const newCommentRef = cache.writeFragment({
              data: addComment,
              fragment: gql`
                fragment NewComment on Comment {
                  id
                  text
                  author {
                    name
                  }
                }
              `,
            });
            return [...existingComments, newCommentRef];
          },
        },
      });
    },
  });

  const handleLoadMore = () => {
    fetchMore({
      variables: {
        offset: data.feed.posts.length,
      },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        return {
          feed: {
            ...fetchMoreResult.feed,
            posts: [...prev.feed.posts, ...fetchMoreResult.feed.posts],
          },
        };
      },
    });
  };

  const handleLike = (postId: string) => {
    likePost({ variables: { postId } });
  };

  const handleComment = (postId: string, text: string) => {
    addComment({ variables: { postId, text } });
  };

  if (loading) return <div>Loading feed...</div>;
  if (error) return <div>Error loading feed: {error.message}</div>;

  return (
    <div className="feed">
      {data.feed.posts.map(post => (
        <div key={post.id} className="post">
          <div className="post-header">
            <img src={post.author.avatar} alt={post.author.name} />
            <div>
              <h3>{post.author.name}</h3>
              <time>{new Date(post.createdAt).toLocaleDateString()}</time>
            </div>
          </div>

          <h2>{post.title}</h2>
          <p>{post.content}</p>

          <div className="post-actions">
            <button onClick={() => handleLike(post.id)}>
              ❤️ {post.likes}
            </button>
            <span>💬 {post.comments.length}</span>
          </div>

          <div className="comments">
            {post.comments.map(comment => (
              <div key={comment.id}>
                <strong>{comment.author.name}</strong>: {comment.text}
              </div>
            ))}
          </div>

          <CommentForm
            onSubmit={(text) => handleComment(post.id, text)}
          />
        </div>
      ))}

      {data.feed.pageInfo.hasNextPage && (
        <button onClick={handleLoadMore}>Load More</button>
      )}
    </div>
  );
}
```

---

#### **Advanced Features**

**1. Authentication:**

```typescript
import { setContext } from '@apollo/client/link/context';

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});
```

**2. Error Handling:**

```typescript
import { onError } from '@apollo/client/link/error';

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) => {
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      );
    });
  }
  if (networkError) {
    console.log(`[Network error]: ${networkError}`);
  }
});

const client = new ApolloClient({
  link: errorLink.concat(authLink).concat(httpLink),
  cache: new InMemoryCache(),
});
```

**3. Local State Management:**

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        isLoggedIn: {
          read() {
            return !!localStorage.getItem('token');
          },
        },
      },
    },
  },
});

// Query local state
const IS_LOGGED_IN = gql`
  query IsLoggedIn {
    isLoggedIn @client
  }
`;
```

**4. Subscriptions (Real-Time Updates):**

```typescript
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { createClient as createWsClient } from 'graphql-ws';
import { split } from '@apollo/client';
import { getMainDefinition } from '@apollo/client/utilities';

const wsLink = new GraphQLWsLink(
  createWsClient({
    url: 'ws://api.example.com/graphql',
  })
);

const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink
);

// Use subscription in component
import { useSubscription } from '@apollo/client';

const NEW_POST_SUBSCRIPTION = gql`
  subscription OnNewPost {
    newPost {
      id
      title
      author {
        name
      }
    }
  }
`;

function NewPostNotifications() {
  const { data, loading } = useSubscription(NEW_POST_SUBSCRIPTION);

  if (loading) return <div>Listening...</div>;
  if (data) {
    return <div>New post: {data.newPost.title}</div>;
  }
  return null;
}
```

---

#### **TypeScript Integration**

**Code Generation (Recommended):**

```bash
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-operations @graphql-codegen/typescript-react-apollo
```

```yaml
# codegen.yml
overwrite: true
schema: "http://localhost:4000/graphql"
documents: "src/**/*.graphql"
generates:
  src/generated/graphql.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
    config:
      withHooks: true
      withComponent: false
```

```graphql
# src/graphql/queries.graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
  }
}
```

Run: `npx graphql-codegen`

```typescript
// Generated types in src/generated/graphql.ts
import { useGetUserQuery } from '../generated/graphql';

function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useGetUserQuery({
    variables: { userId },
  });
  
  // Fully typed data
  const userName = data?.user?.name; // string | undefined
}
```

---

#### **Benefits of GraphQL with React**

1. **Declarative Data Fetching**: Components declare exactly what data they need
2. **Type Safety**: TypeScript integration with auto-generated types
3. **Single Request**: Fetch related data in one query (no waterfalls)
4. **Automatic Caching**: Apollo/Relay cache data automatically
5. **Optimistic UI**: Update UI before server response
6. **Real-Time Updates**: Subscriptions for live data
7. **Developer Experience**: GraphQL Playground, DevTools, auto-documentation
8. **No Over-fetching**: Request only needed fields
9. **Backward Compatible**: Add fields without breaking existing queries
10. **Strong Ecosystem**: Tons of tools, libraries, and community support

---

#### **When to Use GraphQL**

**✅ Use GraphQL When:**

- Building data-intensive applications (dashboards, social media, e-commerce)
- Need to fetch related data (user → posts → comments)
- Multiple clients need different data shapes (web, mobile, desktop)
- Want type safety and auto-generated documentation
- Building a graph-like data model
- Need real-time updates via subscriptions
- Want to reduce API roundtrips

**❌ Avoid GraphQL When:**

- Simple CRUD operations (REST is simpler)
- File uploads are primary feature (GraphQL adds complexity)
- Team lacks GraphQL experience
- Backend doesn't support GraphQL
- Caching strategy is critical (HTTP caching easier with REST)
- Need maximum performance (REST can be faster for simple queries)

---

#### **Summary**

GraphQL integrates with React through **client libraries** (Apollo Client, Relay, urql) that provide:

1. **Hooks**: `useQuery`, `useMutation`, `useSubscription`
2. **Declarative queries**: Components specify exact data needs
3. **Automatic caching**: Normalized cache for performance
4. **Type safety**: TypeScript integration via code generation
5. **Optimistic updates**: Instant UI feedback
6. **Real-time data**: WebSocket subscriptions

**Key Integration Steps:**

1. Install GraphQL client (`@apollo/client`)
2. Wrap app in provider (`<ApolloProvider>`)
3. Define queries/mutations with `gql`
4. Use hooks in components (`useQuery`, `useMutation`)
5. Handle loading/error states
6. Configure caching and optimizations

GraphQL transforms how React apps fetch data, moving from imperative REST calls to **declarative, type-safe data requirements** that co-locate with components.

</details>

---

### 222. What is Apollo Client and how do you use it?

<details>
<summary>View Answer</summary>

**Apollo Client** is a comprehensive **state management library** for JavaScript that enables you to manage both local and remote data with GraphQL. It's the most popular GraphQL client for React, providing intelligent caching, optimistic UI updates, and a powerful developer experience.

Apollo Client handles the entire data management lifecycle:
- **Fetching** data from GraphQL APIs
- **Caching** responses for performance
- **Updating** the UI automatically
- **Managing** loading and error states
- **Synchronizing** local and remote data

---

#### **Core Concepts**

**What Apollo Client Provides:**

| Feature | Description | Benefit |
|---------|-------------|---------|
| **Intelligent Cache** | Normalized cache stores data once | No duplicate data, automatic updates |
| **Declarative Data Fetching** | Components declare data needs via hooks | Co-located queries, easy to understand |
| **Automatic Updates** | UI updates when cache changes | No manual state management |
| **Optimistic UI** | Update UI before server response | Instant feedback, better UX |
| **Polling & Refetching** | Keep data fresh automatically | Real-time feel without WebSockets |
| **Pagination** | Built-in pagination support | Easy infinite scroll |
| **Local State** | Manage local state alongside remote | Single source of truth |
| **DevTools** | Powerful debugging tools | Inspect cache, queries, mutations |
| **Type Safety** | TypeScript support with codegen | Catch errors at compile time |

---

#### **Installation and Setup**

**Installation:**

```bash
npm install @apollo/client graphql

# Or with yarn
yarn add @apollo/client graphql
```

**Basic Setup:**

```typescript
// src/apolloClient.ts
import { ApolloClient, InMemoryCache, ApolloProvider } from '@apollo/client';

// Create Apollo Client instance
export const client = new ApolloClient({
  uri: 'https://api.example.com/graphql', // GraphQL endpoint
  cache: new InMemoryCache(), // Cache configuration
});
```

```typescript
// src/App.tsx
import { ApolloProvider } from '@apollo/client';
import { client } from './apolloClient';

function App() {
  return (
    <ApolloProvider client={client}>
      {/* Your app components */}
      <YourComponents />
    </ApolloProvider>
  );
}

export default App;
```

---

#### **Advanced Configuration**

**Complete Setup with Authentication, Error Handling, and Multiple Links:**

```typescript
// src/apolloClient.ts
import {
  ApolloClient,
  InMemoryCache,
  createHttpLink,
  from,
  ApolloLink,
} from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import { onError } from '@apollo/client/link/error';
import { RetryLink } from '@apollo/client/link/retry';

// 1. HTTP Link - Connection to GraphQL server
const httpLink = createHttpLink({
  uri: process.env.REACT_APP_GRAPHQL_URL || 'http://localhost:4000/graphql',
  credentials: 'include', // Send cookies
});

// 2. Auth Link - Add authentication headers
const authLink = setContext((_, { headers }) => {
  // Get auth token from localStorage
  const token = localStorage.getItem('authToken');
  
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
      'client-name': 'web-app',
      'client-version': '1.0.0',
    },
  };
});

// 3. Error Link - Handle errors globally
const errorLink = onError(({ graphQLErrors, networkError, operation, forward }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path, extensions }) => {
      console.error(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      );
      
      // Handle authentication errors
      if (extensions?.code === 'UNAUTHENTICATED') {
        // Clear token and redirect to login
        localStorage.removeItem('authToken');
        window.location.href = '/login';
      }
      
      // Handle specific error codes
      if (extensions?.code === 'RATE_LIMITED') {
        // Show rate limit message
        console.warn('Rate limited. Please try again later.');
      }
    });
  }

  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
    // Show offline notification
  }
});

// 4. Retry Link - Retry failed requests
const retryLink = new RetryLink({
  delay: {
    initial: 300,
    max: 3000,
    jitter: true,
  },
  attempts: {
    max: 3,
    retryIf: (error, _operation) => {
      // Retry on network errors but not on GraphQL errors
      return !!error && !error.result;
    },
  },
});

// 5. Logging Link - Log all operations (dev only)
const loggingLink = new ApolloLink((operation, forward) => {
  console.log(`[GraphQL Request] ${operation.operationName}`);
  console.log('Variables:', operation.variables);
  
  const startTime = Date.now();
  
  return forward(operation).map((response) => {
    const duration = Date.now() - startTime;
    console.log(`[GraphQL Response] ${operation.operationName} (${duration}ms)`);
    return response;
  });
});

// 6. Cache Configuration
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        // Pagination: Merge results
        posts: {
          keyArgs: ['filter'], // Use 'filter' as cache key
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          },
        },
        
        // Local-only field
        isLoggedIn: {
          read() {
            return !!localStorage.getItem('authToken');
          },
        },
      },
    },
    
    User: {
      fields: {
        // Computed field
        fullName: {
          read(_, { readField }) {
            const firstName = readField('firstName');
            const lastName = readField('lastName');
            return `${firstName} ${lastName}`;
          },
        },
      },
    },
    
    Post: {
      keyFields: ['id'], // Custom cache key
    },
  },
});

// 7. Combine all links
const link = from([
  errorLink,
  retryLink,
  process.env.NODE_ENV === 'development' ? loggingLink : null,
  authLink,
  httpLink,
].filter(Boolean) as ApolloLink[]);

// 8. Create Apollo Client
export const client = new ApolloClient({
  link,
  cache,
  
  // Default options
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network', // Always check server
      errorPolicy: 'all', // Return partial data on errors
    },
    query: {
      fetchPolicy: 'cache-first', // Use cache when available
      errorPolicy: 'all',
    },
    mutate: {
      errorPolicy: 'all',
    },
  },
  
  // Enable dev tools
  connectToDevTools: process.env.NODE_ENV === 'development',
});
```

---

#### **Core API: Queries**

**Defining Queries:**

```typescript
// src/graphql/queries.ts
import { gql } from '@apollo/client';

export const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      firstName
      lastName
      email
      avatar
      posts {
        id
        title
        createdAt
      }
    }
  }
`;

export const GET_POSTS = gql`
  query GetPosts($limit: Int, $offset: Int, $filter: PostFilter) {
    posts(limit: $limit, offset: $offset, filter: $filter) {
      id
      title
      content
      published
      author {
        id
        firstName
        lastName
      }
      comments {
        id
        text
      }
    }
  }
`;

export const SEARCH_USERS = gql`
  query SearchUsers($query: String!) {
    searchUsers(query: $query) {
      id
      firstName
      lastName
      email
    }
  }
`;
```

**Using `useQuery` Hook:**

```typescript
// src/components/UserProfile.tsx
import { useQuery } from '@apollo/client';
import { GET_USER } from '../graphql/queries';

interface User {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  avatar: string;
  posts: Post[];
}

interface Post {
  id: string;
  title: string;
  createdAt: string;
}

interface GetUserData {
  user: User;
}

interface GetUserVars {
  id: string;
}

function UserProfile({ userId }: { userId: string }) {
  const { loading, error, data, refetch, networkStatus } = useQuery<
    GetUserData,
    GetUserVars
  >(GET_USER, {
    variables: { id: userId },
    
    // Fetch policies
    fetchPolicy: 'cache-first', // cache-first, cache-and-network, network-only, no-cache, cache-only
    nextFetchPolicy: 'cache-first', // Policy after initial fetch
    
    // Polling
    pollInterval: 0, // Poll every N milliseconds (0 = disabled)
    
    // Conditional execution
    skip: !userId, // Skip query if no userId
    
    // Notifications
    notifyOnNetworkStatusChange: true, // Re-render on network status change
    
    // Error policy
    errorPolicy: 'all', // none, all, ignore
    
    // Callbacks
    onCompleted: (data) => {
      console.log('Query completed:', data);
    },
    onError: (error) => {
      console.error('Query error:', error);
    },
  });

  // Loading states
  if (loading && !data) return <div>Loading user...</div>;
  
  if (error) {
    return (
      <div>
        <p>Error: {error.message}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  if (!data?.user) return <div>User not found</div>;

  const { user } = data;

  return (
    <div className="user-profile">
      <img src={user.avatar} alt={user.firstName} />
      <h1>{user.firstName} {user.lastName}</h1>
      <p>{user.email}</p>
      
      {/* Refetch button */}
      <button 
        onClick={() => refetch()}
        disabled={networkStatus === 4} // 4 = refetching
      >
        {networkStatus === 4 ? 'Refreshing...' : 'Refresh'}
      </button>

      {/* User's posts */}
      <h2>Posts ({user.posts.length})</h2>
      {user.posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <time>{new Date(post.createdAt).toLocaleDateString()}</time>
        </div>
      ))}
    </div>
  );
}
```

**Lazy Queries (Manual Execution):**

```typescript
import { useLazyQuery } from '@apollo/client';
import { SEARCH_USERS } from '../graphql/queries';

function UserSearch() {
  const [searchUsers, { loading, error, data, called }] = useLazyQuery(
    SEARCH_USERS
  );

  const handleSearch = (query: string) => {
    searchUsers({ variables: { query } });
  };

  return (
    <div>
      <input
        type="text"
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search users..."
      />
      
      {loading && <div>Searching...</div>}
      {error && <div>Error: {error.message}</div>}
      
      {called && data?.searchUsers && (
        <ul>
          {data.searchUsers.map(user => (
            <li key={user.id}>
              {user.firstName} {user.lastName}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

#### **Core API: Mutations**

**Defining Mutations:**

```typescript
// src/graphql/mutations.ts
import { gql } from '@apollo/client';

export const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
      published
      author {
        id
        firstName
        lastName
      }
    }
  }
`;

export const UPDATE_POST = gql`
  mutation UpdatePost($id: ID!, $input: UpdatePostInput!) {
    updatePost(id: $id, input: $input) {
      id
      title
      content
      published
    }
  }
`;

export const DELETE_POST = gql`
  mutation DeletePost($id: ID!) {
    deletePost(id: $id)
  }
`;

export const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(postId: $postId) {
      id
      likes
      isLikedByMe
    }
  }
`;
```

**Using `useMutation` Hook:**

```typescript
// src/components/CreatePostForm.tsx
import { useMutation } from '@apollo/client';
import { CREATE_POST } from '../graphql/mutations';
import { GET_POSTS } from '../graphql/queries';

interface CreatePostInput {
  title: string;
  content: string;
  published: boolean;
}

function CreatePostForm() {
  const [title, setTitle] = React.useState('');
  const [content, setContent] = React.useState('');

  const [createPost, { loading, error, data, reset }] = useMutation(
    CREATE_POST,
    {
      // Option 1: Refetch queries after mutation
      refetchQueries: [
        { query: GET_POSTS, variables: { limit: 10, offset: 0 } },
        'GetPosts', // Can also use operation name
      ],
      
      // Option 2: Update cache manually (more efficient)
      update(cache, { data }) {
        const newPost = data?.createPost;
        if (!newPost) return;

        // Read existing posts from cache
        const existingPosts = cache.readQuery({
          query: GET_POSTS,
          variables: { limit: 10, offset: 0 },
        });

        if (existingPosts) {
          // Write updated posts to cache
          cache.writeQuery({
            query: GET_POSTS,
            variables: { limit: 10, offset: 0 },
            data: {
              posts: [newPost, ...existingPosts.posts],
            },
          });
        }
      },
      
      // Option 3: Optimistic response (immediate UI update)
      optimisticResponse: {
        createPost: {
          __typename: 'Post',
          id: 'temp-id-' + Date.now(),
          title,
          content,
          published: false,
          author: {
            __typename: 'User',
            id: 'current-user-id',
            firstName: 'Current',
            lastName: 'User',
          },
        },
      },
      
      // Error policy
      errorPolicy: 'all',
      
      // Callbacks
      onCompleted: (data) => {
        console.log('Post created:', data.createPost);
        setTitle('');
        setContent('');
      },
      
      onError: (error) => {
        console.error('Failed to create post:', error);
      },
    }
  );

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    try {
      await createPost({
        variables: {
          input: {
            title,
            content,
            published: false,
          },
        },
      });
    } catch (err) {
      // Error handled by onError callback
      console.error('Mutation error:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        required
      />
      
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Post content"
        required
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      
      {error && (
        <div className="error">
          Error: {error.message}
          <button onClick={reset}>Dismiss</button>
        </div>
      )}
      
      {data && (
        <div className="success">
          Post created successfully!
        </div>
      )}
    </form>
  );
}
```

**Optimistic UI with Rollback:**

```typescript
// src/components/LikeButton.tsx
import { useMutation } from '@apollo/client';
import { LIKE_POST } from '../graphql/mutations';

function LikeButton({ postId, likes, isLiked }: { 
  postId: string; 
  likes: number; 
  isLiked: boolean;
}) {
  const [likePost, { loading }] = useMutation(LIKE_POST, {
    variables: { postId },
    
    // Optimistic response - updates UI immediately
    optimisticResponse: {
      likePost: {
        __typename: 'Post',
        id: postId,
        likes: isLiked ? likes - 1 : likes + 1,
        isLikedByMe: !isLiked,
      },
    },
    
    // If mutation fails, Apollo automatically rolls back
    onError: (error) => {
      console.error('Failed to like post:', error);
      // UI automatically reverts to previous state
    },
  });

  return (
    <button 
      onClick={() => likePost()} 
      disabled={loading}
      className={isLiked ? 'liked' : ''}
    >
      ❤️ {likes}
    </button>
  );
}
```

---

#### **Cache Management**

**Reading from Cache:**

```typescript
import { useApolloClient } from '@apollo/client';
import { GET_USER } from '../graphql/queries';

function UserComponent() {
  const client = useApolloClient();

  // Read query from cache
  const cachedUser = client.readQuery({
    query: GET_USER,
    variables: { id: '123' },
  });

  // Read fragment from cache
  const userFragment = client.readFragment({
    id: 'User:123', // Cache ID
    fragment: gql`
      fragment UserInfo on User {
        id
        firstName
        lastName
      }
    `,
  });

  return <div>{cachedUser?.user.firstName}</div>;
}
```

**Writing to Cache:**

```typescript
// Direct cache write
client.writeQuery({
  query: GET_USER,
  variables: { id: '123' },
  data: {
    user: {
      __typename: 'User',
      id: '123',
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
    },
  },
});

// Write fragment
client.writeFragment({
  id: 'User:123',
  fragment: gql`
    fragment UserName on User {
      firstName
      lastName
    }
  `,
  data: {
    firstName: 'Jane',
    lastName: 'Smith',
  },
});
```

**Modifying Cache:**

```typescript
// Update specific fields
cache.modify({
  id: cache.identify({ __typename: 'Post', id: '456' }),
  fields: {
    likes(existingLikes) {
      return existingLikes + 1;
    },
    comments(existingComments = [], { readField }) {
      // Add new comment
      const newComment = { __typename: 'Comment', id: 'new-id', text: 'Hello' };
      return [...existingComments, newComment];
    },
  },
});

// Remove item from cache
cache.evict({ id: 'Post:456' });
cache.gc(); // Garbage collect dangling references
```

**Cache Policies:**

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        // Custom read function
        user: {
          read(existing, { args, toReference }) {
            return existing || toReference({
              __typename: 'User',
              id: args?.id,
            });
          },
        },
        
        // Pagination
        posts: {
          keyArgs: ['filter'], // Cache key
          merge(existing = [], incoming, { args }) {
            const offset = args?.offset || 0;
            const merged = existing.slice(0);
            
            for (let i = 0; i < incoming.length; i++) {
              merged[offset + i] = incoming[i];
            }
            
            return merged;
          },
        },
      },
    },
  },
});
```

---

#### **Advanced Features**

**1. Subscriptions (Real-Time Updates):**

```typescript
import { useSubscription } from '@apollo/client';
import { gql } from '@apollo/client';

const NEW_MESSAGE_SUBSCRIPTION = gql`
  subscription OnNewMessage($chatId: ID!) {
    newMessage(chatId: $chatId) {
      id
      text
      author {
        id
        name
      }
      createdAt
    }
  }
`;

function ChatMessages({ chatId }: { chatId: string }) {
  const { data, loading } = useSubscription(NEW_MESSAGE_SUBSCRIPTION, {
    variables: { chatId },
    
    // Update cache when new message arrives
    onSubscriptionData: ({ client, subscriptionData }) => {
      const newMessage = subscriptionData.data?.newMessage;
      if (!newMessage) return;

      // Add to messages list in cache
      client.cache.modify({
        fields: {
          messages(existingMessages = []) {
            const newMessageRef = client.cache.writeFragment({
              data: newMessage,
              fragment: gql`
                fragment NewMessage on Message {
                  id
                  text
                  author {
                    id
                    name
                  }
                }
              `,
            });
            return [...existingMessages, newMessageRef];
          },
        },
      });
    },
  });

  if (loading) return <div>Connecting...</div>;
  if (data) {
    return <div>New message: {data.newMessage.text}</div>;
  }
  return null;
}
```

**2. Pagination (Infinite Scroll):**

```typescript
import { useQuery } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($limit: Int!, $offset: Int!) {
    posts(limit: $limit, offset: $offset) {
      id
      title
      content
    }
  }
`;

function PostList() {
  const { data, loading, fetchMore } = useQuery(GET_POSTS, {
    variables: { limit: 10, offset: 0 },
  });

  const loadMore = () => {
    fetchMore({
      variables: {
        offset: data?.posts.length || 0,
      },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        
        return {
          posts: [...prev.posts, ...fetchMoreResult.posts],
        };
      },
    });
  };

  return (
    <div>
      {data?.posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
      <button onClick={loadMore} disabled={loading}>
        Load More
      </button>
    </div>
  );
}
```

**3. Local State Management:**

```typescript
import { makeVar, useReactiveVar } from '@apollo/client';

// Create reactive variable
export const isLoggedInVar = makeVar<boolean>(
  !!localStorage.getItem('token')
);

export const cartItemsVar = makeVar<CartItem[]>([]);

// Use in components
function CartBadge() {
  const cartItems = useReactiveVar(cartItemsVar);
  
  return <span>{cartItems.length}</span>;
}

// Update reactive var
cartItemsVar([...cartItemsVar(), newItem]);

// Use in cache policies
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        isLoggedIn: {
          read() {
            return isLoggedInVar();
          },
        },
        cartItems: {
          read() {
            return cartItemsVar();
          },
        },
      },
    },
  },
});
```

**4. File Uploads:**

```typescript
import { gql, useMutation } from '@apollo/client';

const UPLOAD_FILE = gql`
  mutation UploadFile($file: Upload!) {
    uploadFile(file: $file) {
      id
      url
      filename
    }
  }
`;

function FileUpload() {
  const [uploadFile, { loading }] = useMutation(UPLOAD_FILE);

  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    try {
      const { data } = await uploadFile({
        variables: { file },
      });
      
      console.log('Uploaded:', data.uploadFile.url);
    } catch (error) {
      console.error('Upload failed:', error);
    }
  };

  return (
    <input 
      type="file" 
      onChange={handleChange} 
      disabled={loading}
    />
  );
}
```

**5. Batching Requests:**

```typescript
import { BatchHttpLink } from '@apollo/client/link/batch-http';

const batchLink = new BatchHttpLink({
  uri: 'https://api.example.com/graphql',
  batchMax: 10, // Max queries per batch
  batchInterval: 20, // Wait 20ms to batch
});
```

---

#### **Real-World Example: E-Commerce Product Page**

```typescript
// src/components/ProductPage.tsx
import { useQuery, useMutation } from '@apollo/client';
import { gql } from '@apollo/client';

// Queries
const GET_PRODUCT = gql`
  query GetProduct($id: ID!) {
    product(id: $id) {
      id
      name
      description
      price
      images
      inStock
      rating
      reviews {
        id
        rating
        comment
        author {
          name
          avatar
        }
      }
    }
  }
`;

const GET_CART = gql`
  query GetCart {
    cart @client {
      items {
        productId
        quantity
      }
    }
  }
`;

// Mutations
const ADD_TO_CART = gql`
  mutation AddToCart($productId: ID!, $quantity: Int!) {
    addToCart(productId: $productId, quantity: $quantity) {
      id
      items {
        productId
        quantity
      }
    }
  }
`;

const ADD_REVIEW = gql`
  mutation AddReview($productId: ID!, $rating: Int!, $comment: String!) {
    addReview(productId: $productId, rating: $rating, comment: $comment) {
      id
      rating
      comment
      author {
        name
        avatar
      }
    }
  }
`;

interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  images: string[];
  inStock: boolean;
  rating: number;
  reviews: Review[];
}

interface Review {
  id: string;
  rating: number;
  comment: string;
  author: {
    name: string;
    avatar: string;
  };
}

function ProductPage({ productId }: { productId: string }) {
  // Fetch product
  const { 
    loading: productLoading, 
    error: productError, 
    data: productData 
  } = useQuery<{ product: Product }>(GET_PRODUCT, {
    variables: { id: productId },
    fetchPolicy: 'cache-and-network',
  });

  // Add to cart mutation
  const [addToCart, { loading: addingToCart }] = useMutation(ADD_TO_CART, {
    optimisticResponse: {
      addToCart: {
        __typename: 'Cart',
        id: 'cart-1',
        items: [{ productId, quantity: 1 }],
      },
    },
    update(cache, { data }) {
      // Update cart count in UI immediately
      cache.modify({
        fields: {
          cart(existingCart) {
            return data?.addToCart || existingCart;
          },
        },
      });
    },
    onCompleted: () => {
      alert('Added to cart!');
    },
  });

  // Add review mutation
  const [addReview, { loading: submittingReview }] = useMutation(ADD_REVIEW, {
    update(cache, { data }) {
      const newReview = data?.addReview;
      if (!newReview) return;

      // Add review to product's reviews
      cache.modify({
        id: cache.identify({ __typename: 'Product', id: productId }),
        fields: {
          reviews(existingReviews = []) {
            const newReviewRef = cache.writeFragment({
              data: newReview,
              fragment: gql`
                fragment NewReview on Review {
                  id
                  rating
                  comment
                  author {
                    name
                    avatar
                  }
                }
              `,
            });
            return [...existingReviews, newReviewRef];
          },
        },
      });
    },
  });

  const handleAddToCart = () => {
    addToCart({
      variables: { productId, quantity: 1 },
    });
  };

  const handleSubmitReview = (rating: number, comment: string) => {
    addReview({
      variables: { productId, rating, comment },
    });
  };

  if (productLoading && !productData) return <div>Loading product...</div>;
  if (productError) return <div>Error: {productError.message}</div>;
  if (!productData?.product) return <div>Product not found</div>;

  const { product } = productData;

  return (
    <div className="product-page">
      {/* Product Images */}
      <div className="images">
        {product.images.map((img, i) => (
          <img key={i} src={img} alt={product.name} />
        ))}
      </div>

      {/* Product Info */}
      <div className="info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        <p className="rating">⭐ {product.rating.toFixed(1)}</p>
        <p>{product.description}</p>

        <button 
          onClick={handleAddToCart}
          disabled={!product.inStock || addingToCart}
        >
          {addingToCart ? 'Adding...' : 
           product.inStock ? 'Add to Cart' : 'Out of Stock'}
        </button>
      </div>

      {/* Reviews */}
      <div className="reviews">
        <h2>Reviews ({product.reviews.length})</h2>
        {product.reviews.map(review => (
          <div key={review.id} className="review">
            <img src={review.author.avatar} alt={review.author.name} />
            <div>
              <strong>{review.author.name}</strong>
              <span>⭐ {review.rating}</span>
              <p>{review.comment}</p>
            </div>
          </div>
        ))}

        <ReviewForm 
          onSubmit={handleSubmitReview}
          loading={submittingReview}
        />
      </div>
    </div>
  );
}
```

---

#### **Testing with Apollo Client**

**MockedProvider for Testing:**

```typescript
// src/components/__tests__/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import UserProfile from '../UserProfile';
import { GET_USER } from '../../graphql/queries';

const mocks = [
  {
    request: {
      query: GET_USER,
      variables: { id: '123' },
    },
    result: {
      data: {
        user: {
          id: '123',
          firstName: 'John',
          lastName: 'Doe',
          email: 'john@example.com',
        },
      },
    },
  },
];

test('renders user profile', async () => {
  render(
    <MockedProvider mocks={mocks} addTypename={false}>
      <UserProfile userId="123" />
    </MockedProvider>
  );

  // Loading state
  expect(screen.getByText('Loading user...')).toBeInTheDocument();

  // Wait for data
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});

// Test error state
const errorMocks = [
  {
    request: {
      query: GET_USER,
      variables: { id: '123' },
    },
    error: new Error('User not found'),
  },
];

test('handles error', async () => {
  render(
    <MockedProvider mocks={errorMocks} addTypename={false}>
      <UserProfile userId="123" />
    </MockedProvider>
  );

  await waitFor(() => {
    expect(screen.getByText(/Error: User not found/)).toBeInTheDocument();
  });
});
```

---

#### **Performance Optimization**

**Best Practices:**

```typescript
// 1. Use fragments for reusable fields
const USER_FIELDS = gql`
  fragment UserFields on User {
    id
    firstName
    lastName
    avatar
  }
`;

const GET_POST = gql`
  ${USER_FIELDS}
  query GetPost($id: ID!) {
    post(id: $id) {
      id
      title
      author {
        ...UserFields
      }
    }
  }
`;

// 2. Prefetch data on hover
function PostLink({ postId }: { postId: string }) {
  const client = useApolloClient();

  const prefetch = () => {
    client.query({
      query: GET_POST,
      variables: { id: postId },
    });
  };

  return (
    <Link 
      to={`/post/${postId}`}
      onMouseEnter={prefetch}
    >
      View Post
    </Link>
  );
}

// 3. Use @defer and @stream (experimental)
const GET_POST_DEFERRED = gql`
  query GetPost($id: ID!) {
    post(id: $id) {
      id
      title
      content
      ... @defer {
        comments {
          id
          text
        }
      }
    }
  }
`;

// 4. Batch queries in a single request
// Apollo automatically batches if using BatchHttpLink

// 5. Use cache persistence
import { persistCache } from 'apollo3-cache-persist';

const cache = new InMemoryCache();

await persistCache({
  cache,
  storage: window.localStorage,
});
```

---

#### **Apollo Client DevTools**

**Install Browser Extension:**

```bash
# Chrome/Edge
https://chrome.google.com/webstore -> "Apollo Client DevTools"

# Firefox
https://addons.mozilla.org -> "Apollo Client Developer Tools"
```

**Features:**

1. **GraphiQL**: Execute queries/mutations against your GraphQL server
2. **Cache Inspector**: View entire normalized cache
3. **Queries**: See all active queries, their status, and variables
4. **Mutations**: View mutation history
5. **Performance**: Network timing and cache hits/misses

---

#### **Summary**

**Apollo Client provides:**

1. **Declarative Data Fetching**: `useQuery`, `useMutation`, `useSubscription` hooks
2. **Intelligent Caching**: Normalized cache with automatic updates
3. **Optimistic UI**: Instant feedback before server response
4. **Local State Management**: Reactive variables and client-only fields
5. **Type Safety**: TypeScript support with code generation
6. **DevTools**: Powerful debugging and inspection tools
7. **Performance**: Batching, caching, pagination, prefetching
8. **Real-Time**: WebSocket subscriptions for live data
9. **Testing**: MockedProvider for easy component testing
10. **Ecosystem**: Large community, extensive documentation, many integrations

**Core Workflow:**

1. **Install**: `npm install @apollo/client graphql`
2. **Setup**: Create client with `ApolloClient` and wrap app in `<ApolloProvider>`
3. **Define**: Write queries/mutations with `gql`
4. **Use**: Call `useQuery`/`useMutation` hooks in components
5. **Configure**: Set up caching, error handling, authentication
6. **Optimize**: Implement pagination, prefetching, optimistic UI
7. **Debug**: Use Apollo DevTools to inspect cache and queries

Apollo Client is the **most popular and feature-rich** GraphQL client for React, providing everything needed to build modern data-driven applications with excellent developer experience and performance.

</details>

---

### 223. What is Relay framework?

<details>
<summary>View Answer</summary>

**Relay** is a JavaScript framework built by **Facebook (Meta)** for building data-driven React applications powered by GraphQL. While Apollo Client focuses on ease of use and flexibility, Relay is designed for **maximum performance and scalability** at the cost of more strict conventions and steeper learning curve.

Relay is used in production at Facebook/Meta, powering Facebook.com, Instagram, Workplace, and other Meta properties serving billions of users.

---

#### **What is Relay?**

Relay is a **full-stack GraphQL framework** that consists of:

1. **Relay Compiler**: Build-time compiler that generates optimized code
2. **Relay Runtime**: Client-side runtime for executing queries and managing cache
3. **React Integration**: Hooks and components for declarative data fetching
4. **GraphQL Schema Requirements**: Enforces specific schema conventions

**Core Philosophy:**

- **Data Masking**: Components only access data they explicitly declare
- **Colocation**: Queries live alongside components that use them
- **Static Analysis**: Compiler optimizes queries at build time
- **Normalized Cache**: Automatic cache updates with minimal configuration
- **Performance First**: Designed for massive scale

---

#### **Relay vs Apollo Client**

| Feature | Relay | Apollo Client |
|---------|-------|---------------|
| **Developer** | Meta (Facebook) | Apollo GraphQL |
| **Philosophy** | Performance & scale | Ease of use & flexibility |
| **Learning Curve** | ⚠️ Steep | ✅ Gentle |
| **Setup Complexity** | Complex (compiler required) | Simple |
| **Schema Requirements** | Strict conventions (Node interface, connections) | Flexible |
| **Data Masking** | ✅ Enforced (components can't access parent's data) | ❌ Optional |
| **Bundle Size** | ~40KB | ~33KB |
| **Caching** | ✅ Automatic, highly optimized | ✅ Good, manual updates sometimes needed |
| **Type Safety** | ✅ Generated at build time | Via codegen (optional) |
| **Pagination** | ✅ Built-in, automatic | Manual implementation |
| **Suspense Support** | ✅ First-class | Limited |
| **Server Streaming** | ✅ @defer, @stream | Experimental |
| **Community** | Smaller, Meta-focused | Large, broad ecosystem |
| **Best For** | Large-scale apps, performance-critical | Most React apps, rapid development |

---

#### **Installation and Setup**

**1. Install Dependencies:**

```bash
npm install react react-dom
npm install relay-runtime react-relay
npm install --save-dev relay-compiler @types/react-relay babel-plugin-relay
```

**2. Configure Relay Compiler:**

```json
// package.json
{
  "scripts": {
    "relay": "relay-compiler",
    "relay:watch": "relay-compiler --watch"
  },
  "relay": {
    "src": "./src",
    "schema": "./schema.graphql",
    "language": "typescript",
    "exclude": ["**/node_modules/**", "**/__generated__/**"]
  }
}
```

**3. Download GraphQL Schema:**

```bash
# Download schema from server
npx get-graphql-schema http://localhost:4000/graphql > schema.graphql

# Or use introspection query
curl -X POST http://localhost:4000/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { types { name } } }"}' \
  > schema.json
```

**4. Configure Babel:**

```json
// babel.config.json
{
  "plugins": [
    "relay"
  ]
}
```

**5. Create Relay Environment:**

```typescript
// src/RelayEnvironment.ts
import {
  Environment,
  Network,
  RecordSource,
  Store,
  FetchFunction,
} from 'relay-runtime';

// Define how to fetch GraphQL queries
const fetchQuery: FetchFunction = async (operation, variables) => {
  const response = await fetch('http://localhost:4000/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${localStorage.getItem('token')}`,
    },
    body: JSON.stringify({
      query: operation.text,
      variables,
    }),
  });

  return await response.json();
};

// Create Relay environment
export const RelayEnvironment = new Environment({
  network: Network.create(fetchQuery),
  store: new Store(new RecordSource()),
});
```

**6. Wrap App with RelayEnvironmentProvider:**

```typescript
// src/App.tsx
import { RelayEnvironmentProvider } from 'react-relay';
import { RelayEnvironment } from './RelayEnvironment';

function App() {
  return (
    <RelayEnvironmentProvider environment={RelayEnvironment}>
      <YourComponents />
    </RelayEnvironmentProvider>
  );
}

export default App;
```

---

#### **GraphQL Schema Requirements**

Relay enforces specific schema conventions for optimal performance:

**1. Node Interface (for refetching objects by ID):**

```graphql
# Required: Global object identification
interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
  email: String!
}

type Post implements Node {
  id: ID!
  title: String!
  content: String!
}

type Query {
  # Required: Fetch any object by global ID
  node(id: ID!): Node
}
```

**2. Connection Pattern (for pagination):**

```graphql
# Connection for paginating lists
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Query {
  users(
    first: Int
    after: String
    last: Int
    before: String
  ): UserConnection!
}
```

**3. Mutations with Input Objects:**

```graphql
input CreateUserInput {
  name: String!
  email: String!
}

type CreateUserPayload {
  user: User!
  userEdge: UserEdge!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}
```

---

#### **Core API: Queries**

**Using `useLazyLoadQuery`:**

```typescript
// src/components/UserProfile.tsx
import { graphql, useLazyLoadQuery } from 'react-relay';

// Define query with graphql`` template literal
const UserProfileQuery = graphql`
  query UserProfileQuery($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      avatar
      ...UserPosts_user
    }
  }
`;

interface UserProfileProps {
  userId: string;
}

function UserProfile({ userId }: UserProfileProps) {
  // Execute query - Suspense-compatible
  const data = useLazyLoadQuery(UserProfileQuery, { userId });

  return (
    <div>
      <img src={data.user.avatar} alt={data.user.name} />
      <h1>{data.user.name}</h1>
      <p>{data.user.email}</p>
      
      {/* Fragment component */}
      <UserPosts user={data.user} />
    </div>
  );
}

// Wrap with Suspense
function UserProfileContainer({ userId }: UserProfileProps) {
  return (
    <Suspense fallback={<div>Loading user...</div>}>
      <UserProfile userId={userId} />
    </Suspense>
  );
}
```

**Run Relay Compiler:**

```bash
npm run relay
```

This generates:

```typescript
// src/components/__generated__/UserProfileQuery.graphql.ts
export type UserProfileQuery$variables = {
  userId: string;
};

export type UserProfileQuery$data = {
  readonly user: {
    readonly id: string;
    readonly name: string;
    readonly email: string;
    readonly avatar: string;
  };
};
```

---

#### **Fragments (Data Masking)**

**Key Concept**: Each component declares its own data requirements via **fragments**. Components can only access data from their own fragments, not parent's data.

```typescript
// src/components/UserPosts.tsx
import { graphql, useFragment } from 'react-relay';
import type { UserPosts_user$key } from './__generated__/UserPosts_user.graphql';

// Fragment definition - declares what data this component needs
const UserPostsFragment = graphql`
  fragment UserPosts_user on User {
    id
    posts(first: 10) {
      edges {
        node {
          id
          title
          createdAt
          ...PostItem_post
        }
      }
    }
  }
`;

interface UserPostsProps {
  user: UserPosts_user$key; // Fragment reference, not actual data
}

function UserPosts({ user }: UserPostsProps) {
  // Read fragment data
  const data = useFragment(UserPostsFragment, user);

  // Can access posts because it's in the fragment
  return (
    <div>
      <h2>Posts</h2>
      {data.posts.edges.map(({ node }) => (
        <PostItem key={node.id} post={node} />
      ))}
    </div>
  );
}
```

```typescript
// src/components/PostItem.tsx
import { graphql, useFragment } from 'react-relay';
import type { PostItem_post$key } from './__generated__/PostItem_post.graphql';

const PostItemFragment = graphql`
  fragment PostItem_post on Post {
    id
    title
    createdAt
  }
`;

interface PostItemProps {
  post: PostItem_post$key;
}

function PostItem({ post }: PostItemProps) {
  const data = useFragment(PostItemFragment, post);

  return (
    <div>
      <h3>{data.title}</h3>
      <time>{new Date(data.createdAt).toLocaleDateString()}</time>
    </div>
  );
}
```

**Benefits of Data Masking:**

1. **Encapsulation**: Components can't accidentally depend on parent's data
2. **Refactoring**: Change component's data needs without breaking parent
3. **Performance**: Relay knows exactly which components need updates
4. **Type Safety**: TypeScript enforces correct data access

---

#### **Mutations**

```typescript
// src/mutations/CreatePostMutation.ts
import { graphql, commitMutation } from 'react-relay';
import { RelayEnvironment } from '../RelayEnvironment';

const mutation = graphql`
  mutation CreatePostMutation($input: CreatePostInput!) {
    createPost(input: $input) {
      post {
        id
        title
        content
      }
      postEdge {
        node {
          id
          title
        }
      }
    }
  }
`;

interface CreatePostInput {
  title: string;
  content: string;
}

export function createPost(input: CreatePostInput) {
  return new Promise((resolve, reject) => {
    commitMutation(RelayEnvironment, {
      mutation,
      variables: { input },
      
      // Optimistic response
      optimisticResponse: {
        createPost: {
          post: {
            id: 'temp-id-' + Date.now(),
            title: input.title,
            content: input.content,
          },
          postEdge: {
            node: {
              id: 'temp-id-' + Date.now(),
              title: input.title,
            },
          },
        },
      },
      
      // Update store after mutation
      updater: (store) => {
        const payload = store.getRootField('createPost');
        const newPost = payload?.getLinkedRecord('postEdge');
        
        if (newPost) {
          const root = store.getRoot();
          const connection = root.getLinkedRecord('posts');
          
          if (connection) {
            // Add to connection
            const edges = connection.getLinkedRecords('edges') || [];
            connection.setLinkedRecords([newPost, ...edges], 'edges');
          }
        }
      },
      
      onCompleted: (response, errors) => {
        if (errors) {
          reject(errors);
        } else {
          resolve(response);
        }
      },
      
      onError: (error) => {
        reject(error);
      },
    });
  });
}
```

**Using Mutation in Component:**

```typescript
// src/components/CreatePostForm.tsx
import { useMutation } from 'react-relay';
import { graphql } from 'relay-runtime';

const CreatePostMutation = graphql`
  mutation CreatePostFormMutation($input: CreatePostInput!) {
    createPost(input: $input) {
      post {
        id
        title
        content
      }
    }
  }
`;

function CreatePostForm() {
  const [title, setTitle] = React.useState('');
  const [content, setContent] = React.useState('');
  
  const [commitMutation, isMutationInFlight] = useMutation(CreatePostMutation);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    commitMutation({
      variables: {
        input: { title, content },
      },
      onCompleted: (response) => {
        console.log('Post created:', response.createPost.post);
        setTitle('');
        setContent('');
      },
      onError: (error) => {
        console.error('Failed to create post:', error);
      },
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Content"
      />
      <button type="submit" disabled={isMutationInFlight}>
        {isMutationInFlight ? 'Creating...' : 'Create Post'}
      </button>
    </form>
  );
}
```

---

#### **Pagination with Connections**

Relay has **built-in pagination support** via `usePaginationFragment`:

```typescript
// src/components/PostList.tsx
import { graphql, usePaginationFragment } from 'react-relay';
import type { PostList_query$key } from './__generated__/PostList_query.graphql';

const PostListFragment = graphql`
  fragment PostList_query on Query 
  @refetchable(queryName: "PostListPaginationQuery")
  @argumentDefinitions(
    first: { type: "Int", defaultValue: 10 }
    after: { type: "String" }
  ) {
    posts(first: $first, after: $after) 
    @connection(key: "PostList_posts") {
      edges {
        node {
          id
          title
          content
          ...PostItem_post
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

interface PostListProps {
  query: PostList_query$key;
}

function PostList({ query }: PostListProps) {
  const {
    data,
    loadNext,
    hasNext,
    isLoadingNext,
    refetch,
  } = usePaginationFragment(PostListFragment, query);

  const handleLoadMore = () => {
    if (hasNext && !isLoadingNext) {
      loadNext(10); // Load 10 more items
    }
  };

  return (
    <div>
      <button onClick={() => refetch({})}>Refresh</button>
      
      {data.posts.edges.map(({ node }) => (
        <PostItem key={node.id} post={node} />
      ))}
      
      {hasNext && (
        <button onClick={handleLoadMore} disabled={isLoadingNext}>
          {isLoadingNext ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

**Parent Query:**

```typescript
const PostListPageQuery = graphql`
  query PostListPageQuery {
    ...PostList_query
  }
`;

function PostListPage() {
  const data = useLazyLoadQuery(PostListPageQuery, {});

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <PostList query={data} />
    </Suspense>
  );
}
```

---

#### **Subscriptions (Real-Time)**

```typescript
// src/subscriptions/NewPostSubscription.ts
import { graphql, requestSubscription } from 'react-relay';
import { RelayEnvironment } from '../RelayEnvironment';

const subscription = graphql`
  subscription NewPostSubscription {
    newPost {
      postEdge {
        node {
          id
          title
          content
          author {
            id
            name
          }
        }
      }
    }
  }
`;

export function subscribeToNewPosts() {
  return requestSubscription(RelayEnvironment, {
    subscription,
    variables: {},
    
    updater: (store) => {
      const payload = store.getRootField('newPost');
      const newPostEdge = payload?.getLinkedRecord('postEdge');
      
      if (newPostEdge) {
        const root = store.getRoot();
        const posts = root.getLinkedRecord('posts');
        
        if (posts) {
          const edges = posts.getLinkedRecords('edges') || [];
          posts.setLinkedRecords([newPostEdge, ...edges], 'edges');
        }
      }
    },
    
    onNext: (response) => {
      console.log('New post received:', response);
    },
    
    onError: (error) => {
      console.error('Subscription error:', error);
    },
  });
}
```

**Use in Component:**

```typescript
function PostFeed() {
  React.useEffect(() => {
    const subscription = subscribeToNewPosts();
    
    return () => {
      subscription.dispose();
    };
  }, []);

  // ... rest of component
}
```

---

#### **Advanced Features**

**1. Defer and Stream (@defer, @stream):**

```typescript
const UserProfileQuery = graphql`
  query UserProfileQuery($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      
      # Defer loading posts
      ... @defer {
        posts(first: 10) {
          edges {
            node {
              id
              title
            }
          }
        }
      }
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const data = useLazyLoadQuery(UserProfileQuery, { userId });

  return (
    <div>
      <h1>{data.user.name}</h1>
      
      {/* Posts load separately */}
      <Suspense fallback={<div>Loading posts...</div>}>
        {data.user.posts && (
          <div>
            {data.user.posts.edges.map(({ node }) => (
              <div key={node.id}>{node.title}</div>
            ))}
          </div>
        )}
      </Suspense>
    </div>
  );
}
```

**2. Preloading Queries:**

```typescript
import { useQueryLoader, usePreloadedQuery } from 'react-relay';

const UserQuery = graphql`
  query UserPreloadQuery($userId: ID!) {
    user(id: $userId) {
      id
      name
    }
  }
`;

function UserProfileButton({ userId }: { userId: string }) {
  const [queryRef, loadQuery] = useQueryLoader(UserQuery);

  return (
    <>
      <button 
        onMouseEnter={() => loadQuery({ userId })} // Preload on hover
        onClick={() => navigateToProfile(userId)}
      >
        View Profile
      </button>
      
      {queryRef && (
        <Suspense fallback={null}>
          <UserProfileTooltip queryRef={queryRef} />
        </Suspense>
      )}
    </>
  );
}

function UserProfileTooltip({ queryRef }) {
  const data = usePreloadedQuery(UserQuery, queryRef);
  return <div>{data.user.name}</div>;
}
```

**3. Refetching Queries:**

```typescript
import { useRefetchableFragment } from 'react-relay';

const UserFragment = graphql`
  fragment UserRefetch_user on User 
  @refetchable(queryName: "UserRefetchQuery") {
    id
    name
    email
  }
`;

function UserComponent({ user }) {
  const [data, refetch] = useRefetchableFragment(UserFragment, user);

  const handleRefresh = () => {
    refetch({}, { fetchPolicy: 'network-only' });
  };

  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={handleRefresh}>Refresh</button>
    </div>
  );
}
```

**4. Local State Management:**

```typescript
// Define client-side schema extension
const localSchema = `
  extend type User {
    isFollowing: Boolean!
  }
`;

// Update local field
commitLocalUpdate(RelayEnvironment, (store) => {
  const user = store.get('User:123');
  if (user) {
    user.setValue(true, 'isFollowing');
  }
});

// Query local field
const UserFragment = graphql`
  fragment UserFollowButton_user on User {
    id
    isFollowing @client
  }
`;
```

---

#### **Real-World Example: Social Feed**

```typescript
// src/components/SocialFeed.tsx
import { graphql, useLazyLoadQuery, usePaginationFragment } from 'react-relay';

// Main query
const SocialFeedQuery = graphql`
  query SocialFeedQuery {
    viewer {
      id
      name
    }
    ...SocialFeed_query
  }
`;

// Pagination fragment
const SocialFeedFragment = graphql`
  fragment SocialFeed_query on Query
  @refetchable(queryName: "SocialFeedPaginationQuery")
  @argumentDefinitions(
    first: { type: "Int", defaultValue: 20 }
    after: { type: "String" }
  ) {
    feed(first: $first, after: $after)
    @connection(key: "SocialFeed_feed") {
      edges {
        node {
          id
          ...FeedPost_post
        }
      }
    }
  }
`;

function SocialFeed() {
  const queryData = useLazyLoadQuery(SocialFeedQuery, {});
  
  const {
    data,
    loadNext,
    hasNext,
    isLoadingNext,
    refetch,
  } = usePaginationFragment(SocialFeedFragment, queryData);

  // Infinite scroll
  React.useEffect(() => {
    const handleScroll = () => {
      if (
        window.innerHeight + window.scrollY >= document.body.offsetHeight - 500 &&
        hasNext &&
        !isLoadingNext
      ) {
        loadNext(20);
      }
    };

    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [hasNext, isLoadingNext, loadNext]);

  return (
    <div className="feed">
      <h1>Welcome, {queryData.viewer.name}</h1>
      
      <button onClick={() => refetch({})}>Refresh Feed</button>
      
      {data.feed.edges.map(({ node }) => (
        <FeedPost key={node.id} post={node} />
      ))}
      
      {isLoadingNext && <div>Loading more posts...</div>}
    </div>
  );
}

// Post component with fragment
const FeedPostFragment = graphql`
  fragment FeedPost_post on Post {
    id
    title
    content
    likes
    createdAt
    author {
      id
      name
      avatar
    }
    comments(first: 3) {
      edges {
        node {
          id
          text
        }
      }
    }
  }
`;

function FeedPost({ post }) {
  const data = useFragment(FeedPostFragment, post);
  
  const [likeMutation] = useMutation(graphql`
    mutation FeedPostLikeMutation($postId: ID!) {
      likePost(postId: $postId) {
        id
        likes
      }
    }
  `);

  const handleLike = () => {
    likeMutation({
      variables: { postId: data.id },
      optimisticResponse: {
        likePost: {
          id: data.id,
          likes: data.likes + 1,
        },
      },
    });
  };

  return (
    <article className="post">
      <div className="author">
        <img src={data.author.avatar} alt={data.author.name} />
        <span>{data.author.name}</span>
      </div>
      
      <h2>{data.title}</h2>
      <p>{data.content}</p>
      
      <button onClick={handleLike}>❤️ {data.likes}</button>
      
      <div className="comments">
        {data.comments.edges.map(({ node }) => (
          <p key={node.id}>{node.text}</p>
        ))}
      </div>
    </article>
  );
}

// Wrap in Suspense
function SocialFeedPage() {
  return (
    <Suspense fallback={<div>Loading feed...</div>}>
      <SocialFeed />
    </Suspense>
  );
}
```

---

#### **Performance Benefits**

**1. Static Queries (Compile-Time Optimization):**

```typescript
// Relay compiler analyzes queries at build time
// Generates optimized code with:
// - Minimal runtime overhead
// - Tree-shaking of unused fields
// - Type safety without runtime cost
```

**2. Data Masking (Prevents Over-Rendering):**

```typescript
// Only components with affected fragments re-render
// Parent doesn't re-render when child's data changes
```

**3. Automatic Garbage Collection:**

```typescript
// Relay automatically removes unused data from cache
// Configurable retention policies
```

**4. Request Deduplication:**

```typescript
// Identical queries are automatically batched
// Only one network request sent
```

**5. Optimized Cache Normalization:**

```typescript
// Objects stored once by ID
// Automatic updates across all components
```

---

#### **When to Use Relay**

**✅ Use Relay When:**

- Building large-scale applications (Facebook, Instagram scale)
- Performance is critical (millions of users)
- Team can invest in learning curve
- Want strict data isolation between components
- Need automatic pagination and caching
- GraphQL server can follow Relay conventions
- Using React Suspense and Concurrent Mode
- Want compile-time guarantees and optimizations

**❌ Avoid Relay When:**

- Small to medium apps (Apollo is easier)
- Rapid prototyping (setup takes time)
- Can't modify GraphQL schema to follow Relay conventions
- Team unfamiliar with GraphQL/React
- Need maximum flexibility over performance
- Legacy GraphQL API without Node interface/connections

---

#### **Summary**

**Relay provides:**

1. **Performance**: Optimized for massive scale (Facebook/Instagram use it)
2. **Data Masking**: Components isolated, can only access declared data
3. **Colocation**: Queries live with components
4. **Compiler**: Build-time optimization and type generation
5. **Automatic Pagination**: Built-in connection handling
6. **Suspense Integration**: First-class support for React Suspense
7. **Normalized Cache**: Automatic, intelligent caching
8. **Type Safety**: Generated TypeScript types
9. **@defer/@stream**: Server streaming for progressive loading
10. **Schema Conventions**: Requires Node interface and connection pattern

**Key Differences from Apollo:**

| Aspect | Relay | Apollo |
|--------|-------|--------|
| Ease of Use | ⚠️ Complex | ✅ Simple |
| Performance | ✅✅ Excellent | ✅ Good |
| Flexibility | ⚠️ Strict | ✅ Flexible |
| Schema Requirements | Required conventions | No requirements |
| Data Masking | Enforced | Optional |
| Pagination | Automatic | Manual |

**Workflow:**

1. **Install**: `npm install react-relay relay-runtime relay-compiler`
2. **Configure**: Set up `relay-compiler` and download schema
3. **Define**: Write queries/fragments with `graphql``
4. **Compile**: Run `relay-compiler` to generate types
5. **Use**: Call hooks (`useLazyLoadQuery`, `useFragment`, `useMutation`)
6. **Optimize**: Leverage @defer, preloading, data masking

Relay is the **most performant GraphQL client** for React, designed for applications that need to scale to billions of users. It trades ease of use for maximum performance and developer discipline.

</details>

---

### 224. How do you implement GraphQL subscriptions in React?

<details>
<summary>View Answer</summary>

**GraphQL subscriptions** enable **real-time, bidirectional communication** between client and server, allowing the server to push updates to clients when data changes. They are built on **WebSockets** and are perfect for features like live chat, notifications, real-time dashboards, collaborative editing, and live feeds.

Subscriptions complement queries (pull data once) and mutations (send data) by providing a **push-based data flow**.

---

#### **What are GraphQL Subscriptions?**

**GraphQL Operations:**

| Operation | Purpose | Direction | Protocol |
|-----------|---------|-----------|----------|
| **Query** | Fetch data once | Client → Server | HTTP/HTTPS |
| **Mutation** | Modify data | Client → Server | HTTP/HTTPS |
| **Subscription** | ✅ Real-time updates | Server → Client | WebSocket |

**Subscription Definition (Server Schema):**

```graphql
type Subscription {
  # Subscribe to new messages in a chat
  newMessage(chatId: ID!): Message!
  
  # Subscribe to user status changes
  userStatusChanged(userId: ID!): User!
  
  # Subscribe to post likes
  postLiked(postId: ID!): Post!
  
  # Subscribe to new notifications
  newNotification: Notification!
  
  # Subscribe to live scores
  scoreUpdated(matchId: ID!): Match!
}

type Message {
  id: ID!
  text: String!
  author: User!
  chatId: ID!
  createdAt: DateTime!
}

type User {
  id: ID!
  name: String!
  status: UserStatus!
}

enum UserStatus {
  ONLINE
  OFFLINE
  AWAY
}
```

---

#### **How Subscriptions Work**

**Flow:**

```
1. Client opens WebSocket connection to GraphQL server
2. Client sends subscription query over WebSocket
3. Server registers subscription and keeps connection open
4. When data changes, server pushes update to client
5. Client receives update and updates UI
6. Connection stays open for continuous updates
7. Client closes subscription when component unmounts
```

**Protocol:**

Most implementations use **`graphql-ws`** protocol (formerly `subscriptions-transport-ws`):

```
Client                              Server
  |                                    |
  |------- WebSocket Handshake ------>|
  |<------ Connection Established -----|
  |                                    |
  |------- Subscribe Message --------->|
  |        { query, variables }        |
  |                                    |
  |<------- Next (Data Update) --------|
  |<------- Next (Data Update) --------|
  |<------- Next (Data Update) --------|
  |                                    |
  |------- Unsubscribe Message ------->|
  |                                    |
  |------- Close Connection ---------->|
```

---

#### **Implementation: Apollo Client**

**1. Installation:**

```bash
npm install @apollo/client graphql graphql-ws
```

**2. Configure WebSocket Link:**

```typescript
// src/apolloClient.ts
import {
  ApolloClient,
  InMemoryCache,
  HttpLink,
  split,
} from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';

// HTTP link for queries and mutations
const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql',
  credentials: 'include',
});

// WebSocket link for subscriptions
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: {
      // Send auth token with WebSocket connection
      authToken: localStorage.getItem('token'),
    },
    // Reconnect on connection loss
    shouldRetry: () => true,
    retryAttempts: 5,
    retryWait: async (retries) => {
      await new Promise(resolve => setTimeout(resolve, retries * 1000));
    },
    // Connection lifecycle hooks
    on: {
      connected: () => console.log('WebSocket connected'),
      closed: () => console.log('WebSocket closed'),
      error: (error) => console.error('WebSocket error:', error),
    },
  })
);

// Split based on operation type
// Use WebSocket for subscriptions, HTTP for queries/mutations
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,    // Use WebSocket for subscriptions
  httpLink   // Use HTTP for queries/mutations
);

export const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache(),
});
```

**3. Define Subscription:**

```typescript
// src/graphql/subscriptions.ts
import { gql } from '@apollo/client';

export const NEW_MESSAGE_SUBSCRIPTION = gql`
  subscription OnNewMessage($chatId: ID!) {
    newMessage(chatId: $chatId) {
      id
      text
      author {
        id
        name
        avatar
      }
      chatId
      createdAt
    }
  }
`;

export const USER_STATUS_SUBSCRIPTION = gql`
  subscription OnUserStatusChanged($userId: ID!) {
    userStatusChanged(userId: $userId) {
      id
      name
      status
    }
  }
`;

export const NEW_NOTIFICATION_SUBSCRIPTION = gql`
  subscription OnNewNotification {
    newNotification {
      id
      title
      message
      type
      createdAt
      read
    }
  }
`;
```

**4. Use `useSubscription` Hook:**

```typescript
// src/components/ChatRoom.tsx
import { useSubscription, useQuery, gql } from '@apollo/client';
import { NEW_MESSAGE_SUBSCRIPTION } from '../graphql/subscriptions';

const GET_MESSAGES = gql`
  query GetMessages($chatId: ID!) {
    messages(chatId: $chatId) {
      id
      text
      author {
        id
        name
        avatar
      }
      createdAt
    }
  }
`;

interface Message {
  id: string;
  text: string;
  author: {
    id: string;
    name: string;
    avatar: string;
  };
  createdAt: string;
}

function ChatRoom({ chatId }: { chatId: string }) {
  // Fetch existing messages
  const { data: messagesData, loading: loadingMessages } = useQuery(
    GET_MESSAGES,
    {
      variables: { chatId },
    }
  );

  // Subscribe to new messages
  const { data: subscriptionData, loading: subscriptionLoading } = useSubscription(
    NEW_MESSAGE_SUBSCRIPTION,
    {
      variables: { chatId },
      
      // Optional: handle subscription data
      onSubscriptionData: ({ client, subscriptionData }) => {
        const newMessage = subscriptionData.data?.newMessage;
        if (newMessage) {
          console.log('New message received:', newMessage);
          
          // Show notification
          if (Notification.permission === 'granted') {
            new Notification(newMessage.author.name, {
              body: newMessage.text,
            });
          }
        }
      },
      
      // Handle errors
      onError: (error) => {
        console.error('Subscription error:', error);
      },
      
      // Skip subscription if no chatId
      skip: !chatId,
      
      // Reconnect behavior
      shouldResubscribe: true,
    }
  );

  const messages = messagesData?.messages || [];

  if (loadingMessages) return <div>Loading chat...</div>;

  return (
    <div className="chat-room">
      <div className="messages">
        {messages.map((message: Message) => (
          <div key={message.id} className="message">
            <img src={message.author.avatar} alt={message.author.name} />
            <div>
              <strong>{message.author.name}</strong>
              <p>{message.text}</p>
              <time>{new Date(message.createdAt).toLocaleTimeString()}</time>
            </div>
          </div>
        ))}
        
        {/* Show new message immediately from subscription */}
        {subscriptionData?.newMessage && (
          <div className="message new">
            <img 
              src={subscriptionData.newMessage.author.avatar} 
              alt={subscriptionData.newMessage.author.name} 
            />
            <div>
              <strong>{subscriptionData.newMessage.author.name}</strong>
              <p>{subscriptionData.newMessage.text}</p>
              <time>Just now</time>
            </div>
          </div>
        )}
      </div>
      
      {subscriptionLoading && <div>Connecting to live chat...</div>}
    </div>
  );
}
```

---

#### **Manual Subscription with Cache Updates**

For more control, subscribe manually and update the Apollo cache:

```typescript
// src/components/ChatRoomAdvanced.tsx
import { useQuery, gql, useApolloClient } from '@apollo/client';
import { useEffect } from 'react';
import { NEW_MESSAGE_SUBSCRIPTION } from '../graphql/subscriptions';

const GET_MESSAGES = gql`
  query GetMessages($chatId: ID!) {
    messages(chatId: $chatId) {
      id
      text
      author {
        id
        name
        avatar
      }
      createdAt
    }
  }
`;

function ChatRoomAdvanced({ chatId }: { chatId: string }) {
  const client = useApolloClient();
  
  const { data, loading } = useQuery(GET_MESSAGES, {
    variables: { chatId },
  });

  useEffect(() => {
    // Subscribe manually
    const subscription = client
      .subscribe({
        query: NEW_MESSAGE_SUBSCRIPTION,
        variables: { chatId },
      })
      .subscribe({
        next: ({ data }) => {
          const newMessage = data?.newMessage;
          if (!newMessage) return;

          // Update Apollo cache manually
          client.cache.updateQuery(
            {
              query: GET_MESSAGES,
              variables: { chatId },
            },
            (existingData) => {
              if (!existingData) return existingData;

              return {
                messages: [...existingData.messages, newMessage],
              };
            }
          );
        },
        error: (error) => {
          console.error('Subscription error:', error);
        },
      });

    // Cleanup: unsubscribe when component unmounts
    return () => {
      subscription.unsubscribe();
    };
  }, [chatId, client]);

  if (loading) return <div>Loading...</div>;

  return (
    <div className="chat-room">
      {data?.messages.map((message) => (
        <div key={message.id}>{message.text}</div>
      ))}
    </div>
  );
}
```

---

#### **Update Cache with `subscribeToMore`**

Apollo's `subscribeToMore` is the recommended way to update cache from subscriptions:

```typescript
// src/components/ChatRoomSubscribeToMore.tsx
import { useQuery, gql } from '@apollo/client';
import { useEffect } from 'react';
import { NEW_MESSAGE_SUBSCRIPTION } from '../graphql/subscriptions';

const GET_MESSAGES = gql`
  query GetMessages($chatId: ID!) {
    messages(chatId: $chatId) {
      id
      text
      author {
        id
        name
        avatar
      }
      createdAt
    }
  }
`;

function ChatRoomSubscribeToMore({ chatId }: { chatId: string }) {
  const { data, loading, subscribeToMore } = useQuery(GET_MESSAGES, {
    variables: { chatId },
  });

  useEffect(() => {
    // Subscribe to new messages and update cache automatically
    const unsubscribe = subscribeToMore({
      document: NEW_MESSAGE_SUBSCRIPTION,
      variables: { chatId },
      
      // Update cache when new data arrives
      updateQuery: (prev, { subscriptionData }) => {
        if (!subscriptionData.data) return prev;
        
        const newMessage = subscriptionData.data.newMessage;

        // Check if message already exists (prevent duplicates)
        const messageExists = prev.messages.some(
          (msg) => msg.id === newMessage.id
        );
        
        if (messageExists) return prev;

        // Add new message to the list
        return {
          messages: [...prev.messages, newMessage],
        };
      },
      
      // Handle errors
      onError: (error) => {
        console.error('Subscription error:', error);
      },
    });

    // Cleanup
    return () => {
      unsubscribe();
    };
  }, [chatId, subscribeToMore]);

  if (loading) return <div>Loading messages...</div>;

  return (
    <div className="chat-room">
      {data?.messages.map((message) => (
        <div key={message.id} className="message">
          <strong>{message.author.name}:</strong> {message.text}
        </div>
      ))}
    </div>
  );
}
```

---

#### **Real-World Example: Live Notifications**

```typescript
// src/components/NotificationBell.tsx
import { useSubscription, useQuery, gql } from '@apollo/client';
import { useState } from 'react';

const GET_NOTIFICATIONS = gql`
  query GetNotifications {
    notifications {
      id
      title
      message
      type
      read
      createdAt
    }
  }
`;

const NEW_NOTIFICATION_SUBSCRIPTION = gql`
  subscription OnNewNotification {
    newNotification {
      id
      title
      message
      type
      read
      createdAt
    }
  }
`;

interface Notification {
  id: string;
  title: string;
  message: string;
  type: 'INFO' | 'WARNING' | 'ERROR' | 'SUCCESS';
  read: boolean;
  createdAt: string;
}

function NotificationBell() {
  const [isOpen, setIsOpen] = useState(false);

  // Fetch existing notifications
  const { data: notificationsData } = useQuery(GET_NOTIFICATIONS);

  // Subscribe to new notifications
  useSubscription(NEW_NOTIFICATION_SUBSCRIPTION, {
    onSubscriptionData: ({ client, subscriptionData }) => {
      const newNotification = subscriptionData.data?.newNotification;
      if (!newNotification) return;

      // Update cache
      client.cache.updateQuery(
        { query: GET_NOTIFICATIONS },
        (existingData) => {
          if (!existingData) return existingData;

          return {
            notifications: [newNotification, ...existingData.notifications],
          };
        }
      );

      // Show browser notification
      if (Notification.permission === 'granted') {
        new Notification(newNotification.title, {
          body: newNotification.message,
          icon: '/notification-icon.png',
        });
      }

      // Play sound
      const audio = new Audio('/notification-sound.mp3');
      audio.play();
    },
  });

  const notifications = notificationsData?.notifications || [];
  const unreadCount = notifications.filter((n: Notification) => !n.read).length;

  return (
    <div className="notification-bell">
      <button onClick={() => setIsOpen(!isOpen)}>
        🔔
        {unreadCount > 0 && (
          <span className="badge">{unreadCount}</span>
        )}
      </button>

      {isOpen && (
        <div className="notification-dropdown">
          <h3>Notifications</h3>
          {notifications.length === 0 ? (
            <p>No notifications</p>
          ) : (
            notifications.map((notification: Notification) => (
              <div
                key={notification.id}
                className={`notification ${notification.read ? 'read' : 'unread'}`}
              >
                <strong>{notification.title}</strong>
                <p>{notification.message}</p>
                <time>{new Date(notification.createdAt).toLocaleString()}</time>
              </div>
            ))
          )}
        </div>
      )}
    </div>
  );
}
```

---

#### **Real-World Example: Live Dashboard**

```typescript
// src/components/LiveDashboard.tsx
import { useSubscription, useQuery, gql } from '@apollo/client';

const GET_METRICS = gql`
  query GetMetrics {
    metrics {
      activeUsers
      revenue
      orders
      timestamp
    }
  }
`;

const METRICS_UPDATED_SUBSCRIPTION = gql`
  subscription OnMetricsUpdated {
    metricsUpdated {
      activeUsers
      revenue
      orders
      timestamp
    }
  }
`;

function LiveDashboard() {
  // Initial data
  const { data: initialData } = useQuery(GET_METRICS);

  // Real-time updates
  const { data: realtimeData } = useSubscription(METRICS_UPDATED_SUBSCRIPTION);

  // Use subscription data if available, otherwise use initial data
  const metrics = realtimeData?.metricsUpdated || initialData?.metrics;

  if (!metrics) return <div>Loading dashboard...</div>;

  return (
    <div className="dashboard">
      <h1>Live Dashboard</h1>
      
      <div className="metrics">
        <div className="metric">
          <h2>Active Users</h2>
          <p className="value">{metrics.activeUsers.toLocaleString()}</p>
        </div>
        
        <div className="metric">
          <h2>Revenue</h2>
          <p className="value">${metrics.revenue.toLocaleString()}</p>
        </div>
        
        <div className="metric">
          <h2>Orders</h2>
          <p className="value">{metrics.orders.toLocaleString()}</p>
        </div>
      </div>
      
      <p className="last-updated">
        Last updated: {new Date(metrics.timestamp).toLocaleTimeString()}
        <span className="live-indicator">🔴 LIVE</span>
      </p>
    </div>
  );
}
```

---

#### **Implementation: urql (Lightweight Alternative)**

**1. Installation:**

```bash
npm install urql graphql @urql/core wonka
npm install subscriptions-transport-ws
```

**2. Configure Client with Subscriptions:**

```typescript
// src/urqlClient.ts
import { createClient, defaultExchanges, subscriptionExchange } from 'urql';
import { SubscriptionClient } from 'subscriptions-transport-ws';

const subscriptionClient = new SubscriptionClient(
  'ws://localhost:4000/graphql',
  {
    reconnect: true,
    connectionParams: {
      authToken: localStorage.getItem('token'),
    },
  }
);

export const client = createClient({
  url: 'http://localhost:4000/graphql',
  exchanges: [
    ...defaultExchanges,
    subscriptionExchange({
      forwardSubscription: (operation) => subscriptionClient.request(operation),
    }),
  ],
});
```

**3. Use Subscription:**

```typescript
// src/components/ChatRoomUrql.tsx
import { useSubscription } from 'urql';

const NEW_MESSAGE_SUBSCRIPTION = `
  subscription ($chatId: ID!) {
    newMessage(chatId: $chatId) {
      id
      text
      author {
        name
      }
    }
  }
`;

function ChatRoomUrql({ chatId }: { chatId: string }) {
  const [result] = useSubscription({
    query: NEW_MESSAGE_SUBSCRIPTION,
    variables: { chatId },
  });

  const { data, fetching, error } = result;

  if (fetching) return <div>Connecting...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {data && (
        <div className="new-message">
          <strong>{data.newMessage.author.name}:</strong>
          {data.newMessage.text}
        </div>
      )}
    </div>
  );
}
```

---

#### **Implementation: Relay**

```typescript
// src/subscriptions/NewMessageSubscription.ts
import { requestSubscription, graphql } from 'react-relay';
import { RelayEnvironment } from '../RelayEnvironment';

const subscription = graphql`
  subscription NewMessageSubscription($chatId: ID!) {
    newMessage(chatId: $chatId) {
      id
      text
      author {
        id
        name
      }
      chatId
    }
  }
`;

export function subscribeToNewMessages(chatId: string) {
  return requestSubscription(RelayEnvironment, {
    subscription,
    variables: { chatId },
    
    // Update Relay store
    updater: (store) => {
      const payload = store.getRootField('newMessage');
      const newMessage = payload?.getLinkedRecord('message');
      
      if (newMessage) {
        const chatRecord = store.get(chatId);
        if (chatRecord) {
          const messages = chatRecord.getLinkedRecords('messages') || [];
          chatRecord.setLinkedRecords([...messages, newMessage], 'messages');
        }
      }
    },
    
    onNext: (response) => {
      console.log('New message:', response);
    },
    
    onError: (error) => {
      console.error('Subscription error:', error);
    },
  });
}
```

**Use in Component:**

```typescript
import { useEffect } from 'react';

function ChatRoom({ chatId }: { chatId: string }) {
  useEffect(() => {
    const subscription = subscribeToNewMessages(chatId);
    
    return () => {
      subscription.dispose();
    };
  }, [chatId]);

  // ... rest of component
}
```

---

#### **Advanced Patterns**

**1. Conditional Subscriptions:**

```typescript
function ChatRoom({ chatId, isActive }: { chatId: string; isActive: boolean }) {
  useSubscription(NEW_MESSAGE_SUBSCRIPTION, {
    variables: { chatId },
    skip: !isActive, // Only subscribe when chat is active
  });
}
```

**2. Multiple Subscriptions:**

```typescript
function Dashboard() {
  // Subscribe to multiple events
  useSubscription(USER_JOINED_SUBSCRIPTION);
  useSubscription(USER_LEFT_SUBSCRIPTION);
  useSubscription(MESSAGE_SENT_SUBSCRIPTION);
  useSubscription(METRICS_UPDATED_SUBSCRIPTION);
}
```

**3. Subscription with Authentication:**

```typescript
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: async () => {
      // Fetch fresh token
      const token = await getAuthToken();
      return {
        authToken: token,
        userId: getCurrentUserId(),
      };
    },
  })
);
```

**4. Reconnection Handling:**

```typescript
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    retryAttempts: 10,
    retryWait: async (retries) => {
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.min(1000 * 2 ** retries, 30000))
      );
    },
    on: {
      connected: () => {
        console.log('Connected');
        setConnectionStatus('connected');
      },
      closed: () => {
        console.log('Disconnected');
        setConnectionStatus('disconnected');
      },
      error: (error) => {
        console.error('WebSocket error:', error);
      },
    },
  })
);
```

**5. Lazy Subscriptions:**

```typescript
function ChatRoom({ chatId }: { chatId: string }) {
  const [isSubscribed, setIsSubscribed] = useState(false);

  const { data } = useSubscription(NEW_MESSAGE_SUBSCRIPTION, {
    variables: { chatId },
    skip: !isSubscribed,
  });

  return (
    <div>
      <button onClick={() => setIsSubscribed(true)}>
        Enable Live Updates
      </button>
      {data && <div>New message: {data.newMessage.text}</div>}
    </div>
  );
}
```

---

#### **Testing Subscriptions**

**Mock Subscription in Tests:**

```typescript
// src/components/__tests__/ChatRoom.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import ChatRoom from '../ChatRoom';
import { NEW_MESSAGE_SUBSCRIPTION } from '../../graphql/subscriptions';

const mocks = [
  {
    request: {
      query: NEW_MESSAGE_SUBSCRIPTION,
      variables: { chatId: '123' },
    },
    result: {
      data: {
        newMessage: {
          id: '456',
          text: 'Hello from test!',
          author: {
            id: '789',
            name: 'Test User',
            avatar: 'avatar.jpg',
          },
        },
      },
    },
  },
];

test('receives new messages via subscription', async () => {
  render(
    <MockedProvider mocks={mocks} addTypename={false}>
      <ChatRoom chatId="123" />
    </MockedProvider>
  );

  await waitFor(() => {
    expect(screen.getByText('Hello from test!')).toBeInTheDocument();
  });
});
```

---

#### **Performance Considerations**

**1. Limit Active Subscriptions:**

```typescript
// ❌ Bad: Subscribe to all users
useSubscription(USER_STATUS_SUBSCRIPTION); // Broadcasts to all clients

// ✅ Good: Subscribe only to visible users
useSubscription(USER_STATUS_SUBSCRIPTION, {
  variables: { userIds: visibleUserIds },
});
```

**2. Debounce High-Frequency Updates:**

```typescript
function LiveCounter() {
  const { data } = useSubscription(COUNTER_SUBSCRIPTION);
  const [displayValue, setDisplayValue] = useState(0);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDisplayValue(data?.counter || 0);
    }, 100); // Update UI max once per 100ms

    return () => clearTimeout(timer);
  }, [data]);

  return <div>{displayValue}</div>;
}
```

**3. Unsubscribe When Not Visible:**

```typescript
function ChatRoom({ chatId }: { chatId: string }) {
  const [isVisible, setIsVisible] = useState(true);

  useEffect(() => {
    const handleVisibilityChange = () => {
      setIsVisible(!document.hidden);
    };

    document.addEventListener('visibilitychange', handleVisibilityChange);
    return () => {
      document.removeEventListener('visibilitychange', handleVisibilityChange);
    };
  }, []);

  useSubscription(NEW_MESSAGE_SUBSCRIPTION, {
    variables: { chatId },
    skip: !isVisible, // Pause when tab is hidden
  });
}
```

---

#### **Common Use Cases**

| Use Case | Description | Example |
|----------|-------------|---------|
| **Live Chat** | Real-time messages | Slack, Discord, WhatsApp |
| **Notifications** | Push notifications to users | Facebook, Twitter notifications |
| **Dashboards** | Live metrics and analytics | Google Analytics, monitoring tools |
| **Collaborative Editing** | Multi-user document editing | Google Docs, Figma |
| **Live Feeds** | Social media feeds, news | Twitter feed, Instagram stories |
| **Gaming** | Real-time game state | Multiplayer games, live scores |
| **Stock Tickers** | Live price updates | Trading platforms |
| **Location Tracking** | Real-time location updates | Uber, food delivery apps |
| **Progress Updates** | Long-running task updates | File uploads, processing jobs |
| **Presence** | Online/offline status | User status indicators |

---

#### **Troubleshooting**

**1. Connection Issues:**

```typescript
// Check WebSocket connection status
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    on: {
      connected: () => console.log('✅ Connected'),
      closed: (event) => console.log('❌ Closed:', event),
      error: (error) => console.error('⚠️ Error:', error),
    },
  })
);
```

**2. CORS Issues:**

```javascript
// Server-side: Enable CORS for WebSocket
const server = new SubscriptionServer(
  { schema, execute, subscribe },
  { 
    server: httpServer,
    path: '/graphql',
  }
);

// Allow origin
app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true,
}));
```

**3. Authentication Failures:**

```typescript
// Ensure token is sent correctly
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: () => {
      const token = localStorage.getItem('token');
      if (!token) {
        console.warn('No auth token found');
      }
      return {
        authToken: token,
      };
    },
  })
);
```

---

#### **Summary**

**GraphQL subscriptions in React:**

1. **WebSocket-Based**: Use persistent WebSocket connection for bidirectional communication
2. **Real-Time Updates**: Server pushes data to client when changes occur
3. **Setup**: Configure WebSocket link (Apollo: `GraphQLWsLink`, urql: `subscriptionExchange`)
4. **Usage**: Use `useSubscription` hook or `subscribeToMore` for cache updates
5. **Cache Updates**: Manually update Apollo cache when subscription data arrives
6. **Cleanup**: Automatically unsubscribe when component unmounts
7. **Error Handling**: Handle connection errors and implement retry logic
8. **Performance**: Limit active subscriptions, debounce updates, unsubscribe when not needed
9. **Use Cases**: Live chat, notifications, dashboards, collaborative editing, real-time feeds

**Key Steps:**

1. **Install**: `npm install graphql-ws`
2. **Configure**: Set up WebSocket link with `GraphQLWsLink`
3. **Split**: Use `split` to route subscriptions to WebSocket, queries/mutations to HTTP
4. **Define**: Write subscription with `gql`
5. **Subscribe**: Use `useSubscription` hook in component
6. **Update**: Handle cache updates with `subscribeToMore` or manual cache modification
7. **Cleanup**: Subscription auto-unsubscribes on unmount

GraphQL subscriptions transform React apps from **pull-based** (polling) to **push-based** (real-time), enabling modern real-time features with WebSocket efficiency.

</details>

---

### 225. What is normalized caching in GraphQL clients?

<details>
<summary>View Answer</summary>

**Normalized caching** is a technique used by GraphQL clients (Apollo Client, Relay, urql) to store data in a **flat, normalized structure** where each object is stored once by its unique identifier. This eliminates data duplication, ensures consistency across the application, and enables automatic cache updates when data changes.

Without normalized caching, the same user object fetched in different queries would be stored multiple times. With normalized caching, the user is stored once and referenced everywhere it's needed.

---

#### **Problem: Non-Normalized Cache**

**Without normalization:**

```typescript
// Query 1: Get post with author
{
  post(id: "1") {
    id: "1"
    title: "GraphQL Guide"
    author: {
      id: "123"
      name: "John Doe"
      email: "john@example.com"
    }
  }
}

// Query 2: Get user profile
{
  user(id: "123") {
    id: "123"
    name: "John Doe"
    email: "john@example.com"
    avatar: "avatar.jpg"
  }
}

// Cache structure (non-normalized)
{
  'post({"id":"1"})': {
    id: "1",
    title: "GraphQL Guide",
    author: {
      id: "123",
      name: "John Doe",      // ❌ Duplicated
      email: "john@example.com"  // ❌ Duplicated
    }
  },
  'user({"id":"123"})': {
    id: "123",
    name: "John Doe",          // ❌ Duplicated
    email: "john@example.com",     // ❌ Duplicated
    avatar: "avatar.jpg"
  }
}
```

**Problems:**
1. ❌ **Data Duplication**: Same user stored multiple times
2. ❌ **Inconsistency**: Updating user in one place doesn't update other places
3. ❌ **Memory Waste**: More memory used for duplicate data
4. ❌ **Manual Updates**: Need to manually update all copies

---

#### **Solution: Normalized Cache**

**With normalization:**

```typescript
// Same queries as above

// Cache structure (normalized)
{
  'Post:1': {
    __typename: 'Post',
    id: '1',
    title: 'GraphQL Guide',
    author: { __ref: 'User:123' }  // ✅ Reference to User
  },
  'User:123': {
    __typename: 'User',
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'avatar.jpg'
  }
}
```

**Benefits:**
1. ✅ **No Duplication**: Each object stored once
2. ✅ **Automatic Consistency**: Update propagates everywhere
3. ✅ **Memory Efficient**: Less memory used
4. ✅ **Automatic Updates**: All components using the data update automatically

---

#### **How Normalized Caching Works**

**Process:**

```
1. Execute GraphQL query
2. Receive nested response
3. Normalize: Extract objects and generate IDs
4. Store: Save objects in flat structure
5. Reference: Replace nested objects with references
6. Denormalize: When reading, resolve references
```

**Example Flow:**

```typescript
// 1. Query
query GetPost {
  post(id: "1") {
    id
    title
    author {
      id
      name
      posts {
        id
        title
      }
    }
  }
}

// 2. Response (nested)
{
  data: {
    post: {
      __typename: "Post",
      id: "1",
      title: "GraphQL Guide",
      author: {
        __typename: "User",
        id: "123",
        name: "John Doe",
        posts: [
          { __typename: "Post", id: "1", title: "GraphQL Guide" },
          { __typename: "Post", id: "2", title: "React Hooks" }
        ]
      }
    }
  }
}

// 3. Normalize (flatten)
{
  'Post:1': {
    __typename: 'Post',
    id: '1',
    title: 'GraphQL Guide',
    author: { __ref: 'User:123' }
  },
  'Post:2': {
    __typename: 'Post',
    id: '2',
    title: 'React Hooks'
  },
  'User:123': {
    __typename: 'User',
    id: '123',
    name: 'John Doe',
    posts: [
      { __ref: 'Post:1' },
      { __ref: 'Post:2' }
    ]
  }
}

// 4. When reading: Denormalize (resolve references)
// Automatically reconstruct nested structure
```

---

#### **Cache Keys and Identification**

**Default Cache Key:**

Apollo Client generates cache keys using `__typename` and `id` (or `_id`):

```typescript
// Default: __typename:id
'User:123'
'Post:456'
'Comment:789'
```

**Custom Cache Keys:**

```typescript
// src/apolloClient.ts
import { InMemoryCache } from '@apollo/client';

const cache = new InMemoryCache({
  typePolicies: {
    // Custom cache key for Product (use SKU instead of id)
    Product: {
      keyFields: ['sku'],  // Cache key: Product:ABC123
    },
    
    // Composite cache key
    Review: {
      keyFields: ['productId', 'userId'],  // Cache key: Review:product123:user456
    },
    
    // Nested key fields
    Book: {
      keyFields: ['isbn', ['edition', 'version']],  // Book:123456:1:2
    },
    
    // No cache key (not normalized)
    SearchResult: {
      keyFields: false,  // Don't normalize, store in parent
    },
    
    // Custom key function
    User: {
      keyFields: (object, context) => {
        return `User:${object.email}`;  // Use email as key
      },
    },
  },
});
```

**Example:**

```typescript
// Query returns products with SKU
{
  products {
    sku       // "ABC123"
    name      // "Widget"
    price     // 19.99
  }
}

// Cache structure (using SKU as key)
{
  'Product:ABC123': {
    __typename: 'Product',
    sku: 'ABC123',
    name: 'Widget',
    price: 19.99
  }
}
```

---

#### **Cache Configuration**

**Complete Cache Setup:**

```typescript
// src/apolloClient.ts
import { ApolloClient, InMemoryCache, makeVar } from '@apollo/client';

// Reactive variables for local state
export const cartItemsVar = makeVar<string[]>([]);
export const isLoggedInVar = makeVar<boolean>(false);

const cache = new InMemoryCache({
  // Type policies
  typePolicies: {
    Query: {
      fields: {
        // Pagination: Merge strategy
        posts: {
          keyArgs: ['filter', 'sortBy'],  // Cache different filters separately
          
          merge(existing = [], incoming, { args }) {
            const offset = args?.offset || 0;
            const merged = existing.slice(0);
            
            // Insert incoming items at offset
            for (let i = 0; i < incoming.length; i++) {
              merged[offset + i] = incoming[i];
            }
            
            return merged;
          },
          
          read(existing, { args }) {
            const offset = args?.offset || 0;
            const limit = args?.limit || existing?.length;
            
            return existing?.slice(offset, offset + limit);
          },
        },
        
        // Local-only field
        cartItems: {
          read() {
            return cartItemsVar();
          },
        },
        
        isLoggedIn: {
          read() {
            return isLoggedInVar();
          },
        },
      },
    },
    
    User: {
      keyFields: ['id'],
      
      fields: {
        // Computed field
        fullName: {
          read(_, { readField }) {
            const firstName = readField('firstName');
            const lastName = readField('lastName');
            return `${firstName} ${lastName}`;
          },
        },
        
        // Custom merge for array
        friends: {
          merge(existing = [], incoming) {
            // Deduplicate friends
            const merged = [...existing];
            incoming.forEach((friend: any) => {
              if (!merged.some((f: any) => f.__ref === friend.__ref)) {
                merged.push(friend);
              }
            });
            return merged;
          },
        },
      },
    },
    
    Post: {
      keyFields: ['id'],
      
      fields: {
        comments: {
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          },
        },
      },
    },
  },
  
  // Possible types for interfaces/unions
  possibleTypes: {
    SearchResult: ['Post', 'User', 'Comment'],
    Node: ['User', 'Post', 'Comment'],
  },
});

export const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache,
});
```

---

#### **Reading from Cache**

**1. Read Query:**

```typescript
import { useApolloClient } from '@apollo/client';
import { gql } from '@apollo/client';

function MyComponent() {
  const client = useApolloClient();

  const GET_USER = gql`
    query GetUser($id: ID!) {
      user(id: $id) {
        id
        name
        email
      }
    }
  `;

  // Read from cache
  const cachedData = client.readQuery({
    query: GET_USER,
    variables: { id: '123' },
  });

  console.log(cachedData?.user.name);  // "John Doe"
}
```

**2. Read Fragment:**

```typescript
// Read specific object from cache
const user = client.readFragment({
  id: 'User:123',  // Cache ID
  fragment: gql`
    fragment UserInfo on User {
      id
      name
      email
    }
  `,
});

console.log(user?.name);  // "John Doe"
```

**3. Watch Query (React Hook):**

```typescript
import { useQuery } from '@apollo/client';

// Automatically reads from cache and subscribes to updates
const { data } = useQuery(GET_USER, {
  variables: { id: '123' },
  fetchPolicy: 'cache-first',  // Read from cache first
});
```

---

#### **Writing to Cache**

**1. Write Query:**

```typescript
client.writeQuery({
  query: GET_USER,
  variables: { id: '123' },
  data: {
    user: {
      __typename: 'User',
      id: '123',
      name: 'Jane Smith',
      email: 'jane@example.com',
    },
  },
});
```

**2. Write Fragment:**

```typescript
// Update specific object
client.writeFragment({
  id: 'User:123',
  fragment: gql`
    fragment UserName on User {
      name
    }
  `,
  data: {
    name: 'Jane Smith',
  },
});
```

**3. Modify Cache:**

```typescript
// More granular updates
client.cache.modify({
  id: 'User:123',
  fields: {
    // Update field value
    name(existingName) {
      return 'Jane Smith';
    },
    
    // Add to array
    friends(existingFriends = [], { readField }) {
      const newFriendRef = client.cache.writeFragment({
        data: { __typename: 'User', id: '456', name: 'Bob' },
        fragment: gql`
          fragment NewFriend on User {
            id
            name
          }
        `,
      });
      
      return [...existingFriends, newFriendRef];
    },
    
    // Increment counter
    postCount(existingCount = 0) {
      return existingCount + 1;
    },
  },
});
```

---

#### **Cache Updates After Mutations**

**Automatic Updates (Same ID):**

```typescript
// Mutation updates user name
const UPDATE_USER = gql`
  mutation UpdateUser($id: ID!, $name: String!) {
    updateUser(id: $id, name: $name) {
      id        # Same ID
      name      # Updated name
      email
    }
  }
`;

// Apollo automatically updates cache for User:123
// All components using this user automatically re-render
```

**Manual Updates (New Objects):**

```typescript
const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
      author {
        id
        name
      }
    }
  }
`;

function CreatePostForm() {
  const [createPost] = useMutation(CREATE_POST, {
    update(cache, { data }) {
      const newPost = data?.createPost;
      if (!newPost) return;

      // Option 1: Modify existing query
      cache.modify({
        fields: {
          posts(existingPosts = []) {
            const newPostRef = cache.writeFragment({
              data: newPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                  content
                }
              `,
            });
            return [newPostRef, ...existingPosts];
          },
        },
      });

      // Option 2: Update specific query
      const GET_POSTS = gql`
        query GetPosts {
          posts {
            id
            title
          }
        }
      `;

      const existingData = cache.readQuery({ query: GET_POSTS });
      if (existingData) {
        cache.writeQuery({
          query: GET_POSTS,
          data: {
            posts: [newPost, ...existingData.posts],
          },
        });
      }
    },
  });
}
```

---

#### **Cache Eviction (Deleting Objects)**

**Evict Object:**

```typescript
const DELETE_POST = gql`
  mutation DeletePost($id: ID!) {
    deletePost(id: $id)
  }
`;

function DeleteButton({ postId }: { postId: string }) {
  const [deletePost] = useMutation(DELETE_POST, {
    update(cache, { data }) {
      if (data?.deletePost) {
        // Remove from cache
        cache.evict({ id: `Post:${postId}` });
        
        // Garbage collect dangling references
        cache.gc();
      }
    },
  });

  return <button onClick={() => deletePost({ variables: { id: postId } })}>
    Delete
  </button>;
}
```

**Evict Field:**

```typescript
// Remove specific field from object
cache.evict({ 
  id: 'User:123', 
  fieldName: 'friends' 
});
```

---

#### **Real-World Example: Social Media App**

```typescript
// src/components/UserProfile.tsx
import { useQuery, useMutation } from '@apollo/client';
import { gql } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      avatar
      postCount
      followerCount
      posts {
        id
        title
        likes
      }
    }
  }
`;

const FOLLOW_USER = gql`
  mutation FollowUser($userId: ID!) {
    followUser(userId: $userId) {
      id
      followerCount  # Automatically updates in cache
      isFollowing
    }
  }
`;

const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(postId: $postId) {
      id
      likes  # Automatically updates in cache
      isLikedByMe
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { data, loading } = useQuery(GET_USER, {
    variables: { id: userId },
  });

  const [followUser] = useMutation(FOLLOW_USER, {
    optimisticResponse: {
      followUser: {
        __typename: 'User',
        id: userId,
        followerCount: (data?.user.followerCount || 0) + 1,
        isFollowing: true,
      },
    },
  });

  const [likePost] = useMutation(LIKE_POST);

  if (loading) return <div>Loading...</div>;
  if (!data?.user) return <div>User not found</div>;

  const { user } = data;

  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h1>{user.name}</h1>
      
      {/* Automatically updates when followUser mutation completes */}
      <p>{user.followerCount} followers</p>
      
      <button onClick={() => followUser({ variables: { userId } })}>
        Follow
      </button>

      <h2>Posts ({user.postCount})</h2>
      {user.posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          
          {/* Automatically updates when likePost mutation completes */}
          <button onClick={() => likePost({ variables: { postId: post.id } })}>
            ❤️ {post.likes}
          </button>
        </div>
      ))}
    </div>
  );
}

// When followUser mutation completes:
// 1. Cache updates User:123 with new followerCount
// 2. All components using User:123 automatically re-render
// 3. No manual state management needed!
```

---

#### **Cache Persistence**

**Save cache to localStorage:**

```typescript
import { persistCache } from 'apollo3-cache-persist';
import { InMemoryCache } from '@apollo/client';

const cache = new InMemoryCache();

// Persist cache to localStorage
await persistCache({
  cache,
  storage: window.localStorage,
  maxSize: 1048576, // 1 MB
  debug: true,
});

export const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache,
});
```

**Clear persisted cache:**

```typescript
import { persistCache } from 'apollo3-cache-persist';

// Clear cache
await client.clearStore();  // Clear Apollo cache
localStorage.removeItem('apollo-cache-persist');  // Clear persisted cache
```

---

#### **Cache Policies (Fetch Strategies)**

```typescript
const { data } = useQuery(GET_USER, {
  variables: { id: '123' },
  
  // Fetch policies
  fetchPolicy: 'cache-first',  // Default: Use cache, fallback to network
  // fetchPolicy: 'cache-only',    // Only read from cache, never network
  // fetchPolicy: 'network-only',  // Always fetch from network, update cache
  // fetchPolicy: 'no-cache',      // Fetch from network, don't update cache
  // fetchPolicy: 'cache-and-network',  // Use cache immediately, then fetch
});
```

**Policy Comparison:**

| Policy | Read Cache | Network Request | Update Cache | Use Case |
|--------|------------|-----------------|--------------|----------|
| `cache-first` | ✅ First | Only if cache miss | ✅ Yes | Default, best performance |
| `cache-only` | ✅ Only | ❌ Never | N/A | Offline mode |
| `network-only` | ❌ No | ✅ Always | ✅ Yes | Need fresh data |
| `no-cache` | ❌ No | ✅ Always | ❌ No | Don't cache sensitive data |
| `cache-and-network` | ✅ First | ✅ Always | ✅ Yes | Show cached, update with fresh |

---

#### **Debugging Cache**

**1. Apollo DevTools:**

Install browser extension to inspect cache:
- View all cached objects
- See cache IDs and references
- Inspect field values
- Manually modify cache

**2. Log Cache Contents:**

```typescript
// Log entire cache
console.log(client.cache.extract());

// Example output:
{
  'ROOT_QUERY': {
    __typename: 'Query',
    'user({"id":"123"})': { __ref: 'User:123' }
  },
  'User:123': {
    __typename: 'User',
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    posts: [
      { __ref: 'Post:1' },
      { __ref: 'Post:2' }
    ]
  },
  'Post:1': {
    __typename: 'Post',
    id: '1',
    title: 'GraphQL Guide'
  }
}
```

**3. Monitor Cache Changes:**

```typescript
// Watch for cache updates
cache.watch({
  query: GET_USER,
  variables: { id: '123' },
  callback: (data) => {
    console.log('Cache updated:', data);
  },
});
```

---

#### **Normalized Caching in Other Clients**

**Relay:**

Relay pioneered normalized caching for GraphQL:

```typescript
// Relay automatically normalizes by Node interface
// Every type implementing Node interface is cached by id

interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
}

// Cache structure
{
  'User:123': { id: '123', name: 'John' }
}
```

**urql:**

urql uses `@urql/exchange-graphcache` for normalization:

```typescript
import { createClient, cacheExchange } from 'urql';
import { cacheExchange as graphcacheExchange } from '@urql/exchange-graphcache';

const client = createClient({
  url: 'http://localhost:4000/graphql',
  exchanges: [
    graphcacheExchange({
      keys: {
        User: data => data.id,
        Post: data => data.id,
      },
    }),
  ],
});
```

---

#### **Benefits of Normalized Caching**

**1. Consistency:**
```typescript
// Update user in one place
cache.writeFragment({
  id: 'User:123',
  fragment: gql`fragment UserName on User { name }`,
  data: { name: 'Jane Smith' },
});

// Automatically updates everywhere:
// - User profile page
// - Post author names
// - Comment authors
// - Friend lists
// All components re-render with new name
```

**2. Performance:**
```typescript
// Cache hit - no network request
const { data } = useQuery(GET_USER, {
  variables: { id: '123' },
  fetchPolicy: 'cache-first',
});
// Instant response from cache
```

**3. Optimistic UI:**
```typescript
// Update cache immediately before server response
const [updateUser] = useMutation(UPDATE_USER, {
  optimisticResponse: {
    updateUser: {
      __typename: 'User',
      id: '123',
      name: 'New Name',
    },
  },
});
// UI updates instantly, reverts if mutation fails
```

**4. Offline Support:**
```typescript
// All queries can be served from cache when offline
// Mutations queued and sent when online
```

---

#### **Common Pitfalls**

**1. Missing `id` or `__typename`:**

```typescript
// ❌ Bad: No id returned
query {
  user(id: "123") {
    name  # Missing id!
  }
}
// Can't normalize without id

// ✅ Good: Include id
query {
  user(id: "123") {
    id    # Required for normalization
    name
  }
}
```

**2. Inconsistent IDs:**

```typescript
// ❌ Bad: Different ID types
{ id: "123" }   // String
{ id: 123 }     // Number
// Creates different cache keys: User:123 vs User:"123"

// ✅ Good: Consistent ID format
{ id: "123" }   // Always string
```

**3. Not Including Updated Fields:**

```typescript
// ❌ Bad: Mutation doesn't return updated fields
mutation {
  updateUser(id: "123", name: "Jane") {
    success  # Only returns boolean
  }
}
// Cache not updated!

// ✅ Good: Return all updated fields
mutation {
  updateUser(id: "123", name: "Jane") {
    id
    name
    email
  }
}
// Cache automatically updated
```

---

#### **Summary**

**Normalized caching:**

1. **Flat Structure**: Objects stored once by ID (`User:123`)
2. **References**: Nested objects replaced with references (`{ __ref: 'User:123' }`)
3. **Automatic Updates**: Updating object updates all usages
4. **Cache Keys**: Generated from `__typename:id` or custom `keyFields`
5. **Consistency**: Same data everywhere in the app
6. **Performance**: No duplicate data, efficient memory usage
7. **Read/Write**: `readQuery`, `readFragment`, `writeQuery`, `writeFragment`, `modify`
8. **Eviction**: Remove objects with `evict()` and `gc()`
9. **Fetch Policies**: Control when to use cache vs network
10. **Debugging**: Apollo DevTools to inspect cache structure

**Key Operations:**

- **Read**: `client.readQuery()`, `client.readFragment()`
- **Write**: `client.writeQuery()`, `client.writeFragment()`
- **Modify**: `cache.modify()`
- **Delete**: `cache.evict()`, `cache.gc()`
- **Extract**: `cache.extract()` (for debugging)

Normalized caching is the **foundation of efficient GraphQL clients**, enabling automatic UI updates, optimistic responses, and offline support with minimal developer effort.

</details>

---

### 226. How do you handle GraphQL errors in React?

<details>
<parameter name="summary">View Answer</summary>

**GraphQL error handling** in React requires understanding the **two types of errors** that can occur: **network errors** (connection failures, server down) and **GraphQL errors** (validation errors, resolver errors, authorization errors). Both types require different handling strategies to provide good user experience and debugging capabilities.

Unlike REST where errors are primarily HTTP status codes, GraphQL typically returns **200 OK** even when errors occur, with error details in the response body.

---

#### **Types of GraphQL Errors**

**1. Network Errors:**

```typescript
// Connection failures, server unreachable, timeout
{
  "errors": null,
  "networkError": {
    "name": "ServerError",
    "message": "Failed to fetch",
    "statusCode": 500
  }
}
```

**2. GraphQL Errors:**

```typescript
// Query errors, validation errors, resolver errors
{
  "data": null,  // or partial data
  "errors": [
    {
      "message": "User not found",
      "locations": [{ "line": 2, "column": 3 }],
      "path": ["user"],
      "extensions": {
        "code": "NOT_FOUND",
        "userId": "123"
      }
    }
  ]
}
```

**3. Partial Errors:**

```typescript
// Some fields succeed, some fail
{
  "data": {
    "user": {
      "id": "123",
      "name": "John Doe",
      "posts": null  // Failed to fetch posts
    }
  },
  "errors": [
    {
      "message": "Failed to load posts",
      "path": ["user", "posts"]
    }
  ]
}
```

---

#### **Error Handling with Apollo Client**

**1. Query Error Handling:**

```typescript
// src/components/UserProfile.tsx
import { useQuery, gql } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      posts {
        id
        title
      }
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { loading, error, data, refetch } = useQuery(GET_USER, {
    variables: { id: userId },
    
    // Error policy
    errorPolicy: 'all',  // 'none', 'ignore', 'all'
    
    // Callback when error occurs
    onError: (error) => {
      console.error('Query error:', error);
      
      // Log to error tracking service
      if (window.Sentry) {
        window.Sentry.captureException(error);
      }
    },
  });

  // Loading state
  if (loading) return <div>Loading user...</div>;

  // Error handling
  if (error) {
    // Network error
    if (error.networkError) {
      return (
        <div className="error">
          <h2>Connection Error</h2>
          <p>Unable to connect to the server. Please check your internet connection.</p>
          <button onClick={() => refetch()}>Retry</button>
        </div>
      );
    }

    // GraphQL errors
    if (error.graphQLErrors.length > 0) {
      const firstError = error.graphQLErrors[0];
      
      // Handle specific error codes
      if (firstError.extensions?.code === 'UNAUTHENTICATED') {
        return (
          <div className="error">
            <h2>Authentication Required</h2>
            <p>Please log in to view this profile.</p>
            <button onClick={() => window.location.href = '/login'}>
              Log In
            </button>
          </div>
        );
      }
      
      if (firstError.extensions?.code === 'NOT_FOUND') {
        return (
          <div className="error">
            <h2>User Not Found</h2>
            <p>The user you're looking for doesn't exist.</p>
            <button onClick={() => window.history.back()}>Go Back</button>
          </div>
        );
      }
      
      if (firstError.extensions?.code === 'FORBIDDEN') {
        return (
          <div className="error">
            <h2>Access Denied</h2>
            <p>You don't have permission to view this profile.</p>
          </div>
        );
      }
      
      // Generic GraphQL error
      return (
        <div className="error">
          <h2>Error</h2>
          <p>{firstError.message}</p>
          <button onClick={() => refetch()}>Try Again</button>
        </div>
      );
    }

    // Unknown error
    return (
      <div className="error">
        <h2>Something went wrong</h2>
        <p>{error.message}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  // Partial errors (data exists but has errors)
  if (data && error) {
    return (
      <div>
        <div className="user-info">
          <h1>{data.user.name}</h1>
          <p>{data.user.email}</p>
        </div>
        
        {/* Show warning for partial failure */}
        <div className="warning">
          <p>⚠️ Some information could not be loaded</p>
          <button onClick={() => refetch()}>Retry</button>
        </div>
      </div>
    );
  }

  // Success
  return (
    <div className="user-profile">
      <h1>{data.user.name}</h1>
      <p>{data.user.email}</p>
      
      <h2>Posts</h2>
      {data.user.posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

**2. Mutation Error Handling:**

```typescript
// src/components/CreatePostForm.tsx
import { useMutation, gql } from '@apollo/client';
import { useState } from 'react';

const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
    }
  }
`;

function CreatePostForm() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [errorMessage, setErrorMessage] = useState('');

  const [createPost, { loading, error, reset }] = useMutation(CREATE_POST, {
    onError: (error) => {
      console.error('Mutation error:', error);
      
      // Extract user-friendly error message
      if (error.graphQLErrors.length > 0) {
        const gqlError = error.graphQLErrors[0];
        
        // Handle validation errors
        if (gqlError.extensions?.code === 'VALIDATION_ERROR') {
          const validationErrors = gqlError.extensions.validationErrors;
          setErrorMessage(
            Object.values(validationErrors).join(', ')
          );
          return;
        }
        
        // Handle rate limiting
        if (gqlError.extensions?.code === 'RATE_LIMITED') {
          setErrorMessage('Too many requests. Please try again later.');
          return;
        }
        
        setErrorMessage(gqlError.message);
      } else if (error.networkError) {
        setErrorMessage('Network error. Please check your connection.');
      } else {
        setErrorMessage('An unexpected error occurred.');
      }
    },
    
    onCompleted: (data) => {
      console.log('Post created:', data.createPost);
      setTitle('');
      setContent('');
      setErrorMessage('');
    },
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setErrorMessage('');
    
    // Client-side validation
    if (!title.trim()) {
      setErrorMessage('Title is required');
      return;
    }
    
    if (content.length < 10) {
      setErrorMessage('Content must be at least 10 characters');
      return;
    }

    try {
      await createPost({
        variables: {
          input: { title, content },
        },
      });
    } catch (err) {
      // Error handled by onError callback
      console.error('Caught error:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        disabled={loading}
      />
      
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Post content"
        disabled={loading}
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      
      {/* Display error message */}
      {errorMessage && (
        <div className="error-message">
          {errorMessage}
          <button onClick={() => setErrorMessage('')}>✕</button>
        </div>
      )}
      
      {/* Display raw error for debugging */}
      {process.env.NODE_ENV === 'development' && error && (
        <details className="error-details">
          <summary>Debug Info</summary>
          <pre>{JSON.stringify(error, null, 2)}</pre>
        </details>
      )}
    </form>
  );
}
```

---

#### **Global Error Handling**

**Error Link (Apollo Client):**

```typescript
// src/apolloClient.ts
import {
  ApolloClient,
  InMemoryCache,
  createHttpLink,
  from,
} from '@apollo/client';
import { onError } from '@apollo/client/link/error';
import { setContext } from '@apollo/client/link/context';

// Error handling link
const errorLink = onError(({ graphQLErrors, networkError, operation, forward }) => {
  // GraphQL errors
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path, extensions }) => {
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      );
      
      // Handle authentication errors globally
      if (extensions?.code === 'UNAUTHENTICATED') {
        // Clear token
        localStorage.removeItem('authToken');
        
        // Redirect to login
        window.location.href = '/login';
        
        // Don't continue the request
        return;
      }
      
      // Handle authorization errors
      if (extensions?.code === 'FORBIDDEN') {
        window.location.href = '/403';
        return;
      }
      
      // Handle rate limiting
      if (extensions?.code === 'RATE_LIMITED') {
        // Show global notification
        showNotification('Too many requests. Please slow down.', 'warning');
        
        // Retry after delay
        const retryAfter = extensions.retryAfter || 5000;
        return new Promise((resolve) => {
          setTimeout(() => {
            resolve(forward(operation));
          }, retryAfter);
        });
      }
      
      // Log to error tracking
      if (window.Sentry) {
        window.Sentry.captureException(new Error(message), {
          extra: {
            locations,
            path,
            extensions,
          },
        });
      }
    });
  }

  // Network errors
  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
    
    // Check if server is down
    if (networkError.message === 'Failed to fetch') {
      showNotification('Unable to connect to server. Please check your internet connection.', 'error');
    }
    
    // Check for specific status codes
    if ('statusCode' in networkError) {
      const statusCode = networkError.statusCode;
      
      if (statusCode === 500) {
        showNotification('Server error. Please try again later.', 'error');
      } else if (statusCode === 503) {
        showNotification('Service unavailable. Please try again later.', 'error');
      }
    }
    
    // Log to error tracking
    if (window.Sentry) {
      window.Sentry.captureException(networkError);
    }
  }
});

// Auth link
const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('authToken');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

// HTTP link
const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
  credentials: 'include',
});

// Combine links
export const client = new ApolloClient({
  link: from([errorLink, authLink, httpLink]),
  cache: new InMemoryCache(),
  
  // Default error policy
  defaultOptions: {
    watchQuery: {
      errorPolicy: 'all',  // Return both data and errors
    },
    query: {
      errorPolicy: 'all',
    },
    mutate: {
      errorPolicy: 'all',
    },
  },
});

// Helper function to show notifications
function showNotification(message: string, type: 'info' | 'warning' | 'error') {
  // Implement your notification system
  console.log(`[${type.toUpperCase()}] ${message}`);
}
```

---

#### **Error Policies**

Apollo Client provides three error policies:

```typescript
const { data, error } = useQuery(GET_USER, {
  variables: { id: '123' },
  
  // Error policies:
  errorPolicy: 'none',  // Default: treat GraphQL errors as runtime errors
  // errorPolicy: 'ignore',      // Ignore errors, return data only
  // errorPolicy: 'all',         // Return both data and errors
});
```

**Comparison:**

| Policy | Behavior | `data` | `error` | Use Case |
|--------|----------|--------|---------|----------|
| `none` | Throw error | `undefined` | Set | Critical data, fail fast |
| `ignore` | Ignore errors | Set (if any) | `undefined` | Non-critical data |
| `all` | Return both | Set (if any) | Set (if any) | Partial data acceptable |

**Example:**

```typescript
// errorPolicy: 'none' (default)
const { data, error } = useQuery(GET_USER, { errorPolicy: 'none' });
// If error: data = undefined, error = Error object
// Component must handle error state

// errorPolicy: 'ignore'
const { data, error } = useQuery(GET_USER, { errorPolicy: 'ignore' });
// If error: data = partial data, error = undefined
// Component can render partial data without knowing about error

// errorPolicy: 'all'
const { data, error } = useQuery(GET_USER, { errorPolicy: 'all' });
// If error: data = partial data, error = Error object
// Component can show data + error message
```

---

#### **Custom Error Handling Hook**

Create a reusable hook for consistent error handling:

```typescript
// src/hooks/useGraphQLError.ts
import { ApolloError } from '@apollo/client';
import { useCallback } from 'react';

interface ErrorHandlerOptions {
  onNetworkError?: (error: Error) => void;
  onAuthError?: () => void;
  onNotFound?: () => void;
  onForbidden?: () => void;
  onValidationError?: (errors: Record<string, string>) => void;
  onUnknownError?: (message: string) => void;
}

export function useGraphQLError(options: ErrorHandlerOptions = {}) {
  const handleError = useCallback((error: ApolloError | undefined) => {
    if (!error) return null;

    // Network error
    if (error.networkError) {
      if (options.onNetworkError) {
        options.onNetworkError(error.networkError);
      }
      return {
        type: 'network',
        message: 'Unable to connect to server',
        retry: true,
      };
    }

    // GraphQL errors
    if (error.graphQLErrors.length > 0) {
      const gqlError = error.graphQLErrors[0];
      const code = gqlError.extensions?.code;

      switch (code) {
        case 'UNAUTHENTICATED':
          if (options.onAuthError) {
            options.onAuthError();
          }
          return {
            type: 'auth',
            message: 'Please log in to continue',
            retry: false,
          };

        case 'NOT_FOUND':
          if (options.onNotFound) {
            options.onNotFound();
          }
          return {
            type: 'not_found',
            message: 'Resource not found',
            retry: false,
          };

        case 'FORBIDDEN':
          if (options.onForbidden) {
            options.onForbidden();
          }
          return {
            type: 'forbidden',
            message: 'You don\'t have permission to perform this action',
            retry: false,
          };

        case 'VALIDATION_ERROR':
          const validationErrors = gqlError.extensions?.validationErrors || {};
          if (options.onValidationError) {
            options.onValidationError(validationErrors);
          }
          return {
            type: 'validation',
            message: 'Please check your input',
            errors: validationErrors,
            retry: false,
          };

        case 'RATE_LIMITED':
          return {
            type: 'rate_limit',
            message: 'Too many requests. Please try again later.',
            retry: true,
            retryAfter: gqlError.extensions?.retryAfter || 5000,
          };

        default:
          if (options.onUnknownError) {
            options.onUnknownError(gqlError.message);
          }
          return {
            type: 'unknown',
            message: gqlError.message,
            retry: true,
          };
      }
    }

    return {
      type: 'unknown',
      message: error.message,
      retry: true,
    };
  }, [options]);

  return { handleError };
}
```

**Usage:**

```typescript
// src/components/UserProfile.tsx
import { useQuery, gql } from '@apollo/client';
import { useGraphQLError } from '../hooks/useGraphQLError';

function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error, refetch } = useQuery(GET_USER, {
    variables: { id: userId },
  });

  const { handleError } = useGraphQLError({
    onAuthError: () => {
      window.location.href = '/login';
    },
    onNotFound: () => {
      window.location.href = '/404';
    },
  });

  const errorInfo = handleError(error);

  if (loading) return <div>Loading...</div>;

  if (errorInfo) {
    return (
      <div className="error-container">
        <p>{errorInfo.message}</p>
        {errorInfo.retry && (
          <button onClick={() => refetch()}>Retry</button>
        )}
      </div>
    );
  }

  return <div>{data.user.name}</div>;
}
```

---

#### **Error Boundaries**

Catch React errors from GraphQL operations:

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { ApolloError } from '@apollo/client';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
    };
  }

  static getDerivedStateFromError(error: Error): State {
    return {
      hasError: true,
      error,
    };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error boundary caught error:', error, errorInfo);
    
    // Log to error tracking service
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        extra: errorInfo,
      });
    }
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      const error = this.state.error;
      const isApolloError = error instanceof ApolloError;

      return (
        <div className="error-boundary">
          <h1>Something went wrong</h1>
          {isApolloError && error.networkError && (
            <p>Network error. Please check your connection.</p>
          )}
          {isApolloError && error.graphQLErrors.length > 0 && (
            <p>{error.graphQLErrors[0].message}</p>
          )}
          {!isApolloError && (
            <p>{error?.message || 'An unexpected error occurred'}</p>
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

export default ErrorBoundary;
```

**Usage:**

```typescript
// src/App.tsx
import ErrorBoundary from './components/ErrorBoundary';

function App() {
  return (
    <ErrorBoundary>
      <ApolloProvider client={client}>
        <Routes>
          <Route path="/user/:id" element={<UserProfile />} />
        </Routes>
      </ApolloProvider>
    </ErrorBoundary>
  );
}
```

---

#### **Retry Logic**

Automatically retry failed requests:

```typescript
// src/apolloClient.ts
import { RetryLink } from '@apollo/client/link/retry';

const retryLink = new RetryLink({
  delay: {
    initial: 300,      // Initial delay: 300ms
    max: 5000,         // Max delay: 5s
    jitter: true,      // Randomize delay
  },
  attempts: {
    max: 3,            // Max retry attempts
    retryIf: (error, operation) => {
      // Retry on network errors
      if (error && !error.result) {
        return true;
      }
      
      // Retry on specific GraphQL errors
      if (error?.result?.errors) {
        const errors = error.result.errors;
        const isRetryable = errors.some(
          (err: any) => err.extensions?.code === 'RATE_LIMITED'
        );
        return isRetryable;
      }
      
      return false;
    },
  },
});

const client = new ApolloClient({
  link: from([errorLink, retryLink, authLink, httpLink]),
  cache: new InMemoryCache(),
});
```

---

#### **Form Validation Errors**

Handle validation errors from mutations:

```typescript
// src/components/SignupForm.tsx
import { useMutation, gql } from '@apollo/client';
import { useState } from 'react';

const SIGNUP = gql`
  mutation Signup($email: String!, $password: String!) {
    signup(email: $email, password: $password) {
      user {
        id
        email
      }
      token
    }
  }
`;

interface ValidationErrors {
  email?: string;
  password?: string;
}

function SignupForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [validationErrors, setValidationErrors] = useState<ValidationErrors>({});

  const [signup, { loading }] = useMutation(SIGNUP, {
    onError: (error) => {
      const gqlError = error.graphQLErrors[0];
      
      if (gqlError?.extensions?.code === 'VALIDATION_ERROR') {
        // Server validation errors
        setValidationErrors(gqlError.extensions.validationErrors);
      } else {
        // Other errors
        alert(gqlError?.message || 'An error occurred');
      }
    },
    onCompleted: (data) => {
      localStorage.setItem('token', data.signup.token);
      window.location.href = '/dashboard';
    },
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setValidationErrors({});

    // Client-side validation
    const errors: ValidationErrors = {};
    
    if (!email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      errors.email = 'Email is invalid';
    }
    
    if (!password) {
      errors.password = 'Password is required';
    } else if (password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }

    if (Object.keys(errors).length > 0) {
      setValidationErrors(errors);
      return;
    }

    signup({ variables: { email, password } });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
          className={validationErrors.email ? 'error' : ''}
        />
        {validationErrors.email && (
          <span className="error-message">{validationErrors.email}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
          className={validationErrors.password ? 'error' : ''}
        />
        {validationErrors.password && (
          <span className="error-message">{validationErrors.password}</span>
        )}
      </div>

      <button type="submit" disabled={loading}>
        {loading ? 'Signing up...' : 'Sign Up'}
      </button>
    </form>
  );
}
```

---

#### **Real-World Example: E-Commerce Checkout**

```typescript
// src/components/CheckoutForm.tsx
import { useMutation, gql } from '@apollo/client';
import { useState } from 'react';

const CREATE_ORDER = gql`
  mutation CreateOrder($input: CreateOrderInput!) {
    createOrder(input: $input) {
      order {
        id
        total
        status
      }
      errors {
        field
        message
      }
    }
  }
`;

function CheckoutForm() {
  const [formData, setFormData] = useState({
    cardNumber: '',
    expiryDate: '',
    cvv: '',
    address: '',
  });
  const [fieldErrors, setFieldErrors] = useState<Record<string, string>>({});
  const [globalError, setGlobalError] = useState('');

  const [createOrder, { loading }] = useMutation(CREATE_ORDER, {
    onError: (error) => {
      const gqlError = error.graphQLErrors[0];
      
      // Payment errors
      if (gqlError?.extensions?.code === 'PAYMENT_FAILED') {
        setGlobalError('Payment failed. Please check your card details.');
        return;
      }
      
      // Inventory errors
      if (gqlError?.extensions?.code === 'OUT_OF_STOCK') {
        setGlobalError('Some items are out of stock. Please update your cart.');
        return;
      }
      
      // Network errors
      if (error.networkError) {
        setGlobalError('Connection error. Please try again.');
        return;
      }
      
      setGlobalError('An unexpected error occurred. Please try again.');
    },
    
    onCompleted: (data) => {
      if (data.createOrder.errors.length > 0) {
        // Field-level errors from server
        const errors: Record<string, string> = {};
        data.createOrder.errors.forEach((err: any) => {
          errors[err.field] = err.message;
        });
        setFieldErrors(errors);
      } else {
        // Success
        window.location.href = `/order/${data.createOrder.order.id}`;
      }
    },
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setFieldErrors({});
    setGlobalError('');

    try {
      await createOrder({
        variables: {
          input: formData,
        },
      });
    } catch (err) {
      // Handled by onError
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {globalError && (
        <div className="alert alert-error">
          {globalError}
        </div>
      )}

      <input
        type="text"
        value={formData.cardNumber}
        onChange={(e) => setFormData({ ...formData, cardNumber: e.target.value })}
        placeholder="Card Number"
        className={fieldErrors.cardNumber ? 'error' : ''}
      />
      {fieldErrors.cardNumber && <span className="error">{fieldErrors.cardNumber}</span>}

      {/* Other fields... */}

      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Complete Order'}
      </button>
    </form>
  );
}
```

---

#### **Testing Error Handling**

```typescript
// src/components/__tests__/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import { GraphQLError } from 'graphql';
import UserProfile from '../UserProfile';
import { GET_USER } from '../../graphql/queries';

test('displays error message on network error', async () => {
  const mocks = [
    {
      request: {
        query: GET_USER,
        variables: { id: '123' },
      },
      error: new Error('Network error'),
    },
  ];

  render(
    <MockedProvider mocks={mocks}>
      <UserProfile userId="123" />
    </MockedProvider>
  );

  await waitFor(() => {
    expect(screen.getByText(/Connection Error/i)).toBeInTheDocument();
  });
});

test('displays error message on GraphQL error', async () => {
  const mocks = [
    {
      request: {
        query: GET_USER,
        variables: { id: '123' },
      },
      result: {
        errors: [
          new GraphQLError('User not found', {
            extensions: { code: 'NOT_FOUND' },
          }),
        ],
      },
    },
  ];

  render(
    <MockedProvider mocks={mocks}>
      <UserProfile userId="123" />
    </MockedProvider>
  );

  await waitFor(() => {
    expect(screen.getByText(/User Not Found/i)).toBeInTheDocument();
  });
});
```

---

#### **Best Practices**

**1. Use Error Codes:**

```typescript
// Server-side: Include error codes
throw new GraphQLError('User not found', {
  extensions: {
    code: 'NOT_FOUND',
    userId: '123',
  },
});

// Client-side: Handle by code
if (error.extensions?.code === 'NOT_FOUND') {
  // Show not found UI
}
```

**2. Separate Error Types:**

```typescript
// Different handling for different error types
if (error.networkError) {
  // Connection issue - show retry button
} else if (error.graphQLErrors[0]?.extensions?.code === 'UNAUTHENTICATED') {
  // Auth issue - redirect to login
} else {
  // Other errors - show generic message
}
```

**3. User-Friendly Messages:**

```typescript
// ❌ Bad: Show technical error
<p>{error.message}</p>
// "Cannot read property 'posts' of null"

// ✅ Good: Show user-friendly message
<p>We couldn't load your posts. Please try again.</p>
```

**4. Log Errors:**

```typescript
// Always log errors for debugging
console.error('GraphQL error:', error);

// Send to error tracking
Sentry.captureException(error);
```

**5. Provide Recovery Options:**

```typescript
// Always offer a way to recover
<button onClick={() => refetch()}>Retry</button>
<button onClick={() => window.location.reload()}>Refresh Page</button>
<button onClick={() => window.history.back()}>Go Back</button>
```

---

#### **Summary**

**GraphQL error handling in React:**

1. **Two Error Types**: Network errors (connection) and GraphQL errors (query/mutation)
2. **Error Policies**: `none` (throw), `ignore` (suppress), `all` (return both data and errors)
3. **Query Errors**: Use `error` object from `useQuery`, check `networkError` and `graphQLErrors`
4. **Mutation Errors**: Use `onError` callback in `useMutation`
5. **Global Handling**: Use `errorLink` from `@apollo/client/link/error`
6. **Error Codes**: Check `extensions.code` for specific error types
7. **Retry Logic**: Use `RetryLink` for automatic retries
8. **Error Boundaries**: Catch React errors with error boundary component
9. **User Experience**: Show user-friendly messages, provide retry options
10. **Testing**: Mock errors with `MockedProvider` for tests

**Key Steps:**

1. **Check error type**: Network vs GraphQL
2. **Extract error details**: Message, code, extensions
3. **Display user-friendly message**: No technical jargon
4. **Provide recovery**: Retry, refresh, go back
5. **Log errors**: Console, Sentry, analytics
6. **Test error states**: Unit tests for error scenarios

GraphQL error handling requires understanding **both network and GraphQL errors**, using **error policies** appropriately, and providing **excellent user experience** with clear messages and recovery options.

</details>

---

### 227. What is the difference between REST and GraphQL?

<details>
<summary>View Answer</summary>

**REST (Representational State Transfer)** and **GraphQL** are both API architectural patterns for client-server communication, but they differ fundamentally in how data is requested, structured, and delivered. GraphQL was created by Facebook in 2012 to address specific limitations of REST APIs at scale.

---

#### **High-Level Comparison**

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Architecture** | Multiple endpoints | Single endpoint |
| **Data Fetching** | Fixed data structures | Client specifies exact needs |
| **Over-fetching** | ✅ Common (get more data than needed) | ❌ None |
| **Under-fetching** | ✅ Common (need multiple requests) | ❌ None |
| **Versioning** | Required (v1, v2, v3) | ❌ Not needed |
| **Type System** | No built-in types | ✅ Strong typing |
| **Documentation** | Manual (Swagger, OpenAPI) | ✅ Auto-generated |
| **Caching** | ✅ HTTP caching (easy) | ⚠️ Complex (needs normalized cache) |
| **Learning Curve** | ✅ Gentle | ⚠️ Steeper |
| **Tooling** | Mature, widespread | Growing rapidly |
| **Best For** | Simple CRUD, public APIs | Complex data needs, mobile apps |

---

#### **1. Endpoints vs Single Endpoint**

**REST: Multiple Endpoints**

```javascript
// REST API structure
GET    /api/users              // Get all users
GET    /api/users/123          // Get user by ID
POST   /api/users              // Create user
PUT    /api/users/123          // Update user
DELETE /api/users/123          // Delete user
GET    /api/users/123/posts    // Get user's posts
GET    /api/posts/456/comments // Get post's comments
```

Each resource has its own endpoint with different HTTP methods.

**GraphQL: Single Endpoint**

```javascript
// GraphQL API structure
POST /graphql  // Single endpoint for all operations

// Query
POST /graphql
{
  "query": "{ user(id: \"123\") { name posts { title } } }"
}

// Mutation
POST /graphql
{
  "query": "mutation { createUser(name: \"John\") { id name } }"
}

// Subscription
WS /graphql
{
  "query": "subscription { newMessage { text } }"
}
```

All operations go through a single endpoint.

---

#### **2. Data Fetching**

**REST: Fixed Data Structures**

```javascript
// Request
GET /api/users/123

// Response (fixed structure)
{
  "id": 123,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",      // Don't need this
  "address": {                  // Don't need this
    "street": "123 Main St",
    "city": "NYC"
  },
  "createdAt": "2024-01-01",   // Don't need this
  "updatedAt": "2024-12-01"    // Don't need this
}

// Problem: Over-fetching (getting data you don't need)
```

**GraphQL: Client Specifies Data**

```javascript
// Request (specify exact fields)
POST /graphql
{
  "query": "{ user(id: 123) { firstName lastName email } }"
}

// Response (exact data requested)
{
  "data": {
    "user": {
      "firstName": "John",
      "lastName": "Doe",
      "email": "john@example.com"
    }
  }
}

// ✅ No over-fetching
```

---

#### **3. Over-fetching Problem**

**REST: Get More Data Than Needed**

```javascript
// Mobile app needs only name and avatar
GET /api/users/123

// But gets everything:
{
  "id": 123,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "address": { "street": "123 Main St", "city": "NYC", "zip": "10001" },
  "bio": "Long bio text...",
  "website": "https://example.com",
  "socialMedia": { "twitter": "@john", "linkedin": "john-doe" },
  "preferences": { "theme": "dark", "notifications": true },
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-12-01T00:00:00Z",
  "lastLogin": "2024-12-27T10:00:00Z"
}

// Wasted bandwidth: ~80% of data not used
// Problem for mobile users with limited data
```

**GraphQL: Request Exact Data**

```javascript
// Mobile app specifies exact needs
query {
  user(id: 123) {
    firstName
    avatar
  }
}

// Response: Only what's needed
{
  "data": {
    "user": {
      "firstName": "John",
      "avatar": "avatar.jpg"
    }
  }
}

// ✅ Minimal bandwidth usage
```

---

#### **4. Under-fetching Problem**

**REST: Multiple Requests Required**

```javascript
// Get user profile page with posts and comments

// Request 1: Get user
GET /api/users/123
// Response: { id: 123, name: "John" }

// Request 2: Get user's posts
GET /api/users/123/posts
// Response: [{ id: 1, title: "Post 1" }, { id: 2, title: "Post 2" }]

// Request 3: Get comments for post 1
GET /api/posts/1/comments
// Response: [{ id: 1, text: "Comment 1" }]

// Request 4: Get comments for post 2
GET /api/posts/2/comments
// Response: [{ id: 2, text: "Comment 2" }]

// Total: 4 requests (n+1 problem)
// High latency, especially on mobile
```

**GraphQL: Single Request**

```javascript
// One request gets everything
query {
  user(id: 123) {
    name
    posts {
      id
      title
      comments {
        id
        text
      }
    }
  }
}

// Response: All data in one go
{
  "data": {
    "user": {
      "name": "John",
      "posts": [
        {
          "id": 1,
          "title": "Post 1",
          "comments": [{ "id": 1, "text": "Comment 1" }]
        },
        {
          "id": 2,
          "title": "Post 2",
          "comments": [{ "id": 2, "text": "Comment 2" }]
        }
      ]
    }
  }
}

// ✅ Single request
```

---

#### **5. Versioning**

**REST: Requires API Versions**

```javascript
// Version 1
GET /api/v1/users/123
{
  "id": 123,
  "name": "John Doe"  // Single name field
}

// Version 2 (breaking change)
GET /api/v2/users/123
{
  "id": 123,
  "firstName": "John",  // Split into firstName and lastName
  "lastName": "Doe"
}

// Problems:
// - Maintain multiple versions simultaneously
// - Migrate clients from v1 to v2
// - Deprecate old versions
// - More endpoints to maintain
```

**GraphQL: Deprecation Instead of Versioning**

```javascript
// Original schema
type User {
  id: ID!
  name: String!
}

// Add new fields (non-breaking)
type User {
  id: ID!
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}

// Old clients still work
query {
  user(id: 123) {
    name  # Still works
  }
}

// New clients use new fields
query {
  user(id: 123) {
    firstName
    lastName
  }
}

// ✅ No breaking changes
// ✅ Single endpoint
// ✅ Gradual migration
```

---

#### **6. Type System**

**REST: No Built-in Types**

```javascript
// API documentation (manual)
/**
 * GET /api/users/:id
 * 
 * Response:
 * {
 *   id: number
 *   name: string
 *   email: string
 * }
 */

// Runtime type checking needed
fetch('/api/users/123')
  .then(res => res.json())
  .then(data => {
    // No guarantee about data structure
    console.log(data.name);  // Hope this exists
  });
```

**GraphQL: Strong Type System**

```graphql
# Schema definition (types built-in)
type User {
  id: ID!           # ! means non-nullable
  name: String!
  email: String!
  age: Int
  isActive: Boolean!
  posts: [Post!]!   # Array of Posts
}

type Post {
  id: ID!
  title: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users: [User!]!
}
```

```typescript
// TypeScript types auto-generated from schema
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
  isActive: boolean;
  posts: Post[];
}

// Type-safe queries
const { data } = useQuery<{ user: User }>(GET_USER);
console.log(data?.user.name);  // Type-safe
```

---

#### **7. Documentation**

**REST: Manual Documentation**

```yaml
# OpenAPI/Swagger (manual)
openapi: 3.0.0
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User object
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: integer
                  name:
                    type: string

# Manual work to keep in sync
# Documentation can become outdated
```

**GraphQL: Auto-generated Documentation**

```graphql
# Schema IS the documentation
type User {
  id: ID!
  name: String!
  email: String!
}

type Query {
  "Get user by ID"
  user(id: ID!): User
  
  "Get all users"
  users: [User!]!
}
```

```javascript
// GraphiQL/GraphQL Playground auto-generates documentation
// - Auto-complete
// - Type hints
// - Field descriptions
// - Always up-to-date
```

---

#### **8. Caching**

**REST: HTTP Caching (Simple)**

```javascript
// HTTP caching built-in
GET /api/users/123
Cache-Control: max-age=3600

// Browser automatically caches
// CDN caching works out of the box
// ETags, Last-Modified headers
```

**GraphQL: Normalized Caching (Complex)**

```javascript
// POST request (can't use HTTP cache)
POST /graphql

// Needs client-side normalized cache
const cache = new InMemoryCache({
  typePolicies: {
    User: {
      keyFields: ['id'],
    },
  },
});

// More setup required
// But more intelligent caching
```

---

#### **9. Real-World Example: User Dashboard**

**REST Implementation**

```javascript
// Component needs: User name, avatar, post count, latest 3 posts

// Request 1: Get user
fetch('/api/users/123')
  .then(res => res.json())
  .then(user => {
    // Gets: id, name, avatar, email, phone, address, bio, ...
    // Over-fetching: 70% of data not needed
  });

// Request 2: Get post count
fetch('/api/users/123/posts/count')
  .then(res => res.json())
  .then(count => {
    // Gets: { count: 42 }
  });

// Request 3: Get latest posts
fetch('/api/users/123/posts?limit=3&sort=createdAt:desc')
  .then(res => res.json())
  .then(posts => {
    // Gets: Full post objects with content, tags, metadata, ...
    // Over-fetching: Only need id and title
  });

// Total: 3 requests
// Problems:
// - Multiple round trips
// - Over-fetching in each request
// - Waterfall (Request 2 and 3 wait for Request 1)
```

**GraphQL Implementation**

```javascript
// Single request for everything
const GET_USER_DASHBOARD = gql`
  query GetUserDashboard($userId: ID!) {
    user(id: $userId) {
      name
      avatar
      postCount
      latestPosts(limit: 3) {
        id
        title
      }
    }
  }
`;

const { data } = useQuery(GET_USER_DASHBOARD, {
  variables: { userId: '123' },
});

// Response: Exact data needed
{
  "data": {
    "user": {
      "name": "John",
      "avatar": "avatar.jpg",
      "postCount": 42,
      "latestPosts": [
        { "id": "1", "title": "Post 1" },
        { "id": "2", "title": "Post 2" },
        { "id": "3", "title": "Post 3" }
      ]
    }
  }
}

// Total: 1 request
// ✅ No over-fetching
// ✅ No under-fetching
// ✅ Lower latency
```

---

#### **10. Error Handling**

**REST: HTTP Status Codes**

```javascript
// Success
GET /api/users/123
200 OK
{ "id": 123, "name": "John" }

// Not found
GET /api/users/999
404 Not Found
{ "error": "User not found" }

// Server error
GET /api/users/123
500 Internal Server Error
{ "error": "Database connection failed" }

// Clear status codes
// Easy to cache based on status
```

**GraphQL: Always 200 OK**

```javascript
// Success
POST /graphql
200 OK
{
  "data": {
    "user": { "id": "123", "name": "John" }
  }
}

// Not found (still 200 OK)
POST /graphql
200 OK
{
  "data": null,
  "errors": [
    {
      "message": "User not found",
      "extensions": { "code": "NOT_FOUND" }
    }
  ]
}

// Server error (still 200 OK)
POST /graphql
200 OK
{
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": { "code": "INTERNAL_ERROR" }
    }
  ]
}

// Harder to cache based on status
// Need to parse response body
```

---

#### **11. File Uploads**

**REST: Native Support**

```javascript
// Simple multipart/form-data
const formData = new FormData();
formData.append('file', file);
formData.append('userId', '123');

fetch('/api/upload', {
  method: 'POST',
  body: formData,
});

// Built-in browser support
// Progress tracking easy
```

**GraphQL: More Complex**

```javascript
// Requires special handling
const UPLOAD_FILE = gql`
  mutation UploadFile($file: Upload!) {
    uploadFile(file: $file) {
      id
      url
    }
  }
`;

const [uploadFile] = useMutation(UPLOAD_FILE);

// Needs apollo-upload-client or graphql-upload
// More setup required
```

---

#### **12. Real-Time Updates**

**REST: Polling or WebSockets**

```javascript
// Polling (inefficient)
setInterval(() => {
  fetch('/api/messages')
    .then(res => res.json())
    .then(messages => updateUI(messages));
}, 5000);

// WebSockets (separate protocol)
const ws = new WebSocket('ws://api.example.com');
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateUI(data);
};

// Separate implementation
```

**GraphQL: Built-in Subscriptions**

```javascript
// Subscriptions are part of GraphQL spec
const NEW_MESSAGE_SUBSCRIPTION = gql`
  subscription OnNewMessage {
    newMessage {
      id
      text
      author {
        name
      }
    }
  }
`;

const { data } = useSubscription(NEW_MESSAGE_SUBSCRIPTION);

// Unified with queries and mutations
// Same type system
```

---

#### **13. Batching**

**REST: Manual Batching**

```javascript
// Need custom batch endpoint
POST /api/batch
{
  "requests": [
    { "method": "GET", "url": "/api/users/123" },
    { "method": "GET", "url": "/api/posts/456" }
  ]
}

// Not standardized
```

**GraphQL: Native Batching**

```javascript
// Multiple queries in one request
query {
  user(id: 123) {
    name
  }
  post(id: 456) {
    title
  }
  comments(postId: 456) {
    text
  }
}

// Single HTTP request
// Built into GraphQL
```

---

#### **14. Learning Curve**

**REST**

```
✅ Pros:
- Simple to understand (CRUD operations)
- Familiar to most developers
- Lots of tutorials and resources
- No special tooling required

⚠️ Cons:
- Need to learn best practices
- Different teams implement differently
- Documentation practices vary
```

**GraphQL**

```
⚠️ Pros:
- More concepts to learn (types, resolvers, fragments)
- Need to understand schema definition
- Client-side caching complex
- Requires build tooling

✅ Cons:
- But once learned, very powerful
- Consistent patterns
- Great tooling
- Strong community
```

---

#### **15. Use Cases**

**When to Use REST:**

✅ Simple CRUD operations
✅ Public APIs (GitHub API, Twitter API)
✅ Need HTTP caching
✅ File downloads
✅ Third-party integrations
✅ Team unfamiliar with GraphQL
✅ Microservices architecture
✅ Need mature tooling ecosystem

**When to Use GraphQL:**

✅ Complex data requirements
✅ Mobile apps (bandwidth critical)
✅ Multiple clients with different needs (web, mobile, desktop)
✅ Rapid iteration on UI
✅ Need to reduce over/under-fetching
✅ Real-time features
✅ Want strong typing
✅ Large-scale applications (Facebook, GitHub, Shopify)

---

#### **16. Performance Comparison**

**REST**

```
Pros:
✅ HTTP caching (CDN, browser)
✅ Simple server implementation
✅ Stateless
✅ Horizontal scaling easy

Cons:
❌ Multiple round trips
❌ Over-fetching (wasted bandwidth)
❌ Under-fetching (high latency)
```

**GraphQL**

```
Pros:
✅ Single request (lower latency)
✅ No over-fetching (less bandwidth)
✅ No under-fetching (fewer requests)
✅ Efficient for mobile

Cons:
❌ Complex server queries (N+1 problem)
❌ No HTTP caching
❌ Requires DataLoader or similar
```

---

#### **17. Side-by-Side Code Example**

**REST Implementation:**

```typescript
// React component using REST
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    Promise.all([
      fetch(`/api/users/${userId}`).then(r => r.json()),
      fetch(`/api/users/${userId}/posts?limit=5`).then(r => r.json()),
    ])
      .then(([userData, postsData]) => {
        setUser(userData);
        setPosts(postsData);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      {posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

**GraphQL Implementation:**

```typescript
// React component using GraphQL
const GET_USER_PROFILE = gql`
  query GetUserProfile($userId: ID!) {
    user(id: $userId) {
      name
      email
      posts(limit: 5) {
        id
        title
      }
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { loading, data } = useQuery(GET_USER_PROFILE, {
    variables: { userId },
  });

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>{data.user.name}</h1>
      <p>{data.user.email}</p>
      {data.user.posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

---

#### **Summary**

**Key Differences:**

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Philosophy** | Resource-based | Query-based |
| **Endpoints** | Many (`/users`, `/posts`) | One (`/graphql`) |
| **Data** | Fixed by endpoint | Client specifies |
| **Over-fetching** | Common | None |
| **Under-fetching** | Common (N+1) | None |
| **Versioning** | v1, v2, v3 | Field deprecation |
| **Types** | No types | Strong typing |
| **Caching** | HTTP (simple) | Normalized (complex) |
| **Real-time** | Polling/WebSockets | Subscriptions |
| **Learning** | Easy | Moderate |
| **Best For** | Simple APIs, public | Complex needs, mobile |

**Choose REST when:**
- Building simple CRUD APIs
- Need HTTP caching
- Team is unfamiliar with GraphQL
- Public API for third parties

**Choose GraphQL when:**
- Complex, nested data requirements
- Multiple clients with different needs
- Mobile apps (bandwidth matters)
- Want to reduce over/under-fetching
- Need strong typing and introspection

Both REST and GraphQL are valid architectural choices—**REST is simpler for basic use cases**, while **GraphQL excels at complex data fetching and mobile optimization**.

</details>

---

### 228. How do you implement optimistic UI with GraphQL?

<details>
<summary>View Answer</summary>

**Optimistic UI** is a pattern where the UI is updated **immediately** when the user performs an action, before waiting for the server response. This creates an **instant, responsive feel** and significantly improves perceived performance. If the server operation fails, the UI is rolled back to its previous state.

With GraphQL and Apollo Client, optimistic UI is implemented using the `optimisticResponse` option in mutations, which provides a predicted response that updates the cache instantly.

---

#### **Why Optimistic UI?**

**Without Optimistic UI (Traditional):**

```typescript
// User clicks "Like" button
1. Show loading spinner
2. Send request to server (300-1000ms)
3. Wait for response
4. Update UI

User experience: Slow, unresponsive
```

**With Optimistic UI:**

```typescript
// User clicks "Like" button
1. Update UI immediately (0ms) ✨
2. Send request to server in background
3. If success: Keep the optimistic update
4. If failure: Roll back to previous state

User experience: Instant, responsive ✨
```

---

#### **How Optimistic UI Works**

**Flow:**

```
1. User clicks button (e.g., "Like")
2. Mutation called with optimisticResponse
3. Apollo immediately updates cache with optimistic data
4. UI re-renders with optimistic data (instant)
5. Request sent to server
6. Server responds
7. Apollo replaces optimistic data with real data
8. If IDs match, UI stays the same (smooth)
9. If mutation fails, Apollo rolls back cache automatically
```

---

#### **Basic Implementation**

**Simple Optimistic Update:**

```typescript
// src/components/LikeButton.tsx
import { useMutation, gql } from '@apollo/client';

const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(postId: $postId) {
      id
      likes
      isLikedByMe
    }
  }
`;

interface LikeButtonProps {
  postId: string;
  likes: number;
  isLiked: boolean;
}

function LikeButton({ postId, likes, isLiked }: LikeButtonProps) {
  const [likePost, { loading }] = useMutation(LIKE_POST, {
    variables: { postId },
    
    // Optimistic response - predict server response
    optimisticResponse: {
      likePost: {
        __typename: 'Post',
        id: postId,
        likes: isLiked ? likes - 1 : likes + 1,  // Toggle likes
        isLikedByMe: !isLiked,  // Toggle liked state
      },
    },
  });

  return (
    <button onClick={() => likePost()} disabled={loading}>
      {isLiked ? '❤️' : '🤍'} {likes}
    </button>
  );
}

// When user clicks:
// 1. Button immediately shows ❤️ and incremented count
// 2. Request sent to server
// 3. If success: UI stays the same (smooth)
// 4. If failure: Automatically reverts to 🤍 and original count
```

---

#### **Optimistic Response Requirements**

**Must Include:**

1. **`__typename`**: Required for cache normalization
2. **`id`**: Required to identify the object in cache
3. **All fields returned by mutation**: Must match mutation response structure

```typescript
// ❌ Bad: Missing required fields
optimisticResponse: {
  likePost: {
    likes: likes + 1,  // Missing __typename and id
  },
}

// ✅ Good: Complete response
optimisticResponse: {
  likePost: {
    __typename: 'Post',
    id: postId,
    likes: likes + 1,
    isLikedByMe: true,
  },
}
```

---

#### **Cache Updates with Optimistic UI**

**Automatic Cache Update (Same ID):**

```typescript
// Mutation returns updated object with same ID
// Apollo automatically updates cache

const UPDATE_USER = gql`
  mutation UpdateUser($id: ID!, $name: String!) {
    updateUser(id: $id, name: $name) {
      id      # Same ID as before
      name    # Updated name
      email
    }
  }
`;

function EditProfile({ userId, currentName }: Props) {
  const [updateUser] = useMutation(UPDATE_USER, {
    optimisticResponse: {
      updateUser: {
        __typename: 'User',
        id: userId,
        name: newName,  // Optimistic new name
        email: currentEmail,  // Keep current email
      },
    },
  });

  // Apollo finds User:userId in cache and updates it
  // All components using this user automatically re-render
}
```

**Manual Cache Update (New Objects):**

```typescript
// Creating new objects requires manual cache update

const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
      author {
        id
        name
      }
      createdAt
    }
  }
`;

function CreatePostForm() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const [createPost] = useMutation(CREATE_POST, {
    // Optimistic response
    optimisticResponse: {
      createPost: {
        __typename: 'Post',
        id: `temp-${Date.now()}`,  // Temporary ID
        title,
        content,
        author: {
          __typename: 'User',
          id: currentUserId,
          name: currentUserName,
        },
        createdAt: new Date().toISOString(),
      },
    },
    
    // Update cache manually
    update(cache, { data }) {
      const newPost = data?.createPost;
      if (!newPost) return;

      // Add to posts list
      cache.modify({
        fields: {
          posts(existingPosts = []) {
            const newPostRef = cache.writeFragment({
              data: newPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                  content
                  author {
                    id
                    name
                  }
                }
              `,
            });
            return [newPostRef, ...existingPosts];
          },
        },
      });
    },
  });

  const handleSubmit = () => {
    createPost({
      variables: {
        input: { title, content },
      },
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      <textarea value={content} onChange={e => setContent(e.target.value)} />
      <button type="submit">Create Post</button>
    </form>
  );
}

// Flow:
// 1. User submits form
// 2. Optimistic post appears immediately with temp ID
// 3. Request sent to server
// 4. Server returns real post with real ID
// 5. Apollo replaces temp post with real post
// 6. UI updates smoothly (temp ID → real ID)
```

---

#### **Error Handling with Rollback**

**Automatic Rollback:**

```typescript
const [deletePost] = useMutation(DELETE_POST, {
  optimisticResponse: {
    deletePost: true,
  },
  
  update(cache) {
    // Remove from cache optimistically
    cache.evict({ id: `Post:${postId}` });
    cache.gc();
  },
  
  onError: (error) => {
    // Apollo automatically rolls back cache changes
    console.error('Delete failed:', error);
    
    // Show error message to user
    alert('Failed to delete post. Please try again.');
  },
});

// When delete fails:
// 1. Post reappears in UI automatically
// 2. User sees error message
// 3. No manual rollback needed
```

---

#### **Real-World Example: Social Media Feed**

**Complete Implementation:**

```typescript
// src/components/Post.tsx
import { useMutation, gql } from '@apollo/client';

const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(postId: $postId) {
      id
      likes
      isLikedByMe
    }
  }
`;

const ADD_COMMENT = gql`
  mutation AddComment($postId: ID!, $text: String!) {
    addComment(postId: $postId, text: $text) {
      id
      text
      author {
        id
        name
        avatar
      }
      createdAt
    }
  }
`;

interface Post {
  id: string;
  title: string;
  content: string;
  likes: number;
  isLikedByMe: boolean;
  comments: Comment[];
  author: User;
}

function Post({ post }: { post: Post }) {
  const [commentText, setCommentText] = useState('');

  // Like mutation with optimistic UI
  const [likePost] = useMutation(LIKE_POST, {
    optimisticResponse: {
      likePost: {
        __typename: 'Post',
        id: post.id,
        likes: post.isLikedByMe ? post.likes - 1 : post.likes + 1,
        isLikedByMe: !post.isLikedByMe,
      },
    },
  });

  // Comment mutation with optimistic UI
  const [addComment] = useMutation(ADD_COMMENT, {
    optimisticResponse: {
      addComment: {
        __typename: 'Comment',
        id: `temp-comment-${Date.now()}`,
        text: commentText,
        author: {
          __typename: 'User',
          id: currentUser.id,
          name: currentUser.name,
          avatar: currentUser.avatar,
        },
        createdAt: new Date().toISOString(),
      },
    },
    
    update(cache, { data }) {
      const newComment = data?.addComment;
      if (!newComment) return;

      // Add comment to post's comments
      cache.modify({
        id: cache.identify({ __typename: 'Post', id: post.id }),
        fields: {
          comments(existingComments = []) {
            const newCommentRef = cache.writeFragment({
              data: newComment,
              fragment: gql`
                fragment NewComment on Comment {
                  id
                  text
                  author {
                    id
                    name
                    avatar
                  }
                  createdAt
                }
              `,
            });
            return [...existingComments, newCommentRef];
          },
        },
      });
    },
    
    onCompleted: () => {
      setCommentText('');
    },
  });

  const handleLike = () => {
    likePost({ variables: { postId: post.id } });
  };

  const handleComment = (e: React.FormEvent) => {
    e.preventDefault();
    if (!commentText.trim()) return;

    addComment({
      variables: {
        postId: post.id,
        text: commentText,
      },
    });
  };

  return (
    <article className="post">
      <div className="author">
        <img src={post.author.avatar} alt={post.author.name} />
        <span>{post.author.name}</span>
      </div>

      <h2>{post.title}</h2>
      <p>{post.content}</p>

      {/* Like button - updates instantly */}
      <button onClick={handleLike} className={post.isLikedByMe ? 'liked' : ''}>
        {post.isLikedByMe ? '❤️' : '🤍'} {post.likes}
      </button>

      {/* Comments */}
      <div className="comments">
        {post.comments.map(comment => (
          <div key={comment.id} className="comment">
            <img src={comment.author.avatar} alt={comment.author.name} />
            <div>
              <strong>{comment.author.name}</strong>
              <p>{comment.text}</p>
              <time>{new Date(comment.createdAt).toLocaleString()}</time>
            </div>
          </div>
        ))}
      </div>

      {/* Add comment - appears instantly */}
      <form onSubmit={handleComment}>
        <input
          value={commentText}
          onChange={e => setCommentText(e.target.value)}
          placeholder="Add a comment..."
        />
        <button type="submit">Post</button>
      </form>
    </article>
  );
}
```

---

#### **Advanced Patterns**

**1. Optimistic Toggle:**

```typescript
// Toggle between two states (follow/unfollow, bookmark/unbookmark)
function FollowButton({ userId, isFollowing }: Props) {
  const [followUser] = useMutation(FOLLOW_USER, {
    optimisticResponse: {
      followUser: {
        __typename: 'User',
        id: userId,
        isFollowing: !isFollowing,  // Toggle
        followerCount: isFollowing ? 
          user.followerCount - 1 : 
          user.followerCount + 1,
      },
    },
  });

  return (
    <button onClick={() => followUser({ variables: { userId } })}>
      {isFollowing ? 'Unfollow' : 'Follow'}
    </button>
  );
}
```

**2. Optimistic List Operations:**

```typescript
// Add item to list
const ADD_TODO = gql`
  mutation AddTodo($text: String!) {
    addTodo(text: $text) {
      id
      text
      completed
    }
  }
`;

function TodoForm() {
  const [addTodo] = useMutation(ADD_TODO, {
    optimisticResponse: {
      addTodo: {
        __typename: 'Todo',
        id: `temp-${Date.now()}`,
        text: inputValue,
        completed: false,
      },
    },
    
    update(cache, { data }) {
      cache.modify({
        fields: {
          todos(existingTodos = []) {
            const newTodoRef = cache.writeFragment({
              data: data?.addTodo,
              fragment: gql`
                fragment NewTodo on Todo {
                  id
                  text
                  completed
                }
              `,
            });
            return [...existingTodos, newTodoRef];
          },
        },
      });
    },
  });
}
```

**3. Optimistic Delete:**

```typescript
const DELETE_TODO = gql`
  mutation DeleteTodo($id: ID!) {
    deleteTodo(id: $id)
  }
`;

function TodoItem({ todo }: { todo: Todo }) {
  const [deleteTodo] = useMutation(DELETE_TODO, {
    optimisticResponse: {
      deleteTodo: true,
    },
    
    update(cache) {
      // Remove from cache immediately
      cache.evict({ id: cache.identify(todo) });
      cache.gc();
    },
    
    onError: (error) => {
      // Automatically rolled back if error
      console.error('Delete failed:', error);
      alert('Failed to delete. Please try again.');
    },
  });

  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={() => deleteTodo({ variables: { id: todo.id } })}>
        Delete
      </button>
    </div>
  );
}
```

**4. Optimistic Update with Dependent Data:**

```typescript
// When updating one field affects others
const INCREMENT_COUNTER = gql`
  mutation IncrementCounter($id: ID!) {
    incrementCounter(id: $id) {
      id
      count
      lastModified
    }
  }
`;

function Counter({ counter }: { counter: Counter }) {
  const [increment] = useMutation(INCREMENT_COUNTER, {
    optimisticResponse: {
      incrementCounter: {
        __typename: 'Counter',
        id: counter.id,
        count: counter.count + 1,  // Increment
        lastModified: new Date().toISOString(),  // Update timestamp
      },
    },
  });

  return (
    <div>
      <span>Count: {counter.count}</span>
      <button onClick={() => increment({ variables: { id: counter.id } })}>
        +1
      </button>
      <small>Last updated: {new Date(counter.lastModified).toLocaleString()}</small>
    </div>
  );
}
```

---

#### **Handling Optimistic Response Mismatches**

**Problem: Optimistic Data Doesn't Match Real Data:**

```typescript
// Optimistic response
optimisticResponse: {
  createPost: {
    id: 'temp-123',
    title: 'My Post',
    likes: 0,  // Assume 0 likes
  },
}

// Real server response
{
  createPost: {
    id: 'real-456',
    title: 'My Post',
    likes: 5,  // Server added 5 default likes (auto-liked by followers)
  }
}

// UI will "jump" from 0 → 5 likes
```

**Solution: Match Server Logic:**

```typescript
// Better optimistic response that matches server behavior
optimisticResponse: {
  createPost: {
    id: `temp-${Date.now()}`,
    title: inputValue,
    likes: followerCount,  // Match server's auto-like logic
  },
}
```

---

#### **Complex Example: Shopping Cart**

```typescript
// src/components/ShoppingCart.tsx
import { useMutation, gql } from '@apollo/client';

const ADD_TO_CART = gql`
  mutation AddToCart($productId: ID!, $quantity: Int!) {
    addToCart(productId: $productId, quantity: $quantity) {
      id
      items {
        id
        product {
          id
          name
          price
        }
        quantity
      }
      total
      itemCount
    }
  }
`;

const REMOVE_FROM_CART = gql`
  mutation RemoveFromCart($itemId: ID!) {
    removeFromCart(itemId: $itemId) {
      id
      items {
        id
        product {
          id
          name
          price
        }
        quantity
      }
      total
      itemCount
    }
  }
`;

function ProductCard({ product }: { product: Product }) {
  const { data: cartData } = useQuery(GET_CART);
  const cart = cartData?.cart;

  const [addToCart] = useMutation(ADD_TO_CART, {
    optimisticResponse: {
      addToCart: {
        __typename: 'Cart',
        id: cart.id,
        items: [
          ...cart.items,
          {
            __typename: 'CartItem',
            id: `temp-${Date.now()}`,
            product: {
              __typename: 'Product',
              id: product.id,
              name: product.name,
              price: product.price,
            },
            quantity: 1,
          },
        ],
        total: cart.total + product.price,
        itemCount: cart.itemCount + 1,
      },
    },
  });

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button 
        onClick={() => addToCart({ 
          variables: { productId: product.id, quantity: 1 } 
        })}
      >
        Add to Cart
      </button>
    </div>
  );
}

function CartItem({ item }: { item: CartItem }) {
  const [removeFromCart] = useMutation(REMOVE_FROM_CART, {
    optimisticResponse: ({ itemId }) => {
      const remainingItems = cart.items.filter(i => i.id !== itemId);
      const newTotal = remainingItems.reduce(
        (sum, i) => sum + i.product.price * i.quantity, 
        0
      );

      return {
        removeFromCart: {
          __typename: 'Cart',
          id: cart.id,
          items: remainingItems,
          total: newTotal,
          itemCount: remainingItems.length,
        },
      };
    },
  });

  return (
    <div className="cart-item">
      <span>{item.product.name}</span>
      <span>${item.product.price} × {item.quantity}</span>
      <button onClick={() => removeFromCart({ variables: { itemId: item.id } })}>
        Remove
      </button>
    </div>
  );
}

// User experience:
// 1. Click "Add to Cart" → Item appears instantly in cart badge
// 2. Cart total updates immediately
// 3. Navigate to cart → Item is already there (no loading)
// 4. Click "Remove" → Item disappears instantly
// 5. If network fails → Items reappear, error shown
```

---

#### **Testing Optimistic UI**

```typescript
// src/components/__tests__/LikeButton.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import LikeButton from '../LikeButton';
import { LIKE_POST } from '../../graphql/mutations';

test('updates UI optimistically before server response', async () => {
  const mocks = [
    {
      request: {
        query: LIKE_POST,
        variables: { postId: '123' },
      },
      result: {
        data: {
          likePost: {
            id: '123',
            likes: 11,
            isLikedByMe: true,
          },
        },
      },
      delay: 1000, // Simulate slow network
    },
  ];

  render(
    <MockedProvider mocks={mocks}>
      <LikeButton postId="123" likes={10} isLiked={false} />
    </MockedProvider>
  );

  const button = screen.getByRole('button');
  
  // Initial state
  expect(button).toHaveTextContent('🤍 10');

  // Click button
  fireEvent.click(button);

  // Should update immediately (optimistic)
  expect(button).toHaveTextContent('❤️ 11');

  // Wait for real response
  await waitFor(() => {
    expect(button).toHaveTextContent('❤️ 11'); // Still shows same value
  }, { timeout: 2000 });
});

test('rolls back on error', async () => {
  const mocks = [
    {
      request: {
        query: LIKE_POST,
        variables: { postId: '123' },
      },
      error: new Error('Network error'),
    },
  ];

  render(
    <MockedProvider mocks={mocks}>
      <LikeButton postId="123" likes={10} isLiked={false} />
    </MockedProvider>
  );

  const button = screen.getByRole('button');
  
  // Click button
  fireEvent.click(button);

  // Should update optimistically
  expect(button).toHaveTextContent('❤️ 11');

  // Wait for error
  await waitFor(() => {
    // Should roll back to original state
    expect(button).toHaveTextContent('🤍 10');
  });
});
```

---

#### **Best Practices**

**1. Keep Optimistic Response Realistic:**

```typescript
// ❌ Bad: Unrealistic optimistic response
optimisticResponse: {
  createPost: {
    id: 'temp',
    likes: 1000,  // Unrealistic for new post
  },
}

// ✅ Good: Realistic optimistic response
optimisticResponse: {
  createPost: {
    id: `temp-${Date.now()}`,
    likes: 0,  // New posts start with 0 likes
  },
}
```

**2. Always Include Required Fields:**

```typescript
// ✅ Include __typename, id, and all fields from mutation response
optimisticResponse: {
  updateUser: {
    __typename: 'User',  // Required
    id: userId,          // Required
    name: newName,       // All fields from mutation
    email: currentEmail,
    avatar: currentAvatar,
  },
}
```

**3. Handle Errors Gracefully:**

```typescript
const [mutation] = useMutation(MUTATION, {
  optimisticResponse: { /* ... */ },
  
  onError: (error) => {
    // Apollo rolls back automatically
    
    // Show user-friendly error message
    toast.error('Action failed. Please try again.');
    
    // Log for debugging
    console.error('Mutation failed:', error);
  },
});
```

**4. Use for Quick Actions Only:**

```typescript
// ✅ Good use cases:
// - Like/unlike
// - Follow/unfollow
// - Add to cart
// - Mark as read
// - Toggle checkbox

// ❌ Avoid for:
// - Complex mutations with side effects
// - Mutations that return server-generated data (payment processing)
// - When optimistic response is hard to predict
```

---

#### **Summary**

**Optimistic UI with GraphQL:**

1. **Instant Updates**: UI updates immediately before server response
2. **Implementation**: Use `optimisticResponse` in `useMutation`
3. **Requirements**: Must include `__typename`, `id`, and all returned fields
4. **Automatic Rollback**: Apollo reverts changes on error
5. **Cache Updates**: Works with automatic (same ID) and manual cache updates
6. **Use Cases**: Likes, follows, toggles, adds to cart, simple updates
7. **Testing**: Test both optimistic update and rollback scenarios
8. **UX Benefit**: Perceived performance improvement, app feels instant

**Key Steps:**

1. **Define mutation** with return fields
2. **Add `optimisticResponse`** with predicted data
3. **Include required fields**: `__typename`, `id`
4. **Match structure**: Same as real server response
5. **Handle errors**: Show message, automatic rollback
6. **Test**: Verify instant update and error rollback

Optimistic UI transforms GraphQL apps from feeling like traditional web apps to feeling like **native, instant applications** that respond immediately to user actions.

</details>

---

### 229. What is code generation with GraphQL?

<details>
<summary>View Answer</summary>

**GraphQL Code Generation** is the process of automatically generating **TypeScript types, hooks, and utilities** from your GraphQL schema and operations (queries, mutations, subscriptions). This eliminates manual type definitions, ensures type safety, and keeps your frontend code in sync with your GraphQL API.

The most popular tool is **GraphQL Code Generator** (`@graphql-codegen/cli`), which generates type-safe code from GraphQL files.

---

#### **Why Code Generation?**

**Without Code Generation (Manual):**

```typescript
// ❌ Manual types - prone to errors, out of sync
interface User {
  id: string;
  name: string;
  email: string;
  posts: Post[];
}

interface Post {
  id: string;
  title: string;
  content: string;
}

// Manual typing of query result
const { data } = useQuery<{ user: User }>(GET_USER);

// Problems:
// 1. Types can become outdated when schema changes
// 2. Easy to make typos or mistakes
// 3. No autocomplete for GraphQL operations
// 4. Manual maintenance burden
```

**With Code Generation (Automatic):**

```typescript
// ✅ Auto-generated from GraphQL schema
import { useGetUserQuery, User, Post } from './generated/graphql';

// Fully typed hook generated automatically
const { data, loading, error } = useGetUserQuery({
  variables: { id: '123' },
});

// data.user is fully typed with autocomplete
// TypeScript knows all fields, their types, and nullable properties

// Benefits:
// 1. Always in sync with schema ✨
// 2. Full TypeScript autocomplete
// 3. Catch errors at compile time
// 4. Zero manual maintenance
```

---

#### **How It Works**

**Flow:**

```
1. Write GraphQL schema (backend)
2. Write GraphQL operations (queries/mutations) (frontend)
3. Run code generator
4. Generator reads schema + operations
5. Generates TypeScript types and hooks
6. Import and use generated code
```

---

#### **Setup and Installation**

**1. Install Dependencies:**

```bash
npm install --save-dev @graphql-codegen/cli
npm install --save-dev @graphql-codegen/typescript
npm install --save-dev @graphql-codegen/typescript-operations
npm install --save-dev @graphql-codegen/typescript-react-apollo
```

**2. Create Configuration File:**

```yaml
# codegen.yml
schema: http://localhost:4000/graphql  # GraphQL API endpoint
documents: 'src/**/*.graphql'          # Your GraphQL operations
generates:
  src/generated/graphql.tsx:           # Output file
    plugins:
      - typescript                      # Generate base TypeScript types
      - typescript-operations          # Generate types for operations
      - typescript-react-apollo        # Generate React hooks
    config:
      withHooks: true                  # Generate useQuery hooks
      withHOC: false
      withComponent: false
```

**3. Add Script to package.json:**

```json
{
  "scripts": {
    "codegen": "graphql-codegen --config codegen.yml",
    "codegen:watch": "graphql-codegen --config codegen.yml --watch"
  }
}
```

**4. Run Generator:**

```bash
npm run codegen
```

---

#### **Writing GraphQL Operations**

**Define Operations in .graphql Files:**

```graphql
# src/graphql/queries.graphql

query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    avatar
    posts {
      id
      title
      content
      likes
      createdAt
    }
  }
}

query GetPosts($limit: Int, $offset: Int) {
  posts(limit: $limit, offset: $offset) {
    id
    title
    author {
      id
      name
    }
    likes
    comments {
      id
      text
    }
  }
}

mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
    content
    author {
      id
      name
    }
    createdAt
  }
}

mutation LikePost($postId: ID!) {
  likePost(postId: $postId) {
    id
    likes
    isLikedByMe
  }
}
```

---

#### **Generated Code Example**

**What Gets Generated:**

```typescript
// src/generated/graphql.tsx (auto-generated)

// 1. Base types from schema
export type User = {
  __typename?: 'User';
  id: Scalars['ID'];
  name: Scalars['String'];
  email: Scalars['String'];
  avatar?: Maybe<Scalars['String']>;
  posts: Array<Post>;
};

export type Post = {
  __typename?: 'Post';
  id: Scalars['ID'];
  title: Scalars['String'];
  content: Scalars['String'];
  likes: Scalars['Int'];
  createdAt: Scalars['DateTime'];
  author: User;
  comments: Array<Comment>;
};

// 2. Query types
export type GetUserQueryVariables = Exact<{
  id: Scalars['ID'];
}>;

export type GetUserQuery = {
  __typename?: 'Query';
  user: {
    __typename?: 'User';
    id: string;
    name: string;
    email: string;
    avatar?: string | null;
    posts: Array<{
      __typename?: 'Post';
      id: string;
      title: string;
      content: string;
      likes: number;
      createdAt: any;
    }>;
  };
};

// 3. React hooks
export function useGetUserQuery(
  baseOptions: Apollo.QueryHookOptions<GetUserQuery, GetUserQueryVariables>
) {
  const options = {...defaultOptions, ...baseOptions}
  return Apollo.useQuery<GetUserQuery, GetUserQueryVariables>(
    GetUserDocument, 
    options
  );
}

// 4. Mutation hooks
export type CreatePostMutationVariables = Exact<{
  input: CreatePostInput;
}>;

export type CreatePostMutation = {
  __typename?: 'Mutation';
  createPost: {
    __typename?: 'Post';
    id: string;
    title: string;
    content: string;
    author: {
      __typename?: 'User';
      id: string;
      name: string;
    };
    createdAt: any;
  };
};

export function useCreatePostMutation(
  baseOptions?: Apollo.MutationHookOptions<CreatePostMutation, CreatePostMutationVariables>
) {
  const options = {...defaultOptions, ...baseOptions}
  return Apollo.useMutation<CreatePostMutation, CreatePostMutationVariables>(
    CreatePostDocument, 
    options
  );
}
```

---

#### **Using Generated Code**

**Query Example:**

```typescript
// src/components/UserProfile.tsx
import { useGetUserQuery } from '../generated/graphql';

function UserProfile({ userId }: { userId: string }) {
  // Fully typed hook - no manual types needed
  const { data, loading, error } = useGetUserQuery({
    variables: { id: userId },
  });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data?.user) return <div>User not found</div>;

  // TypeScript knows exact structure of data.user
  // Full autocomplete available ✨
  return (
    <div>
      <img src={data.user.avatar ?? '/default-avatar.png'} alt={data.user.name} />
      <h1>{data.user.name}</h1>
      <p>{data.user.email}</p>

      <h2>Posts</h2>
      {data.user.posts.map(post => (
        <article key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.content}</p>
          <span>{post.likes} likes</span>
          <time>{new Date(post.createdAt).toLocaleDateString()}</time>
        </article>
      ))}
    </div>
  );
}
```

**Mutation Example:**

```typescript
// src/components/CreatePostForm.tsx
import { useState } from 'react';
import { useCreatePostMutation } from '../generated/graphql';

function CreatePostForm() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  // Fully typed mutation hook
  const [createPost, { loading, error }] = useCreatePostMutation({
    onCompleted: (data) => {
      // data is fully typed
      console.log('Created post:', data.createPost.id);
      setTitle('');
      setContent('');
    },
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // TypeScript ensures correct variable types
    createPost({
      variables: {
        input: {
          title,
          content,
        },
      },
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={content}
        onChange={e => setContent(e.target.value)}
        placeholder="Content"
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
      {error && <div>Error: {error.message}</div>}
    </form>
  );
}
```

---

#### **Advanced Configuration**

**1. Multiple Output Files:**

```yaml
# codegen.yml
schema: http://localhost:4000/graphql
documents: 'src/**/*.graphql'
generates:
  # Generate types only
  src/generated/types.ts:
    plugins:
      - typescript
  
  # Generate operation types
  src/generated/operations.ts:
    plugins:
      - typescript-operations
  
  # Generate React hooks
  src/generated/hooks.tsx:
    plugins:
      - typescript-react-apollo
    config:
      withHooks: true
```

**2. Custom Scalars:**

```yaml
# codegen.yml
generates:
  src/generated/graphql.tsx:
    plugins:
      - typescript
      - typescript-operations
      - typescript-react-apollo
    config:
      scalars:
        DateTime: string          # Map DateTime to string
        JSON: any                 # Map JSON to any
        Upload: File              # Map Upload to File
        UUID: string              # Map UUID to string
```

**3. Fragment Masking (Type Safety):**

```graphql
# src/graphql/fragments.graphql

fragment UserBasic on User {
  id
  name
  avatar
}

fragment UserDetailed on User {
  ...UserBasic
  email
  bio
  createdAt
}
```

```typescript
// Generated types ensure you can't access fields not in fragment
import { UserBasicFragment, UserDetailedFragment } from '../generated/graphql';

function UserCard({ user }: { user: UserBasicFragment }) {
  // ✅ Can access: id, name, avatar
  // ❌ Cannot access: email, bio (not in fragment)
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      {/* user.email would be TypeScript error */}
    </div>
  );
}
```

**4. Introspection from File:**

```yaml
# codegen.yml - use schema file instead of endpoint
schema: ./schema.graphql
documents: 'src/**/*.graphql'
generates:
  src/generated/graphql.tsx:
    plugins:
      - typescript
      - typescript-operations
      - typescript-react-apollo
```

---

#### **Real-World Example: E-Commerce App**

**GraphQL Operations:**

```graphql
# src/graphql/product.graphql

query GetProducts($category: String, $limit: Int) {
  products(category: $category, limit: $limit) {
    id
    name
    description
    price
    images
    inStock
    rating
    reviewCount
  }
}

query GetProduct($id: ID!) {
  product(id: $id) {
    id
    name
    description
    price
    images
    inStock
    rating
    reviews {
      id
      author {
        id
        name
        avatar
      }
      rating
      text
      createdAt
    }
    relatedProducts {
      id
      name
      price
      images
    }
  }
}

mutation AddToCart($productId: ID!, $quantity: Int!) {
  addToCart(productId: $productId, quantity: $quantity) {
    id
    items {
      id
      product {
        id
        name
        price
      }
      quantity
    }
    total
  }
}

mutation Checkout($input: CheckoutInput!) {
  checkout(input: $input) {
    id
    status
    total
    items {
      product {
        name
      }
      quantity
    }
  }
}
```

**Using Generated Code:**

```typescript
// src/components/ProductList.tsx
import { useGetProductsQuery } from '../generated/graphql';

function ProductList({ category }: { category?: string }) {
  const { data, loading, error } = useGetProductsQuery({
    variables: { category, limit: 20 },
  });

  if (loading) return <div>Loading products...</div>;
  if (error) return <div>Error loading products</div>;

  return (
    <div className="product-grid">
      {data?.products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// src/components/ProductCard.tsx
import { useAddToCartMutation, GetProductsQuery } from '../generated/graphql';

type Product = GetProductsQuery['products'][0];

function ProductCard({ product }: { product: Product }) {
  const [addToCart, { loading }] = useAddToCartMutation({
    onCompleted: () => {
      alert('Added to cart!');
    },
  });

  return (
    <div className="product-card">
      <img src={product.images[0]} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <div>
        {'⭐'.repeat(Math.round(product.rating))} ({product.reviewCount} reviews)
      </div>
      <button
        onClick={() => addToCart({ variables: { productId: product.id, quantity: 1 } })}
        disabled={!product.inStock || loading}
      >
        {product.inStock ? 'Add to Cart' : 'Out of Stock'}
      </button>
    </div>
  );
}

// src/components/ProductDetail.tsx
import { useGetProductQuery } from '../generated/graphql';
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { id } = useParams<{ id: string }>();
  const { data, loading } = useGetProductQuery({
    variables: { id: id! },
    skip: !id,
  });

  if (loading) return <div>Loading...</div>;
  if (!data?.product) return <div>Product not found</div>;

  const { product } = data;

  return (
    <div className="product-detail">
      <div className="images">
        {product.images.map((img, i) => (
          <img key={i} src={img} alt={`${product.name} ${i + 1}`} />
        ))}
      </div>

      <div className="info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        <div className="rating">
          {'⭐'.repeat(Math.round(product.rating))} ({product.reviews.length} reviews)
        </div>
        <p>{product.description}</p>
        <button disabled={!product.inStock}>
          {product.inStock ? 'Add to Cart' : 'Out of Stock'}
        </button>
      </div>

      <div className="reviews">
        <h2>Reviews</h2>
        {product.reviews.map(review => (
          <div key={review.id} className="review">
            <div className="author">
              <img src={review.author.avatar} alt={review.author.name} />
              <strong>{review.author.name}</strong>
            </div>
            <div>{'⭐'.repeat(review.rating)}</div>
            <p>{review.text}</p>
            <time>{new Date(review.createdAt).toLocaleDateString()}</time>
          </div>
        ))}
      </div>

      <div className="related">
        <h2>Related Products</h2>
        <div className="grid">
          {product.relatedProducts.map(related => (
            <ProductCard key={related.id} product={related} />
          ))}
        </div>
      </div>
    </div>
  );
}
```

---

#### **Workflow Integration**

**1. Watch Mode for Development:**

```json
{
  "scripts": {
    "dev": "vite",
    "codegen": "graphql-codegen --config codegen.yml",
    "codegen:watch": "graphql-codegen --config codegen.yml --watch"
  }
}
```

Run in parallel:

```bash
# Terminal 1
npm run dev

# Terminal 2
npm run codegen:watch
```

**2. Pre-commit Hook:**

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run codegen && git add src/generated"
    }
  }
}
```

**3. CI/CD Integration:**

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install
      - run: npm run codegen  # Generate types
      - run: npm run type-check  # TypeScript check
      - run: npm test
```

---

#### **Other Code Generation Tools**

**1. GraphQL Code Generator (Most Popular):**

```bash
npm install --save-dev @graphql-codegen/cli
```

- Highly configurable
- Supports many plugins
- Works with Apollo, urql, Relay, React Query

**2. Apollo CLI:**

```bash
npm install --save-dev apollo
```

```json
{
  "scripts": {
    "codegen": "apollo client:codegen --target typescript"
  }
}
```

**3. Relay Compiler:**

```bash
npm install --save-dev relay-compiler
```

```json
{
  "scripts": {
    "relay": "relay-compiler"
  }
}
```

**4. GraphQL Zeus:**

```bash
npm install graphql-zeus
```

Generates a type-safe query builder.

---

#### **Benefits of Code Generation**

| Benefit | Description |
|---------|-------------|
| **Type Safety** | Catch errors at compile time, not runtime |
| **Autocomplete** | Full IDE autocomplete for all GraphQL operations |
| **Refactoring** | Rename fields in schema → TypeScript errors show what to fix |
| **Documentation** | Generated types serve as documentation |
| **Consistency** | Frontend always matches backend schema |
| **Less Boilerplate** | No manual type definitions needed |
| **Error Prevention** | Impossible to query fields that don't exist |
| **Onboarding** | New developers get instant type hints |

---

#### **Common Issues and Solutions**

**1. Stale Generated Files:**

```bash
# Always regenerate before starting development
npm run codegen
npm run dev
```

**2. Schema Changes Not Reflected:**

```bash
# Clear cache and regenerate
rm -rf src/generated
npm run codegen
```

**3. Circular Dependencies:**

```yaml
# Use separate output files
generates:
  src/generated/types.ts:
    plugins:
      - typescript
  src/generated/hooks.tsx:
    plugins:
      - typescript-react-apollo
```

**4. Large Generated Files:**

```yaml
# Split by feature
generates:
  src/generated/user.tsx:
    documents: 'src/graphql/user/*.graphql'
    plugins:
      - typescript-react-apollo
  src/generated/product.tsx:
    documents: 'src/graphql/product/*.graphql'
    plugins:
      - typescript-react-apollo
```

---

#### **Testing with Generated Types**

```typescript
// src/components/__tests__/ProductCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { MockedProvider } from '@apollo/client/testing';
import ProductCard from '../ProductCard';
import { AddToCartDocument, GetProductsQuery } from '../../generated/graphql';

// Use generated types for mock data
const mockProduct: GetProductsQuery['products'][0] = {
  __typename: 'Product',
  id: '1',
  name: 'Test Product',
  price: 29.99,
  images: ['/test-image.jpg'],
  inStock: true,
  rating: 4.5,
  reviewCount: 10,
  description: 'Test description',
};

const mocks = [
  {
    request: {
      query: AddToCartDocument,
      variables: { productId: '1', quantity: 1 },
    },
    result: {
      data: {
        addToCart: {
          __typename: 'Cart',
          id: 'cart-1',
          items: [],
          total: 29.99,
        },
      },
    },
  },
];

test('adds product to cart', async () => {
  render(
    <MockedProvider mocks={mocks}>
      <ProductCard product={mockProduct} />
    </MockedProvider>
  );

  const button = screen.getByText('Add to Cart');
  fireEvent.click(button);

  // Test implementation...
});
```

---

#### **Summary**

**GraphQL Code Generation:**

1. **Purpose**: Automatically generate TypeScript types and hooks from GraphQL schema
2. **Tool**: GraphQL Code Generator (`@graphql-codegen/cli`) most popular
3. **Setup**: Install → Configure codegen.yml → Write .graphql files → Run generator
4. **Output**: TypeScript types + React hooks (useQuery, useMutation)
5. **Benefits**: Type safety, autocomplete, consistency, less boilerplate
6. **Workflow**: Run in watch mode during development, integrate with CI/CD
7. **Result**: Fully typed GraphQL operations with zero manual maintenance

**Key Steps:**

1. **Install** GraphQL Code Generator and plugins
2. **Configure** codegen.yml with schema URL and documents path
3. **Write** GraphQL operations in .graphql files
4. **Run** `npm run codegen` to generate types
5. **Import** generated hooks and types
6. **Use** with full TypeScript type safety
7. **Regenerate** when schema or operations change

Code generation transforms GraphQL + TypeScript development from manual type maintenance into a **fully automated, type-safe workflow** where your frontend is always in sync with your backend.

</details>

---

### 230. How do you implement pagination with GraphQL?

<details>
<parameter name="summary">View Answer</summary>

**Pagination in GraphQL** allows you to load data in chunks rather than all at once, improving performance and user experience. GraphQL supports multiple pagination patterns, with **cursor-based pagination** (Relay-style) and **offset-based pagination** being the most common.

---

#### **Why Pagination?**

**Without Pagination:**

```typescript
// ❌ Loading all posts at once
query GetAllPosts {
  posts {
    id
    title
    content
  }
}

// Problems:
// - Slow initial load (thousands of records)
// - High memory usage
// - Poor user experience
// - Wasted bandwidth
```

**With Pagination:**

```typescript
// ✅ Load posts in chunks
query GetPosts($first: Int!, $after: String) {
  posts(first: $first, after: $after) {
    edges {
      node {
        id
        title
        content
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}

// Benefits:
// - Fast initial load
// - Lower memory usage
// - Better UX with infinite scroll
// - Load data as needed
```

---

#### **Pagination Strategies**

**1. Cursor-Based Pagination (Relay Specification)**

Most recommended for infinite scroll and real-time data.

**Schema Definition:**

```graphql
type Query {
  posts(first: Int, after: String, last: Int, before: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  createdAt: DateTime!
}
```

**2. Offset-Based Pagination**

Simpler but less efficient for large datasets.

```graphql
type Query {
  posts(limit: Int, offset: Int): PostsResult!
}

type PostsResult {
  posts: [Post!]!
  total: Int!
  hasMore: Boolean!
}
```

**3. Page-Based Pagination**

Traditional numbered pages.

```graphql
type Query {
  posts(page: Int, pageSize: Int): PostsPage!
}

type PostsPage {
  posts: [Post!]!
  page: Int!
  pageSize: Int!
  totalPages: Int!
  totalCount: Int!
}
```

---

#### **Cursor-Based Pagination with Apollo Client**

**Query Definition:**

```graphql
# src/graphql/queries.graphql
query GetPosts($first: Int!, $after: String) {
  posts(first: $first, after: $after) {
    edges {
      node {
        id
        title
        content
        author {
          id
          name
          avatar
        }
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

**Component Implementation:**

```typescript
// src/components/PostList.tsx
import { useQuery, gql } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($first: Int!, $after: String) {
    posts(first: $first, after: $after) {
      edges {
        node {
          id
          title
          content
          author {
            id
            name
            avatar
          }
          createdAt
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

function PostList() {
  const { data, loading, error, fetchMore } = useQuery(GET_POSTS, {
    variables: { first: 10 },  // Load first 10 posts
  });

  const loadMore = () => {
    fetchMore({
      variables: {
        after: data.posts.pageInfo.endCursor,  // Use last cursor
      },
    });
  };

  if (loading && !data) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  const posts = data.posts.edges.map(edge => edge.node);

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
          <div>
            <img src={post.author.avatar} alt={post.author.name} />
            <span>{post.author.name}</span>
          </div>
          <time>{new Date(post.createdAt).toLocaleDateString()}</time>
        </article>
      ))}

      {data.posts.pageInfo.hasNextPage && (
        <button onClick={loadMore} disabled={loading}>
          {loading ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

---

#### **Merge Function for Pagination**

**Configure Apollo Client to Merge Results:**

```typescript
// src/apollo/client.ts
import { ApolloClient, InMemoryCache } from '@apollo/client';
import { relayStylePagination } from '@apollo/client/utilities';

const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        posts: relayStylePagination(),  // Built-in Relay pagination
      },
    },
  },
});

// Or custom merge function
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        posts: {
          keyArgs: false,  // Don't cache by arguments
          
          merge(existing, incoming, { args }) {
            if (!existing) return incoming;

            // Merge edges
            const edges = existing.edges ? [...existing.edges] : [];
            const incomingEdges = incoming.edges || [];

            // Append new edges
            return {
              ...incoming,
              edges: [...edges, ...incomingEdges],
            };
          },
        },
      },
    },
  },
});

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache,
});
```

---

#### **Infinite Scroll Implementation**

**With Intersection Observer:**

```typescript
// src/components/InfinitePostList.tsx
import { useQuery, gql } from '@apollo/client';
import { useEffect, useRef } from 'react';

const GET_POSTS = gql`
  query GetPosts($first: Int!, $after: String) {
    posts(first: $first, after: $after) {
      edges {
        node {
          id
          title
          content
          author {
            id
            name
          }
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

function InfinitePostList() {
  const { data, loading, fetchMore } = useQuery(GET_POSTS, {
    variables: { first: 20 },
  });

  const loadMoreRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!loadMoreRef.current || loading) return;

    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && data?.posts.pageInfo.hasNextPage) {
          fetchMore({
            variables: {
              after: data.posts.pageInfo.endCursor,
            },
          });
        }
      },
      { threshold: 1.0 }
    );

    observer.observe(loadMoreRef.current);

    return () => observer.disconnect();
  }, [data, loading, fetchMore]);

  const posts = data?.posts.edges.map(edge => edge.node) || [];

  return (
    <div>
      {posts.map(post => (
        <article key={post.id} className="post">
          <h2>{post.title}</h2>
          <p>{post.content}</p>
          <span>by {post.author.name}</span>
        </article>
      ))}

      {data?.posts.pageInfo.hasNextPage && (
        <div ref={loadMoreRef} className="load-more">
          {loading ? 'Loading more...' : 'Scroll for more'}
        </div>
      )}

      {!data?.posts.pageInfo.hasNextPage && (
        <div className="end-message">No more posts</div>
      )}
    </div>
  );
}
```

---

#### **Offset-Based Pagination**

**Schema:**

```graphql
type Query {
  posts(limit: Int, offset: Int): PostsResult!
}

type PostsResult {
  posts: [Post!]!
  total: Int!
  hasMore: Boolean!
}
```

**Query:**

```graphql
query GetPosts($limit: Int!, $offset: Int!) {
  posts(limit: $limit, offset: $offset) {
    posts {
      id
      title
      content
    }
    total
    hasMore
  }
}
```

**Implementation:**

```typescript
// src/components/OffsetPostList.tsx
import { useQuery, gql } from '@apollo/client';
import { useState } from 'react';

const GET_POSTS = gql`
  query GetPosts($limit: Int!, $offset: Int!) {
    posts(limit: $limit, offset: $offset) {
      posts {
        id
        title
        content
      }
      total
      hasMore
    }
  }
`;

function OffsetPostList() {
  const [page, setPage] = useState(0);
  const limit = 10;

  const { data, loading, error } = useQuery(GET_POSTS, {
    variables: {
      limit,
      offset: page * limit,
    },
  });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  const posts = data.posts.posts;
  const totalPages = Math.ceil(data.posts.total / limit);

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}

      <div className="pagination">
        <button
          onClick={() => setPage(p => Math.max(0, p - 1))}
          disabled={page === 0}
        >
          Previous
        </button>

        <span>
          Page {page + 1} of {totalPages}
        </span>

        <button
          onClick={() => setPage(p => p + 1)}
          disabled={!data.posts.hasMore}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

**Cache Configuration for Offset Pagination:**

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        posts: {
          keyArgs: [],  // Don't cache by offset
          
          merge(existing, incoming, { args }) {
            // Replace existing data (don't append)
            return incoming;
          },
        },
      },
    },
  },
});
```

---

#### **Page-Based Pagination**

**Implementation with Page Numbers:**

```typescript
// src/components/PagedPostList.tsx
import { useQuery, gql } from '@apollo/client';
import { useState } from 'react';

const GET_POSTS = gql`
  query GetPosts($page: Int!, $pageSize: Int!) {
    posts(page: $page, pageSize: $pageSize) {
      posts {
        id
        title
        content
      }
      page
      pageSize
      totalPages
      totalCount
    }
  }
`;

function PagedPostList() {
  const [currentPage, setCurrentPage] = useState(1);
  const pageSize = 10;

  const { data, loading } = useQuery(GET_POSTS, {
    variables: { page: currentPage, pageSize },
  });

  if (loading && !data) return <div>Loading...</div>;

  const posts = data?.posts.posts || [];
  const totalPages = data?.posts.totalPages || 1;

  // Generate page numbers
  const pageNumbers = Array.from({ length: totalPages }, (_, i) => i + 1);

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}

      <div className="pagination">
        <button
          onClick={() => setCurrentPage(1)}
          disabled={currentPage === 1}
        >
          First
        </button>

        <button
          onClick={() => setCurrentPage(p => p - 1)}
          disabled={currentPage === 1}
        >
          Previous
        </button>

        {pageNumbers.map(num => (
          <button
            key={num}
            onClick={() => setCurrentPage(num)}
            className={currentPage === num ? 'active' : ''}
          >
            {num}
          </button>
        ))}

        <button
          onClick={() => setCurrentPage(p => p + 1)}
          disabled={currentPage === totalPages}
        >
          Next
        </button>

        <button
          onClick={() => setCurrentPage(totalPages)}
          disabled={currentPage === totalPages}
        >
          Last
        </button>
      </div>

      <div>
        Showing page {currentPage} of {totalPages} ({data?.posts.totalCount} total posts)
      </div>
    </div>
  );
}
```

---

#### **Bidirectional Pagination (Forward & Backward)**

**Query with Both Directions:**

```graphql
query GetPosts(
  $first: Int
  $after: String
  $last: Int
  $before: String
) {
  posts(first: $first, after: $after, last: $last, before: $before) {
    edges {
      node {
        id
        title
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
  }
}
```

**Implementation:**

```typescript
function BidirectionalPostList() {
  const { data, fetchMore } = useQuery(GET_POSTS, {
    variables: { first: 10 },
  });

  const loadNext = () => {
    fetchMore({
      variables: {
        first: 10,
        after: data.posts.pageInfo.endCursor,
      },
    });
  };

  const loadPrevious = () => {
    fetchMore({
      variables: {
        last: 10,
        before: data.posts.pageInfo.startCursor,
      },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        
        return {
          posts: {
            ...fetchMoreResult.posts,
            edges: [
              ...fetchMoreResult.posts.edges,
              ...prev.posts.edges,
            ],
          },
        };
      },
    });
  };

  return (
    <div>
      {data?.posts.pageInfo.hasPreviousPage && (
        <button onClick={loadPrevious}>Load Previous</button>
      )}

      {data?.posts.edges.map(edge => (
        <div key={edge.node.id}>{edge.node.title}</div>
      ))}

      {data?.posts.pageInfo.hasNextPage && (
        <button onClick={loadNext}>Load Next</button>
      )}
    </div>
  );
}
```

---

#### **Real-World Example: Social Media Feed**

**Complete Implementation:**

```typescript
// src/components/SocialFeed.tsx
import { useQuery, gql } from '@apollo/client';
import { useEffect, useRef, useState } from 'react';

const GET_FEED = gql`
  query GetFeed($first: Int!, $after: String) {
    feed(first: $first, after: $after) {
      edges {
        node {
          id
          content
          author {
            id
            name
            avatar
          }
          likes
          comments {
            id
          }
          createdAt
          isLikedByMe
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

interface Post {
  id: string;
  content: string;
  author: {
    id: string;
    name: string;
    avatar: string;
  };
  likes: number;
  comments: Array<{ id: string }>;
  createdAt: string;
  isLikedByMe: boolean;
}

function SocialFeed() {
  const [isLoadingMore, setIsLoadingMore] = useState(false);
  const observerRef = useRef<HTMLDivElement>(null);

  const { data, loading, error, fetchMore } = useQuery(GET_FEED, {
    variables: { first: 20 },
    notifyOnNetworkStatusChange: true,
  });

  // Infinite scroll
  useEffect(() => {
    if (!observerRef.current || loading || isLoadingMore) return;
    if (!data?.feed.pageInfo.hasNextPage) return;

    const observer = new IntersectionObserver(
      async (entries) => {
        if (entries[0].isIntersecting) {
          setIsLoadingMore(true);
          
          await fetchMore({
            variables: {
              after: data.feed.pageInfo.endCursor,
            },
          });
          
          setIsLoadingMore(false);
        }
      },
      { threshold: 0.5 }
    );

    observer.observe(observerRef.current);
    return () => observer.disconnect();
  }, [data, loading, isLoadingMore, fetchMore]);

  if (loading && !data) {
    return (
      <div className="feed-skeleton">
        {[...Array(3)].map((_, i) => (
          <div key={i} className="post-skeleton">
            <div className="skeleton-avatar" />
            <div className="skeleton-content" />
          </div>
        ))}
      </div>
    );
  }

  if (error) return <div>Error loading feed: {error.message}</div>;

  const posts: Post[] = data?.feed.edges.map(edge => edge.node) || [];

  return (
    <div className="social-feed">
      <header>
        <h1>Feed</h1>
      </header>

      <div className="posts">
        {posts.map(post => (
          <article key={post.id} className="post">
            <div className="post-header">
              <img
                src={post.author.avatar}
                alt={post.author.name}
                className="avatar"
              />
              <div>
                <strong>{post.author.name}</strong>
                <time>{formatTimeAgo(post.createdAt)}</time>
              </div>
            </div>

            <p className="post-content">{post.content}</p>

            <div className="post-actions">
              <button className={post.isLikedByMe ? 'liked' : ''}>
                {post.isLikedByMe ? '❤️' : '🤍'} {post.likes}
              </button>
              <button>
                💬 {post.comments.length}
              </button>
              <button>
                🔄 Share
              </button>
            </div>
          </article>
        ))}
      </div>

      {/* Loading indicator for infinite scroll */}
      {data?.feed.pageInfo.hasNextPage && (
        <div ref={observerRef} className="load-more-indicator">
          {isLoadingMore ? (
            <div className="spinner">Loading more posts...</div>
          ) : (
            <div>Scroll for more</div>
          )}
        </div>
      )}

      {/* End of feed message */}
      {!data?.feed.pageInfo.hasNextPage && posts.length > 0 && (
        <div className="end-of-feed">
          <p>You're all caught up! 🎉</p>
        </div>
      )}
    </div>
  );
}

function formatTimeAgo(dateString: string): string {
  const date = new Date(dateString);
  const now = new Date();
  const seconds = Math.floor((now.getTime() - date.getTime()) / 1000);

  if (seconds < 60) return 'just now';
  if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
  if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
  if (seconds < 604800) return `${Math.floor(seconds / 86400)}d ago`;
  
  return date.toLocaleDateString();
}

export default SocialFeed;
```

**Apollo Cache Configuration:**

```typescript
// src/apollo/client.ts
import { ApolloClient, InMemoryCache } from '@apollo/client';
import { relayStylePagination } from '@apollo/client/utilities';

const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        feed: relayStylePagination(),
      },
    },
  },
});

export const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache,
});
```

---

#### **Pagination with Filters**

**Query with Filters:**

```graphql
query GetPosts(
  $first: Int!
  $after: String
  $category: String
  $authorId: ID
  $search: String
) {
  posts(
    first: $first
    after: $after
    category: $category
    authorId: $authorId
    search: $search
  ) {
    edges {
      node {
        id
        title
        category
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

**Implementation with Filters:**

```typescript
function FilteredPostList() {
  const [category, setCategory] = useState<string>();
  const [search, setSearch] = useState('');

  const { data, fetchMore, refetch } = useQuery(GET_POSTS, {
    variables: {
      first: 20,
      category,
      search,
    },
  });

  // Refetch when filters change
  useEffect(() => {
    refetch({ first: 20, category, search, after: null });
  }, [category, search, refetch]);

  const loadMore = () => {
    fetchMore({
      variables: {
        after: data.posts.pageInfo.endCursor,
        category,
        search,
      },
    });
  };

  return (
    <div>
      <div className="filters">
        <select value={category} onChange={e => setCategory(e.target.value)}>
          <option value="">All Categories</option>
          <option value="tech">Tech</option>
          <option value="design">Design</option>
        </select>

        <input
          value={search}
          onChange={e => setSearch(e.target.value)}
          placeholder="Search..."
        />
      </div>

      {data?.posts.edges.map(edge => (
        <div key={edge.node.id}>{edge.node.title}</div>
      ))}

      {data?.posts.pageInfo.hasNextPage && (
        <button onClick={loadMore}>Load More</button>
      )}
    </div>
  );
}
```

**Cache Configuration with Filters:**

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        posts: {
          keyArgs: ['category', 'authorId', 'search'],  // Cache separately per filter
          
          merge(existing, incoming, { args }) {
            if (!existing) return incoming;
            if (!args?.after) return incoming;  // Reset if no cursor

            return {
              ...incoming,
              edges: [...(existing.edges || []), ...(incoming.edges || [])],
            };
          },
        },
      },
    },
  },
});
```

---

#### **Comparison of Pagination Strategies**

| Strategy | Best For | Pros | Cons |
|----------|----------|------|------|
| **Cursor-Based** | Infinite scroll, real-time feeds | Consistent results, handles insertions/deletions well, efficient | More complex implementation |
| **Offset-Based** | Simple lists, small datasets | Easy to implement, direct page access | Inefficient for large datasets, inconsistent with real-time data |
| **Page-Based** | Traditional pagination UI, reports | Familiar UX, total page count | Can miss items with insertions, inefficient |

---

#### **Best Practices**

**1. Use Cursor-Based for Dynamic Data:**

```typescript
// ✅ Good for social feeds, chat messages
query GetFeed($first: Int!, $after: String) {
  feed(first: $first, after: $after) {
    edges { ... }
    pageInfo { hasNextPage, endCursor }
  }
}
```

**2. Set Reasonable Page Sizes:**

```typescript
// ❌ Too small - many requests
const { data } = useQuery(GET_POSTS, { variables: { first: 5 } });

// ❌ Too large - slow load
const { data } = useQuery(GET_POSTS, { variables: { first: 1000 } });

// ✅ Balanced
const { data } = useQuery(GET_POSTS, { variables: { first: 20 } });
```

**3. Handle Loading States:**

```typescript
function PostList() {
  const { data, loading, fetchMore } = useQuery(GET_POSTS);

  if (loading && !data) return <Skeleton />;  // Initial load
  
  return (
    <>
      {data.posts.edges.map(edge => <Post key={edge.cursor} {...edge.node} />)}
      {loading && <div>Loading more...</div>}  // Pagination load
    </>
  );
}
```

**4. Implement Error Recovery:**

```typescript
const { data, error, fetchMore } = useQuery(GET_POSTS);

const loadMore = async () => {
  try {
    await fetchMore({ variables: { after: data.posts.pageInfo.endCursor } });
  } catch (err) {
    console.error('Failed to load more:', err);
    toast.error('Failed to load more posts');
  }
};
```

---

#### **Summary**

**GraphQL Pagination:**

1. **Purpose**: Load data in chunks for performance and UX
2. **Strategies**: Cursor-based (Relay), offset-based, page-based
3. **Recommended**: Cursor-based for most use cases
4. **Apollo Client**: Use `fetchMore` with merge functions
5. **Infinite Scroll**: Intersection Observer + automatic fetching
6. **Cache**: Configure `relayStylePagination()` or custom merge
7. **Filters**: Include filter args in `keyArgs` for separate caching

**Key Components:**

- **`edges`**: Array of results with cursors
- **`pageInfo`**: Metadata (hasNextPage, endCursor)
- **`fetchMore`**: Load additional results
- **Merge function**: Combine paginated results in cache

Pagination transforms GraphQL apps from loading everything upfront to **progressively loading data as needed**, creating fast, responsive user experiences that scale to millions of records.

</details>

