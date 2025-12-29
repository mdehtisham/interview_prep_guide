# NestJS GraphQL - Top Interview Questions

## GraphQL Fundamentals

1. What is GraphQL and how is it different from REST?

<details>
<summary><strong>Answer</strong></summary>

**GraphQL** is a **query language** for APIs and a **runtime** for executing queries against your data, developed by Facebook in 2012. Unlike REST which exposes multiple endpoints for different resources, GraphQL uses a **single endpoint** where clients specify exactly what data they need using a strongly-typed schema.

### **Key Differences: GraphQL vs REST**

| Feature | REST | GraphQL |
|---------|------|----------|
| **Endpoints** | Multiple endpoints (`/users`, `/posts`, `/comments`) | Single endpoint (`/graphql`) |
| **Data Fetching** | Server decides what data to return | Client specifies exact fields needed |
| **Over-fetching** | Common (returns more data than needed) | No over-fetching (client controls) |
| **Under-fetching** | Common (multiple requests needed) | No under-fetching (single request) |
| **Versioning** | URL versioning (`/v1/users`, `/v2/users`) | No versioning needed (schema evolution) |
| **Type System** | No built-in type system | Strongly typed schema (SDL) |
| **Documentation** | Manual (Swagger, OpenAPI) | Auto-generated from schema |
| **Caching** | HTTP caching (headers, CDN) | More complex (Apollo Cache) |
| **Real-time** | SSE, WebSockets separately | Built-in subscriptions |
| **Request Method** | GET, POST, PUT, DELETE, PATCH | POST only (queries in body) |

### **REST Example**

```typescript
// REST: Multiple endpoints, over-fetching

// GET /users/1
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "address": { ... },  // Not needed but returned
  "phone": "123-456",  // Not needed but returned
}

// GET /users/1/posts (separate request)
[
  {
    "id": 101,
    "title": "My Post",
    "content": "...",
    "author": { ... },  // Duplicate user data
    "comments": [ ... ] // Not needed but returned
  }
]

// Problems:
// 1. Over-fetching: Getting address, phone, comments we don't need
// 2. Under-fetching: Need 2 requests for user + posts
// 3. Duplicate data: User info repeated in posts
```

### **GraphQL Example**

```typescript
// GraphQL: Single request, exact data

query {
  user(id: 1) {
    name
    email
    posts {
      title
      content
    }
  }
}

// Response: Exactly what was requested
{
  "data": {
    "user": {
      "name": "John Doe",
      "email": "john@example.com",
      "posts": [
        {
          "title": "My Post",
          "content": "..."
        }
      ]
    }
  }
}

// Benefits:
// 1. No over-fetching: Only name, email, posts returned
// 2. No under-fetching: User + posts in single request
// 3. No duplicate data: User info not repeated
```

### **REST vs GraphQL: Real-World Scenario**

```typescript
// SCENARIO: Display user profile with recent posts and comments count

// ===== REST APPROACH =====
// Request 1: GET /users/1
const user = await fetch('/api/users/1');
// Returns: { id, name, email, age, address, phone, ... } // Over-fetching

// Request 2: GET /users/1/posts
const posts = await fetch('/api/users/1/posts');
// Returns: [{ id, title, content, body, author, tags, ... }] // Over-fetching

// Request 3: GET /posts/101/comments (for each post)
const comments1 = await fetch('/api/posts/101/comments');
const comments2 = await fetch('/api/posts/102/comments');
// Need N requests for N posts (N+1 problem)

// Total: 1 + 1 + N requests
// Over-fetching: Getting unused fields (age, address, body, tags)

// ===== GRAPHQL APPROACH =====
// Single request
const data = await graphql(`
  query {
    user(id: 1) {
      name
      email
      posts(limit: 5) {
        title
        commentsCount  # Aggregate field, no separate request
      }
    }
  }
`);

// Total: 1 request
// Exact data: Only name, email, title, commentsCount
// No N+1: commentsCount resolved efficiently on server
```

### **When to Use REST vs GraphQL**

**Use REST when:**
- Simple CRUD operations
- Well-defined resources with fixed structure
- HTTP caching is critical
- Team unfamiliar with GraphQL
- Simple microservices

**Use GraphQL when:**
- Complex data requirements
- Multiple clients (web, mobile) need different data
- Frequent schema changes
- Avoiding over-fetching/under-fetching
- Real-time updates (subscriptions)
- Frontend-driven development

### **GraphQL Core Concepts**

```graphql
# Schema Definition Language (SDL)
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users: [User!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String): User!
}

type Subscription {
  userCreated: User!
}
```

**Interview Tip**: **GraphQL** is a **query language** for APIs with a **single endpoint** where **clients specify exact data** needed, unlike REST which has **multiple endpoints** returning fixed data. **Key advantages**: (1) **No over-fetching** - client requests only needed fields (`user { name email }` vs REST returning all user fields), (2) **No under-fetching** - fetch related data in one request (`user { posts { title } }` vs REST requiring multiple requests), (3) **Strongly typed schema** - SDL defines types, auto-generates docs, (4) **No versioning** - evolve schema without breaking changes. **Trade-offs**: REST better for simple CRUD, HTTP caching; GraphQL better for complex data needs, multiple clients, avoiding multiple requests. **Example**: REST needs 3+ requests for user + posts + comments (over-fetches unused fields), GraphQL gets exact data in 1 request with query structure matching response.

</details>
2. What are the advantages of GraphQL (single endpoint, client-specified data, type safety)?

<details>
<summary><strong>Answer</strong></summary>

GraphQL's main advantages are: **(1) Single endpoint** - all queries go to `/graphql` vs REST's multiple endpoints; **(2) Client-specified data** - clients request exact fields needed, eliminating over-fetching/under-fetching; **(3) Strong type safety** - schema defines types with validation, auto-generated docs; **(4) Efficient data fetching** - nested queries get related data in one request; **(5) Schema evolution** - add fields without versioning; **(6) Real-time subscriptions** - built-in WebSocket support; **(7) Introspection** - query the schema itself for tooling.

### **1. Single Endpoint**

```typescript
// REST: Multiple endpoints for different resources
GET    /api/users
GET    /api/users/:id
POST   /api/users
PUT    /api/users/:id
DELETE /api/users/:id
GET    /api/posts
GET    /api/posts/:id
// ... hundreds of endpoints

// GraphQL: Single endpoint for everything
POST /graphql

// All operations through single endpoint:
query { users { id name } }              # Fetch users
query { user(id: 1) { name posts { title } } }  # Fetch user with posts
mutation { createUser(name: "John") { id } }    # Create user
mutation { deletePost(id: 5) }                  # Delete post
subscription { userCreated { name } }           # Real-time updates

// Benefits:
// - Simpler routing
// - Single entry point for security/logging
// - Gateway-friendly (single proxy target)
// - Easier rate limiting
```

### **2. Client-Specified Data (No Over-fetching/Under-fetching)**

```typescript
// ===== OVER-FETCHING (REST Problem) =====
// Mobile app only needs user name and avatar
// GET /api/users/1
{
  "id": 1,
  "name": "John",
  "email": "john@example.com",
  "avatar": "https://...",
  "bio": "Long biography text...",        // Not needed - wasted bandwidth
  "address": { ... },                      // Not needed
  "phone": "123-456-7890",                // Not needed
  "preferences": { ... },                  // Not needed
  "createdAt": "2023-01-01",              // Not needed
  "updatedAt": "2023-12-01"               // Not needed
}
// Mobile gets 10 KB when only needs 200 bytes

// GraphQL: Request exactly what's needed
query {
  user(id: 1) {
    name
    avatar
  }
}
// Response: Only 200 bytes
{
  "data": {
    "user": {
      "name": "John",
      "avatar": "https://..."
    }
  }
}

// ===== UNDER-FETCHING (REST Problem) =====
// Need user with their posts and each post's comments

// REST: Multiple requests (N+1 problem)
const user = await fetch('/api/users/1');           // Request 1
const posts = await fetch('/api/users/1/posts');    // Request 2
const comments1 = await fetch('/api/posts/1/comments'); // Request 3
const comments2 = await fetch('/api/posts/2/comments'); // Request 4
// Total: 1 + 1 + N requests (if N posts)

// GraphQL: Single request
query {
  user(id: 1) {
    name
    posts {
      title
      comments {
        content
        author { name }
      }
    }
  }
}
// Total: 1 request for all data
```

### **3. Strong Type Safety**

```typescript
// GraphQL Schema (SDL) - Strongly Typed
type User {
  id: ID!           # Required ID
  name: String!     # Required String
  email: String!    # Required String
  age: Int          # Optional Int
  posts: [Post!]!   # Required array of Posts (can be empty)
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!     # Required User relationship
}

// Type validation at compile time
query {
  user(id: "abc") {  # ❌ Error: ID expected, got String
    name
    age
  }
}

query {
  user(id: 1) {
    title  # ❌ Error: Field 'title' doesn't exist on User
  }
}

// Benefits:
// - Catch errors before runtime
// - IDE autocomplete for all fields
// - Auto-generated TypeScript types
// - Contract between frontend and backend
```

### **4. Efficient Nested Data Fetching**

```typescript
// Get user with posts, each post with comments, each comment with author

// REST: 1 + N + M requests (waterfall)
const user = await fetch('/api/users/1');              // 1 request
const posts = await fetch('/api/users/1/posts');       // 1 request
// For each post, fetch comments
for (const post of posts) {
  const comments = await fetch(`/api/posts/${post.id}/comments`); // N requests
  // For each comment, fetch author
  for (const comment of comments) {
    const author = await fetch(`/api/users/${comment.authorId}`); // M requests
  }
}
// Total: 1 + 1 + N + M requests (could be 100+ requests)
// Slow: Each request waits for previous to complete

// GraphQL: Single optimized request
query {
  user(id: 1) {
    name
    posts {
      title
      comments {
        content
        author {
          name
          avatar
        }
      }
    }
  }
}
// Total: 1 request
// Server resolves efficiently with DataLoader (batching)
// Response shape matches query structure
```

### **5. Schema Evolution Without Versioning**

```typescript
// REST: Breaking change requires versioning
// v1/users - returns { id, name, email }
// v2/users - returns { id, firstName, lastName, email }
// Now maintain two endpoints forever

GET /api/v1/users  # Old clients
GET /api/v2/users  # New clients

// GraphQL: Add fields without breaking changes
type User {
  id: ID!
  name: String!         # Old field (keep for backward compatibility)
  firstName: String!    # New field
  lastName: String!     # New field
  email: String!
}

# Old clients still work
query {
  user(id: 1) {
    name  # Still works
    email
  }
}

# New clients use new fields
query {
  user(id: 1) {
    firstName  # New field
    lastName   # New field
    email
  }
}

// Deprecation: Mark old fields
type User {
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}

// No versioning needed - single evolving schema
```

### **6. Real-time Subscriptions (Built-in)**

```typescript
// REST: Need separate WebSocket server
const ws = new WebSocket('ws://localhost:3001');
ws.onmessage = (event) => {
  // Custom protocol, manual parsing
};

// GraphQL: Built-in subscriptions
subscription {
  messageCreated(roomId: "room-1") {
    id
    content
    author {
      name
      avatar
    }
    createdAt
  }
}

// Client code (Apollo Client)
const { data } = useSubscription(MESSAGE_CREATED, {
  variables: { roomId: 'room-1' },
});

// Benefits:
// - Unified protocol (GraphQL over WebSocket)
// - Type-safe subscriptions
// - Same schema for queries/mutations/subscriptions
// - Automatic reconnection, error handling
```

### **7. Introspection & Auto-generated Docs**

```typescript
// Query the schema itself
query {
  __schema {
    types {
      name
      fields {
        name
        type {
          name
        }
      }
    }
  }
}

// Get available queries
query {
  __schema {
    queryType {
      fields {
        name
        description
        args {
          name
          type {
            name
          }
        }
      }
    }
  }
}

// Benefits:
// - GraphQL Playground auto-generates docs
// - IDE autocomplete (IntelliSense)
// - Auto-generate client code
// - Schema validation tools
// - No manual Swagger/OpenAPI maintenance
```

### **8. Better Developer Experience**

```typescript
// Frontend developer wants to add a new field

// REST Approach:
// 1. Ask backend to add field to endpoint
// 2. Wait for backend deployment
// 3. Update frontend code
// 4. If needs different structure, create new endpoint (/api/v2)

// GraphQL Approach:
// 1. Add field to schema (backend)
type User {
  id: ID!
  name: String!
  bio: String!  # New field
}

// 2. Frontend immediately uses it (no endpoint changes)
query {
  user(id: 1) {
    name
    bio  # Just add to query
  }
}

// 3. Multiple clients get different data without new endpoints
// Mobile: only id, name
query { user(id: 1) { id name } }

// Web: id, name, bio, posts
query { user(id: 1) { id name bio posts { title } } }

// Admin dashboard: everything
query { user(id: 1) { id name email bio posts { id title content } } }

// Same endpoint, different data per client
```

### **9. Aggregation & Computed Fields**

```typescript
// REST: Need separate endpoints or complex query params
GET /api/users/1              # User info
GET /api/users/1/posts/count  # Post count
GET /api/users/1/stats        # Various stats

// GraphQL: Computed fields in schema
type User {
  id: ID!
  name: String!
  posts: [Post!]!
  postsCount: Int!        # Computed field
  commentsCount: Int!     # Computed field
  isActive: Boolean!      # Computed field (last login < 7 days)
}

query {
  user(id: 1) {
    name
    postsCount     # No separate request
    commentsCount  # No separate request
    isActive       # No separate request
  }
}

// Server resolves efficiently
@ResolveField(() => Int)
async postsCount(@Parent() user: User) {
  return this.postService.countByUserId(user.id);
}
```

### **Production Benefits Summary**

```typescript
// Mobile App (Bandwidth Critical)
// GraphQL: Request minimal data
query {
  user(id: 1) {
    name
    avatar  # Only 200 bytes
  }
}
// REST: Would get entire user object (10 KB)

// Web Dashboard (Rich UI)
// GraphQL: Get everything in one request
query {
  dashboard {
    user { name email role }
    stats { views clicks conversions }
    recentPosts { title views comments { count } }
    notifications { message read createdAt }
  }
}
// REST: Would need 5+ requests

// Microservices (Gateway Pattern)
// GraphQL: Single gateway federates multiple services
// - User service: user { id name }
// - Post service: post { id title }
// - Comment service: comment { id content }
// Gateway stitches schemas together
```

**Interview Tip**: GraphQL's **key advantages**: (1) **Single endpoint** (`/graphql`) vs REST's multiple endpoints - simplifies routing, security, rate limiting; (2) **Client-specified data** - request exact fields (`user { name avatar }`), eliminates **over-fetching** (REST returns all fields, waste bandwidth) and **under-fetching** (REST needs multiple requests, GraphQL gets related data in one query); (3) **Strong type safety** - schema defines types (`String!`, `Int`, `[Post!]!`), compile-time validation, auto-generated TypeScript types; (4) **Efficient nested fetching** - get user + posts + comments in 1 request vs REST's N+1 requests; (5) **No versioning** - add fields without breaking (`firstName`/`lastName` alongside deprecated `name`); (6) **Built-in subscriptions** - real-time WebSocket updates; (7) **Introspection** - query schema itself, auto-docs in Playground, IDE autocomplete. **Production benefits**: mobile saves bandwidth (request only needed fields), web gets complex data in one request, microservices use GraphQL gateway for service federation. **Trade-off**: REST better for simple CRUD, HTTP caching; GraphQL better for complex data needs, multiple clients with different requirements.

</details>
3. What are Queries, Mutations, and Subscriptions?

<details>
<summary><strong>Answer</strong></summary>

**Queries**, **Mutations**, and **Subscriptions** are the three root operation types in GraphQL: **(1) Queries** - read/fetch data (like GET in REST), always return data, no side effects; **(2) Mutations** - write/modify data (like POST/PUT/DELETE in REST), create/update/delete operations, have side effects; **(3) Subscriptions** - real-time data updates via WebSockets, client subscribes to events, server pushes data when events occur. All three use the same **strongly-typed schema** and **SDL (Schema Definition Language)**.

### **Schema Components**

```graphql
# ===== SCALAR TYPES (Built-in primitives) =====
ID      # Unique identifier (string, but serialized as ID)
String  # UTF-8 character sequence
Int     # 32-bit integer
Float   # Floating-point number
Boolean # true or false

# ===== OBJECT TYPES (Custom types) =====
type User {
  id: ID!              # Required ID field
  name: String!        # Required String field
  email: String!       # Required String field
  age: Int             # Optional Int field (nullable)
  isActive: Boolean!   # Required Boolean field
  posts: [Post!]!      # Required array of Posts (can be empty, but not null)
  createdAt: String!   # Could use custom DateTime scalar
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!        # Relationship: Post belongs to User
  comments: [Comment!]! # Relationship: Post has many Comments
  likes: Int!
}

type Comment {
  id: ID!
  content: String!
  author: User!        # Relationship: Comment belongs to User
  post: Post!          # Relationship: Comment belongs to Post
}

# ===== INPUT TYPES (For mutations) =====
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  name: String         # All fields optional for updates
  email: String
}

input CreatePostInput {
  title: String!
  content: String!
  authorId: ID!
}

# ===== ENUM TYPES (Fixed set of values) =====
enum Role {
  ADMIN
  MODERATOR
  USER
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# ===== ROOT TYPES (Entry points) =====
type Query {
  # User queries
  user(id: ID!): User
  users(role: Role): [User!]!
  
  # Post queries
  post(id: ID!): Post
  posts(status: PostStatus, limit: Int): [Post!]!
}

type Mutation {
  # User mutations
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  
  # Post mutations
  createPost(input: CreatePostInput!): Post!
  publishPost(id: ID!): Post!
}

type Subscription {
  # Real-time subscriptions
  userCreated: User!
  postPublished: Post!
  commentAdded(postId: ID!): Comment!
}

# ===== INTERFACE (Shared fields) =====
interface Node {
  id: ID!
  createdAt: String!
}

type User implements Node {
  id: ID!
  createdAt: String!
  name: String!
  # ... other User fields
}

type Post implements Node {
  id: ID!
  createdAt: String!
  title: String!
  # ... other Post fields
}

# ===== UNION TYPES (One of multiple types) =====
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}
```

### **Type Modifiers (Nullability)**

```graphql
# Field type syntax
type User {
  # Required field (cannot be null)
  name: String!
  
  # Optional field (can be null)
  age: Int
  
  # Required array, can be empty, items required
  posts: [Post!]!      # ✅ []
                       # ✅ [Post, Post]
                       # ❌ null
                       # ❌ [null, Post]
  
  # Optional array, items required
  posts: [Post!]       # ✅ null
                       # ✅ []
                       # ✅ [Post, Post]
                       # ❌ [null, Post]
  
  # Required array, items optional
  posts: [Post]!       # ✅ []
                       # ✅ [Post, null, Post]
                       # ❌ null
  
  # Optional array, items optional
  posts: [Post]        # ✅ null
                       # ✅ []
                       # ✅ [Post, null, Post]
}
```

### **NestJS Code-First Schema**

```typescript
// ===== OBJECT TYPE =====
// user.model.ts
import { ObjectType, Field, ID, Int } from '@nestjs/graphql';
import { Post } from './post.model';

@ObjectType({ description: 'User object type' })
export class User {
  @Field(() => ID, { description: 'User unique identifier' })
  id: string;

  @Field({ description: 'User full name' })
  name: string;

  @Field()
  email: string;

  @Field(() => Int, { nullable: true, description: 'User age' })
  age?: number;

  @Field({ defaultValue: true })
  isActive: boolean;

  @Field(() => [Post], { description: 'User posts' })
  posts: Post[];

  @Field()
  createdAt: Date;
}

// post.model.ts
@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field()
  content: string;

  @Field()
  published: boolean;

  @Field(() => User, { description: 'Post author' })
  author: User;

  @Field(() => [Comment])
  comments: Comment[];

  @Field(() => Int)
  likes: number;
}

// ===== INPUT TYPE =====
// user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, MinLength, IsOptional } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;
}

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;
}

// ===== ENUM TYPE =====
// role.enum.ts
import { registerEnumType } from '@nestjs/graphql';

export enum Role {
  ADMIN = 'ADMIN',
  MODERATOR = 'MODERATOR',
  USER = 'USER',
}

registerEnumType(Role, {
  name: 'Role',
  description: 'User role',
});

// ===== ROOT TYPES (Resolvers) =====
// user.resolver.ts
import { Resolver, Query, Mutation, Args, ID } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  // Query: Defines schema.query.user field
  @Query(() => User, { nullable: true })
  async user(@Args('id', { type: () => ID }) id: string): Promise<User> {
    return this.userService.findById(id);
  }

  // Query: Defines schema.query.users field
  @Query(() => [User])
  async users(
    @Args('role', { type: () => Role, nullable: true }) role?: Role,
  ): Promise<User[]> {
    return this.userService.findAll(role);
  }

  // Mutation: Defines schema.mutation.createUser field
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }

  // Mutation: Defines schema.mutation.deleteUser field
  @Mutation(() => Boolean)
  async deleteUser(
    @Args('id', { type: () => ID }) id: string,
  ): Promise<boolean> {
    await this.userService.delete(id);
    return true;
  }
}
```

### **Generated Schema (from Code-First)**

```graphql
# NestJS auto-generates this schema.gql from decorators

type User {
  """User unique identifier"""
  id: ID!
  
  """User full name"""
  name: String!
  
  email: String!
  
  """User age"""
  age: Int
  
  isActive: Boolean!
  
  """User posts"""
  posts: [Post!]!
  
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!
  comments: [Comment!]!
  likes: Int!
}

input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  name: String
  email: String
}

enum Role {
  ADMIN
  MODERATOR
  USER
}

type Query {
  user(id: ID!): User
  users(role: Role): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}
```

### **Schema Directives**

```graphql
# Deprecation
type User {
  name: String! @deprecated(reason: "Use firstName and lastName instead")
  firstName: String!
  lastName: String!
}

# Custom directives
directive @auth(requires: Role = USER) on FIELD_DEFINITION

type Query {
  users: [User!]! @auth(requires: ADMIN)
  me: User! @auth
}

# NestJS implementation
import { Directive } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field()
  @Directive('@deprecated(reason: "Use firstName and lastName")')
  name: string;
  
  @Field()
  firstName: string;
  
  @Field()
  lastName: string;
}
```

### **Schema Introspection**

```graphql
# Query the schema itself
query IntrospectionQuery {
  __schema {
    types {
      name
      kind
      description
      fields {
        name
        type {
          name
          kind
        }
      }
    }
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
  }
}

# Get type information
query {
  __type(name: "User") {
    name
    kind
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

### **Schema Benefits**

```typescript
// 1. AUTO-GENERATED DOCUMENTATION
// GraphQL Playground automatically shows:
// - All available types
// - All fields and their types
// - Field descriptions
// - Required vs optional fields
// No manual Swagger/OpenAPI needed

// 2. TYPE SAFETY
// Invalid queries caught at compile-time:
query {
  user(id: "abc") {
    invalidField  # ❌ Error: Field doesn't exist
  }
}

query {
  user(id: 123) {  # ❌ Error: Expected String, got Int
    name
  }
}

// 3. IDE AUTOCOMPLETE
// IDE reads schema, provides IntelliSense:
query {
  user(id: "1") {
    name   # ✅ Autocomplete suggests: name, email, age, posts
  }
}

// 4. CLIENT CODE GENERATION
// Tools like GraphQL Code Generator create TypeScript types:
// user.query.ts
export interface User {
  id: string;
  name: string;
  email: string;
  posts: Post[];
}

export interface GetUserQuery {
  user: User;
}

// Type-safe client queries
const { data } = useQuery<GetUserQuery>(GET_USER);
const userName = data.user.name; // TypeScript knows this exists
```

**Interview Tip**: **GraphQL Schema** is the **contract** between client and server, defining: (1) **Object types** - entities like `type User { id: ID! name: String! }`, (2) **Fields** - properties with types (scalars: `String`, `Int`, `Boolean`, `ID`, `Float`), (3) **Relationships** - `User` has `posts: [Post!]!`, (4) **Operations** - root types (`Query`, `Mutation`, `Subscription`) define available operations, (5) **Input types** - for mutations (`input CreateUserInput { name: String! }`), (6) **Enums** - fixed values (`enum Role { ADMIN USER }`). **Nullability**: `String!` = required, `String` = optional, `[Post!]!` = required non-empty array, `[Post]` = optional array. Written in **SDL (Schema Definition Language)** or **Code-First** (TypeScript decorators `@ObjectType()`, `@Field()`). **Benefits**: (1) auto-generated docs in Playground, (2) compile-time type validation (invalid fields/types rejected), (3) IDE autocomplete, (4) single source of truth. **NestJS**: use `@ObjectType()` for types, `@InputType()` for inputs, `@Field()` for fields, `registerEnumType()` for enums; schema auto-generated from decorators to `schema.gql`. Schema enables **introspection** (`__schema`, `__type` queries) for tooling.

</details>

## NestJS GraphQL Setup

5. What are the two approaches in NestJS: Code First vs Schema First?

<details>
<summary><strong>Answer</strong></summary>

NestJS provides **two approaches** for building GraphQL APIs: **(1) Code First** - define schema using **TypeScript decorators** (`@ObjectType()`, `@Field()`), NestJS auto-generates `.graphql` schema file from TypeScript classes; **(2) Schema First** - manually write `.graphql` schema file in **SDL (Schema Definition Language)**, NestJS auto-generates TypeScript types from schema. **Code First** is preferred for TypeScript-heavy teams (type safety, single source of truth in TS), while **Schema First** is preferred when schema design comes first or working with non-TS clients.

### **Code First Approach**

```typescript
// ===== DEFINE TYPES USING TYPESCRIPT DECORATORS =====

// user.model.ts
import { ObjectType, Field, ID, Int } from '@nestjs/graphql';

@ObjectType()  // Marks class as GraphQL type
export class User {
  @Field(() => ID)  // Define GraphQL field
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;

  @Field(() => Int, { nullable: true })
  age?: number;

  @Field(() => [Post])  // Relationship
  posts: Post[];
}

@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field()
  content: string;

  @Field(() => User)
  author: User;
}

// user.input.ts
import { InputType, Field } from '@nestjs/graphql';

@InputType()  // For mutations
export class CreateUserInput {
  @Field()
  name: string;

  @Field()
  email: string;
}

// user.resolver.ts
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User> {
    // Implementation
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    // Implementation
  }
}

// ===== AUTO-GENERATED SCHEMA (schema.gql) =====
# NestJS automatically generates this file:

type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

input CreateUserInput {
  name: String!
  email: String!
}

type Query {
  user(id: String!): User
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

### **Schema First Approach**

```graphql
# ===== MANUALLY WRITE SCHEMA (schema.graphql) =====

type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

input CreateUserInput {
  name: String!
  email: String!
}

type Query {
  user(id: ID!): User
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

```typescript
// ===== AUTO-GENERATED TYPESCRIPT TYPES (graphql.ts) =====
// NestJS generates these from schema:

export interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
  posts: Post[];
}

export interface Post {
  id: string;
  title: string;
  content: string;
  author: User;
}

export interface CreateUserInput {
  name: string;
  email: string;
}

export interface Query {
  user(id: string): User | null;
}

export interface Mutation {
  createUser(input: CreateUserInput): User;
}

// ===== IMPLEMENT RESOLVER USING GENERATED TYPES =====
// user.resolver.ts
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { User, CreateUserInput } from './graphql';

@Resolver('User')  // String-based
export class UserResolver {
  @Query('user')  // Matches schema Query.user
  async user(@Args('id') id: string): Promise<User> {
    // Implementation
  }

  @Mutation('createUser')  // Matches schema Mutation.createUser
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    // Implementation
  }
}
```

### **Module Configuration**

```typescript
// app.module.ts

// ===== CODE FIRST CONFIGURATION =====
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),  // Auto-generate schema
      // OR: autoSchemaFile: true,  // In-memory schema (no file)
      sortSchema: true,  // Sort schema alphabetically
    }),
  ],
})
export class AppModule {}

// ===== SCHEMA FIRST CONFIGURATION =====
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      typePaths: ['./**/*.graphql'],  // Load .graphql files
      definitions: {
        path: join(process.cwd(), 'src/graphql.ts'),  // Generate TS types here
        outputAs: 'class',  // Generate as classes (or 'interface')
      },
    }),
  ],
})
export class AppModule {}
```

### **Comparison Table**

| Aspect | Code First | Schema First |
|--------|------------|-------------|
| **Schema Source** | TypeScript classes with decorators | `.graphql` files (SDL) |
| **Generated** | Auto-generates `.gql` schema | Auto-generates TS types |
| **Type Safety** | Full TypeScript type safety | Generated types (less safe) |
| **Single Source of Truth** | TypeScript code | GraphQL schema |
| **Decorator Usage** | Heavy (`@ObjectType()`, `@Field()`) | Minimal (`@Resolver()`, `@Query()`) |
| **Schema Design** | Follows code structure | Independent schema design |
| **Learning Curve** | Steeper (decorators) | Easier (familiar SDL) |
| **Flexibility** | Limited by TS constraints | Full GraphQL flexibility |
| **IDE Support** | Excellent (TypeScript) | Good (GraphQL plugins) |
| **Team Preference** | TypeScript-heavy teams | Schema-first designers |
| **Refactoring** | Easier (TS refactoring tools) | Manual (schema + code) |
| **Documentation** | From TS comments | From schema descriptions |

### **When to Use Code First**

```typescript
// ✅ Use Code First when:
// 1. TypeScript-first team
// 2. Want type safety everywhere
// 3. Prefer code-driven development
// 4. Using TypeORM/Prisma entities as GraphQL types
// 5. Want automatic schema generation
// 6. Single source of truth in TypeScript

// Example: TypeORM entity = GraphQL type
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { ObjectType, Field, ID } from '@nestjs/graphql';

@Entity()
@ObjectType()  // Dual purpose: DB entity + GraphQL type
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)
  id: string;

  @Column()
  @Field()
  name: string;

  @Column({ unique: true })
  @Field()
  email: string;
}
// Single class serves both TypeORM and GraphQL
```

### **When to Use Schema First**

```typescript
// ✅ Use Schema First when:
// 1. Schema design comes first (contract-driven)
// 2. Non-TypeScript clients need schema
// 3. Team familiar with GraphQL SDL
// 4. Want full GraphQL flexibility
// 5. Schema shared across multiple services
// 6. GraphQL schema is the contract

// Example: Multiple services use same schema
// schema.graphql (shared)
type User {
  id: ID!
  name: String!
  email: String!
}

// user-service (NestJS) implements User
// auth-service (Go) implements User
// mobile-app reads schema for code generation
```

### **Code First Workflow**

```typescript
// 1. Write TypeScript classes with decorators
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;
}

// 2. NestJS auto-generates schema.gql
// type User { id: ID! }

// 3. Schema appears in GraphQL Playground
// 4. Frontend queries the schema
```

### **Schema First Workflow**

```graphql
# 1. Write schema.graphql manually
type User {
  id: ID!
  name: String!
}
```

```typescript
// 2. NestJS generates graphql.ts
export interface User {
  id: string;
  name: string;
}

// 3. Implement resolver using generated types
@Resolver('User')
export class UserResolver {
  @Query('user')
  async user(): Promise<User> {
    return { id: '1', name: 'John' };
  }
}
```

### **Hybrid Approach (Possible)**

```typescript
// Some teams use Code First for most types
// But manually write complex SDL for specific features

// app.module.ts
GraphQLModule.forRoot({
  autoSchemaFile: true,  // Code First
  include: [UserModule, PostModule],  // These use Code First
  
  // Merge with manual schema for complex types
  typeDefs: `
    type ComplexQuery {
      search(query: String!): [SearchResult!]!
    }
    
    union SearchResult = User | Post | Comment
  `,
}),
```

**Interview Tip**: NestJS offers **two GraphQL approaches**: (1) **Code First** - define schema using **TypeScript decorators** (`@ObjectType()`, `@Field()`), NestJS **auto-generates** `schema.gql` from classes, single source of truth in TypeScript, full type safety, ideal for TypeScript teams; (2) **Schema First** - manually write `schema.graphql` in **SDL**, NestJS **auto-generates** TypeScript interfaces from schema, schema is source of truth, ideal for schema-first design. **Key difference**: Code First = TS → Schema, Schema First = Schema → TS. **Configuration**: Code First uses `autoSchemaFile: 'path/to/schema.gql'`, Schema First uses `typePaths: ['./**/*.graphql']` + `definitions: { path: 'graphql.ts' }`. **Choose Code First** for: TypeScript teams, type safety, single TS source, TypeORM entities as GraphQL types. **Choose Schema First** for: contract-first, GraphQL SDL expertise, multi-language services, schema as contract. **Trade-offs**: Code First (more decorators, less flexible) vs Schema First (manual schema maintenance, less type-safe).

</details>

<details>
<summary><strong>Answer</strong></summary>

**Code First** vs **Schema First** differences: **(1) Source of truth** - Code First uses TypeScript classes, Schema First uses `.graphql` files; **(2) Generation direction** - Code First generates schema from TS (TS → Schema), Schema First generates TS from schema (Schema → TS); **(3) Type safety** - Code First has native TS type safety, Schema First uses generated interfaces (less safe); **(4) Decorators** - Code First uses many decorators (`@ObjectType()`, `@Field()`), Schema First minimal (`@Resolver()`, `@Query()`); **(5) Workflow** - Code First writes TS classes first, Schema First writes SDL schema first; **(6) Flexibility** - Code First constrained by TypeScript, Schema First has full GraphQL flexibility.

### **Key Differences Table**

| Feature | Code First | Schema First |
|---------|------------|-------------|
| **Definition** | TypeScript classes with decorators | `.graphql` files with SDL |
| **Source of Truth** | TypeScript code | GraphQL schema |
| **Schema File** | Auto-generated (`schema.gql`) | Manually written |
| **TypeScript Types** | Native (classes) | Auto-generated (interfaces) |
| **Type Safety** | Full native TypeScript | Generated (potential mismatch) |
| **Decorators** | Heavy (`@ObjectType`, `@Field`) | Light (`@Resolver`, `@Query`) |
| **Workflow** | Write TS → Schema generated | Write Schema → TS generated |
| **Schema Design** | Follows code structure | Independent design |
| **Flexibility** | Limited by TypeScript | Full GraphQL features |
| **Learning Curve** | Steeper (decorator syntax) | Easier (GraphQL SDL) |
| **Refactoring** | Easy (TypeScript tools) | Manual (schema + code) |
| **IDE Support** | Full TypeScript IntelliSense | GraphQL plugin required |
| **Documentation** | TS comments → descriptions | Schema descriptions |
| **Validation** | Compile-time (TypeScript) | Runtime (GraphQL validation) |
| **Team Fit** | TypeScript-heavy teams | Schema designers, multi-lang |
| **Entity Reuse** | Easy (TypeORM entities) | Separate entities + schema |
| **Schema Sharing** | Harder (TS-specific) | Easy (`.graphql` standard) |
| **Migration** | TS changes → schema updates | Schema changes → TS regen |

### **Example: Same Type in Both Approaches**

**Code First:**
```typescript
// user.model.ts
import { ObjectType, Field, ID, Int } from '@nestjs/graphql';

@ObjectType({ description: 'User account' })
export class User {
  @Field(() => ID, { description: 'Unique identifier' })
  id: string;

  @Field({ description: 'Full name' })
  name: string;

  @Field()
  email: string;

  @Field(() => Int, { nullable: true })
  age?: number;

  @Field(() => [Post], { description: 'User posts' })
  posts: Post[];
}

// Generates schema.gql:
# """
# User account
# """
# type User {
#   """Unique identifier"""
#   id: ID!
#   
#   """Full name"""
#   name: String!
#   
#   email: String!
#   age: Int
#   
#   """User posts"""
#   posts: [Post!]!
# }
```

**Schema First:**
```graphql
# schema.graphql
"""
User account
"""
type User {
  "Unique identifier"
  id: ID!
  
  "Full name"
  name: String!
  
  email: String!
  age: Int
  
  "User posts"
  posts: [Post!]!
}
```

```typescript
// Generates graphql.ts:
export interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
  posts: Post[];
}
```

### **Type Safety Comparison**

**Code First: Full Native Type Safety**
```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;  // TypeScript knows this is string

  @Field()
  name: string;

  @Field(() => Int)
  age: number;  // TypeScript enforces number
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(): Promise<User> {
    return {
      id: '123',
      name: 'John',
      age: 30,  // ✅ TypeScript validates
      // age: '30',  // ❌ TypeScript error: Type 'string' not assignable
    };
  }
}
// Compile-time type checking - errors caught immediately
```

**Schema First: Generated Type Safety**
```graphql
# schema.graphql
type User {
  id: ID!
  name: String!
  age: Int!
}
```

```typescript
// Generated graphql.ts
export interface User {
  id: string;
  name: string;
  age: number;
}

// user.resolver.ts
@Resolver('User')
export class UserResolver {
  @Query('user')
  async user(): Promise<User> {
    return {
      id: '123',
      name: 'John',
      age: 30,  // ✅ Interface validates
      // age: '30',  // ❌ TypeScript error
    };
  }
}
// Type safety via generated interfaces
// Risk: Interface might not match schema if generation fails
```

### **Resolver Implementation Differences**

**Code First Resolver:**
```typescript
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';  // Import class
import { CreateUserInput } from './user.input';

@Resolver(() => User)  // Pass class reference
export class UserResolver {
  @Query(() => User, { name: 'user', nullable: true })  // Return type as class
  async getUser(@Args('id') id: string): Promise<User> {
    return this.userService.findById(id);
  }

  @Mutation(() => User)  // Return type as class
  async createUser(
    @Args('input') input: CreateUserInput,  // Input type as class
  ): Promise<User> {
    return this.userService.create(input);
  }
}
// Type-safe: User and CreateUserInput are actual classes
```

**Schema First Resolver:**
```typescript
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { User, CreateUserInput } from './graphql';  // Import generated interfaces

@Resolver('User')  // String reference
export class UserResolver {
  @Query('user')  // String reference to schema Query.user
  async getUser(@Args('id') id: string): Promise<User> {
    return this.userService.findById(id);
  }

  @Mutation('createUser')  // String reference to schema Mutation.createUser
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }
}
// Less type-safe: String references, interfaces (not classes)
```

### **Configuration Differences**

**Code First Configuration:**
```typescript
// app.module.ts
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      
      // OPTION 1: Generate schema file
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      
      // OPTION 2: In-memory schema (no file)
      // autoSchemaFile: true,
      
      // Optional: Sort schema fields alphabetically
      sortSchema: true,
      
      // Optional: Build schema from specific modules
      include: [UserModule, PostModule],
    }),
  ],
})
export class AppModule {}

// No manual schema file needed
// Schema auto-generated on server start
```

**Schema First Configuration:**
```typescript
// app.module.ts
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      
      // Load schema files
      typePaths: ['./**/*.graphql'],  // Or specific: ['./schema.graphql']
      
      // Generate TypeScript definitions
      definitions: {
        path: join(process.cwd(), 'src/graphql.ts'),
        outputAs: 'class',  // or 'interface'
        watch: true,  // Auto-regenerate on schema changes
      },
    }),
  ],
})
export class AppModule {}

// Must have schema.graphql files
// TypeScript types auto-generated on server start
```

### **Workflow Comparison**

**Code First Workflow:**
```typescript
// Step 1: Write TypeScript classes
@ObjectType()
export class User {
  @Field(() => ID) id: string;
  @Field() name: string;
}

// Step 2: Write resolver
@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(): Promise<User> { /* ... */ }
}

// Step 3: Start server
// → schema.gql auto-generated
// → Available in Playground

// Step 4: Add new field
@Field(() => Int, { nullable: true })
age?: number;
// → Schema automatically updated
// → No manual schema editing
```

**Schema First Workflow:**
```graphql
# Step 1: Write schema.graphql
type User {
  id: ID!
  name: String!
}

type Query {
  user(id: ID!): User
}
```

```typescript
// Step 2: Start server (generates graphql.ts)
export interface User {
  id: string;
  name: string;
}

// Step 3: Write resolver using generated types
@Resolver('User')
export class UserResolver {
  @Query('user')
  async user(): Promise<User> { /* ... */ }
}

// Step 4: Add new field
// Edit schema.graphql:
type User {
  id: ID!
  name: String!
  age: Int  # Added
}
// → Restart server to regenerate graphql.ts
// → Update resolver if needed
```

### **Entity Reuse Example**

**Code First: Easy Entity Reuse**
```typescript
// user.entity.ts (TypeORM + GraphQL)
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { ObjectType, Field, ID } from '@nestjs/graphql';

@Entity()  // TypeORM decorator
@ObjectType()  // GraphQL decorator
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)  // Both database and GraphQL
  id: string;

  @Column()
  @Field()  // Both database and GraphQL
  name: string;

  @Column({ unique: true })
  @Field()  // Both database and GraphQL
  email: string;
}
// Single class for database AND GraphQL schema
```

**Schema First: Separate Entities**
```graphql
# schema.graphql (GraphQL)
type User {
  id: ID!
  name: String!
  email: String!
}
```

```typescript
// user.entity.ts (TypeORM only)
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;
}
// Separate: Database entity != GraphQL type
// Need manual mapping in resolver
```

### **Complex Types Comparison**

**Code First: Union Types**
```typescript
import { createUnionType } from '@nestjs/graphql';

export const SearchResult = createUnionType({
  name: 'SearchResult',
  types: () => [User, Post, Comment] as const,
  resolveType(value) {
    if (value.email) return User;
    if (value.title) return Post;
    return Comment;
  },
});

@Resolver()
export class SearchResolver {
  @Query(() => [SearchResult])
  async search(@Args('query') query: string): Promise<typeof SearchResult[]> {
    // ...
  }
}
```

**Schema First: Union Types**
```graphql
# schema.graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}
```

```typescript
// Simpler in schema, but less type-safe in resolver
@Resolver()
export class SearchResolver {
  @Query('search')
  async search(@Args('query') query: string): Promise<Array<User | Post | Comment>> {
    // TypeScript doesn't know about union from schema
  }
}
```

**Interview Tip**: **Code First** vs **Schema First** key differences: (1) **Source of truth** - Code First = TypeScript classes, Schema First = `.graphql` files; (2) **Generation** - Code First generates schema from TS (`autoSchemaFile: 'schema.gql'`), Schema First generates TS from schema (`typePaths: ['./**/*.graphql']` + `definitions: { path: 'graphql.ts' }`); (3) **Type safety** - Code First native TS classes (full safety), Schema First generated interfaces (less safe, risk of mismatch); (4) **Decorators** - Code First heavy (`@ObjectType()`, `@Field()` on every field), Schema First minimal (only `@Resolver()`, `@Query()`); (5) **Workflow** - Code First writes TS first (TS → Schema), Schema First writes schema first (Schema → TS); (6) **Entity reuse** - Code First easy (TypeORM entity = GraphQL type with dual decorators), Schema First separate (entity != type, manual mapping). **Choose Code First** for TypeScript teams, type safety, single source in TS, entity reuse. **Choose Schema First** for schema-first design, GraphQL expertise, multi-language services, schema as contract. **Configuration**: Code First uses `autoSchemaFile`, Schema First uses `typePaths` + `definitions`. **Trade-off**: Code First more decorators but safer, Schema First simpler schema but less type-safe resolvers.

</details>
7. How do you install GraphQL dependencies (`@nestjs/graphql`, `@nestjs/apollo`)?

<details>
<summary><strong>Answer</strong></summary>

Install GraphQL dependencies using **npm** or **yarn**: **(1) Core packages** - `@nestjs/graphql` (NestJS GraphQL module), `@nestjs/apollo` (Apollo driver), `@apollo/server` (Apollo Server), `graphql` (GraphQL.js library); **(2) Code First** - add `type-graphql` or use built-in decorators; **(3) Subscriptions** - add `graphql-subscriptions` for PubSub, `graphql-ws` for WebSocket transport. Use `npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql` for basic setup.

### **Basic Installation (Required Packages)**

```bash
# Using npm
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql

# Using yarn
yarn add @nestjs/graphql @nestjs/apollo @apollo/server graphql

# Using pnpm
pnpm add @nestjs/graphql @nestjs/apollo @apollo/server graphql
```

### **Package Breakdown**

```typescript
// @nestjs/graphql
// - NestJS GraphQL integration module
// - Provides GraphQLModule, decorators (@Query, @Mutation, @Resolver)
// - Handles Code First and Schema First approaches
// - Auto-generates schema from TypeScript classes

// @nestjs/apollo
// - Apollo Server driver for NestJS
// - Integrates Apollo Server with NestJS
// - Provides ApolloDriver and ApolloDriverConfig

// @apollo/server
// - Apollo Server v4 (GraphQL server implementation)
// - Handles GraphQL query execution
// - Provides GraphQL Playground/Sandbox

// graphql
// - GraphQL.js reference implementation
// - Core GraphQL functionality
// - Required peer dependency
```

### **Code First Approach (Additional)**

```bash
# No additional packages needed - NestJS has built-in decorators
# But optionally install ts-morph for better schema generation
npm install ts-morph --save-dev
```

### **Schema First Approach (Additional)**

```bash
# Optional: Install for better GraphQL schema tooling
npm install @graphql-tools/schema @graphql-tools/utils
```

### **Subscriptions Support**

```bash
# For real-time subscriptions via WebSockets
npm install graphql-subscriptions graphql-ws

# graphql-subscriptions: PubSub implementation for subscriptions
# graphql-ws: WebSocket transport protocol for subscriptions
```

### **Development Tools (Optional)**

```bash
# GraphQL Code Generator (generates TypeScript types from schema)
npm install -D @graphql-codegen/cli @graphql-codegen/typescript

# GraphQL ESLint plugin
npm install -D @graphql-eslint/eslint-plugin

# GraphQL Tag for syntax highlighting
npm install graphql-tag
```

### **Complete Installation Example**

```bash
# Full setup for production-ready GraphQL API

# 1. Core dependencies
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql

# 2. Subscriptions (if needed)
npm install graphql-subscriptions graphql-ws

# 3. Validation (recommended)
npm install class-validator class-transformer

# 4. Development tools
npm install -D ts-morph
```

### **Package Versions (Compatibility)**

```json
// package.json
{
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/graphql": "^12.0.0",      // Latest NestJS GraphQL
    "@nestjs/apollo": "^12.0.0",       // Latest Apollo driver
    "@apollo/server": "^4.9.0",        // Apollo Server v4
    "graphql": "^16.8.0",              // GraphQL.js v16
    "graphql-subscriptions": "^2.0.0", // PubSub
    "graphql-ws": "^5.14.0",           // WebSocket transport
    "class-validator": "^0.14.0",
    "class-transformer": "^0.5.1"
  },
  "devDependencies": {
    "ts-morph": "^20.0.0"
  }
}
```

### **Verify Installation**

```typescript
// app.module.ts - Test imports
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';  // ✅ Should work
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';  // ✅ Should work

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
    }),
  ],
})
export class AppModule {}

// If imports work without errors, installation is successful
```

### **Alternative: Using Fastify Instead of Express**

```bash
# If using Fastify instead of Express
npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql
npm install @nestjs/platform-fastify
npm install @as-integrations/fastify
```

### **Troubleshooting Common Issues**

```bash
# Issue 1: Peer dependency warnings
# Solution: Install exact versions
npm install @nestjs/graphql@^12.0.0 @nestjs/apollo@^12.0.0 @apollo/server@^4.9.0 graphql@^16.8.0

# Issue 2: "Cannot find module '@nestjs/graphql'"
# Solution: Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Issue 3: TypeScript errors
# Solution: Update tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}

# Issue 4: GraphQL version mismatch
# Solution: Check graphql version
npm list graphql
# Should be v16.x for @nestjs/graphql v12
```

**Interview Tip**: Install GraphQL with **4 core packages**: `@nestjs/graphql` (NestJS integration, decorators), `@nestjs/apollo` (Apollo driver), `@apollo/server` (Apollo Server v4), `graphql` (GraphQL.js core). Command: `npm install @nestjs/graphql @nestjs/apollo @apollo/server graphql`. **Optional packages**: `graphql-subscriptions` + `graphql-ws` for real-time subscriptions, `class-validator` + `class-transformer` for input validation, `ts-morph` dev dependency for better Code First schema generation. **Compatibility**: Use `@nestjs/graphql@^12` with `@apollo/server@^4` and `graphql@^16`. **Verify**: Import `GraphQLModule` and `ApolloDriver` - if no errors, installation successful. **Note**: NestJS v10+ requires Apollo Server v4 (not v3), @nestjs/apollo replaces old apollo-server-express integration.

</details>
8. How do you configure `GraphQLModule` using `GraphQLModule.forRoot()`?

<details>
<summary><strong>Answer</strong></summary>

Configure `GraphQLModule` in `app.module.ts` using **`GraphQLModule.forRoot<ApolloDriverConfig>()`** with: **(1) driver** - `ApolloDriver` (required); **(2) Code First** - `autoSchemaFile: true` or path to generate schema; **(3) Schema First** - `typePaths: ['./**/*.graphql']` to load schema files; **(4) Playground** - `playground: true` enables GraphQL Playground; **(5) Additional options** - `context`, `formatError`, `subscriptions`, `plugins`. Configuration defines how GraphQL server behaves.

### **Basic Configuration (Code First)**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      // Required: Specify driver
      driver: ApolloDriver,
      
      // Code First: Auto-generate schema
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      // OR: autoSchemaFile: true,  // In-memory (no file)
      
      // Enable GraphQL Playground
      playground: true,
    }),
  ],
})
export class AppModule {}
```

### **Basic Configuration (Schema First)**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      // Required: Specify driver
      driver: ApolloDriver,
      
      // Schema First: Load schema files
      typePaths: ['./**/*.graphql'],
      
      // Generate TypeScript definitions
      definitions: {
        path: join(process.cwd(), 'src/graphql.ts'),
        outputAs: 'class',  // or 'interface'
      },
      
      // Enable GraphQL Playground
      playground: true,
    }),
  ],
})
export class AppModule {}
```

### **Complete Production Configuration**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ApolloServerPluginLandingPageLocalDefault } from '@apollo/server/plugin/landingPage/default';
import { join } from 'path';
import { Request, Response } from 'express';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      // ===== DRIVER =====
      driver: ApolloDriver,
      
      // ===== SCHEMA (Code First) =====
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      sortSchema: true,  // Sort fields alphabetically
      
      // ===== PLAYGROUND =====
      playground: false,  // Disable in production
      // Use Apollo Sandbox instead
      plugins: [
        ApolloServerPluginLandingPageLocalDefault(),
      ],
      
      // ===== CONTEXT =====
      // Add request/response to GraphQL context
      context: ({ req, res }: { req: Request; res: Response }) => ({
        req,
        res,
      }),
      
      // ===== ERROR FORMATTING =====
      formatError: (error) => {
        // Customize error response
        return {
          message: error.message,
          code: error.extensions?.code,
          path: error.path,
        };
      },
      
      // ===== CORS =====
      cors: {
        origin: ['http://localhost:3000', 'https://myapp.com'],
        credentials: true,
      },
      
      // ===== INTROSPECTION =====
      introspection: process.env.NODE_ENV !== 'production',
      
      // ===== SUBSCRIPTIONS (WebSocket) =====
      subscriptions: {
        'graphql-ws': true,  // Enable graphql-ws protocol
        'subscriptions-transport-ws': false,  // Deprecated protocol
      },
      
      // ===== CACHE CONTROL =====
      cache: 'bounded',  // Memory cache
      
      // ===== MODULES TO INCLUDE =====
      include: [UserModule, PostModule],
    }),
  ],
})
export class AppModule {}
```

### **Configuration Options Explained**

```typescript
GraphQLModule.forRoot<ApolloDriverConfig>({
  // ===== REQUIRED =====
  driver: ApolloDriver,  // Apollo Server driver
  
  // ===== SCHEMA GENERATION (Choose one) =====
  // Code First:
  autoSchemaFile: 'src/schema.gql',  // Generate schema file
  autoSchemaFile: true,              // In-memory schema (no file)
  sortSchema: true,                  // Sort schema alphabetically
  
  // Schema First:
  typePaths: ['./**/*.graphql'],     // Load schema files
  definitions: {                     // Generate TS types
    path: 'src/graphql.ts',
    outputAs: 'class',  // or 'interface'
    watch: true,        // Auto-regenerate on changes
  },
  
  // ===== DEVELOPMENT TOOLS =====
  playground: true,               // Enable GraphQL Playground (dev only)
  introspection: true,            // Enable schema introspection
  debug: true,                    // Show detailed errors (dev only)
  
  // ===== PRODUCTION SETTINGS =====
  playground: false,              // Disable Playground in prod
  introspection: false,           // Disable introspection in prod
  plugins: [                      // Use Apollo Sandbox instead
    ApolloServerPluginLandingPageLocalDefault(),
  ],
  
  // ===== CONTEXT =====
  context: ({ req, res }) => ({   // Add req/res to context
    req,                          // Access in resolvers via @Context()
    res,
    user: req.user,               // After authentication middleware
  }),
  
  // ===== ERROR HANDLING =====
  formatError: (error) => ({      // Customize error response
    message: error.message,
    code: error.extensions?.code,
    locations: error.locations,
    path: error.path,
  }),
  
  // ===== CORS =====
  cors: {
    origin: 'http://localhost:3000',  // Or array of origins
    credentials: true,                // Allow cookies
  },
  
  // ===== SUBSCRIPTIONS =====
  subscriptions: {
    'graphql-ws': true,              // Modern protocol
    'subscriptions-transport-ws': false,  // Old protocol
  },
  installSubscriptionHandlers: true, // Auto-setup WebSocket
  
  // ===== PERFORMANCE =====
  cache: 'bounded',                // In-memory cache
  persistedQueries: false,         // Disable for security
  
  // ===== MODULES =====
  include: [UserModule, PostModule],  // Only include specific modules
  
  // ===== PATH =====
  path: '/graphql',                // GraphQL endpoint (default)
  
  // ===== UPLOAD =====
  uploads: false,                  // File uploads (legacy, use alternative)
})
```

### **Environment-Specific Configuration**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    ConfigModule.forRoot(),
    
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        autoSchemaFile: true,
        
        // Environment-based settings
        playground: config.get('NODE_ENV') !== 'production',
        introspection: config.get('NODE_ENV') !== 'production',
        debug: config.get('NODE_ENV') === 'development',
        
        // CORS from environment
        cors: {
          origin: config.get('CORS_ORIGIN'),
          credentials: true,
        },
        
        context: ({ req, res }) => ({ req, res }),
      }),
    }),
  ],
})
export class AppModule {}
```

### **Multiple GraphQL Endpoints (Federation)**

```typescript
// app.module.ts - Multiple GraphQL APIs
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    // Public API
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: 'schema-public.gql',
      path: '/graphql',  // Public endpoint
      include: [UserModule, PostModule],
    }),
    
    // Admin API (separate endpoint)
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: 'schema-admin.gql',
      path: '/admin/graphql',  // Admin endpoint
      include: [AdminModule],
    }),
  ],
})
export class AppModule {}
```

### **Context with Authentication**

```typescript
// app.module.ts
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: true,
  
  // Add authenticated user to context
  context: async ({ req, res }) => {
    // Extract token from header
    const token = req.headers.authorization?.split(' ')[1];
    
    // Verify token (pseudo-code)
    let user = null;
    if (token) {
      try {
        user = await verifyJWT(token);
      } catch (err) {
        // Invalid token
      }
    }
    
    return { req, res, user };
  },
}),

// Access in resolver:
@Query(() => User)
async me(@Context() context) {
  return context.user;  // User from context
}
```

### **Custom Plugins**

```typescript
// logger.plugin.ts
import {
  ApolloServerPlugin,
  GraphQLRequestListener,
} from '@apollo/server';

export class LoggingPlugin implements ApolloServerPlugin {
  async requestDidStart(): Promise<GraphQLRequestListener<any>> {
    return {
      async willSendResponse({ response }) {
        console.log('GraphQL Response:', response);
      },
    };
  }
}

// app.module.ts
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: true,
  plugins: [new LoggingPlugin()],
}),
```

### **Error Formatting Example**

```typescript
// app.module.ts
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: true,
  
  formatError: (error) => {
    // Don't expose internal errors in production
    if (process.env.NODE_ENV === 'production') {
      return {
        message: 'Internal server error',
        code: 'INTERNAL_SERVER_ERROR',
      };
    }
    
    // Development: Show full error
    return {
      message: error.message,
      code: error.extensions?.code,
      locations: error.locations,
      path: error.path,
      extensions: error.extensions,
    };
  },
}),
```

**Interview Tip**: Configure GraphQL with **`GraphQLModule.forRoot<ApolloDriverConfig>()`** in `app.module.ts`: **(1) driver** - `ApolloDriver` (required), **(2) Code First** - `autoSchemaFile: 'src/schema.gql'` or `true` for in-memory, `sortSchema: true` optional, **(3) Schema First** - `typePaths: ['./**/*.graphql']` + `definitions: { path: 'src/graphql.ts', outputAs: 'class' }`, **(4) Playground** - `playground: true` for dev, `false` for production, **(5) Context** - `context: ({ req, res }) => ({ req, res, user: req.user })` adds request data to resolvers, **(6) Environment** - use `forRootAsync` with `ConfigService` for env-based config, **(7) Production** - disable `playground`/`introspection`, enable error formatting, set CORS. **Key options**: `autoSchemaFile` (Code First), `typePaths` (Schema First), `playground` (dev tool), `context` (pass req/res), `formatError` (customize errors), `subscriptions` (WebSocket), `cors` (cross-origin), `include` (specific modules). **Common pattern**: `playground: process.env.NODE_ENV !== 'production'` for environment-based settings.

</details>

## Code First Approach

9. What is Code First approach?

<details>
<summary><strong>Answer</strong></summary>

**Code First** approach means defining GraphQL schema using **TypeScript classes and decorators** (`@ObjectType()`, `@Field()`), and NestJS **auto-generates** the `.graphql` schema file from your code. Source of truth is **TypeScript code**, not SDL. Benefits: **(1) Type safety** - compile-time validation, **(2) Single source** - no schema duplication, **(3) Entity reuse** - TypeORM entities can be GraphQL types, **(4) Refactoring** - TypeScript tools work seamlessly. Use `autoSchemaFile` config to enable.

### **Code First Workflow**

```typescript
// Step 1: Write TypeScript classes with decorators
// user.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()  // Marks class as GraphQL type
export class User {
  @Field(() => ID)  // Each field needs @Field()
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;
}

// Step 2: NestJS auto-generates schema.gql:
# type User {
#   id: ID!
#   name: String!
#   email: String!
# }

// Step 3: Schema available in GraphQL Playground
// No manual .graphql file needed!
```

### **Configuration**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      
      // Enable Code First
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      // Generates schema.gql in src/
      
      // OR: In-memory schema (no file)
      // autoSchemaFile: true,
      
      // Optional: Sort fields alphabetically
      sortSchema: true,
    }),
  ],
})
export class AppModule {}
```

### **Complete Example**

```typescript
// ===== MODELS =====
// user.model.ts
import { ObjectType, Field, ID, Int } from '@nestjs/graphql';
import { Post } from './post.model';

@ObjectType({ description: 'User account' })
export class User {
  @Field(() => ID, { description: 'Unique identifier' })
  id: string;

  @Field({ description: 'Full name' })
  name: string;

  @Field()
  email: string;

  @Field(() => Int, { nullable: true })
  age?: number;

  @Field(() => [Post], { description: 'User posts' })
  posts: Post[];

  @Field({ defaultValue: true })
  isActive: boolean;
}

// post.model.ts
@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field()
  content: string;

  @Field(() => User)
  author: User;
}

// ===== INPUTS =====
// user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, MinLength } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;
}

// ===== RESOLVERS =====
// user.resolver.ts
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';
import { CreateUserInput } from './user.input';

@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return [];
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return null;
  }
}

// ===== AUTO-GENERATED SCHEMA (schema.gql) =====
# """User account"""
# type User {
#   """Unique identifier"""
#   id: ID!
#   
#   """Full name"""
#   name: String!
#   
#   email: String!
#   age: Int
#   
#   """User posts"""
#   posts: [Post!]!
#   
#   isActive: Boolean!
# }
# 
# type Post {
#   id: ID!
#   title: String!
#   content: String!
#   author: User!
# }
# 
# input CreateUserInput {
#   name: String!
#   email: String!
# }
# 
# type Query {
#   users: [User!]!
# }
# 
# type Mutation {
#   createUser(input: CreateUserInput!): User!
# }
```

### **Benefits of Code First**

```typescript
// 1. TYPE SAFETY
// TypeScript validates at compile-time
@ObjectType()
export class User {
  @Field(() => Int)
  age: number;  // TypeScript enforces number
}

const user: User = {
  age: '30',  // ❌ TypeScript error: Type 'string' not assignable to 'number'
};

// 2. SINGLE SOURCE OF TRUTH
// No schema duplication - TypeScript is the source
// Change TypeScript class → schema automatically updates

// 3. ENTITY REUSE
// Use same class for database and GraphQL
import { Entity, Column } from 'typeorm';
import { ObjectType, Field } from '@nestjs/graphql';

@Entity()
@ObjectType()  // Both TypeORM and GraphQL
export class User {
  @Column()
  @Field()  // Database column AND GraphQL field
  name: string;
}

// 4. REFACTORING
// Rename field in TypeScript:
@Field()
fullName: string;  // Renamed from 'name'

// Schema automatically updated:
// type User { fullName: String! }

// 5. IDE SUPPORT
// Full TypeScript IntelliSense
// Auto-complete, type checking, imports
```

### **Code First vs Manual Schema**

```typescript
// CODE FIRST: Write TypeScript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;
  
  @Field()
  name: string;
}
// → Schema auto-generated

// MANUAL SCHEMA: Write .graphql file
// type User {
//   id: ID!
//   name: String!
// }
// → Then generate TypeScript interfaces

// Code First: TS → Schema
// Schema First: Schema → TS
```

**Interview Tip**: **Code First** means defining schema using **TypeScript decorators** (`@ObjectType()`, `@Field()`) instead of writing `.graphql` files. NestJS **auto-generates** schema from classes. **Config**: `autoSchemaFile: 'src/schema.gql'` or `true` (in-memory). **Benefits**: (1) **type safety** - compile-time validation, (2) **single source** - TypeScript is truth, no duplication, (3) **entity reuse** - TypeORM entity = GraphQL type with dual decorators, (4) **refactoring** - rename in TS, schema updates automatically. **Workflow**: write `@ObjectType()` class → start server → schema.gql generated → available in Playground. **Decorators**: `@ObjectType()` (type), `@InputType()` (input), `@Field()` (field), `@Resolver()` (resolver). **Trade-off**: more decorators vs full TypeScript integration. Better for TypeScript-heavy teams.

</details>
10. How do you define GraphQL types using TypeScript classes?

<details>
<summary><strong>Answer</strong></summary>

Define GraphQL types using **TypeScript classes** with **`@ObjectType()`** decorator for the class and **`@Field()`** for each field. Import from `@nestjs/graphql`: `ObjectType`, `Field`, `ID`, `Int`, `Float`. Field types map to GraphQL scalars: `string` → `String`, `number` → `Float` (use `Int` for integers), `boolean` → `Boolean`. Use `() => ID` for ID fields, `() => [Type]` for arrays, `{ nullable: true }` for optional fields.

### **Basic Type Definition**

```typescript
// user.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()  // Marks class as GraphQL object type
export class User {
  @Field(() => ID)  // GraphQL ID type
  id: string;

  @Field()  // GraphQL String (inferred from TS string)
  name: string;

  @Field()  // GraphQL String
  email: string;
}

// Generated schema:
// type User {
//   id: ID!
//   name: String!
//   email: String!
// }
```

### **All Scalar Types**

```typescript
import { ObjectType, Field, ID, Int, Float } from '@nestjs/graphql';

@ObjectType()
export class Example {
  // ID type
  @Field(() => ID)
  id: string;  // or number

  // String type (auto-inferred)
  @Field()
  name: string;

  // Int type (must specify)
  @Field(() => Int)
  age: number;

  // Float type (auto for number, or explicit)
  @Field(() => Float)
  price: number;

  // Boolean type (auto-inferred)
  @Field()
  isActive: boolean;

  // Date (mapped to String in GraphQL)
  @Field()
  createdAt: Date;
}

// Generated:
// type Example {
//   id: ID!
//   name: String!
//   age: Int!
//   price: Float!
//   isActive: Boolean!
//   createdAt: DateTime!  # Custom scalar
// }
```

### **Nullable Fields**

```typescript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()  // Required (non-null)
  name: string;

  @Field({ nullable: true })  // Optional
  bio?: string;

  @Field(() => Int, { nullable: true })
  age?: number;
}

// Generated:
// type User {
//   id: ID!
//   name: String!  # Required
//   bio: String    # Optional (nullable)
//   age: Int       # Optional
// }
```

### **Array Fields**

```typescript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  // Array of strings (items and array non-null)
  @Field(() => [String])
  tags: string[];

  // Array of Posts (items and array non-null)
  @Field(() => [Post])
  posts: Post[];

  // Nullable array
  @Field(() => [String], { nullable: true })
  optionalTags?: string[];

  // Array with nullable items
  @Field(() => [String], { nullable: 'items' })
  tagsWithNulls: (string | null)[];

  // Both array and items nullable
  @Field(() => [String], { nullable: 'itemsAndList' })
  fullyOptional?: (string | null)[];
}

// Generated:
// type User {
//   id: ID!
//   tags: [String!]!           # Array and items required
//   posts: [Post!]!            # Array and items required
//   optionalTags: [String!]    # Array optional, items required
//   tagsWithNulls: [String]!   # Array required, items optional
//   fullyOptional: [String]    # Both optional
// }
```

### **Relationships**

```typescript
// user.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';
import { Post } from './post.model';

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  // One-to-many relationship
  @Field(() => [Post], { description: 'User posts' })
  posts: Post[];
}

// post.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';
import { User } from './user.model';

@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  // Many-to-one relationship
  @Field(() => User, { description: 'Post author' })
  author: User;
}

// Generated:
// type User {
//   id: ID!
//   name: String!
//   posts: [Post!]!
// }
// 
// type Post {
//   id: ID!
//   title: String!
//   author: User!
// }
```

### **Field Options**

```typescript
@ObjectType({ description: 'User account' })  // Type description
export class User {
  @Field(() => ID, {
    description: 'Unique identifier',  // Field description
    nullable: false,                   // Explicit non-null (default)
    deprecationReason: 'Use newId',    // Deprecation
  })
  id: string;

  @Field({
    name: 'fullName',  // Different name in schema
    description: 'User full name',
    defaultValue: 'Anonymous',  // Default value
  })
  name: string;

  @Field(() => Int, {
    nullable: true,
    description: 'User age in years',
  })
  age?: number;
}

// Generated:
// """User account"""
// type User {
//   """Unique identifier"""
//   id: ID! @deprecated(reason: "Use newId")
//   
//   """User full name"""
//   fullName: String!
//   
//   """User age in years"""
//   age: Int
// }
```

### **Complex Example**

```typescript
// user.model.ts
import { ObjectType, Field, ID, Int, registerEnumType } from '@nestjs/graphql';
import { Post } from './post.model';
import { Profile } from './profile.model';

// Enum
export enum Role {
  ADMIN = 'ADMIN',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
}

registerEnumType(Role, {
  name: 'Role',
  description: 'User role',
});

@ObjectType({ description: 'User account type' })
export class User {
  @Field(() => ID)
  id: string;

  @Field({ description: 'User full name' })
  name: string;

  @Field()
  email: string;

  @Field(() => Int, { nullable: true })
  age?: number;

  @Field(() => Role, { description: 'User role' })
  role: Role;

  // One-to-one
  @Field(() => Profile, { nullable: true })
  profile?: Profile;

  // One-to-many
  @Field(() => [Post], { description: 'User posts' })
  posts: Post[];

  @Field({ defaultValue: true })
  isActive: boolean;

  @Field()
  createdAt: Date;

  @Field()
  updatedAt: Date;
}

// Generated:
// enum Role {
//   ADMIN
//   USER
//   MODERATOR
// }
// 
// """User account type"""
// type User {
//   id: ID!
//   
//   """User full name"""
//   name: String!
//   
//   email: String!
//   age: Int
//   
//   """User role"""
//   role: Role!
//   
//   profile: Profile
//   
//   """User posts"""
//   posts: [Post!]!
//   
//   isActive: Boolean!
//   createdAt: DateTime!
//   updatedAt: DateTime!
// }
```

### **Reusing TypeORM Entities**

```typescript
// user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { ObjectType, Field, ID } from '@nestjs/graphql';

@Entity()      // TypeORM entity
@ObjectType()  // GraphQL type
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)  // Both database and GraphQL
  id: string;

  @Column()
  @Field()  // Both database and GraphQL
  name: string;

  @Column({ unique: true })
  @Field()  // Both database and GraphQL
  email: string;

  @Column({ nullable: true })
  @Field({ nullable: true })  // Both database and GraphQL
  bio?: string;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  @Field()  // Both database and GraphQL
  createdAt: Date;
}

// Single class serves both purposes:
// - TypeORM entity for database
// - GraphQL type for API
```

**Interview Tip**: Define GraphQL types using **TypeScript classes** with **`@ObjectType()`** on class, **`@Field()`** on each field. **Imports**: `{ ObjectType, Field, ID, Int, Float }` from `@nestjs/graphql`. **Type mapping**: `string` → `String`, `number` → `Float` (or `() => Int`), `boolean` → `Boolean`, `Date` → `DateTime`. **ID fields**: `@Field(() => ID)` for unique identifiers. **Arrays**: `@Field(() => [Post])` for array of Posts, `[String]` for strings. **Nullable**: `{ nullable: true }` for optional fields (`age?: number`). **Relationships**: `@Field(() => Post)` for single, `@Field(() => [Post])` for many. **Field options**: `description`, `nullable`, `defaultValue`, `name` (rename), `deprecationReason`. **Entity reuse**: add both `@Entity()` (TypeORM) and `@ObjectType()` (GraphQL) to same class, use `@Column()` and `@Field()` on fields - single source for database and API.

</details>
11. What is `@ObjectType()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**`@ObjectType()`** marks a TypeScript class as a **GraphQL object type**, making it available in the schema. Applied at **class level**, it tells NestJS to generate a `type` definition in GraphQL schema. Options: **`description`** (type documentation), **`isAbstract`** (for inheritance, won't generate type), **`implements`** (interface implementation). Without `@ObjectType()`, class won't appear in schema even if fields have `@Field()`.

### **Basic Usage**

```typescript
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()  // Marks class as GraphQL type
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;
}

// Generated schema:
// type User {
//   id: ID!
//   name: String!
// }
```

### **With Options**

```typescript
@ObjectType({
  description: 'User account object',  // Adds schema documentation
})
export class User {
  @Field(() => ID, { description: 'Unique user ID' })
  id: string;

  @Field()
  name: string;
}

// Generated schema with descriptions:
// """
// User account object
// """
// type User {
//   """Unique user ID"""
//   id: ID!
//   name: String!
// }
```

### **Abstract Types (for Inheritance)**

```typescript
// Base class - won't generate schema type
@ObjectType({ isAbstract: true })
abstract class Node {
  @Field(() => ID)
  id: string;

  @Field()
  createdAt: Date;
}

// Concrete types - will generate schema
@ObjectType()
export class User extends Node {
  @Field()
  name: string;
}

@ObjectType()
export class Post extends Node {
  @Field()
  title: string;
}

// Generated schema (Node NOT included):
// type User {
//   id: ID!
//   createdAt: DateTime!
//   name: String!
// }
// 
// type Post {
//   id: ID!
//   createdAt: DateTime!
//   title: String!
// }
```

### **Interface Implementation**

```typescript
// Define GraphQL interface
import { InterfaceType } from '@nestjs/graphql';

@InterfaceType()
abstract class Node {
  @Field(() => ID)
  id: string;
}

// Implement interface
@ObjectType({ implements: () => [Node] })
export class User extends Node {
  @Field()
  name: string;
}

@ObjectType({ implements: () => [Node] })
export class Post extends Node {
  @Field()
  title: string;
}

// Generated schema:
// interface Node {
//   id: ID!
// }
// 
// type User implements Node {
//   id: ID!
//   name: String!
// }
// 
// type Post implements Node {
//   id: ID!
//   title: String!
// }
```

**Interview Tip**: **`@ObjectType()`** decorator marks a class as **GraphQL object type**, generating `type` in schema. Applied at **class level** before class definition. **Options**: `description` (documentation in schema), `isAbstract: true` (inheritance base, no schema generation), `implements: () => [Interface]` (implement GraphQL interface). **Without it**, class won't appear in schema even with `@Field()` decorators. **Example**: `@ObjectType({ description: 'User account' })` → generates `type User` with description. Used with `@Field()` on properties to define GraphQL fields.

</details>
12. What is `@Field()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**`@Field()`** marks a class property as a **GraphQL field**, making it queryable in the schema. Applied to **each property** you want exposed. Options: **type function** `() => Type` (explicit type), **`nullable`** (optional field), **`description`** (documentation), **`defaultValue`**, **`name`** (rename), **`deprecationReason`**. Without `@Field()`, property won't appear in schema.

### **Basic Usage**

```typescript
import { ObjectType, Field } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field()  // String type auto-inferred
  name: string;

  @Field()  // String type auto-inferred
  email: string;
  
  // Property without @Field() - NOT in schema
  password: string;  // Hidden from GraphQL
}

// Generated schema:
// type User {
//   name: String!
//   email: String!
//   // password NOT included
// }
```

### **Explicit Type**

```typescript
import { ObjectType, Field, ID, Int } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field(() => ID)  // Explicit ID type
  id: string;

  @Field()  // Inferred String
  name: string;

  @Field(() => Int)  // Explicit Int (not Float)
  age: number;
}

// Generated schema:
// type User {
//   id: ID!
//   name: String!
//   age: Int!
// }
```

### **Field Options**

```typescript
@ObjectType()
export class User {
  @Field(() => ID, {
    description: 'Unique user identifier',  // Documentation
    nullable: false,  // Explicit non-null (default)
  })
  id: string;

  @Field({
    description: 'User full name',
    nullable: false,
  })
  name: string;

  @Field(() => Int, {
    nullable: true,  // Optional field
    description: 'User age in years',
  })
  age?: number;

  @Field({
    defaultValue: true,  // Default value
    description: 'Account active status',
  })
  isActive: boolean;

  @Field({
    name: 'emailAddress',  // Different name in schema
    description: 'User email',
  })
  email: string;

  @Field({
    deprecationReason: 'Use firstName and lastName instead',
  })
  fullName: string;
}

// Generated schema:
// type User {
//   """Unique user identifier"""
//   id: ID!
//   
//   """User full name"""
//   name: String!
//   
//   """User age in years"""
//   age: Int
//   
//   """Account active status"""
//   isActive: Boolean!
//   
//   """User email"""
//   emailAddress: String!
//   
//   fullName: String! @deprecated(reason: "Use firstName and lastName instead")
// }
```

### **Array Fields**

```typescript
@ObjectType()
export class User {
  @Field(() => [String])  // Array of strings
  tags: string[];

  @Field(() => [Post])  // Array of Posts
  posts: Post[];

  @Field(() => [String], { nullable: true })  // Optional array
  optionalTags?: string[];

  @Field(() => [String], { nullable: 'items' })  // Nullable items
  tagsWithNulls: (string | null)[];

  @Field(() => [String], { nullable: 'itemsAndList' })  // Both nullable
  fullyOptional?: (string | null)[];
}

// Generated schema:
// type User {
//   tags: [String!]!           # Required array, required items
//   posts: [Post!]!            # Required array, required items
//   optionalTags: [String!]    # Optional array, required items
//   tagsWithNulls: [String]!   # Required array, optional items
//   fullyOptional: [String]    # Optional array, optional items
// }
```

### **Relationship Fields**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field(() => Profile, { nullable: true })  // One-to-one
  profile?: Profile;

  @Field(() => [Post])  // One-to-many
  posts: Post[];
}

// post.model.ts
@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field(() => User)  // Many-to-one
  author: User;
}
```

### **Private Fields (Excluded)**

```typescript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  // Private fields - NO @Field() decorator
  password: string;         // Not in schema
  passwordHash: string;     // Not in schema
  internalNotes: string;    // Not in schema
  
  // These are TypeScript-only, hidden from GraphQL
}

// Generated schema (password fields excluded):
// type User {
//   id: ID!
//   name: String!
// }
```

**Interview Tip**: **`@Field()`** marks a property as **GraphQL field**, exposing it in schema. Applied to **each property** individually. **Syntax**: `@Field()` (inferred type), `@Field(() => Type)` (explicit type), `@Field(() => Int)` (specify Int vs Float), `@Field(() => [Type])` (arrays). **Options**: `nullable: true` (optional field for `age?: number`), `description` (docs), `defaultValue` (default), `name` (rename in schema), `deprecationReason` (deprecate old fields). **Arrays**: `[String!]!` (both required), `nullable: 'items'` (items optional), `nullable: 'itemsAndList'` (both optional). **Private fields**: properties without `@Field()` stay hidden (e.g., `password` field excluded from schema). **Relationships**: `@Field(() => Post)` for single, `@Field(() => [Post])` for array.

</details>
13. How do you define nullable fields?

<details>
<summary><strong>Answer</strong></summary>

Define nullable fields using **`{ nullable: true }`** option in `@Field()` decorator, and TypeScript optional property **`?:`**. By default, fields are **required** (`!` in schema). For arrays: **`nullable: true`** = array optional, **`nullable: 'items'`** = items optional, **`nullable: 'itemsAndList'`** = both optional. Nullable fields map to optional GraphQL fields (no `!` in schema).

### **Basic Nullable Fields**

```typescript
import { ObjectType, Field, Int } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field()  // Required field (default)
  name: string;  // name: String!

  @Field({ nullable: true })  // Optional field
  bio?: string;  // bio: String (no !)

  @Field(() => Int, { nullable: true })  // Optional Int
  age?: number;  // age: Int

  @Field({ nullable: false })  // Explicit required (same as default)
  email: string;  // email: String!
}

// Generated schema:
// type User {
//   name: String!   # Required
//   bio: String     # Nullable
//   age: Int        # Nullable
//   email: String!  # Required
// }
```

### **Array Nullability**

```typescript
@ObjectType()
export class User {
  // Default: Both array and items required
  @Field(() => [String])
  tags: string[];  // tags: [String!]!

  // Array optional, items required
  @Field(() => [String], { nullable: true })
  optionalTags?: string[];  // optionalTags: [String!]

  // Array required, items optional
  @Field(() => [String], { nullable: 'items' })
  tagsWithNulls: (string | null)[];  // tagsWithNulls: [String]!

  // Both array and items optional
  @Field(() => [String], { nullable: 'itemsAndList' })
  fullyOptional?: (string | null)[];  // fullyOptional: [String]
}

// Generated schema:
// type User {
//   tags: [String!]!        # Array required, items required
//   optionalTags: [String!] # Array optional, items required
//   tagsWithNulls: [String]! # Array required, items optional
//   fullyOptional: [String] # Array optional, items optional
// }
```

### **Relationship Nullability**

```typescript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  // Required relationship
  @Field(() => Profile)
  profile: Profile;  // profile: Profile!

  // Optional relationship
  @Field(() => Profile, { nullable: true })
  optionalProfile?: Profile;  // optionalProfile: Profile

  // Required array of posts
  @Field(() => [Post])
  posts: Post[];  // posts: [Post!]!

  // Optional array of posts
  @Field(() => [Post], { nullable: true })
  optionalPosts?: Post[];  // optionalPosts: [Post!]
}
```

### **Complex Nullability Examples**

```typescript
@ObjectType()
export class Example {
  // 1. Required field
  @Field()
  requiredString: string;  // requiredString: String!

  // 2. Optional field
  @Field({ nullable: true })
  optionalString?: string;  // optionalString: String

  // 3. Required array, required items
  @Field(() => [String])
  requiredArray: string[];  // requiredArray: [String!]!

  // 4. Optional array, required items
  @Field(() => [String], { nullable: true })
  optionalArray?: string[];  // optionalArray: [String!]

  // 5. Required array, optional items
  @Field(() => [String], { nullable: 'items' })
  arrayWithNullableItems: (string | null)[];  // arrayWithNullableItems: [String]!

  // 6. Optional array, optional items
  @Field(() => [String], { nullable: 'itemsAndList' })
  fullyOptionalArray?: (string | null)[];  // fullyOptionalArray: [String]
}
```

### **TypeScript vs GraphQL Nullability**

```typescript
@ObjectType()
export class User {
  // TypeScript optional (?) + nullable: true
  @Field({ nullable: true })
  bio?: string;  // ✅ Correct: Both match

  // TypeScript required + nullable: false (or omit)
  @Field()
  name: string;  // ✅ Correct: Both match

  // TypeScript optional (?) + nullable: false
  @Field({ nullable: false })  // ❌ Warning: Mismatch
  email?: string;  // TypeScript allows undefined, but GraphQL doesn't

  // TypeScript required + nullable: true
  @Field({ nullable: true })  // ❌ Warning: Mismatch
  age: number;  // TypeScript requires value, but GraphQL allows null
}

// Best practice: Match TypeScript and GraphQL nullability
// Optional in TS (?) → nullable: true
// Required in TS → no nullable option (or nullable: false)
```

### **Default Values for Nullable Fields**

```typescript
@ObjectType()
export class User {
  @Field({ nullable: true, defaultValue: 'No bio' })
  bio?: string;  // bio: String = "No bio"

  @Field(() => Int, { nullable: true, defaultValue: 0 })
  score?: number;  // score: Int = 0

  @Field({ nullable: true, defaultValue: true })
  isActive?: boolean;  // isActive: Boolean = true

  @Field(() => [String], { nullable: true, defaultValue: [] })
  tags?: string[];  // tags: [String!] = []
}

// Generated schema:
// type User {
//   bio: String = "No bio"
//   score: Int = 0
//   isActive: Boolean = true
//   tags: [String!] = []
// }
```

### **Validation with Nullable Fields**

```typescript
import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, IsOptional, MinLength } from 'class-validator';

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()  // Validation: Optional
  @MinLength(2)  // But if provided, must be >= 2 chars
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  @Field(() => Int, { nullable: true })
  @IsOptional()
  age?: number;
}
```

**Interview Tip**: Define nullable fields with **`{ nullable: true }`** in `@Field()` decorator + TypeScript **`?:`** optional property. **Default**: fields are **required** (GraphQL `!`). **Arrays**: `nullable: true` = optional array `[String!]`, `nullable: 'items'` = optional items `[String]!`, `nullable: 'itemsAndList'` = both optional `[String]`. **Example**: `@Field({ nullable: true }) bio?: string` → `bio: String` (no `!`). **Best practice**: match TypeScript and GraphQL nullability (TS `?:` with GraphQL `nullable: true`). **Default values**: `defaultValue: 'value'` provides default for nullable fields. **Common pattern**: update inputs have nullable fields for partial updates (`UpdateUserInput` with all fields optional).

</details>
14. What is `@InputType()` for input types?

<details>
<summary><strong>Answer</strong></summary>

**`@InputType()`** marks a class as **GraphQL input type**, used for **mutation/query arguments**. Input types are like object types but for **data input** (not output). Applied at class level, use `@Field()` for properties. Input types generate `input` in schema (not `type`). Used with **`@Args('input')`** in mutations for structured data (create/update operations).

### **Basic Input Type**

```typescript
import { InputType, Field } from '@nestjs/graphql';

@InputType()  // Marks as GraphQL input
export class CreateUserInput {
  @Field()
  name: string;

  @Field()
  email: string;

  @Field()
  password: string;
}

// Generated schema:
// input CreateUserInput {
//   name: String!
//   email: String!
//   password: String!
// }
```

### **Usage in Mutation**

```typescript
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';
import { CreateUserInput } from './user.input';

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,  // Input type as argument
  ): Promise<User> {
    // input.name, input.email, input.password are type-safe
    return this.userService.create(input);
  }
}

// GraphQL mutation:
// mutation {
//   createUser(input: {
//     name: "John"
//     email: "john@example.com"
//     password: "secret123"
//   }) {
//     id
//     name
//     email
//   }
// }
```

### **Optional Fields in Input**

```typescript
import { InputType, Field, Int } from '@nestjs/graphql';

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })  // Optional field
  name?: string;

  @Field({ nullable: true })
  email?: string;

  @Field(() => Int, { nullable: true })
  age?: number;
}

// Generated schema:
// input UpdateUserInput {
//   name: String
//   email: String
//   age: Int
// }

// Usage:
@Mutation(() => User)
async updateUser(
  @Args('id') id: string,
  @Args('input') input: UpdateUserInput,  // Partial update
): Promise<User> {
  return this.userService.update(id, input);
}

// GraphQL mutation (partial update):
// mutation {
//   updateUser(id: "1", input: {
//     name: "John Smith"  # Only update name
//   }) {
//     id
//     name
//   }
// }
```

### **Validation in Input Types**

```typescript
import { InputType, Field, Int } from '@nestjs/graphql';
import { IsEmail, MinLength, Min, Max, IsOptional } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  name: string;

  @Field()
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;

  @Field(() => Int, { nullable: true })
  @IsOptional()
  @Min(18, { message: 'Must be at least 18 years old' })
  @Max(120, { message: 'Invalid age' })
  age?: number;
}

// Validation happens automatically with ValidationPipe
// Invalid input → GraphQL error with validation messages
```

### **Nested Input Types**

```typescript
// address.input.ts
@InputType()
export class AddressInput {
  @Field()
  street: string;

  @Field()
  city: string;

  @Field()
  country: string;
}

// user.input.ts
@InputType()
export class CreateUserInput {
  @Field()
  name: string;

  @Field()
  email: string;

  @Field(() => AddressInput)  // Nested input
  address: AddressInput;
}

// Generated schema:
// input AddressInput {
//   street: String!
//   city: String!
//   country: String!
// }
// 
// input CreateUserInput {
//   name: String!
//   email: String!
//   address: AddressInput!
// }

// Usage:
// mutation {
//   createUser(input: {
//     name: "John"
//     email: "john@example.com"
//     address: {
//       street: "123 Main St"
//       city: "NYC"
//       country: "USA"
//     }
//   }) {
//     id
//     name
//   }
// }
```

### **Input vs Object Type**

```typescript
// OUTPUT TYPE (query/mutation response)
@ObjectType()  // Use @ObjectType for output
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;

  // Can include computed fields
  @Field(() => [Post])
  posts: Post[];
}

// INPUT TYPE (mutation/query argument)
@InputType()  // Use @InputType for input
export class CreateUserInput {
  // No id (server generates)
  @Field()
  name: string;

  @Field()
  email: string;

  @Field()
  password: string;  // Not in User output

  // No computed fields (posts)
}

// Key difference: Input types are for data going IN
// Object types are for data coming OUT
```

### **Array Input Fields**

```typescript
@InputType()
export class CreatePostInput {
  @Field()
  title: string;

  @Field()
  content: string;

  @Field(() => [String])  // Array of strings
  tags: string[];

  @Field(() => [ID], { nullable: true })  // Optional array of IDs
  categoryIds?: string[];
}

// Generated schema:
// input CreatePostInput {
//   title: String!
//   content: String!
//   tags: [String!]!
//   categoryIds: [ID!]
// }
```

### **Complete CRUD Example**

```typescript
// user.input.ts
@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;
}

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  // CREATE
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }

  // UPDATE
  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    return this.userService.update(id, input);
  }
}
```

**Interview Tip**: **`@InputType()`** marks a class as **GraphQL input type** for **mutation/query arguments**. Used for **data input** (create/update), generates `input` in schema (not `type`). **Usage**: `@Args('input') input: CreateUserInput` in mutations. **Difference from `@ObjectType()`**: InputType = data going IN (arguments), ObjectType = data coming OUT (responses). **Validation**: combine with `class-validator` decorators (`@IsEmail()`, `@MinLength()`) for automatic validation. **Optional fields**: `{ nullable: true }` for partial updates (`UpdateUserInput`). **Nested**: `@Field(() => AddressInput)` for nested input objects. **Arrays**: `@Field(() => [String])` for array inputs. **Best practice**: separate CreateInput (all required) and UpdateInput (all optional) for different operations.

</details>

## Resolvers

15. What is a Resolver in GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**Resolver** is a function/method that **resolves a GraphQL field value**. Every field in schema needs a resolver. In NestJS, resolvers are **classes** with methods decorated by `@Query()`, `@Mutation()`, `@Subscription()`. Resolvers handle business logic, fetch data from DB/API, and return data matching schema type. Think of resolvers as **controllers for GraphQL** (like REST controllers handle HTTP endpoints).

### **Resolver Basics**

```typescript
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';

@Resolver(() => User)  // Resolver for User type
export class UserResolver {
  constructor(private userService: UserService) {}

  // Query resolver
  @Query(() => [User], { name: 'users' })
  async findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Query resolver with arguments
  @Query(() => User, { name: 'user', nullable: true })
  async findOne(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Mutation resolver
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }
}

// Generated schema:
// type Query {
//   users: [User!]!
//   user(id: String!): User
// }
// 
// type Mutation {
//   createUser(input: CreateUserInput!): User!
// }
```

### **How Resolvers Work**

```typescript
// GraphQL query:
// query {
//   users {        # 1. Calls UserResolver.findAll()
//     id
//     name
//     posts {      # 2. Calls UserResolver.posts() for each user
//       id
//       title
//     }
//   }
// }

@Resolver(() => User)
export class UserResolver {
  // 1. Root resolver - fetches users
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
    // Returns: [{ id: '1', name: 'John' }, { id: '2', name: 'Jane' }]
  }

  // 2. Field resolver - resolves 'posts' for each user
  @ResolveField(() => [Post], { name: 'posts' })
  async getPosts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
    // Called for each user separately
  }
}
```

### **Resolver Types**

```typescript
// 1. Query Resolver (Read operations)
@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }
}

// 2. Mutation Resolver (Write operations)
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);
  }

  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    return this.userService.update(id, input);
  }

  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    return this.userService.delete(id);
  }
}

// 3. Subscription Resolver (Real-time)
@Resolver(() => User)
export class UserResolver {
  constructor(private pubSub: PubSub) {}

  @Subscription(() => User)
  userCreated() {
    return this.pubSub.asyncIterator('userCreated');
  }
}

// 4. Field Resolver (Resolve specific fields)
@Resolver(() => User)
export class UserResolver {
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
  }

  @ResolveField(() => String)
  async fullName(@Parent() user: User): Promise<string> {
    return `${user.firstName} ${user.lastName}`;
  }
}
```

### **Resolver with Service Layer**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.userRepo.find();
  }

  async findById(id: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { id } });
  }

  async create(input: CreateUserInput): Promise<User> {
    const user = this.userRepo.create(input);
    return this.userRepo.save(user);
  }

  async update(id: string, input: UpdateUserInput): Promise<User> {
    await this.userRepo.update(id, input);
    return this.findById(id);
  }

  async delete(id: string): Promise<boolean> {
    const result = await this.userRepo.delete(id);
    return result.affected > 0;
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}  // Inject service

  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();  // Delegate to service
  }

  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);
  }
}

// Module setup
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UserResolver, UserService],
})
export class UserModule {}
```

### **Resolver Context**

```typescript
import { Resolver, Query, Context } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User, { nullable: true })
  async me(@Context() context): Promise<User | null> {
    // Access request, response, user from context
    const userId = context.req.user?.id;
    if (!userId) return null;
    
    return this.userService.findById(userId);
  }
}

// Configure context in GraphQLModule
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,n  autoSchemaFile: true,
  context: ({ req, res }) => ({ req, res }),  // Add req/res to context
}),
```

**Interview Tip**: **Resolver** is a class with methods that **resolve GraphQL fields**. In NestJS, resolvers are classes with `@Resolver()` decorator containing `@Query()` (read), `@Mutation()` (write), `@Subscription()` (real-time), `@ResolveField()` (field resolution) methods. **Think of resolvers as controllers** for GraphQL (handle business logic, fetch data, return responses). **Structure**: Resolver (thin, routing) → Service (business logic, DB access). **Each field** can have a resolver; if not defined, GraphQL uses default resolver (returns field value from parent object).

</details>
16. What is `@Resolver()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**`@Resolver()`** marks a class as a **GraphQL resolver**, making it available to resolve queries, mutations, and subscriptions. Applied at **class level**, optionally takes **type function** `() => ObjectType` to indicate which type it resolves. Without `@Resolver()`, class methods won't be registered as GraphQL operations. NestJS automatically registers resolvers as providers.

### **Basic Usage**

```typescript
import { Resolver, Query } from '@nestjs/graphql';
import { User } from './user.model';

@Resolver(() => User)  // Resolver for User type
export class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return [];
  }
}

// Without @Resolver(), class won't be registered
```

### **With Type Argument**

```typescript
// Specify which type this resolver handles
@Resolver(() => User)  // Required for @ResolveField
export class UserResolver {
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
  }
}
```

### **Without Type Argument**

```typescript
// For Query/Mutation only (no @ResolveField)
@Resolver()  // No type specified
export class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);
  }
}
```

### **Complete Example**

```typescript
@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);
  }
}

// Module
@Module({
  providers: [UserResolver, UserService, PostService],
})
export class UserModule {}
```

**Interview Tip**: **`@Resolver()`** marks a class as **GraphQL resolver**, enabling `@Query()`, `@Mutation()`, `@Subscription()`, `@ResolveField()` methods. Applied at **class level**. **Syntax**: `@Resolver()` (no type, Query/Mutation only) or `@Resolver(() => User)` (with type, enables `@ResolveField`). **Type argument required** for field resolvers. **Registration**: add to module `providers` array, NestJS auto-discovers. **DI support**: inject services via constructor.

</details>
17. How do you create query resolvers using `@Query()`?

<details>
<summary><strong>Answer</strong></summary>

**`@Query()`** decorator marks a method as **GraphQL query resolver**. Applied to methods inside `@Resolver()` class. Takes **return type function** `() => Type` (required) and **options** (`name`, `nullable`, `description`). Query methods handle **read operations** (fetch data), equivalent to REST GET. Method name becomes query name unless specified in options.

### **Basic Query**

```typescript
import { Resolver, Query } from '@nestjs/graphql';
import { User } from './user.model';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => [User])  // Return array of Users
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }
}

// Generated schema:
// type Query {
//   users: [User!]!
// }

// GraphQL query:
// query {
//   users {
//     id
//     name
//   }
// }
```

### **Query with Arguments**

```typescript
import { Resolver, Query, Args } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  @Query(() => [User])
  async searchUsers(@Args('name') name: string): Promise<User[]> {
    return this.userService.searchByName(name);
  }
}

// Generated schema:
// type Query {
//   user(id: String!): User
//   searchUsers(name: String!): [User!]!
// }

// GraphQL query:
// query {
//   user(id: "1") {
//     id
//     name
//   }
//   searchUsers(name: "John") {
//     id
//     name
//   }
// }
```

### **Query Options**

```typescript
@Resolver(() => User)
export class UserResolver {
  // Custom name
  @Query(() => [User], { name: 'allUsers' })
  async findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Nullable result
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // With description
  @Query(() => [User], {
    name: 'users',
    description: 'Get all users from the system',
  })
  async getUsers(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Deprecated query
  @Query(() => [User], {
    deprecationReason: 'Use allUsers instead',
  })
  async oldGetUsers(): Promise<User[]> {
    return this.userService.findAll();
  }
}

// Generated schema:
// type Query {
//   allUsers: [User!]!
//   user(id: String!): User
//   """Get all users from the system"""
//   users: [User!]!
//   oldGetUsers: [User!]! @deprecated(reason: "Use allUsers instead")
// }
```

### **Query with Multiple Arguments**

```typescript
import { Resolver, Query, Args, Int } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  async users(
    @Args('skip', { type: () => Int, defaultValue: 0 }) skip: number,
    @Args('take', { type: () => Int, defaultValue: 10 }) take: number,
    @Args('search', { nullable: true }) search?: string,
  ): Promise<User[]> {
    return this.userService.findAll({ skip, take, search });
  }
}

// Generated schema:
// type Query {
//   users(skip: Int = 0, take: Int = 10, search: String): [User!]!
// }

// GraphQL query:
// query {
//   users(skip: 0, take: 20, search: "John") {
//     id
//     name
//   }
// }
```

### **Query with Input Type**

```typescript
import { InputType, Field, Int } from '@nestjs/graphql';

@InputType()
export class UsersFilterInput {
  @Field({ nullable: true })
  name?: string;

  @Field(() => Int, { nullable: true })
  minAge?: number;

  @Field(() => Int, { nullable: true })
  maxAge?: number;
}

@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  async users(@Args('filter') filter: UsersFilterInput): Promise<User[]> {
    return this.userService.filter(filter);
  }
}

// GraphQL query:
// query {
//   users(filter: { name: "John", minAge: 18, maxAge: 65 }) {
//     id
//     name
//     age
//   }
// }
```

### **Complete CRUD Queries**

```typescript
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Get all users
  @Query(() => [User], { name: 'users' })
  async findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Get single user by ID
  @Query(() => User, { name: 'user', nullable: true })
  async findOne(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Get users by email
  @Query(() => User, { name: 'userByEmail', nullable: true })
  async findByEmail(@Args('email') email: string): Promise<User | null> {
    return this.userService.findByEmail(email);
  }

  // Search users
  @Query(() => [User], { name: 'searchUsers' })
  async search(@Args('query') query: string): Promise<User[]> {
    return this.userService.search(query);
  }

  // Paginated users
  @Query(() => [User])
  async paginatedUsers(
    @Args('page', { type: () => Int, defaultValue: 1 }) page: number,
    @Args('limit', { type: () => Int, defaultValue: 10 }) limit: number,
  ): Promise<User[]> {
    return this.userService.paginate(page, limit);
  }
}
```

### **Error Handling in Queries**

```typescript
import { NotFoundException } from '@nestjs/common';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
}

// GraphQL error response:
// {
//   "errors": [{
//     "message": "User with ID 999 not found",
//     "extensions": { "code": "NOT_FOUND" }
//   }]
// }
```

**Interview Tip**: **`@Query()`** marks a method as **GraphQL query resolver** for **read operations**. Applied to methods in `@Resolver()` class. **Syntax**: `@Query(() => ReturnType)` where ReturnType is `User`, `[User]`, etc. **Options**: `name` (custom query name), `nullable: true` (optional result), `description` (docs), `deprecationReason`. **Arguments**: use `@Args('name')` for simple args or `@Args('input') input: InputType` for complex filters. **Method name** becomes query name unless `name` option provided. **Return**: `Promise<Type>` or `Promise<Type[]>`. Think of queries as **GET endpoints** in REST.

</details>
18. How do you create mutation resolvers using `@Mutation()`?

<details>
<summary><strong>Answer</strong></summary>

**`@Mutation()`** decorator marks a method as **GraphQL mutation resolver**. Applied to methods inside `@Resolver()` class. Takes **return type function** `() => Type` and **options**. Mutation methods handle **write operations** (create/update/delete), equivalent to REST POST/PUT/DELETE. Mutations should have **side effects** and modify data.

### **Basic Mutation**

```typescript
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';
import { CreateUserInput } from './user.input';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }
}

// Generated schema:
// type Mutation {
//   createUser(input: CreateUserInput!): User!
// }

// GraphQL mutation:
// mutation {
//   createUser(input: {
//     name: "John"
//     email: "john@example.com"
//   }) {
//     id
//     name
//     email
//   }
// }
```

### **CRUD Mutations**

```typescript
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // CREATE
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }

  // UPDATE
  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    return this.userService.update(id, input);
  }

  // DELETE
  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    return this.userService.delete(id);
  }
}

// Generated schema:
// type Mutation {
//   createUser(input: CreateUserInput!): User!
//   updateUser(id: String!, input: UpdateUserInput!): User!
//   deleteUser(id: String!): Boolean!
// }
```

### **Mutation Options**

```typescript
@Resolver(() => User)
export class UserResolver {
  // Custom name
  @Mutation(() => User, { name: 'registerUser' })
  async create(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);
  }

  // With description
  @Mutation(() => User, {
    description: 'Update user information',
  })
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    return this.userService.update(id, input);
  }

  // Nullable result
  @Mutation(() => User, { nullable: true })
  async softDeleteUser(@Args('id') id: string): Promise<User | null> {
    return this.userService.softDelete(id);
  }
}

// Generated schema:
// type Mutation {
//   registerUser(input: CreateUserInput!): User!
//   """Update user information"""
//   updateUser(id: String!, input: UpdateUserInput!): User!
//   softDeleteUser(id: String!): User
// }
```

### **Complex Mutations**

```typescript
@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;
}

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;
}

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    // Validation happens automatically
    const hashedPassword = await bcrypt.hash(input.password, 10);
    
    return this.userService.create({
      ...input,
      password: hashedPassword,
    });
  }

  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return this.userService.update(id, input);
  }
}
```

### **Mutation with Multiple Arguments**

```typescript
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async assignRole(
    @Args('userId') userId: string,
    @Args('role') role: string,
  ): Promise<User> {
    return this.userService.assignRole(userId, role);
  }

  @Mutation(() => Boolean)
  async addUserToGroup(
    @Args('userId') userId: string,
    @Args('groupId') groupId: string,
  ): Promise<boolean> {
    return this.userService.addToGroup(userId, groupId);
  }
}

// Generated schema:
// type Mutation {
//   assignRole(userId: String!, role: String!): User!
//   addUserToGroup(userId: String!, groupId: String!): Boolean!
// }
```

### **Mutation Return Types**

```typescript
// Return created object
@Mutation(() => User)
async createUser(@Args('input') input: CreateUserInput): Promise<User> {
  return this.userService.create(input);
}

// Return boolean (success/failure)
@Mutation(() => Boolean)
async deleteUser(@Args('id') id: string): Promise<boolean> {
  const result = await this.userService.delete(id);
  return result.affected > 0;
}

// Return custom response type
@ObjectType()
export class MutationResponse {
  @Field()
  success: boolean;

  @Field()
  message: string;

  @Field(() => User, { nullable: true })
  user?: User;
}

@Mutation(() => MutationResponse)
async createUser(
  @Args('input') input: CreateUserInput,
): Promise<MutationResponse> {
  try {
    const user = await this.userService.create(input);
    return {
      success: true,
      message: 'User created successfully',
      user,
    };
  } catch (error) {
    return {
      success: false,
      message: error.message,
    };
  }
}
```

### **Complete Example**

```typescript
// user.input.ts
@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;
}

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }

  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return this.userService.update(id, input);
  }

  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    const result = await this.userService.delete(id);
    return result.affected > 0;
  }
}

// GraphQL mutations:
// mutation {
//   createUser(input: { name: "John", email: "john@example.com", password: "secret123" }) {
//     id
//     name
//     email
//   }
// }
//
// mutation {
//   updateUser(id: "1", input: { name: "John Smith" }) {
//     id
//     name
//   }
// }
//
// mutation {
//   deleteUser(id: "1")
// }
```

**Interview Tip**: **`@Mutation()`** marks a method as **GraphQL mutation resolver** for **write operations** (create/update/delete). Applied to methods in `@Resolver()` class. **Syntax**: `@Mutation(() => ReturnType)` where ReturnType is `User`, `Boolean`, etc. **Options**: `name` (custom mutation name), `nullable`, `description`. **Arguments**: typically `@Args('input') input: CreateUserInput` for complex data. **CRUD pattern**: createUser (return created object), updateUser (return updated object), deleteUser (return Boolean). Think of mutations as **POST/PUT/DELETE endpoints** in REST. **Validation**: combine with `class-validator` on InputTypes for automatic validation.

</details>
19. How do you define resolver return types?

<details>
<summary><strong>Answer</strong></summary>

**Return types** define what data the resolver returns. Specified in **`@Query()`/`@Mutation()`** decorator as **type function** `() => Type`. Can return **single object** (`User`), **array** (`[User]`), **scalar** (`String`, `Int`, `Boolean`), or **nullable** (`{ nullable: true }`). TypeScript return type must match GraphQL return type. Use `Promise<Type>` for async resolvers.

### **Single Object Return**

```typescript
import { Resolver, Query } from '@nestjs/graphql';
import { User } from './user.model';

@Resolver(() => User)
export class UserResolver {
  // Returns single User
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }
}

// Generated schema:
// type Query {
//   user(id: String!): User
// }
```

### **Array Return**

```typescript
@Resolver(() => User)
export class UserResolver {
  // Returns array of Users
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Returns nullable array
  @Query(() => [User], { nullable: true })
  async optionalUsers(): Promise<User[] | null> {
    return this.userService.findAllOrNull();
  }

  // Returns array with nullable items
  @Query(() => [User], { nullable: 'items' })
  async usersWithNulls(): Promise<(User | null)[]> {
    return this.userService.findAllWithDeleted();
  }
}

// Generated schema:
// type Query {
//   users: [User!]!           # Required array, required items
//   optionalUsers: [User!]    # Optional array, required items
//   usersWithNulls: [User]!   # Required array, optional items
// }
```

### **Scalar Return Types**

```typescript
import { Resolver, Mutation, Int } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  // Boolean return
  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    const result = await this.userService.delete(id);
    return result.affected > 0;
  }

  // String return
  @Query(() => String)
  async serverStatus(): Promise<string> {
    return 'Server is running';
  }

  // Int return
  @Query(() => Int)
  async userCount(): Promise<number> {
    return this.userService.count();
  }

  // Float return (default for number)
  @Query(() => Number)  // or @Query(() => Float)
  async averageAge(): Promise<number> {
    return this.userService.getAverageAge();
  }
}

// Generated schema:
// type Query {
//   serverStatus: String!
//   userCount: Int!
//   averageAge: Float!
// }
// 
// type Mutation {
//   deleteUser(id: String!): Boolean!
// }
```

### **Custom Object Type Return**

```typescript
// Define custom response type
@ObjectType()
export class PaginatedUsers {
  @Field(() => [User])
  users: User[];

  @Field(() => Int)
  total: number;

  @Field(() => Int)
  page: number;

  @Field(() => Int)
  totalPages: number;
}

@Resolver(() => User)
export class UserResolver {
  @Query(() => PaginatedUsers)
  async paginatedUsers(
    @Args('page', { type: () => Int }) page: number,
    @Args('limit', { type: () => Int }) limit: number,
  ): Promise<PaginatedUsers> {
    const [users, total] = await this.userService.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      users,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}

// Generated schema:
// type PaginatedUsers {
//   users: [User!]!
//   total: Int!
//   page: Int!
//   totalPages: Int!
// }
// 
// type Query {
//   paginatedUsers(page: Int!, limit: Int!): PaginatedUsers!
// }
```

### **Union Types**

```typescript
import { createUnionType } from '@nestjs/graphql';

// Define union type
const SearchResult = createUnionType({
  name: 'SearchResult',
  types: () => [User, Post, Comment] as const,
  resolveType(value) {
    if (value.email) return User;
    if (value.title) return Post;
    if (value.text) return Comment;
    return null;
  },
});

@Resolver()
export class SearchResolver {
  @Query(() => [SearchResult])
  async search(@Args('query') query: string): Promise<typeof SearchResult[]> {
    return this.searchService.search(query);
  }
}

// Generated schema:
// union SearchResult = User | Post | Comment
// 
// type Query {
//   search(query: String!): [SearchResult!]!
// }
```

### **Interface Return Types**

```typescript
import { InterfaceType, Field, ID } from '@nestjs/graphql';

// Define interface
@InterfaceType()
abstract class Node {
  @Field(() => ID)
  id: string;

  @Field()
  createdAt: Date;
}

// Implement interface
@ObjectType({ implements: () => [Node] })
export class User extends Node {
  @Field()
  name: string;
}

@ObjectType({ implements: () => [Node] })
export class Post extends Node {
  @Field()
  title: string;
}

@Resolver()
export class NodeResolver {
  @Query(() => Node, { nullable: true })
  async node(@Args('id') id: string): Promise<typeof Node | null> {
    // Return User or Post (both implement Node)
    return this.nodeService.findById(id);
  }
}

// Generated schema:
// interface Node {
//   id: ID!
//   createdAt: DateTime!
// }
// 
// type Query {
//   node(id: String!): Node
// }
```

### **Nullable Return Types**

```typescript
@Resolver(() => User)
export class UserResolver {
  // Non-nullable (default) - throws error if null
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return user;
  }

  // Nullable - can return null
  @Query(() => User, { nullable: true })
  async optionalUser(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Nullable array
  @Query(() => [User], { nullable: true })
  async optionalUsers(): Promise<User[] | null> {
    return this.userService.findAll();
  }

  // Array with nullable items
  @Query(() => [User], { nullable: 'items' })
  async usersWithNulls(): Promise<(User | null)[]> {
    return this.userService.findAllWithDeleted();
  }

  // Both array and items nullable
  @Query(() => [User], { nullable: 'itemsAndList' })
  async fullyOptional(): Promise<(User | null)[] | null> {
    return this.userService.findOptional();
  }
}

// Generated schema:
// type Query {
//   user(id: String!): User!              # Non-nullable
//   optionalUser(id: String!): User       # Nullable
//   optionalUsers: [User!]                # Nullable array
//   usersWithNulls: [User]!               # Nullable items
//   fullyOptional: [User]                 # Both nullable
// }
```

### **Generic Response Pattern**

```typescript
// Generic mutation response
@ObjectType()
export class MutationResponse {
  @Field()
  success: boolean;

  @Field()
  message: string;
}

@ObjectType()
export class CreateUserResponse extends MutationResponse {
  @Field(() => User, { nullable: true })
  user?: User;
}

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => CreateUserResponse)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<CreateUserResponse> {
    try {
      const user = await this.userService.create(input);
      return {
        success: true,
        message: 'User created successfully',
        user,
      };
    } catch (error) {
      return {
        success: false,
        message: error.message,
      };
    }
  }
}
```

**Interview Tip**: **Return types** specified in `@Query()`/`@Mutation()` decorator as **type function** `() => Type`. **Single**: `() => User` for `User!`, **Array**: `() => [User]` for `[User!]!`, **Nullable**: `{ nullable: true }` for `User`, **Scalars**: `() => Boolean`, `() => Int`, `() => String`. **TypeScript return** must match: `Promise<User>`, `Promise<User[]>`, `Promise<boolean>`. **Complex**: custom `@ObjectType()` for paginated/custom responses. **Arrays**: `nullable: true` = optional array, `nullable: 'items'` = optional items, `nullable: 'itemsAndList'` = both optional.

</details>
20. How do you access arguments using `@Args()`?

<details>
<summary><strong>Answer</strong></summary>

**`@Args()`** decorator extracts **arguments from GraphQL query/mutation**. Applied to method parameters in resolvers. **Syntax**: `@Args('name')` for single arg, `@Args('name', options)` with type/default/validation, `@Args()` for all args object. Arguments map to GraphQL schema arguments, validated automatically with `class-validator`.

### **Basic Argument**

```typescript
import { Resolver, Query, Args } from '@nestjs/graphql';\n\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => User, { nullable: true })\n  async user(\n    @Args('id') id: string,  // Extract 'id' argument\n  ): Promise<User | null> {\n    return this.userService.findById(id);\n  }\n}\n\n// Generated schema:\n// type Query {\n//   user(id: String!): User\n// }\n\n// GraphQL query:\n// query {\n//   user(id: \"1\") {\n//     id\n//     name\n//   }\n// }\n```\n\n### **Multiple Arguments**\n\n```typescript\nimport { Resolver, Query, Args, Int } from '@nestjs/graphql';\n\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => [User])\n  async users(\n    @Args('skip', { type: () => Int, defaultValue: 0 }) skip: number,\n    @Args('take', { type: () => Int, defaultValue: 10 }) take: number,\n    @Args('search', { nullable: true }) search?: string,\n  ): Promise<User[]> {\n    return this.userService.findAll({ skip, take, search });\n  }\n}\n\n// Generated schema:\n// type Query {\n//   users(skip: Int = 0, take: Int = 10, search: String): [User!]!\n// }\n\n// GraphQL query:\n// query {\n//   users(skip: 0, take: 20, search: \"John\") {\n//     id\n//     name\n//   }\n// }\n```\n\n### **Explicit Type**\n\n```typescript\nimport { Resolver, Mutation, Args, Int, Float } from '@nestjs/graphql';\n\n@Resolver(() => Product)\nexport class ProductResolver {\n  @Mutation(() => Product)\n  async updatePrice(\n    @Args('id') id: string,\n    @Args('price', { type: () => Float }) price: number,  // Explicit Float\n    @Args('quantity', { type: () => Int }) quantity: number,  // Explicit Int\n  ): Promise<Product> {\n    return this.productService.updatePrice(id, price, quantity);\n  }\n}\n\n// Generated schema:\n// type Mutation {\n//   updatePrice(id: String!, price: Float!, quantity: Int!): Product!\n// }\n```\n\n### **Input Type Argument**\n\n```typescript\n// user.input.ts\n@InputType()\nexport class CreateUserInput {\n  @Field()\n  @MinLength(2)\n  name: string;\n\n  @Field()\n  @IsEmail()\n  email: string;\n\n  @Field()\n  @MinLength(8)\n  password: string;\n}\n\n// user.resolver.ts\n@Resolver(() => User)\nexport class UserResolver {\n  @Mutation(() => User)\n  async createUser(\n    @Args('input') input: CreateUserInput,  // Input type argument\n  ): Promise<User> {\n    // input.name, input.email, input.password are validated\n    return this.userService.create(input);\n  }\n}\n\n// Generated schema:\n// type Mutation {\n//   createUser(input: CreateUserInput!): User!\n// }\n\n// GraphQL mutation:\n// mutation {\n//   createUser(input: {\n//     name: \"John\"\n//     email: \"john@example.com\"\n//     password: \"secret123\"\n//   }) {\n//     id\n//     name\n//   }\n// }\n```\n\n### **Optional Arguments**\n\n```typescript\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => [User])\n  async users(\n    @Args('name', { nullable: true }) name?: string,\n    @Args('email', { nullable: true }) email?: string,\n    @Args('minAge', { type: () => Int, nullable: true }) minAge?: number,\n  ): Promise<User[]> {\n    return this.userService.filter({ name, email, minAge });\n  }\n}\n\n// Generated schema:\n// type Query {\n//   users(name: String, email: String, minAge: Int): [User!]!\n// }\n\n// All arguments are optional\n// query { users { id name } }  // No arguments\n// query { users(name: \"John\") { id name } }  // Only name\n```\n\n### **Default Values**\n\n```typescript\n@Resolver(() => Post)\nexport class PostResolver {\n  @Query(() => [Post])\n  async posts(\n    @Args('page', { type: () => Int, defaultValue: 1 }) page: number,\n    @Args('limit', { type: () => Int, defaultValue: 10 }) limit: number,\n    @Args('sortBy', { defaultValue: 'createdAt' }) sortBy: string,\n    @Args('order', { defaultValue: 'DESC' }) order: 'ASC' | 'DESC',\n  ): Promise<Post[]> {\n    return this.postService.findAll({ page, limit, sortBy, order });\n  }\n}\n\n// Generated schema:\n// type Query {\n//   posts(\n//     page: Int = 1\n//     limit: Int = 10\n//     sortBy: String = \"createdAt\"\n//     order: String = \"DESC\"\n//   ): [Post!]!\n// }\n\n// GraphQL query (all defaults):\n// query { posts { id title } }\n// GraphQL query (override defaults):\n// query { posts(page: 2, limit: 20) { id title } }\n```\n\n### **Array Arguments**\n\n```typescript\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => [User])\n  async usersByIds(\n    @Args('ids', { type: () => [String] }) ids: string[],\n  ): Promise<User[]> {\n    return this.userService.findByIds(ids);\n  }\n\n  @Mutation(() => [Post])\n  async createPosts(\n    @Args('titles', { type: () => [String] }) titles: string[],\n  ): Promise<Post[]> {\n    return this.postService.createMultiple(titles);\n  }\n}\n\n// Generated schema:\n// type Query {\n//   usersByIds(ids: [String!]!): [User!]!\n// }\n// \n// type Mutation {\n//   createPosts(titles: [String!]!): [Post!]!\n// }\n\n// GraphQL query:\n// query {\n//   usersByIds(ids: [\"1\", \"2\", \"3\"]) {\n//     id\n//     name\n//   }\n// }\n```\n\n### **Validation on Arguments**\n\n```typescript\nimport { Min, Max } from 'class-validator';\nimport { ArgsType, Field, Int } from '@nestjs/graphql';\n\n// Define args class for validation\n@ArgsType()\nclass PaginationArgs {\n  @Field(() => Int, { defaultValue: 1 })\n  @Min(1)\n  page: number;\n\n  @Field(() => Int, { defaultValue: 10 })\n  @Min(1)\n  @Max(100)\n  limit: number;\n}\n\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => [User])\n  async users(@Args() args: PaginationArgs): Promise<User[]> {\n    // args.page and args.limit are validated\n    return this.userService.paginate(args.page, args.limit);\n  }\n}\n\n// GraphQL query:\n// query { users(page: 1, limit: 10) { id name } }\n// query { users(page: 0, limit: 10) }  // Error: page must be >= 1\n// query { users(page: 1, limit: 200) }  // Error: limit must be <= 100\n```\n\n### **All Arguments Object**\n\n```typescript\nimport { ArgsType, Field, Int } from '@nestjs/graphql';\n\n@ArgsType()\nexport class GetUsersArgs {\n  @Field({ nullable: true })\n  name?: string;\n\n  @Field(() => Int, { nullable: true })\n  age?: number;\n\n  @Field(() => Int, { defaultValue: 0 })\n  skip: number;\n\n  @Field(() => Int, { defaultValue: 10 })\n  take: number;\n}\n\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => [User])\n  async users(@Args() args: GetUsersArgs): Promise<User[]> {\n    // Access: args.name, args.age, args.skip, args.take\n    return this.userService.findAll(args);\n  }\n}\n\n// Generated schema:\n// type Query {\n//   users(name: String, age: Int, skip: Int = 0, take: Int = 10): [User!]!\n// }\n```\n\n### **Custom Argument Name**\n\n```typescript\n@Resolver(() => User)\nexport class UserResolver {\n  @Query(() => User, { nullable: true })\n  async user(\n    @Args('userId') id: string,  // GraphQL: userId, TypeScript: id\n  ): Promise<User | null> {\n    return this.userService.findById(id);\n  }\n}\n\n// Generated schema:\n// type Query {\n//   user(userId: String!): User\n// }\n\n// GraphQL query:\n// query {\n//   user(userId: \"1\") {  // Note: userId, not id\n//     id\n//     name\n//   }\n// }\n```\n\n### **Complete Example**\n\n```typescript\n// Pagination args\n@ArgsType()\nclass PaginationArgs {\n  @Field(() => Int, { defaultValue: 1 })\n  @Min(1)\n  page: number;\n\n  @Field(() => Int, { defaultValue: 10 })\n  @Min(1)\n  @Max(100)\n  limit: number;\n}\n\n// Filter input\n@InputType()\nclass UserFilterInput {\n  @Field({ nullable: true })\n  name?: string;\n\n  @Field(() => Int, { nullable: true })\n  @Min(0)\n  minAge?: number;\n\n  @Field(() => Int, { nullable: true })\n  @Max(150)\n  maxAge?: number;\n}\n\n@Resolver(() => User)\nexport class UserResolver {\n  // Single argument\n  @Query(() => User, { nullable: true })\n  async user(@Args('id') id: string): Promise<User | null> {\n    return this.userService.findById(id);\n  }\n\n  // Multiple arguments\n  @Query(() => [User])\n  async users(\n    @Args() pagination: PaginationArgs,\n    @Args('filter', { nullable: true }) filter?: UserFilterInput,\n  ): Promise<User[]> {\n    return this.userService.findAll({ pagination, filter });\n  }\n\n  // Input type argument\n  @Mutation(() => User)\n  async createUser(\n    @Args('input') input: CreateUserInput,\n  ): Promise<User> {\n    return this.userService.create(input);\n  }\n}\n\n// GraphQL queries:\n// query { user(id: \"1\") { id name } }\n// query { users(page: 1, limit: 10) { id name } }\n// query { users(page: 1, limit: 10, filter: { name: \"John\", minAge: 18 }) { id name } }\n// mutation { createUser(input: { name: \"John\", email: \"john@example.com\" }) { id } }\n```\n\n**Interview Tip**: **`@Args()`** extracts **arguments from GraphQL query/mutation**. **Syntax**: `@Args('name')` for single arg, `@Args('name', options)` with type/default/validation. **Options**: `type: () => Int` (explicit type), `defaultValue: 10` (default), `nullable: true` (optional). **Input types**: `@Args('input') input: CreateUserInput` for complex data. **Validation**: use `@ArgsType()` class with `class-validator` decorators for automatic validation. **Arrays**: `@Args('ids', { type: () => [String] })`. **All args**: `@Args() args: ArgsClass` to get all arguments as object. **TypeScript type** must match GraphQL type.\n\n</details>

---

## Queries

21. How do you implement a simple Query?

<details>
<summary><strong>Answer</strong></summary>

Implement a simple query by: (1) Create **resolver class** with `@Resolver()`, (2) Define **query method** with `@Query(() => ReturnType)`, (3) Return data from **service layer**. Query fetches data from database/API, no side effects. Register resolver in module **providers**. Query becomes available in GraphQL schema.

### **Basic Query - Get All**

```typescript
// user.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;
}

// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.userRepo.find();
  }
}

// user.resolver.ts
import { Resolver, Query } from '@nestjs/graphql';
import { User } from './user.model';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => [User], { name: 'users' })
  async findAll(): Promise<User[]> {
    return this.userService.findAll();
  }
}

// user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UserResolver } from './user.resolver';
import { UserService } from './user.service';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UserResolver, UserService],
})
export class UserModule {}

// Generated schema:
// type User {
//   id: ID!
//   name: String!
//   email: String!
// }
// 
// type Query {
//   users: [User!]!
// }

// GraphQL query:
// query {
//   users {
//     id
//     name
//     email
//   }
// }
```

### **Simple Query - Get By ID**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  async findById(id: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { id } });
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  @Query(() => User, { name: 'user', nullable: true })
  async findOne(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }
}

// Generated schema:
// type Query {
//   user(id: String!): User
// }

// GraphQL query:
// query {
//   user(id: "1") {
//     id
//     name
//     email
//   }
// }
```

### **Multiple Queries**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  async findAll(): Promise<User[]> {
    return this.userRepo.find();
  }

  async findById(id: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { email } });
  }

  async count(): Promise<number> {
    return this.userRepo.count();
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Get all users
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Get user by ID
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Get user by email
  @Query(() => User, { nullable: true })
  async userByEmail(@Args('email') email: string): Promise<User | null> {
    return this.userService.findByEmail(email);
  }

  // Get user count
  @Query(() => Int)
  async userCount(): Promise<number> {
    return this.userService.count();
  }
}

// Generated schema:
// type Query {
//   users: [User!]!
//   user(id: String!): User
//   userByEmail(email: String!): User
//   userCount: Int!
// }
```

### **Query with Error Handling**

```typescript
import { NotFoundException } from '@nestjs/common';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
}

// GraphQL error:
// {
//   "errors": [{
//     "message": "User with ID 999 not found",
//     "extensions": { "code": "NOT_FOUND" }
//   }]
// }
```

### **Complete Example with TypeORM**

```typescript
// user.entity.ts (TypeORM entity)
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { ObjectType, Field, ID } from '@nestjs/graphql';

@Entity()
@ObjectType()  // Both TypeORM and GraphQL decorators
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)
  id: string;

  @Column()
  @Field()
  name: string;

  @Column({ unique: true })
  @Field()
  email: string;

  @Column()
  password: string;  // No @Field() - hidden from GraphQL

  @Column({ default: true })
  @Field()
  isActive: boolean;
}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.userRepo.find({ where: { isActive: true } });
  }

  async findById(id: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { id, isActive: true } });
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => [User], { description: 'Get all active users' })
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Query(() => User, { nullable: true, description: 'Get user by ID' })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }
}

// user.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UserResolver, UserService],
  exports: [UserService],
})
export class UserModule {}

// app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'testdb',
      entities: [User],
      synchronize: true,
    }),
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      playground: true,
    }),
    UserModule,
  ],
})
export class AppModule {}
```

**Interview Tip**: Implement query: (1) **Model** with `@ObjectType()` and `@Field()`, (2) **Service** with database logic (`findAll()`, `findById()`), (3) **Resolver** with `@Resolver(() => User)` and `@Query(() => [User])` calling service methods, (4) **Register** resolver in module providers. **Query method** returns `Promise<Type>` or `Promise<Type[]>`, delegates business logic to service. **Error handling**: throw NestJS exceptions (NotFoundException, BadRequestException) → converted to GraphQL errors automatically. **Pattern**: Resolver (thin, routing) → Service (business logic) → Repository (database).

</details>

---

22. How do you pass arguments to queries?

<details>
<summary><strong>Answer</strong></summary>

**Passing arguments to GraphQL queries** allows clients to filter, search, or specify which data they want. In NestJS, you use the `@Args()` decorator to extract arguments from the GraphQL query and pass them to resolver methods. Arguments can be **scalars** (string, number), **objects** (input types), or **arrays**, and support **validation**, **optional/required flags**, and **default values**.

### **Basic Usage with `@Args()`**

```typescript
// users.resolver.ts
import { Resolver, Query, Args } from '@nestjs/graphql';
import { User } from './models/user.model';
import { UsersService } from './users.service';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  // Single argument - user by ID
  @Query(() => User, { nullable: true })
  async user(
    @Args('id') id: string, // Simple string argument
  ): Promise<User | null> {
    return this.usersService.findById(id);
  }

  // Multiple arguments
  @Query(() => [User])
  async users(
    @Args('role', { nullable: true }) role?: string, // Optional argument
    @Args('limit', { defaultValue: 10 }) limit?: number, // Default value
  ): Promise<User[]> {
    return this.usersService.findAll({ role, limit });
  }
}
```

**GraphQL Query:**
```graphql
# Single argument
query {
  user(id: "123") {
    id
    name
    email
  }
}

# Multiple arguments
query {
  users(role: "ADMIN", limit: 5) {
    id
    name
    role
  }
}
```

### **Advanced Examples**

#### **1. Using Input Types for Complex Arguments**

```typescript
// dtos/filter-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsOptional, IsEmail, IsEnum, MinLength } from 'class-validator';

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
}

@InputType()
export class FilterUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  @Field(() => UserRole, { nullable: true })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @Field({ nullable: true })
  @IsOptional()
  isActive?: boolean;

  @Field({ nullable: true })
  @IsOptional()
  createdAfter?: Date;
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User], { description: 'Search users with filters' })
  async searchUsers(
    @Args('filter') filter: FilterUserInput, // Object argument
  ): Promise<User[]> {
    return this.usersService.search(filter);
  }
}
```

**GraphQL Query:**
```graphql
query {
  searchUsers(filter: {
    role: ADMIN
    isActive: true
    createdAfter: "2024-01-01T00:00:00Z"
  }) {
    id
    name
    role
  }
}
```

#### **2. Array Arguments**

```typescript
// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  // Fetch multiple users by IDs
  @Query(() => [User])
  async usersByIds(
    @Args('ids', { type: () => [String] }) ids: string[], // Array argument
  ): Promise<User[]> {
    return this.usersService.findByIds(ids);
  }

  // Search with multiple roles
  @Query(() => [User])
  async usersByRoles(
    @Args('roles', { type: () => [UserRole] }) roles: UserRole[],
  ): Promise<User[]> {
    return this.usersService.findByRoles(roles);
  }
}
```

**GraphQL Query:**
```graphql
query {
  usersByIds(ids: ["123", "456", "789"]) {
    id
    name
  }

  usersByRoles(roles: [ADMIN, MODERATOR]) {
    id
    name
    role
  }
}
```

#### **3. Pagination and Sorting Arguments**

```typescript
// dtos/pagination.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { Min, Max } from 'class-validator';

@InputType()
export class PaginationInput {
  @Field(() => Int, { defaultValue: 0 })
  @Min(0)
  offset: number = 0;

  @Field(() => Int, { defaultValue: 20 })
  @Min(1)
  @Max(100)
  limit: number = 20;
}

@InputType()
export class SortInput {
  @Field({ defaultValue: 'createdAt' })
  field: string;

  @Field({ defaultValue: 'DESC' })
  order: 'ASC' | 'DESC';
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User])
  async users(
    @Args('pagination', { nullable: true }) pagination?: PaginationInput,
    @Args('sort', { nullable: true }) sort?: SortInput,
    @Args('filter', { nullable: true }) filter?: FilterUserInput,
  ): Promise<User[]> {
    return this.usersService.findAll({
      ...pagination,
      ...sort,
      ...filter,
    });
  }
}
```

**GraphQL Query:**
```graphql
query {
  users(
    pagination: { offset: 0, limit: 10 }
    sort: { field: "name", order: ASC }
    filter: { role: ADMIN }
  ) {
    id
    name
    email
  }
}
```

### **Production-Grade Implementation**

```typescript
// ============================================
// 1. MODELS
// ============================================
// models/user.model.ts
import { ObjectType, Field, ID, registerEnumType } from '@nestjs/graphql';

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
}

export enum UserStatus {
  ACTIVE = 'ACTIVE',
  INACTIVE = 'INACTIVE',
  SUSPENDED = 'SUSPENDED',
}

registerEnumType(UserRole, { name: 'UserRole' });
registerEnumType(UserStatus, { name: 'UserStatus' });

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  email: string;

  @Field()
  name: string;

  @Field(() => UserRole)
  role: UserRole;

  @Field(() => UserStatus)
  status: UserStatus;

  @Field()
  createdAt: Date;

  @Field()
  updatedAt: Date;
}

// ============================================
// 2. INPUT TYPES & DTOs
// ============================================
// dtos/user-query.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { 
  IsOptional, 
  IsEmail, 
  IsEnum, 
  Min, 
  Max, 
  MinLength,
  IsDateString 
} from 'class-validator';

@InputType()
export class UserFilterInput {
  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  @Field(() => UserRole, { nullable: true })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @Field(() => UserStatus, { nullable: true })
  @IsOptional()
  @IsEnum(UserStatus)
  status?: UserStatus;

  @Field({ nullable: true })
  @IsOptional()
  @IsDateString()
  createdAfter?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsDateString()
  createdBefore?: string;

  @Field({ nullable: true })
  @IsOptional()
  ageMin?: number;

  @Field({ nullable: true })
  @IsOptional()
  ageMax?: number;
}

@InputType()
export class PaginationInput {
  @Field(() => Int, { defaultValue: 0, description: 'Number of records to skip' })
  @Min(0)
  offset: number = 0;

  @Field(() => Int, { defaultValue: 20, description: 'Max records to return' })
  @Min(1)
  @Max(100)
  limit: number = 20;
}

@InputType()
export class SortInput {
  @Field({ defaultValue: 'createdAt' })
  field: string;

  @Field({ defaultValue: 'DESC' })
  order: 'ASC' | 'DESC';
}

@InputType()
export class SearchUsersInput {
  @Field(() => UserFilterInput, { nullable: true })
  filter?: UserFilterInput;

  @Field(() => PaginationInput, { nullable: true })
  pagination?: PaginationInput;

  @Field(() => SortInput, { nullable: true })
  sort?: SortInput;
}

// ============================================
// 3. SERVICE
// ============================================
// users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, FindOptionsWhere, Between, MoreThan, LessThan } from 'typeorm';
import { User, UserRole, UserStatus } from './models/user.model';
import { UserFilterInput, PaginationInput, SortInput } from './dtos/user-query.input';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  /**
   * Find user by ID
   * @throws NotFoundException if user not found
   */
  async findById(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User with ID "${id}" not found`);
    }
    return user;
  }

  /**
   * Find multiple users by IDs
   * Filters out non-existent IDs (no error thrown)
   */
  async findByIds(ids: string[]): Promise<User[]> {
    if (!ids || ids.length === 0) {
      return [];
    }
    return this.userRepository.findByIds(ids);
  }

  /**
   * Find users by roles
   */
  async findByRoles(roles: UserRole[]): Promise<User[]> {
    if (!roles || roles.length === 0) {
      return [];
    }
    return this.userRepository.find({
      where: { role: In(roles) },
    });
  }

  /**
   * Search users with filters, pagination, and sorting
   */
  async search(
    filter?: UserFilterInput,
    pagination?: PaginationInput,
    sort?: SortInput,
  ): Promise<User[]> {
    const query = this.userRepository.createQueryBuilder('user');

    // Apply filters
    if (filter) {
      if (filter.name) {
        query.andWhere('user.name ILIKE :name', { name: `%${filter.name}%` });
      }
      if (filter.email) {
        query.andWhere('user.email ILIKE :email', { email: `%${filter.email}%` });
      }
      if (filter.role) {
        query.andWhere('user.role = :role', { role: filter.role });
      }
      if (filter.status) {
        query.andWhere('user.status = :status', { status: filter.status });
      }
      if (filter.createdAfter) {
        query.andWhere('user.createdAt >= :createdAfter', { 
          createdAfter: new Date(filter.createdAfter) 
        });
      }
      if (filter.createdBefore) {
        query.andWhere('user.createdAt <= :createdBefore', { 
          createdBefore: new Date(filter.createdBefore) 
        });
      }
      if (filter.ageMin !== undefined) {
        query.andWhere('user.age >= :ageMin', { ageMin: filter.ageMin });
      }
      if (filter.ageMax !== undefined) {
        query.andWhere('user.age <= :ageMax', { ageMax: filter.ageMax });
      }
    }

    // Apply sorting
    if (sort) {
      query.orderBy(`user.${sort.field}`, sort.order);
    }

    // Apply pagination
    if (pagination) {
      query.skip(pagination.offset).take(pagination.limit);
    }

    return query.getMany();
  }

  /**
   * Count users matching filters (for pagination metadata)
   */
  async count(filter?: UserFilterInput): Promise<number> {
    const query = this.userRepository.createQueryBuilder('user');

    // Apply same filters as search()
    if (filter) {
      if (filter.name) {
        query.andWhere('user.name ILIKE :name', { name: `%${filter.name}%` });
      }
      // ... other filters ...
    }

    return query.getCount();
  }
}

// ============================================
// 4. RESOLVER
// ============================================
// users.resolver.ts
import { 
  Resolver, 
  Query, 
  Args, 
  ID,
  Int,
} from '@nestjs/graphql';
import { UsePipes, ValidationPipe } from '@nestjs/common';
import { User, UserRole } from './models/user.model';
import { UsersService } from './users.service';
import { 
  UserFilterInput, 
  PaginationInput, 
  SortInput,
  SearchUsersInput 
} from './dtos/user-query.input';

@Resolver(() => User)
@UsePipes(new ValidationPipe({ transform: true })) // Enable validation
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  /**
   * Get single user by ID
   * @throws NotFoundException if not found
   */
  @Query(() => User, { 
    description: 'Get user by ID',
    nullable: false, // Will throw error if not found
  })
  async user(
    @Args('id', { type: () => ID, description: 'User ID' }) id: string,
  ): Promise<User> {
    return this.usersService.findById(id);
  }

  /**
   * Get multiple users by IDs
   * Returns only existing users (filters out non-existent)
   */
  @Query(() => [User], { 
    description: 'Get multiple users by IDs' 
  })
  async usersByIds(
    @Args('ids', { type: () => [ID], description: 'Array of user IDs' }) 
    ids: string[],
  ): Promise<User[]> {
    return this.usersService.findByIds(ids);
  }

  /**
   * Get users by roles
   */
  @Query(() => [User], { 
    description: 'Get users by roles' 
  })
  async usersByRoles(
    @Args('roles', { type: () => [UserRole], description: 'Array of roles' }) 
    roles: UserRole[],
  ): Promise<User[]> {
    return this.usersService.findByRoles(roles);
  }

  /**
   * Search users with filters, pagination, and sorting
   * All arguments optional
   */
  @Query(() => [User], { 
    description: 'Search users with filters, pagination, and sorting' 
  })
  async searchUsers(
    @Args('filter', { nullable: true }) filter?: UserFilterInput,
    @Args('pagination', { nullable: true }) pagination?: PaginationInput,
    @Args('sort', { nullable: true }) sort?: SortInput,
  ): Promise<User[]> {
    return this.usersService.search(filter, pagination, sort);
  }

  /**
   * Alternative: Single input object approach
   */
  @Query(() => [User], { 
    description: 'Search users (single input object)' 
  })
  async users(
    @Args('input', { nullable: true }) input?: SearchUsersInput,
  ): Promise<User[]> {
    return this.usersService.search(
      input?.filter,
      input?.pagination,
      input?.sort,
    );
  }

  /**
   * Count users matching filter
   */
  @Query(() => Int, { 
    description: 'Count users matching filter' 
  })
  async usersCount(
    @Args('filter', { nullable: true }) filter?: UserFilterInput,
  ): Promise<number> {
    return this.usersService.count(filter);
  }
}

// ============================================
// 5. MODULE
// ============================================
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './models/user.model';
import { UsersService } from './users.service';
import { UsersResolver } from './users.resolver';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService, UsersResolver],
  exports: [UsersService],
})
export class UsersModule {}
```

### **GraphQL Queries (Client Usage)**

```graphql
# ============================================
# 1. Simple argument
# ============================================
query GetUser {
  user(id: "123") {
    id
    name
    email
  }
}

# ============================================
# 2. Multiple IDs
# ============================================
query GetMultipleUsers {
  usersByIds(ids: ["123", "456", "789"]) {
    id
    name
  }
}

# ============================================
# 3. Filter by roles
# ============================================
query GetAdmins {
  usersByRoles(roles: [ADMIN, MODERATOR]) {
    id
    name
    role
  }
}

# ============================================
# 4. Complex search
# ============================================
query SearchUsers {
  searchUsers(
    filter: {
      role: ADMIN
      status: ACTIVE
      createdAfter: "2024-01-01T00:00:00Z"
      name: "John"
    }
    pagination: {
      offset: 0
      limit: 10
    }
    sort: {
      field: "name"
      order: ASC
    }
  ) {
    id
    name
    email
    role
    status
    createdAt
  }
}

# ============================================
# 5. Single input object approach
# ============================================
query GetUsers {
  users(input: {
    filter: { role: USER, status: ACTIVE }
    pagination: { offset: 0, limit: 20 }
    sort: { field: "createdAt", order: DESC }
  }) {
    id
    name
  }
}

# ============================================
# 6. Count with filter
# ============================================
query CountActiveAdmins {
  usersCount(filter: { role: ADMIN, status: ACTIVE })
}

# ============================================
# 7. Using variables
# ============================================
query SearchUsers($filter: UserFilterInput, $pagination: PaginationInput) {
  searchUsers(filter: $filter, pagination: $pagination) {
    id
    name
  }
}

# Variables:
# {
#   "filter": { "role": "ADMIN" },
#   "pagination": { "offset": 0, "limit": 10 }
# }
```

### **Error Handling**

```typescript
// users.resolver.ts
import { NotFoundException, BadRequestException } from '@nestjs/common';

@Resolver(() => User)
export class UsersResolver {
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    // Validate ID format
    if (!id || id.trim() === '') {
      throw new BadRequestException('User ID is required');
    }

    // Service throws NotFoundException if not found
    return this.usersService.findById(id);
  }

  @Query(() => [User])
  async searchUsers(
    @Args('filter', { nullable: true }) filter?: UserFilterInput,
    @Args('pagination', { nullable: true }) pagination?: PaginationInput,
  ): Promise<User[]> {
    // Validate pagination limits
    if (pagination && pagination.limit > 100) {
      throw new BadRequestException('Limit cannot exceed 100');
    }

    return this.usersService.search(filter, pagination);
  }
}
```

**GraphQL Error Response:**
```json
{
  "errors": [
    {
      "message": "User with ID \"999\" not found",
      "extensions": {
        "code": "NOT_FOUND",
        "exception": {
          "stacktrace": [...]
        }
      }
    }
  ],
  "data": {
    "user": null
  }
}
```

### **Best Practices**

1. **Use Input Types for Complex Arguments**: Group related arguments into `@InputType()` classes for better organization and reusability
2. **Validate Arguments**: Use `class-validator` decorators on input types and enable `ValidationPipe`
3. **Set Defaults**: Use `defaultValue` in `@Args()` or `@Field()` for optional arguments
4. **Use Enums**: Define enums with `registerEnumType()` for type-safe string constants
5. **Nullable vs Required**: Mark arguments as `nullable: true` for optional, omit for required
6. **Use Descriptive Names**: Name arguments clearly (`filter`, `pagination`, `sort` vs generic `args`)
7. **Limit Pagination**: Set max limits (e.g., 100) to prevent performance issues
8. **Type Safety**: Always specify types explicitly: `type: () => [String]`, `type: () => ID`
9. **Document Arguments**: Add `description` to `@Args()` and `@Field()` for auto-generated docs
10. **Single vs Multiple Arguments**: For 1-2 args use separate `@Args()`, for 3+ use input object

**Interview Tip**: Pass arguments using `@Args('name')` for scalars or `@Args('input') input: InputType` for objects. Define **Input Types** with `@InputType()` for complex arguments, validate with `class-validator`. Use `nullable: true` for optional, `defaultValue` for defaults. **Pattern**: Simple args → separate `@Args()`, complex/many args → single Input Type object. Enable `ValidationPipe` globally or per-resolver. **Best practice**: Group related filters into Input Types (FilterInput, PaginationInput, SortInput), set max limits on pagination, use enums for constants, always validate. For production: validate IDs, handle not-found cases, limit result sizes, use query builders for dynamic filters.

</details>

---

23. How do you implement pagination in queries?

<details>
<summary><strong>Answer</strong></summary>

**Pagination** in GraphQL allows clients to fetch large datasets in smaller chunks, improving performance and user experience. NestJS supports **offset-based pagination** (skip/limit), **cursor-based pagination** (relay-style), and **page-based pagination** (page/pageSize). Cursor-based pagination is preferred for large datasets and infinite scrolling as it handles data changes better than offset pagination.

### **Basic Offset Pagination**

```typescript
// dtos/pagination.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { Min, Max } from 'class-validator';

@InputType()
export class PaginationInput {
  @Field(() => Int, { defaultValue: 0 })
  @Min(0)
  offset: number = 0; // Number of records to skip

  @Field(() => Int, { defaultValue: 20 })
  @Min(1)
  @Max(100) // Prevent excessive data fetching
  limit: number = 20; // Max records to return
}

// users.resolver.ts
import { Resolver, Query, Args } from '@nestjs/graphql';
import { User } from './models/user.model';
import { UsersService } from './users.service';
import { PaginationInput } from './dtos/pagination.input';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User])
  async users(
    @Args('pagination', { nullable: true }) pagination?: PaginationInput,
  ): Promise<User[]> {
    const { offset, limit } = pagination || { offset: 0, limit: 20 };
    return this.usersService.findAll(offset, limit);
  }
}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async findAll(offset: number, limit: number): Promise<User[]> {
    return this.userRepository.find({
      skip: offset,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }
}
```

**GraphQL Query:**
```graphql
query {
  users(pagination: { offset: 0, limit: 10 }) {
    id
    name
  }
}

# Page 2
query {
  users(pagination: { offset: 10, limit: 10 }) {
    id
    name
  }
}
```

### **Advanced Examples**

#### **1. Paginated Response with Metadata**

```typescript
// models/paginated-response.model.ts
import { ObjectType, Field, Int } from '@nestjs/graphql';
import { Type } from '@nestjs/common';

export interface IPaginatedType<T> {
  items: T[];
  total: number;
  hasMore: boolean;
  pageInfo: PageInfo;
}

@ObjectType()
export class PageInfo {
  @Field(() => Int)
  offset: number;

  @Field(() => Int)
  limit: number;

  @Field(() => Int)
  total: number;

  @Field(() => Int)
  totalPages: number;

  @Field(() => Int)
  currentPage: number;

  @Field()
  hasNextPage: boolean;

  @Field()
  hasPreviousPage: boolean;
}

export function Paginated<T>(classRef: Type<T>): Type<IPaginatedType<T>> {
  @ObjectType({ isAbstract: true })
  abstract class PaginatedType implements IPaginatedType<T> {
    @Field(() => [classRef])
    items: T[];

    @Field(() => Int)
    total: number;

    @Field()
    hasMore: boolean;

    @Field(() => PageInfo)
    pageInfo: PageInfo;
  }
  return PaginatedType as Type<IPaginatedType<T>>;
}

// Specific paginated type for Users
@ObjectType()
export class PaginatedUsers extends Paginated(User) {}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => PaginatedUsers)
  async paginatedUsers(
    @Args('pagination', { nullable: true }) pagination?: PaginationInput,
  ): Promise<PaginatedUsers> {
    const { offset, limit } = pagination || { offset: 0, limit: 20 };
    const [items, total] = await this.usersService.findAndCount(offset, limit);
    
    const totalPages = Math.ceil(total / limit);
    const currentPage = Math.floor(offset / limit) + 1;

    return {
      items,
      total,
      hasMore: offset + limit < total,
      pageInfo: {
        offset,
        limit,
        total,
        totalPages,
        currentPage,
        hasNextPage: currentPage < totalPages,
        hasPreviousPage: currentPage > 1,
      },
    };
  }
}

// users.service.ts
async findAndCount(offset: number, limit: number): Promise<[User[], number]> {
  return this.userRepository.findAndCount({
    skip: offset,
    take: limit,
    order: { createdAt: 'DESC' },
  });
}
```

**GraphQL Query:**
```graphql
query {
  paginatedUsers(pagination: { offset: 0, limit: 10 }) {
    items {
      id
      name
      email
    }
    total
    hasMore
    pageInfo {
      offset
      limit
      total
      totalPages
      currentPage
      hasNextPage
      hasPreviousPage
    }
  }
}
```

#### **2. Cursor-Based Pagination (Relay Style)**

```typescript
// models/connection.model.ts
import { ObjectType, Field } from '@nestjs/graphql';
import { Type } from '@nestjs/common';

@ObjectType()
export class PageInfoCursor {
  @Field()
  hasNextPage: boolean;

  @Field()
  hasPreviousPage: boolean;

  @Field({ nullable: true })
  startCursor?: string;

  @Field({ nullable: true })
  endCursor?: string;
}

export function Connection<T>(classRef: Type<T>) {
  @ObjectType(`${classRef.name}Edge`, { isAbstract: true })
  abstract class Edge {
    @Field(() => classRef)
    node: T;

    @Field()
    cursor: string;
  }

  @ObjectType(`${classRef.name}Connection`, { isAbstract: true })
  abstract class ConnectionType {
    @Field(() => [Edge])
    edges: Edge[];

    @Field(() => PageInfoCursor)
    pageInfo: PageInfoCursor;

    @Field(() => Int)
    totalCount: number;
  }

  return { Edge, Connection: ConnectionType };
}

// Specific connection for Users
const { Edge: UserEdge, Connection: UserConnectionBase } = Connection(User);

@ObjectType()
export class UserEdge extends UserEdge {}

@ObjectType()
export class UserConnection extends UserConnectionBase {}

// dtos/cursor-pagination.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { Min, Max } from 'class-validator';

@InputType()
export class CursorPaginationInput {
  @Field({ nullable: true })
  after?: string; // Cursor to fetch after

  @Field({ nullable: true })
  before?: string; // Cursor to fetch before

  @Field(() => Int, { nullable: true })
  @Min(1)
  @Max(100)
  first?: number; // Fetch first N records

  @Field(() => Int, { nullable: true })
  @Min(1)
  @Max(100)
  last?: number; // Fetch last N records
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => UserConnection)
  async usersConnection(
    @Args('pagination', { nullable: true }) pagination?: CursorPaginationInput,
  ): Promise<UserConnection> {
    return this.usersService.findWithCursor(pagination);
  }
}

// users.service.ts
import { Buffer } from 'buffer';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  /**
   * Encode cursor (base64 encoding of ID)
   */
  private encodeCursor(id: string): string {
    return Buffer.from(`cursor:${id}`).toString('base64');
  }

  /**
   * Decode cursor to get ID
   */
  private decodeCursor(cursor: string): string {
    return Buffer.from(cursor, 'base64').toString('utf-8').replace('cursor:', '');
  }

  async findWithCursor(pagination?: CursorPaginationInput): Promise<UserConnection> {
    const { after, before, first, last } = pagination || {};
    
    // Default to first 20 if nothing specified
    const limit = first || last || 20;
    
    const query = this.userRepository
      .createQueryBuilder('user')
      .orderBy('user.id', 'ASC');

    // Handle 'after' cursor (forward pagination)
    if (after) {
      const afterId = this.decodeCursor(after);
      query.andWhere('user.id > :afterId', { afterId });
    }

    // Handle 'before' cursor (backward pagination)
    if (before) {
      const beforeId = this.decodeCursor(before);
      query.andWhere('user.id < :beforeId', { beforeId });
    }

    // Fetch limit + 1 to check if there are more records
    const users = await query.take(limit + 1).getMany();
    
    // Get total count
    const totalCount = await this.userRepository.count();

    // Check if there are more records
    const hasMore = users.length > limit;
    if (hasMore) {
      users.pop(); // Remove the extra record
    }

    // Handle 'last' (reverse order)
    if (last) {
      users.reverse();
    }

    const edges = users.map(user => ({
      node: user,
      cursor: this.encodeCursor(user.id),
    }));

    const startCursor = edges.length > 0 ? edges[0].cursor : null;
    const endCursor = edges.length > 0 ? edges[edges.length - 1].cursor : null;

    return {
      edges,
      pageInfo: {
        hasNextPage: hasMore && !!first,
        hasPreviousPage: !!after || !!before,
        startCursor,
        endCursor,
      },
      totalCount,
    };
  }
}
```

**GraphQL Query:**
```graphql
# First page
query {
  usersConnection(pagination: { first: 10 }) {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
    totalCount
  }
}

# Next page (use endCursor from previous query)
query {
  usersConnection(pagination: { first: 10, after: "Y3Vyc29yOjEw" }) {
    edges {
      node {
        id
        name
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

#### **3. Page-Based Pagination**

```typescript
// dtos/page-pagination.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { Min, Max } from 'class-validator';

@InputType()
export class PagePaginationInput {
  @Field(() => Int, { defaultValue: 1 })
  @Min(1)
  page: number = 1;

  @Field(() => Int, { defaultValue: 20 })
  @Min(1)
  @Max(100)
  pageSize: number = 20;
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  @Query(() => PaginatedUsers)
  async usersByPage(
    @Args('pagination', { nullable: true }) pagination?: PagePaginationInput,
  ): Promise<PaginatedUsers> {
    const { page, pageSize } = pagination || { page: 1, pageSize: 20 };
    const offset = (page - 1) * pageSize;
    
    const [items, total] = await this.usersService.findAndCount(offset, pageSize);
    const totalPages = Math.ceil(total / pageSize);

    return {
      items,
      total,
      hasMore: page < totalPages,
      pageInfo: {
        offset,
        limit: pageSize,
        total,
        totalPages,
        currentPage: page,
        hasNextPage: page < totalPages,
        hasPreviousPage: page > 1,
      },
    };
  }
}
```

**GraphQL Query:**
```graphql
query {
  usersByPage(pagination: { page: 1, pageSize: 10 }) {
    items {
      id
      name
    }
    pageInfo {
      currentPage
      totalPages
      hasNextPage
    }
  }
}
```

### **Production-Grade Implementation**

```typescript
// ============================================
// Complete Pagination System
// ============================================

// models/pagination.models.ts
import { ObjectType, Field, Int, InterfaceType } from '@nestjs/graphql';
import { Type } from '@nestjs/common';

// Offset pagination page info
@ObjectType()
export class PageInfo {
  @Field(() => Int, { description: 'Number of records skipped' })
  offset: number;

  @Field(() => Int, { description: 'Max records returned' })
  limit: number;

  @Field(() => Int, { description: 'Total records matching filter' })
  total: number;

  @Field(() => Int, { description: 'Total pages' })
  totalPages: number;

  @Field(() => Int, { description: 'Current page number (1-indexed)' })
  currentPage: number;

  @Field({ description: 'Has next page' })
  hasNextPage: boolean;

  @Field({ description: 'Has previous page' })
  hasPreviousPage: boolean;
}

// Cursor pagination page info
@ObjectType()
export class CursorPageInfo {
  @Field({ description: 'Has next page' })
  hasNextPage: boolean;

  @Field({ description: 'Has previous page' })
  hasPreviousPage: boolean;

  @Field({ nullable: true, description: 'First cursor in current page' })
  startCursor?: string;

  @Field({ nullable: true, description: 'Last cursor in current page' })
  endCursor?: string;
}

// Generic paginated response
export function Paginated<T>(classRef: Type<T>): any {
  @ObjectType(`Paginated${classRef.name}`, { isAbstract: true })
  class PaginatedType {
    @Field(() => [classRef], { description: 'Page items' })
    items: T[];

    @Field(() => Int, { description: 'Total count' })
    total: number;

    @Field({ description: 'Has more records' })
    hasMore: boolean;

    @Field(() => PageInfo, { description: 'Pagination metadata' })
    pageInfo: PageInfo;
  }
  return PaginatedType;
}

// Generic cursor connection
export function Connection<T>(classRef: Type<T>): any {
  @ObjectType(`${classRef.name}Edge`)
  class EdgeType {
    @Field(() => classRef)
    node: T;

    @Field({ description: 'Cursor for this node' })
    cursor: string;
  }

  @ObjectType(`${classRef.name}Connection`)
  class ConnectionType {
    @Field(() => [EdgeType], { description: 'List of edges' })
    edges: EdgeType[];

    @Field(() => CursorPageInfo, { description: 'Page metadata' })
    pageInfo: CursorPageInfo;

    @Field(() => Int, { description: 'Total count' })
    totalCount: number;
  }

  return { Edge: EdgeType, Connection: ConnectionType };
}

// dtos/pagination.inputs.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { Min, Max, IsOptional } from 'class-validator';

@InputType()
export class OffsetPaginationInput {
  @Field(() => Int, { defaultValue: 0, description: 'Records to skip' })
  @Min(0)
  @IsOptional()
  offset?: number = 0;

  @Field(() => Int, { defaultValue: 20, description: 'Max records' })
  @Min(1)
  @Max(100)
  @IsOptional()
  limit?: number = 20;
}

@InputType()
export class PagePaginationInput {
  @Field(() => Int, { defaultValue: 1, description: 'Page number (1-indexed)' })
  @Min(1)
  @IsOptional()
  page?: number = 1;

  @Field(() => Int, { defaultValue: 20, description: 'Records per page' })
  @Min(1)
  @Max(100)
  @IsOptional()
  pageSize?: number = 20;
}

@InputType()
export class CursorPaginationInput {
  @Field({ nullable: true, description: 'Cursor to fetch after' })
  @IsOptional()
  after?: string;

  @Field({ nullable: true, description: 'Cursor to fetch before' })
  @IsOptional()
  before?: string;

  @Field(() => Int, { nullable: true, description: 'First N records' })
  @Min(1)
  @Max(100)
  @IsOptional()
  first?: number;

  @Field(() => Int, { nullable: true, description: 'Last N records' })
  @Min(1)
  @Max(100)
  @IsOptional()
  last?: number;
}

// users.service.ts - Complete pagination service
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, FindOptionsWhere } from 'typeorm';
import { User } from './models/user.model';
import { Buffer } from 'buffer';

export interface PaginatedResult<T> {
  items: T[];
  total: number;
  hasMore: boolean;
  pageInfo: PageInfo;
}

export interface CursorConnection<T> {
  edges: Array<{ node: T; cursor: string }>;
  pageInfo: CursorPageInfo;
  totalCount: number;
}

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  /**
   * Offset-based pagination
   */
  async findWithOffsetPagination(
    offset: number = 0,
    limit: number = 20,
    filter?: FindOptionsWhere<User>,
  ): Promise<PaginatedResult<User>> {
    // Validate inputs
    offset = Math.max(0, offset);
    limit = Math.min(Math.max(1, limit), 100);

    const [items, total] = await this.userRepository.findAndCount({
      where: filter,
      skip: offset,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    const totalPages = Math.ceil(total / limit);
    const currentPage = Math.floor(offset / limit) + 1;

    return {
      items,
      total,
      hasMore: offset + limit < total,
      pageInfo: {
        offset,
        limit,
        total,
        totalPages,
        currentPage,
        hasNextPage: currentPage < totalPages,
        hasPreviousPage: currentPage > 1,
      },
    };
  }

  /**
   * Page-based pagination (convenience wrapper)
   */
  async findWithPagePagination(
    page: number = 1,
    pageSize: number = 20,
    filter?: FindOptionsWhere<User>,
  ): Promise<PaginatedResult<User>> {
    page = Math.max(1, page);
    const offset = (page - 1) * pageSize;
    return this.findWithOffsetPagination(offset, pageSize, filter);
  }

  /**
   * Cursor-based pagination
   */
  async findWithCursorPagination(
    pagination?: CursorPaginationInput,
    filter?: FindOptionsWhere<User>,
  ): Promise<CursorConnection<User>> {
    const { after, before, first, last } = pagination || {};
    
    // Validate: can't use both first and last
    if (first && last) {
      throw new Error('Cannot use both "first" and "last"');
    }

    // Default to first 20
    const limit = Math.min(first || last || 20, 100);
    
    const query = this.userRepository
      .createQueryBuilder('user')
      .orderBy('user.id', 'ASC');

    // Apply filters
    if (filter) {
      query.where(filter);
    }

    // Handle 'after' cursor (forward pagination)
    if (after) {
      const afterId = this.decodeCursor(after);
      query.andWhere('user.id > :afterId', { afterId });
    }

    // Handle 'before' cursor (backward pagination)
    if (before) {
      const beforeId = this.decodeCursor(before);
      query.andWhere('user.id < :beforeId', { beforeId });
    }

    // Fetch limit + 1 to check for more records
    const users = await query.take(limit + 1).getMany();
    
    // Check if there are more records
    const hasMore = users.length > limit;
    if (hasMore) {
      users.pop();
    }

    // Reverse if fetching last N
    if (last) {
      users.reverse();
    }

    // Build edges
    const edges = users.map(user => ({
      node: user,
      cursor: this.encodeCursor(user.id),
    }));

    const startCursor = edges.length > 0 ? edges[0].cursor : undefined;
    const endCursor = edges.length > 0 ? edges[edges.length - 1].cursor : undefined;

    // Get total count (cached or computed)
    const totalCount = await query.getCount();

    return {
      edges,
      pageInfo: {
        hasNextPage: hasMore && !!first,
        hasPreviousPage: !!after || !!before,
        startCursor,
        endCursor,
      },
      totalCount,
    };
  }

  /**
   * Encode cursor (base64 of ID)
   */
  private encodeCursor(id: string): string {
    return Buffer.from(`cursor:${id}`, 'utf-8').toString('base64');
  }

  /**
   * Decode cursor to ID
   */
  private decodeCursor(cursor: string): string {
    try {
      const decoded = Buffer.from(cursor, 'base64').toString('utf-8');
      return decoded.replace('cursor:', '');
    } catch (error) {
      throw new Error('Invalid cursor');
    }
  }
}

// users.resolver.ts - All pagination methods
import { Resolver, Query, Args } from '@nestjs/graphql';
import { UsePipes, ValidationPipe } from '@nestjs/common';

@ObjectType()
export class PaginatedUsers extends Paginated(User) {}

const { Edge: UserEdge, Connection: UserConnection } = Connection(User);

@ObjectType()
export class UserEdgeType extends UserEdge {}

@ObjectType()
export class UserConnectionType extends UserConnection {}

@Resolver(() => User)
@UsePipes(new ValidationPipe({ transform: true }))
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  /**
   * Offset pagination
   */
  @Query(() => PaginatedUsers, { description: 'Get users with offset pagination' })
  async users(
    @Args('pagination', { nullable: true }) pagination?: OffsetPaginationInput,
  ): Promise<PaginatedUsers> {
    const { offset, limit } = pagination || {};
    return this.usersService.findWithOffsetPagination(offset, limit);
  }

  /**
   * Page-based pagination
   */
  @Query(() => PaginatedUsers, { description: 'Get users by page number' })
  async usersByPage(
    @Args('pagination', { nullable: true }) pagination?: PagePaginationInput,
  ): Promise<PaginatedUsers> {
    const { page, pageSize } = pagination || {};
    return this.usersService.findWithPagePagination(page, pageSize);
  }

  /**
   * Cursor pagination (Relay-style)
   */
  @Query(() => UserConnectionType, { description: 'Get users with cursor pagination' })
  async usersConnection(
    @Args('pagination', { nullable: true }) pagination?: CursorPaginationInput,
  ): Promise<UserConnectionType> {
    return this.usersService.findWithCursorPagination(pagination);
  }
}
```

### **Client Usage Examples**

```graphql
# ============================================
# 1. Offset Pagination
# ============================================
query GetUsersPage1 {
  users(pagination: { offset: 0, limit: 10 }) {
    items {
      id
      name
    }
    total
    pageInfo {
      currentPage
      totalPages
      hasNextPage
    }
  }
}

# ============================================
# 2. Page-based Pagination
# ============================================
query GetUsersPage2 {
  usersByPage(pagination: { page: 2, pageSize: 10 }) {
    items {
      id
      name
    }
    pageInfo {
      currentPage
      totalPages
      hasNextPage
      hasPreviousPage
    }
  }
}

# ============================================
# 3. Cursor Pagination (Initial)
# ============================================
query GetUsersFirst10 {
  usersConnection(pagination: { first: 10 }) {
    edges {
      node {
        id
        name
        email
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}

# ============================================
# 4. Cursor Pagination (Next Page)
# ============================================
query GetUsersNext10 {
  usersConnection(pagination: { first: 10, after: "Y3Vyc29yOjEw" }) {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      endCursor
    }
  }
}

# ============================================
# 5. Cursor Pagination (Previous Page)
# ============================================
query GetUsersPrevious10 {
  usersConnection(pagination: { last: 10, before: "Y3Vyc29yOjIw" }) {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasPreviousPage
      startCursor
    }
  }
}
```

### **Best Practices**

1. **Choose Right Pagination Type**:
   - **Offset**: Simple, good for small datasets, page numbers
   - **Cursor**: Large datasets, infinite scroll, handles data changes
   - **Page**: User-friendly, page numbers visible

2. **Set Maximum Limits**: Always cap limit/first/last (e.g., 100) to prevent performance issues

3. **Return Metadata**: Include total count, page info, has_more flags for UX

4. **Handle Empty Results**: Return empty arrays, not null, with proper metadata

5. **Validate Input**: Check pagination params, prevent negative offsets

6. **Index Database**: Add indexes on sort columns for performance

7. **Cache Total Count**: Cache `totalCount` queries for large tables

8. **Consistent Ordering**: Always specify `ORDER BY` for predictable pagination

9. **Document Limits**: Clearly document max limits in schema descriptions

10. **Use Cursor for Large Data**: Prefer cursor pagination for tables >10k rows

**Interview Tip**: Implement pagination using **offset** (`skip/take` in TypeORM), **page** (offset = (page-1) × pageSize), or **cursor** (Relay style with base64-encoded IDs). **Offset pagination**: simple, `@Args('pagination') { offset, limit }`, use `findAndCount()` for total. **Cursor pagination**: stable for infinite scroll, encode ID as cursor, fetch limit+1 to check `hasNextPage`. Return **metadata**: `totalCount`, `pageInfo` with `hasNext/hasPrevious`, `totalPages`. **Best practices**: (1) Set max limits (100), (2) Always ORDER BY, (3) Validate inputs, (4) Use cursor for large datasets, (5) Cache total counts. **Production**: Index sort columns, handle empty results, validate cursors, prefer cursor over offset for scaling.

</details>

---
24. How do you implement filtering and sorting?

<details>
<summary><strong>Answer</strong></summary>

**Filtering and sorting** in GraphQL queries allow clients to search, filter, and order data dynamically. In NestJS, you define **Input Types** for filter criteria and sort options, then use **query builders** or **TypeORM find options** to apply them. Production implementations support **multiple filters**, **operators** (equals, contains, greater than), **case-insensitive search**, **null handling**, and **multiple sort fields**.

### **Basic Filtering and Sorting**

```typescript
// dtos/filter.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsOptional } from 'class-validator';

@InputType()
export class UserFilterInput {
  @Field({ nullable: true })
  @IsOptional()
  name?: string; // Filter by name (contains)

  @Field({ nullable: true })
  @IsOptional()
  email?: string; // Filter by email (exact match)

  @Field({ nullable: true })
  @IsOptional()
  role?: string; // Filter by role
}

@InputType()
export class SortInput {
  @Field({ defaultValue: 'createdAt' })
  field: string; // Field to sort by

  @Field({ defaultValue: 'DESC' })
  order: 'ASC' | 'DESC'; // Sort direction
}

// users.resolver.ts
import { Resolver, Query, Args } from '@nestjs/graphql';
import { User } from './models/user.model';
import { UsersService } from './users.service';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User])
  async users(
    @Args('filter', { nullable: true }) filter?: UserFilterInput,
    @Args('sort', { nullable: true }) sort?: SortInput,
  ): Promise<User[]> {
    return this.usersService.findAll(filter, sort);
  }
}

// users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async findAll(filter?: UserFilterInput, sort?: SortInput): Promise<User[]> {
    const query = this.userRepository.createQueryBuilder('user');

    // Apply filters
    if (filter) {
      if (filter.name) {
        query.andWhere('user.name ILIKE :name', { name: `%${filter.name}%` });
      }
      if (filter.email) {
        query.andWhere('user.email = :email', { email: filter.email });
      }
      if (filter.role) {
        query.andWhere('user.role = :role', { role: filter.role });
      }
    }

    // Apply sorting
    if (sort) {
      query.orderBy(`user.${sort.field}`, sort.order);
    }

    return query.getMany();
  }
}
```

**GraphQL Query:**
```graphql
query {
  users(
    filter: { name: "John", role: "ADMIN" }
    sort: { field: "createdAt", order: DESC }
  ) {
    id
    name
    email
    role
  }
}
```

### **Advanced Examples**

#### **1. Advanced Filters with Operators**

```typescript
// dtos/advanced-filter.input.ts
import { InputType, Field, registerEnumType } from '@nestjs/graphql';
import { IsOptional, IsEnum, IsDateString, Min, Max } from 'class-validator';

// Filter operators
export enum StringOperator {
  EQUALS = 'EQUALS',
  CONTAINS = 'CONTAINS',
  STARTS_WITH = 'STARTS_WITH',
  ENDS_WITH = 'ENDS_WITH',
}

export enum NumberOperator {
  EQUALS = 'EQUALS',
  GT = 'GT', // Greater than
  GTE = 'GTE', // Greater than or equal
  LT = 'LT', // Less than
  LTE = 'LTE', // Less than or equal
  BETWEEN = 'BETWEEN',
}

export enum DateOperator {
  EQUALS = 'EQUALS',
  BEFORE = 'BEFORE',
  AFTER = 'AFTER',
  BETWEEN = 'BETWEEN',
}

registerEnumType(StringOperator, { name: 'StringOperator' });
registerEnumType(NumberOperator, { name: 'NumberOperator' });
registerEnumType(DateOperator, { name: 'DateOperator' });

// String filter
@InputType()
export class StringFilter {
  @Field(() => StringOperator, { defaultValue: StringOperator.EQUALS })
  operator: StringOperator;

  @Field()
  value: string;

  @Field({ defaultValue: false })
  caseSensitive: boolean = false;
}

// Number filter
@InputType()
export class NumberFilter {
  @Field(() => NumberOperator)
  operator: NumberOperator;

  @Field()
  value: number;

  @Field({ nullable: true }) // For BETWEEN operator
  valueTo?: number;
}

// Date filter
@InputType()
export class DateFilter {
  @Field(() => DateOperator)
  operator: DateOperator;

  @Field()
  @IsDateString()
  value: string;

  @Field({ nullable: true }) // For BETWEEN operator
  @IsDateString()
  @IsOptional()
  valueTo?: string;
}

// Advanced user filter
@InputType()
export class AdvancedUserFilterInput {
  @Field(() => StringFilter, { nullable: true })
  @IsOptional()
  name?: StringFilter;

  @Field(() => StringFilter, { nullable: true })
  @IsOptional()
  email?: StringFilter;

  @Field(() => [String], { nullable: true })
  @IsOptional()
  roles?: string[]; // Array of roles (OR condition)

  @Field(() => NumberFilter, { nullable: true })
  @IsOptional()
  age?: NumberFilter;

  @Field(() => DateFilter, { nullable: true })
  @IsOptional()
  createdAt?: DateFilter;

  @Field({ nullable: true })
  @IsOptional()
  isActive?: boolean;

  @Field({ nullable: true })
  @IsOptional()
  isVerified?: boolean;
}

// users.service.ts
@Injectable()
export class UsersService {
  async findWithAdvancedFilters(
    filter?: AdvancedUserFilterInput,
  ): Promise<User[]> {
    const query = this.userRepository.createQueryBuilder('user');

    if (filter) {
      // String filter with operators
      if (filter.name) {
        this.applyStringFilter(query, 'user.name', filter.name);
      }

      if (filter.email) {
        this.applyStringFilter(query, 'user.email', filter.email);
      }

      // Array filter (OR condition)
      if (filter.roles && filter.roles.length > 0) {
        query.andWhere('user.role IN (:...roles)', { roles: filter.roles });
      }

      // Number filter with operators
      if (filter.age) {
        this.applyNumberFilter(query, 'user.age', filter.age);
      }

      // Date filter with operators
      if (filter.createdAt) {
        this.applyDateFilter(query, 'user.createdAt', filter.createdAt);
      }

      // Boolean filters
      if (filter.isActive !== undefined) {
        query.andWhere('user.isActive = :isActive', { isActive: filter.isActive });
      }

      if (filter.isVerified !== undefined) {
        query.andWhere('user.isVerified = :isVerified', { 
          isVerified: filter.isVerified 
        });
      }
    }

    return query.getMany();
  }

  /**
   * Apply string filter with operator
   */
  private applyStringFilter(
    query: SelectQueryBuilder<User>,
    field: string,
    filter: StringFilter,
  ): void {
    const { operator, value, caseSensitive } = filter;
    const comparison = caseSensitive ? '=' : 'ILIKE';
    
    switch (operator) {
      case StringOperator.EQUALS:
        query.andWhere(`${field} ${comparison} :value`, { value });
        break;
      case StringOperator.CONTAINS:
        query.andWhere(`${field} ${comparison} :value`, { value: `%${value}%` });
        break;
      case StringOperator.STARTS_WITH:
        query.andWhere(`${field} ${comparison} :value`, { value: `${value}%` });
        break;
      case StringOperator.ENDS_WITH:
        query.andWhere(`${field} ${comparison} :value`, { value: `%${value}` });
        break;
    }
  }

  /**
   * Apply number filter with operator
   */
  private applyNumberFilter(
    query: SelectQueryBuilder<User>,
    field: string,
    filter: NumberFilter,
  ): void {
    const { operator, value, valueTo } = filter;

    switch (operator) {
      case NumberOperator.EQUALS:
        query.andWhere(`${field} = :value`, { value });
        break;
      case NumberOperator.GT:
        query.andWhere(`${field} > :value`, { value });
        break;
      case NumberOperator.GTE:
        query.andWhere(`${field} >= :value`, { value });
        break;
      case NumberOperator.LT:
        query.andWhere(`${field} < :value`, { value });
        break;
      case NumberOperator.LTE:
        query.andWhere(`${field} <= :value`, { value });
        break;
      case NumberOperator.BETWEEN:
        query.andWhere(`${field} BETWEEN :value AND :valueTo`, { 
          value, 
          valueTo 
        });
        break;
    }
  }

  /**
   * Apply date filter with operator
   */
  private applyDateFilter(
    query: SelectQueryBuilder<User>,
    field: string,
    filter: DateFilter,
  ): void {
    const { operator, value, valueTo } = filter;
    const dateValue = new Date(value);
    const dateValueTo = valueTo ? new Date(valueTo) : null;

    switch (operator) {
      case DateOperator.EQUALS:
        query.andWhere(`DATE(${field}) = DATE(:value)`, { value: dateValue });
        break;
      case DateOperator.BEFORE:
        query.andWhere(`${field} < :value`, { value: dateValue });
        break;
      case DateOperator.AFTER:
        query.andWhere(`${field} > :value`, { value: dateValue });
        break;
      case DateOperator.BETWEEN:
        query.andWhere(`${field} BETWEEN :value AND :valueTo`, { 
          value: dateValue, 
          valueTo: dateValueTo 
        });
        break;
    }
  }
}
```

**GraphQL Query:**
```graphql
query {
  users(filter: {
    name: { operator: CONTAINS, value: "John", caseSensitive: false }
    age: { operator: GTE, value: 18 }
    createdAt: { operator: AFTER, value: "2024-01-01T00:00:00Z" }
    roles: ["ADMIN", "MODERATOR"]
    isActive: true
  }) {
    id
    name
    age
    role
  }
}
```

#### **2. Multi-Field Sorting**

```typescript
// dtos/multi-sort.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsEnum, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

export enum SortOrder {
  ASC = 'ASC',
  DESC = 'DESC',
}

export enum UserSortField {
  NAME = 'name',
  EMAIL = 'email',
  CREATED_AT = 'createdAt',
  UPDATED_AT = 'updatedAt',
  AGE = 'age',
}

registerEnumType(SortOrder, { name: 'SortOrder' });
registerEnumType(UserSortField, { name: 'UserSortField' });

@InputType()
export class SortFieldInput {
  @Field(() => UserSortField)
  @IsEnum(UserSortField)
  field: UserSortField;

  @Field(() => SortOrder, { defaultValue: SortOrder.ASC })
  @IsEnum(SortOrder)
  order: SortOrder = SortOrder.ASC;

  @Field({ nullable: true, description: 'How to handle null values' })
  nulls?: 'FIRST' | 'LAST';
}

@InputType()
export class MultiSortInput {
  @Field(() => [SortFieldInput])
  @ValidateNested({ each: true })
  @Type(() => SortFieldInput)
  sorts: SortFieldInput[];
}

// users.service.ts
async findWithMultiSort(sort?: MultiSortInput): Promise<User[]> {
  const query = this.userRepository.createQueryBuilder('user');

  if (sort && sort.sorts.length > 0) {
    sort.sorts.forEach((sortField, index) => {
      const orderBy = `user.${sortField.field}`;
      const nulls = sortField.nulls ? `NULLS ${sortField.nulls}` : '';

      if (index === 0) {
        query.orderBy(orderBy, sortField.order, nulls);
      } else {
        query.addOrderBy(orderBy, sortField.order, nulls);
      }
    });
  }

  return query.getMany();
}
```

**GraphQL Query:**
```graphql
query {
  users(sort: {
    sorts: [
      { field: NAME, order: ASC, nulls: LAST }
      { field: CREATED_AT, order: DESC }
    ]
  }) {
    id
    name
    createdAt
  }
}
```

#### **3. Full-Text Search**

```typescript
// dtos/search.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsOptional, MinLength } from 'class-validator';

@InputType()
export class SearchInput {
  @Field({ description: 'Search query' })
  @MinLength(2)
  query: string;

  @Field(() => [String], { 
    nullable: true,
    description: 'Fields to search in' 
  })
  @IsOptional()
  fields?: string[]; // ['name', 'email', 'bio']

  @Field({ defaultValue: false })
  exactMatch: boolean = false;
}

// users.service.ts
async search(searchInput: SearchInput): Promise<User[]> {
  const { query, fields, exactMatch } = searchInput;
  const qb = this.userRepository.createQueryBuilder('user');

  const searchFields = fields || ['name', 'email', 'bio'];
  const searchPattern = exactMatch ? query : `%${query}%`;
  const comparison = 'ILIKE';

  // Build OR conditions for each search field
  const conditions = searchFields
    .map((field, index) => `user.${field} ${comparison} :query${index}`)
    .join(' OR ');

  // Add parameters for each field
  const parameters = searchFields.reduce((params, field, index) => {
    params[`query${index}`] = searchPattern;
    return params;
  }, {});

  qb.where(`(${conditions})`, parameters);

  return qb.getMany();
}

// PostgreSQL Full-Text Search
async fullTextSearch(query: string): Promise<User[]> {
  return this.userRepository
    .createQueryBuilder('user')
    .where(
      `to_tsvector('english', user.name || ' ' || user.bio) @@ plainto_tsquery('english', :query)`,
      { query }
    )
    .orderBy(
      `ts_rank(to_tsvector('english', user.name || ' ' || user.bio), plainto_tsquery('english', :query))`,
      'DESC'
    )
    .getMany();
}
```

### **Production-Grade Implementation**

```typescript
// ============================================
// Complete Filter & Sort System
// ============================================

// dtos/query-options.input.ts
import { InputType, Field, Int, registerEnumType } from '@nestjs/graphql';
import { IsOptional, ValidateNested, IsEnum, Min, Max } from 'class-validator';
import { Type } from 'class-transformer';

// Enums
export enum StringOperator {
  EQUALS = 'EQUALS',
  NOT_EQUALS = 'NOT_EQUALS',
  CONTAINS = 'CONTAINS',
  NOT_CONTAINS = 'NOT_CONTAINS',
  STARTS_WITH = 'STARTS_WITH',
  ENDS_WITH = 'ENDS_WITH',
  IN = 'IN',
  NOT_IN = 'NOT_IN',
  IS_NULL = 'IS_NULL',
  IS_NOT_NULL = 'IS_NOT_NULL',
}

export enum NumberOperator {
  EQUALS = 'EQUALS',
  NOT_EQUALS = 'NOT_EQUALS',
  GT = 'GT',
  GTE = 'GTE',
  LT = 'LT',
  LTE = 'LTE',
  BETWEEN = 'BETWEEN',
  IN = 'IN',
  NOT_IN = 'NOT_IN',
}

export enum SortOrder {
  ASC = 'ASC',
  DESC = 'DESC',
}

export enum NullHandling {
  FIRST = 'FIRST',
  LAST = 'LAST',
}

registerEnumType(StringOperator, { name: 'StringOperator' });
registerEnumType(NumberOperator, { name: 'NumberOperator' });
registerEnumType(SortOrder, { name: 'SortOrder' });
registerEnumType(NullHandling, { name: 'NullHandling' });

// Filter types
@InputType()
export class StringFilterInput {
  @Field(() => StringOperator, { defaultValue: StringOperator.EQUALS })
  operator: StringOperator;

  @Field({ nullable: true })
  value?: string;

  @Field(() => [String], { nullable: true })
  values?: string[]; // For IN/NOT_IN

  @Field({ defaultValue: false })
  caseSensitive: boolean = false;
}

@InputType()
export class NumberFilterInput {
  @Field(() => NumberOperator, { defaultValue: NumberOperator.EQUALS })
  operator: NumberOperator;

  @Field({ nullable: true })
  value?: number;

  @Field({ nullable: true })
  valueFrom?: number; // For BETWEEN

  @Field({ nullable: true })
  valueTo?: number; // For BETWEEN

  @Field(() => [Number], { nullable: true })
  values?: number[]; // For IN/NOT_IN
}

@InputType()
export class BooleanFilterInput {
  @Field()
  value: boolean;
}

// User-specific filters
@InputType()
export class UserFiltersInput {
  @Field(() => StringFilterInput, { nullable: true })
  @ValidateNested()
  @Type(() => StringFilterInput)
  @IsOptional()
  name?: StringFilterInput;

  @Field(() => StringFilterInput, { nullable: true })
  @ValidateNested()
  @Type(() => StringFilterInput)
  @IsOptional()
  email?: StringFilterInput;

  @Field(() => [String], { nullable: true })
  @IsOptional()
  roles?: string[];

  @Field(() => NumberFilterInput, { nullable: true })
  @ValidateNested()
  @Type(() => NumberFilterInput)
  @IsOptional()
  age?: NumberFilterInput;

  @Field(() => BooleanFilterInput, { nullable: true })
  @ValidateNested()
  @Type(() => BooleanFilterInput)
  @IsOptional()
  isActive?: BooleanFilterInput;

  @Field({ nullable: true })
  @IsOptional()
  search?: string; // Full-text search

  // Date range filters
  @Field({ nullable: true })
  @IsOptional()
  createdAfter?: Date;

  @Field({ nullable: true })
  @IsOptional()
  createdBefore?: Date;
}

// Sort input
@InputType()
export class SortFieldInput {
  @Field()
  field: string;

  @Field(() => SortOrder, { defaultValue: SortOrder.ASC })
  @IsEnum(SortOrder)
  order: SortOrder = SortOrder.ASC;

  @Field(() => NullHandling, { nullable: true })
  @IsEnum(NullHandling)
  @IsOptional()
  nulls?: NullHandling;
}

@InputType()
export class UserSortInput {
  @Field(() => [SortFieldInput])
  @ValidateNested({ each: true })
  @Type(() => SortFieldInput)
  fields: SortFieldInput[];
}

// Pagination
@InputType()
export class PaginationInput {
  @Field(() => Int, { defaultValue: 0 })
  @Min(0)
  offset: number = 0;

  @Field(() => Int, { defaultValue: 20 })
  @Min(1)
  @Max(100)
  limit: number = 20;
}

// Combined query options
@InputType()
export class UserQueryOptionsInput {
  @Field(() => UserFiltersInput, { nullable: true })
  @ValidateNested()
  @Type(() => UserFiltersInput)
  @IsOptional()
  filters?: UserFiltersInput;

  @Field(() => UserSortInput, { nullable: true })
  @ValidateNested()
  @Type(() => UserSortInput)
  @IsOptional()
  sort?: UserSortInput;

  @Field(() => PaginationInput, { nullable: true })
  @ValidateNested()
  @Type(() => PaginationInput)
  @IsOptional()
  pagination?: PaginationInput;
}

// ============================================
// Service with Complete Filter/Sort Logic
// ============================================
import { Injectable, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, SelectQueryBuilder, Brackets } from 'typeorm';
import { User } from './models/user.model';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  /**
   * Find users with filters, sorting, and pagination
   */
  async findWithOptions(options?: UserQueryOptionsInput): Promise<{
    items: User[];
    total: number;
  }> {
    const query = this.userRepository.createQueryBuilder('user');

    // Apply filters
    if (options?.filters) {
      this.applyFilters(query, options.filters);
    }

    // Apply sorting
    if (options?.sort) {
      this.applySorting(query, options.sort);
    } else {
      // Default sort
      query.orderBy('user.createdAt', 'DESC');
    }

    // Get total count before pagination
    const total = await query.getCount();

    // Apply pagination
    if (options?.pagination) {
      const { offset, limit } = options.pagination;
      query.skip(offset).take(limit);
    }

    const items = await query.getMany();

    return { items, total };
  }

  /**
   * Apply filters to query builder
   */
  private applyFilters(
    query: SelectQueryBuilder<User>,
    filters: UserFiltersInput,
  ): void {
    // String filters
    if (filters.name) {
      this.applyStringFilter(query, 'user.name', filters.name);
    }

    if (filters.email) {
      this.applyStringFilter(query, 'user.email', filters.email);
    }

    // Array filter (roles)
    if (filters.roles && filters.roles.length > 0) {
      query.andWhere('user.role IN (:...roles)', { roles: filters.roles });
    }

    // Number filter
    if (filters.age) {
      this.applyNumberFilter(query, 'user.age', filters.age);
    }

    // Boolean filter
    if (filters.isActive) {
      query.andWhere('user.isActive = :isActive', { 
        isActive: filters.isActive.value 
      });
    }

    // Date range filters
    if (filters.createdAfter) {
      query.andWhere('user.createdAt >= :createdAfter', { 
        createdAfter: filters.createdAfter 
      });
    }

    if (filters.createdBefore) {
      query.andWhere('user.createdAt <= :createdBefore', { 
        createdBefore: filters.createdBefore 
      });
    }

    // Full-text search
    if (filters.search) {
      query.andWhere(
        new Brackets(qb => {
          qb.where('user.name ILIKE :search', { search: `%${filters.search}%` })
            .orWhere('user.email ILIKE :search', { search: `%${filters.search}%` })
            .orWhere('user.bio ILIKE :search', { search: `%${filters.search}%` });
        })
      );
    }
  }

  /**
   * Apply string filter with operator
   */
  private applyStringFilter(
    query: SelectQueryBuilder<User>,
    field: string,
    filter: StringFilterInput,
  ): void {
    const { operator, value, values, caseSensitive } = filter;
    const comparison = caseSensitive ? '=' : 'ILIKE';
    const paramName = field.replace('.', '_');

    switch (operator) {
      case StringOperator.EQUALS:
        query.andWhere(`${field} ${comparison} :${paramName}`, { 
          [paramName]: value 
        });
        break;

      case StringOperator.NOT_EQUALS:
        query.andWhere(`${field} ${comparison === 'ILIKE' ? 'NOT ILIKE' : '!='} :${paramName}`, { 
          [paramName]: value 
        });
        break;

      case StringOperator.CONTAINS:
        query.andWhere(`${field} ${comparison} :${paramName}`, { 
          [paramName]: `%${value}%` 
        });
        break;

      case StringOperator.NOT_CONTAINS:
        query.andWhere(`${field} NOT ${comparison} :${paramName}`, { 
          [paramName]: `%${value}%` 
        });
        break;

      case StringOperator.STARTS_WITH:
        query.andWhere(`${field} ${comparison} :${paramName}`, { 
          [paramName]: `${value}%` 
        });
        break;

      case StringOperator.ENDS_WITH:
        query.andWhere(`${field} ${comparison} :${paramName}`, { 
          [paramName]: `%${value}` 
        });
        break;

      case StringOperator.IN:
        if (values && values.length > 0) {
          query.andWhere(`${field} IN (:...${paramName})`, { 
            [paramName]: values 
          });
        }
        break;

      case StringOperator.NOT_IN:
        if (values && values.length > 0) {
          query.andWhere(`${field} NOT IN (:...${paramName})`, { 
            [paramName]: values 
          });
        }
        break;

      case StringOperator.IS_NULL:
        query.andWhere(`${field} IS NULL`);
        break;

      case StringOperator.IS_NOT_NULL:
        query.andWhere(`${field} IS NOT NULL`);
        break;

      default:
        throw new BadRequestException(`Invalid string operator: ${operator}`);
    }
  }

  /**
   * Apply number filter with operator
   */
  private applyNumberFilter(
    query: SelectQueryBuilder<User>,
    field: string,
    filter: NumberFilterInput,
  ): void {
    const { operator, value, valueFrom, valueTo, values } = filter;
    const paramName = field.replace('.', '_');

    switch (operator) {
      case NumberOperator.EQUALS:
        query.andWhere(`${field} = :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.NOT_EQUALS:
        query.andWhere(`${field} != :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.GT:
        query.andWhere(`${field} > :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.GTE:
        query.andWhere(`${field} >= :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.LT:
        query.andWhere(`${field} < :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.LTE:
        query.andWhere(`${field} <= :${paramName}`, { [paramName]: value });
        break;

      case NumberOperator.BETWEEN:
        query.andWhere(`${field} BETWEEN :${paramName}From AND :${paramName}To`, {
          [`${paramName}From`]: valueFrom,
          [`${paramName}To`]: valueTo,
        });
        break;

      case NumberOperator.IN:
        if (values && values.length > 0) {
          query.andWhere(`${field} IN (:...${paramName})`, { 
            [paramName]: values 
          });
        }
        break;

      case NumberOperator.NOT_IN:
        if (values && values.length > 0) {
          query.andWhere(`${field} NOT IN (:...${paramName})`, { 
            [paramName]: values 
          });
        }
        break;

      default:
        throw new BadRequestException(`Invalid number operator: ${operator}`);
    }
  }

  /**
   * Apply sorting to query builder
   */
  private applySorting(
    query: SelectQueryBuilder<User>,
    sort: UserSortInput,
  ): void {
    // Whitelist allowed sort fields to prevent SQL injection
    const allowedFields = ['name', 'email', 'createdAt', 'updatedAt', 'age'];

    sort.fields.forEach((sortField, index) => {
      // Validate field
      if (!allowedFields.includes(sortField.field)) {
        throw new BadRequestException(`Invalid sort field: ${sortField.field}`);
      }

      const field = `user.${sortField.field}`;
      const { order, nulls } = sortField;
      const nullsClause = nulls ? `NULLS ${nulls}` : '';

      if (index === 0) {
        query.orderBy(field, order, nullsClause);
      } else {
        query.addOrderBy(field, order, nullsClause);
      }
    });
  }
}

// ============================================
// Resolver
// ============================================
import { Resolver, Query, Args } from '@nestjs/graphql';
import { UsePipes, ValidationPipe } from '@nestjs/common';

@ObjectType()
export class UserQueryResult {
  @Field(() => [User])
  items: User[];

  @Field(() => Int)
  total: number;
}

@Resolver(() => User)
@UsePipes(new ValidationPipe({ transform: true, whitelist: true }))
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => UserQueryResult, { 
    description: 'Search users with filters, sorting, and pagination' 
  })
  async queryUsers(
    @Args('options', { nullable: true }) options?: UserQueryOptionsInput,
  ): Promise<UserQueryResult> {
    return this.usersService.findWithOptions(options);
  }
}
```

### **GraphQL Queries (Client Usage)**

```graphql
# ============================================
# 1. Basic filtering and sorting
# ============================================
query {
  queryUsers(options: {
    filters: {
      name: { operator: CONTAINS, value: "John" }
      roles: ["ADMIN", "MODERATOR"]
      isActive: { value: true }
    }
    sort: {
      fields: [
        { field: "name", order: ASC }
      ]
    }
    pagination: { offset: 0, limit: 10 }
  }) {
    items {
      id
      name
      email
      role
    }
    total
  }
}

# ============================================
# 2. Advanced filtering with operators
# ============================================
query {
  queryUsers(options: {
    filters: {
      age: { operator: BETWEEN, valueFrom: 18, valueTo: 65 }
      email: { operator: ENDS_WITH, value: "@company.com" }
      name: { operator: NOT_CONTAINS, value: "test" }
      createdAfter: "2024-01-01T00:00:00Z"
    }
    sort: {
      fields: [
        { field: "age", order: DESC, nulls: LAST }
        { field: "name", order: ASC }
      ]
    }
  }) {
    items {
      id
      name
      age
    }
    total
  }
}

# ============================================
# 3. Full-text search
# ============================================
query {
  queryUsers(options: {
    filters: {
      search: "john developer"
    }
    pagination: { offset: 0, limit: 20 }
  }) {
    items {
      id
      name
      email
      bio
    }
    total
  }
}

# ============================================
# 4. Multiple conditions
# ============================================
query SearchActiveAdmins {
  queryUsers(options: {
    filters: {
      roles: ["ADMIN"]
      isActive: { value: true }
      age: { operator: GTE, value: 25 }
      createdAfter: "2023-01-01T00:00:00Z"
    }
    sort: {
      fields: [
        { field: "createdAt", order: DESC }
      ]
    }
    pagination: { offset: 0, limit: 50 }
  }) {
    items {
      id
      name
      role
      createdAt
    }
    total
  }
}
```

### **Best Practices**

1. **Whitelist Sort Fields**: Validate allowed sort fields to prevent SQL injection
2. **Use Query Builders**: Prefer TypeORM query builders over raw SQL for dynamic queries
3. **Index Filtered/Sorted Columns**: Add database indexes on frequently filtered/sorted fields
4. **Validate Operators**: Check operator values to prevent invalid queries
5. **Case-Insensitive Search**: Use `ILIKE` for PostgreSQL, consider full-text search for better performance
6. **Limit Result Sets**: Always apply pagination to prevent performance issues
7. **Use Enums**: Define enums for operators, sort fields for type safety
8. **Handle Null Values**: Support null handling in sorting (`NULLS FIRST/LAST`)
9. **Sanitize Input**: Validate and sanitize all filter values
10. **Optimize Complex Filters**: Cache filter results, use materialized views for heavy queries

**Interview Tip**: Implement filters using **Input Types** with operators (EQUALS, CONTAINS, GT, BETWEEN). Build **dynamic queries** with TypeORM query builder: `andWhere()` for filters, `orderBy/addOrderBy()` for sorting. **Pattern**: (1) Define FilterInput with operators, (2) Service applies filters conditionally to query builder, (3) Whitelist allowed sort fields. **Advanced**: Support multiple operators (string: CONTAINS/STARTS_WITH, number: GT/LTE/BETWEEN, date: BEFORE/AFTER), case-insensitive search (ILIKE), multi-field sorting, null handling (NULLS FIRST/LAST), full-text search. **Best practices**: Index filtered columns, validate inputs, use Brackets() for OR conditions, apply pagination always, cache heavy queries. **Production**: Sanitize inputs, prevent SQL injection, limit result sizes, use enums for type safety.

</details>

---

## Mutations

25. How do you implement a Mutation for creating data?

<details>
<summary><strong>Answer</strong></summary>

**Mutations** in GraphQL are used to **create, update, or delete** data (write operations). In NestJS, you define mutations using the `@Mutation()` decorator in resolvers, accept **Input Types** (defined with `@InputType()`) for data, and return the created entity. Production implementations include **validation** (class-validator), **error handling**, **transactions**, **authorization checks**, and **return proper GraphQL types**.

### **Basic Create Mutation**

```typescript
// dtos/create-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, IsNotEmpty, MinLength, IsEnum } from 'class-validator';

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
}

@InputType()
export class CreateUserInput {
  @Field()
  @IsNotEmpty()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;

  @Field(() => UserRole, { defaultValue: UserRole.USER })
  @IsEnum(UserRole)
  role: UserRole = UserRole.USER;
}

// users.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './models/user.model';
import { UsersService } from './users.service';
import { CreateUserInput } from './dtos/create-user.input';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => User, { description: 'Create a new user' })
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.usersService.create(input);
  }
}

// users.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async create(input: CreateUserInput): Promise<User> {
    // Check if user already exists
    const existingUser = await this.userRepository.findOne({
      where: { email: input.email },
    });

    if (existingUser) {
      throw new ConflictException('User with this email already exists');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(input.password, 10);

    // Create user
    const user = this.userRepository.create({
      ...input,
      password: hashedPassword,
    });

    return this.userRepository.save(user);
  }
}
```

**GraphQL Mutation:**
```graphql
mutation {
  createUser(input: {
    name: "John Doe"
    email: "john@example.com"
    password: "securePassword123"
    role: ADMIN
  }) {
    id
    name
    email
    role
    createdAt
  }
}
```

### **Advanced Examples**

#### **1. Create with Nested Relations**

```typescript
// dtos/create-post.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsNotEmpty, MinLength, IsArray, IsOptional } from 'class-validator';

@InputType()
export class CreatePostInput {
  @Field()
  @IsNotEmpty()
  @MinLength(5)
  title: string;

  @Field()
  @IsNotEmpty()
  content: string;

  @Field()
  authorId: string; // Reference to existing user

  @Field(() => [String], { nullable: true })
  @IsArray()
  @IsOptional()
  tags?: string[];

  @Field({ nullable: true })
  @IsOptional()
  published?: boolean;
}

// posts.resolver.ts
@Resolver(() => Post)
export class PostsResolver {
  constructor(
    private readonly postsService: PostsService,
    private readonly usersService: UsersService,
  ) {}

  @Mutation(() => Post)
  async createPost(
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    // Verify author exists
    await this.usersService.findById(input.authorId);
    
    return this.postsService.create(input);
  }
}

// posts.service.ts
@Injectable()
export class PostsService {
  constructor(
    @InjectRepository(Post)
    private readonly postRepository: Repository<Post>,
    @InjectRepository(Tag)
    private readonly tagRepository: Repository<Tag>,
  ) {}

  async create(input: CreatePostInput): Promise<Post> {
    // Create or find tags
    const tags = await this.findOrCreateTags(input.tags || []);

    // Create post
    const post = this.postRepository.create({
      title: input.title,
      content: input.content,
      author: { id: input.authorId }, // Reference existing user
      tags,
      published: input.published ?? false,
    });

    return this.postRepository.save(post);
  }

  private async findOrCreateTags(tagNames: string[]): Promise<Tag[]> {
    const tags: Tag[] = [];

    for (const name of tagNames) {
      let tag = await this.tagRepository.findOne({ where: { name } });
      
      if (!tag) {
        tag = this.tagRepository.create({ name });
        tag = await this.tagRepository.save(tag);
      }

      tags.push(tag);
    }

    return tags;
  }
}
```

**GraphQL Mutation:**
```graphql
mutation {
  createPost(input: {
    title: "Getting Started with NestJS"
    content: "NestJS is a powerful framework..."
    authorId: "123"
    tags: ["nestjs", "graphql", "typescript"]
    published: true
  }) {
    id
    title
    author {
      id
      name
    }
    tags {
      id
      name
    }
    published
    createdAt
  }
}
```

#### **2. Batch Create (Multiple Items)**

```typescript
// dtos/create-users.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
export class CreateUsersInput {
  @Field(() => [CreateUserInput])
  @ValidateNested({ each: true })
  @Type(() => CreateUserInput)
  users: CreateUserInput[];
}

// models/batch-result.model.ts
import { ObjectType, Field, Int } from '@nestjs/graphql';

@ObjectType()
export class BatchCreateResult {
  @Field(() => [User])
  created: User[];

  @Field(() => Int)
  successCount: number;

  @Field(() => Int)
  failureCount: number;

  @Field(() => [String])
  errors: string[];
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  @Mutation(() => BatchCreateResult)
  async createUsers(
    @Args('input') input: CreateUsersInput,
  ): Promise<BatchCreateResult> {
    return this.usersService.createBatch(input.users);
  }
}

// users.service.ts
async createBatch(inputs: CreateUserInput[]): Promise<BatchCreateResult> {
  const created: User[] = [];
  const errors: string[] = [];

  for (const input of inputs) {
    try {
      const user = await this.create(input);
      created.push(user);
    } catch (error) {
      errors.push(`Failed to create user ${input.email}: ${error.message}`);
    }
  }

  return {
    created,
    successCount: created.length,
    failureCount: errors.length,
    errors,
  };
}
```

**GraphQL Mutation:**
```graphql
mutation {
  createUsers(input: {
    users: [
      { name: "User 1", email: "user1@example.com", password: "pass123" }
      { name: "User 2", email: "user2@example.com", password: "pass456" }
      { name: "User 3", email: "user3@example.com", password: "pass789" }
    ]
  }) {
    created {
      id
      name
      email
    }
    successCount
    failureCount
    errors
  }
}
```

#### **3. Create with File Upload**

```typescript
// dtos/create-profile.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsNotEmpty, IsOptional, IsUrl } from 'class-validator';
import { GraphQLUpload, FileUpload } from 'graphql-upload';

@InputType()
export class CreateProfileInput {
  @Field()
  @IsNotEmpty()
  userId: string;

  @Field({ nullable: true })
  @IsOptional()
  bio?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsUrl()
  website?: string;

  @Field(() => GraphQLUpload, { nullable: true })
  @IsOptional()
  avatar?: FileUpload;
}

// profiles.resolver.ts
import { GraphQLUpload, FileUpload } from 'graphql-upload';

@Resolver(() => Profile)
export class ProfilesResolver {
  constructor(
    private readonly profilesService: ProfilesService,
    private readonly uploadService: UploadService,
  ) {}

  @Mutation(() => Profile)
  async createProfile(
    @Args('input') input: CreateProfileInput,
  ): Promise<Profile> {
    let avatarUrl: string | undefined;

    // Upload avatar if provided
    if (input.avatar) {
      avatarUrl = await this.uploadService.uploadFile(input.avatar);
    }

    return this.profilesService.create({
      ...input,
      avatarUrl,
    });
  }
}

// upload.service.ts
import { Injectable } from '@nestjs/common';
import { createWriteStream } from 'fs';
import { join } from 'path';
import { v4 as uuid } from 'uuid';
import { FileUpload } from 'graphql-upload';

@Injectable()
export class UploadService {
  async uploadFile(file: FileUpload): Promise<string> {
    const { createReadStream, filename, mimetype } = await file;

    // Validate file type
    if (!mimetype.startsWith('image/')) {
      throw new BadRequestException('Only images are allowed');
    }

    // Generate unique filename
    const uniqueFilename = `${uuid()}-${filename}`;
    const uploadPath = join(process.cwd(), 'uploads', uniqueFilename);

    // Save file
    await new Promise((resolve, reject) => {
      createReadStream()
        .pipe(createWriteStream(uploadPath))
        .on('finish', resolve)
        .on('error', reject);
    });

    // Return public URL
    return `/uploads/${uniqueFilename}`;
  }
}
```

**GraphQL Mutation:**
```graphql
mutation($avatar: Upload) {
  createProfile(input: {
    userId: "123"
    bio: "Software developer"
    website: "https://johndoe.com"
    avatar: $avatar
  }) {
    id
    bio
    avatarUrl
    createdAt
  }
}
```

### **Production-Grade Implementation**

```typescript
// ============================================
// Complete Create Mutation System
// ============================================

// models/user.model.ts
import { ObjectType, Field, ID, registerEnumType } from '@nestjs/graphql';
import { Column, Entity, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
}

export enum UserStatus {
  ACTIVE = 'ACTIVE',
  INACTIVE = 'INACTIVE',
  SUSPENDED = 'SUSPENDED',
}

registerEnumType(UserRole, { name: 'UserRole' });
registerEnumType(UserStatus, { name: 'UserStatus' });

@ObjectType()
@Entity('users')
export class User {
  @Field(() => ID)
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Field()
  @Column()
  name: string;

  @Field()
  @Column({ unique: true })
  email: string;

  // Password not exposed in GraphQL
  @Column()
  password: string;

  @Field(() => UserRole)
  @Column({ type: 'enum', enum: UserRole, default: UserRole.USER })
  role: UserRole;

  @Field(() => UserStatus)
  @Column({ type: 'enum', enum: UserStatus, default: UserStatus.ACTIVE })
  status: UserStatus;

  @Field()
  @CreateDateColumn()
  createdAt: Date;

  @Field()
  @UpdateDateColumn()
  updatedAt: Date;
}

// dtos/create-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { 
  IsEmail, 
  IsNotEmpty, 
  MinLength, 
  MaxLength,
  IsEnum, 
  Matches,
  IsOptional 
} from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field({ description: 'User full name' })
  @IsNotEmpty({ message: 'Name is required' })
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  @MaxLength(100, { message: 'Name must not exceed 100 characters' })
  name: string;

  @Field({ description: 'User email address' })
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field({ description: 'User password (min 8 characters)' })
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain uppercase, lowercase, number, and special character',
  })
  password: string;

  @Field(() => UserRole, { nullable: true, defaultValue: UserRole.USER })
  @IsEnum(UserRole)
  @IsOptional()
  role?: UserRole;

  @Field({ nullable: true })
  @IsOptional()
  @MaxLength(500)
  bio?: string;
}

// users.service.ts
import { 
  Injectable, 
  ConflictException, 
  BadRequestException,
  InternalServerErrorException 
} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, QueryRunner, DataSource } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { User, UserStatus } from './models/user.model';
import { CreateUserInput } from './dtos/create-user.input';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly dataSource: DataSource,
  ) {}

  /**
   * Create a new user with validation and transaction
   */
  async create(input: CreateUserInput): Promise<User> {
    // Create query runner for transaction
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Check if email already exists
      const existingUser = await queryRunner.manager.findOne(User, {
        where: { email: input.email.toLowerCase() },
      });

      if (existingUser) {
        throw new ConflictException(
          `User with email "${input.email}" already exists`
        );
      }

      // Hash password
      const hashedPassword = await this.hashPassword(input.password);

      // Create user entity
      const user = queryRunner.manager.create(User, {
        name: input.name.trim(),
        email: input.email.toLowerCase(),
        password: hashedPassword,
        role: input.role,
        status: UserStatus.ACTIVE,
        bio: input.bio?.trim(),
      });

      // Save user
      const savedUser = await queryRunner.manager.save(User, user);

      // Commit transaction
      await queryRunner.commitTransaction();

      // TODO: Send welcome email (async, non-blocking)
      this.sendWelcomeEmail(savedUser).catch(err => 
        console.error('Failed to send welcome email:', err)
      );

      return savedUser;

    } catch (error) {
      // Rollback transaction on error
      await queryRunner.rollbackTransaction();

      // Re-throw known errors
      if (error instanceof ConflictException || error instanceof BadRequestException) {
        throw error;
      }

      // Log and throw generic error
      console.error('Error creating user:', error);
      throw new InternalServerErrorException('Failed to create user');

    } finally {
      // Release query runner
      await queryRunner.release();
    }
  }

  /**
   * Hash password using bcrypt
   */
  private async hashPassword(password: string): Promise<string> {
    const saltRounds = 10;
    return bcrypt.hash(password, saltRounds);
  }

  /**
   * Send welcome email (async)
   */
  private async sendWelcomeEmail(user: User): Promise<void> {
    // Implementation: Send email via email service
    console.log(`Sending welcome email to ${user.email}`);
  }

  /**
   * Check if user exists by email
   */
  async existsByEmail(email: string): Promise<boolean> {
    const count = await this.userRepository.count({
      where: { email: email.toLowerCase() },
    });
    return count > 0;
  }

  /**
   * Find user by ID (throws if not found)
   */
  async findById(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException(`User with ID "${id}" not found`);
    }

    return user;
  }
}

// users.resolver.ts
import { 
  Resolver, 
  Mutation, 
  Args,
  Context,
} from '@nestjs/graphql';
import { 
  UseGuards, 
  UsePipes, 
  ValidationPipe,
  ForbiddenException,
} from '@nestjs/common';
import { User, UserRole } from './models/user.model';
import { UsersService } from './users.service';
import { CreateUserInput } from './dtos/create-user.input';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { Roles } from '../auth/decorators/roles.decorator';
import { CurrentUser } from '../auth/decorators/current-user.decorator';

@Resolver(() => User)
@UsePipes(new ValidationPipe({ 
  transform: true, 
  whitelist: true,
  forbidNonWhitelisted: true,
}))
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  /**
   * Public user registration
   */
  @Mutation(() => User, { 
    description: 'Register a new user (public)' 
  })
  async registerUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    // Public registration: force USER role
    return this.usersService.create({
      ...input,
      role: UserRole.USER,
    });
  }

  /**
   * Admin-only user creation (can set any role)
   */
  @Mutation(() => User, { 
    description: 'Create user with any role (admin only)' 
  })
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles(UserRole.ADMIN)
  async createUser(
    @Args('input') input: CreateUserInput,
    @CurrentUser() currentUser: User,
  ): Promise<User> {
    // Admins can create users with any role
    return this.usersService.create(input);
  }

  /**
   * Check if email is available
   */
  @Query(() => Boolean, { 
    description: 'Check if email is available for registration' 
  })
  async isEmailAvailable(
    @Args('email') email: string,
  ): Promise<boolean> {
    return !(await this.usersService.existsByEmail(email));
  }
}

// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './models/user.model';
import { UsersService } from './users.service';
import { UsersResolver } from './users.resolver';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService, UsersResolver],
  exports: [UsersService],
})
export class UsersModule {}
```

### **GraphQL Mutations (Client Usage)**

```graphql
# ============================================
# 1. Public registration
# ============================================
mutation RegisterUser {
  registerUser(input: {
    name: "John Doe"
    email: "john@example.com"
    password: "SecurePass123!"
  }) {
    id
    name
    email
    role
    status
    createdAt
  }
}

# ============================================
# 2. Admin create user with role
# ============================================
mutation CreateAdmin {
  createUser(input: {
    name: "Admin User"
    email: "admin@example.com"
    password: "AdminPass123!"
    role: ADMIN
  }) {
    id
    name
    email
    role
  }
}

# ============================================
# 3. Check email availability
# ============================================
query CheckEmail {
  isEmailAvailable(email: "test@example.com")
}

# ============================================
# 4. Create with variables
# ============================================
mutation RegisterUser($input: CreateUserInput!) {
  registerUser(input: $input) {
    id
    name
    email
  }
}

# Variables:
{
  "input": {
    "name": "Jane Smith",
    "email": "jane@example.com",
    "password": "SecurePass456!"
  }
}
```

### **Error Handling**

```typescript
// Custom GraphQL errors
import { GraphQLError } from 'graphql';

throw new GraphQLError('User with this email already exists', {
  extensions: {
    code: 'USER_ALREADY_EXISTS',
    email: input.email,
  },
});

// NestJS exceptions (auto-converted to GraphQL errors)
throw new ConflictException('Email already exists');
throw new BadRequestException('Invalid input data');
throw new UnauthorizedException('Authentication required');
throw new ForbiddenException('Insufficient permissions');
```

**GraphQL Error Response:**
```json
{
  "errors": [
    {
      "message": "User with email \"john@example.com\" already exists",
      "extensions": {
        "code": "CONFLICT",
        "exception": {
          "message": "User with email \"john@example.com\" already exists",
          "error": "Conflict",
          "statusCode": 409
        }
      }
    }
  ],
  "data": {
    "registerUser": null
  }
}
```

### **Best Practices**

1. **Use Input Types**: Always use `@InputType()` for mutation arguments, never raw scalars
2. **Validate Input**: Use `class-validator` decorators on Input Types
3. **Enable ValidationPipe**: Apply globally or per-resolver to auto-validate
4. **Hash Sensitive Data**: Always hash passwords before storing
5. **Use Transactions**: Wrap create operations in transactions for data consistency
6. **Check Uniqueness**: Verify unique constraints before creating
7. **Sanitize Input**: Trim strings, lowercase emails
8. **Return Created Entity**: Return the full created object for client cache updates
9. **Handle Errors**: Use proper HTTP exceptions (ConflictException, BadRequestException)
10. **Authorization**: Use guards to restrict who can create certain entities

**Interview Tip**: Implement create mutation using `@Mutation(() => ReturnType)` with `@Args('input') input: InputType`. **Pattern**: (1) Define **Input Type** with `@InputType()` and validation decorators, (2) **Service** validates uniqueness, hashes sensitive data, saves to database, (3) **Resolver** calls service and returns created entity. **Transaction**: Use QueryRunner for atomic operations. **Validation**: Apply `ValidationPipe` to auto-validate inputs, throw proper exceptions (ConflictException for duplicates, BadRequestException for invalid data). **Best practices**: Check uniqueness first, use transactions, sanitize inputs, hash passwords, return full created entity, use guards for authorization. **Production**: Add logging, send async emails/notifications, handle edge cases, proper error messages.

</details>

---

26. How do you implement a Mutation for updating data?

<details>
<summary><strong>Answer</strong></summary>

**Update mutations** modify existing data in the database. In NestJS, use `@Mutation()` with **Update Input Types** (often with **partial fields** using `PartialType`), find the entity by ID, validate permissions, merge changes, and return the updated entity. Production implementations include **optimistic locking**, **partial updates**, **authorization checks**, **audit logging**, and **handling non-existent entities**.

### **Basic Update Mutation**

```typescript
// dtos/update-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsOptional, IsEmail, MinLength, IsEnum } from 'class-validator';
import { UserRole } from '../models/user.model';

@InputType()
export class UpdateUserInput {
  @Field(() => ID)
  id: string; // Required: Which user to update

  @Field({ nullable: true })
  @IsOptional()
  @MinLength(2)
  name?: string;

  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  @Field(() => UserRole, { nullable: true })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @Field({ nullable: true })
  @IsOptional()
  bio?: string;
}

// users.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './models/user.model';
import { UsersService } from './users.service';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => User, { description: 'Update user' })
  async updateUser(
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    return this.usersService.update(input.id, input);
  }
}

// users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async update(id: string, input: UpdateUserInput): Promise<User> {
    // Find existing user
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException(`User with ID "${id}" not found`);
    }

    // Check email uniqueness if being updated
    if (input.email && input.email !== user.email) {
      const existingUser = await this.userRepository.findOne({
        where: { email: input.email },
      });
      
      if (existingUser) {
        throw new ConflictException('Email already in use');
      }
    }

    // Merge changes
    Object.assign(user, input);

    // Save and return
    return this.userRepository.save(user);
  }
}
```

**GraphQL Mutation:**
```graphql
mutation {
  updateUser(input: {
    id: "123"
    name: "John Updated"
    bio: "Software engineer at XYZ"
  }) {
    id
    name
    email
    bio
    updatedAt
  }
}
```

### **Advanced Examples**

#### **1. Using PartialType for DRY Updates**

```typescript
// dtos/update-user.input.ts
import { InputType, Field, ID, PartialType } from '@nestjs/graphql';
import { CreateUserInput } from './create-user.input';

// Automatically makes all CreateUserInput fields optional
@InputType()
export class UpdateUserInput extends PartialType(CreateUserInput) {
  @Field(() => ID)
  id: string; // Add ID field for updates
}

// Now UpdateUserInput has all fields from CreateUserInput as optional
// Plus the required id field
```

#### **2. Separate ID from Input**

```typescript
// dtos/update-user-data.input.ts
import { InputType, Field, PartialType } from '@nestjs/graphql';
import { CreateUserInput } from './create-user.input';

@InputType()
export class UpdateUserDataInput extends PartialType(CreateUserInput) {
  // All fields from CreateUserInput, made optional
  // No ID field here
}

// users.resolver.ts
@Resolver(() => User)
export class UsersResolver {
  @Mutation(() => User)
  async updateUser(
    @Args('id', { type: () => ID }) id: string,
    @Args('data') data: UpdateUserDataInput,
  ): Promise<User> {
    return this.usersService.update(id, data);
  }
}
```

**GraphQL Mutation:**
```graphql
mutation {
  updateUser(
    id: "123"
    data: { name: "New Name", bio: "New bio" }
  ) {
    id
    name
    bio
  }
}
```

#### **3. Update with Optimistic Locking**

```typescript
// models/user.model.ts
import { ObjectType, Field } from '@nestjs/graphql';
import { Entity, Column, VersionColumn } from 'typeorm';

@ObjectType()
@Entity()
export class User {
  // ... other fields ...

  @Field()
  @VersionColumn() // Auto-incremented on each update
  version: number;
}

// dtos/update-user.input.ts
@InputType()
export class UpdateUserInput extends PartialType(CreateUserInput) {
  @Field(() => ID)
  id: string;

  @Field(() => Int)
  version: number; // Required for optimistic locking
}

// users.service.ts
import { OptimisticLockVersionMismatchError } from 'typeorm';

async update(id: string, input: UpdateUserInput): Promise<User> {
  const user = await this.userRepository.findOne({ where: { id } });
  
  if (!user) {
    throw new NotFoundException(`User not found`);
  }

  // Check version for optimistic locking
  if (user.version !== input.version) {
    throw new ConflictException(
      'User has been modified by another process. Please refresh and try again.'
    );
  }

  // Merge changes (version will auto-increment)
  Object.assign(user, input);

  try {
    return await this.userRepository.save(user);
  } catch (error) {
    if (error instanceof OptimisticLockVersionMismatchError) {
      throw new ConflictException('Concurrent update detected');
    }
    throw error;
  }
}
```

#### **4. Partial Update with Nested Relations**

```typescript
// dtos/update-post.input.ts
import { InputType, Field, ID } from '@nestjs/graphql';
import { IsOptional, MinLength } from 'class-validator';

@InputType()
export class UpdatePostInput {
  @Field(() => ID)
  id: string;

  @Field({ nullable: true })
  @IsOptional()
  @MinLength(5)
  title?: string;

  @Field({ nullable: true })
  @IsOptional()
  content?: string;

  @Field(() => [String], { nullable: true })
  @IsOptional()
  tags?: string[]; // Update tags

  @Field({ nullable: true })
  @IsOptional()
  published?: boolean;
}

// posts.service.ts
async update(id: string, input: UpdatePostInput): Promise<Post> {
  const post = await this.postRepository.findOne({
    where: { id },
    relations: ['tags', 'author'],
  });

  if (!post) {
    throw new NotFoundException('Post not found');
  }

  // Update scalar fields
  if (input.title) post.title = input.title;
  if (input.content) post.content = input.content;
  if (input.published !== undefined) post.published = input.published;

  // Update tags if provided
  if (input.tags) {
    const tags = await this.findOrCreateTags(input.tags);
    post.tags = tags;
  }

  return this.postRepository.save(post);
}
```

### **Production-Grade Implementation**

```typescript
// ============================================
// Complete Update System
// ============================================

// users.service.ts
import { 
  Injectable, 
  NotFoundException, 
  ConflictException,
  ForbiddenException,
  BadRequestException 
} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, DataSource } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly dataSource: DataSource,
  ) {}

  /**
   * Update user with validation, authorization, and transaction
   */
  async update(
    id: string,
    input: UpdateUserInput,
    currentUser?: User,
  ): Promise<User> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Find existing user
      const user = await queryRunner.manager.findOne(User, {
        where: { id },
        lock: { mode: 'pessimistic_write' }, // Lock for update
      });

      if (!user) {
        throw new NotFoundException(`User with ID "${id}" not found`);
      }

      // Authorization check
      if (currentUser) {
        const canUpdate = 
          currentUser.id === user.id || 
          currentUser.role === UserRole.ADMIN;
        
        if (!canUpdate) {
          throw new ForbiddenException(
            'You can only update your own profile'
          );
        }

        // Only admins can change roles
        if (input.role && input.role !== user.role) {
          if (currentUser.role !== UserRole.ADMIN) {
            throw new ForbiddenException('Only admins can change roles');
          }
        }
      }

      // Validate email uniqueness if being changed
      if (input.email && input.email !== user.email) {
        const emailExists = await queryRunner.manager.findOne(User, {
          where: { email: input.email.toLowerCase() },
        });

        if (emailExists) {
          throw new ConflictException(
            `Email "${input.email}" is already in use`
          );
        }
      }

      // Update password if provided
      if (input.password) {
        input.password = await bcrypt.hash(input.password, 10);
      }

      // Merge changes (only provided fields)
      const updatedUser = queryRunner.manager.merge(User, user, {
        ...input,
        email: input.email?.toLowerCase(),
        updatedAt: new Date(),
      });

      // Save
      const savedUser = await queryRunner.manager.save(User, updatedUser);

      // Commit transaction
      await queryRunner.commitTransaction();

      // TODO: Log audit trail
      this.logUpdate(user, savedUser).catch(err =>
        console.error('Failed to log update:', err)
      );

      return savedUser;

    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }

  /**
   * Partial update (patch)
   */
  async patch(id: string, updates: Partial<User>): Promise<User> {
    const result = await this.userRepository.update(id, updates);
    
    if (result.affected === 0) {
      throw new NotFoundException(`User with ID "${id}" not found`);
    }

    return this.findById(id);
  }

  /**
   * Audit logging
   */
  private async logUpdate(oldUser: User, newUser: User): Promise<void> {
    const changes = this.getChanges(oldUser, newUser);
    console.log('User updated:', { id: newUser.id, changes });
    // TODO: Save to audit log table
  }

  /**
   * Get diff between old and new values
   */
  private getChanges(oldObj: any, newObj: any): Record<string, any> {
    const changes: Record<string, any> = {};
    
    for (const key of Object.keys(newObj)) {
      if (oldObj[key] !== newObj[key]) {
        changes[key] = { from: oldObj[key], to: newObj[key] };
      }
    }
    
    return changes;
  }

  async findById(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User not found`);
    }
    return user;
  }
}

// users.resolver.ts
import { UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { CurrentUser } from '../auth/decorators/current-user.decorator';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  /**
   * Update user (requires authentication)
   */
  @Mutation(() => User, { description: 'Update user profile' })
  @UseGuards(JwtAuthGuard)
  async updateUser(
    @Args('id', { type: () => ID }) id: string,
    @Args('data') data: UpdateUserDataInput,
    @CurrentUser() currentUser: User,
  ): Promise<User> {
    return this.usersService.update(id, data, currentUser);
  }

  /**
   * Update own profile (self-service)
   */
  @Mutation(() => User, { description: 'Update own profile' })
  @UseGuards(JwtAuthGuard)
  async updateMyProfile(
    @Args('data') data: UpdateUserDataInput,
    @CurrentUser() currentUser: User,
  ): Promise<User> {
    return this.usersService.update(currentUser.id, data, currentUser);
  }
}
```

### **GraphQL Mutations**

```graphql
# Update user by ID
mutation {
  updateUser(
    id: "123"
    data: {
      name: "Updated Name"
      bio: "New bio"
    }
  ) {
    id
    name
    bio
    updatedAt
  }
}

# Update own profile
mutation {
  updateMyProfile(data: {
    name: "My New Name"
    bio: "Updated bio"
  }) {
    id
    name
    bio
  }
}

# With optimistic locking
mutation {
  updateUser(input: {
    id: "123"
    version: 5
    name: "New Name"
  }) {
    id
    name
    version
  }
}
```

### **Best Practices**

1. **Use PartialType**: Extend from CreateInput with PartialType for DRY code
2. **Separate ID**: Pass ID as separate argument or include in input
3. **Optimistic Locking**: Use `@VersionColumn()` for concurrent update protection
4. **Pessimistic Locking**: Use `lock: { mode: 'pessimistic_write' }` in transactions
5. **Validate Uniqueness**: Check constraints when updating unique fields
6. **Authorization**: Verify user can update the resource
7. **Partial Updates**: Only update provided fields, don't overwrite with undefined
8. **Audit Trail**: Log what changed, who changed it, when
9. **Return Updated Entity**: Return full entity for client cache updates
10. **Handle Not Found**: Throw NotFoundException if entity doesn't exist

**Interview Tip**: Update mutation: `@Mutation(() => User)` with `@Args('id') id: string, @Args('data') data: UpdateInput`. **Pattern**: (1) Find entity by ID (throw NotFoundException if missing), (2) Validate authorization (user can update?), (3) Check uniqueness constraints, (4) Merge changes with `Object.assign()` or `merge()`, (5) Save and return. **PartialType**: Use `extends PartialType(CreateInput)` to make all fields optional. **Optimistic locking**: Add `@VersionColumn()`, check version before update. **Transaction**: Use QueryRunner with pessimistic lock. **Best practices**: Validate permissions, check uniqueness, only update provided fields, log audit trail, use transactions for data consistency. **Production**: Add version checking, audit logging, authorization guards, handle concurrent updates.

</details>

---

27. How do you implement a Mutation for deleting data?

<details>
<summary><strong>Answer</strong></summary>

Implement delete mutation: (1) Define **mutation method** with `@Mutation(() => Boolean)` or custom response, (2) Accept **ID argument** via `@Args('id')`, (3) Call **service delete method**, (4) Return **boolean** (success) or deleted object. Handle **authorization** (check ownership), **soft delete** (mark inactive), **cascade deletes** (relations), and **errors** (not found, foreign key constraints).

### **Basic Delete**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}

  async delete(id: string): Promise<boolean> {
    const result = await this.userRepo.delete(id);
    return result.affected > 0;
  }
}

// user.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    return this.userService.delete(id);
  }
}

// GraphQL mutation:
// mutation {
//   deleteUser(id: "1")
// }
```

### **Delete with Validation**

```typescript
import { NotFoundException } from '@nestjs/common';

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => Boolean)
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return this.userService.delete(id);
  }
}
```

### **Soft Delete**

```typescript
// user.entity.ts
@Entity()
@ObjectType()
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)
  id: string;

  @Column()
  @Field()
  name: string;

  @Column({ default: true })
  @Field()
  isActive: boolean;

  @DeleteDateColumn()
  deletedAt?: Date;
}

// user.service.ts
@Injectable()
export class UserService {
  async softDelete(id: string): Promise<User> {
    const user = await this.userRepo.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    await this.userRepo.softDelete(id);
    return { ...user, isActive: false };
  }

  async restore(id: string): Promise<User> {
    await this.userRepo.restore(id);
    return this.userRepo.findOne({ where: { id }, withDeleted: true });
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  // Soft delete (mark inactive)
  @Mutation(() => User)
  async deleteUser(@Args('id') id: string): Promise<User> {
    return this.userService.softDelete(id);
  }

  // Restore soft-deleted user
  @Mutation(() => User)
  async restoreUser(@Args('id') id: string): Promise<User> {
    return this.userService.restore(id);
  }
}
```

### **Delete with Authorization**

```typescript
import { UseGuards, ForbiddenException } from '@nestjs/common';
import { GqlAuthGuard } from './guards/gql-auth.guard';
import { CurrentUser } from './decorators/current-user.decorator';

@Resolver(() => Post)
export class PostResolver {
  constructor(private postService: PostService) {}

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)  // Require authentication
  async deletePost(
    @Args('id') id: string,
    @CurrentUser() currentUser: User,  // Get current user
  ): Promise<boolean> {
    const post = await this.postService.findById(id);
    
    if (!post) {
      throw new NotFoundException('Post not found');
    }
    
    // Check ownership
    if (post.authorId !== currentUser.id) {
      throw new ForbiddenException('You can only delete your own posts');
    }
    
    return this.postService.delete(id);
  }
}
```

### **Cascade Delete**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
    @InjectRepository(Post)
    private postRepo: Repository<Post>,
  ) {}

  async delete(id: string): Promise<boolean> {
    const user = await this.userRepo.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Delete related posts first (cascade)
    await this.postRepo.delete({ authorId: id });
    
    // Then delete user
    const result = await this.userRepo.delete(id);
    return result.affected > 0;
  }
}

// Or use TypeORM cascade:
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author, { cascade: ['remove'] })
  posts: Post[];
}
```

### **Bulk Delete**

```typescript
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => Int)
  async deleteUsers(
    @Args('ids', { type: () => [String] }) ids: string[],
  ): Promise<number> {
    const result = await this.userService.deleteMany(ids);
    return result.affected;
  }
}

// user.service.ts
async deleteMany(ids: string[]): Promise<{ affected: number }> {
  const result = await this.userRepo.delete(ids);
  return { affected: result.affected || 0 };
}

// GraphQL mutation:
// mutation {
//   deleteUsers(ids: ["1", "2", "3"])
// }
```

### **Delete with Custom Response**

```typescript
@ObjectType()
export class DeleteResponse {
  @Field()
  success: boolean;

  @Field()
  message: string;

  @Field(() => User, { nullable: true })
  deletedUser?: User;
}

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => DeleteResponse)
  async deleteUser(@Args('id') id: string): Promise<DeleteResponse> {
    try {
      const user = await this.userService.findById(id);
      
      if (!user) {
        return {
          success: false,
          message: 'User not found',
        };
      }
      
      await this.userService.delete(id);
      
      return {
        success: true,
        message: 'User deleted successfully',
        deletedUser: user,
      };
    } catch (error) {
      return {
        success: false,
        message: error.message,
      };
    }
  }
}
```

**Interview Tip**: Delete mutation returns **Boolean** (success) or deleted object. **Steps**: (1) Validate ID exists, (2) Check authorization (ownership/admin), (3) Handle **soft delete** (mark inactive, use `@DeleteDateColumn`) vs **hard delete** (remove from DB). **Cascade**: delete related entities first or use TypeORM cascade options. **Bulk delete**: accept array of IDs, return count affected. **Error handling**: throw NotFoundException (not found), ForbiddenException (not authorized). **Pattern**: `@Mutation(() => Boolean) async deleteUser(@Args('id') id: string)` calling `this.userService.delete(id)`.

</details>
28. How do you handle input validation in mutations?

<details>
<summary><strong>Answer</strong></summary>

Handle input validation using **class-validator** decorators on InputType classes + **ValidationPipe** globally. Apply validation rules (`@IsEmail()`, `@MinLength()`, `@Max()`) to InputType fields. Enable ValidationPipe in main.ts or GraphQLModule. Validation happens **automatically before resolver execution**. Errors return as GraphQL errors with validation messages.

### **Basic Validation with class-validator**

```typescript
// npm install class-validator class-transformer

import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, MinLength, MaxLength, IsOptional } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  @MaxLength(50, { message: 'Name cannot exceed 50 characters' })
  name: string;

  @Field()
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;
}

// Enable ValidationPipe in main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,  // Strip non-whitelisted properties
      forbidNonWhitelisted: true,  // Throw error for extra properties
      transform: true,  // Auto-transform to DTO types
    }),
  );
  
  await app.listen(3000);
}
```

### **Common Validation Decorators**

```typescript
import {
  IsEmail,
  MinLength,
  MaxLength,
  Min,
  Max,
  IsInt,
  IsPositive,
  IsUrl,
  IsDate,
  IsEnum,
  Matches,
  IsOptional,
  ValidateNested,
} from 'class-validator';
import { Type } from 'class-transformer';

enum UserRole {
  USER = 'USER',
  ADMIN = 'ADMIN',
}

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  @MaxLength(50)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @Field(() => Int, { nullable: true })
  @IsOptional()
  @IsInt()
  @Min(18)
  @Max(120)
  age?: number;

  @Field({ nullable: true })
  @IsOptional()
  @IsUrl()
  website?: string;

  @Field(() => UserRole, { nullable: true })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}
```

### **Nested Object Validation**

```typescript
@InputType()
export class AddressInput {
  @Field()
  @MinLength(5)
  street: string;

  @Field()
  @MinLength(2)
  city: string;

  @Field()
  @MinLength(2)
  country: string;
}

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2)
  name: string;

  @Field()
  @IsEmail()
  email: string;

  @Field(() => AddressInput)
  @ValidateNested()  // Validate nested object
  @Type(() => AddressInput)  // Transform to class instance
  address: AddressInput;
}

// GraphQL mutation:
// mutation {
//   createUser(input: {
//     name: "John"
//     email: "john@example.com"
//     address: {
//       street: "123 Main St"
//       city: "NYC"
//       country: "USA"
//     }
//   }) { id }
// }
```

### **Array Validation**

```typescript
import { ArrayMinSize, ArrayMaxSize, IsArray } from 'class-validator';

@InputType()
export class CreatePostInput {
  @Field()
  @MinLength(5)
  title: string;

  @Field(() => [String])
  @IsArray()
  @ArrayMinSize(1, { message: 'At least one tag is required' })
  @ArrayMaxSize(10, { message: 'Maximum 10 tags allowed' })
  tags: string[];
}
```

### **Custom Validation**

```typescript
import { registerDecorator, ValidationOptions, ValidatorConstraint, ValidatorConstraintInterface } from 'class-validator';

// Custom validator: email must be unique
@ValidatorConstraint({ async: true })
export class IsEmailUniqueConstraint implements ValidatorConstraintInterface {
  constructor(private userService: UserService) {}

  async validate(email: string): Promise<boolean> {
    const user = await this.userService.findByEmail(email);
    return !user;  // Return true if email is unique
  }

  defaultMessage(): string {
    return 'Email already exists';
  }
}

// Custom decorator
export function IsEmailUnique(validationOptions?: ValidationOptions) {
  return function (object: object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsEmailUniqueConstraint,
    });
  };
}

// Usage
@InputType()
export class CreateUserInput {
  @Field()
  @IsEmail()
  @IsEmailUnique({ message: 'Email is already taken' })
  email: string;
}
```

### **Validation Error Handling**

```typescript
// Validation errors are automatically converted to GraphQL errors

// Invalid input:
// mutation {
//   createUser(input: {
//     name: "A"  // Too short
//     email: "invalid-email"  // Invalid format
//     password: "123"  // Too short
//   }) { id }
// }

// GraphQL error response:
// {
//   "errors": [
//     {
//       "message": "Bad Request Exception",
//       "extensions": {
//         "code": "BAD_USER_INPUT",
//         "response": {
//           "message": [
//             "Name must be at least 2 characters",
//             "Invalid email format",
//             "Password must be at least 8 characters"
//           ],
//           "error": "Bad Request",
//           "statusCode": 400
//         }
//       }
//     }
//   ]
// }
```

### **Complete Example**

```typescript
// user.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { IsEmail, MinLength, Min, Max, IsOptional } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  name: string;

  @Field()
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;

  @Field(() => Int, { nullable: true })
  @IsOptional()
  @Min(18, { message: 'Must be at least 18 years old' })
  @Max(120, { message: 'Invalid age' })
  age?: number;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,  // Validated automatically
  ): Promise<User> {
    // If validation fails, this code never executes
    return this.userService.create(input);
  }
}

// main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );
  
  await app.listen(3000);
}
bootstrap();
```

**Interview Tip**: Input validation uses **class-validator** decorators on InputType classes + **ValidationPipe** enabled globally. **Common decorators**: `@IsEmail()`, `@MinLength()`, `@Max()`, `@IsOptional()`, `@ValidateNested()` (nested objects), `@ArrayMinSize()` (arrays). **Enable**: `app.useGlobalPipes(new ValidationPipe({ whitelist: true }))` in main.ts. **Automatic**: validation happens before resolver execution, errors returned as GraphQL errors with messages. **Custom validators**: implement `ValidatorConstraintInterface` for complex rules (e.g., unique email check). **Options**: `whitelist: true` (strip extra fields), `transform: true` (auto-convert types).

</details>

---

## Subscriptions

29. What are GraphQL Subscriptions?

<details>
<summary><strong>Answer</strong></summary>

**GraphQL Subscriptions** enable **real-time, bidirectional communication** between client and server over **WebSocket**. Unlike queries (one-time read) and mutations (one-time write), subscriptions maintain **persistent connection** and **push updates** to clients when events occur. Use for: live notifications, chat messages, real-time dashboards, collaborative editing.

### **How Subscriptions Work**

```typescript
// 1. Client subscribes
subscription {
  userCreated {  // Subscription starts
    id
    name
  }
}

// 2. Server publishes event when user is created
// 3. Client receives update automatically
// 4. Connection stays open until client unsubscribes
```

### **Subscriptions vs Queries/Mutations**

| **Feature** | **Query** | **Mutation** | **Subscription** |
|------------|-----------|--------------|------------------|
| **Operation** | Read data | Write data | Real-time updates |
| **Protocol** | HTTP | HTTP | WebSocket |
| **Connection** | One-time | One-time | Persistent |
| **Direction** | Client → Server | Client → Server | Server → Client |
| **Use case** | Fetch data | Create/update/delete | Live updates |
| **Example** | Get users | Create user | User created event |

### **When to Use Subscriptions**

```typescript
// ✅ Good use cases:
// - Live chat messages
// - Real-time notifications
// - Live sports scores
// - Stock price updates
// - Collaborative editing (Google Docs-like)
// - IoT sensor data
// - Live auction bids

// ❌ Don't use for:
// - Initial data loading (use Query)
// - One-time updates (use Mutation + refetch)
// - Infrequent updates (polling is simpler)
// - Large data transfers (use Query with pagination)
```

### **Basic Subscription Setup**

```typescript
// Install dependencies
// npm install graphql-subscriptions graphql-ws

// app.module.ts
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      subscriptions: {
        'graphql-ws': true,  // Enable WebSocket for subscriptions
      },
    }),
  ],
})
export class AppModule {}
```

### **Simple Subscription Example**

```typescript
// pubsub.ts - Create PubSub instance
import { PubSub } from 'graphql-subscriptions';

export const pubSub = new PubSub();

// user.resolver.ts
import { Resolver, Mutation, Subscription, Args } from '@nestjs/graphql';
import { pubSub } from './pubsub';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Mutation: Create user and publish event
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    const user = await this.userService.create(input);
    
    // Publish event
    pubSub.publish('userCreated', { userCreated: user });
    
    return user;
  }

  // Subscription: Listen for userCreated events
  @Subscription(() => User)
  userCreated() {
    return pubSub.asyncIterator('userCreated');
  }
}

// Generated schema:
// type Mutation {
//   createUser(input: CreateUserInput!): User!
// }
// 
// type Subscription {
//   userCreated: User!
// }

// Client subscription:
// subscription {
//   userCreated {
//     id
//     name
//     email
//   }
// }
```

### **Subscription with Filters**

```typescript
@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  async createPost(
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    const post = await this.postService.create(input);
    
    // Publish with categoryId
    pubSub.publish('postCreated', {
      postCreated: post,
      categoryId: post.categoryId,
    });
    
    return post;
  }

  @Subscription(() => Post, {
    filter: (payload, variables) => {
      // Only send to clients subscribed to this category
      return payload.categoryId === variables.categoryId;
    },
  })
  postCreated(
    @Args('categoryId') categoryId: string,
  ) {
    return pubSub.asyncIterator('postCreated');
  }
}

// Client subscription (filtered):
// subscription {
//   postCreated(categoryId: "tech") {
//     id
//     title
//   }
// }
```

### **Real-World Use Cases**

```typescript
// 1. Live Chat
@Subscription(() => Message)
messageAdded(
  @Args('roomId') roomId: string,
) {
  return pubSub.asyncIterator(`message_${roomId}`);
}

// 2. Notifications
@Subscription(() => Notification, {
  filter: (payload, variables, context) => {
    return payload.userId === context.req.user.id;
  },
})
notificationReceived() {
  return pubSub.asyncIterator('notification');
}

// 3. Live Dashboard
@Subscription(() => DashboardUpdate)
dashboardUpdate() {
  return pubSub.asyncIterator('dashboardUpdate');
}

// 4. Typing Indicator
@Subscription(() => TypingStatus)
UserTyping(
  @Args('roomId') roomId: string,
) {
  return pubSub.asyncIterator(`typing_${roomId}`);
}
```

**Interview Tip**: **Subscriptions** enable **real-time updates** via **WebSocket**, maintaining persistent connection. Unlike queries (one-time read) or mutations (one-time write), subscriptions **push data** to clients when events occur. **Setup**: install `graphql-subscriptions` + `graphql-ws`, enable `subscriptions: { 'graphql-ws': true }` in GraphQLModule. **Pattern**: (1) Mutation publishes event via `pubSub.publish('eventName', data)`, (2) Subscription listens via `pubSub.asyncIterator('eventName')`, (3) Clients receive updates automatically. **Use for**: chat, notifications, live dashboards, collaborative editing. **Filters**: use `filter` option to send updates only to specific clients.

</details>
30. How do you implement real-time updates using Subscriptions?

<details>
<summary><strong>Answer</strong></summary>

Implement real-time updates: (1) **Install** `graphql-subscriptions` + `graphql-ws`, (2) **Enable** WebSocket in GraphQLModule config, (3) **Create PubSub** instance, (4) **Publish events** in mutations via `pubSub.publish()`, (5) **Subscribe** via `@Subscription()` decorator with `pubSub.asyncIterator()`. Clients connect via WebSocket and receive updates automatically.

### **Setup Subscriptions**

```typescript
// 1. Install dependencies
// npm install graphql-subscriptions graphql-ws

// 2. Configure GraphQL module
// app.module.ts
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      playground: true,
      subscriptions: {
        'graphql-ws': {  // Enable WebSocket
          path: '/graphql',
        },
      },
    }),
  ],
})
export class AppModule {}

// 3. Create PubSub provider
// pubsub.provider.ts
import { PubSub } from 'graphql-subscriptions';

export const PUB_SUB = 'PUB_SUB';

export const pubSubProvider = {
  provide: PUB_SUB,
  useValue: new PubSub(),
};

// Register in module
@Module({
  providers: [pubSubProvider, UserResolver],
  exports: [PUB_SUB],
})
export class UserModule {}
```

### **Complete Real-Time Example**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;
}

// user.resolver.ts
import { Resolver, Mutation, Subscription, Args, Inject } from '@nestjs/graphql';
import { PubSub } from 'graphql-subscriptions';
import { PUB_SUB } from './pubsub.provider';

@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  // Mutation: Create user and publish event
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    const user = await this.userService.create(input);
    
    // Publish event to subscribers
    await this.pubSub.publish('userCreated', {
      userCreated: user,
    });
    
    return user;
  }

  // Mutation: Update user and publish event
  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    const user = await this.userService.update(id, input);
    
    await this.pubSub.publish('userUpdated', {
      userUpdated: user,
    });
    
    return user;
  }

  // Subscription: Listen for new users
  @Subscription(() => User)
  userCreated() {
    return this.pubSub.asyncIterator('userCreated');
  }

  // Subscription: Listen for user updates
  @Subscription(() => User)
  userUpdated() {
    return this.pubSub.asyncIterator('userUpdated');
  }
}

// Client usage:
// subscription {
//   userCreated {
//     id
//     name
//     email
//   }
// }
```

### **Chat Application Example**

```typescript
// message.model.ts
@ObjectType()
export class Message {
  @Field(() => ID)
  id: string;

  @Field()
  text: string;

  @Field(() => User)
  author: User;

  @Field()
  roomId: string;

  @Field()
  createdAt: Date;
}

// message.resolver.ts
@Resolver(() => Message)
export class MessageResolver {
  constructor(
    private messageService: MessageService,
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  // Send message mutation
  @Mutation(() => Message)
  @UseGuards(GqlAuthGuard)
  async sendMessage(
    @Args('roomId') roomId: string,
    @Args('text') text: string,
    @CurrentUser() user: User,
  ): Promise<Message> {
    const message = await this.messageService.create({
      text,
      roomId,
      authorId: user.id,
    });

    // Publish to room subscribers
    await this.pubSub.publish(`messageAdded_${roomId}`, {
      messageAdded: message,
      roomId,
    });

    return message;
  }

  // Subscribe to room messages
  @Subscription(() => Message, {
    filter: (payload, variables) => {
      // Only send to clients in this room
      return payload.roomId === variables.roomId;
    },
  })
  messageAdded(
    @Args('roomId') roomId: string,
  ) {
    return this.pubSub.asyncIterator(`messageAdded_${roomId}`);
  }
}

// Client:
// subscription {
//   messageAdded(roomId: "room1") {
//     id
//     text
//     author { name }
//     createdAt
//   }
// }
```

### **Notification System**

```typescript
@ObjectType()
export class Notification {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field()
  message: string;

  @Field()
  userId: string;
}

@Resolver(() => Notification)
export class NotificationResolver {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  // Subscription with authentication filter
  @Subscription(() => Notification, {
    filter: (payload, variables, context) => {
      // Only send to notification owner
      return payload.notification.userId === context.req.user?.id;
    },
  })
  @UseGuards(GqlAuthGuard)
  notificationReceived() {
    return this.pubSub.asyncIterator('notification');
  }
}

// Publish notification from any service
@Injectable()
export class OrderService {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  async createOrder(userId: string, orderData: any) {
    const order = await this.orderRepo.save(orderData);

    // Send notification
    await this.pubSub.publish('notification', {
      notification: {
        id: uuid(),
        title: 'Order Created',
        message: `Your order #${order.id} has been placed`,
        userId,
      },
    });

    return order;
  }
}
```

### **Live Dashboard Updates**

```typescript
@ObjectType()
export class DashboardStats {
  @Field(() => Int)
  totalUsers: number;

  @Field(() => Int)
  activeUsers: number;

  @Field(() => Float)
  revenue: number;

  @Field()
  updatedAt: Date;
}

@Resolver()
export class DashboardResolver {
  constructor(
    private statsService: StatsService,
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {
    // Update stats every 5 seconds
    setInterval(async () => {
      const stats = await this.statsService.getLatest();
      await this.pubSub.publish('dashboardUpdated', {
        dashboardUpdated: stats,
      });
    }, 5000);
  }

  @Subscription(() => DashboardStats)
  @UseGuards(GqlAuthGuard, AdminGuard)
  dashboardUpdated() {
    return this.pubSub.asyncIterator('dashboardUpdated');
  }
}
```

### **Production PubSub with Redis**

```typescript
// For production, use Redis-based PubSub
// npm install graphql-redis-subscriptions ioredis

import { RedisPubSub } from 'graphql-redis-subscriptions';
import Redis from 'ioredis';

const options = {
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT),
  password: process.env.REDIS_PASSWORD,
};

export const pubSubProvider = {
  provide: PUB_SUB,
  useValue: new RedisPubSub({
    publisher: new Redis(options),
    subscriber: new Redis(options),
  }),
};

// Scales across multiple server instances
```

**Interview Tip**: Implement real-time updates: (1) **Install** `graphql-subscriptions graphql-ws`, (2) **Enable** `subscriptions: { 'graphql-ws': true }` in GraphQLModule, (3) **Create PubSub** provider, (4) In **mutations**: `pubSub.publish('eventName', { data })`, (5) In **subscriptions**: `@Subscription(() => Type)` with `pubSub.asyncIterator('eventName')`. **Filters**: use `filter` option to send only to specific clients. **Production**: use **Redis PubSub** for horizontal scaling. **Use cases**: chat, notifications, live dashboards, collaborative editing. **Client** connects via WebSocket and receives updates automatically when events published.

</details>
31. What is `@Subscription()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**`@Subscription()`** marks a method as **GraphQL subscription resolver**, defining which events clients can subscribe to. Applied to methods in `@Resolver()` class, takes **return type** and **options** (filter, resolve). Method returns **async iterator** from `pubSub.asyncIterator('eventName')`. When event published, subscribers receive updates via WebSocket.

### **Basic Usage**

```typescript
import { Resolver, Subscription } from '@nestjs/graphql';
import { PubSub } from 'graphql-subscriptions';

const pubSub = new PubSub();

@Resolver(() => User)
export class UserResolver {
  @Subscription(() => User)  // Return type
  userCreated() {
    return pubSub.asyncIterator('userCreated');
  }
}

// Generated schema:
// type Subscription {
//   userCreated: User!
// }

// Client:
// subscription {
//   userCreated {
//     id
//     name
//   }
// }
```

### **With Arguments**

```typescript
@Resolver(() => Message)
export class MessageResolver {
  @Subscription(() => Message)
  messageAdded(
    @Args('roomId') roomId: string,
  ) {
    return pubSub.asyncIterator(`message_${roomId}`);
  }
}

// Generated schema:
// type Subscription {
//   messageAdded(roomId: String!): Message!
// }

// Client:
// subscription {
//   messageAdded(roomId: "room1") {
//     id
//     text
//   }
// }
```

### **With Filter**

```typescript
@Resolver(() => Post)
export class PostResolver {
  @Subscription(() => Post, {
    filter: (payload, variables) => {
      // Only send if post matches filter
      return payload.post.categoryId === variables.categoryId;
    },
  })
  postCreated(
    @Args('categoryId') categoryId: string,
  ) {
    return pubSub.asyncIterator('postCreated');
  }
}

// Only receives posts for subscribed category
```

### **With Resolve (Transform Data)**

```typescript
@Resolver(() => Notification)
export class NotificationResolver {
  @Subscription(() => Notification, {
    resolve: (payload) => {
      // Transform payload before sending to client
      return {
        ...payload.notification,
        formattedDate: new Date(payload.notification.createdAt).toLocaleDateString(),
      };
    },
  })
  notificationReceived() {
    return pubSub.asyncIterator('notification');
  }
}
```

### **With Authentication**

```typescript
import { UseGuards } from '@nestjs/common';
import { GqlAuthGuard } from './guards/gql-auth.guard';

@Resolver(() => Message)
export class MessageResolver {
  @Subscription(() => Message, {
    filter: (payload, variables, context) => {
      // Only send to authorized users in room
      const userId = context.req.user?.id;
      return payload.roomMembers.includes(userId);
    },
  })
  @UseGuards(GqlAuthGuard)  // Require authentication
  messageAdded(
    @Args('roomId') roomId: string,
  ) {
    return pubSub.asyncIterator(`message_${roomId}`);
  }
}
```

### **Multiple Event Sources**

```typescript
@Resolver(() => User)
export class UserResolver {
  @Subscription(() => User, {
    resolve: (payload) => {
      // payload can come from userCreated OR userUpdated
      return payload.userCreated || payload.userUpdated;
    },
  })
  userChanged() {
    // Listen to multiple events
    return pubSub.asyncIterator(['userCreated', 'userUpdated']);
  }
}
```

### **Options Reference**

```typescript
@Subscription(() => ReturnType, {
  // Filter: decide if payload should be sent to this client
  filter: (payload, variables, context) => boolean,
  
  // Resolve: transform payload before sending
  resolve: (payload, variables, context) => transformedPayload,
  
  // Name: custom subscription name in schema
  name: 'customName',
  
  // Description: schema documentation
  description: 'Subscription description',
  
  // Nullable: allow null return
  nullable: true,
})
```

### **Complete Example**

```typescript
// chat.resolver.ts
import { Resolver, Mutation, Subscription, Args, Inject } from '@nestjs/graphql';
import { PubSub } from 'graphql-subscriptions';
import { UseGuards } from '@nestjs/common';
import { GqlAuthGuard } from './guards/gql-auth.guard';
import { CurrentUser } from './decorators/current-user.decorator';

@Resolver(() => Message)
export class ChatResolver {
  constructor(
    private chatService: ChatService,
    @Inject('PUB_SUB') private pubSub: PubSub,
  ) {}

  // Mutation: Send message
  @Mutation(() => Message)
  @UseGuards(GqlAuthGuard)
  async sendMessage(
    @Args('roomId') roomId: string,
    @Args('text') text: string,
    @CurrentUser() user: User,
  ): Promise<Message> {
    const message = await this.chatService.create({
      roomId,
      text,
      authorId: user.id,
    });

    // Publish event
    await this.pubSub.publish('messageAdded', {
      messageAdded: message,
      roomId,
    });

    return message;
  }

  // Subscription: Listen for messages
  @Subscription(() => Message, {
    name: 'messageAdded',
    description: 'Receive new messages in a room',
    filter: (payload, variables) => {
      // Only send messages for subscribed room
      return payload.roomId === variables.roomId;
    },
  })
  @UseGuards(GqlAuthGuard)
  messageAdded(
    @Args('roomId') roomId: string,
  ) {
    return this.pubSub.asyncIterator('messageAdded');
  }

  // Subscription: Typing indicator
  @Subscription(() => TypingStatus)
  @UseGuards(GqlAuthGuard)
  userTyping(
    @Args('roomId') roomId: string,
  ) {
    return this.pubSub.asyncIterator(`typing_${roomId}`);
  }
}

// Client usage:
// subscription {
//   messageAdded(roomId: "room1") {
//     id
//     text
//     author { name }
//     createdAt
//   }
// }
```

**Interview Tip**: **`@Subscription()`** marks method as **subscription resolver** for real-time updates. Applied to methods in `@Resolver()`, takes **return type** `() => Type` and **options**. Method returns **`pubSub.asyncIterator('eventName')`**. **Options**: `filter` (decide which clients receive update), `resolve` (transform payload), `name` (custom subscription name). **With args**: `@Args('roomId')` for subscription parameters. **Authentication**: use `@UseGuards(GqlAuthGuard)` + `filter` with context to check user permissions. **Multiple events**: pass array to `asyncIterator(['event1', 'event2'])`. **Pattern**: Mutation publishes event → Subscription listens → Clients receive updates via WebSocket.

</details>
32. How do you use PubSub for subscriptions?

<details>
<summary><strong>Answer</strong></summary>

**PubSub** (Publish-Subscribe) manages subscription events: **publish** events from mutations/services, **subscribe** via `asyncIterator()` in subscription resolvers. Create **PubSub instance** as provider, inject into resolvers. Use `pubSub.publish('eventName', { data })` to send updates, `pubSub.asyncIterator('eventName')` to listen. For production, use **Redis PubSub** for horizontal scaling.

### **Basic PubSub Setup**

```typescript
// 1. Create PubSub provider
// pubsub.provider.ts
import { PubSub } from 'graphql-subscriptions';

export const PUB_SUB = 'PUB_SUB';

export const pubSubProvider = {
  provide: PUB_SUB,
  useValue: new PubSub(),
};

// 2. Register in module
// user.module.ts
import { Module } from '@nestjs/common';
import { pubSubProvider } from './pubsub.provider';

@Module({
  providers: [
    pubSubProvider,  // Register PubSub
    UserResolver,
    UserService,
  ],
  exports: [PUB_SUB],  // Export for other modules
})
export class UserModule {}

// 3. Inject and use in resolver
// user.resolver.ts
import { Resolver, Mutation, Subscription, Args, Inject } from '@nestjs/graphql';
import { PubSub } from 'graphql-subscriptions';
import { PUB_SUB } from './pubsub.provider';

@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    @Inject(PUB_SUB) private pubSub: PubSub,  // Inject PubSub
  ) {}

  // Publish: In mutation
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    const user = await this.userService.create(input);
    
    // Publish event
    await this.pubSub.publish('userCreated', {
      userCreated: user,  // Must match subscription field name
    });
    
    return user;
  }

  // Subscribe: In subscription
  @Subscription(() => User)
  userCreated() {
    return this.pubSub.asyncIterator('userCreated');  // Listen for events
  }
}
```

### **PubSub Methods**

```typescript
import { PubSub } from 'graphql-subscriptions';

const pubSub = new PubSub();

// 1. Publish event
await pubSub.publish('eventName', {
  fieldName: data,  // Payload
});

// 2. Subscribe to events
const asyncIterator = pubSub.asyncIterator('eventName');

// 3. Subscribe to multiple events
const asyncIterator = pubSub.asyncIterator(['event1', 'event2']);

// 4. Unsubscribe (automatic when client disconnects)
```

### **Publishing from Service**

```typescript
// You can publish from anywhere, not just mutations

// order.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { PubSub } from 'graphql-subscriptions';
import { PUB_SUB } from './pubsub.provider';

@Injectable()
export class OrderService {
  constructor(
    @InjectRepository(Order)
    private orderRepo: Repository<Order>,
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  async create(userId: string, orderData: any): Promise<Order> {
    const order = await this.orderRepo.save({
      ...orderData,
      userId,
      status: 'PENDING',
    });

    // Publish from service
    await this.pubSub.publish('orderCreated', {
      orderCreated: order,
      userId,
    });

    return order;
  }

  async updateStatus(orderId: string, status: string): Promise<Order> {
    await this.orderRepo.update(orderId, { status });
    const order = await this.orderRepo.findOne({ where: { id: orderId } });

    // Publish status update
    await this.pubSub.publish('orderStatusChanged', {
      orderStatusChanged: order,
      userId: order.userId,
    });

    return order;
  }
}
```

### **Multiple Event Channels**

```typescript
@Resolver(() => Post)
export class PostResolver {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  @Mutation(() => Post)
  async createPost(
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    const post = await this.postService.create(input);

    // Publish to multiple channels
    await this.pubSub.publish('postCreated', {
      postCreated: post,
    });

    // Also publish category-specific event
    await this.pubSub.publish(`post_${post.categoryId}`, {
      postInCategory: post,
    });

    return post;
  }

  // General subscription
  @Subscription(() => Post)
  postCreated() {
    return this.pubSub.asyncIterator('postCreated');
  }

  // Category-specific subscription
  @Subscription(() => Post)
  postInCategory(
    @Args('categoryId') categoryId: string,
  ) {
    return this.pubSub.asyncIterator(`post_${categoryId}`);
  }
}
```

### **Dynamic Event Names**

```typescript
@Resolver(() => Message)
export class MessageResolver {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  @Mutation(() => Message)
  async sendMessage(
    @Args('roomId') roomId: string,
    @Args('text') text: string,
  ): Promise<Message> {
    const message = await this.messageService.create({ roomId, text });

    // Dynamic event name per room
    await this.pubSub.publish(`message_room_${roomId}`, {
      messageAdded: message,
    });

    return message;
  }

  @Subscription(() => Message)
  messageAdded(
    @Args('roomId') roomId: string,
  ) {
    // Dynamic channel per room
    return this.pubSub.asyncIterator(`message_room_${roomId}`);
  }
}
```

### **Redis PubSub (Production)**

```typescript
// For production with multiple server instances
// npm install graphql-redis-subscriptions ioredis

import { RedisPubSub } from 'graphql-redis-subscriptions';
import Redis from 'ioredis';

const redisOptions = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => {
    return Math.min(times * 50, 2000);
  },
};

export const pubSubProvider = {
  provide: PUB_SUB,
  useFactory: () => {
    return new RedisPubSub({
      publisher: new Redis(redisOptions),
      subscriber: new Redis(redisOptions),
    });
  },
};

// Scales horizontally - events shared across all server instances
```

### **Complete Example**

```typescript
// pubsub.module.ts
import { Module, Global } from '@nestjs/common';
import { PubSub } from 'graphql-subscriptions';

export const PUB_SUB = 'PUB_SUB';

@Global()  // Make available everywhere
@Module({
  providers: [
    {
      provide: PUB_SUB,
      useValue: new PubSub(),
    },
  ],
  exports: [PUB_SUB],
})
export class PubSubModule {}

// notification.service.ts
@Injectable()
export class NotificationService {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  async sendToUser(userId: string, notification: any): Promise<void> {
    await this.pubSub.publish('notification', {
      notification: {
        ...notification,
        userId,
        createdAt: new Date(),
      },
    });
  }

  async sendToAll(notification: any): Promise<void> {
    await this.pubSub.publish('globalNotification', {
      globalNotification: notification,
    });
  }
}

// notification.resolver.ts
@Resolver(() => Notification)
export class NotificationResolver {
  constructor(
    @Inject(PUB_SUB) private pubSub: PubSub,
  ) {}

  @Subscription(() => Notification, {
    filter: (payload, variables, context) => {
      return payload.notification.userId === context.req.user.id;
    },
  })
  @UseGuards(GqlAuthGuard)
  notification() {
    return this.pubSub.asyncIterator('notification');
  }

  @Subscription(() => Notification)
  globalNotification() {
    return this.pubSub.asyncIterator('globalNotification');
  }
}
```

**Interview Tip**: **PubSub** manages subscription events via **publish-subscribe pattern**. **Setup**: create provider `new PubSub()`, inject via `@Inject(PUB_SUB)`. **Publish**: `pubSub.publish('eventName', { fieldName: data })` from mutations/services. **Subscribe**: `pubSub.asyncIterator('eventName')` in subscription methods. **Payload key** must match subscription field name. **Dynamic channels**: use template strings for per-room/user events (`message_${roomId}`). **Production**: use **RedisPubSub** for horizontal scaling across multiple servers. **Pattern**: Mutation/Service publishes → PubSub broadcasts → Subscriptions receive → Clients get updates via WebSocket.

</details>
33. What is the difference between WebSockets and Subscriptions?

<details>
<summary><strong>Answer</strong></summary>

**WebSockets** is the **transport protocol** (bidirectional TCP connection), while **GraphQL Subscriptions** is the **application-level feature** built on top of WebSockets. WebSockets = **how data travels** (low-level), Subscriptions = **what data travels** (high-level, typed schema). Subscriptions use WebSocket for communication but add GraphQL features: type safety, schema validation, filtering, authentication.

### **Comparison Table**

| **Aspect** | **WebSockets** | **GraphQL Subscriptions** |
|-----------|---------------|---------------------------|
| **Level** | Protocol/Transport | Application/Feature |
| **What** | Bidirectional TCP connection | Real-time GraphQL operations |
| **Data Format** | Any (text, binary, JSON) | Typed GraphQL schema |
| **Schema** | No schema | Strongly typed schema |
| **Validation** | None | Automatic validation |
| **Filtering** | Manual | Built-in filter support |
| **Authentication** | Manual | Integrated with Guards |
| **Query Language** | N/A | GraphQL syntax |
| **Example** | Raw socket.send(data) | GraphQL subscription query |

### **WebSocket (Low-Level)**

```typescript
// Raw WebSocket server
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // Send any data, no type safety
  sendMessage(message: any) {
    this.server.emit('message', message);  // No validation
  }
}

// Client
const socket = io('http://localhost:3000');
socket.on('message', (data) => {
  console.log(data);  // Unknown structure
});
```

### **GraphQL Subscription (High-Level)**

```typescript
// GraphQL subscription with schema
@ObjectType()
export class Message {
  @Field(() => ID)
  id: string;

  @Field()
  text: string;

  @Field(() => User)
  author: User;
}

@Resolver(() => Message)
export class MessageResolver {
  @Subscription(() => Message)  // Typed return
  messageAdded() {
    return pubSub.asyncIterator('messageAdded');
  }
}

// Client (type-safe)
subscription {
  messageAdded {
    id
    text
    author { name }  # Structured, validated query
  }
}
```

### **Architecture**

```typescript
// Layered architecture:

// ┌─────────────────────────────────────┐
// │   GraphQL Subscriptions Layer       │
// │   - Schema validation               │
// │   - Type checking                   │
// │   - Filtering                       │
// │   - Authentication                  │
// └─────────────────────────────────────┘
//              ↕
// ┌─────────────────────────────────────┐
// │   WebSocket Protocol Layer          │
// │   - Connection management           │
// │   - Bidirectional communication     │
// │   - Message framing                 │
// └─────────────────────────────────────┘
//              ↕
// ┌─────────────────────────────────────┐
// │   TCP/IP Network Layer              │
// └─────────────────────────────────────┘
```

### **WebSocket Configuration in GraphQL**

```typescript
// GraphQL uses WebSocket under the hood
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      subscriptions: {
        'graphql-ws': true,  // Uses WebSocket protocol
        // But adds GraphQL features on top
      },
    }),
  ],
})
export class AppModule {}
```

### **Feature Comparison Example**

```typescript
// ❌ Raw WebSocket - Manual everything
@WebSocketGateway()
export class ChatGateway {
  @SubscribeMessage('sendMessage')
  handleMessage(client: Socket, payload: any) {
    // 1. No type safety
    // 2. Manual validation
    if (!payload.text || typeof payload.text !== 'string') {
      return { error: 'Invalid message' };
    }
    
    // 3. Manual authentication
    const user = this.verifyToken(payload.token);
    if (!user) {
      return { error: 'Unauthorized' };
    }
    
    // 4. Manual filtering
    this.server.to(payload.roomId).emit('message', {
      text: payload.text,
      author: user.name,
    });
  }
}

// ✅ GraphQL Subscription - Automatic everything
@Resolver(() => Message)
export class MessageResolver {
  @Mutation(() => Message)
  @UseGuards(GqlAuthGuard)  // 3. Authentication integrated
  async sendMessage(
    @Args('input') input: SendMessageInput,  // 2. Auto-validated
    @CurrentUser() user: User,
  ): Promise<Message> {
    const message = await this.messageService.create({
      text: input.text,
      roomId: input.roomId,
      authorId: user.id,
    });

    await this.pubSub.publish('messageAdded', { messageAdded: message });
    return message;
  }

  @Subscription(() => Message, {  // 1. Type-safe
    filter: (payload, variables) => {  // 4. Built-in filtering
      return payload.messageAdded.roomId === variables.roomId;
    },
  })
  messageAdded(@Args('roomId') roomId: string) {
    return this.pubSub.asyncIterator('messageAdded');
  }
}
```

### **When to Use Each**

```typescript
// Use GraphQL Subscriptions when:
// ✅ Need type safety and schema validation
// ✅ Want GraphQL query syntax for real-time data
// ✅ Already using GraphQL for queries/mutations
// ✅ Need filtering and authentication out of the box
// ✅ Complex data with nested relationships

// Use Raw WebSockets when:
// ✅ Need ultra-low latency (gaming, trading)
// ✅ Binary data (video streaming, file transfer)
// ✅ Custom protocol requirements
// ✅ Not using GraphQL
// ✅ Simple message passing
```

### **Under the Hood**

```typescript
// GraphQL subscription over WebSocket protocol:

// 1. Client connects (WebSocket handshake)
const wsUrl = 'ws://localhost:3000/graphql';

// 2. Client sends GraphQL subscription (over WebSocket)
{
  type: 'subscribe',
  id: '1',
  payload: {
    query: `
      subscription {
        messageAdded(roomId: "room1") {
          id
          text
        }
      }
    `
  }
}

// 3. Server validates against schema (GraphQL layer)
// 4. Server sends data (over WebSocket)
{
  type: 'next',
  id: '1',
  payload: {
    data: {
      messageAdded: {
        id: '123',
        text: 'Hello'
      }
    }
  }
}
```

**Interview Tip**: **WebSockets** = **transport protocol** (TCP connection), **GraphQL Subscriptions** = **application feature** using WebSocket. WebSocket is **how** (low-level socket communication), Subscription is **what** (high-level typed operations). **Key differences**: WebSocket has no schema/validation (send any data), Subscriptions have schema/type safety/validation. **Subscriptions built on WebSocket** but add: GraphQL syntax, type checking, filtering, authentication integration. **Analogy**: WebSocket is like TCP, GraphQL Subscription is like HTTP - one is transport, other is application protocol. **Use Subscriptions** when using GraphQL (inherit all benefits), use raw WebSocket for custom protocols or non-GraphQL apps.

</details>

---

## Relations & Data Loading

34. How do you define relationships between types?

<details>
<summary><strong>Answer</strong></summary>

Define relationships using **`@Field(() => RelatedType)`** in models and **`@ResolveField()`** in resolvers. Mark relationship fields with type function pointing to related type. Implement field resolver to fetch related data. Supports **one-to-one**, **one-to-many**, **many-to-one**, **many-to-many**. Resolver method receives parent object via `@Parent()` decorator.

### **One-to-Many Relationship**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => [Post])  // One user has many posts
  posts: Post[];
}

// post.model.ts
@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field(() => User)  // Many posts belong to one user
  author: User;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Resolve posts for each user
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
  }
}

// post.resolver.ts
@Resolver(() => Post)
export class PostResolver {
  @ResolveField(() => User)
  async author(@Parent() post: Post): Promise<User> {
    return this.userService.findById(post.authorId);
  }
}
```

### **One-to-One Relationship**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => Profile, { nullable: true })  // One user has one profile
  profile?: Profile;
}

// profile.model.ts
@ObjectType()
export class Profile {
  @Field(() => ID)
  id: string;

  @Field()
  bio: string;

  @Field(() => User)  // One profile belongs to one user
  user: User;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  @ResolveField(() => Profile, { nullable: true })
  async profile(@Parent() user: User): Promise<Profile | null> {
    return this.profileService.findByUserId(user.id);
  }
}
```

### **Many-to-Many Relationship**

```typescript
// student.model.ts
@ObjectType()
export class Student {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => [Course])  // Many students, many courses
  courses: Course[];
}

// course.model.ts
@ObjectType()
export class Course {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field(() => [Student])  // Many courses, many students
  students: Student[];
}

// student.resolver.ts
@Resolver(() => Student)
export class StudentResolver {
  @ResolveField(() => [Course])
  async courses(@Parent() student: Student): Promise<Course[]> {
    return this.courseService.findByStudentId(student.id);
  }
}

// course.resolver.ts
@Resolver(() => Course)
export class CourseResolver {
  @ResolveField(() => [Student])
  async students(@Parent() course: Course): Promise<Student[]> {
    return this.studentService.findByCourseId(course.id);
  }
}
```

### **With TypeORM Entities**

```typescript
// user.entity.ts
@Entity()
@ObjectType()
export class User {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)
  id: string;

  @Column()
  @Field()
  name: string;

  @OneToMany(() => Post, post => post.author)  // TypeORM relation
  @Field(() => [Post])  // GraphQL field
  posts: Post[];
}

// post.entity.ts
@Entity()
@ObjectType()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  @Field(() => ID)
  id: string;

  @Column()
  @Field()
  title: string;

  @ManyToOne(() => User, user => user.posts)  // TypeORM relation
  @Field(() => User)  // GraphQL field
  author: User;

  @Column()
  authorId: string;  // Foreign key (not exposed in GraphQL)
}
```

### **Lazy Loading with Field Resolvers**

```typescript
// Only fetch related data when requested in query

@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    // Only fetch user, not posts
    return this.userService.findById(id);
  }

  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    // Only called if client requests 'posts' field
    return this.postService.findByUserId(user.id);
  }
}

// Client query without posts:
// query {
//   user(id: "1") {
//     id
//     name  // posts NOT fetched
//   }
// }

// Client query with posts:
// query {
//   user(id: "1") {
//     id
//     name
//     posts {  // Now posts are fetched
//       id
//       title
//     }
//   }
// }
```

### **Complete Example**

```typescript
// Models
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => [Post])
  posts: Post[];

  @Field(() => Profile, { nullable: true })
  profile?: Profile;
}

@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field(() => User)
  author: User;

  @Field(() => [Comment])
  comments: Comment[];
}

@ObjectType()
export class Comment {
  @Field(() => ID)
  id: string;

  @Field()
  text: string;

  @Field(() => User)
  author: User;

  @Field(() => Post)
  post: Post;
}

// Resolvers
@Resolver(() => User)
export class UserResolver {
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    return this.postService.findByUserId(user.id);
  }

  @ResolveField(() => Profile, { nullable: true })
  async profile(@Parent() user: User): Promise<Profile | null> {
    return this.profileService.findByUserId(user.id);
  }
}

@Resolver(() => Post)
export class PostResolver {
  @ResolveField(() => User)
  async author(@Parent() post: Post): Promise<User> {
    return this.userService.findById(post.authorId);
  }

  @ResolveField(() => [Comment])
  async comments(@Parent() post: Post): Promise<Comment[]> {
    return this.commentService.findByPostId(post.id);
  }
}
```

**Interview Tip**: Define relationships with **`@Field(() => RelatedType)`** in models + **`@ResolveField()`** in resolvers. **Pattern**: Model declares field type, resolver fetches data. **Types**: one-to-many (`@Field(() => [Post])`), one-to-one (`@Field(() => Profile)`), many-to-many (both sides have arrays). **Field resolver** receives parent via `@Parent()`, returns related data. **Lazy loading**: relations only fetched when client requests them in query (efficient). **With TypeORM**: use both `@OneToMany/@ManyToOne` (TypeORM) and `@Field()` (GraphQL) decorators on same property. **Example**: User has posts → `@ResolveField(() => [Post]) posts(@Parent() user)` fetches user's posts.

</details>
35. What is the N+1 problem in GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**N+1 problem** occurs when fetching a list of items requires **1 query for the list + N queries for related data** (one per item). In GraphQL with field resolvers, querying users with posts executes **1 query for users + 1 query per user for their posts**, resulting in **N+1 database calls**. This causes **severe performance issues** with large datasets.

### **N+1 Problem Example**

```typescript
// user.resolver.ts - PROBLEMATIC CODE
@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    // 1 query: SELECT * FROM users
    return this.userService.findAll();
  }

  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    // Called N times (once per user)
    // SELECT * FROM posts WHERE userId = ?
    return this.postService.findByUserId(user.id);
  }
}

// Client query:
// query {
//   users {  // 1 query: fetch all users
//     id
//     name
//     posts {  // N queries: fetch posts for EACH user
//       id
//       title
//     }
//   }
// }

// Execution with 100 users:
// Query 1: SELECT * FROM users (returns 100 users)
// Query 2: SELECT * FROM posts WHERE userId = 1
// Query 3: SELECT * FROM posts WHERE userId = 2
// Query 4: SELECT * FROM posts WHERE userId = 3
// ...
// Query 101: SELECT * FROM posts WHERE userId = 100
//
// TOTAL: 101 queries! (1 + 100)
```

### **Real-World Impact**

```typescript
// Scenario: E-commerce product listing with reviews

@Resolver(() => Product)
export class ProductResolver {
  @Query(() => [Product])
  async products(): Promise<Product[]> {
    // 1 query: Get 50 products
    return this.productService.findAll();
  }

  @ResolveField(() => [Review])
  async reviews(@Parent() product: Product): Promise<Review[]> {
    // 50 queries: One per product
    return this.reviewService.findByProductId(product.id);
  }

  @ResolveField(() => Category)
  async category(@Parent() product: Product): Promise<Category> {
    // 50 queries: One per product
    return this.categoryService.findById(product.categoryId);
  }

  @ResolveField(() => Seller)
  async seller(@Parent() product: Product): Promise<Seller> {
    // 50 queries: One per product
    return this.sellerService.findById(product.sellerId);
  }
}

// Query requesting products with reviews, category, seller:
// - 1 query for products
// - 50 queries for reviews
// - 50 queries for categories
// - 50 queries for sellers
// TOTAL: 151 queries for 50 products!
//
// Performance:
// - Each query: ~10ms
// - Total time: 151 × 10ms = 1.51 seconds
// - Database connections exhausted
// - High CPU usage
```

### **Detecting N+1 Problems**

```typescript
// Method 1: TypeORM Query Logger
// ormconfig.ts
export default {
  type: 'postgres',
  logging: ['query'],  // Log all queries
  logger: 'advanced-console',
};

// Console output will show:
// query: SELECT * FROM users
// query: SELECT * FROM posts WHERE userId = $1 -- [1]
// query: SELECT * FROM posts WHERE userId = $1 -- [2]
// query: SELECT * FROM posts WHERE userId = $1 -- [3]
// ... (N+1 pattern detected!)

// Method 2: Custom Logging Interceptor
@Injectable()
export class QueryCountInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const startTime = Date.now();
    let queryCount = 0;

    // Hook into TypeORM query logging
    const originalQuery = this.dataSource.query;
    this.dataSource.query = (...args) => {
      queryCount++;
      return originalQuery.apply(this.dataSource, args);
    };

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - startTime;
        console.warn(`⚠️  ${queryCount} queries in ${duration}ms`);
        
        if (queryCount > 10) {
          console.error('🔥 Potential N+1 problem detected!');
        }
      }),
    );
  }
}

// Method 3: Apollo Server Plugins
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      plugins: [
        {
          async requestDidStart() {
            const queryCount = { count: 0 };
            return {
              async willSendResponse({ response }) {
                // Log query count in response extensions
                response.extensions = {
                  ...response.extensions,
                  queryCount: queryCount.count,
                };
              },
            };
          },
        },
      ],
    }),
  ],
})
export class AppModule {}
```

### **Common N+1 Scenarios**

```typescript
// Scenario 1: Nested relationships (worst case)
query {
  users {              // 1 query
    posts {            // N queries (per user)
      comments {       // N*M queries (per post)
        author {       // N*M*K queries (per comment)
          profile {    // N*M*K*L queries
            avatar
          }
        }
      }
    }
  }
}
// With 10 users, 5 posts each, 3 comments each:
// 1 + 10 + 50 + 150 + 150 = 361 queries!

// Scenario 2: Multiple relationships at same level
query {
  posts {           // 1 query
    author { ... }  // N queries
    category { ... }// N queries
    tags { ... }    // N queries
    comments { ... }// N queries
  }
}
// With 50 posts: 1 + 50 + 50 + 50 + 50 = 201 queries

// Scenario 3: Conditional field resolvers
@ResolveField(() => User)
async author(@Parent() post: Post): Promise<User> {
  // Called even if author data already loaded
  // No caching = repeated queries for same user
  return this.userService.findById(post.authorId);
}
```

### **Solutions Overview**

```typescript
// Solution 1: DataLoader (recommended)
// - Batches multiple requests into single query
// - Caches results within single request
// - See next question for details

// Solution 2: Eager loading with TypeORM
@Query(() => [User])
async users(): Promise<User[]> {
  return this.userRepository.find({
    relations: ['posts', 'profile'],  // Load relations upfront
  });
}
// Problem: Loads ALL relations even if not requested
// GraphQL advantage lost (over-fetching)

// Solution 3: Join queries
@Query(() => [User])
async users(@Info() info: GraphQLResolveInfo): Promise<User[]> {
  // Parse GraphQL query to determine requested fields
  const requestedRelations = this.parseRequestedFields(info);
  return this.userRepository.find({
    relations: requestedRelations,  // Only load requested relations
  });
}
// Better, but complex and doesn't work with nested resolvers

// Solution 4: Manual batching in resolver
private pendingUserIds: Set<string> = new Set();
private userBatchTimeout: NodeJS.Timeout;

@ResolveField(() => User)
async author(@Parent() post: Post): Promise<User> {
  this.pendingUserIds.add(post.authorId);
  
  // Batch requests using debounce
  return new Promise((resolve) => {
    clearTimeout(this.userBatchTimeout);
    this.userBatchTimeout = setTimeout(async () => {
      const users = await this.userService.findByIds(
        Array.from(this.pendingUserIds)
      );
      this.pendingUserIds.clear();
      resolve(users.find(u => u.id === post.authorId));
    }, 10);
  });
}
// Problem: Complex, error-prone, DataLoader is better
```

### **Performance Comparison**

```typescript
// Benchmark: 100 users, each with 5 posts

// Without optimization (N+1):
// - Queries: 101 (1 for users + 100 for posts)
// - Time: ~1010ms (10ms per query)
// - DB connections: High

// With DataLoader:
// - Queries: 2 (1 for users + 1 batched for all posts)
// - Time: ~20ms
// - Improvement: 50x faster

// With eager loading:
// - Queries: 1 (with LEFT JOIN)
// - Time: ~15ms
// - Problem: Over-fetching (loads posts even if not requested)
```

**Interview Tip**: **N+1 problem** occurs when field resolvers execute **1 query for list + N queries for each item's relations** (e.g., 100 users → 101 queries). **Impact**: severe performance degradation, database overload, slow responses. **Example**: `users { posts { ... } }` → 1 query for users, then separate query for each user's posts. **Detection**: enable query logging, count queries per request, watch for repeated similar queries. **Solutions**: **DataLoader** (batching + caching, recommended), eager loading (loses GraphQL benefits), or manual batching (complex). **Key insight**: GraphQL's flexibility creates N+1 naturally with naive field resolvers—always use DataLoader for production resolvers that fetch related data.

</details>

36. What is DataLoader and how does it solve N+1 problem?

<details>
<summary><strong>Answer</strong></summary>

**DataLoader** is a **batching and caching utility** from Facebook that solves the N+1 problem by **collecting multiple data requests** and **executing them as a single batch query**. It uses **request coalescing** (collects requests in event loop tick) and **per-request caching** (deduplicates identical requests). Install with `npm i dataloader`, create loaders with batch functions, inject via context.

### **Basic DataLoader Setup**

```typescript
// Installation
// npm install dataloader
// npm install --save-dev @types/dataloader

// post.dataloader.ts
import DataLoader from 'dataloader';
import { Injectable } from '@nestjs/common';
import { PostService } from './post.service';
import { Post } from './post.model';

@Injectable()
export class PostDataLoader {
  constructor(private postService: PostService) {}

  // Create DataLoader instance
  createLoader(): DataLoader<string, Post[]> {
    return new DataLoader<string, Post[]>(
      async (userIds: readonly string[]) => {
        // Batch function: receives array of keys (user IDs)
        // Must return array in SAME ORDER as keys
        console.log('Batching posts for users:', userIds);
        
        // Single query for all users' posts
        const posts = await this.postService.findByUserIds(
          userIds as string[]
        );
        
        // Group posts by userId
        const postsByUserId = new Map<string, Post[]>();
        posts.forEach(post => {
          const userPosts = postsByUserId.get(post.authorId) || [];
          userPosts.push(post);
          postsByUserId.set(post.authorId, userPosts);
        });
        
        // Return in same order as input userIds
        return userIds.map(userId => postsByUserId.get(userId) || []);
      },
      {
        cache: true,  // Enable per-request caching (default)
      }
    );
  }
}

// post.service.ts
@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
  ) {}

  // Batch query: fetch posts for multiple users at once
  async findByUserIds(userIds: string[]): Promise<Post[]> {
    return this.postRepository
      .createQueryBuilder('post')
      .where('post.authorId IN (:...userIds)', { userIds })
      .getMany();
  }
}
```

### **Inject DataLoader via Context**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { PostDataLoader } from './post/post.dataloader';
import { UserDataLoader } from './user/user.dataloader';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (
        postDataLoader: PostDataLoader,
        userDataLoader: UserDataLoader,
      ) => ({
        autoSchemaFile: true,
        context: ({ req }) => {
          // Create new DataLoader instances per request
          // This ensures caching is per-request, not global
          return {
            req,
            loaders: {
              posts: postDataLoader.createLoader(),
              users: userDataLoader.createLoader(),
            },
          };
        },
      }),
      inject: [PostDataLoader, UserDataLoader],
    }),
  ],
  providers: [PostDataLoader, UserDataLoader],
})
export class AppModule {}
```

### **Use DataLoader in Resolver**

```typescript
// user.resolver.ts
import { Resolver, Query, ResolveField, Parent, Context } from '@nestjs/graphql';
import { User } from './user.model';
import { Post } from '../post/post.model';
import { UserService } from './user.service';

// Context type with loaders
interface GraphQLContext {
  loaders: {
    posts: DataLoader<string, Post[]>;
    users: DataLoader<string, User>;
  };
}

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    // 1 query: fetch all users
    return this.userService.findAll();
  }

  @ResolveField(() => [Post])
  async posts(
    @Parent() user: User,
    @Context() context: GraphQLContext,
  ): Promise<Post[]> {
    // Use DataLoader instead of direct service call
    // DataLoader batches and caches automatically
    return context.loaders.posts.load(user.id);
    
    // Behind the scenes:
    // 1st call: loader.load('user1') - queued
    // 2nd call: loader.load('user2') - queued
    // 3rd call: loader.load('user3') - queued
    // ... (all queued in same event loop tick)
    // Then: batch function called once with ['user1', 'user2', 'user3']
    // Result: Single query for all users' posts!
  }
}

// Query execution:
// query {
//   users {  // Query 1: SELECT * FROM users
//     id
//     name
//     posts {  // Query 2: SELECT * FROM posts WHERE authorId IN (1,2,3,...,100)
//       id     // ONE batched query instead of 100!
//       title
//     }
//   }
// }
//
// Result: 2 queries instead of 101 (50x improvement!)
```

### **DataLoader with Caching**

```typescript
// post.resolver.ts
@Resolver(() => Post)
export class PostResolver {
  @ResolveField(() => User)
  async author(
    @Parent() post: Post,
    @Context() context: GraphQLContext,
  ): Promise<User> {
    // DataLoader automatically caches within request
    return context.loaders.users.load(post.authorId);
  }
}

// Query with duplicate authors:
query {
  posts {  // Returns 10 posts
    id
    title
    author {  // Same author appears in multiple posts
      id
      name
    }
  }
}

// Execution:
// - 10 posts, but only 3 unique authors
// - loader.load() called 10 times
// - DataLoader deduplicates: only passes [author1, author2, author3] to batch function
// - Batch function executes ONCE: SELECT * FROM users WHERE id IN (1,2,3)
// - Cache hits for duplicate authors (7 cache hits, 0 additional queries)
//
// Result: 1 query for 10 posts instead of 10 queries!
```

### **Complete Production Example**

```typescript
// dataloader.factory.ts - Centralized DataLoader factory
import { Injectable } from '@nestjs/common';
import DataLoader from 'dataloader';
import { PostService } from './post/post.service';
import { UserService } from './user/user.service';
import { CommentService } from './comment/comment.service';
import { Post } from './post/post.model';
import { User } from './user/user.model';
import { Comment } from './comment/comment.model';

export interface IDataLoaders {
  postsByUserId: DataLoader<string, Post[]>;
  userById: DataLoader<string, User>;
  commentsByPostId: DataLoader<string, Comment[]>;
  postsByIds: DataLoader<string, Post>;
}

@Injectable()
export class DataLoaderFactory {
  constructor(
    private postService: PostService,
    private userService: UserService,
    private commentService: CommentService,
  ) {}

  createLoaders(): IDataLoaders {
    return {
      // Posts by user ID (one-to-many)
      postsByUserId: new DataLoader<string, Post[]>(
        async (userIds: readonly string[]) => {
          const posts = await this.postService.findByUserIds(userIds as string[]);
          const postsByUserId = this.groupBy(posts, 'authorId');
          return userIds.map(id => postsByUserId.get(id) || []);
        },
        { cache: true },
      ),

      // User by ID (one-to-one)
      userById: new DataLoader<string, User>(
        async (userIds: readonly string[]) => {
          const users = await this.userService.findByIds(userIds as string[]);
          const userMap = new Map(users.map(u => [u.id, u]));
          return userIds.map(id => userMap.get(id));
        },
        { cache: true },
      ),

      // Comments by post ID (one-to-many)
      commentsByPostId: new DataLoader<string, Comment[]>(
        async (postIds: readonly string[]) => {
          const comments = await this.commentService.findByPostIds(postIds as string[]);
          const commentsByPostId = this.groupBy(comments, 'postId');
          return postIds.map(id => commentsByPostId.get(id) || []);
        },
        { cache: true },
      ),

      // Posts by IDs (batching individual lookups)
      postsByIds: new DataLoader<string, Post>(
        async (postIds: readonly string[]) => {
          const posts = await this.postService.findByIds(postIds as string[]);
          const postMap = new Map(posts.map(p => [p.id, p]));
          return postIds.map(id => postMap.get(id));
        },
        { cache: true },
      ),
    };
  }

  // Helper: Group array of objects by key
  private groupBy<T>(array: T[], key: keyof T): Map<any, T[]> {
    const map = new Map<any, T[]>();
    array.forEach(item => {
      const keyValue = item[key];
      const group = map.get(keyValue) || [];
      group.push(item);
      map.set(keyValue, group);
    });
    return map;
  }
}

// app.module.ts
@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (dataLoaderFactory: DataLoaderFactory) => ({
        autoSchemaFile: true,
        context: () => ({
          loaders: dataLoaderFactory.createLoaders(),  // New loaders per request
        }),
      }),
      inject: [DataLoaderFactory],
    }),
  ],
  providers: [DataLoaderFactory],
})
export class AppModule {}

// Usage in resolvers
@Resolver(() => User)
export class UserResolver {
  @ResolveField(() => [Post])
  async posts(
    @Parent() user: User,
    @Context() { loaders }: { loaders: IDataLoaders },
  ): Promise<Post[]> {
    return loaders.postsByUserId.load(user.id);
  }
}

@Resolver(() => Post)
export class PostResolver {
  @ResolveField(() => User)
  async author(
    @Parent() post: Post,
    @Context() { loaders }: { loaders: IDataLoaders },
  ): Promise<User> {
    return loaders.userById.load(post.authorId);
  }

  @ResolveField(() => [Comment])
  async comments(
    @Parent() post: Post,
    @Context() { loaders }: { loaders: IDataLoaders },
  ): Promise<Comment[]> {
    return loaders.commentsByPostId.load(post.id);
  }
}
```

### **Advanced: DataLoader Options**

```typescript
// Custom caching strategy
const userLoader = new DataLoader<string, User>(
  async (ids) => {
    const users = await userService.findByIds(ids);
    const userMap = new Map(users.map(u => [u.id, u]));
    return ids.map(id => userMap.get(id));
  },
  {
    // Enable/disable caching (default: true)
    cache: true,
    
    // Custom cache implementation (e.g., Redis)
    cacheMap: new Map(),  // Can use custom Map implementation
    
    // Custom cache key function
    cacheKeyFn: (key: string) => `user:${key}`,
    
    // Batch scheduler (default: process.nextTick)
    batchScheduleFn: (callback) => setTimeout(callback, 10),
    
    // Max batch size
    maxBatchSize: 100,  // Split large batches
  }
);

// Manual cache manipulation
userLoader.clear('user-1');  // Clear single key
userLoader.clearAll();       // Clear all cache
userLoader.prime('user-1', user);  // Pre-populate cache
```

### **Error Handling with DataLoader**

```typescript
// Handle errors in batch function
const userLoader = new DataLoader<string, User>(async (ids) => {
  try {
    const users = await userService.findByIds(ids);
    const userMap = new Map(users.map(u => [u.id, u]));
    
    return ids.map(id => {
      const user = userMap.get(id);
      if (!user) {
        // Return Error for missing users
        return new Error(`User ${id} not found`);
      }
      return user;
    });
  } catch (error) {
    // Return same error for all keys
    return ids.map(() => error);
  }
});

// In resolver
@ResolveField(() => User, { nullable: true })
async author(
  @Parent() post: Post,
  @Context() { loaders }: { loaders: IDataLoaders },
): Promise<User | null> {
  try {
    return await loaders.userById.load(post.authorId);
  } catch (error) {
    console.error('Failed to load author:', error);
    return null;  // Handle gracefully
  }
}
```

### **Performance Comparison**

```typescript
// Benchmark: Query 100 users with their posts

// WITHOUT DataLoader (N+1):
// - Query 1: SELECT * FROM users (100 users)
// - Query 2-101: SELECT * FROM posts WHERE authorId = ? (×100)
// - Total: 101 queries
// - Time: ~1010ms (10ms per query)
// - DB connections: 101

// WITH DataLoader:
// - Query 1: SELECT * FROM users (100 users)
// - Query 2: SELECT * FROM posts WHERE authorId IN (1,2,3,...,100)
// - Total: 2 queries
// - Time: ~20ms
// - DB connections: 2
// - Improvement: 50x faster, 50x fewer connections

// Nested query with caching:
query {
  posts {  // 100 posts
    author {  // Only 10 unique authors
      id
      name
    }
  }
}
// Without DataLoader: 101 queries (1 + 100)
// With DataLoader: 2 queries (1 for posts + 1 batched for unique authors)
// Cache hits: 90 (10 authors loaded, 90 cache hits)
```

**Interview Tip**: **DataLoader** solves N+1 by **batching multiple requests into single query** + **per-request caching**. **How**: collects `.load()` calls in event loop tick, executes batch function once with all keys. **Setup**: create loader with batch function (receives keys array, returns results in same order), inject via GraphQL context (new instance per request). **Key**: batch function MUST return array in SAME ORDER as input keys. **Caching**: deduplicates identical requests within single request (e.g., same author in multiple posts → 1 query + cache hits). **Usage**: `context.loaders.users.load(id)` in resolver instead of direct service call. **Result**: 100 users with posts → 2 queries (users + batched posts) instead of 101. Always use DataLoader for production GraphQL resolvers that fetch related data.

</details>

37. How do you implement `@ResolveField()` for field resolvers?

<details>
<summary><strong>Answer</strong></summary>

**`@ResolveField()`** decorator defines **field resolvers** that compute or fetch field values **on-demand** (lazy loading). Applied to methods in `@Resolver()` class, receives **parent object** via `@Parent()` decorator, and can access **args**, **context**, **info**. Used for **computed fields**, **relationships**, **derived data**, and **custom logic**. Executes **only when field is requested** in query.

### **Basic Field Resolver**

```typescript
// user.model.ts
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  firstName: string;

  @Field()
  lastName: string;

  // Field declared but value computed by resolver
  @Field()
  fullName: string;

  @Field()
  email: string;
}

// user.resolver.ts
import { Resolver, Query, ResolveField, Parent } from '@nestjs/graphql';
import { User } from './user.model';
import { UserService } from './user.service';

@Resolver(() => User)  // Specifies which type this resolves
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    return this.userService.findById(id);
  }

  // Field resolver for computed field
  @ResolveField(() => String)  // Return type
  fullName(@Parent() user: User): string {
    // @Parent() provides the parent User object
    return `${user.firstName} ${user.lastName}`;
  }
}

// Usage:
// query {
//   user(id: "1") {
//     id
//     firstName
//     lastName
//     fullName  // Computed by field resolver
//   }
// }
```

### **Field Resolver with Arguments**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => [Post])
  posts: Post[];
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  // Field resolver with arguments
  @ResolveField(() => [Post])
  async posts(
    @Parent() user: User,
    @Args('limit', { type: () => Int, nullable: true, defaultValue: 10 }) limit: number,
    @Args('offset', { type: () => Int, nullable: true, defaultValue: 0 }) offset: number,
    @Args('status', { type: () => String, nullable: true }) status?: string,
  ): Promise<Post[]> {
    // Fetch posts with pagination and filtering
    return this.postService.findByUserId(user.id, {
      limit,
      offset,
      status,
    });
  }
}

// Usage with arguments:
// query {
//   user(id: "1") {
//     name
//     posts(limit: 5, status: "published") {
//       id
//       title
//     }
//   }
// }
```

### **Field Resolver with Context**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  email: string;

  // Only visible to user themselves or admins
  @Field({ nullable: true })
  phoneNumber?: string;

  @Field(() => Boolean)
  isFollowing: boolean;
}

// user.resolver.ts
interface GraphQLContext {
  req: Request;
  user: User;  // Current authenticated user
  loaders: any;
}

@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private followService: FollowService,
  ) {}

  // Access control in field resolver
  @ResolveField(() => String, { nullable: true })
  phoneNumber(
    @Parent() user: User,
    @Context() context: GraphQLContext,
  ): string | null {
    // Only return phone number if user is viewing their own profile
    // or if current user is admin
    const currentUser = context.user;
    if (!currentUser) return null;
    
    if (currentUser.id === user.id || currentUser.role === 'admin') {
      return user.phoneNumber;
    }
    
    return null;  // Hidden from other users
  }

  // Computed field based on current user
  @ResolveField(() => Boolean)
  async isFollowing(
    @Parent() user: User,
    @Context() context: GraphQLContext,
  ): Promise<boolean> {
    const currentUser = context.user;
    if (!currentUser) return false;
    
    // Check if current user follows this user
    return this.followService.isFollowing(currentUser.id, user.id);
  }
}

// Query result depends on who is authenticated:
// query {
//   user(id: "123") {
//     name
//     phoneNumber  // Only if viewing own profile or admin
//     isFollowing  // Only if authenticated
//   }
// }
```

### **Field Resolver with DataLoader**

```typescript
// post.model.ts
@ObjectType()
export class Post {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field(() => User)
  author: User;

  @Field(() => [Comment])
  comments: Comment[];

  @Field(() => Int)
  commentsCount: number;

  @Field(() => Int)
  likesCount: number;
}

// post.resolver.ts
@Resolver(() => Post)
export class PostResolver {
  constructor(
    private postService: PostService,
    private commentService: CommentService,
    private likeService: LikeService,
  ) {}

  // Resolve relationship with DataLoader (batching)
  @ResolveField(() => User)
  async author(
    @Parent() post: Post,
    @Context() { loaders }: GraphQLContext,
  ): Promise<User> {
    // Use DataLoader to batch and cache user lookups
    return loaders.userById.load(post.authorId);
  }

  // Resolve one-to-many relationship
  @ResolveField(() => [Comment])
  async comments(
    @Parent() post: Post,
    @Context() { loaders }: GraphQLContext,
  ): Promise<Comment[]> {
    return loaders.commentsByPostId.load(post.id);
  }

  // Computed aggregation field
  @ResolveField(() => Int)
  async commentsCount(
    @Parent() post: Post,
    @Context() { loaders }: GraphQLContext,
  ): Promise<number> {
    const comments = await loaders.commentsByPostId.load(post.id);
    return comments.length;
  }

  @ResolveField(() => Int)
  async likesCount(
    @Parent() post: Post,
    @Context() { loaders }: GraphQLContext,
  ): Promise<number> {
    // Use specialized count loader for efficiency
    return loaders.likesCountByPostId.load(post.id);
  }
}
```

### **Field Resolver with GraphQL Info**

```typescript
// Advanced: Optimize based on requested fields
import { GraphQLResolveInfo } from 'graphql';
import { Info } from '@nestjs/graphql';
import graphqlFields from 'graphql-fields';

@Resolver(() => User)
export class UserResolver {
  @ResolveField(() => [Post])
  async posts(
    @Parent() user: User,
    @Info() info: GraphQLResolveInfo,
  ): Promise<Post[]> {
    // Parse which fields are requested
    const fields = graphqlFields(info);
    
    // Optimize query based on requested fields
    const relations = [];
    if (fields.author) relations.push('author');
    if (fields.comments) relations.push('comments');
    
    return this.postService.findByUserId(user.id, { relations });
  }
}
```

### **Complete Production Example**

```typescript
// user.model.ts
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  firstName: string;

  @Field()
  lastName: string;

  @Field()
  email: string;

  @Field()  // Computed
  fullName: string;

  @Field(() => [Post])  // Relationship
  posts: Post[];

  @Field(() => Profile, { nullable: true })  // One-to-one
  profile?: Profile;

  @Field(() => Int)  // Aggregation
  postsCount: number;

  @Field(() => Boolean)  // Context-dependent
  isFollowing: boolean;

  @Field({ nullable: true })  // Access-controlled
  phoneNumber?: string;
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
    private profileService: ProfileService,
    private followService: FollowService,
  ) {}

  @Query(() => [User])
  async users(): Promise<User[]> {
    // Just fetch users, field resolvers handle rest
    return this.userService.findAll();
  }

  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // 1. Computed field (synchronous)
  @ResolveField(() => String)
  fullName(@Parent() user: User): string {
    return `${user.firstName} ${user.lastName}`;
  }

  // 2. Relationship field with DataLoader (async, batched)
  @ResolveField(() => [Post])
  async posts(
    @Parent() user: User,
    @Context() { loaders }: GraphQLContext,
    @Args('limit', { type: () => Int, nullable: true }) limit?: number,
  ): Promise<Post[]> {
    const posts = await loaders.postsByUserId.load(user.id);
    return limit ? posts.slice(0, limit) : posts;
  }

  // 3. One-to-one relationship
  @ResolveField(() => Profile, { nullable: true })
  async profile(
    @Parent() user: User,
    @Context() { loaders }: GraphQLContext,
  ): Promise<Profile | null> {
    return loaders.profileByUserId.load(user.id);
  }

  // 4. Aggregation field (computed from relationship)
  @ResolveField(() => Int)
  async postsCount(
    @Parent() user: User,
    @Context() { loaders }: GraphQLContext,
  ): Promise<number> {
    // Option 1: Load posts and count (if posts might be requested)
    const posts = await loaders.postsByUserId.load(user.id);
    return posts.length;
    
    // Option 2: Use separate count loader (more efficient if only count needed)
    // return loaders.postsCountByUserId.load(user.id);
  }

  // 5. Context-dependent field
  @ResolveField(() => Boolean)
  async isFollowing(
    @Parent() user: User,
    @Context() { user: currentUser }: GraphQLContext,
  ): Promise<boolean> {
    if (!currentUser) return false;
    return this.followService.isFollowing(currentUser.id, user.id);
  }

  // 6. Access-controlled field
  @ResolveField(() => String, { nullable: true })
  phoneNumber(
    @Parent() user: User,
    @Context() { user: currentUser }: GraphQLContext,
  ): string | null {
    // Only visible to owner or admin
    if (!currentUser) return null;
    if (currentUser.id === user.id || currentUser.role === 'admin') {
      return user.phoneNumber;
    }
    return null;
  }
}

// Usage examples:
// 1. Request only basic fields (no field resolvers executed)
query {
  user(id: "1") {
    firstName
    lastName
    email
  }
}

// 2. Request computed field (fullName resolver executes)
query {
  user(id: "1") {
    fullName  // firstName + lastName
  }
}

// 3. Request relationship (posts resolver executes with DataLoader)
query {
  users {
    id
    fullName
    posts(limit: 5) {  // With arguments
      title
    }
  }
}

// 4. Request multiple fields (respective resolvers execute)
query {
  user(id: "1") {
    fullName        // Computed
    posts { ... }   // Relationship
    profile { ... } // One-to-one
    postsCount      // Aggregation
    isFollowing     // Context-dependent
    phoneNumber     // Access-controlled
  }
}
```

### **Field Resolver Best Practices**

```typescript
// 1. Use nullable for optional fields
@ResolveField(() => Profile, { nullable: true })
async profile(@Parent() user: User): Promise<Profile | null> {
  return this.profileService.findByUserId(user.id);
}

// 2. Handle errors gracefully
@ResolveField(() => User, { nullable: true })
async author(@Parent() post: Post): Promise<User | null> {
  try {
    return await this.userService.findById(post.authorId);
  } catch (error) {
    console.error(`Failed to load author ${post.authorId}:`, error);
    return null;  // Return null instead of throwing
  }
}

// 3. Use DataLoader for all database lookups
@ResolveField(() => User)
async author(
  @Parent() post: Post,
  @Context() { loaders }: GraphQLContext,
): Promise<User> {
  return loaders.userById.load(post.authorId);  // Batched + cached
}

// 4. Type arguments properly
@ResolveField(() => [Post])
async posts(
  @Parent() user: User,
  @Args('filter', { type: () => PostFilterInput, nullable: true })
  filter?: PostFilterInput,
): Promise<Post[]> {
  return this.postService.findByUserId(user.id, filter);
}

// 5. Document complex field resolvers
/**
 * Resolves user's posts with optional filtering and pagination.
 * Uses DataLoader for batching and caching.
 * 
 * @param user - Parent user object
 * @param args - Filter and pagination arguments
 * @param context - GraphQL context with loaders
 * @returns Array of posts matching criteria
 */
@ResolveField(() => [Post], {
  description: 'User posts with optional filtering and pagination',
})
async posts(
  @Parent() user: User,
  @Args() args: PostsArgs,
  @Context() context: GraphQLContext,
): Promise<Post[]> {
  // Implementation
}
```

**Interview Tip**: **`@ResolveField()`** defines **field resolvers** for lazy-loaded fields in `@Resolver()` class. **Usage**: `@ResolveField(() => ReturnType) methodName(@Parent() parent, @Args() args, @Context() ctx)`. **Parent**: use `@Parent()` to access parent object. **Execution**: only runs when field is requested in query (selective). **Common uses**: computed fields (`fullName`), relationships (`posts`, `author`), aggregations (`postsCount`), context-dependent data (`isFollowing`). **Always use DataLoader** for relationship resolvers to avoid N+1. **Key pattern**: model declares field with `@Field()`, resolver computes/fetches value with `@ResolveField()`. Example: User's posts resolver receives User via `@Parent()`, returns posts array, only executes if query requests `posts` field.

</details>

## Validation

38. How do you validate input in GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**Input validation** in GraphQL uses **`class-validator`** decorators on **InputType classes** + **`ValidationPipe`** globally or per-resolver. Define input DTOs with `@InputType()`, add validation decorators (`@IsEmail()`, `@MinLength()`, etc.), enable `ValidationPipe` in `main.ts` or resolver. **Schema validation** (types, required fields) is automatic; **business logic validation** uses class-validator. **Errors** thrown as GraphQL errors with validation messages.

### **Basic Input Validation Setup**

```typescript
// Installation
// npm install class-validator class-transformer

// create-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import { IsEmail, IsNotEmpty, MinLength, MaxLength } from 'class-validator';

@InputType()
export class CreateUserInput {
  @Field()
  @IsNotEmpty({ message: 'Name is required' })
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  @MaxLength(50, { message: 'Name must be at most 50 characters' })
  name: string;

  @Field()
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;
}

// user.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';
import { CreateUserInput } from './dto/create-user.input';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,  // Automatically validated
  ): Promise<User> {
    // Validation happens before this method is called
    return this.userService.create(input);
  }
}

// main.ts - Enable ValidationPipe globally
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable validation globally
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,  // Auto-transform to DTO types
      whitelist: true,  // Strip unknown properties
      forbidNonWhitelisted: true,  // Throw error for unknown properties
    }),
  );
  
  await app.listen(3000);
}
bootstrap();

// GraphQL query with validation:
mutation {
  createUser(input: {
    name: "A"  # Too short - validation fails
    email: "invalid-email"  # Invalid format - validation fails
    password: "short"  # Too short - validation fails
  }) {
    id
    name
  }
}

// Error response:
// {
//   "errors": [
//     {
//       "message": "Bad Request Exception",
//       "extensions": {
//         "code": "BAD_USER_INPUT",
//         "validationErrors": [
//           {
//             "property": "name",
//             "constraints": {
//               "minLength": "Name must be at least 2 characters"
//             }
//           },
//           {
//             "property": "email",
//             "constraints": {
//               "isEmail": "Invalid email format"
//             }
//           },
//           {
//             "property": "password",
//             "constraints": {
//               "minLength": "Password must be at least 8 characters"
//             }
//           }
//         ]
//       }
//     }
//   ]
// }
```

### **Common Validation Decorators**

```typescript
import {
  IsString,
  IsInt,
  IsBoolean,
  IsEmail,
  IsUrl,
  IsUUID,
  IsDate,
  IsEnum,
  IsOptional,
  IsNotEmpty,
  MinLength,
  MaxLength,
  Min,
  Max,
  Matches,
  IsArray,
  ArrayMinSize,
  ArrayMaxSize,
  ValidateNested,
  IsPositive,
  IsIn,
} from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
export class CreatePostInput {
  // String validation
  @Field()
  @IsString()
  @IsNotEmpty()
  @MinLength(5)
  @MaxLength(200)
  title: string;

  // Email validation
  @Field()
  @IsEmail({}, { message: 'Invalid email address' })
  authorEmail: string;

  // URL validation
  @Field({ nullable: true })
  @IsOptional()
  @IsUrl({}, { message: 'Invalid URL format' })
  thumbnailUrl?: string;

  // Number validation
  @Field(() => Int)
  @IsInt()
  @Min(1)
  @Max(100)
  priority: number;

  // Enum validation
  @Field(() => PostStatus)
  @IsEnum(PostStatus, { message: 'Invalid status' })
  status: PostStatus;

  // Boolean validation
  @Field()
  @IsBoolean()
  isPublished: boolean;

  // Array validation
  @Field(() => [String])
  @IsArray()
  @ArrayMinSize(1)
  @ArrayMaxSize(10)
  @IsString({ each: true })
  tags: string[];

  // UUID validation
  @Field()
  @IsUUID('4', { message: 'Invalid category ID format' })
  categoryId: string;

  // Regex pattern validation
  @Field()
  @Matches(/^[a-z0-9-]+$/, {
    message: 'Slug can only contain lowercase letters, numbers, and hyphens',
  })
  slug: string;

  // Date validation
  @Field(() => Date, { nullable: true })
  @IsOptional()
  @IsDate()
  @Type(() => Date)  // Transform to Date object
  publishedAt?: Date;

  // Nested object validation
  @Field(() => PostMetadataInput, { nullable: true })
  @IsOptional()
  @ValidateNested()
  @Type(() => PostMetadataInput)
  metadata?: PostMetadataInput;
}

@InputType()
class PostMetadataInput {
  @Field()
  @IsString()
  description: string;

  @Field(() => [String])
  @IsArray()
  keywords: string[];
}
```

### **Custom Validation**

```typescript
// Custom validator decorator
import {
  registerDecorator,
  ValidationOptions,
  ValidatorConstraint,
  ValidatorConstraintInterface,
  ValidationArguments,
} from 'class-validator';
import { Injectable } from '@nestjs/common';

// Custom validator: Check if username is unique
@ValidatorConstraint({ name: 'IsUsernameUnique', async: true })
@Injectable()
export class IsUsernameUniqueConstraint implements ValidatorConstraintInterface {
  constructor(private userService: UserService) {}

  async validate(username: string): Promise<boolean> {
    const user = await this.userService.findByUsername(username);
    return !user;  // Returns true if username doesn't exist (is unique)
  }

  defaultMessage(args: ValidationArguments): string {
    return `Username "${args.value}" is already taken`;
  }
}

// Custom decorator function
export function IsUsernameUnique(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsUsernameUniqueConstraint,
    });
  };
}

// Usage in InputType
@InputType()
export class CreateUserInput {
  @Field()
  @IsNotEmpty()
  @MinLength(3)
  @IsUsernameUnique()  // Custom async validator
  username: string;
}

// Register custom validator in module
@Module({
  providers: [
    UserService,
    UserResolver,
    IsUsernameUniqueConstraint,  // Register validator
  ],
})
export class UserModule {}
```

### **Conditional Validation**

```typescript
import { ValidateIf } from 'class-validator';

@InputType()
export class UpdateUserInput {
  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  // Only validate if email is being changed
  @Field({ nullable: true })
  @ValidateIf(o => o.email !== undefined)
  @IsNotEmpty({ message: 'Email confirmation is required when changing email' })
  @Matches(/^\d{6}$/, { message: 'Invalid confirmation code' })
  emailConfirmationCode?: string;

  @Field({ nullable: true })
  @IsOptional()
  @MinLength(8)
  newPassword?: string;

  // Only validate if newPassword is provided
  @Field({ nullable: true })
  @ValidateIf(o => o.newPassword !== undefined)
  @IsNotEmpty({ message: 'Current password required when changing password' })
  currentPassword?: string;
}
```

### **Cross-Field Validation**

```typescript
import { Validate, ValidationArguments } from 'class-validator';

// Custom validator for cross-field validation
@ValidatorConstraint({ name: 'IsPasswordMatch' })
class IsPasswordMatchConstraint implements ValidatorConstraintInterface {
  validate(confirmPassword: string, args: ValidationArguments): boolean {
    const object = args.object as any;
    return confirmPassword === object.password;
  }

  defaultMessage(): string {
    return 'Passwords do not match';
  }
}

@InputType()
export class RegisterInput {
  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;

  @Field()
  @Validate(IsPasswordMatchConstraint)  // Cross-field validation
  confirmPassword: string;
}
```

### **Complete Production Example**

```typescript
// create-product.input.ts
import { InputType, Field, Int, Float } from '@nestjs/graphql';
import {
  IsNotEmpty,
  IsString,
  MinLength,
  MaxLength,
  IsNumber,
  Min,
  IsUrl,
  IsOptional,
  IsArray,
  ArrayMinSize,
  IsEnum,
  ValidateNested,
  IsUUID,
  Max,
} from 'class-validator';
import { Type } from 'class-transformer';

enum ProductStatus {
  DRAFT = 'DRAFT',
  PUBLISHED = 'PUBLISHED',
  ARCHIVED = 'ARCHIVED',
}

@InputType()
class ProductDimensionsInput {
  @Field(() => Float)
  @IsNumber()
  @Min(0.1)
  @Max(1000)
  length: number;

  @Field(() => Float)
  @IsNumber()
  @Min(0.1)
  @Max(1000)
  width: number;

  @Field(() => Float)
  @IsNumber()
  @Min(0.1)
  @Max(1000)
  height: number;

  @Field(() => Float)
  @IsNumber()
  @Min(0.01)
  @Max(1000)
  weight: number;
}

@InputType()
export class CreateProductInput {
  // Basic fields
  @Field()
  @IsNotEmpty({ message: 'Product name is required' })
  @IsString()
  @MinLength(3, { message: 'Product name must be at least 3 characters' })
  @MaxLength(200, { message: 'Product name must be at most 200 characters' })
  name: string;

  @Field()
  @IsNotEmpty()
  @IsString()
  @MinLength(10)
  @MaxLength(2000)
  description: string;

  // Price validation
  @Field(() => Float)
  @IsNumber({}, { message: 'Price must be a number' })
  @Min(0.01, { message: 'Price must be at least $0.01' })
  @Max(1000000, { message: 'Price cannot exceed $1,000,000' })
  price: number;

  // Stock validation
  @Field(() => Int)
  @IsNumber()
  @Min(0, { message: 'Stock cannot be negative' })
  stock: number;

  // Category reference
  @Field()
  @IsUUID('4', { message: 'Invalid category ID' })
  categoryId: string;

  // Optional fields
  @Field({ nullable: true })
  @IsOptional()
  @IsUrl({}, { message: 'Invalid thumbnail URL' })
  thumbnailUrl?: string;

  @Field(() => [String], { nullable: true })
  @IsOptional()
  @IsArray()
  @ArrayMinSize(1)
  @IsUrl({}, { each: true, message: 'Each image must be a valid URL' })
  images?: string[];

  // Tags
  @Field(() => [String])
  @IsArray()
  @ArrayMinSize(1, { message: 'At least one tag is required' })
  @IsString({ each: true })
  tags: string[];

  // Enum validation
  @Field(() => ProductStatus)
  @IsEnum(ProductStatus, { message: 'Invalid product status' })
  status: ProductStatus;

  // Nested object validation
  @Field(() => ProductDimensionsInput, { nullable: true })
  @IsOptional()
  @ValidateNested()
  @Type(() => ProductDimensionsInput)
  dimensions?: ProductDimensionsInput;
}

// product.resolver.ts
@Resolver(() => Product)
export class ProductResolver {
  constructor(private productService: ProductService) {}

  @Mutation(() => Product)
  async createProduct(
    @Args('input') input: CreateProductInput,
  ): Promise<Product> {
    // Input is already validated by ValidationPipe
    return this.productService.create(input);
  }

  @Mutation(() => Product)
  async updateProduct(
    @Args('id') id: string,
    @Args('input') input: UpdateProductInput,
  ): Promise<Product> {
    return this.productService.update(id, input);
  }
}

// GraphQL mutation:
mutation {
  createProduct(input: {
    name: "Gaming Laptop"
    description: "High-performance laptop for gaming"
    price: 1299.99
    stock: 10
    categoryId: "550e8400-e29b-41d4-a716-446655440000"
    tags: ["gaming", "laptop", "tech"]
    status: PUBLISHED
    dimensions: {
      length: 35.5
      width: 24.5
      height: 2.5
      weight: 2.3
    }
  }) {
    id
    name
    price
  }
}
```

### **Validation Error Handling**

```typescript
// custom-validation.pipe.ts
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  BadRequestException,
} from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
export class CustomValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }

    const object = plainToClass(metatype, value);
    const errors = await validate(object);

    if (errors.length > 0) {
      // Custom error formatting
      const formattedErrors = errors.map(error => ({
        field: error.property,
        errors: Object.values(error.constraints || {}),
      }));

      throw new BadRequestException({
        message: 'Validation failed',
        errors: formattedErrors,
      });
    }

    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

**Interview Tip**: **Validate GraphQL inputs** with **`class-validator`** decorators on `@InputType()` classes + **`ValidationPipe`**. **Setup**: install class-validator, create input DTO with validation decorators (`@IsEmail()`, `@MinLength()`, etc.), enable `ValidationPipe` globally. **Automatic**: schema validation (types, required) by GraphQL, business logic validation by class-validator. **Common decorators**: `@IsNotEmpty()`, `@IsEmail()`, `@MinLength()`, `@Min()`, `@IsEnum()`, `@IsArray()`, `@ValidateNested()`. **Custom validation**: create `ValidatorConstraint` class, use `@Validate()` or custom decorator. **Errors**: thrown as GraphQL errors with `BAD_USER_INPUT` code, include field-level validation messages. **Key**: validation happens before resolver method executes, invalid input never reaches business logic.

</details>

39. How do you use class-validator with GraphQL inputs?

<details>
<summary><strong>Answer</strong></summary>

**class-validator** integrates with GraphQL via **`@InputType()` decorators** + validation decorators on input properties. Define input DTOs with `@InputType()`, add validation rules (`@IsString()`, `@IsEmail()`, `@MinLength()`, etc.) to fields marked with `@Field()`. Enable `ValidationPipe` to automatically validate inputs before resolver execution. Supports **sync/async validators**, **custom validators**, **nested validation**, and **conditional validation**.

### **Basic Usage**

```typescript
// Installation
// npm install class-validator class-transformer

// dto/create-user.input.ts
import { InputType, Field } from '@nestjs/graphql';
import {
  IsEmail,
  IsNotEmpty,
  MinLength,
  MaxLength,
  IsString,
  Matches,
} from 'class-validator';

@InputType()  // Marks as GraphQL input type
export class CreateUserInput {
  // Combine @Field() (GraphQL) with validation decorators (class-validator)
  @Field()  // GraphQL field
  @IsNotEmpty({ message: 'Name cannot be empty' })  // Validation
  @IsString({ message: 'Name must be a string' })
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  @MaxLength(50, { message: 'Name must not exceed 50 characters' })
  name: string;

  @Field()
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @Field()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;
}

// user.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { User } from './user.model';
import { CreateUserInput } from './dto/create-user.input';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,  // Automatically validated
  ): Promise<User> {
    // Validation already passed at this point
    return this.userService.create(input);
  }
}

// main.ts - Enable ValidationPipe
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,  // Auto-transform plain objects to class instances
      whitelist: true,  // Strip non-whitelisted properties
    }),
  );
  
  await app.listen(3000);
}
```

### **All Validation Decorators**

```typescript
import {
  // Type validators
  IsString,
  IsNumber,
  IsInt,
  IsBoolean,
  IsDate,
  IsArray,
  IsObject,
  IsEnum,
  
  // Format validators
  IsEmail,
  IsUrl,
  IsUUID,
  IsISO8601,
  IsPhoneNumber,
  IsCreditCard,
  IsIP,
  IsJSON,
  
  // String validators
  MinLength,
  MaxLength,
  Matches,
  Contains,
  IsAlpha,
  IsAlphanumeric,
  IsLowercase,
  IsUppercase,
  
  // Number validators
  Min,
  Max,
  IsPositive,
  IsNegative,
  IsDivisibleBy,
  
  // Array validators
  ArrayMinSize,
  ArrayMaxSize,
  ArrayNotEmpty,
  ArrayUnique,
  
  // Conditional
  IsOptional,
  IsNotEmpty,
  ValidateIf,
  
  // Nested validation
  ValidateNested,
  
  // Custom
  Validate,
} from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
export class ComprehensiveInput {
  // String validation
  @Field()
  @IsString()
  @MinLength(5)
  @MaxLength(100)
  @Matches(/^[a-zA-Z0-9 ]+$/, { message: 'Only alphanumeric characters allowed' })
  title: string;

  // Email
  @Field()
  @IsEmail({}, { message: 'Invalid email address' })
  email: string;

  // URL
  @Field({ nullable: true })
  @IsOptional()
  @IsUrl({ protocols: ['http', 'https'] }, { message: 'Invalid URL' })
  website?: string;

  // Number validation
  @Field(() => Int)
  @IsInt()
  @Min(1, { message: 'Age must be at least 1' })
  @Max(120, { message: 'Age cannot exceed 120' })
  age: number;

  @Field(() => Float)
  @IsNumber({ maxDecimalPlaces: 2 })
  @IsPositive()
  price: number;

  // Boolean
  @Field()
  @IsBoolean()
  isActive: boolean;

  // Enum
  @Field(() => UserRole)
  @IsEnum(UserRole, { message: 'Invalid role' })
  role: UserRole;

  // Array validation
  @Field(() => [String])
  @IsArray()
  @ArrayNotEmpty({ message: 'At least one tag required' })
  @ArrayMinSize(1)
  @ArrayMaxSize(10)
  @IsString({ each: true })
  tags: string[];

  // UUID
  @Field()
  @IsUUID('4', { message: 'Invalid UUID format' })
  userId: string;

  // Date
  @Field(() => Date)
  @IsDate()
  @Type(() => Date)  // Transform string to Date
  birthDate: Date;

  // Phone number
  @Field({ nullable: true })
  @IsOptional()
  @IsPhoneNumber('US', { message: 'Invalid US phone number' })
  phone?: string;

  // Nested object validation
  @Field(() => AddressInput)
  @ValidateNested()
  @Type(() => AddressInput)
  address: AddressInput;

  // Array of nested objects
  @Field(() => [SocialLinkInput], { nullable: true })
  @IsOptional()
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => SocialLinkInput)
  socialLinks?: SocialLinkInput[];
}

@InputType()
class AddressInput {
  @Field()
  @IsString()
  @IsNotEmpty()
  street: string;

  @Field()
  @IsString()
  @IsNotEmpty()
  city: string;

  @Field()
  @Matches(/^\d{5}$/, { message: 'ZIP code must be 5 digits' })
  zipCode: string;
}

@InputType()
class SocialLinkInput {
  @Field()
  @IsEnum(['twitter', 'linkedin', 'github'])
  platform: string;

  @Field()
  @IsUrl()
  url: string;
}
```

### **Custom Validators**

```typescript
// validators/is-strong-password.validator.ts
import {
  registerDecorator,
  ValidationOptions,
  ValidatorConstraint,
  ValidatorConstraintInterface,
  ValidationArguments,
} from 'class-validator';

// Custom validator: Strong password
@ValidatorConstraint({ name: 'isStrongPassword' })
export class IsStrongPasswordConstraint implements ValidatorConstraintInterface {
  validate(password: string, args: ValidationArguments): boolean {
    if (!password) return false;
    
    // Check password strength
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    const isLongEnough = password.length >= 8;
    
    return hasUpperCase && hasLowerCase && hasNumbers && hasSpecialChar && isLongEnough;
  }

  defaultMessage(args: ValidationArguments): string {
    return 'Password must contain uppercase, lowercase, number, special character, and be at least 8 characters';
  }
}

// Custom decorator
export function IsStrongPassword(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsStrongPasswordConstraint,
    });
  };
}

// Usage
@InputType()
export class RegisterInput {
  @Field()
  @IsEmail()
  email: string;

  @Field()
  @IsStrongPassword()  // Custom validator
  password: string;
}
```

### **Async Validators (Database Checks)**

```typescript
// validators/is-email-unique.validator.ts
import {
  ValidatorConstraint,
  ValidatorConstraintInterface,
  ValidationArguments,
} from 'class-validator';
import { Injectable } from '@nestjs/common';
import { UserService } from '../user.service';

@ValidatorConstraint({ name: 'IsEmailUnique', async: true })  // async: true
@Injectable()
export class IsEmailUniqueConstraint implements ValidatorConstraintInterface {
  constructor(private userService: UserService) {}

  async validate(email: string, args: ValidationArguments): Promise<boolean> {
    // Async database check
    const user = await this.userService.findByEmail(email);
    return !user;  // True if email is unique (not found)
  }

  defaultMessage(args: ValidationArguments): string {
    return `Email "${args.value}" is already registered`;
  }
}

// Decorator
export function IsEmailUnique(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsEmailUniqueConstraint,
    });
  };
}

// Register in module
@Module({
  providers: [
    UserService,
    UserResolver,
    IsEmailUniqueConstraint,  // Register async validator
  ],
})
export class UserModule {}

// Usage
@InputType()
export class CreateUserInput {
  @Field()
  @IsEmail()
  @IsEmailUnique()  // Async validator - checks database
  email: string;
}
```

### **Conditional Validation**

```typescript
import { ValidateIf, Equals } from 'class-validator';

@InputType()
export class UpdateProfileInput {
  @Field({ nullable: true })
  @IsOptional()
  @IsEmail()
  email?: string;

  // Only validate if email is being changed
  @Field({ nullable: true })
  @ValidateIf(o => o.email !== undefined)
  @IsNotEmpty({ message: 'Verification code required when changing email' })
  @Matches(/^\d{6}$/, { message: 'Verification code must be 6 digits' })
  emailVerificationCode?: string;

  @Field({ nullable: true })
  @IsOptional()
  @MinLength(8)
  newPassword?: string;

  // Only validate if newPassword is provided
  @Field({ nullable: true })
  @ValidateIf(o => o.newPassword !== undefined)
  @IsNotEmpty({ message: 'Current password required when changing password' })
  currentPassword?: string;

  // Conditional enum validation
  @Field({ nullable: true })
  @IsOptional()
  @IsEnum(['basic', 'premium', 'enterprise'])
  planType?: string;

  // Only required if planType is 'enterprise'
  @Field({ nullable: true })
  @ValidateIf(o => o.planType === 'enterprise')
  @IsNotEmpty({ message: 'Company name required for enterprise plan' })
  companyName?: string;
}
```

### **Cross-Field Validation**

```typescript
import { Validate, ValidationArguments } from 'class-validator';

// Password match validator
@ValidatorConstraint({ name: 'PasswordMatch' })
class PasswordMatchConstraint implements ValidatorConstraintInterface {
  validate(confirmPassword: string, args: ValidationArguments): boolean {
    const object = args.object as any;
    return confirmPassword === object.password;
  }

  defaultMessage(args: ValidationArguments): string {
    return 'Password confirmation does not match';
  }
}

// Date range validator
@ValidatorConstraint({ name: 'IsDateAfter' })
class IsDateAfterConstraint implements ValidatorConstraintInterface {
  validate(endDate: Date, args: ValidationArguments): boolean {
    const object = args.object as any;
    return endDate > object.startDate;
  }

  defaultMessage(args: ValidationArguments): string {
    return 'End date must be after start date';
  }
}

@InputType()
export class CreateEventInput {
  @Field()
  @IsNotEmpty()
  title: string;

  @Field(() => Date)
  @IsDate()
  @Type(() => Date)
  startDate: Date;

  @Field(() => Date)
  @IsDate()
  @Type(() => Date)
  @Validate(IsDateAfterConstraint)  // Cross-field validation
  endDate: Date;
}

@InputType()
export class RegisterInput {
  @Field()
  @IsEmail()
  email: string;

  @Field()
  @MinLength(8)
  password: string;

  @Field()
  @Validate(PasswordMatchConstraint)  // Cross-field validation
  confirmPassword: string;
}
```

### **Complete Production Example**

```typescript
// dto/create-article.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import {
  IsNotEmpty,
  IsString,
  MinLength,
  MaxLength,
  IsUrl,
  IsOptional,
  IsEnum,
  IsArray,
  ArrayMinSize,
  ArrayMaxSize,
  ValidateNested,
  IsUUID,
  Matches,
} from 'class-validator';
import { Type } from 'class-transformer';

enum ArticleStatus {
  DRAFT = 'DRAFT',
  PUBLISHED = 'PUBLISHED',
  ARCHIVED = 'ARCHIVED',
}

@InputType()
class ArticleMetadataInput {
  @Field()
  @IsString()
  @MaxLength(160)
  description: string;

  @Field(() => [String])
  @IsArray()
  @ArrayMaxSize(10)
  @IsString({ each: true })
  keywords: string[];

  @Field({ nullable: true })
  @IsOptional()
  @IsUrl()
  canonicalUrl?: string;
}

@InputType()
export class CreateArticleInput {
  // Title with custom slug
  @Field()
  @IsNotEmpty({ message: 'Title is required' })
  @MinLength(5, { message: 'Title must be at least 5 characters' })
  @MaxLength(200, { message: 'Title cannot exceed 200 characters' })
  title: string;

  @Field()
  @IsString()
  @Matches(/^[a-z0-9]+(?:-[a-z0-9]+)*$/, {
    message: 'Slug must be lowercase with hyphens (e.g., my-article-title)',
  })
  slug: string;

  // Content
  @Field()
  @IsNotEmpty()
  @MinLength(100, { message: 'Content must be at least 100 characters' })
  content: string;

  // Featured image
  @Field({ nullable: true })
  @IsOptional()
  @IsUrl({}, { message: 'Invalid featured image URL' })
  featuredImageUrl?: string;

  // Category
  @Field()
  @IsUUID('4', { message: 'Invalid category ID' })
  categoryId: string;

  // Tags
  @Field(() => [String])
  @IsArray({ message: 'Tags must be an array' })
  @ArrayMinSize(1, { message: 'At least one tag is required' })
  @ArrayMaxSize(5, { message: 'Maximum 5 tags allowed' })
  @IsString({ each: true })
  tags: string[];

  // Status
  @Field(() => ArticleStatus)
  @IsEnum(ArticleStatus, { message: 'Invalid article status' })
  status: ArticleStatus;

  // SEO metadata (nested validation)
  @Field(() => ArticleMetadataInput)
  @ValidateNested()
  @Type(() => ArticleMetadataInput)
  metadata: ArticleMetadataInput;
}

// article.resolver.ts
@Resolver(() => Article)
export class ArticleResolver {
  constructor(private articleService: ArticleService) {}

  @Mutation(() => Article)
  async createArticle(
    @Args('input') input: CreateArticleInput,
    @Context() { user }: GraphQLContext,
  ): Promise<Article> {
    // Input is fully validated at this point
    return this.articleService.create(input, user.id);
  }
}

// GraphQL mutation:
mutation {
  createArticle(input: {
    title: "Getting Started with NestJS GraphQL"
    slug: "getting-started-nestjs-graphql"
    content: "Lorem ipsum dolor sit amet... (100+ chars)"
    categoryId: "550e8400-e29b-41d4-a716-446655440000"
    tags: ["nestjs", "graphql", "typescript"]
    status: PUBLISHED
    metadata: {
      description: "Learn how to build GraphQL APIs with NestJS"
      keywords: ["nestjs", "graphql", "api"]
    }
  }) {
    id
    title
    slug
  }
}
```

### **Validation Groups**

```typescript
// Different validation for create vs update
import { MinLength, IsEmail } from 'class-validator';

@InputType()
export class UserInput {
  @Field()
  @MinLength(2, { groups: ['create', 'update'] })
  name: string;

  @Field()
  @IsEmail({}, { groups: ['create'] })  // Only validate on create
  email: string;

  @Field({ nullable: true })
  @MinLength(8, { groups: ['create'] })  // Only required on create
  password?: string;
}

// In resolver, specify validation groups
@Mutation(() => User)
async createUser(
  @Args('input', new ValidationPipe({ groups: ['create'] }))
  input: UserInput,
): Promise<User> {
  return this.userService.create(input);
}

@Mutation(() => User)
async updateUser(
  @Args('id') id: string,
  @Args('input', new ValidationPipe({ groups: ['update'] }))
  input: UserInput,
): Promise<User> {
  return this.userService.update(id, input);
}
```

**Interview Tip**: Use **class-validator** with GraphQL by combining **`@Field()` (GraphQL) + validation decorators on `@InputType()` classes**. **Pattern**: `@Field()` defines GraphQL schema, validation decorators (`@IsEmail()`, `@MinLength()`, etc.) define validation rules. **Automatic**: `ValidationPipe` intercepts inputs, transforms to class instance, runs validators, throws on failure. **Key decorators**: `@IsNotEmpty()`, `@IsString()`, `@MinLength()`, `@IsEmail()`, `@IsEnum()`, `@ValidateNested()` (nested), `@IsOptional()` (nullable). **Custom validators**: implement `ValidatorConstraintInterface`, use `async: true` for database checks. **Cross-field**: access full object via `args.object` in validator. **Best practice**: always use `@Type()` from class-transformer for nested objects/dates to ensure proper transformation.

</details>

40. How do you use ValidationPipe with GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**ValidationPipe** automatically validates GraphQL inputs decorated with **class-validator**. Enable **globally** in `main.ts` with `app.useGlobalPipes()` or **per-resolver** with `@UsePipes()`. Transforms plain objects to class instances, runs validation, throws `BadRequestException` on failure. Configure with options: **`transform`** (auto-transform types), **`whitelist`** (strip unknown props), **`forbidNonWhitelisted`** (reject unknown props), **`skipMissingProperties`**.

### **Global ValidationPipe Setup**

```typescript
// main.ts - Enable globally (recommended)
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable ValidationPipe globally for all routes/resolvers
  app.useGlobalPipes(
    new ValidationPipe({
      // Transform plain objects to class instances
      transform: true,
      
      // Remove properties not in DTO class
      whitelist: true,
      
      // Throw error if non-whitelisted properties are present
      forbidNonWhitelisted: true,
      
      // Skip validation for missing properties (nullable fields)
      skipMissingProperties: false,
      
      // Disable detailed error messages in production
      disableErrorMessages: false,
      
      // Validate arrays/nested objects
      validateCustomDecorators: true,
      
      // Transform payload based on DTO class types
      transformOptions: {
        enableImplicitConversion: true,  // Auto-convert types (string -> number)
      },
    }),
  );

  await app.listen(3000);
}
bootstrap();

// Now all GraphQL inputs are automatically validated!
```

### **Per-Resolver ValidationPipe**

```typescript
// user.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { UsePipes, ValidationPipe } from '@nestjs/common';
import { User } from './user.model';
import { CreateUserInput } from './dto/create-user.input';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Option 1: Apply ValidationPipe to entire resolver
  @UsePipes(new ValidationPipe({ transform: true }))
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    return this.userService.create(input);
  }

  // Option 2: Apply ValidationPipe to specific argument
  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args(
      'input',
      new ValidationPipe({
        transform: true,
        whitelist: true,
      }),
    )
    input: UpdateUserInput,
  ): Promise<User> {
    return this.userService.update(id, input);
  }

  // Option 3: Apply to all methods in resolver
}

// Apply to all methods
@Resolver(() => User)
@UsePipes(new ValidationPipe({ transform: true, whitelist: true }))
export class UserResolver {
  // All mutations/queries in this resolver are validated
}
```

### **ValidationPipe Configuration Options**

```typescript
import { ValidationPipe, ValidationPipeOptions } from '@nestjs/common';

const validationOptions: ValidationPipeOptions = {
  // === Transformation Options ===
  
  // Transform plain object to class instance
  transform: true,
  
  // Transformation options (from class-transformer)
  transformOptions: {
    // Auto-convert primitive types ("123" -> 123)
    enableImplicitConversion: true,
    
    // Exclude properties with @Exclude() decorator
    excludeExtraneousValues: false,
  },

  // === Validation Options ===
  
  // Strip properties not in DTO class
  whitelist: true,
  
  // Throw error for non-whitelisted properties
  forbidNonWhitelisted: true,
  
  // Skip validation for undefined properties
  skipMissingProperties: false,
  
  // Skip validation for null values
  skipNullProperties: false,
  
  // Skip validation for undefined values
  skipUndefinedProperties: false,

  // === Error Options ===
  
  // Disable error messages (production)
  disableErrorMessages: false,
  
  // Custom error message formatter
  exceptionFactory: (errors) => {
    return new BadRequestException({
      message: 'Validation failed',
      errors: errors.map(error => ({
        field: error.property,
        constraints: error.constraints,
      })),
    });
  },

  // === Other Options ===
  
  // Validate custom decorators
  validateCustomDecorators: true,
  
  // Stop at first error
  stopAtFirstError: false,
  
  // Validation groups (for conditional validation)
  groups: [],
  
  // Always validate (ignore @IsOptional() in some cases)
  always: false,
};

app.useGlobalPipes(new ValidationPipe(validationOptions));
```

### **Transform Option (Type Conversion)**

```typescript
// dto/create-post.input.ts
import { InputType, Field, Int } from '@nestjs/graphql';
import { IsInt, Min, IsDate } from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
export class CreatePostInput {
  @Field()
  title: string;

  // Without transform: categoryId would be string from GraphQL
  // With transform: automatically converted to number
  @Field(() => Int)
  @IsInt()
  @Min(1)
  categoryId: number;  // Auto-converted from GraphQL Int

  // Date transformation
  @Field(() => Date)
  @IsDate()
  @Type(() => Date)  // Required: transform string to Date
  publishedAt: Date;
}

// With transform: true, ValidationPipe automatically converts types
@Mutation(() => Post)
async createPost(
  @Args('input') input: CreatePostInput,
): Promise<Post> {
  // input.categoryId is number (not string)
  // input.publishedAt is Date object (not string)
  console.log(typeof input.categoryId);  // 'number'
  console.log(input.publishedAt instanceof Date);  // true
  
  return this.postService.create(input);
}
```

### **Whitelist Option (Strip Unknown Properties)**

```typescript
// dto/create-user.input.ts
@InputType()
export class CreateUserInput {
  @Field()
  @IsString()
  name: string;

  @Field()
  @IsEmail()
  email: string;

  // Only these two fields are defined
}

// GraphQL mutation with extra fields:
mutation {
  createUser(input: {
    name: "John"
    email: "john@example.com"
    isAdmin: true  # NOT in CreateUserInput
    deleteAllUsers: true  # NOT in CreateUserInput
  }) {
    id
  }
}

// With whitelist: true
// - Extra fields (isAdmin, deleteAllUsers) are silently removed
// - input only contains { name, email }

// With forbidNonWhitelisted: true
// - Throws error: "property isAdmin should not exist"
// - Prevents malicious/accidental extra fields
```

### **Custom Error Messages**

```typescript
// Custom exception factory
import { BadRequestException } from '@nestjs/common';
import { ValidationError } from 'class-validator';

function customExceptionFactory(errors: ValidationError[]) {
  // Format errors for GraphQL
  const formattedErrors = errors.map(error => ({
    field: error.property,
    value: error.value,
    constraints: Object.values(error.constraints || {}),
  }));

  return new BadRequestException({
    statusCode: 400,
    message: 'Input validation failed',
    errors: formattedErrors,
    timestamp: new Date().toISOString(),
  });
}

app.useGlobalPipes(
  new ValidationPipe({
    exceptionFactory: customExceptionFactory,
  }),
);

// Error response format:
// {
//   "errors": [{
//     "message": {
//       "statusCode": 400,
//       "message": "Input validation failed",
//       "errors": [
//         {
//           "field": "email",
//           "value": "invalid-email",
//           "constraints": ["Invalid email format"]
//         }
//       ],
//       "timestamp": "2024-01-15T10:30:00.000Z"
//     },
//     "extensions": { "code": "BAD_USER_INPUT" }
//   }]
// }
```

### **ValidationPipe with Nested Objects**

```typescript
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
class AddressInput {
  @Field()
  @IsString()
  street: string;

  @Field()
  @IsString()
  city: string;
}

@InputType()
export class CreateUserInput {
  @Field()
  @IsString()
  name: string;

  // Nested validation
  @Field(() => AddressInput)
  @ValidateNested()  // Important: validate nested object
  @Type(() => AddressInput)  // Important: transform to class instance
  address: AddressInput;

  // Array of nested objects
  @Field(() => [AddressInput], { nullable: true })
  @IsOptional()
  @IsArray()
  @ValidateNested({ each: true })  // Validate each element
  @Type(() => AddressInput)  // Transform each element
  additionalAddresses?: AddressInput[];
}

// ValidationPipe with transform: true will:
// 1. Transform plain address object to AddressInput instance
// 2. Validate all fields in AddressInput
// 3. Transform array elements to AddressInput instances
// 4. Validate each array element
```

### **Environment-Specific Configuration**

```typescript
// main.ts - Different configs for dev/prod
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);
  const isProduction = configService.get('NODE_ENV') === 'production';

  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      whitelist: true,
      forbidNonWhitelisted: !isProduction,  // Only strict in dev
      disableErrorMessages: isProduction,   // Hide details in prod
      
      // Custom error handling in production
      exceptionFactory: (errors) => {
        if (isProduction) {
          // Generic error in production
          return new BadRequestException('Invalid input');
        }
        // Detailed errors in development
        return new BadRequestException({
          message: 'Validation failed',
          errors: errors.map(e => ({
            field: e.property,
            constraints: e.constraints,
          })),
        });
      },
    }),
  );

  await app.listen(3000);
}
```

### **Complete Production Example**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe, BadRequestException } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Production-ready ValidationPipe
  app.useGlobalPipes(
    new ValidationPipe({
      // Transform plain objects to DTOs
      transform: true,
      
      // Auto-convert primitive types
      transformOptions: {
        enableImplicitConversion: true,
      },
      
      // Security: strip unknown properties
      whitelist: true,
      
      // Security: reject unknown properties
      forbidNonWhitelisted: true,
      
      // Validate all properties
      skipMissingProperties: false,
      skipNullProperties: false,
      skipUndefinedProperties: false,
      
      // Validate custom decorators
      validateCustomDecorators: true,
      
      // Format errors for GraphQL
      exceptionFactory: (errors) => {
        const formattedErrors = errors.map(error => {
          const constraints = error.constraints
            ? Object.values(error.constraints)
            : [];
          
          return {
            field: error.property,
            errors: constraints,
          };
        });

        return new BadRequestException({
          message: 'Validation failed',
          validationErrors: formattedErrors,
        });
      },
    }),
  );

  await app.listen(3000);
}
bootstrap();

// dto/create-order.input.ts
import { InputType, Field, Int, Float } from '@nestjs/graphql';
import {
  IsUUID,
  IsInt,
  Min,
  IsArray,
  ArrayMinSize,
  ValidateNested,
  IsNumber,
  IsOptional,
  IsString,
} from 'class-validator';
import { Type } from 'class-transformer';

@InputType()
class OrderItemInput {
  @Field()
  @IsUUID('4')
  productId: string;

  @Field(() => Int)
  @IsInt()
  @Min(1)
  quantity: number;
}

@InputType()
export class CreateOrderInput {
  @Field()
  @IsUUID('4')
  customerId: string;

  @Field(() => [OrderItemInput])
  @IsArray()
  @ArrayMinSize(1, { message: 'Order must contain at least one item' })
  @ValidateNested({ each: true })
  @Type(() => OrderItemInput)
  items: OrderItemInput[];

  @Field({ nullable: true })
  @IsOptional()
  @IsString()
  notes?: string;
}

// order.resolver.ts
@Resolver(() => Order)
export class OrderResolver {
  constructor(private orderService: OrderService) {}

  // ValidationPipe automatically validates CreateOrderInput
  @Mutation(() => Order)
  async createOrder(
    @Args('input') input: CreateOrderInput,
  ): Promise<Order> {
    // Input is guaranteed to be valid:
    // - customerId is valid UUID
    // - items array has at least 1 item
    // - each item has valid productId and quantity
    // - no unknown properties present
    // - all types are properly converted
    
    return this.orderService.create(input);
  }
}
```

### **Selective Validation (Skip for Specific Args)**

```typescript
// Skip validation for specific arguments
@Mutation(() => User)
async updateUser(
  @Args('id') id: string,  // Not validated (plain string)
  @Args('input') input: UpdateUserInput,  // Validated by global pipe
): Promise<User> {
  return this.userService.update(id, input);
}

// Disable validation for specific argument
@Mutation(() => User)
async partialUpdate(
  @Args('id') id: string,
  @Args('data', { type: () => GraphQLJSONObject })  // Skip validation
  data: Record<string, any>,
): Promise<User> {
  // data is not validated - use with caution!
  return this.userService.partialUpdate(id, data);
}
```

**Interview Tip**: **ValidationPipe** integrates with GraphQL by **intercepting arguments, transforming to DTO classes, running class-validator, throwing on errors**. **Setup**: enable globally with `app.useGlobalPipes(new ValidationPipe())` or per-resolver with `@UsePipes()`. **Key options**: `transform: true` (auto-convert types), `whitelist: true` (strip unknown props), `forbidNonWhitelisted: true` (reject unknown props for security). **How it works**: plain GraphQL input → transform to class instance → run validators → throw `BadRequestException` if invalid → return validated DTO to resolver. **Automatic**: works seamlessly with `@InputType()` + class-validator decorators. **Best practice**: always enable globally with `transform`, `whitelist`, and `forbidNonWhitelisted` for security. Use `@Type()` for nested objects/dates. **Result**: resolver always receives valid, type-safe input.

</details>

## Authentication & Authorization

41. How do you implement authentication in GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**Authentication** in GraphQL uses **Guards** (from REST) + **context** to extract/validate tokens. Extract **JWT/session from request headers** in GraphQL context, attach **user to context**, use **`@UseGuards(AuthGuard)`** on resolvers to protect queries/mutations. Common pattern: **JwtAuthGuard** validates token, populates `context.user`, resolver accesses via **`@Context()`** or custom **`@CurrentUser()`** decorator.

### **Basic JWT Authentication Setup**

```typescript
// Installation
// npm install @nestjs/jwt @nestjs/passport passport passport-jwt
// npm install --save-dev @types/passport-jwt

// auth/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { UserService } from '../user/user.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private userService: UserService) {
    super({
      // Extract JWT from Authorization header
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET || 'your-secret-key',
    });
  }

  // Validate JWT payload and return user
  async validate(payload: any) {
    const user = await this.userService.findById(payload.sub);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }
    return user;  // Attached to context.user
  }
}

// auth/jwt-auth.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  // Override to work with GraphQL context
  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;  // Extract request from GraphQL context
  }
}

// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { JwtStrategy } from './jwt.strategy';
import { AuthService } from './auth.service';
import { AuthResolver } from './auth.resolver';
import { UserModule } from '../user/user.module';

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'your-secret-key',
      signOptions: { expiresIn: '1d' },
    }),
    UserModule,
  ],
  providers: [AuthService, AuthResolver, JwtStrategy],
  exports: [AuthService, JwtModule],
})
export class AuthModule {}
```

### **GraphQL Context with User**

```typescript
// app.module.ts - Add user to GraphQL context
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      context: ({ req }) => ({
        req,  // Include request object in context
        // User will be attached by JwtStrategy after guard validation
      }),
    }),
  ],
})
export class AppModule {}

// The flow:
// 1. Client sends request with Authorization header
// 2. JwtAuthGuard extracts token from header
// 3. JwtStrategy validates token and returns user
// 4. User is attached to req.user
// 5. GraphQL context includes req (with user)
// 6. Resolver accesses user via @Context() or @CurrentUser()
```

### **Protected Resolvers**

```typescript
// user.resolver.ts
import { Resolver, Query, Mutation, Args, Context } from '@nestjs/graphql';
import { UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { User } from './user.model';
import { UserService } from './user.service';

interface GraphQLContext {
  req: {
    user: User;  // Populated by JwtStrategy
  };
}

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Public query (no guard)
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Protected query (requires authentication)
  @Query(() => User)
  @UseGuards(JwtAuthGuard)  // Protect with JWT guard
  async me(@Context() context: GraphQLContext): Promise<User> {
    // User is already validated and attached to context
    return context.req.user;
  }

  // Protected mutation
  @Mutation(() => User)
  @UseGuards(JwtAuthGuard)
  async updateProfile(
    @Context() context: GraphQLContext,
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    const currentUser = context.req.user;
    return this.userService.update(currentUser.id, input);
  }
}

// GraphQL query (with authentication):
// Request headers:
// Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

query {
  me {  // Requires authentication
    id
    name
    email
  }
}
```

### **Custom CurrentUser Decorator**

```typescript
// decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const CurrentUser = createParamDecorator(
  (data: unknown, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user;  // Extract user from request
  },
);

// Usage in resolver (cleaner than @Context())
@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  @UseGuards(JwtAuthGuard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;  // Much cleaner!
  }

  @Mutation(() => Post)
  @UseGuards(JwtAuthGuard)
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user.id, input);
  }
}
```

### **Login/Register Mutations**

```typescript
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UserService } from '../user/user.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private userService: UserService,
    private jwtService: JwtService,
  ) {}

  async register(email: string, password: string, name: string) {
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Create user
    const user = await this.userService.create({
      email,
      password: hashedPassword,
      name,
    });

    // Generate JWT token
    const token = this.generateToken(user);
    
    return { user, token };
  }

  async login(email: string, password: string) {
    // Find user
    const user = await this.userService.findByEmail(email);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Generate token
    const token = this.generateToken(user);
    
    return { user, token };
  }

  private generateToken(user: any): string {
    const payload = { sub: user.id, email: user.email };
    return this.jwtService.sign(payload);
  }
}

// auth/dto/auth-response.model.ts
import { ObjectType, Field } from '@nestjs/graphql';
import { User } from '../../user/user.model';

@ObjectType()
export class AuthResponse {
  @Field(() => User)
  user: User;

  @Field()
  token: string;  // JWT token
}

// auth/auth.resolver.ts
import { Resolver, Mutation, Args } from '@nestjs/graphql';
import { AuthService } from './auth.service';
import { AuthResponse } from './dto/auth-response.model';
import { RegisterInput } from './dto/register.input';
import { LoginInput } from './dto/login.input';

@Resolver()
export class AuthResolver {
  constructor(private authService: AuthService) {}

  @Mutation(() => AuthResponse)
  async register(@Args('input') input: RegisterInput): Promise<AuthResponse> {
    return this.authService.register(input.email, input.password, input.name);
  }

  @Mutation(() => AuthResponse)
  async login(@Args('input') input: LoginInput): Promise<AuthResponse> {
    return this.authService.login(input.email, input.password);
  }
}

// GraphQL mutations:
mutation {
  register(input: {
    email: "user@example.com"
    password: "SecurePass123"
    name: "John Doe"
  }) {
    user {
      id
      name
      email
    }
    token  # Save this for subsequent requests
  }
}

mutation {
  login(input: {
    email: "user@example.com"
    password: "SecurePass123"
  }) {
    user {
      id
      name
    }
    token
  }
}
```

### **Global vs Per-Resolver Guards**

```typescript
// Option 1: Global guard (protect all resolvers by default)
// app.module.ts
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/jwt-auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,  // All resolvers require auth
    },
  ],
})
export class AppModule {}

// Make specific resolvers public with @Public() decorator
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// Update guard to check for @Public() decorator
import { Reflector } from '@nestjs/core';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;  // Skip authentication for public routes
    }
    
    return super.canActivate(context);
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }
}

// Usage
@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  @Public()  // Explicitly public
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Query(() => User)
  // No @Public() = requires authentication (global guard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }
}

// Option 2: Per-resolver guards (opt-in authentication)
// Don't use global guard, add @UseGuards() where needed
@Resolver(() => Post)
export class PostResolver {
  @Query(() => [Post])
  async posts(): Promise<Post[]> {
    return this.postService.findAll();  // Public
  }

  @Mutation(() => Post)
  @UseGuards(JwtAuthGuard)  // Protected
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }
}
```

### **Complete Production Example**

```typescript
// auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { UserService } from '../../user/user.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private configService: ConfigService,
    private userService: UserService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  async validate(payload: { sub: string; email: string }) {
    // Fetch fresh user data from database
    const user = await this.userService.findById(payload.sub);
    
    if (!user) {
      throw new UnauthorizedException('User not found');
    }
    
    if (!user.isActive) {
      throw new UnauthorizedException('Account is deactivated');
    }
    
    // User object will be attached to request
    return user;
  }
}

// auth/guards/gql-auth.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class GqlAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Check if route is marked as public
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    return super.canActivate(context);
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }

  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('Authentication required');
    }
    return user;
  }
}

// decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const CurrentUser = createParamDecorator(
  (data: keyof User | undefined, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    
    // Return specific field if requested
    return data ? user?.[data] : user;
  },
);

// decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { APP_GUARD } from '@nestjs/core';
import { GqlAuthGuard } from './auth/guards/gql-auth.guard';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      context: ({ req }) => ({ req }),  // Pass request to context
    }),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: GqlAuthGuard,  // Global authentication
    },
  ],
})
export class AppModule {}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Public queries
  @Query(() => User, { nullable: true })
  @Public()
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Protected queries (default with global guard)
  @Query(() => User)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  @Mutation(() => User)
  async updateProfile(
    @CurrentUser() user: User,
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    return this.userService.update(user.id, input);
  }

  @Mutation(() => Boolean)
  async deleteAccount(@CurrentUser('id') userId: string): Promise<boolean> {
    await this.userService.delete(userId);
    return true;
  }
}

// auth.resolver.ts
@Resolver()
export class AuthResolver {
  constructor(private authService: AuthService) {}

  @Mutation(() => AuthResponse)
  @Public()  // Public mutation
  async register(@Args('input') input: RegisterInput): Promise<AuthResponse> {
    return this.authService.register(input);
  }

  @Mutation(() => AuthResponse)
  @Public()
  async login(@Args('input') input: LoginInput): Promise<AuthResponse> {
    return this.authService.login(input);
  }

  @Mutation(() => Boolean)
  async logout(@CurrentUser() user: User): Promise<boolean> {
    // Implement logout logic (e.g., token blacklist)
    return true;
  }
}
```

**Interview Tip**: **GraphQL authentication** uses **JWT + Guards + context**. **Pattern**: extract JWT from `Authorization` header in GraphQL context, validate with `JwtAuthGuard` (extends `AuthGuard('jwt')`), attach user to `req.user` via `JwtStrategy.validate()`, access in resolver with `@CurrentUser()` decorator. **Key**: override `getRequest()` in guard to extract request from GraphQL context (`GqlExecutionContext.create(context).getContext().req`). **Setup**: install passport-jwt, create `JwtStrategy` (validates token, returns user), create `JwtAuthGuard` (adapts for GraphQL), use `@UseGuards(JwtAuthGuard)` on protected resolvers. **Custom decorator**: `@CurrentUser()` extracts `req.user` from GraphQL context (cleaner than `@Context()`). **Global**: use `APP_GUARD` + `@Public()` decorator to protect all by default. **Login flow**: register/login mutations return JWT token, client includes in `Authorization: Bearer <token>` header.

</details>

42. How do you use Guards with GraphQL resolvers?

<details>
<summary><strong>Answer</strong></summary>

**Guards** protect GraphQL resolvers by implementing `CanActivate` interface and **overriding `getRequest()`** to extract request from **`GqlExecutionContext`**. Apply with **`@UseGuards()`** decorator on resolver methods or classes. Guards run **before** resolver execution, return **`true`** (allow) or **`false`** (deny), can throw exceptions. Common guards: **`JwtAuthGuard`** (authentication), **`RolesGuard`** (authorization), **custom business logic guards**.

### **Basic Guard for GraphQL**

```typescript
// guards/auth.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Convert to GraphQL context
    const ctx = GqlExecutionContext.create(context);
    const request = ctx.getContext().req;  // Extract request
    
    // Check if user is authenticated
    if (!request.user) {
      throw new UnauthorizedException('Authentication required');
    }
    
    return true;  // Allow access
  }
}

// Usage in resolver
import { Resolver, Query, UseGuards } from '@nestjs/graphql';
import { AuthGuard } from './guards/auth.guard';

@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  @UseGuards(AuthGuard)  // Apply guard to this query
  async me(@Context() context): Promise<User> {
    return context.req.user;
  }
}
```

### **JWT Auth Guard for GraphQL**

```typescript
// guards/gql-auth.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class GqlAuthGuard extends AuthGuard('jwt') {
  // Override to extract request from GraphQL context
  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }

  // Optional: Custom error handling
  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('Invalid or missing token');
    }
    return user;
  }
}

// Usage
@Resolver(() => Post)
export class PostResolver {
  @Query(() => [Post])
  async posts(): Promise<Post[]> {
    return this.postService.findAll();  // Public
  }

  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)  // Protected
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }

  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async updatePost(
    @CurrentUser() user: User,
    @Args('id') id: string,
    @Args('input') input: UpdatePostInput,
  ): Promise<Post> {
    return this.postService.update(id, user, input);
  }
}
```

### **Apply Guards at Class Level**

```typescript
// Apply guard to all methods in resolver
@Resolver(() => User)
@UseGuards(GqlAuthGuard)  // All queries/mutations require auth
export class UserResolver {
  @Query(() => User)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;  // Protected by class-level guard
  }

  @Query(() => [User])
  async myFollowers(@CurrentUser() user: User): Promise<User[]> {
    return this.userService.getFollowers(user.id);  // Protected
  }

  @Mutation(() => User)
  async updateProfile(
    @CurrentUser() user: User,
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    return this.userService.update(user.id, input);  // Protected
  }
}
```

### **Multiple Guards (Chaining)**

```typescript
// Apply multiple guards (all must pass)
@Resolver(() => AdminPanel)
export class AdminResolver {
  @Query(() => [User])
  @UseGuards(GqlAuthGuard, RolesGuard)  // 1. Auth, then 2. Roles
  @Roles('admin')  // RolesGuard checks this metadata
  async allUsers(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, RolesGuard, EmailVerifiedGuard)  // 3 guards
  @Roles('admin')
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    await this.userService.delete(id);
    return true;
  }
}

// Guards execute in order:
// 1. GqlAuthGuard - checks authentication
// 2. RolesGuard - checks user role
// 3. EmailVerifiedGuard - checks email verification
// If any returns false or throws, execution stops
```

### **Global Guard with Public Routes**

```typescript
// decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// guards/gql-auth.guard.ts
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class GqlAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Check if route is marked as public
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),  // Method-level metadata
      context.getClass(),    // Class-level metadata
    ]);
    
    if (isPublic) {
      return true;  // Skip authentication
    }
    
    return super.canActivate(context);  // Run normal auth
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }
}

// app.module.ts - Apply guard globally
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: GqlAuthGuard,  // All resolvers protected by default
    },
  ],
})
export class AppModule {}

// Usage: Mark public routes explicitly
@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  @Public()  // Explicitly public
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Query(() => User)
  // No @Public() = requires authentication (global guard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }
}

@Resolver(() => Auth)
export class AuthResolver {
  @Mutation(() => AuthResponse)
  @Public()  // Login must be public
  async login(@Args('input') input: LoginInput): Promise<AuthResponse> {
    return this.authService.login(input);
  }

  @Mutation(() => AuthResponse)
  @Public()
  async register(@Args('input') input: RegisterInput): Promise<AuthResponse> {
    return this.authService.register(input);
  }
}
```

### **Custom Business Logic Guards**

```typescript
// guards/resource-owner.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  ForbiddenException,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { PostService } from '../post/post.service';

@Injectable()
export class ResourceOwnerGuard implements CanActivate {
  constructor(private postService: PostService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const ctx = GqlExecutionContext.create(context);
    const request = ctx.getContext().req;
    const args = ctx.getArgs();
    
    // Get current user
    const user = request.user;
    if (!user) {
      throw new ForbiddenException('Authentication required');
    }
    
    // Get resource ID from arguments
    const postId = args.id;
    if (!postId) {
      return true;  // No resource to check
    }
    
    // Check if user owns the resource
    const post = await this.postService.findById(postId);
    if (!post) {
      throw new ForbiddenException('Post not found');
    }
    
    if (post.authorId !== user.id && user.role !== 'admin') {
      throw new ForbiddenException('You do not own this resource');
    }
    
    return true;
  }
}

// Usage
@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard, ResourceOwnerGuard)  // Check auth + ownership
  async updatePost(
    @Args('id') id: string,
    @Args('input') input: UpdatePostInput,
  ): Promise<Post> {
    // User is authenticated and owns the post
    return this.postService.update(id, input);
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, ResourceOwnerGuard)
  async deletePost(@Args('id') id: string): Promise<boolean> {
    await this.postService.delete(id);
    return true;
  }
}
```

### **Throttle/Rate Limit Guard**

```typescript
// guards/throttle.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { Reflector } from '@nestjs/core';

@Injectable()
export class ThrottleGuard implements CanActivate {
  private requests = new Map<string, number[]>();

  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const ctx = GqlExecutionContext.create(context);
    const request = ctx.getContext().req;
    
    // Get rate limit config from metadata
    const limit = this.reflector.get<number>('throttle-limit', context.getHandler());
    const ttl = this.reflector.get<number>('throttle-ttl', context.getHandler());
    
    if (!limit || !ttl) {
      return true;  // No rate limit configured
    }
    
    // Track requests by IP or user ID
    const key = request.user?.id || request.ip;
    const now = Date.now();
    const timestamps = this.requests.get(key) || [];
    
    // Remove old timestamps
    const recentTimestamps = timestamps.filter(t => now - t < ttl);
    
    if (recentTimestamps.length >= limit) {
      throw new HttpException('Too many requests', HttpStatus.TOO_MANY_REQUESTS);
    }
    
    // Add current timestamp
    recentTimestamps.push(now);
    this.requests.set(key, recentTimestamps);
    
    return true;
  }
}

// Decorator for rate limit config
export const Throttle = (limit: number, ttl: number) => {
  return (target: any, propertyKey?: string, descriptor?: PropertyDescriptor) => {
    if (descriptor) {
      SetMetadata('throttle-limit', limit)(target, propertyKey, descriptor);
      SetMetadata('throttle-ttl', ttl)(target, propertyKey, descriptor);
    }
  };
};

// Usage
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => AuthResponse)
  @UseGuards(ThrottleGuard)
  @Throttle(5, 60000)  // 5 requests per 60 seconds
  async login(@Args('input') input: LoginInput): Promise<AuthResponse> {
    return this.authService.login(input);
  }
}
```

### **Complete Production Example**

```typescript
// guards/gql-auth.guard.ts
import {
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class GqlAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Check if @Public() decorator is present
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    return super.canActivate(context);
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }

  handleRequest(err: any, user: any, info: any, context: ExecutionContext) {
    if (err || !user) {
      throw err || new UnauthorizedException('Authentication required');
    }
    return user;
  }
}

// guards/roles.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  ForbiddenException,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { GqlExecutionContext } from '@nestjs/graphql';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from @Roles() decorator
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredRoles) {
      return true;  // No role requirement
    }
    
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    
    if (!user) {
      throw new ForbiddenException('User not found');
    }
    
    const hasRole = requiredRoles.some(role => user.roles?.includes(role));
    if (!hasRole) {
      throw new ForbiddenException(
        `Required roles: ${requiredRoles.join(', ')}`,
      );
    }
    
    return true;
  }
}

// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// app.module.ts
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: GqlAuthGuard,  // Global authentication
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard,  // Global authorization (runs after auth)
    },
  ],
})
export class AppModule {}

// Usage in resolvers
@Resolver(() => Post)
export class PostResolver {
  // Public query
  @Query(() => [Post])
  @Public()
  async posts(): Promise<Post[]> {
    return this.postService.findAll();
  }

  // Authenticated query
  @Query(() => [Post])
  async myPosts(@CurrentUser() user: User): Promise<Post[]> {
    return this.postService.findByAuthor(user.id);
  }

  // Role-based mutation
  @Mutation(() => Post)
  @Roles('admin', 'moderator')
  async featurePost(@Args('id') id: string): Promise<Post> {
    return this.postService.feature(id);
  }

  // Multiple guards with custom logic
  @Mutation(() => Boolean)
  @UseGuards(ResourceOwnerGuard)  // Additional guard
  async deletePost(@Args('id') id: string): Promise<boolean> {
    await this.postService.delete(id);
    return true;
  }
}

@Resolver(() => User)
export class AdminResolver {
  // Admin-only queries
  @Query(() => [User])
  @Roles('admin')
  async allUsers(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Mutation(() => Boolean)
  @Roles('admin')
  async banUser(@Args('id') id: string): Promise<boolean> {
    await this.userService.ban(id);
    return true;
  }
}
```

### **Testing Guards**

```typescript
// guards/__tests__/gql-auth.guard.spec.ts
import { ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { GqlAuthGuard } from '../gql-auth.guard';

describe('GqlAuthGuard', () => {
  let guard: GqlAuthGuard;

  beforeEach(() => {
    guard = new GqlAuthGuard(new Reflector());
  }

  it('should allow access when user is authenticated', () => {
    const mockContext = {
      getContext: () => ({
        req: { user: { id: '1', email: 'test@example.com' } },
      }),
    };

    jest.spyOn(GqlExecutionContext, 'create').mockReturnValue(mockContext as any);

    const result = guard.canActivate({} as ExecutionContext);
    expect(result).toBe(true);
  });

  it('should deny access when user is not authenticated', () => {
    const mockContext = {
      getContext: () => ({ req: {} }),
    };

    jest.spyOn(GqlExecutionContext, 'create').mockReturnValue(mockContext as any);

    expect(() => guard.canActivate({} as ExecutionContext)).toThrow(
      UnauthorizedException,
    );
  });
});
```

**Interview Tip**: **Guards** protect GraphQL resolvers by implementing `CanActivate`, **overriding `getRequest()`** to extract request from **`GqlExecutionContext`**. **Pattern**: `GqlExecutionContext.create(context).getContext().req` gets request from GraphQL context. **Key method**: `canActivate()` returns `true` (allow), `false` (deny), or throws exception. **Apply**: `@UseGuards()` on method/class, or globally with `APP_GUARD`. **Common guards**: `GqlAuthGuard extends AuthGuard('jwt')` for auth, `RolesGuard` for authorization. **Multiple guards**: execute in order, all must pass. **Global + public**: use `APP_GUARD` + `@Public()` decorator + `Reflector` to skip auth for specific routes. **Custom guards**: inject services, access args via `ctx.getArgs()`, implement business logic (ownership, rate limiting). **Key difference from REST**: must override `getRequest()` to work with GraphQL context instead of HTTP context.

</details>

43. How do you implement role-based authorization?

<details>
<summary><strong>Answer</strong></summary>

**Role-based authorization** uses **`@Roles()` decorator** to define required roles + **`RolesGuard`** to check if authenticated user has required role. Guard reads roles metadata from decorator, compares with `user.roles` from context, returns **`true`** (authorized) or throws **`ForbiddenException`**. Uses **`Reflector`** to read metadata, **`GqlExecutionContext`** to access user. Apply with **`@UseGuards(RolesGuard)`** or globally with `APP_GUARD`.

### **Basic Roles Setup**

```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// Usage:
// @Roles('admin')
// @Roles('admin', 'moderator')
// @Roles('user')  // Default role

// guards/roles.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  ForbiddenException,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { GqlExecutionContext } from '@nestjs/graphql';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from @Roles() decorator metadata
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),  // Method-level @Roles()
      context.getClass(),    // Class-level @Roles()
    ]);

    if (!requiredRoles) {
      return true;  // No roles required, allow access
    }

    // Extract user from GraphQL context
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;

    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Check if user has any of the required roles
    const hasRole = requiredRoles.some(role => user.roles?.includes(role));

    if (!hasRole) {
      throw new ForbiddenException(
        `Access denied. Required roles: ${requiredRoles.join(', ')}`,
      );
    }

    return true;
  }
}

// user.model.ts - User with roles
import { ObjectType, Field, ID } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;

  @Field(() => [String])
  roles: string[];  // ['user', 'admin', 'moderator']
}
```

### **Using Roles in Resolvers**

```typescript
// user.resolver.ts
import { Resolver, Query, Mutation, Args, UseGuards } from '@nestjs/graphql';
import { GqlAuthGuard } from '../auth/guards/gql-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { Roles } from '../auth/decorators/roles.decorator';
import { CurrentUser } from '../auth/decorators/current-user.decorator';
import { User } from './user.model';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Public query (no guards)
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Authenticated query (any logged-in user)
  @Query(() => User)
  @UseGuards(GqlAuthGuard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  // Admin-only query
  @Query(() => [User])
  @UseGuards(GqlAuthGuard, RolesGuard)  // Auth + Roles guards
  @Roles('admin')  // Only admins
  async allUsersWithDetails(): Promise<User[]> {
    return this.userService.findAllWithSensitiveData();
  }

  // Admin or moderator
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, RolesGuard)
  @Roles('admin', 'moderator')  // Either admin OR moderator
  async banUser(@Args('id') id: string): Promise<boolean> {
    await this.userService.ban(id);
    return true;
  }

  // Admin only
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, RolesGuard)
  @Roles('admin')
  async deleteUser(@Args('id') id: string): Promise<boolean> {
    await this.userService.delete(id);
    return true;
  }
}

// post.resolver.ts
@Resolver(() => Post)
export class PostResolver {
  // Public
  @Query(() => [Post])
  async posts(): Promise<Post[]> {
    return this.postService.findPublished();
  }

  // Authenticated user
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }

  // Admin or moderator
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard, RolesGuard)
  @Roles('admin', 'moderator')
  async featurePost(@Args('id') id: string): Promise<Post> {
    return this.postService.feature(id);
  }

  // Admin only
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, RolesGuard)
  @Roles('admin')
  async deleteAnyPost(@Args('id') id: string): Promise<boolean> {
    await this.postService.adminDelete(id);
    return true;
  }
}
```

### **Global Guards (All Resolvers Protected)**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { GqlAuthGuard } from './auth/guards/gql-auth.guard';
import { RolesGuard } from './auth/guards/roles.guard';

@Module({
  providers: [
    // Global authentication (runs first)
    {
      provide: APP_GUARD,
      useClass: GqlAuthGuard,
    },
    // Global authorization (runs second)
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}

// With global guards, use @Public() for public routes
import { Public } from './auth/decorators/public.decorator';

@Resolver(() => Post)
export class PostResolver {
  // Public (skip global guards)
  @Query(() => [Post])
  @Public()
  async posts(): Promise<Post[]> {
    return this.postService.findAll();
  }

  // Authenticated (global auth guard, no @Roles)
  @Mutation(() => Post)
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }

  // Admin only (global guards + @Roles)
  @Mutation(() => Boolean)
  @Roles('admin')
  async deletePost(@Args('id') id: string): Promise<boolean> {
    await this.postService.delete(id);
    return true;
  }
}
```

### **Enum for Roles (Type Safety)**

```typescript
// constants/roles.enum.ts
export enum Role {
  USER = 'user',
  MODERATOR = 'moderator',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
}

// decorators/roles.decorator.ts (updated)
import { SetMetadata } from '@nestjs/common';
import { Role } from '../constants/roles.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// Usage with type safety
import { Role } from '../constants/roles.enum';

@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  @UseGuards(GqlAuthGuard, RolesGuard)
  @Roles(Role.ADMIN, Role.SUPER_ADMIN)  // Type-safe
  async allUsers(): Promise<User[]> {
    return this.userService.findAll();
  }
}

// user.model.ts (updated)
import { Role } from '../constants/roles.enum';
import { registerEnumType } from '@nestjs/graphql';

registerEnumType(Role, {
  name: 'Role',
  description: 'User roles',
});

@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field(() => [Role])  // Type-safe roles in GraphQL
  roles: Role[];
}
```

### **Hierarchical Roles**

```typescript
// Role hierarchy: super_admin > admin > moderator > user
const ROLE_HIERARCHY: Record<string, number> = {
  user: 1,
  moderator: 2,
  admin: 3,
  super_admin: 4,
};

// guards/roles.guard.ts (updated with hierarchy)
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;

    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Get user's highest role level
    const userMaxLevel = Math.max(
      ...user.roles.map(role => ROLE_HIERARCHY[role] || 0),
    );

    // Get minimum required role level
    const requiredLevel = Math.min(
      ...requiredRoles.map(role => ROLE_HIERARCHY[role] || 0),
    );

    // User's level must be >= required level
    if (userMaxLevel >= requiredLevel) {
      return true;
    }

    throw new ForbiddenException(
      `Insufficient permissions. Required: ${requiredRoles.join(', ')}`,
    );
  }
}

// Now admin can access moderator endpoints
@Mutation(() => Boolean)
@Roles(Role.MODERATOR)  // Admin can also access (higher in hierarchy)
async hideComment(@Args('id') id: string): Promise<boolean> {
  await this.commentService.hide(id);
  return true;
}
```

### **Resource-Based Authorization (Ownership)**

```typescript
// Combine roles with resource ownership
// guards/resource-owner-or-admin.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  ForbiddenException,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { PostService } from '../post/post.service';
import { Role } from '../constants/roles.enum';

@Injectable()
export class ResourceOwnerOrAdminGuard implements CanActivate {
  constructor(private postService: PostService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    const args = ctx.getArgs();

    if (!user) {
      throw new ForbiddenException('Authentication required');
    }

    // Admins can access any resource
    if (user.roles.includes(Role.ADMIN)) {
      return true;
    }

    // Regular users can only access their own resources
    const postId = args.id;
    if (!postId) {
      return true;
    }

    const post = await this.postService.findById(postId);
    if (!post) {
      throw new ForbiddenException('Resource not found');
    }

    if (post.authorId !== user.id) {
      throw new ForbiddenException('You can only modify your own posts');
    }

    return true;
  }
}

// Usage
@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard, ResourceOwnerOrAdminGuard)
  async updatePost(
    @Args('id') id: string,
    @Args('input') input: UpdatePostInput,
  ): Promise<Post> {
    // Owner or admin can update
    return this.postService.update(id, input);
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard, ResourceOwnerOrAdminGuard)
  async deletePost(@Args('id') id: string): Promise<boolean> {
    // Owner or admin can delete
    await this.postService.delete(id);
    return true;
  }
}
```

### **Complete Production Example**

```typescript
// constants/roles.enum.ts
export enum Role {
  USER = 'user',
  MODERATOR = 'moderator',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
}

export const ROLE_HIERARCHY: Record<Role, number> = {
  [Role.USER]: 1,
  [Role.MODERATOR]: 2,
  [Role.ADMIN]: 3,
  [Role.SUPER_ADMIN]: 4,
};

// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Role } from '../constants/roles.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// guards/roles.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  ForbiddenException,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { GqlExecutionContext } from '@nestjs/graphql';
import { ROLES_KEY } from '../decorators/roles.decorator';
import { Role, ROLE_HIERARCHY } from '../constants/roles.enum';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles || requiredRoles.length === 0) {
      return true;  // No roles required
    }

    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;

    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Check hierarchical permissions
    const userMaxLevel = Math.max(
      ...user.roles.map((role: Role) => ROLE_HIERARCHY[role] || 0),
    );

    const requiredLevel = Math.min(
      ...requiredRoles.map(role => ROLE_HIERARCHY[role] || Infinity),
    );

    if (userMaxLevel >= requiredLevel) {
      return true;
    }

    throw new ForbiddenException(
      `Insufficient permissions. Required role: ${requiredRoles.join(' or ')}`,
    );
  }
}

// app.module.ts
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: GqlAuthGuard,  // Authentication
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard,  // Authorization
    },
  ],
})
export class AppModule {}

// admin.resolver.ts
import { Role } from './constants/roles.enum';

@Resolver(() => AdminPanel)
export class AdminResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  // Moderator and above
  @Query(() => [Report])
  @Roles(Role.MODERATOR)
  async reports(): Promise<Report[]> {
    return this.reportService.findAll();
  }

  @Mutation(() => Boolean)
  @Roles(Role.MODERATOR)
  async hideContent(@Args('id') id: string): Promise<boolean> {
    await this.contentService.hide(id);
    return true;
  }

  // Admin only
  @Query(() => [User])
  @Roles(Role.ADMIN)
  async allUsers(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Mutation(() => Boolean)
  @Roles(Role.ADMIN)
  async banUser(@Args('id') id: string): Promise<boolean> {
    await this.userService.ban(id);
    return true;
  }

  // Super admin only
  @Mutation(() => Boolean)
  @Roles(Role.SUPER_ADMIN)
  async deleteDatabase(): Promise<boolean> {
    // Dangerous operation
    await this.databaseService.wipeAll();
    return true;
  }

  @Mutation(() => User)
  @Roles(Role.SUPER_ADMIN)
  async promoteToAdmin(@Args('userId') userId: string): Promise<User> {
    return this.userService.addRole(userId, Role.ADMIN);
  }
}

// user.resolver.ts
@Resolver(() => User)
export class UserResolver {
  // Public
  @Query(() => User, { nullable: true })
  @Public()
  async user(@Args('id') id: string): Promise<User | null> {
    return this.userService.findById(id);
  }

  // Authenticated (any user)
  @Query(() => User)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  @Mutation(() => User)
  async updateProfile(
    @CurrentUser() user: User,
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    return this.userService.update(user.id, input);
  }

  // Admin or higher
  @Query(() => UserStatistics)
  @Roles(Role.ADMIN)
  async userStatistics(): Promise<UserStatistics> {
    return this.userService.getStatistics();
  }
}
```

### **Field-Level Authorization**

```typescript
// Restrict specific fields based on roles
import { Parent, ResolveField } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  // Sensitive field (admin only)
  @ResolveField(() => String, { nullable: true })
  email(
    @Parent() user: User,
    @CurrentUser() currentUser: User,
  ): string | null {
    // Users can see their own email
    if (currentUser.id === user.id) {
      return user.email;
    }
    
    // Admins can see all emails
    if (currentUser.roles.includes(Role.ADMIN)) {
      return user.email;
    }
    
    // Others cannot see email
    return null;
  }

  @ResolveField(() => [String], { nullable: true })
  roles(
    @Parent() user: User,
    @CurrentUser() currentUser: User,
  ): string[] | null {
    // Only admins can see roles
    if (currentUser.roles.includes(Role.ADMIN)) {
      return user.roles;
    }
    return null;
  }
}
```

**Interview Tip**: **Role-based authorization** uses **`@Roles()` decorator** (sets metadata) + **`RolesGuard`** (checks user roles). **Pattern**: guard reads required roles from decorator metadata via `Reflector`, extracts user from GraphQL context, compares `user.roles` with required roles. **Setup**: create `@Roles(...roles)` decorator with `SetMetadata()`, implement `RolesGuard` with `Reflector` + `GqlExecutionContext`, apply with `@UseGuards(GqlAuthGuard, RolesGuard)`. **Key**: `RolesGuard` runs AFTER `GqlAuthGuard` (needs authenticated user). **Flexible**: `@Roles('admin', 'moderator')` = admin OR moderator (any match). **Hierarchical**: assign numeric levels to roles, check if user level >= required level (admin automatically gets moderator access). **Global**: use `APP_GUARD` + `@Public()` for public routes. **Resource-based**: combine with ownership checks (owner or admin). **Type-safe**: use enum for roles instead of strings.

</details>

44. How do you access current user in resolvers using custom decorators?

<details>
<summary><strong>Answer</strong></summary>

**Custom decorators** extract data from **GraphQL context** using **`createParamDecorator()`** + **`GqlExecutionContext`**. Create decorator with `createParamDecorator()`, extract context with `GqlExecutionContext.create()`, access `req.user` from context, return user or specific property. Common decorators: **`@CurrentUser()`** (full user), **`@CurrentUser('id')`** (specific field), **`@CurrentUserId()`**, **`@IsAdmin()`**. Cleaner than `@Context()` in every resolver.

### **Basic @CurrentUser Decorator**

```typescript
// decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const CurrentUser = createParamDecorator(
  (data: unknown, context: ExecutionContext) => {
    // Convert to GraphQL execution context
    const ctx = GqlExecutionContext.create(context);
    
    // Extract request from GraphQL context
    const request = ctx.getContext().req;
    
    // Return authenticated user (set by AuthGuard)
    return request.user;
  },
);

// Usage in resolver
import { Resolver, Query, Mutation, Args, UseGuards } from '@nestjs/graphql';
import { GqlAuthGuard } from '../auth/guards/gql-auth.guard';
import { CurrentUser } from '../decorators/current-user.decorator';
import { User } from './user.model';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => User)
  @UseGuards(GqlAuthGuard)
  async me(@CurrentUser() user: User): Promise<User> {
    // 'user' is automatically extracted from context
    return user;
  }

  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async updateProfile(
    @CurrentUser() user: User,
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    return this.userService.update(user.id, input);
  }
}

// Compare with @Context() (verbose)
@Query(() => User)
@UseGuards(GqlAuthGuard)
async me(@Context() context: any): Promise<User> {
  return context.req.user;  // More verbose
}

// With @CurrentUser() (clean)
@Query(() => User)
@UseGuards(GqlAuthGuard)
async me(@CurrentUser() user: User): Promise<User> {
  return user;  // Clean and type-safe
}
```

### **Extract Specific User Property**

```typescript
// decorators/current-user.decorator.ts (enhanced)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

// Generic version: extract full user or specific property
export const CurrentUser = createParamDecorator(
  (data: string | undefined, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    
    // If no property specified, return full user
    if (!data) {
      return user;
    }
    
    // Return specific property
    return user?.[data];
  },
);

// Usage examples
@Resolver(() => Post)
export class PostResolver {
  constructor(private postService: PostService) {}

  // Get full user object
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async createPost(
    @CurrentUser() user: User,  // Full user
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }

  // Get only user ID
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deletePost(
    @CurrentUser('id') userId: string,  // Only ID
    @Args('id') postId: string,
  ): Promise<boolean> {
    await this.postService.delete(postId, userId);
    return true;
  }

  // Get user email
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async subscribeToNewsletter(
    @CurrentUser('email') email: string,  // Only email
  ): Promise<boolean> {
    await this.newsletterService.subscribe(email);
    return true;
  }

  // Get user roles
  @Query(() => [String])
  @UseGuards(GqlAuthGuard)
  async myPermissions(
    @CurrentUser('roles') roles: string[],  // Only roles
  ): Promise<string[]> {
    return roles;
  }
}
```

### **Type-Safe CurrentUser Decorator**

```typescript
// decorators/current-user.decorator.ts (type-safe)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { User } from '../user/user.model';

// Overloaded type signatures for type safety
export const CurrentUser = createParamDecorator(
  <K extends keyof User>(data: K | undefined, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user as User;
    
    return data ? user?.[data] : user;
  },
);

// Now TypeScript knows the return type!
@Mutation(() => Post)
@UseGuards(GqlAuthGuard)
async createPost(
  @CurrentUser('id') userId: string,  // TypeScript knows this is string
  @CurrentUser('email') email: string,  // TypeScript knows this is string
  @CurrentUser('roles') roles: string[],  // TypeScript knows this is string[]
  @CurrentUser() user: User,  // TypeScript knows this is User
  @Args('input') input: CreatePostInput,
): Promise<Post> {
  // All parameters are properly typed
  return this.postService.create(user, input);
}
```

### **Specialized Decorators**

```typescript
// decorators/current-user-id.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const CurrentUserId = createParamDecorator(
  (data: unknown, context: ExecutionContext): string => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user?.id;
  },
);

// decorators/is-admin.decorator.ts
export const IsAdmin = createParamDecorator(
  (data: unknown, context: ExecutionContext): boolean => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    return user?.roles?.includes('admin') || false;
  },
);

// decorators/current-user-roles.decorator.ts
export const CurrentUserRoles = createParamDecorator(
  (data: unknown, context: ExecutionContext): string[] => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user?.roles || [];
  },
);

// Usage
@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async createPost(
    @CurrentUserId() userId: string,  // Just the ID
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(userId, input);
  }

  @Query(() => AdminStats)
  @UseGuards(GqlAuthGuard)
  async adminStats(
    @IsAdmin() isAdmin: boolean,  // Check if admin
  ): Promise<AdminStats | null> {
    if (!isAdmin) {
      throw new ForbiddenException('Admin only');
    }
    return this.statsService.getAdminStats();
  }

  @Query(() => [String])
  @UseGuards(GqlAuthGuard)
  async myPermissions(
    @CurrentUserRoles() roles: string[],  // Just roles
  ): Promise<string[]> {
    return this.permissionService.getPermissions(roles);
  }
}
```

### **Decorator with Optional User**

```typescript
// decorators/optional-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const OptionalUser = createParamDecorator(
  (data: unknown, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    // Return user if exists, undefined otherwise
    return ctx.getContext().req.user || null;
  },
);

// Usage: No guard needed, user might not be authenticated
@Resolver(() => Post)
export class PostResolver {
  @Query(() => [Post])
  async posts(
    @OptionalUser() user: User | null,  // Optional
  ): Promise<Post[]> {
    // Return personalized feed if authenticated
    if (user) {
      return this.postService.getPersonalizedFeed(user.id);
    }
    
    // Return public feed if not authenticated
    return this.postService.getPublicFeed();
  }

  @Query(() => Post, { nullable: true })
  async post(
    @Args('id') id: string,
    @OptionalUser() user: User | null,
  ): Promise<Post | null> {
    const post = await this.postService.findById(id);
    
    // Include private data if user is author
    if (user && post.authorId === user.id) {
      return this.postService.withPrivateData(post);
    }
    
    return post;
  }
}
```

### **Decorator with Validation**

```typescript
// decorators/validated-user.decorator.ts
import {
  createParamDecorator,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const ValidatedUser = createParamDecorator(
  (data: unknown, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user;
    
    // Throw if no user (extra validation)
    if (!user) {
      throw new UnauthorizedException('User not authenticated');
    }
    
    // Throw if user is not active
    if (!user.isActive) {
      throw new UnauthorizedException('Account is deactivated');
    }
    
    // Throw if email is not verified
    if (!user.emailVerified) {
      throw new UnauthorizedException('Email not verified');
    }
    
    return user;
  },
);

// Usage: Automatic validation
@Mutation(() => Post)
@UseGuards(GqlAuthGuard)
async createPost(
  @ValidatedUser() user: User,  // Guaranteed to be active and verified
  @Args('input') input: CreatePostInput,
): Promise<Post> {
  return this.postService.create(user, input);
}
```

### **Complete Production Example**

```typescript
// decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { User } from '../user/user.model';

/**
 * Extract current authenticated user from GraphQL context.
 * Use without parameter to get full user object.
 * Use with parameter to get specific user property.
 * 
 * @example
 * @CurrentUser() user: User                 // Full user
 * @CurrentUser('id') userId: string         // Just ID
 * @CurrentUser('email') email: string       // Just email
 */
export const CurrentUser = createParamDecorator(
  <K extends keyof User>(
    data: K | undefined,
    context: ExecutionContext,
  ): User | User[K] | undefined => {
    const ctx = GqlExecutionContext.create(context);
    const user = ctx.getContext().req.user as User;
    
    if (!user) {
      return undefined;
    }
    
    return data ? user[data] : user;
  },
);

// decorators/current-user-id.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

/**
 * Extract current user's ID from GraphQL context.
 * Shorthand for @CurrentUser('id')
 */
export const CurrentUserId = createParamDecorator(
  (data: unknown, context: ExecutionContext): string | undefined => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user?.id;
  },
);

// decorators/optional-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { User } from '../user/user.model';

/**
 * Extract user from context if authenticated, null otherwise.
 * Use for queries that work differently for authenticated users.
 * Does not require AuthGuard.
 */
export const OptionalUser = createParamDecorator(
  (data: unknown, context: ExecutionContext): User | null => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user || null;
  },
);

// user.resolver.ts
import { Resolver, Query, Mutation, Args, UseGuards } from '@nestjs/graphql';
import { GqlAuthGuard } from '../auth/guards/gql-auth.guard';
import {
  CurrentUser,
  CurrentUserId,
  OptionalUser,
} from '../decorators';
import { User } from './user.model';
import { UserService } from './user.service';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  // Public query with optional user context
  @Query(() => [User])
  async users(
    @OptionalUser() currentUser: User | null,
  ): Promise<User[]> {
    // Personalize if authenticated
    if (currentUser) {
      return this.userService.getRecommended(currentUser.id);
    }
    return this.userService.findAll();
  }

  // Protected: Get current user
  @Query(() => User)
  @UseGuards(GqlAuthGuard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  // Protected: Update profile
  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async updateProfile(
    @CurrentUserId() userId: string,  // Just ID
    @Args('input') input: UpdateProfileInput,
  ): Promise<User> {
    return this.userService.update(userId, input);
  }

  // Protected: Delete account
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deleteAccount(
    @CurrentUser('id') userId: string,  // Specific property
  ): Promise<boolean> {
    await this.userService.delete(userId);
    return true;
  }

  // Protected: Check permissions
  @Query(() => [String])
  @UseGuards(GqlAuthGuard)
  async myPermissions(
    @CurrentUser('roles') roles: string[],
  ): Promise<string[]> {
    return this.permissionService.getPermissions(roles);
  }
}

// post.resolver.ts
@Resolver(() => Post)
export class PostResolver {
  constructor(private postService: PostService) {}

  // Public with optional personalization
  @Query(() => [Post])
  async posts(
    @OptionalUser() user: User | null,
    @Args('limit', { type: () => Int, defaultValue: 10 }) limit: number,
  ): Promise<Post[]> {
    if (user) {
      return this.postService.getPersonalizedFeed(user.id, limit);
    }
    return this.postService.getPublicFeed(limit);
  }

  // Protected: Create post
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    return this.postService.create(user, input);
  }

  // Protected: Like post
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async likePost(
    @CurrentUserId() userId: string,
    @Args('postId') postId: string,
  ): Promise<Post> {
    return this.postService.like(postId, userId);
  }

  // Protected: My posts
  @Query(() => [Post])
  @UseGuards(GqlAuthGuard)
  async myPosts(
    @CurrentUser('id') userId: string,
  ): Promise<Post[]> {
    return this.postService.findByAuthor(userId);
  }
}

// comment.resolver.ts
@Resolver(() => Comment)
export class CommentResolver {
  @Mutation(() => Comment)
  @UseGuards(GqlAuthGuard)
  async createComment(
    @CurrentUserId() authorId: string,  // Cleaner than passing full user
    @Args('postId') postId: string,
    @Args('text') text: string,
  ): Promise<Comment> {
    return this.commentService.create({
      postId,
      authorId,
      text,
    });
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deleteComment(
    @CurrentUserId() userId: string,
    @Args('id') id: string,
  ): Promise<boolean> {
    await this.commentService.delete(id, userId);
    return true;
  }
}
```

### **Testing Custom Decorators**

```typescript
// decorators/__tests__/current-user.decorator.spec.ts
import { ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { CurrentUser } from '../current-user.decorator';

describe('CurrentUser Decorator', () => {
  it('should extract user from context', () => {
    const mockUser = { id: '1', email: 'test@example.com', roles: ['user'] };
    const mockContext = {
      getContext: () => ({ req: { user: mockUser } }),
    } as any;

    jest.spyOn(GqlExecutionContext, 'create').mockReturnValue(mockContext);

    const decorator = CurrentUser();
    const result = decorator(undefined, {} as ExecutionContext);

    expect(result).toEqual(mockUser);
  });

  it('should extract specific user property', () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    const mockContext = {
      getContext: () => ({ req: { user: mockUser } }),
    } as any;

    jest.spyOn(GqlExecutionContext, 'create').mockReturnValue(mockContext);

    const decorator = CurrentUser();
    const result = decorator('email', {} as ExecutionContext);

    expect(result).toBe('test@example.com');
  });
});
```

**Interview Tip**: **Custom decorators** extract user from GraphQL context using **`createParamDecorator()`** + **`GqlExecutionContext`**. **Pattern**: `GqlExecutionContext.create(context).getContext().req.user`. **Basic**: `@CurrentUser()` extracts full user (cleaner than `@Context()`). **Enhanced**: `@CurrentUser('id')` extracts specific property (e.g., just ID). **Specialized**: create `@CurrentUserId()`, `@CurrentUserRoles()`, `@IsAdmin()` for common use cases. **Optional**: `@OptionalUser()` returns user or null (no guard needed). **Type-safe**: use generics with `keyof User` for TypeScript type inference. **Validation**: add validation logic in decorator (check `isActive`, `emailVerified`). **Benefits**: cleaner resolver signatures, reusable logic, type safety, better readability. **Always** use custom decorators instead of `@Context()` for accessing user in resolvers.

</details>

## Error Handling

45. How do you handle errors in GraphQL?

<details>
<summary><strong>Answer</strong></summary>

**Error handling** in GraphQL uses **standard exceptions** (`BadRequestException`, `UnauthorizedException`, etc.) that are automatically formatted by Apollo. Throw exceptions in resolvers/services, Apollo catches and formats as **`errors`** array in response with **`message`**, **`extensions.code`**, **`path`**, **`locations`**. Custom errors with **`GraphQLError`** or **exception filters**. **Data + errors** can coexist (partial responses). Configure **`formatError`** in GraphQL module for custom formatting.

### **Basic Error Handling**

```typescript
// Throw standard NestJS exceptions in resolvers
import {
  BadRequestException,
  NotFoundException,
  UnauthorizedException,
  ForbiddenException,
  InternalServerErrorException,
} from '@nestjs/common';
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';

@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    const user = await this.userService.findById(id);
    
    // Throw exception if not found
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    // Validation error
    if (!input.email.includes('@')) {
      throw new BadRequestException('Invalid email format');
    }
    
    // Check if email exists
    const existingUser = await this.userService.findByEmail(input.email);
    if (existingUser) {
      throw new BadRequestException('Email already registered');
    }
    
    return this.userService.create(input);
  }

  @Mutation(() => Boolean)
  async deleteUser(
    @Args('id') id: string,
    @CurrentUser() currentUser: User,
  ): Promise<boolean> {
    // Authorization check
    if (currentUser.id !== id && !currentUser.roles.includes('admin')) {
      throw new ForbiddenException('You can only delete your own account');
    }
    
    await this.userService.delete(id);
    return true;
  }
}

// GraphQL error response format:
// {
//   "errors": [
//     {
//       "message": "User with ID 123 not found",
//       "extensions": {
//         "code": "NOT_FOUND",
//         "statusCode": 404
//       },
//       "path": ["user"],
//       "locations": [{ "line": 2, "column": 3 }]
//     }
//   ],
//   "data": null
// }
```

### **Error Types and Codes**

```typescript
import {
  BadRequestException,         // 400 - BAD_REQUEST
  UnauthorizedException,       // 401 - UNAUTHORIZED
  ForbiddenException,          // 403 - FORBIDDEN
  NotFoundException,           // 404 - NOT_FOUND
  ConflictException,           // 409 - CONFLICT
  UnprocessableEntityException,// 422 - UNPROCESSABLE_ENTITY
  InternalServerErrorException,// 500 - INTERNAL_SERVER_ERROR
} from '@nestjs/common';

@Resolver(() => Post)
export class PostResolver {
  // 400 Bad Request
  @Mutation(() => Post)
  async createPost(@Args('input') input: CreatePostInput): Promise<Post> {
    if (!input.title) {
      throw new BadRequestException('Title is required');
    }
    return this.postService.create(input);
  }

  // 401 Unauthorized
  @Query(() => [Post])
  async myPosts(@CurrentUser() user: User | undefined): Promise<Post[]> {
    if (!user) {
      throw new UnauthorizedException('Authentication required');
    }
    return this.postService.findByAuthor(user.id);
  }

  // 403 Forbidden
  @Mutation(() => Boolean)
  async deletePost(
    @Args('id') id: string,
    @CurrentUser() user: User,
  ): Promise<boolean> {
    const post = await this.postService.findById(id);
    if (post.authorId !== user.id) {
      throw new ForbiddenException('You can only delete your own posts');
    }
    await this.postService.delete(id);
    return true;
  }

  // 404 Not Found
  @Query(() => Post)
  async post(@Args('id') id: string): Promise<Post> {
    const post = await this.postService.findById(id);
    if (!post) {
      throw new NotFoundException('Post not found');
    }
    return post;
  }

  // 409 Conflict
  @Mutation(() => Post)
  async publishPost(@Args('id') id: string): Promise<Post> {
    const post = await this.postService.findById(id);
    if (post.status === 'published') {
      throw new ConflictException('Post is already published');
    }
    return this.postService.publish(id);
  }

  // 500 Internal Server Error
  @Query(() => [Post])
  async posts(): Promise<Post[]> {
    try {
      return await this.postService.findAll();
    } catch (error) {
      console.error('Database error:', error);
      throw new InternalServerErrorException('Failed to fetch posts');
    }
  }
}
```

### **Partial Responses (Data + Errors)**

```typescript
// GraphQL can return both data and errors
@Resolver(() => User)
export class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return this.userService.findAll();
  }

  // Field resolver with error handling
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    try {
      return await this.postService.findByUserId(user.id);
    } catch (error) {
      // Log error but don't crash entire query
      console.error(`Failed to load posts for user ${user.id}:`, error);
      return [];  // Return empty array instead of throwing
    }
  }
}

// Query:
// query {
//   users {
//     id
//     name
//     posts {  // If this fails, other users still returned
//       id
//       title
//     }
//   }
// }

// Response with partial error:
// {
//   "data": {
//     "users": [
//       { "id": "1", "name": "John", "posts": [] },  // Failed but returned empty
//       { "id": "2", "name": "Jane", "posts": [...] }  // Success
//     ]
//   },
//   "errors": [
//     {
//       "message": "Failed to load posts",
//       "path": ["users", 0, "posts"]
//     }
//   ]
// }
```

### **Try-Catch in Resolvers**

```typescript
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
  ): Promise<User> {
    try {
      return await this.userService.update(id, input);
    } catch (error) {
      // Log original error
      console.error('Update user failed:', error);
      
      // Rethrow as user-friendly error
      if (error.code === '23505') {  // PostgreSQL unique violation
        throw new BadRequestException('Email already in use');
      }
      
      if (error.name === 'QueryFailedError') {
        throw new BadRequestException('Invalid update data');
      }
      
      // Generic error for unknown issues
      throw new InternalServerErrorException('Failed to update user');
    }
  }

  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    try {
      const user = await this.userService.findById(id);
      if (!user) {
        throw new NotFoundException(`User ${id} not found`);
      }
      return user;
    } catch (error) {
      // Re-throw known errors
      if (error instanceof NotFoundException) {
        throw error;
      }
      
      // Wrap unknown errors
      throw new InternalServerErrorException('Failed to fetch user');
    }
  }
}
```

### **Service-Level Error Handling**

```typescript
// user.service.ts - Throw errors from services
import { Injectable, NotFoundException, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async findById(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      // Throw from service
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }

  async create(input: CreateUserInput): Promise<User> {
    // Check if email exists
    const existingUser = await this.userRepository.findOne({
      where: { email: input.email },
    });
    
    if (existingUser) {
      throw new BadRequestException('Email already registered');
    }
    
    try {
      const user = this.userRepository.create(input);
      return await this.userRepository.save(user);
    } catch (error) {
      console.error('Failed to create user:', error);
      throw new InternalServerErrorException('Failed to create user');
    }
  }

  async delete(id: string): Promise<void> {
    const user = await this.findById(id);  // Throws NotFoundException if not found
    
    try {
      await this.userRepository.remove(user);
    } catch (error) {
      console.error('Failed to delete user:', error);
      throw new InternalServerErrorException('Failed to delete user');
    }
  }
}

// Resolver just calls service (errors propagate automatically)
@Resolver(() => User)
export class UserResolver {
  constructor(private userService: UserService) {}

  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    return this.userService.findById(id);  // Throws NotFoundException
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    return this.userService.create(input);  // Throws BadRequestException
  }
}
```

### **Async Error Handling**

```typescript
// Handle errors in async operations
@Resolver(() => Report)
export class ReportResolver {
  constructor(
    private reportService: ReportService,
    private emailService: EmailService,
  ) {}

  @Mutation(() => Report)
  async createReport(
    @Args('input') input: CreateReportInput,
    @CurrentUser() user: User,
  ): Promise<Report> {
    try {
      // Create report
      const report = await this.reportService.create(input, user.id);
      
      // Send notification (fire and forget, catch errors)
      this.emailService.sendReportNotification(report)
        .catch(error => {
          console.error('Failed to send notification:', error);
          // Don't fail the mutation if email fails
        });
      
      return report;
    } catch (error) {
      console.error('Failed to create report:', error);
      throw new InternalServerErrorException('Failed to create report');
    }
  }
}
```

### **Multiple Errors**

```typescript
// GraphQL can return multiple errors
import { GraphQLError } from 'graphql';

@Resolver(() => User)
export class UserResolver {
  @Mutation(() => [User])
  async createUsers(
    @Args('inputs', { type: () => [CreateUserInput] })
    inputs: CreateUserInput[],
  ): Promise<User[]> {
    const users: User[] = [];
    const errors: GraphQLError[] = [];

    for (const input of inputs) {
      try {
        const user = await this.userService.create(input);
        users.push(user);
      } catch (error) {
        // Collect errors instead of throwing immediately
        errors.push(
          new GraphQLError(`Failed to create user ${input.email}: ${error.message}`, {
            extensions: { code: 'CREATE_FAILED', email: input.email },
          }),
        );
      }
    }

    // If all failed, throw
    if (users.length === 0 && errors.length > 0) {
      throw errors[0];  // Throw first error
    }

    return users;  // Return successful users (errors in response)
  }
}
```

### **Complete Production Example**

```typescript
// user.service.ts
import {
  Injectable,
  NotFoundException,
  BadRequestException,
  InternalServerErrorException,
} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async findById(id: string): Promise<User> {
    try {
      const user = await this.userRepository.findOne({ where: { id } });
      
      if (!user) {
        throw new NotFoundException(`User with ID ${id} not found`);
      }
      
      return user;
    } catch (error) {
      if (error instanceof NotFoundException) {
        throw error;
      }
      
      console.error('Database error in findById:', error);
      throw new InternalServerErrorException('Failed to fetch user');
    }
  }

  async create(input: CreateUserInput): Promise<User> {
    // Validate input
    if (!input.email || !input.password) {
      throw new BadRequestException('Email and password are required');
    }

    // Check for existing user
    const existingUser = await this.userRepository.findOne({
      where: { email: input.email },
    });

    if (existingUser) {
      throw new BadRequestException(`Email ${input.email} is already registered`);
    }

    try {
      const user = this.userRepository.create(input);
      return await this.userRepository.save(user);
    } catch (error) {
      console.error('Failed to create user:', error);
      
      // Handle specific database errors
      if (error.code === '23505') {  // Unique constraint violation
        throw new BadRequestException('Email already in use');
      }
      
      throw new InternalServerErrorException('Failed to create user');
    }
  }

  async update(id: string, input: UpdateUserInput): Promise<User> {
    const user = await this.findById(id);  // Throws if not found

    // Validate email change
    if (input.email && input.email !== user.email) {
      const existingUser = await this.userRepository.findOne({
        where: { email: input.email },
      });
      
      if (existingUser) {
        throw new BadRequestException('Email already in use');
      }
    }

    try {
      Object.assign(user, input);
      return await this.userRepository.save(user);
    } catch (error) {
      console.error('Failed to update user:', error);
      throw new InternalServerErrorException('Failed to update user');
    }
  }

  async delete(id: string): Promise<void> {
    const user = await this.findById(id);

    try {
      await this.userRepository.remove(user);
    } catch (error) {
      console.error('Failed to delete user:', error);
      throw new InternalServerErrorException('Failed to delete user');
    }
  }
}

// user.resolver.ts
import {
  Resolver,
  Query,
  Mutation,
  Args,
  ResolveField,
  Parent,
} from '@nestjs/graphql';
import {
  UseGuards,
  ForbiddenException,
  BadRequestException,
} from '@nestjs/common';
import { GqlAuthGuard } from '../auth/guards/gql-auth.guard';
import { CurrentUser } from '../decorators/current-user.decorator';
import { User } from './user.model';
import { UserService } from './user.service';
import { Post } from '../post/post.model';
import { PostService } from '../post/post.service';

@Resolver(() => User)
export class UserResolver {
  constructor(
    private userService: UserService,
    private postService: PostService,
  ) {}

  // Public query with error handling
  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string): Promise<User | null> {
    try {
      return await this.userService.findById(id);
    } catch (error) {
      if (error instanceof NotFoundException) {
        return null;  // Return null instead of throwing (nullable field)
      }
      throw error;  // Re-throw other errors
    }
  }

  // Protected query
  @Query(() => User)
  @UseGuards(GqlAuthGuard)
  async me(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  // Mutation with validation
  @Mutation(() => User)
  async createUser(
    @Args('input') input: CreateUserInput,
  ): Promise<User> {
    // Service handles validation and throws exceptions
    return this.userService.create(input);
  }

  // Protected mutation with authorization
  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async updateUser(
    @Args('id') id: string,
    @Args('input') input: UpdateUserInput,
    @CurrentUser() currentUser: User,
  ): Promise<User> {
    // Authorization check
    if (currentUser.id !== id && !currentUser.roles.includes('admin')) {
      throw new ForbiddenException('You can only update your own profile');
    }

    return this.userService.update(id, input);
  }

  // Protected deletion
  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deleteUser(
    @Args('id') id: string,
    @CurrentUser() currentUser: User,
  ): Promise<boolean> {
    if (currentUser.id !== id && !currentUser.roles.includes('admin')) {
      throw new ForbiddenException('You can only delete your own account');
    }

    await this.userService.delete(id);
    return true;
  }

  // Field resolver with graceful error handling
  @ResolveField(() => [Post])
  async posts(@Parent() user: User): Promise<Post[]> {
    try {
      return await this.postService.findByAuthor(user.id);
    } catch (error) {
      console.error(`Failed to load posts for user ${user.id}:`, error);
      return [];  // Return empty array instead of failing entire query
    }
  }
}
```

**Interview Tip**: **GraphQL errors** handled by throwing **standard NestJS exceptions** (`NotFoundException`, `BadRequestException`, etc.), automatically formatted by Apollo into `errors` array. **Response format**: `{ errors: [{ message, extensions: { code }, path, locations }], data }`. **Data + errors**: can coexist (partial responses). **Best practice**: throw from services, let exceptions propagate to resolvers, Apollo handles formatting. **Field resolvers**: catch errors and return fallback (empty array) to avoid failing entire query. **Error codes**: `BAD_REQUEST` (400), `UNAUTHORIZED` (401), `FORBIDDEN` (403), `NOT_FOUND` (404), `INTERNAL_SERVER_ERROR` (500). **Try-catch**: use for database errors, transform to user-friendly messages. **Validation**: use ValidationPipe for input validation (automatic errors). **Custom formatting**: use `formatError` option in GraphQL module (next question).

</details>

46. What is the error format in GraphQL responses?

<details>
<summary><strong>Answer</strong></summary>

**GraphQL error format** includes **`message`** (error text), **`extensions`** (code, status, details), **`path`** (field path in query), **`locations`** (line/column in query). Errors in **`errors`** array, data in **`data`** field (can both exist). Customize with **`formatError`** in `GraphQLModule.forRoot()`. **Extensions** contain error code (`BAD_REQUEST`, `NOT_FOUND`, etc.), status code, validation errors, stack trace (dev only).

### **Standard GraphQL Error Format**

```typescript
// Standard error response structure
{
  "errors": [
    {
      // Error message
      "message": "User with ID 123 not found",
      
      // Additional error metadata
      "extensions": {
        "code": "NOT_FOUND",        // Error code (BAD_REQUEST, UNAUTHORIZED, etc.)
        "statusCode": 404,           // HTTP status code
        "exception": {               // Exception details (dev only)
          "stacktrace": [...]
        }
      },
      
      // Path to field that caused error
      "path": ["user"],
      
      // Location in GraphQL query
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ]
    }
  ],
  
  // Data (can be null or partial)
  "data": null
}

// Multiple errors example
{
  "errors": [
    {
      "message": "Email is required",
      "extensions": { "code": "BAD_REQUEST" },
      "path": ["createUser"]
    },
    {
      "message": "Password must be at least 8 characters",
      "extensions": { "code": "BAD_REQUEST" },
      "path": ["createUser"]
    }
  ],
  "data": { "createUser": null }
}
```

### **Default Error Codes**

```typescript
// NestJS exception → GraphQL error code mapping
import {
  BadRequestException,         // BAD_REQUEST
  UnauthorizedException,       // UNAUTHENTICATED
  ForbiddenException,          // FORBIDDEN
  NotFoundException,           // NOT_FOUND
  ConflictException,           // CONFLICT
  InternalServerErrorException,// INTERNAL_SERVER_ERROR
} from '@nestjs/common';

// Examples of error responses

// 1. NotFoundException → NOT_FOUND
throw new NotFoundException('User not found');
// Response:
// {
//   "errors": [{
//     "message": "User not found",
//     "extensions": {
//       "code": "NOT_FOUND",
//       "statusCode": 404
//     }
//   }]
// }

// 2. BadRequestException → BAD_REQUEST  
throw new BadRequestException('Invalid input');
// Response:
// {
//   "errors": [{
//     "message": "Invalid input",
//     "extensions": {
//       "code": "BAD_REQUEST",
//       "statusCode": 400
//     }
//   }]
// }

// 3. UnauthorizedException → UNAUTHENTICATED
throw new UnauthorizedException('Authentication required');
// Response:
// {
//   "errors": [{
//     "message": "Authentication required",
//     "extensions": {
//       "code": "UNAUTHENTICATED",
//       "statusCode": 401
//     }
//   }]
// }

// 4. ForbiddenException → FORBIDDEN
throw new ForbiddenException('Access denied');
// Response:
// {
//   "errors": [{
//     "message": "Access denied",
//     "extensions": {
//       "code": "FORBIDDEN",
//       "statusCode": 403
//     }
//   }]
// }
```

### **Customize Error Format**

```typescript
// app.module.ts - Custom error formatting
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { GraphQLError, GraphQLFormattedError } from 'graphql';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      
      // Custom error formatter
      formatError: (error: GraphQLError): GraphQLFormattedError => {
        const originalError = error.extensions?.originalError as any;
        const statusCode = originalError?.statusCode || 500;
        
        // Customize error format
        return {
          message: error.message,
          code: error.extensions?.code || 'INTERNAL_SERVER_ERROR',
          statusCode: statusCode,
          path: error.path,
          locations: error.locations,
          // Add custom fields
          timestamp: new Date().toISOString(),
          // Hide stacktrace in production
          ...(process.env.NODE_ENV === 'development' && {
            stacktrace: error.extensions?.exception?.stacktrace,
          }),
        };
      },
    }),
  ],
})
export class AppModule {}

// Custom error response:
// {
//   "errors": [{
//     "message": "User not found",
//     "code": "NOT_FOUND",
//     "statusCode": 404,
//     "path": ["user"],
//     "locations": [{ "line": 2, "column": 3 }],
//     "timestamp": "2024-01-15T10:30:00.000Z"
//   }]
// }
```

### **Hide Sensitive Information (Production)**

```typescript
// app.module.ts - Production-safe error formatting
import { ConfigService } from '@nestjs/config';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        
        return {
          autoSchemaFile: true,
          
          formatError: (error: GraphQLError) => {
            // In production, hide sensitive details
            if (isProduction) {
              // Don't expose internal errors
              if (error.extensions?.code === 'INTERNAL_SERVER_ERROR') {
                return {
                  message: 'An internal error occurred',
                  code: 'INTERNAL_SERVER_ERROR',
                  path: error.path,
                };
              }
              
              // Remove stacktrace
              const { exception, ...extensions } = error.extensions || {};
              
              return {
                message: error.message,
                extensions,
                path: error.path,
              };
            }
            
            // In development, show everything
            return {
              message: error.message,
              extensions: error.extensions,
              path: error.path,
              locations: error.locations,
            };
          },
        };
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### **Validation Errors Format**

```typescript
// ValidationPipe errors are automatically formatted
import { ValidationPipe } from '@nestjs/common';

// With ValidationPipe enabled
app.useGlobalPipes(new ValidationPipe());

// Invalid input mutation:
mutation {
  createUser(input: {
    name: "A"                    # Too short
    email: "invalid"            # Invalid format
    password: "short"           # Too short
  }) {
    id
  }
}

// Error response with validation details:
{
  "errors": [
    {
      "message": "Bad Request Exception",
      "extensions": {
        "code": "BAD_REQUEST",
        "statusCode": 400,
        "validationErrors": [
          {
            "property": "name",
            "constraints": {
              "minLength": "Name must be at least 2 characters"
            }
          },
          {
            "property": "email",
            "constraints": {
              "isEmail": "Invalid email format"
            }
          },
          {
            "property": "password",
            "constraints": {
              "minLength": "Password must be at least 8 characters"
            }
          }
        ]
      },
      "path": ["createUser"]
    }
  ],
  "data": { "createUser": null }
}
```

### **Custom Error Format with Details**

```typescript
// app.module.ts - Detailed error format
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      
      formatError: (error: GraphQLError) => {
        const originalError = error.extensions?.originalError as any;
        const exception = error.extensions?.exception as any;
        
        // Base error object
        const formattedError: any = {
          message: error.message,
          timestamp: new Date().toISOString(),
        };
        
        // Add error code
        formattedError.code = error.extensions?.code || 'INTERNAL_SERVER_ERROR';
        
        // Add status code
        formattedError.statusCode = originalError?.statusCode || 500;
        
        // Add path (field in query that caused error)
        if (error.path) {
          formattedError.path = error.path;
        }
        
        // Add locations (line/column in query)
        if (error.locations) {
          formattedError.locations = error.locations;
        }
        
        // Add validation errors
        if (originalError?.message?.includes('Validation')) {
          formattedError.validationErrors = originalError.validationErrors;
        }
        
        // Add stacktrace in development
        if (process.env.NODE_ENV === 'development' && exception?.stacktrace) {
          formattedError.stacktrace = exception.stacktrace;
        }
        
        return formattedError;
      },
    }),
  ],
})
export class AppModule {}
```

### **Partial Data with Errors**

```typescript
// Query with multiple fields
query {
  user(id: "1") {    # Success
    id
    name
    posts {          # Fails
      id
      title
    }
  }
  
  allUsers {         # Success
    id
    name
  }
}

// Response with partial data + errors
{
  "data": {
    "user": {
      "id": "1",
      "name": "John Doe",
      "posts": null    // Field failed
    },
    "allUsers": [      // Field succeeded
      { "id": "1", "name": "John" },
      { "id": "2", "name": "Jane" }
    ]
  },
  "errors": [
    {
      "message": "Failed to load posts",
      "extensions": { "code": "INTERNAL_SERVER_ERROR" },
      "path": ["user", "posts"]  // Shows which field failed
    }
  ]
}
```

### **Complete Production Example**

```typescript
// config/graphql-config.ts
import { ApolloDriverConfig } from '@nestjs/apollo';
import { GraphQLError, GraphQLFormattedError } from 'graphql';

export const getGraphQLConfig = (isProduction: boolean): ApolloDriverConfig => ({
  autoSchemaFile: true,
  
  formatError: (error: GraphQLError): GraphQLFormattedError => {
    const originalError = error.extensions?.originalError as any;
    const statusCode = originalError?.statusCode || 500;
    const code = error.extensions?.code || 'INTERNAL_SERVER_ERROR';
    
    // Base error format
    const formattedError: any = {
      message: error.message,
      code,
      statusCode,
    };
    
    // Add path to show which field failed
    if (error.path) {
      formattedError.path = error.path;
    }
    
    // Production: Hide sensitive information
    if (isProduction) {
      // Generic message for internal errors
      if (code === 'INTERNAL_SERVER_ERROR') {
        formattedError.message = 'An internal error occurred';
      }
      
      // Add timestamp
      formattedError.timestamp = new Date().toISOString();
      
      return formattedError;
    }
    
    // Development: Show detailed information
    // Add query locations
    if (error.locations) {
      formattedError.locations = error.locations;
    }
    
    // Add validation errors
    if (originalError?.validationErrors) {
      formattedError.validationErrors = originalError.validationErrors;
    }
    
    // Add stacktrace
    const exception = error.extensions?.exception as any;
    if (exception?.stacktrace) {
      formattedError.stacktrace = exception.stacktrace;
    }
    
    formattedError.timestamp = new Date().toISOString();
    
    return formattedError;
  },
  
  // Disable playground in production
  playground: !isProduction,
  
  // Disable introspection in production
  introspection: !isProduction,
});

// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver } from '@nestjs/apollo';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { getGraphQLConfig } from './config/graphql-config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    GraphQLModule.forRootAsync({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        return getGraphQLConfig(isProduction);
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Development error (detailed):
// {
//   "errors": [{
//     "message": "User with ID 123 not found",
//     "code": "NOT_FOUND",
//     "statusCode": 404,
//     "path": ["user"],
//     "locations": [{ "line": 2, "column": 3 }],
//     "stacktrace": ["Error: User with ID 123 not found", "  at..."],
//     "timestamp": "2024-01-15T10:30:00.000Z"
//   }]
// }

// Production error (sanitized):
// {
//   "errors": [{
//     "message": "User with ID 123 not found",
//     "code": "NOT_FOUND",
//     "statusCode": 404,
//     "path": ["user"],
//     "timestamp": "2024-01-15T10:30:00.000Z"
//   }]
// }

// Internal error in production (generic):
// {
//   "errors": [{
//     "message": "An internal error occurred",
//     "code": "INTERNAL_SERVER_ERROR",
//     "statusCode": 500,
//     "timestamp": "2024-01-15T10:30:00.000Z"
//   }]
// }
```

### **Error Logging**

```typescript
// Log errors for monitoring
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      
      formatError: (error: GraphQLError) => {
        // Log error to monitoring service
        console.error('GraphQL Error:', {
          message: error.message,
          code: error.extensions?.code,
          path: error.path,
          timestamp: new Date().toISOString(),
        });
        
        // Send to error tracking (e.g., Sentry)
        // Sentry.captureException(error);
        
        // Return formatted error
        return {
          message: error.message,
          code: error.extensions?.code,
          path: error.path,
        };
      },
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: **GraphQL error format** includes **`message`** (error text), **`extensions`** (code, statusCode, details), **`path`** (field in query), **`locations`** (line/column). **Structure**: `{ errors: [...], data: {...} }` - both can coexist (partial responses). **Extensions**: contains `code` (`BAD_REQUEST`, `NOT_FOUND`, `UNAUTHENTICATED`, `FORBIDDEN`, `INTERNAL_SERVER_ERROR`), `statusCode` (HTTP code), validation errors, stacktrace (dev only). **Customize**: use `formatError` in `GraphQLModule.forRoot()`. **Production**: hide stacktraces, sanitize internal errors to generic message, add timestamp. **Development**: show full details, stacktraces, query locations. **Path array**: shows which field failed (e.g., `["user", "posts"]` = user.posts failed). **ValidationPipe errors**: automatically formatted with field-level validation messages in extensions.

</details>

47. How do you throw custom errors?

<details>
<summary><strong>Answer</strong></summary>

**Custom errors** created by extending **`HttpException`** or using **`GraphQLError`** with custom **`extensions`**. Create custom exception classes extending `HttpException`, specify status code and custom data in `extensions`. Or throw `GraphQLError` directly with custom code and metadata. Use **exception filters** to catch and transform errors. Custom errors provide **domain-specific error codes**, **structured metadata**, and **consistent error handling**.

### **Custom Exception Class**

```typescript
// exceptions/user-not-found.exception.ts
import { NotFoundException } from '@nestjs/common';

export class UserNotFoundException extends NotFoundException {
  constructor(userId: string) {
    super({
      message: `User with ID ${userId} not found`,
      error: 'UserNotFound',
      userId,  // Include in error response
    });
  }
}

// Usage in resolver
@Resolver(() => User)
export class UserResolver {
  @Query(() => User)
  async user(@Args('id') id: string): Promise<User> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new UserNotFoundException(id);  // Custom exception
    }
    
    return user;
  }
}

// Error response:
// {
//   "errors": [{
//     "message": "User with ID 123 not found",
//     "extensions": {
//       "code": "NOT_FOUND",
//       "error": "UserNotFound",
//       "userId": "123",
//       "statusCode": 404
//     },
//     "path": ["user"]
//   }]
// }
```

### **Custom Error with Error Codes**

```typescript
// exceptions/custom-errors.ts
import { HttpException, HttpStatus } from '@nestjs/common';

// Base custom error
export class DomainException extends HttpException {
  constructor(
    message: string,
    errorCode: string,
    statusCode: HttpStatus = HttpStatus.BAD_REQUEST,
    metadata?: Record<string, any>,
  ) {
    super(
      {
        message,
        errorCode,
        ...metadata,
      },
      statusCode,
    );
  }
}

// Specific domain errors
export class EmailAlreadyExistsException extends DomainException {
  constructor(email: string) {
    super(
      `Email ${email} is already registered`,
      'EMAIL_ALREADY_EXISTS',
      HttpStatus.CONFLICT,
      { email },
    );
  }
}

export class InsufficientBalanceException extends DomainException {
  constructor(required: number, available: number) {
    super(
      `Insufficient balance. Required: ${required}, Available: ${available}`,
      'INSUFFICIENT_BALANCE',
      HttpStatus.BAD_REQUEST,
      { required, available },
    );
  }
}

export class ResourceLockedException extends DomainException {
  constructor(resourceId: string, lockedBy: string) {
    super(
      `Resource is locked by another user`,
      'RESOURCE_LOCKED',
      HttpStatus.CONFLICT,
      { resourceId, lockedBy },
    );
  }
}

// Usage
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async register(@Args('input') input: RegisterInput): Promise<User> {
    const existingUser = await this.userService.findByEmail(input.email);
    
    if (existingUser) {
      throw new EmailAlreadyExistsException(input.email);
    }
    
    return this.userService.create(input);
  }
}

@Resolver(() => Transaction)
export class TransactionResolver {
  @Mutation(() => Transaction)
  async transfer(
    @Args('input') input: TransferInput,
    @CurrentUser() user: User,
  ): Promise<Transaction> {
    const balance = await this.walletService.getBalance(user.id);
    
    if (balance < input.amount) {
      throw new InsufficientBalanceException(input.amount, balance);
    }
    
    return this.transactionService.transfer(user.id, input);
  }
}

// Error responses:
// {
//   "errors": [{
//     "message": "Email john@example.com is already registered",
//     "extensions": {
//       "errorCode": "EMAIL_ALREADY_EXISTS",
//       "email": "john@example.com",
//       "statusCode": 409
//     }
//   }]
// }

// {
//   "errors": [{
//     "message": "Insufficient balance. Required: 100, Available: 50",
//     "extensions": {
//       "errorCode": "INSUFFICIENT_BALANCE",
//       "required": 100,
//       "available": 50,
//       "statusCode": 400
//     }
//   }]
// }
```

### **Using GraphQLError Directly**

```typescript
import { GraphQLError } from 'graphql';

@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  async publishPost(
    @Args('id') id: string,
    @CurrentUser() user: User,
  ): Promise<Post> {
    const post = await this.postService.findById(id);
    
    if (!post) {
      throw new GraphQLError('Post not found', {
        extensions: {
          code: 'POST_NOT_FOUND',
          statusCode: 404,
          postId: id,
        },
      });
    }
    
    if (post.authorId !== user.id) {
      throw new GraphQLError('You can only publish your own posts', {
        extensions: {
          code: 'NOT_POST_OWNER',
          statusCode: 403,
          postId: id,
          authorId: post.authorId,
          userId: user.id,
        },
      });
    }
    
    if (post.status === 'published') {
      throw new GraphQLError('Post is already published', {
        extensions: {
          code: 'ALREADY_PUBLISHED',
          statusCode: 409,
          postId: id,
          publishedAt: post.publishedAt,
        },
      });
    }
    
    return this.postService.publish(id);
  }
}
```

### **Enum for Error Codes**

```typescript
// constants/error-codes.enum.ts
export enum ErrorCode {
  // User errors
  USER_NOT_FOUND = 'USER_NOT_FOUND',
  EMAIL_ALREADY_EXISTS = 'EMAIL_ALREADY_EXISTS',
  INVALID_CREDENTIALS = 'INVALID_CREDENTIALS',
  EMAIL_NOT_VERIFIED = 'EMAIL_NOT_VERIFIED',
  
  // Post errors
  POST_NOT_FOUND = 'POST_NOT_FOUND',
  POST_ALREADY_PUBLISHED = 'POST_ALREADY_PUBLISHED',
  
  // Authorization errors
  INSUFFICIENT_PERMISSIONS = 'INSUFFICIENT_PERMISSIONS',
  RESOURCE_LOCKED = 'RESOURCE_LOCKED',
  
  // Business logic errors
  INSUFFICIENT_BALANCE = 'INSUFFICIENT_BALANCE',
  QUOTA_EXCEEDED = 'QUOTA_EXCEEDED',
}

// exceptions/domain-exception.ts
import { HttpException, HttpStatus } from '@nestjs/common';
import { ErrorCode } from '../constants/error-codes.enum';

export class DomainException extends HttpException {
  constructor(
    message: string,
    code: ErrorCode,
    statusCode: HttpStatus = HttpStatus.BAD_REQUEST,
    metadata?: Record<string, any>,
  ) {
    super(
      {
        message,
        code,  // Use enum for type safety
        ...metadata,
      },
      statusCode,
    );
  }
}

// Specific exceptions
export class UserNotFoundException extends DomainException {
  constructor(userId: string) {
    super(
      `User with ID ${userId} not found`,
      ErrorCode.USER_NOT_FOUND,
      HttpStatus.NOT_FOUND,
      { userId },
    );
  }
}

export class EmailAlreadyExistsException extends DomainException {
  constructor(email: string) {
    super(
      `Email ${email} is already registered`,
      ErrorCode.EMAIL_ALREADY_EXISTS,
      HttpStatus.CONFLICT,
      { email },
    );
  }
}

export class QuotaExceededException extends DomainException {
  constructor(limit: number, current: number) {
    super(
      `Quota exceeded. Limit: ${limit}, Current: ${current}`,
      ErrorCode.QUOTA_EXCEEDED,
      HttpStatus.TOO_MANY_REQUESTS,
      { limit, current },
    );
  }
}
```

### **Exception Filter (Advanced)**

```typescript
// filters/graphql-exception.filter.ts
import {
  Catch,
  ArgumentsHost,
  ExceptionFilter,
  HttpException,
} from '@nestjs/common';
import { GqlArgumentsHost } from '@nestjs/graphql';
import { GraphQLError } from 'graphql';

@Catch(HttpException)
export class GraphQLExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const gqlHost = GqlArgumentsHost.create(host);
    const response = exception.getResponse() as any;
    const status = exception.getStatus();
    
    // Transform HttpException to GraphQLError
    return new GraphQLError(
      response.message || exception.message,
      {
        extensions: {
          code: this.getErrorCode(status),
          statusCode: status,
          ...response,  // Include custom data from exception
          timestamp: new Date().toISOString(),
        },
      },
    );
  }

  private getErrorCode(statusCode: number): string {
    const codeMap: Record<number, string> = {
      400: 'BAD_REQUEST',
      401: 'UNAUTHENTICATED',
      403: 'FORBIDDEN',
      404: 'NOT_FOUND',
      409: 'CONFLICT',
      422: 'UNPROCESSABLE_ENTITY',
      429: 'TOO_MANY_REQUESTS',
      500: 'INTERNAL_SERVER_ERROR',
    };
    return codeMap[statusCode] || 'INTERNAL_SERVER_ERROR';
  }
}

// Apply filter globally
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: GraphQLExceptionFilter,
    },
  ],
})
export class AppModule {}
```

### **Validation Errors (Structured)**

```typescript
// exceptions/validation.exception.ts
import { BadRequestException } from '@nestjs/common';

interface ValidationError {
  field: string;
  message: string;
  value?: any;
}

export class ValidationException extends BadRequestException {
  constructor(errors: ValidationError[]) {
    super({
      message: 'Validation failed',
      code: 'VALIDATION_ERROR',
      errors,
    });
  }
}

// Usage
@Resolver(() => User)
export class UserResolver {
  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput): Promise<User> {
    const errors: ValidationError[] = [];
    
    // Custom validation
    if (input.age < 18) {
      errors.push({
        field: 'age',
        message: 'Must be at least 18 years old',
        value: input.age,
      });
    }
    
    if (!input.email.endsWith('@company.com')) {
      errors.push({
        field: 'email',
        message: 'Must use company email',
        value: input.email,
      });
    }
    
    if (errors.length > 0) {
      throw new ValidationException(errors);
    }
    
    return this.userService.create(input);
  }
}

// Error response:
// {
//   "errors": [{
//     "message": "Validation failed",
//     "extensions": {
//       "code": "VALIDATION_ERROR",
//       "errors": [
//         {
//           "field": "age",
//           "message": "Must be at least 18 years old",
//           "value": 16
//         },
//         {
//           "field": "email",
//           "message": "Must use company email",
//           "value": "user@gmail.com"
//         }
//       ],
//       "statusCode": 400
//     }
//   }]
// }
```

### **Complete Production Example**

```typescript
// constants/error-codes.enum.ts
export enum ErrorCode {
  // Authentication
  INVALID_CREDENTIALS = 'INVALID_CREDENTIALS',
  TOKEN_EXPIRED = 'TOKEN_EXPIRED',
  EMAIL_NOT_VERIFIED = 'EMAIL_NOT_VERIFIED',
  
  // Authorization
  INSUFFICIENT_PERMISSIONS = 'INSUFFICIENT_PERMISSIONS',
  RESOURCE_LOCKED = 'RESOURCE_LOCKED',
  
  // Validation
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  EMAIL_ALREADY_EXISTS = 'EMAIL_ALREADY_EXISTS',
  USERNAME_ALREADY_EXISTS = 'USERNAME_ALREADY_EXISTS',
  
  // Business Logic
  INSUFFICIENT_BALANCE = 'INSUFFICIENT_BALANCE',
  QUOTA_EXCEEDED = 'QUOTA_EXCEEDED',
  INVALID_OPERATION = 'INVALID_OPERATION',
  
  // Resources
  USER_NOT_FOUND = 'USER_NOT_FOUND',
  POST_NOT_FOUND = 'POST_NOT_FOUND',
  COMMENT_NOT_FOUND = 'COMMENT_NOT_FOUND',
}

// exceptions/base-domain.exception.ts
import { HttpException, HttpStatus } from '@nestjs/common';
import { ErrorCode } from '../constants/error-codes.enum';

export class BaseDomainException extends HttpException {
  constructor(
    message: string,
    code: ErrorCode,
    statusCode: HttpStatus,
    metadata?: Record<string, any>,
  ) {
    super(
      {
        message,
        code,
        timestamp: new Date().toISOString(),
        ...metadata,
      },
      statusCode,
    );
  }
}

// exceptions/user.exceptions.ts
import { HttpStatus } from '@nestjs/common';
import { BaseDomainException } from './base-domain.exception';
import { ErrorCode } from '../constants/error-codes.enum';

export class UserNotFoundException extends BaseDomainException {
  constructor(identifier: string) {
    super(
      `User not found`,
      ErrorCode.USER_NOT_FOUND,
      HttpStatus.NOT_FOUND,
      { identifier },
    );
  }
}

export class EmailAlreadyExistsException extends BaseDomainException {
  constructor(email: string) {
    super(
      `Email is already registered`,
      ErrorCode.EMAIL_ALREADY_EXISTS,
      HttpStatus.CONFLICT,
      { email },
    );
  }
}

export class InvalidCredentialsException extends BaseDomainException {
  constructor() {
    super(
      `Invalid email or password`,
      ErrorCode.INVALID_CREDENTIALS,
      HttpStatus.UNAUTHORIZED,
    );
  }
}

export class EmailNotVerifiedException extends BaseDomainException {
  constructor(email: string) {
    super(
      `Email not verified. Please check your inbox.`,
      ErrorCode.EMAIL_NOT_VERIFIED,
      HttpStatus.FORBIDDEN,
      { email },
    );
  }
}

// exceptions/business.exceptions.ts
export class InsufficientBalanceException extends BaseDomainException {
  constructor(required: number, available: number, currency: string = 'USD') {
    super(
      `Insufficient balance`,
      ErrorCode.INSUFFICIENT_BALANCE,
      HttpStatus.BAD_REQUEST,
      { required, available, currency },
    );
  }
}

export class QuotaExceededException extends BaseDomainException {
  constructor(resource: string, limit: number, current: number) {
    super(
      `${resource} quota exceeded`,
      ErrorCode.QUOTA_EXCEEDED,
      HttpStatus.TOO_MANY_REQUESTS,
      { resource, limit, current, resetAt: this.getResetTime() },
    );
  }

  private getResetTime(): string {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    tomorrow.setHours(0, 0, 0, 0);
    return tomorrow.toISOString();
  }
}

// user.service.ts
import {
  UserNotFoundException,
  EmailAlreadyExistsException,
  InvalidCredentialsException,
  EmailNotVerifiedException,
} from './exceptions/user.exceptions';

@Injectable()
export class UserService {
  async findById(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new UserNotFoundException(id);
    }
    
    return user;
  }

  async register(input: RegisterInput): Promise<User> {
    const existingUser = await this.userRepository.findOne({
      where: { email: input.email },
    });
    
    if (existingUser) {
      throw new EmailAlreadyExistsException(input.email);
    }
    
    const user = await this.userRepository.save(input);
    
    // Send verification email
    await this.emailService.sendVerificationEmail(user);
    
    return user;
  }

  async login(email: string, password: string): Promise<{ user: User; token: string }> {
    const user = await this.userRepository.findOne({ where: { email } });
    
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new InvalidCredentialsException();
    }
    
    if (!user.emailVerified) {
      throw new EmailNotVerifiedException(email);
    }
    
    const token = this.jwtService.sign({ sub: user.id });
    
    return { user, token };
  }
}

// post.resolver.ts
import { QuotaExceededException } from './exceptions/business.exceptions';

@Resolver(() => Post)
export class PostResolver {
  @Mutation(() => Post)
  @UseGuards(GqlAuthGuard)
  async createPost(
    @CurrentUser() user: User,
    @Args('input') input: CreatePostInput,
  ): Promise<Post> {
    // Check quota
    const todayPosts = await this.postService.countTodayPosts(user.id);
    const dailyLimit = user.plan === 'free' ? 5 : 100;
    
    if (todayPosts >= dailyLimit) {
      throw new QuotaExceededException('posts', dailyLimit, todayPosts);
    }
    
    return this.postService.create(user, input);
  }
}

// Error responses:

// 1. User not found
// {
//   "errors": [{
//     "message": "User not found",
//     "extensions": {
//       "code": "USER_NOT_FOUND",
//       "statusCode": 404,
//       "identifier": "123",
//       "timestamp": "2024-01-15T10:30:00.000Z"
//     }
//   }]
// }

// 2. Email already exists
// {
//   "errors": [{
//     "message": "Email is already registered",
//     "extensions": {
//       "code": "EMAIL_ALREADY_EXISTS",
//       "statusCode": 409,
//       "email": "john@example.com",
//       "timestamp": "2024-01-15T10:30:00.000Z"
//     }
//   }]
// }

// 3. Quota exceeded
// {
//   "errors": [{
//     "message": "posts quota exceeded",
//     "extensions": {
//       "code": "QUOTA_EXCEEDED",
//       "statusCode": 429,
//       "resource": "posts",
//       "limit": 5,
//       "current": 5,
//       "resetAt": "2024-01-16T00:00:00.000Z",
//       "timestamp": "2024-01-15T10:30:00.000Z"
//     }
//   }]
// }
```

**Interview Tip**: **Custom errors** created by **extending `HttpException`** with custom code/metadata, or throwing **`GraphQLError`** directly. **Pattern**: extend `HttpException`, pass custom data in constructor, include in error response extensions. **Best practice**: create domain-specific exceptions (`UserNotFoundException`, `EmailAlreadyExistsException`), use **enum for error codes** (type safety), include relevant metadata (userId, email, etc.). **Base class**: create `BaseDomainException` extending `HttpException`, reuse for all custom errors. **GraphQLError**: use for one-off errors with `new GraphQLError(message, { extensions: { code, ...metadata } })`. **Exception filters**: catch and transform exceptions globally with `ExceptionFilter`. **Structured validation**: create `ValidationException` with array of field-level errors. **Metadata**: include actionable data (limits, current values, resetAt for quotas). **Client benefits**: consistent error format, machine-readable codes, detailed context for error handling.

</details>

## GraphQL Playground

48. What is GraphQL Playground?

<details>
<summary><strong>Answer</strong></summary>

**GraphQL Playground** is an **interactive IDE** for testing GraphQL queries, mutations, subscriptions. Provides **auto-completion**, **schema documentation**, **query history**, **variables editor**, **HTTP headers** configuration. Enabled by default in development at `/graphql` endpoint. Features include **syntax highlighting**, **multi-tab interface**, **schema explorer**, **response viewer**. Disable in production for security.

### **Default Playground Access**

```typescript
// app.module.ts - Playground enabled by default
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      // playground: true,  // Enabled by default in development
    }),
  ],
})
export class AppModule {}

// Start app and access Playground:
// npm run start:dev
// Open browser: http://localhost:3000/graphql

// Playground interface shows:
// - Query editor (left panel)
// - Response viewer (right panel)
// - Schema documentation (Docs tab)
// - Query history
// - Variables editor
// - HTTP headers editor
```

### **Playground Features**

```typescript
// 1. Auto-completion
// Type 'query {' and press Ctrl+Space to see available queries

query {
  # Start typing 'use' and get suggestions:
  # - user
  # - users
  # - userStatistics
  users {  # Auto-complete fields
    id
    name  # Ctrl+Space shows available fields
    email
  }
}

// 2. Schema Documentation
// Click "DOCS" or "SCHEMA" tab to browse:
// - All types
// - All queries
// - All mutations
// - All subscriptions
// - Field descriptions
// - Type definitions

// 3. Query Variables
// Bottom panel for variables (JSON format)
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
  }
}

// Variables panel:
{
  "id": "123"
}

// 4. HTTP Headers
// Bottom panel for headers
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json"
}

// 5. Multi-tab Interface
// Click "+" to open new tab
// Tab 1: Query users
// Tab 2: Create user mutation
// Tab 3: Test authentication

// 6. Query History
// Click history icon to see past queries
// Re-run previous queries

// 7. Prettify Query
// Click "Prettify" button to format query

// 8. Copy Curl Command
// Click "Copy cURL" to get equivalent curl command
```

### **Common Playground Operations**

```typescript
// Testing queries
query GetAllUsers {
  users {
    id
    name
    email
    posts {
      id
      title
    }
  }
}

// Testing mutations
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
  }
}

// Variables:
{
  "input": {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "SecurePass123"
  }
}

// Testing with authentication
// 1. First, login to get token:
mutation Login {
  login(input: {
    email: "user@example.com"
    password: "password123"
  }) {
    token
  }
}

// 2. Copy token from response
// 3. Add to HTTP Headers:
{
  "Authorization": "Bearer <paste-token-here>"
}

// 4. Now test protected queries:
query Me {
  me {
    id
    name
    email
  }
}

// Testing fragments
fragment UserFields on User {
  id
  name
  email
}

query GetUsers {
  users {
    ...UserFields
    posts {
      id
      title
    }
  }
  
  me {
    ...UserFields
  }
}

// Testing aliases
query GetMultipleUsers {
  user1: user(id: "1") {
    id
    name
  }
  
  user2: user(id: "2") {
    id
    name
  }
}

// Testing directives
query GetUser($includeEmail: Boolean!) {
  user(id: "1") {
    id
    name
    email @include(if: $includeEmail)
  }
}

// Variables:
{
  "includeEmail": true
}
```

### **Playground vs GraphiQL vs Apollo Studio**

```typescript
// GraphQL Playground (default in NestJS)
// - Feature-rich IDE
// - Multiple tabs
// - Query history
// - Themes (light/dark)
// - File upload support
// - Subscriptions support

// GraphiQL (alternative)
// - Simpler interface
// - Single tab
// - Basic features
// - Lightweight

// Apollo Studio (cloud-based)
// - Advanced analytics
// - Query performance
// - Schema validation
// - Team collaboration
// - Production monitoring

// Switch to GraphiQL in NestJS:
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      playground: false,  // Disable Playground
      // Use custom GraphiQL endpoint instead
    }),
  ],
})
export class AppModule {}
```

### **Playground Settings**

```typescript
// app.module.ts - Custom Playground settings
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      playground: {
        // Custom settings
        settings: {
          'request.credentials': 'include',  // Include cookies
          'editor.theme': 'dark',            // Dark theme
          'editor.fontSize': 14,
          'editor.fontFamily': '"Fira Code", monospace',
          'editor.reuseHeaders': true,       // Reuse headers across tabs
          'tracing.hideTracingResponse': false,
        },
        
        // Default tabs
        tabs: [
          {
            endpoint: '/graphql',
            query: `
              # Welcome to GraphQL Playground
              # Try this sample query:
              
              query GetUsers {
                users {
                  id
                  name
                }
              }
            `,
          },
        ],
      },
    }),
  ],
})
export class AppModule {}
```

### **Playground Security**

```typescript
// NEVER enable Playground in production!
import { ConfigService } from '@nestjs/config';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => ({
        autoSchemaFile: true,
        // Only enable in development
        playground: configService.get('NODE_ENV') !== 'production',
        
        // Also disable introspection in production
        introspection: configService.get('NODE_ENV') !== 'production',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Why disable in production:
// 1. Security: Exposes schema and allows query testing
// 2. Performance: Adds overhead
// 3. Attack surface: Allows exploration of API
// 4. Data exposure: Could leak sensitive information
```

### **Testing Workflow with Playground**

```typescript
// Step 1: Explore Schema
// - Click "SCHEMA" tab
// - Browse available queries, mutations, types
// - Read field descriptions

// Step 2: Write Query
// - Use auto-completion
// - Add fields as needed
// - Use fragments for reusability

query GetUserWithPosts($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts(limit: 5) {
      id
      title
      createdAt
    }
  }
}

// Step 3: Add Variables
{
  "userId": "123"
}

// Step 4: Test Authentication
// - Login first
// - Copy token
// - Add to headers

// Step 5: Execute Query
// - Click Play button
// - Review response
// - Check for errors

// Step 6: Iterate
// - Modify query based on response
// - Add/remove fields
// - Test edge cases

// Step 7: Save Query
// - Copy query for use in client
// - Or save in query history
```

### **Playground Tips & Tricks**

```typescript
// 1. Keyboard Shortcuts
// Ctrl/Cmd + Enter: Execute query
// Ctrl/Cmd + Space: Auto-complete
// Ctrl/Cmd + /: Comment/uncomment
// Ctrl/Cmd + D: Duplicate line
// Ctrl/Cmd + F: Find
// Ctrl/Cmd + Shift + F: Format query

// 2. Query Name for Organization
query GetUserProfile($id: ID!) {  # Named query
  user(id: $id) {
    name
  }
}

mutation CreateNewPost($input: CreatePostInput!) {  # Named mutation
  createPost(input: $input) {
    id
  }
}

// 3. Use Fragments for Complex Queries
fragment PostFields on Post {
  id
  title
  content
  createdAt
}

fragment UserFields on User {
  id
  name
  email
}

query ComplexQuery {
  posts {
    ...PostFields
    author {
      ...UserFields
    }
  }
}

// 4. Test Error Handling
# Invalid ID to test error response
query {
  user(id: "invalid-id") {
    id
    name
  }
}

# Missing required field
mutation {
  createUser(input: {
    # name missing (required)
    email: "test@example.com"
  }) {
    id
  }
}

// 5. Export/Import Queries
// - Save queries in .graphql files
// - Import into Playground
// - Share with team
```

**Interview Tip**: **GraphQL Playground** is **interactive IDE** for testing GraphQL APIs, enabled by default at `/graphql` endpoint. **Features**: auto-completion, schema docs, query history, variables editor, HTTP headers, multi-tabs, syntax highlighting. **Use cases**: test queries/mutations during development, explore schema, debug authentication, test error handling. **Schema explorer**: browse types, fields, descriptions without reading code. **Variables**: test dynamic queries with JSON variables. **Headers**: add `Authorization` header for protected endpoints. **Security**: ALWAYS disable in production (`playground: false`) to prevent schema exposure. **Alternative**: GraphiQL (simpler), Apollo Studio (production monitoring). **Best practice**: use Playground during development, document queries for client team, disable before deployment.

</details>

49. How do you enable/disable Playground in production?

<details>
<summary><strong>Answer</strong></summary>

**Disable Playground** in production by setting **`playground: false`** in `GraphQLModule.forRoot()`. Best practice: use **environment variable** to toggle based on `NODE_ENV`. Also disable **`introspection`** in production (prevents schema queries). Configure conditionally with **`useFactory`** + `ConfigService`. Security risk: Playground exposes schema, allows query testing, potential data leaks.

### **Disable Playground (Simple)**

```typescript
// app.module.ts - Hardcoded disable
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      playground: false,  // Disabled
      introspection: false,  // Also disable schema introspection
    }),
  ],
})
export class AppModule {}

// Now accessing /graphql returns:
// "GET query missing."
// Instead of Playground UI
```

### **Environment-Based Configuration (Recommended)**

```typescript
// .env.development
NODE_ENV=development
GRAPHQL_PLAYGROUND=true

// .env.production
NODE_ENV=production
GRAPHQL_PLAYGROUND=false

// app.module.ts - Environment-based
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => ({
        autoSchemaFile: true,
        
        // Enable Playground only in development
        playground: configService.get('NODE_ENV') !== 'production',
        
        // Also disable introspection in production
        introspection: configService.get('NODE_ENV') !== 'production',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Development: npm run start:dev
// - Playground enabled at http://localhost:3000/graphql

// Production: npm run build && npm run start:prod
// - Playground disabled
// - Introspection disabled
```

### **Advanced Configuration**

```typescript
// app.module.ts - Advanced environment-based config
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        const isDevelopment = configService.get('NODE_ENV') === 'development';
        const isStaging = configService.get('NODE_ENV') === 'staging';
        
        return {
          autoSchemaFile: true,
          
          // Playground: only in development
          playground: isDevelopment,
          
          // Introspection: dev and staging, not production
          introspection: !isProduction,
          
          // Debug mode: only in development
          debug: isDevelopment,
          
          // Error formatting: hide details in production
          formatError: (error) => {
            if (isProduction) {
              // Hide sensitive error details
              return {
                message: error.message,
                code: error.extensions?.code,
              };
            }
            // Full error details in dev/staging
            return error;
          },
          
          // Context
          context: ({ req }) => ({ req }),
        };
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### **Custom Playground Path**

```typescript
// app.module.ts - Custom path for Playground
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      path: '/api/graphql',  // Change GraphQL endpoint
      playground: {
        // Playground will be at /api/graphql
        endpoint: '/api/graphql',
      },
    }),
  ],
})
export class AppModule {}

// Access Playground at:
// http://localhost:3000/api/graphql
```

### **Conditional Playground with Auth**

```typescript
// app.module.ts - Playground with authentication
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: true,
      
      // Enable Playground but require auth
      playground: {
        settings: {
          'request.credentials': 'include',  // Include cookies
        },
      },
    }),
  ],
})
export class AppModule {}

// Add middleware to protect Playground
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Protect Playground endpoint with basic auth
  if (process.env.NODE_ENV !== 'production') {
    const basicAuth = require('express-basic-auth');
    
    app.use(
      '/graphql',
      basicAuth({
        challenge: true,
        users: {
          admin: process.env.PLAYGROUND_PASSWORD || 'secret',
        },
      }),
    );
  }
  
  await app.listen(3000);
}
bootstrap();
```

### **Completely Disable in Production**

```typescript
// production.config.ts
export const productionGraphQLConfig = {
  autoSchemaFile: true,
  
  // Disable Playground
  playground: false,
  
  // Disable introspection (prevents schema queries)
  introspection: false,
  
  // Disable debug mode
  debug: false,
  
  // Sanitize error messages
  formatError: (error: any) => ({
    message: error.message,
    code: error.extensions?.code,
  }),
  
  // Enable CORS only for specific origins
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
    credentials: true,
  },
};

// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ConfigService } from '@nestjs/config';
import { productionGraphQLConfig } from './config/production.config';
import { developmentGraphQLConfig } from './config/development.config';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        
        return isProduction
          ? productionGraphQLConfig
          : developmentGraphQLConfig;
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### **Testing Production Config Locally**

```typescript
// Test production settings locally
// .env.test
NODE_ENV=production

// Run with production config:
// NODE_ENV=production npm run start

// Verify:
// 1. Visit http://localhost:3000/graphql
//    Should see: "GET query missing." (no Playground)

// 2. Try introspection query:
query {
  __schema {
    types {
      name
    }
  }
}
// Should fail with: "GraphQL introspection is not allowed"

// 3. Test via curl:
curl -X POST http://localhost:3000/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ users { id name } }"}'
// Should work (API still functional)
```

### **Security Best Practices**

```typescript
// Complete production security config
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ConfigService } from '@nestjs/config';
import helmet from 'helmet';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        
        return {
          autoSchemaFile: true,
          
          // 1. Disable Playground
          playground: !isProduction,
          
          // 2. Disable introspection
          introspection: !isProduction,
          
          // 3. Disable debug mode
          debug: !isProduction,
          
          // 4. Sanitize errors
          formatError: (error) => {
            if (isProduction && error.extensions?.code === 'INTERNAL_SERVER_ERROR') {
              return {
                message: 'An internal error occurred',
                code: 'INTERNAL_SERVER_ERROR',
              };
            }
            return {
              message: error.message,
              code: error.extensions?.code,
              ...(isProduction ? {} : { path: error.path }),
            };
          },
          
          // 5. Rate limiting (via context)
          context: ({ req }) => {
            // Add rate limiting logic
            return { req };
          },
          
          // 6. CORS restrictions
          cors: {
            origin: configService.get('ALLOWED_ORIGINS')?.split(',') || [],
            credentials: true,
          },
        };
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// main.ts - Additional security
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  if (process.env.NODE_ENV === 'production') {
    // Use helmet for security headers
    app.use(helmet());
    
    // Disable X-Powered-By header
    app.getHttpAdapter().getInstance().disable('x-powered-by');
  }
  
  await app.listen(3000);
}
```

### **Alternative: Use Apollo Studio**

```typescript
// For production, use Apollo Studio instead of Playground
// app.module.ts
import { ApolloServerPluginLandingPageDisabled } from '@apollo/server/plugin/disabled';
import {
  ApolloServerPluginLandingPageLocalDefault,
  ApolloServerPluginLandingPageProductionDefault,
} from '@apollo/server/plugin/landingPage/default';

@Module({
  imports: [
    GraphQLModule.forRootAsync<ApolloDriverConfig>({
      driver: ApolloDriver,
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        
        return {
          autoSchemaFile: true,
          introspection: !isProduction,
          
          plugins: isProduction
            ? [
                // Production: Use Apollo Studio or disable completely
                ApolloServerPluginLandingPageDisabled(),
              ]
            : [
                // Development: Use local landing page
                ApolloServerPluginLandingPageLocalDefault(),
              ],
        };
      },
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: **Disable Playground in production** with **`playground: false`** in GraphQL module config. **Best practice**: use environment variable (`NODE_ENV`) to toggle conditionally via `useFactory` + `ConfigService`. **Also disable**: `introspection: false` (prevents schema queries), `debug: false`. **Why**: security risk - exposes schema, allows query testing, potential data leaks, attack surface. **Development**: `playground: true`, full error details, introspection enabled. **Production**: all disabled, sanitized errors, CORS restrictions. **Testing**: run locally with `NODE_ENV=production` to verify Playground disabled. **Alternative**: protect Playground with basic auth in staging, use Apollo Studio for production monitoring. **Key**: NEVER expose Playground/introspection in production - major security vulnerability.

</details>

50. How do you test queries using Playground?

<details>
<summary><strong>Answer</strong></summary>

**Test queries in Playground** by writing query in left panel, adding variables in variables panel (JSON), setting headers in HTTP headers panel, clicking play button. Use **auto-completion** (Ctrl+Space) for field suggestions, **schema explorer** (DOCS tab) to browse types, **query history** to re-run queries. Test **mutations** with variables, **authentication** with Bearer token in headers, **error handling** with invalid inputs. Save queries for documentation.

### **Basic Query Testing**

```typescript
// 1. Open Playground
// http://localhost:3000/graphql

// 2. Write query in left panel
query GetAllUsers {
  users {
    id
    name
    email
  }
}

// 3. Click Play button (or Ctrl+Enter)
// 4. See response in right panel:
{
  "data": {
    "users": [
      { "id": "1", "name": "John", "email": "john@example.com" },
      { "id": "2", "name": "Jane", "email": "jane@example.com" }
    ]
  }
}

// 5. Modify query to add more fields
query GetAllUsers {
  users {
    id
    name
    email
    createdAt  # Add new field (auto-complete available)
    posts {    # Add nested field
      id
      title
    }
  }
}
```

### **Testing with Variables**

```typescript
// Query with variables
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
    }
  }
}

// Variables panel (bottom left):
{
  "userId": "123"
}

// Test different values:
// Change "userId": "456" and re-run
// Test invalid ID: "userId": "invalid"
// Test non-existent ID: "userId": "999"

// Multiple variables:
query GetUserPosts($userId: ID!, $limit: Int!) {
  user(id: $userId) {
    name
    posts(limit: $limit) {
      id
      title
    }
  }
}

// Variables:
{
  "userId": "123",
  "limit": 5
}
```

### **Testing Mutations**

```typescript
// Create user mutation
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
  }
}

// Variables:
{
  "input": {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "SecurePass123"
  }
}

// Click Play to execute
// Response:
{
  "data": {
    "createUser": {
      "id": "new-user-id",
      "name": "John Doe",
      "email": "john@example.com"
    }
  }
}

// Update mutation
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    id
    name
    email
  }
}

// Variables:
{
  "id": "123",
  "input": {
    "name": "John Updated"
  }
}

// Delete mutation
mutation DeleteUser($id: ID!) {
  deleteUser(id: $id)
}

// Variables:
{
  "id": "123"
}
```

### **Testing Authentication**

```typescript
// Step 1: Login to get token
mutation Login {
  login(input: {
    email: "user@example.com"
    password: "password123"
  }) {
    token
    user {
      id
      name
    }
  }
}

// Response:
{
  "data": {
    "login": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
        "id": "1",
        "name": "John Doe"
      }
    }
  }
}

// Step 2: Copy token from response

// Step 3: Add to HTTP Headers panel (bottom left)
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 4: Now test protected queries
query Me {
  me {
    id
    name
    email
  }
}

// Response (with valid token):
{
  "data": {
    "me": {
      "id": "1",
      "name": "John Doe",
      "email": "john@example.com"
    }
  }
}

// Test without token:
// Remove Authorization header
// Re-run query
// Should get error:
{
  "errors": [{
    "message": "Unauthorized",
    "extensions": { "code": "UNAUTHENTICATED" }
  }]
}

// Test with invalid token:
{
  "Authorization": "Bearer invalid-token"
}
// Should get error:
{
  "errors": [{
    "message": "Invalid token",
    "extensions": { "code": "UNAUTHENTICATED" }
  }]
}
```

### **Testing Error Handling**

```typescript
// Test validation errors
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
  }
}

// Invalid input (missing required fields):
{
  "input": {
    "name": "A",  // Too short
    "email": "invalid-email",  // Invalid format
    "password": "123"  // Too short
  }
}

// Error response:
{
  "errors": [{
    "message": "Bad Request Exception",
    "extensions": {
      "code": "BAD_REQUEST",
      "validationErrors": [
        {
          "property": "name",
          "constraints": { "minLength": "Name must be at least 2 characters" }
        },
        {
          "property": "email",
          "constraints": { "isEmail": "Invalid email format" }
        },
        {
          "property": "password",
          "constraints": { "minLength": "Password must be at least 8 characters" }
        }
      ]
    }
  }]
}

// Test not found errors
query {
  user(id: "non-existent-id") {
    id
    name
  }
}

// Error response:
{
  "errors": [{
    "message": "User not found",
    "extensions": { "code": "NOT_FOUND" },
    "path": ["user"]
  }],
  "data": { "user": null }
}

// Test authorization errors
mutation DeleteUser($id: ID!) {
  deleteUser(id: $id)
}

// Variables (trying to delete another user):
{
  "id": "other-user-id"
}

// Error response:
{
  "errors": [{
    "message": "You can only delete your own account",
    "extensions": { "code": "FORBIDDEN" }
  }]
}
```

### **Using Schema Explorer**

```typescript
// Click "DOCS" or "SCHEMA" tab on right side

// 1. Browse root types:
// - Query (all queries)
// - Mutation (all mutations)
// - Subscription (all subscriptions)

// 2. Click on "Query" to see all queries:
// - users: [User!]!
// - user(id: ID!): User
// - me: User!
// - posts(limit: Int): [Post!]!

// 3. Click on a query to see details:
// user(id: ID!): User
//   - Arguments:
//     - id: ID! (required)
//   - Returns: User (nullable)
//   - Description: "Get user by ID"

// 4. Click on type to see fields:
// User
//   - id: ID!
//   - name: String!
//   - email: String!
//   - posts: [Post!]!
//   - createdAt: DateTime!

// 5. Use this info to construct queries:
// Click on field name to add to query
```

### **Testing Complex Queries**

```typescript
// Test with fragments
fragment UserFields on User {
  id
  name
  email
}

fragment PostFields on Post {
  id
  title
  content
  createdAt
}

query GetUsersWithPosts {
  users {
    ...UserFields
    posts {
      ...PostFields
      author {
        ...UserFields
      }
    }
  }
}

// Test with aliases
query GetMultipleUsers {
  user1: user(id: "1") {
    id
    name
  }
  
  user2: user(id: "2") {
    id
    name
  }
  
  allUsers: users {
    id
    name
  }
}

// Test with directives
query GetUser($id: ID!, $includeEmail: Boolean!, $includePosts: Boolean!) {
  user(id: $id) {
    id
    name
    email @include(if: $includeEmail)
    posts @include(if: $includePosts) {
      id
      title
    }
  }
}

// Variables:
{
  "id": "123",
  "includeEmail": true,
  "includePosts": false
}

// Test nested queries
query {
  user(id: "1") {
    name
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

### **Playground Workflow Best Practices**

```typescript
// 1. Start with schema exploration
// - Click DOCS tab
// - Browse available queries
// - Read field descriptions

// 2. Write simple query first
query {
  users {
    id
    name
  }
}

// 3. Gradually add complexity
query {
  users {
    id
    name
    email  // Added
    posts {  // Added nested field
      id
      title
    }
  }
}

// 4. Use variables for dynamic queries
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
  }
}

// 5. Test edge cases
// - Invalid IDs
// - Missing required fields
// - Large limits
// - Empty results

// 6. Test authentication flow
// - Login
// - Copy token
// - Add to headers
// - Test protected endpoints

// 7. Document queries
// - Name queries clearly
// - Add comments
// - Save in .graphql files

// Example documented query:
# Get user profile with recent posts
# Requires authentication
query GetUserProfile($userId: ID!, $postsLimit: Int = 10) {
  user(id: $userId) {
    id
    name
    email
    
    # User's recent posts
    posts(limit: $postsLimit) {
      id
      title
      createdAt
    }
  }
}

// 8. Use query history
// - Click history icon
// - Re-run previous successful queries
// - Modify and test variations

// 9. Export for client use
// - Copy working queries
// - Share with frontend team
// - Save in query library
```

### **Testing Performance**

```typescript
// Test query performance in Playground

// 1. Simple query (baseline)
query {
  users {
    id
    name
  }
}
// Check response time in browser DevTools Network tab

// 2. Query with relationships (check for N+1)
query {
  users {
    id
    name
    posts {  // Does this cause N+1?
      id
      title
    }
  }
}
// Check:
// - Response time (should be fast with DataLoader)
// - Database queries in logs (should be 2, not N+1)

// 3. Deep nesting (potential performance issue)
query {
  users {
    posts {
      comments {
        author {
          posts {
            comments {  // Very deep!
              author {
                name
              }
            }
          }
        }
      }
    }
  }
}
// This might be slow - test with limit:
query {
  users(limit: 5) {
    posts(limit: 5) {
      title
    }
  }
}

// 4. Large result sets
query {
  posts(limit: 1000) {  // Large limit
    id
    title
    content
  }
}
// Check response size and time

// 5. Use @defer/@stream for large results (if supported)
query {
  users {
    id
    name
    posts @defer {
      id
      title
    }
  }
}
```

**Interview Tip**: **Test queries in Playground** by writing query in editor, adding variables (JSON) in variables panel, setting headers (e.g., `Authorization: Bearer <token>`) in HTTP headers panel, clicking play. **Workflow**: explore schema (DOCS tab) → write simple query → use auto-completion (Ctrl+Space) → add variables → execute → iterate. **Testing**: mutations with variables, authentication (login → get token → add to headers), error handling (invalid inputs), edge cases (missing fields, large limits). **Features**: query history (re-run), fragments (reusable fields), aliases (multiple queries), directives (`@include`, `@skip`). **Best practice**: name queries, add comments, test both success and error cases, save working queries for documentation. **Performance**: check response times, watch for N+1 problems, test with realistic data sizes. **Security**: test authorization (try accessing protected resources without token, with invalid token, as wrong user).

</details>

---

**Congratulations!** You've completed all 50 NestJS GraphQL interview questions. This comprehensive guide covers everything from fundamentals to advanced topics including DataLoader, authentication, error handling, and production best practices.
