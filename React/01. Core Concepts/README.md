# Core Concepts

> Beginner / Junior Level (0-1 years)

---

## Questions

1. What is React and why is it popular?
2. What is the difference between React and ReactDOM?
3. What is JSX?
4. What are components in React?
5. What is the difference between functional and class components?
6. What are props in React?
7. What is state in React?
8. What is the Virtual DOM?
9. How does React's reconciliation algorithm work?
10. What is the difference between state and props?

---

## Detailed Answers

### 1. What is React and why is it popular?

<details>
<summary>View Answer</summary>

**What is React?**

React is a **JavaScript library** for building user interfaces, developed and maintained by Facebook (Meta). It focuses on creating reusable UI components and efficiently updating the view when data changes.

**Key Characteristics:**
- **Component-Based**: Build encapsulated components that manage their own state
- **Declarative**: Describe what the UI should look like, React handles the updates
- **Learn Once, Write Anywhere**: Can render on server (Next.js), mobile (React Native), VR, etc.
- **Virtual DOM**: Efficient rendering using a lightweight copy of the actual DOM

**Why is React Popular?**

1. **Component Reusability**
```jsx
// Reusable Button component used across the application
const Button = ({ children, onClick, variant = 'primary' }) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// Usage in multiple places
<Button onClick={handleSubmit}>Submit</Button>
<Button onClick={handleCancel} variant="secondary">Cancel</Button>
```

2. **Virtual DOM for Performance**
```jsx
// When state changes, React efficiently updates only what changed
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p> {/* Only this updates, not entire component */}
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

3. **Strong Ecosystem**
- **State Management**: Redux, Zustand, Jotai, Recoil
- **Routing**: React Router, TanStack Router
- **UI Libraries**: Material-UI, Ant Design, Chakra UI, shadcn/ui
- **Forms**: React Hook Form, Formik
- **Data Fetching**: TanStack Query, SWR, Apollo Client

4. **Large Community & Job Market**
- 220k+ stars on GitHub
- Used by Facebook, Instagram, Netflix, Airbnb, Uber
- Abundant tutorials, courses, and documentation
- High demand in job market

5. **Declarative Approach**
```jsx
// Imperative (vanilla JS) - HOW to do it
const button = document.createElement('button');
button.textContent = 'Click me';
button.addEventListener('click', () => {
  alert('Clicked!');
});
document.body.appendChild(button);

// Declarative (React) - WHAT you want
const App = () => (
  <button onClick={() => alert('Clicked!')}>
    Click me
  </button>
);
```

6. **React Hooks** (Modern approach since React 16.8)
```jsx
// Simplified state management with hooks
const TodoList = () => {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  useEffect(() => {
    // Side effects like API calls
    fetchTodos().then(setTodos);
  }, []);
  
  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input }]);
    setInput('');
  };
  
  return (
    <div>
      <input 
        value={input} 
        onChange={(e) => setInput(e.target.value)} 
      />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
};
```

**Production Use Cases:**
- **Single Page Applications (SPAs)**: Dashboard applications
- **E-commerce**: Product catalogs, shopping carts
- **Social Media**: News feeds, messaging interfaces
- **Content Management**: Admin panels, blog platforms
- **Mobile Apps**: React Native for iOS/Android

**Interview Tips:**
- Mention React is a **library**, not a framework (unlike Angular)
- Highlight the Virtual DOM and reconciliation for performance
- Discuss the shift from class components to functional components with hooks
- Mention React's flexibility - can be used with various architectures

</details>

---

### 2. What is the difference between React and ReactDOM?

<details>
<summary>View Answer</summary>

**React vs ReactDOM: Separation of Concerns**

**React** and **ReactDOM** are two separate packages with distinct responsibilities:

**1. React (Core Library)**
- **Package**: `react`
- **Purpose**: Core functionality for creating and managing components
- **Responsibilities**:
  - Component creation and lifecycle
  - State management
  - Hooks (useState, useEffect, etc.)
  - JSX transformation logic
  - Virtual DOM representation

```jsx
// From 'react' package
import React, { useState, useEffect, Component } from 'react';

// Creating components (React's responsibility)
const MyComponent = () => {
  const [state, setState] = useState(0); // React hook
  
  useEffect(() => {
    console.log('Effect ran');
  }, []); // React hook
  
  return <div>Hello</div>; // JSX (React)
};
```

**2. ReactDOM (Renderer for Web)**
- **Package**: `react-dom`
- **Purpose**: Rendering React components to the actual browser DOM
- **Responsibilities**:
  - Mounting components to the DOM
  - Updating the real DOM
  - Handling browser-specific features
  - Event handling in the browser

```jsx
// From 'react-dom' package
import ReactDOM from 'react-dom/client';

// Rendering to DOM (ReactDOM's responsibility)
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

**Why the Separation?**

This separation enables React to target **multiple platforms**:

```jsx
// Web (Browser) - ReactDOM
import ReactDOM from 'react-dom/client';
ReactDOM.createRoot(document.getElementById('root')).render(<App />);

// Mobile - React Native
import { AppRegistry } from 'react-native';
AppRegistry.registerComponent('MyApp', () => App);

// VR - React 360
import { AppRegistry } from 'react-360';
AppRegistry.registerComponent('MyVRApp', () => App);

// Canvas - React Three Fiber
import { Canvas } from '@react-three/fiber';
ReactDOM.createRoot(root).render(
  <Canvas>
    <App3D />
  </Canvas>
);
```

**Complete Example: Package Separation**

```jsx
// App.js - Uses ONLY 'react' package
import React, { useState } from 'react';

const App = () => {
  const [count, setCount] = useState(0);
  
  // All this logic is platform-agnostic
  // It doesn't know about DOM, mobile, or VR
  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
};

export default App;

// index.js - Uses 'react-dom' for web rendering
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// ReactDOM bridges React components to browser DOM
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**ReactDOM Methods (React 18+)**

```jsx
// 1. createRoot - Modern API (React 18+)
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);

// 2. hydrateRoot - For Server-Side Rendering (SSR)
const root = ReactDOM.hydrateRoot(
  document.getElementById('root'),
  <App />
);

// 3. Portals - Render outside parent hierarchy
import { createPortal } from 'react-dom';

const Modal = ({ children }) => {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root') // Different DOM node
  );
};

// 4. flushSync - Force synchronous update (use sparingly)
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(count + 1); // Guaranteed to be updated synchronously
});
console.log(count); // Will reflect updated value
```

**Legacy vs Modern ReactDOM API**

```jsx
// ❌ Legacy API (React 17 and earlier)
import ReactDOM from 'react-dom';
ReactDOM.render(<App />, document.getElementById('root'));

// ✅ Modern API (React 18+)
import ReactDOM from 'react-dom/client';
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

**Production Considerations:**

```jsx
// Development vs Production rendering
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));

if (process.env.NODE_ENV === 'production') {
  // Production: No StrictMode for performance
  root.render(<App />);
} else {
  // Development: StrictMode for detecting issues
  root.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
}
```

**Comparison Table:**

| Aspect | React | ReactDOM |
|--------|-------|----------|
| **Package** | `react` | `react-dom` |
| **Purpose** | Component logic | DOM rendering |
| **Platform** | Agnostic | Web-specific |
| **Exports** | useState, useEffect, Component, createElement | createRoot, hydrateRoot, createPortal |
| **Depends on** | Nothing | React core |
| **Alternative** | - | React Native (mobile), React Three Fiber (3D) |

**Interview Tips:**
- Explain that React is the **platform-agnostic core**, ReactDOM is the **web renderer**
- Mention this design enables React to target multiple platforms (web, mobile, VR, canvas)
- Discuss the new `createRoot` API in React 18 vs legacy `render`
- Reference portals as a ReactDOM-specific feature
- Highlight that React Native uses `react-native` package instead of `react-dom`

</details>

---

### 3. What is JSX?

<details>
<summary>View Answer</summary>

**What is JSX?**

**JSX (JavaScript XML)** is a syntax extension for JavaScript that looks similar to HTML/XML but gets compiled to regular JavaScript function calls. It allows you to write UI structures in a declarative way within JavaScript code.

**JSX is NOT:**
- ❌ HTML inside JavaScript
- ❌ A template language
- ❌ Required to use React (but highly recommended)

**JSX IS:**
- ✅ Syntactic sugar for `React.createElement()`
- ✅ Compiled to JavaScript by Babel/TypeScript
- ✅ More readable and maintainable than vanilla JS

**JSX Compilation Process**

```jsx
// ✅ What you write (JSX)
const element = <h1 className="greeting">Hello, World!</h1>;

// 🔄 What Babel compiles it to (JavaScript)
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, World!'
);

// 📦 What actually renders in the browser
{
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, World!'
  }
}
```

**JSX Syntax Rules**

**1. Single Root Element**
```jsx
// ❌ Error: Multiple root elements
const Component = () => {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
};

// ✅ Solution 1: Wrapper div
const Component = () => {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
};

// ✅ Solution 2: React Fragment (Preferred)
const Component = () => {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
};

// ✅ Solution 3: Array (with keys)
const Component = () => {
  return [
    <h1 key="title">Title</h1>,
    <p key="para">Paragraph</p>
  ];
};
```

**2. className instead of class**
```jsx
// ❌ Wrong: 'class' is a reserved keyword in JavaScript
<div class="container">Content</div>

// ✅ Correct: Use 'className'
<div className="container">Content</div>

// ✅ Dynamic classes
const isActive = true;
<button className={isActive ? 'active' : 'inactive'}>
  Click me
</button>

// ✅ Multiple classes
<div className={`card ${isActive ? 'active' : ''} ${isLarge ? 'large' : ''}`}>
  Content
</div>
```

**3. Self-Closing Tags**
```jsx
// ❌ Wrong: Must be self-closed or have closing tag
<img src="image.jpg">
<input type="text">

// ✅ Correct: Self-closing
<img src="image.jpg" />
<input type="text" />
<MyComponent />
```

**4. JavaScript Expressions in JSX**
```jsx
const User = ({ name, age, isAdmin }) => {
  return (
    <div>
      {/* Variables */}
      <h1>{name}</h1>
      
      {/* Expressions */}
      <p>Age: {age + 5}</p>
      
      {/* Conditional (ternary) */}
      <span>{isAdmin ? 'Admin' : 'User'}</span>
      
      {/* Logical AND */}
      {isAdmin && <button>Delete</button>}
      
      {/* Function calls */}
      <p>{formatDate(new Date())}</p>
      
      {/* Template literals */}
      <p>{`Hello, ${name}!`}</p>
      
      {/* Arrays (will render all items) */}
      {['Item 1', 'Item 2', 'Item 3']}
      
      {/* ❌ Cannot use statements (if, for, while) */}
      {/* {if (isAdmin) { return <div>Admin</div> }} */}
      
      {/* ✅ Use IIFE for complex logic */}
      {(() => {
        if (isAdmin) return <div>Admin</div>;
        return <div>User</div>;
      })()}
    </div>
  );
};
```

**5. CamelCase for Attributes**
```jsx
// HTML attributes become camelCase in JSX
<button onclick="handleClick()">      // HTML
<button onClick={handleClick}>        // JSX

<div tabindex="0">                    // HTML
<div tabIndex="0">                    // JSX

<label for="input">                   // HTML
<label htmlFor="input">               // JSX

<div style="color: red;">             // HTML
<div style={{ color: 'red' }}>       // JSX (object)
```

**JSX vs React.createElement Comparison**

```jsx
// Complex JSX
const Profile = ({ user }) => (
  <div className="profile">
    <img src={user.avatar} alt={user.name} />
    <h2>{user.name}</h2>
    <p>{user.bio}</p>
    <button onClick={() => follow(user.id)}>Follow</button>
  </div>
);

// Equivalent React.createElement (verbose!)
const Profile = ({ user }) => 
  React.createElement(
    'div',
    { className: 'profile' },
    React.createElement('img', { 
      src: user.avatar, 
      alt: user.name 
    }),
    React.createElement('h2', null, user.name),
    React.createElement('p', null, user.bio),
    React.createElement(
      'button',
      { onClick: () => follow(user.id) },
      'Follow'
    )
  );
```

**JSX with Components**

```jsx
// Custom components must start with capital letter
// ❌ lowercase = HTML element
const mycomponent = () => <div>Hello</div>;
<mycomponent />  // Renders <mycomponent></mycomponent> (invalid HTML)

// ✅ PascalCase = React component
const MyComponent = () => <div>Hello</div>;
<MyComponent />  // Renders the component correctly

// Composition
const App = () => (
  <div>
    <Header />
    <Main>
      <Sidebar />
      <Content />
    </Main>
    <Footer />
  </div>
);
```

**JSX Spread Attributes**

```jsx
const Button = (props) => {
  return <button {...props}>Click me</button>;
};

// Usage
<Button onClick={handleClick} className="primary" disabled />

// Equivalent to:
<button onClick={handleClick} className="primary" disabled>
  Click me
</button>

// ⚠️ Be careful with spread - can cause unnecessary re-renders
// ✅ Better: Destructure what you need
const Button = ({ onClick, className, ...rest }) => {
  return (
    <button 
      onClick={onClick} 
      className={`btn ${className}`}
      {...rest}
    >
      Click me
    </button>
  );
};
```

**JSX Children Patterns**

```jsx
// 1. Text children
<div>Simple text</div>

// 2. Element children
<div>
  <h1>Title</h1>
  <p>Content</p>
</div>

// 3. Expression children
<div>{count}</div>

// 4. Function children (render props)
<DataProvider>
  {(data) => <div>{data.name}</div>}
</DataProvider>

// 5. Array children (must have keys)
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>

// 6. Mixed children
<div>
  Text node
  <span>Element</span>
  {variable}
  {array.map(item => <div key={item.id}>{item}</div>)}
</div>
```

**Inline Styles in JSX**

```jsx
// JSX requires style as an object, not string
// ❌ Wrong
<div style="color: red; font-size: 16px;">Text</div>

// ✅ Correct
<div style={{ color: 'red', fontSize: '16px' }}>Text</div>

// ✅ Better: Extract to variable
const buttonStyle = {
  backgroundColor: '#007bff',
  color: 'white',
  padding: '10px 20px',
  border: 'none',
  borderRadius: '4px',
  cursor: 'pointer',
};

<button style={buttonStyle}>Click me</button>

// ✅ Dynamic styles
const isDanger = true;
<button style={{
  backgroundColor: isDanger ? 'red' : 'blue',
  fontSize: '1rem',
}}>
  {isDanger ? 'Delete' : 'Save'}
</button>
```

**Production Best Practices**

```jsx
// ✅ 1. Always provide keys for lists
const TodoList = ({ todos }) => (
  <ul>
    {todos.map(todo => (
      <li key={todo.id}>{todo.text}</li>  // ✅ Unique id
    ))}
  </ul>
);

// ❌ Don't use index as key if list can change
{todos.map((todo, index) => (
  <li key={index}>{todo.text}</li>  // ❌ Can cause bugs
))}

// ✅ 2. Avoid inline functions for callbacks (prevents re-renders)
// ❌ Bad
<button onClick={() => handleClick(item.id)}>Delete</button>

// ✅ Good
const handleDelete = useCallback(() => {
  handleClick(item.id);
}, [item.id]);
<button onClick={handleDelete}>Delete</button>

// ✅ 3. Extract complex JSX into components
// ❌ Hard to read
<div>
  {isLoading ? (
    <div className="spinner">
      <div className="loader"></div>
    </div>
  ) : data ? (
    <div className="data">
      {data.map(item => (
        <div key={item.id}>
          {item.name}
        </div>
      ))}
    </div>
  ) : (
    <div>No data</div>
  )}
</div>

// ✅ Clean and maintainable
const DataDisplay = () => {
  if (isLoading) return <LoadingSpinner />;
  if (!data) return <EmptyState />;
  return <DataList items={data} />;
};
```

**Interview Tips:**
- Explain JSX is syntactic sugar for `React.createElement()`
- Mention Babel compiles JSX to JavaScript
- Highlight key differences from HTML (className, htmlFor, style object)
- Discuss why single root element is required (compiles to single function call)
- Mention that JSX prevents XSS attacks by escaping values by default
- Explain PascalCase convention for components vs lowercase for HTML elements

</details>

---

### 4. What are components in React?

<details>
<summary>View Answer</summary>

**What are Components?**

Components are the **building blocks** of a React application. They are independent, reusable pieces of UI that encapsulate their own structure, logic, and styling. Think of them as custom HTML elements.

**Key Characteristics:**
- **Reusable**: Write once, use multiple times
- **Composable**: Combine small components to build complex UIs
- **Encapsulated**: Manage their own state and logic
- **Declarative**: Describe what to render, not how to update

**Types of Components**

**1. Functional Components (Modern, Recommended)**

```jsx
// Simple functional component
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};

// With props
const Greeting = (props) => {
  return <h1>Hello, {props.name}!</h1>;
};

// With destructured props (preferred)
const UserCard = ({ name, email, avatar }) => {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
};

// With state (using hooks)
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
};
```

**2. Class Components (Legacy, Less Common Now)**

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, World!</h1>;
  }
}

// With props and state
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

**Component Composition**

```jsx
// Small, focused components
const Avatar = ({ url, alt }) => (
  <img src={url} alt={alt} className="avatar" />
);

const UserName = ({ name, verified }) => (
  <span className="username">
    {name}
    {verified && <VerifiedBadge />}
  </span>
);

const VerifiedBadge = () => (
  <span className="verified-badge">✓</span>
);

// Compose into larger component
const UserProfile = ({ user }) => (
  <div className="user-profile">
    <Avatar url={user.avatar} alt={user.name} />
    <UserName name={user.name} verified={user.verified} />
    <p className="bio">{user.bio}</p>
  </div>
);

// Usage
const App = () => {
  const user = {
    name: 'John Doe',
    avatar: '/avatar.jpg',
    verified: true,
    bio: 'Software Engineer'
  };
  
  return <UserProfile user={user} />;
};
```

**Component Props**

```jsx
// Props are read-only (immutable)
const Button = ({ label, onClick, variant = 'primary', disabled = false }) => {
  // ❌ Cannot modify props
  // label = 'New Label';  // Error in strict mode
  
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};

// Usage with different props
<Button label="Save" onClick={handleSave} />
<Button label="Cancel" onClick={handleCancel} variant="secondary" />
<Button label="Delete" onClick={handleDelete} variant="danger" disabled />
```

**Component State**

```jsx
// State is internal and mutable
const TodoList = () => {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { 
        id: Date.now(), 
        text: input,
        completed: false 
      }]);
      setInput('');
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
        placeholder="Add todo..."
      />
      <button onClick={addTodo}>Add</button>
      
      <ul>
        {todos.map(todo => (
          <li 
            key={todo.id}
            onClick={() => toggleTodo(todo.id)}
            style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none' 
            }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

**Presentational vs Container Components**

```jsx
// 1. Presentational Component (UI only, no logic)
const ProductCard = ({ product, onAddToCart }) => (
  <div className="product-card">
    <img src={product.image} alt={product.name} />
    <h3>{product.name}</h3>
    <p className="price">${product.price}</p>
    <button onClick={() => onAddToCart(product)}>
      Add to Cart
    </button>
  </div>
);

// 2. Container Component (logic, data fetching)
const ProductListContainer = () => {
  const [products, setProducts] = useState([]);
  const [cart, setCart] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchProducts()
      .then(data => setProducts(data))
      .finally(() => setLoading(false));
  }, []);
  
  const handleAddToCart = (product) => {
    setCart([...cart, product]);
    toast.success(`${product.name} added to cart`);
  };
  
  if (loading) return <LoadingSpinner />;
  
  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
};
```

**Higher-Order Components (HOC)**

```jsx
// HOC: A function that takes a component and returns a new component
const withAuth = (Component) => {
  return (props) => {
    const { user } = useAuth();
    
    if (!user) {
      return <Navigate to="/login" />;
    }
    
    return <Component {...props} user={user} />;
  };
};

// Usage
const Dashboard = ({ user }) => (
  <div>
    <h1>Welcome, {user.name}!</h1>
    {/* Dashboard content */}
  </div>
);

const ProtectedDashboard = withAuth(Dashboard);

// In routing
<Route path="/dashboard" element={<ProtectedDashboard />} />
```

**Render Props Pattern**

```jsx
// Component with render prop
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
};

// Usage
<MouseTracker 
  render={({ x, y }) => (
    <div>
      Mouse position: {x}, {y}
    </div>
  )}
/>

// Or with children prop
const MouseTracker2 = ({ children }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  // ... same logic
  
  return children(position);
};

<MouseTracker2>
  {({ x, y }) => <div>Mouse: {x}, {y}</div>}
</MouseTracker2>
```

**Compound Components Pattern**

```jsx
// Parent component with shared state
const Tabs = ({ children, defaultTab = 0 }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          isActive: index === activeTab,
          onClick: () => setActiveTab(index),
          index,
        })
      )}
    </div>
  );
};

// Child components
const Tab = ({ label, isActive, onClick }) => (
  <button
    className={`tab ${isActive ? 'active' : ''}`}
    onClick={onClick}
  >
    {label}
  </button>
);

const TabPanel = ({ children, isActive }) => (
  isActive ? <div className="tab-panel">{children}</div> : null
);

// Usage
<Tabs defaultTab={0}>
  <Tab label="Profile" />
  <Tab label="Settings" />
  <Tab label="Logout" />
</Tabs>
```

**Component Lifecycle (Functional Components with Hooks)**

```jsx
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // componentDidMount + componentDidUpdate (when userId changes)
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(data => setUser(data))
      .finally(() => setLoading(false));
  }, [userId]); // Dependency array
  
  // componentWillUnmount (cleanup)
  useEffect(() => {
    const subscription = subscribeToUserUpdates(userId, setUser);
    
    return () => {
      // Cleanup function (runs on unmount or before re-running effect)
      subscription.unsubscribe();
    };
  }, [userId]);
  
  if (loading) return <Spinner />;
  if (!user) return <NotFound />;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

**Production-Grade Component Example**

```jsx
import { useState, useEffect, useCallback, memo } from 'react';
import PropTypes from 'prop-types';

/**
 * ProductCard - Displays product information with add to cart functionality
 * 
 * @param {Object} product - Product data
 * @param {Function} onAddToCart - Callback when item added to cart
 * @param {boolean} isInCart - Whether product is already in cart
 * @param {boolean} disabled - Disable add to cart button
 */
const ProductCard = memo(({ 
  product, 
  onAddToCart, 
  isInCart = false,
  disabled = false 
}) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [isHovered, setIsHovered] = useState(false);
  
  // Memoize handler to prevent re-renders
  const handleAddToCart = useCallback(() => {
    if (!disabled && !isInCart) {
      onAddToCart(product);
    }
  }, [product, onAddToCart, disabled, isInCart]);
  
  // Format price
  const formattedPrice = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(product.price);
  
  return (
    <article 
      className="product-card"
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      aria-label={`Product: ${product.name}`}
    >
      <div className="product-image">
        {!imageLoaded && <Skeleton />}
        <img
          src={product.image}
          alt={product.name}
          onLoad={() => setImageLoaded(true)}
          loading="lazy"
        />
        {isHovered && product.discount && (
          <span className="discount-badge">
            {product.discount}% OFF
          </span>
        )}
      </div>
      
      <div className="product-info">
        <h3>{product.name}</h3>
        <p className="description">{product.description}</p>
        
        <div className="price-section">
          {product.originalPrice && (
            <span className="original-price">
              ${product.originalPrice}
            </span>
          )}
          <span className="price">{formattedPrice}</span>
        </div>
        
        <button
          onClick={handleAddToCart}
          disabled={disabled || isInCart}
          className={`add-to-cart ${isInCart ? 'in-cart' : ''}`}
          aria-label={isInCart ? 'Already in cart' : 'Add to cart'}
        >
          {isInCart ? '✓ In Cart' : 'Add to Cart'}
        </button>
      </div>
    </article>
  );
});

// PropTypes for type checking
ProductCard.propTypes = {
  product: PropTypes.shape({
    id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]).isRequired,
    name: PropTypes.string.isRequired,
    price: PropTypes.number.isRequired,
    image: PropTypes.string.isRequired,
    description: PropTypes.string,
    discount: PropTypes.number,
    originalPrice: PropTypes.number,
  }).isRequired,
  onAddToCart: PropTypes.func.isRequired,
  isInCart: PropTypes.bool,
  disabled: PropTypes.bool,
};

ProductCard.displayName = 'ProductCard';

export default ProductCard;
```

**Interview Tips:**
- Components are the fundamental building blocks of React applications
- Mention functional components with hooks are now the standard (post-2019)
- Discuss component composition and the "thinking in React" philosophy
- Explain props flow down (unidirectional data flow), events flow up
- Highlight patterns: Presentational/Container, HOC, Render Props, Compound Components
- Mention React.memo for performance optimization
- Discuss component naming conventions (PascalCase)

</details>

---

### 5. What is the difference between functional and class components?

<details>
<summary>View Answer</summary>

**Functional vs Class Components**

React has evolved from class-based components to functional components with hooks. While both can achieve the same results, the approach and syntax differ significantly.

**Comparison Table**

| Aspect | Functional Components | Class Components |
|--------|----------------------|------------------|
| **Syntax** | JavaScript function | ES6 class |
| **State** | `useState` hook | `this.state` |
| **Lifecycle** | `useEffect` hook | Lifecycle methods |
| **`this` binding** | Not needed | Required |
| **Performance** | Slightly better (no instance) | Slightly slower |
| **Code length** | Shorter, cleaner | More verbose |
| **Learning curve** | Easier | Steeper |
| **Recommended** | ✅ Yes (Modern) | ❌ Legacy |
| **Released** | React 16.8 (2019) | React 0.14 (2015) |

**1. Basic Syntax**

```jsx
// ✅ Functional Component (Modern)
const Welcome = (props) => {
  return <h1>Hello, {props.name}!</h1>;
};

// Or with implicit return
const Welcome = ({ name }) => <h1>Hello, {name}!</h1>;

// ❌ Class Component (Legacy)
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

**2. State Management**

```jsx
// ✅ Functional Component with useState
const Counter = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');
  
  const increment = () => setCount(count + 1);
  const updateName = (newName) => setName(newName);
  
  return (
    <div>
      <p>{name}: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

// ❌ Class Component with this.state
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      name: 'John'
    };
    // Must bind methods
    this.increment = this.increment.bind(this);
  }
  
  increment() {
    this.setState({ count: this.state.count + 1 });
  }
  
  updateName = (newName) => {
    this.setState({ name: newName });
  };
  
  render() {
    return (
      <div>
        <p>{this.state.name}: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

**3. Lifecycle Methods**

```jsx
// ✅ Functional Component with useEffect
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  // componentDidMount + componentDidUpdate
  useEffect(() => {
    console.log('Component mounted or userId changed');
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // componentWillUnmount (cleanup)
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Polling...');
    }, 5000);
    
    return () => {
      console.log('Cleanup: clearing interval');
      clearInterval(timer);
    };
  }, []);
  
  // Runs on every render (rare use case)
  useEffect(() => {
    console.log('Runs after every render');
  });
  
  return <div>{user?.name}</div>;
};

// ❌ Class Component with lifecycle methods
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = { user: null };
    this.timer = null;
  }
  
  componentDidMount() {
    console.log('Component mounted');
    fetchUser(this.props.userId).then(user => {
      this.setState({ user });
    });
    
    this.timer = setInterval(() => {
      console.log('Polling...');
    }, 5000);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      console.log('userId changed');
      fetchUser(this.props.userId).then(user => {
        this.setState({ user });
      });
    }
  }
  
  componentWillUnmount() {
    console.log('Cleanup: clearing interval');
    clearInterval(this.timer);
  }
  
  render() {
    return <div>{this.state.user?.name}</div>;
  }
}
```

**4. Event Handlers and `this` Binding**

```jsx
// ✅ Functional Component - No 'this' binding needed
const Form = () => {
  const [value, setValue] = useState('');
  
  // Arrow functions automatically have correct context
  const handleChange = (e) => {
    setValue(e.target.value);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
};

// ❌ Class Component - 'this' binding required
class Form extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: '' };
    
    // Method 1: Bind in constructor
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }
  
  // Method 2: Arrow function (class property)
  handleChange = (e) => {
    this.setState({ value: e.target.value });
  };
  
  handleSubmit(e) {
    e.preventDefault();
    console.log('Submitted:', this.state.value);
  }
  
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input 
          value={this.state.value} 
          onChange={this.handleChange} 
        />
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

**5. Multiple State Values**

```jsx
// ✅ Functional Component - Multiple useState calls
const UserForm = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  // Each state is independent
  return (
    <form>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input value={age} onChange={(e) => setAge(e.target.value)} />
    </form>
  );
};

// ❌ Class Component - Single state object
class UserForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: '',
      email: '',
      age: 0
    };
  }
  
  // Must spread existing state to avoid overwriting
  updateName = (e) => {
    this.setState({ name: e.target.value });
  };
  
  updateEmail = (e) => {
    this.setState({ email: e.target.value });
  };
  
  render() {
    return (
      <form>
        <input value={this.state.name} onChange={this.updateName} />
        <input value={this.state.email} onChange={this.updateEmail} />
      </form>
    );
  }
}
```

**6. Custom Hooks (Functional Only)**

```jsx
// ✅ Custom hook - Logic reuse (Functional only)
const useWindowWidth = () => {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return width;
};

// Usage in functional component
const ResponsiveComponent = () => {
  const width = useWindowWidth();
  
  return <div>Window width: {width}px</div>;
};

// ❌ Class Component - Cannot use hooks
// Must use HOC or render props for logic reuse
class ResponsiveComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { width: window.innerWidth };
  }
  
  componentDidMount() {
    window.addEventListener('resize', this.handleResize);
  }
  
  componentWillUnmount() {
    window.removeEventListener('resize', this.handleResize);
  }
  
  handleResize = () => {
    this.setState({ width: window.innerWidth });
  };
  
  render() {
    return <div>Window width: {this.state.width}px</div>;
  }
}
```

**7. Complex Example: Data Fetching**

```jsx
// ✅ Functional Component
const UserList = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('/api/users');
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUsers();
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
};

// ❌ Class Component
class UserList extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
      loading: true,
      error: null
    };
  }
  
  async componentDidMount() {
    try {
      this.setState({ loading: true });
      const response = await fetch('/api/users');
      const data = await response.json();
      this.setState({ users: data, loading: false });
    } catch (err) {
      this.setState({ error: err.message, loading: false });
    }
  }
  
  render() {
    const { users, loading, error } = this.state;
    
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
}
```

**8. Performance Optimization**

```jsx
// ✅ Functional Component
const ExpensiveComponent = memo(({ data, onUpdate }) => {
  // useMemo for expensive calculations
  const processedData = useMemo(() => {
    return data.map(item => expensiveOperation(item));
  }, [data]);
  
  // useCallback for function memoization
  const handleClick = useCallback(() => {
    onUpdate(data);
  }, [data, onUpdate]);
  
  return (
    <div onClick={handleClick}>
      {processedData.map(item => <div key={item.id}>{item.value}</div>)}
    </div>
  );
});

// ❌ Class Component
class ExpensiveComponent extends React.PureComponent {
  // PureComponent does shallow comparison of props
  
  processData = () => {
    return this.props.data.map(item => expensiveOperation(item));
  };
  
  handleClick = () => {
    this.props.onUpdate(this.props.data);
  };
  
  render() {
    const processedData = this.processData();
    
    return (
      <div onClick={this.handleClick}>
        {processedData.map(item => <div key={item.id}>{item.value}</div>)}
      </div>
    );
  }
}
```

**When to Use Each (Current Guidelines)**

```jsx
// ✅ ALWAYS use Functional Components (2025 Standard)
// - Cleaner syntax
// - Hooks for state and lifecycle
// - Custom hooks for logic reuse
// - Better performance (no class instance)
// - Easier to test
// - Future of React

const ModernComponent = () => {
  const [state, setState] = useState(initialState);
  
  useEffect(() => {
    // Side effects
  }, []);
  
  return <div>Modern React</div>;
};

// ❌ Only use Class Components if:
// 1. Working with legacy codebase
// 2. Error boundaries (until React adds hooks support)
class ErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Migration Path: Class to Functional**

```jsx
// Before: Class Component
class OldComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, name: 'John' };
  }
  
  componentDidMount() {
    document.title = `${this.state.name}: ${this.state.count}`;
  }
  
  componentDidUpdate() {
    document.title = `${this.state.name}: ${this.state.count}`;
  }
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  render() {
    return (
      <div>
        <p>{this.state.name}: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

// After: Functional Component
const NewComponent = () => {
  const [count, setCount] = useState(0);
  const [name] = useState('John');
  
  useEffect(() => {
    document.title = `${name}: ${count}`;
  }, [name, count]); // Combines componentDidMount and componentDidUpdate
  
  const increment = () => setCount(count + 1);
  
  return (
    <div>
      <p>{name}: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};
```

**Interview Tips:**
- Functional components with hooks are the **modern standard** (since 2019)
- Class components are now considered **legacy** but still supported
- Hooks solve problems: logic reuse, complex lifecycle methods, `this` binding confusion
- Only exception: Error boundaries still require class components
- Mention performance benefits of functional components (no class instance overhead)
- Discuss how hooks make code more readable and testable
- If working on legacy project, might see class components, but new code should use functional

</details>

---

### 6. What are props in React?

<details>
<summary>View Answer</summary>

**What are Props?**

**Props (Properties)** are the mechanism for passing data from parent components to child components in React. They are **read-only** (immutable) and flow in **one direction** (top-down/unidirectional data flow).

**Key Characteristics:**
- **Read-Only**: Cannot be modified by the receiving component
- **Unidirectional**: Flow from parent to child only
- **Any Data Type**: Can pass strings, numbers, objects, arrays, functions, even components
- **Required for Communication**: Primary way components communicate

**Basic Props Usage**

```jsx
// Parent component passes props
const App = () => {
  return (
    <Greeting 
      name="John" 
      age={30} 
      isAdmin={true}
    />
  );
};

// Child component receives props
const Greeting = (props) => {
  return (
    <div>
      <h1>Hello, {props.name}!</h1>
      <p>Age: {props.age}</p>
      <p>Role: {props.isAdmin ? 'Admin' : 'User'}</p>
    </div>
  );
};

// ✅ Better: Destructure props (Recommended)
const Greeting = ({ name, age, isAdmin }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Age: {age}</p>
      <p>Role: {isAdmin ? 'Admin' : 'User'}</p>
    </div>
  );
};
```

**Props are Immutable**

```jsx
const Button = ({ label }) => {
  // ❌ WRONG: Cannot modify props
  // label = 'New Label'; // Error in strict mode
  // Props are read-only!
  
  // ✅ CORRECT: Use props as-is
  return <button>{label}</button>;
  
  // ✅ Or create local state if you need to modify
  const [buttonLabel, setButtonLabel] = useState(label);
  return <button onClick={() => setButtonLabel('Clicked')}>{buttonLabel}</button>;
};
```

**Different Types of Props**

```jsx
const UserProfile = ({
  // 1. String props
  name,
  email,
  
  // 2. Number props
  age,
  score,
  
  // 3. Boolean props
  isActive,
  isAdmin,
  
  // 4. Array props
  hobbies,
  
  // 5. Object props
  address,
  
  // 6. Function props (callbacks)
  onEdit,
  onDelete,
  
  // 7. JSX/Component props
  icon,
  children,
}) => {
  return (
    <div className="user-profile">
      {/* String & Number */}
      <h2>{name} ({age})</h2>
      <p>{email}</p>
      
      {/* Boolean */}
      {isActive && <span className="badge">Active</span>}
      
      {/* Array */}
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
      
      {/* Object */}
      <p>{address.city}, {address.country}</p>
      
      {/* Functions */}
      <button onClick={() => onEdit(name)}>Edit</button>
      <button onClick={() => onDelete(name)}>Delete</button>
      
      {/* JSX/Component */}
      <div className="icon">{icon}</div>
      
      {/* Children */}
      <div className="content">{children}</div>
    </div>
  );
};

// Usage
<UserProfile
  name="John Doe"
  email="john@example.com"
  age={30}
  score={95.5}
  isActive={true}
  isAdmin={false}
  hobbies={['Reading', 'Gaming', 'Coding']}
  address={{ city: 'New York', country: 'USA' }}
  onEdit={(name) => console.log(`Editing ${name}`)}
  onDelete={(name) => console.log(`Deleting ${name}`)}
  icon={<StarIcon />}
>
  <p>This is additional content passed as children</p>
</UserProfile>
```

**Default Props**

```jsx
// Method 1: Default parameters (Modern, Recommended)
const Button = ({ 
  label = 'Click Me',
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick = () => {}
}) => {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
};

// Method 2: defaultProps (Legacy, but still valid)
const Button = ({ label, variant, size, disabled, onClick }) => {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
};

Button.defaultProps = {
  label: 'Click Me',
  variant: 'primary',
  size: 'medium',
  disabled: false,
  onClick: () => {}
};

// Usage - missing props will use defaults
<Button />
<Button label="Submit" variant="success" />
```

**Props Validation with PropTypes**

```jsx
import PropTypes from 'prop-types';

const UserCard = ({ name, age, email, avatar, isActive, onDelete }) => {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      {isActive && <span>Active</span>}
      <button onClick={onDelete}>Delete</button>
    </div>
  );
};

// PropTypes validation (Development mode warnings)
UserCard.propTypes = {
  name: PropTypes.string.isRequired,      // Required string
  age: PropTypes.number.isRequired,       // Required number
  email: PropTypes.string.isRequired,     // Required string
  avatar: PropTypes.string,               // Optional string
  isActive: PropTypes.bool,               // Optional boolean
  onDelete: PropTypes.func.isRequired,    // Required function
};

UserCard.defaultProps = {
  avatar: '/default-avatar.png',
  isActive: false,
};

// Advanced PropTypes
const Product = ({ product, onUpdate }) => { /* ... */ };

Product.propTypes = {
  product: PropTypes.shape({              // Object with specific shape
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    price: PropTypes.number.isRequired,
    category: PropTypes.oneOf(['Electronics', 'Clothing', 'Food']), // Enum
    tags: PropTypes.arrayOf(PropTypes.string), // Array of strings
  }).isRequired,
  
  onUpdate: PropTypes.func.isRequired,
};
```

**Children Prop (Special Prop)**

```jsx
// Children can be anything: text, elements, components, functions
const Card = ({ children, title }) => {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
};

// Usage 1: Text children
<Card title="Simple">
  Just some text content
</Card>

// Usage 2: Element children
<Card title="Complex">
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <button>Action</button>
</Card>

// Usage 3: Component children
<Card title="Components">
  <UserProfile user={userData} />
  <Comments comments={commentData} />
</Card>

// Usage 4: Function children (Render Props)
<Card title="Function">
  {(data) => <div>Rendered: {data}</div>}
</Card>
```

**Passing Functions as Props (Callbacks)**

```jsx
// Parent component
const TodoApp = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build App', completed: false },
  ]);
  
  // Functions passed as props
  const handleToggle = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const handleDelete = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}    // Function prop
          onDelete={handleDelete}    // Function prop
        />
      ))}
    </div>
  );
};

// Child component
const TodoItem = ({ todo, onToggle, onDelete }) => {
  return (
    <div className="todo-item">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)} // Call parent function
      />
      <span style={{ 
        textDecoration: todo.completed ? 'line-through' : 'none' 
      }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
};
```

**Spread Props Pattern**

```jsx
// Pass all props to child component
const Wrapper = (props) => {
  return <ChildComponent {...props} />;
};

// Better: Destructure specific props, spread the rest
const Wrapper = ({ className, style, ...rest }) => {
  return (
    <div className={`wrapper ${className}`} style={style}>
      <ChildComponent {...rest} />
    </div>
  );
};

// Usage
<Wrapper 
  className="custom"
  style={{ margin: 10 }}
  title="Hello"
  onClick={handleClick}
  data={myData}
/>
// className and style go to wrapper div
// title, onClick, data spread to ChildComponent
```

**Props Destructuring Patterns**

```jsx
// 1. Basic destructuring
const User = ({ name, age }) => (
  <div>{name} is {age} years old</div>
);

// 2. Destructuring with defaults
const User = ({ name = 'Anonymous', age = 0 }) => (
  <div>{name} is {age} years old</div>
);

// 3. Nested destructuring
const User = ({ user: { name, age, address: { city } } }) => (
  <div>{name}, {age}, from {city}</div>
);

// 4. Rest properties
const Button = ({ label, onClick, ...restProps }) => (
  <button onClick={onClick} {...restProps}>
    {label}
  </button>
);

// 5. Renaming while destructuring
const User = ({ name: userName, age: userAge }) => (
  <div>{userName} is {userAge} years old</div>
);
```

**Props vs Attributes**

```jsx
// HTML attributes (lowercase)
<input type="text" value="hello" class="input" />

// React props (camelCase)
<Input 
  type="text" 
  value="hello" 
  className="input"     // className instead of class
  onChange={handleChange}
  disabled={false}
/>

// Custom component props (any name you want)
<UserProfile
  userName="John"       // Custom prop name
  userAge={30}          // Custom prop name
  isVerified={true}     // Custom prop name
/>
```

**Common Props Patterns**

```jsx
// 1. Configuration props
<DataTable
  columns={['Name', 'Age', 'Email']}
  data={users}
  sortable={true}
  filterable={true}
  pageSize={10}
/>

// 2. Callback props (event handlers)
<Form
  onSubmit={handleSubmit}
  onCancel={handleCancel}
  onValidate={handleValidate}
  onChange={handleChange}
/>

// 3. Render props
<DataProvider
  render={(data) => <UserList users={data} />}
/>

// 4. Component props (pass components as props)
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  footer={<Footer />}
/>

// 5. Boolean props (shorthand)
<Button 
  disabled        // Same as disabled={true}
  loading
  primary
/>
```

**Production Example: Complete Component**

```jsx
import { memo, useCallback } from 'react';
import PropTypes from 'prop-types';

/**
 * ProductCard - Displays product information with purchase actions
 * 
 * @param {Object} product - Product data object
 * @param {Function} onAddToCart - Callback when adding to cart
 * @param {Function} onViewDetails - Callback when viewing details
 * @param {boolean} isInCart - Whether product is in cart
 * @param {boolean} isWishlisted - Whether product is wishlisted
 * @param {Function} onToggleWishlist - Callback for wishlist toggle
 * @param {string} currency - Currency symbol
 * @param {string} className - Additional CSS classes
 */
const ProductCard = memo(({
  product,
  onAddToCart,
  onViewDetails,
  isInCart = false,
  isWishlisted = false,
  onToggleWishlist,
  currency = '$',
  className = '',
}) => {
  // Memoized handlers
  const handleAddToCart = useCallback(() => {
    onAddToCart(product.id);
  }, [product.id, onAddToCart]);
  
  const handleViewDetails = useCallback(() => {
    onViewDetails(product.id);
  }, [product.id, onViewDetails]);
  
  const handleWishlistToggle = useCallback(() => {
    onToggleWishlist(product.id);
  }, [product.id, onToggleWishlist]);
  
  // Format price
  const formattedPrice = `${currency}${product.price.toFixed(2)}`;
  const hasDiscount = product.originalPrice > product.price;
  
  return (
    <article className={`product-card ${className}`}>
      <div className="product-image">
        <img 
          src={product.image} 
          alt={product.name}
          loading="lazy"
        />
        
        {/* Wishlist button */}
        <button
          className={`wishlist-btn ${isWishlisted ? 'active' : ''}`}
          onClick={handleWishlistToggle}
          aria-label={isWishlisted ? 'Remove from wishlist' : 'Add to wishlist'}
        >
          {isWishlisted ? '❤️' : '🤍'}
        </button>
        
        {/* Discount badge */}
        {hasDiscount && (
          <span className="discount-badge">
            {Math.round((1 - product.price / product.originalPrice) * 100)}% OFF
          </span>
        )}
      </div>
      
      <div className="product-info">
        <h3 className="product-name">{product.name}</h3>
        <p className="product-description">{product.description}</p>
        
        <div className="product-rating">
          {'⭐'.repeat(Math.floor(product.rating))}
          <span className="rating-count">({product.reviewCount})</span>
        </div>
        
        <div className="product-price">
          {hasDiscount && (
            <span className="original-price">
              {currency}{product.originalPrice.toFixed(2)}
            </span>
          )}
          <span className="current-price">{formattedPrice}</span>
        </div>
        
        <div className="product-actions">
          <button
            className={`add-to-cart-btn ${isInCart ? 'in-cart' : ''}`}
            onClick={handleAddToCart}
            disabled={isInCart || !product.inStock}
          >
            {isInCart ? '✓ In Cart' : product.inStock ? 'Add to Cart' : 'Out of Stock'}
          </button>
          
          <button
            className="view-details-btn"
            onClick={handleViewDetails}
          >
            View Details
          </button>
        </div>
      </div>
    </article>
  );
});

// PropTypes validation
ProductCard.propTypes = {
  product: PropTypes.shape({
    id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]).isRequired,
    name: PropTypes.string.isRequired,
    description: PropTypes.string,
    image: PropTypes.string.isRequired,
    price: PropTypes.number.isRequired,
    originalPrice: PropTypes.number,
    rating: PropTypes.number,
    reviewCount: PropTypes.number,
    inStock: PropTypes.bool,
  }).isRequired,
  onAddToCart: PropTypes.func.isRequired,
  onViewDetails: PropTypes.func.isRequired,
  onToggleWishlist: PropTypes.func,
  isInCart: PropTypes.bool,
  isWishlisted: PropTypes.bool,
  currency: PropTypes.string,
  className: PropTypes.string,
};

ProductCard.displayName = 'ProductCard';

export default ProductCard;
```

**Props Best Practices**

```jsx
// ✅ 1. Use descriptive prop names
<Button onClick={handleSubmit} isLoading={loading} variant="primary" />

// ❌ Avoid unclear names
<Button click={handleSubmit} load={loading} type="primary" />

// ✅ 2. Destructure props for readability
const Component = ({ title, description, onClick }) => { /* ... */ };

// ❌ Avoid accessing via props object everywhere
const Component = (props) => {
  return <div>{props.title} {props.description}</div>;
};

// ✅ 3. Provide default values
const Button = ({ label = 'Click', onClick = () => {} }) => { /* ... */ };

// ✅ 4. Use PropTypes or TypeScript for validation
Component.propTypes = { /* ... */ };

// ✅ 5. Memoize function props to prevent unnecessary re-renders
const handleClick = useCallback(() => { /* ... */ }, [dependency]);
<Child onClick={handleClick} />

// ❌ Avoid creating new functions on every render
<Child onClick={() => doSomething()} /> // Creates new function each render
```

**Interview Tips:**
- Props are **read-only** and enable **unidirectional data flow** (parent → child)
- Props can be any JavaScript value: primitives, objects, arrays, functions, JSX
- Mention **children** is a special prop for component composition
- Discuss **PropTypes** for runtime validation (or TypeScript for compile-time)
- Explain props vs state: props are external (passed in), state is internal (managed within)
- Highlight that function props (callbacks) allow child-to-parent communication
- Mention React.memo and useCallback for optimization with props

</details>

---

### 7. What is state in React?

<details>
<summary>View Answer</summary>

**What is State?**

**State** is an internal data storage mechanism for React components. Unlike props (which are passed from parent), state is **owned and managed** by the component itself. When state changes, React automatically re-renders the component.

**Key Characteristics:**
- **Mutable**: Can be changed within the component
- **Internal**: Owned by the component (not passed from parent)
- **Private**: Not accessible from outside the component
- **Triggers Re-renders**: State updates cause component to re-render
- **Asynchronous**: State updates may be batched for performance

**State in Functional Components (Hooks)**

```jsx
import { useState } from 'react';

const Counter = () => {
  // useState hook: [currentValue, updaterFunction]
  const [count, setCount] = useState(0); // Initial state: 0
  
  const increment = () => {
    setCount(count + 1); // Update state
  };
  
  const decrement = () => {
    setCount(count - 1);
  };
  
  const reset = () => {
    setCount(0);
  };
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

**Multiple State Variables**

```jsx
const UserForm = () => {
  // Multiple independent state variables
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  const [isSubscribed, setIsSubscribed] = useState(false);
  
  return (
    <form>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input 
        type="number"
        value={age} 
        onChange={(e) => setAge(Number(e.target.value))}
        placeholder="Age"
      />
      <label>
        <input 
          type="checkbox"
          checked={isSubscribed}
          onChange={(e) => setIsSubscribed(e.target.checked)}
        />
        Subscribe to newsletter
      </label>
    </form>
  );
};
```

**State with Objects**

```jsx
const UserProfile = () => {
  // Object state
  const [user, setUser] = useState({
    name: 'John',
    age: 30,
    email: 'john@example.com',
    address: {
      city: 'New York',
      country: 'USA'
    }
  });
  
  // ❌ WRONG: Mutating state directly (doesn't trigger re-render)
  const updateNameWrong = () => {
    user.name = 'Jane'; // Don't do this!
    setUser(user); // Won't work properly
  };
  
  // ✅ CORRECT: Create new object with spread operator
  const updateName = () => {
    setUser({
      ...user,           // Copy all existing properties
      name: 'Jane'       // Override specific property
    });
  };
  
  // ✅ Update nested object property
  const updateCity = () => {
    setUser({
      ...user,
      address: {
        ...user.address,
        city: 'Los Angeles'
      }
    });
  };
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Age: {user.age}</p>
      <p>Email: {user.email}</p>
      <p>Location: {user.address.city}, {user.address.country}</p>
      <button onClick={updateName}>Change Name</button>
      <button onClick={updateCity}>Change City</button>
    </div>
  );
};
```

**State with Arrays**

```jsx
const TodoList = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build Project', completed: false },
  ]);
  const [input, setInput] = useState('');
  
  // ✅ Add item to array
  const addTodo = () => {
    if (input.trim()) {
      setTodos([
        ...todos,
        { id: Date.now(), text: input, completed: false }
      ]);
      setInput('');
    }
  };
  
  // ✅ Remove item from array
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  // ✅ Update item in array
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  // ✅ Clear all items
  const clearCompleted = () => {
    setTodos(todos.filter(todo => !todo.completed));
  };
  
  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add</button>
      <button onClick={clearCompleted}>Clear Completed</button>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

**Functional State Updates**

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  // ❌ PROBLEM: May not work correctly with rapid updates
  const incrementWrong = () => {
    setCount(count + 1);
    setCount(count + 1); // Uses same old value, only increments by 1
  };
  
  // ✅ SOLUTION: Use functional update (previous state)
  const incrementCorrect = () => {
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1); // Now increments by 2
  };
  
  // ✅ Use when new state depends on old state
  const incrementMultiple = () => {
    // Increments by 3
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
  };
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={incrementCorrect}>Increment</button>
    </div>
  );
};
```

**Lazy Initial State**

```jsx
// ❌ BAD: Expensive calculation runs on every render
const Component = () => {
  const [data, setData] = useState(expensiveCalculation());
  // expensiveCalculation() runs every render!
};

// ✅ GOOD: Lazy initialization (only runs once)
const Component = () => {
  const [data, setData] = useState(() => expensiveCalculation());
  // Function only called on initial render
  
  // Or with localStorage
  const [user, setUser] = useState(() => {
    const saved = localStorage.getItem('user');
    return saved ? JSON.parse(saved) : null;
  });
};

// Real example
const DataTable = () => {
  const [data, setData] = useState(() => {
    // Only runs once on mount
    console.log('Processing large dataset...');
    return processLargeDataset(rawData);
  });
};
```

**State vs Props Comparison**

```jsx
// Props: Data passed FROM PARENT
const ChildComponent = ({ title, count, onIncrement }) => {
  // ❌ Cannot modify props
  // title = 'New Title'; // Error
  
  return (
    <div>
      <h1>{title}</h1>
      <p>Count from parent: {count}</p>
      <button onClick={onIncrement}>Increment in Parent</button>
    </div>
  );
};

// State: Data managed WITHIN COMPONENT
const ParentComponent = () => {
  const [count, setCount] = useState(0); // Internal state
  
  const increment = () => {
    setCount(count + 1); // Can modify own state
  };
  
  return (
    <ChildComponent 
      title="Counter App"    // Prop (read-only for child)
      count={count}          // Prop (derived from parent state)
      onIncrement={increment} // Prop (callback)
    />
  );
};
```

**Comparison Table: State vs Props**

| Aspect | State | Props |
|--------|-------|-------|
| **Ownership** | Owned by component | Passed from parent |
| **Mutability** | Mutable (within component) | Immutable (read-only) |
| **Control** | Controlled by component | Controlled by parent |
| **Changes** | Can be changed using setState | Cannot be changed |
| **Purpose** | Internal data management | Component communication |
| **Triggers Re-render** | Yes (when updated) | Yes (when parent updates) |

**State Batching (React 18+)**

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  // React 18: Automatic batching
  const handleClick = () => {
    setCount(count + 1);  // Doesn't re-render yet
    setFlag(!flag);       // Doesn't re-render yet
    // React batches these updates → Single re-render
  };
  
  // Even in async functions (React 18+)
  const handleAsyncClick = async () => {
    await fetchData();
    setCount(count + 1);  // Batched
    setFlag(!flag);       // Batched → Single re-render
  };
  
  // Force synchronous update (rare, use sparingly)
  const handleSyncUpdate = () => {
    flushSync(() => {
      setCount(count + 1); // Immediate re-render
    });
    console.log(count); // Will reflect updated value
  };
};
```

**Common State Patterns**

```jsx
// 1. Toggle State
const [isOpen, setIsOpen] = useState(false);
const toggle = () => setIsOpen(prev => !prev);

// 2. Loading State
const [isLoading, setIsLoading] = useState(false);
const [data, setData] = useState(null);
const [error, setError] = useState(null);

useEffect(() => {
  const fetchData = async () => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/data');
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  fetchData();
}, []);

// 3. Form State
const [formData, setFormData] = useState({
  username: '',
  password: '',
  email: ''
});

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: value
  }));
};

// 4. Multi-step State
const [step, setStep] = useState(1);
const nextStep = () => setStep(prev => prev + 1);
const prevStep = () => setStep(prev => prev - 1);

// 5. Selected Items State (Set pattern)
const [selectedIds, setSelectedIds] = useState(new Set());

const toggleSelection = (id) => {
  setSelectedIds(prev => {
    const newSet = new Set(prev);
    if (newSet.has(id)) {
      newSet.delete(id);
    } else {
      newSet.add(id);
    }
    return newSet;
  });
};
```

**State Lifting (Shared State)**

```jsx
// Lift state UP to nearest common ancestor
const ParentComponent = () => {
  // State lifted to parent
  const [sharedData, setSharedData] = useState('');
  
  return (
    <div>
      <ChildA data={sharedData} onChange={setSharedData} />
      <ChildB data={sharedData} />
    </div>
  );
};

const ChildA = ({ data, onChange }) => (
  <input value={data} onChange={(e) => onChange(e.target.value)} />
);

const ChildB = ({ data }) => (
  <div>You typed: {data}</div>
);
```

**Production Example: Complex State Management**

```jsx
import { useState, useCallback, useEffect } from 'react';

const ShoppingCart = () => {
  // Cart items state
  const [items, setItems] = useState([]);
  
  // UI state
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [notification, setNotification] = useState(null);
  
  // Checkout state
  const [isCheckoutOpen, setIsCheckoutOpen] = useState(false);
  
  // Load cart from localStorage on mount
  useEffect(() => {
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      setItems(JSON.parse(savedCart));
    }
  }, []);
  
  // Save cart to localStorage whenever it changes
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(items));
  }, [items]);
  
  // Add item to cart
  const addItem = useCallback((product) => {
    setItems(prevItems => {
      const existingItem = prevItems.find(item => item.id === product.id);
      
      if (existingItem) {
        // Increase quantity if item exists
        return prevItems.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        // Add new item
        return [...prevItems, { ...product, quantity: 1 }];
      }
    });
    
    setNotification(`${product.name} added to cart`);
    setTimeout(() => setNotification(null), 3000);
  }, []);
  
  // Remove item from cart
  const removeItem = useCallback((productId) => {
    setItems(prevItems => prevItems.filter(item => item.id !== productId));
    setNotification('Item removed from cart');
    setTimeout(() => setNotification(null), 3000);
  }, []);
  
  // Update item quantity
  const updateQuantity = useCallback((productId, newQuantity) => {
    if (newQuantity <= 0) {
      removeItem(productId);
      return;
    }
    
    setItems(prevItems =>
      prevItems.map(item =>
        item.id === productId
          ? { ...item, quantity: newQuantity }
          : item
      )
    );
  }, [removeItem]);
  
  // Calculate totals
  const subtotal = items.reduce((sum, item) => 
    sum + (item.price * item.quantity), 0
  );
  const tax = subtotal * 0.1;
  const total = subtotal + tax;
  
  // Checkout process
  const handleCheckout = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ items, total }),
      });
      
      if (!response.ok) throw new Error('Checkout failed');
      
      const result = await response.json();
      setItems([]); // Clear cart
      setNotification('Order placed successfully!');
      setIsCheckoutOpen(false);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div className="shopping-cart">
      {notification && (
        <div className="notification">{notification}</div>
      )}
      
      {error && (
        <div className="error">{error}</div>
      )}
      
      <h2>Shopping Cart ({items.length} items)</h2>
      
      {items.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <>
          <ul className="cart-items">
            {items.map(item => (
              <li key={item.id}>
                <img src={item.image} alt={item.name} />
                <div>
                  <h3>{item.name}</h3>
                  <p>${item.price.toFixed(2)}</p>
                </div>
                <div className="quantity-controls">
                  <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>
                    -
                  </button>
                  <span>{item.quantity}</span>
                  <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                    +
                  </button>
                </div>
                <button onClick={() => removeItem(item.id)}>Remove</button>
              </li>
            ))}
          </ul>
          
          <div className="cart-summary">
            <p>Subtotal: ${subtotal.toFixed(2)}</p>
            <p>Tax: ${tax.toFixed(2)}</p>
            <h3>Total: ${total.toFixed(2)}</h3>
            
            <button 
              onClick={() => setIsCheckoutOpen(true)}
              disabled={isLoading}
            >
              {isLoading ? 'Processing...' : 'Checkout'}
            </button>
          </div>
        </>
      )}
    </div>
  );
};

export default ShoppingCart;
```

**State Best Practices**

```jsx
// ✅ 1. Keep state minimal and derived
const [users, setUsers] = useState([]);
// Derive from state instead of storing separately
const activeUsers = users.filter(u => u.isActive);
const userCount = users.length;

// ❌ Don't duplicate state
// const [activeUsers, setActiveUsers] = useState([]);
// const [userCount, setUserCount] = useState(0);

// ✅ 2. Use functional updates when new state depends on old
setCount(prevCount => prevCount + 1);

// ✅ 3. Batch related state updates
const [user, setUser] = useState({ name: '', email: '' });
// Better than: const [name, setName] = ...; const [email, setEmail] = ...;

// ✅ 4. Initialize state correctly
const [data, setData] = useState(() => expensiveCalculation());

// ✅ 5. Don't store props in state (unless intentional)
// ❌ Anti-pattern
const Child = ({ initialValue }) => {
  const [value, setValue] = useState(initialValue);
  // Won't update if prop changes!
};

// ✅ Use prop directly or useEffect to sync
const Child = ({ value }) => {
  return <div>{value}</div>;
};
```

**Interview Tips:**
- State is **internal** data managed by the component, props are **external** data passed in
- State changes trigger **re-renders** automatically
- Use **functional updates** when new state depends on previous state
- State updates are **asynchronous** and may be **batched** for performance
- Don't mutate state directly - always create new objects/arrays
- Mention **useState** hook for functional components
- Discuss state lifting for sharing state between components
- Highlight difference: state is mutable (within component), props are immutable

</details>

---

### 8. What is the Virtual DOM?

<details>
<summary>View Answer</summary>

**What is the Virtual DOM?**

The **Virtual DOM (VDOM)** is a lightweight JavaScript representation of the actual DOM. It's a programming concept where a "virtual" representation of the UI is kept in memory and synced with the real DOM through a process called **reconciliation**. This is the core of React's performance optimization strategy.

**Key Concepts:**
- **Virtual DOM**: In-memory JavaScript object representing the DOM
- **Real DOM**: Actual browser DOM (HTML elements)
- **Reconciliation**: Process of comparing Virtual DOM with Real DOM
- **Diffing Algorithm**: Algorithm to find differences efficiently
- **Batch Updates**: Multiple changes applied together for performance

**Why Virtual DOM?**

```jsx
// Problem with Real DOM manipulation
// Direct DOM manipulation is SLOW:

// ❌ Slow: Each operation directly touches the browser DOM
document.getElementById('title').innerText = 'New Title';
document.getElementById('count').innerText = '5';
document.getElementById('status').innerText = 'Active';
// Browser reflows/repaints 3 times!

// ✅ React's Virtual DOM approach
// React batches updates and applies them efficiently:
setTitle('New Title');
setCount(5);
setStatus('Active');
// React calculates differences, updates Real DOM ONCE
```

**How Virtual DOM Works**

**Step 1: Initial Render**
```jsx
const App = () => {
  return (
    <div id="app">
      <h1>Hello World</h1>
      <p>Count: 0</p>
    </div>
  );
};

// React creates Virtual DOM object:
const virtualDOM = {
  type: 'div',
  props: { id: 'app' },
  children: [
    {
      type: 'h1',
      props: {},
      children: ['Hello World']
    },
    {
      type: 'p',
      props: {},
      children: ['Count: 0']
    }
  ]
};

// React renders to Real DOM
<div id="app">
  <h1>Hello World</h1>
  <p>Count: 0</p>
</div>
```

**Step 2: State Update**
```jsx
const App = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div id="app">
      <h1>Hello World</h1>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// When user clicks button:
// 1. State changes: count = 0 → 1
// 2. React creates NEW Virtual DOM
const newVirtualDOM = {
  type: 'div',
  props: { id: 'app' },
  children: [
    {
      type: 'h1',
      props: {},
      children: ['Hello World']  // Unchanged
    },
    {
      type: 'p',
      props: {},
      children: ['Count: 1']     // Changed!
    },
    {
      type: 'button',
      props: { onClick: handler },
      children: ['Increment']    // Unchanged
    }
  ]
};

// 3. React compares OLD vs NEW Virtual DOM (Diffing)
// 4. React finds: Only <p> text content changed
// 5. React updates ONLY that text node in Real DOM
```

**Virtual DOM Process Visualization**

```
┌─────────────────────────────────────────────────────┐
│                   REACT COMPONENT                   │
│                                                     │
│  const App = () => {                                │
│    const [count, setCount] = useState(0);          │
│    return <div>Count: {count}</div>;               │
│  }                                                  │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│              CREATE VIRTUAL DOM (JS Object)         │
│                                                     │
│  {                                                  │
│    type: 'div',                                     │
│    props: {},                                       │
│    children: ['Count: 0']                          │
│  }                                                  │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│              STATE UPDATE: setCount(1)              │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│            CREATE NEW VIRTUAL DOM                   │
│                                                     │
│  {                                                  │
│    type: 'div',                                     │
│    props: {},                                       │
│    children: ['Count: 1']  ← Changed               │
│  }                                                  │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│       DIFFING ALGORITHM (Reconciliation)            │
│                                                     │
│  Compare OLD vs NEW Virtual DOM                     │
│  Find: Text changed from 'Count: 0' to 'Count: 1' │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│        UPDATE REAL DOM (Minimal Changes)            │
│                                                     │
│  document.getElementById('root').textContent        │
│    = 'Count: 1'                                     │
└─────────────────────────────────────────────────────┘
```

**Reconciliation (Diffing) Algorithm**

**Rule 1: Different Element Types → Replace**
```jsx
// Old Virtual DOM
<div>
  <Counter />
</div>

// New Virtual DOM
<span>
  <Counter />
</span>

// React behavior:
// - Destroy entire <div> and <Counter>
// - Create new <span> and new <Counter>
// - Counter loses its state!
```

**Rule 2: Same Element Type → Update Props**
```jsx
// Old Virtual DOM
<div className="old" title="Old Title">
  Content
</div>

// New Virtual DOM
<div className="new" title="New Title">
  Content
</div>

// React behavior:
// - Keep same DOM node
// - Update only changed attributes: className, title
// - Don't touch children
```

**Rule 3: Component Type → Recursive**
```jsx
// Old Virtual DOM
<Counter count={1} />

// New Virtual DOM
<Counter count={2} />

// React behavior:
// - Component type unchanged (Counter)
// - Update props: count 1 → 2
// - Component re-renders with new props
// - Apply rules recursively to component's output
```

**Rule 4: Keys for Lists → Optimize**
```jsx
// ❌ Without keys - React can't track items
<ul>
  {items.map(item => (
    <li>{item.name}</li>
  ))}
</ul>
// Problem: Adding item at start = Re-render ALL items

// ✅ With keys - React tracks each item
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
// Benefit: React knows which items are new/moved/deleted
```

**Performance Comparison**

```jsx
// Example: Update 1 item in list of 1000
const TodoList = () => {
  const [todos, setTodos] = useState(/* 1000 items */);
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  // WITHOUT Virtual DOM:
  // - Destroy and recreate all 1000 <li> elements
  // - Browser reflows/repaints entire list
  // - Time: ~50-100ms
  
  // WITH Virtual DOM:
  // - React diffs: finds only 1 <li> changed
  // - Updates only that 1 <li>'s checkbox
  // - Time: ~1-2ms
  // - 50x FASTER!
};
```

**Virtual DOM vs Real DOM Operations**

```jsx
// Scenario: Update user profile (3 fields change)

// ❌ Direct Real DOM manipulation (SLOW)
function updateProfileDirect(user) {
  document.getElementById('name').textContent = user.name;
  // Browser reflow + repaint
  
  document.getElementById('email').textContent = user.email;
  // Browser reflow + repaint
  
  document.getElementById('avatar').src = user.avatar;
  // Browser reflow + repaint
  
  // Total: 3 reflows, 3 repaints = SLOW
}

// ✅ React with Virtual DOM (FAST)
function UserProfile({ user }) {
  return (
    <div>
      <h1 id="name">{user.name}</h1>
      <p id="email">{user.email}</p>
      <img id="avatar" src={user.avatar} />
    </div>
  );
}

// React process:
// 1. Create new Virtual DOM with updated values
// 2. Diff against old Virtual DOM
// 3. Batch all 3 changes
// 4. Apply to Real DOM in ONE operation
// Total: 1 reflow, 1 repaint = FAST
```

**Key Benefits of Virtual DOM**

**1. Performance Optimization**
```jsx
// Batched updates
const Component = () => {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);
  const [c, setC] = useState(0);
  
  const handleClick = () => {
    setA(1);  // Virtual DOM update
    setB(2);  // Virtual DOM update
    setC(3);  // Virtual DOM update
    
    // React batches these:
    // 1. Creates new Virtual DOM with all changes
    // 2. One diff operation
    // 3. One Real DOM update
    // = Single browser repaint!
  };
};
```

**2. Declarative Programming**
```jsx
// You declare WHAT the UI should look like
const TodoApp = () => {
  const [todos, setTodos] = useState([...]);
  
  // Just describe the UI based on current state
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.completed ? '✓' : '○'} {todo.text}
        </li>
      ))}
    </ul>
  );
  
  // React figures out HOW to update the DOM efficiently
};
```

**3. Cross-Platform Rendering**
```jsx
// Same Virtual DOM concept works across platforms:

// Web (ReactDOM)
import ReactDOM from 'react-dom';
ReactDOM.createRoot(root).render(<App />);

// Mobile (React Native)
import { AppRegistry } from 'react-native';
AppRegistry.registerComponent('App', () => App);

// Both use the same React core and Virtual DOM
// Different renderers handle actual platform updates
```

**Common Misconceptions**

```jsx
// ❌ Myth 1: "Virtual DOM is always faster than Real DOM"
// Truth: For simple operations, direct DOM can be faster
// Example: Updating single text node

// Direct DOM (faster for this case)
document.getElementById('text').textContent = 'Hello';

// React (adds Virtual DOM overhead)
setText('Hello'); // Creates Virtual DOM, diffs, updates

// Virtual DOM shines with:
// - Complex UIs
// - Multiple updates
// - Conditional rendering
// - Dynamic lists

// ❌ Myth 2: "Virtual DOM replaces entire Real DOM"
// Truth: React updates only changed parts

// ❌ Myth 3: "Virtual DOM means no performance concerns"
// Truth: Still need optimization (keys, memoization, etc.)
```

**Virtual DOM Limitations**

```jsx
// 1. Initial render overhead
// First render creates entire Virtual DOM before Real DOM
// Solution: Server-Side Rendering (SSR) for better initial load

// 2. Memory overhead
// Virtual DOM objects consume memory
// For huge lists (10,000+ items), consider virtualization:
import { FixedSizeList } from 'react-window';

const LargeList = ({ items }) => (
  <FixedSizeList
    height={600}
    itemCount={items.length}
    itemSize={50}
  >
    {({ index }) => <div>{items[index].name}</div>}
  </FixedSizeList>
);

// 3. Not always fastest for simple apps
// For very simple apps, direct DOM might be faster
// Virtual DOM benefits scale with complexity
```

**Optimizing Virtual DOM Performance**

```jsx
// 1. ✅ Use keys for lists
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li> // Unique, stable key
  ))}
</ul>

// ❌ Don't use index as key for dynamic lists
{items.map((item, index) => (
  <li key={index}>{item.name}</li> // Bad if items can be reordered
))}

// 2. ✅ Memoize components to skip diffing
const ExpensiveComponent = memo(({ data }) => {
  return <div>{/* Complex rendering */}</div>;
});

// 3. ✅ Use React.Fragment to avoid extra DOM nodes
return (
  <>
    <Header />
    <Content />
    <Footer />
  </>
);

// 4. ✅ Avoid inline object/array creation
// ❌ Creates new object every render
<Component style={{ margin: 10 }} />

// ✅ Define outside component or use useMemo
const style = { margin: 10 };
<Component style={style} />
```

**React Fiber (Modern Virtual DOM)**

```jsx
// React 16+ uses "Fiber" - Improved Virtual DOM

// Old Virtual DOM: Synchronous rendering
// Problem: Large updates block browser (janky UI)

// Fiber: Asynchronous, incremental rendering
// Benefits:
// 1. Ability to pause work and resume later
// 2. Priority-based rendering (user input > animations)
// 3. Better error handling (Error Boundaries)
// 4. Concurrent features (React 18+)

// Example: Concurrent rendering
import { startTransition } from 'react';

const SearchResults = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = (e) => {
    setQuery(e.target.value); // High priority (instant)
    
    startTransition(() => {
      // Low priority (can be interrupted)
      const filtered = items.filter(item => 
        item.name.includes(e.target.value)
      );
      setResults(filtered);
    });
  };
  
  // User typing remains smooth even with expensive filtering
};
```

**Interview Tips:**
- Virtual DOM is an **in-memory representation** of the Real DOM
- React uses **reconciliation** (diffing algorithm) to find minimal changes
- Benefits: **Performance** (batched updates), **declarative** programming, **cross-platform**
- Not a direct Real DOM replacement - React still updates Real DOM
- Mention **React Fiber** as the modern implementation (since React 16)
- Discuss **keys** importance for list reconciliation
- Explain that Virtual DOM adds overhead but pays off for complex UIs
- Highlight that it enables React's programming model (declarative, component-based)

</details>

---

### 9. How does React's reconciliation algorithm work?

<details>
<summary>View Answer</summary>

**What is Reconciliation?**

**Reconciliation** is the process React uses to update the DOM efficiently. When a component's state or props change, React needs to figure out how to update the UI. The reconciliation algorithm (also called the "diffing algorithm") compares the new Virtual DOM with the previous one and determines the minimal set of changes needed.

**Key Concepts:**
- **Diffing**: Comparing old and new Virtual DOM trees
- **Heuristic O(n) Algorithm**: React uses assumptions to achieve linear time complexity
- **Element Types**: Different types trigger different update strategies
- **Keys**: Help React identify which items changed in lists
- **Fiber Architecture**: Modern implementation (React 16+)

**The Reconciliation Process**

```jsx
// Step-by-step example

// 1. Initial Render
const App = () => {
  const [count, setCount] = useState(0);
  return <div>Count: {count}</div>;
};

// React creates Virtual DOM tree:
{
  type: 'div',
  props: {},
  children: ['Count: 0']
}

// 2. State Update (user clicks button)
setCount(1);

// React creates NEW Virtual DOM tree:
{
  type: 'div',
  props: {},
  children: ['Count: 1']  // Changed!
}

// 3. Reconciliation (Diffing)
// React compares old vs new:
// - Element type: 'div' === 'div' ✓ (same)
// - Props: {} === {} ✓ (same)
// - Children: 'Count: 0' !== 'Count: 1' ✗ (different)

// 4. Minimal Update
// React updates ONLY the text node in Real DOM
document.getElementById('root').firstChild.textContent = 'Count: 1';
```

**Reconciliation Rules**

**Rule 1: Elements of Different Types**

```jsx
// Scenario: Root element type changes
// Old Virtual DOM
<div>
  <Counter />
</div>

// New Virtual DOM  
<span>
  <Counter />
</span>

// React's Behavior:
// 1. Destroy entire old tree:
//    - Unmount <div>
//    - Unmount <Counter> (loses state!)
//    - Call componentWillUnmount / cleanup functions
// 2. Build new tree from scratch:
//    - Mount new <span>
//    - Mount new <Counter> (fresh state)
//    - Call componentDidMount / effects

// ⚠️ Important: Component state is LOST!
```

**Real Example:**
```jsx
const App = () => {
  const [showDiv, setShowDiv] = useState(true);
  
  return (
    <>
      <button onClick={() => setShowDiv(!showDiv)}>Toggle</button>
      
      {showDiv ? (
        <div>
          <Counter /> {/* Counter state preserved when toggling */}
        </div>
      ) : (
        <section>
          <Counter /> {/* NEW Counter - state reset! */}
        </section>
      )}
    </>
  );
};

// When toggling:
// - <div> → <section>: Different types
// - Old Counter destroyed, new Counter created
// - Counter loses its state!

// ✅ Solution: Keep same element type
{showDiv ? (
  <div className="wrapper">
    <Counter />
  </div>
) : (
  <div className="wrapper-alternate"> {/* Same type: div */}
    <Counter /> {/* State preserved! */}
  </div>
)}
```

**Rule 2: DOM Elements of Same Type**

```jsx
// Scenario: Same element type, different attributes
// Old Virtual DOM
<div className="old" title="Old Title" style={{ color: 'red' }}>
  <span>Content</span>
</div>

// New Virtual DOM
<div className="new" title="New Title" style={{ color: 'blue' }}>
  <span>Content</span>
</div>

// React's Behavior:
// 1. Keep the same DOM node (no recreation)
// 2. Update only changed attributes:
//    - className: "old" → "new"
//    - title: "Old Title" → "New Title"
//    - style.color: "red" → "blue"
// 3. Recursively process children

// Performance: Very efficient - minimal DOM operations
```

**Real Example:**
```jsx
const Button = ({ variant, disabled }) => {
  const [count, setCount] = useState(0);
  
  return (
    <button 
      className={`btn btn-${variant}`}  // Attribute may change
      disabled={disabled}                 // Attribute may change
      onClick={() => setCount(count + 1)}
    >
      Clicked {count} times
    </button>
  );
};

// When variant or disabled changes:
// - React keeps same <button> element
// - Updates only className and disabled attributes
// - Component state (count) is PRESERVED
// - Very fast update!
```

**Rule 3: Component Elements of Same Type**

```jsx
// Scenario: Same component type, different props
// Old Virtual DOM
<UserProfile userId={1} showDetails={false} />

// New Virtual DOM
<UserProfile userId={1} showDetails={true} />

// React's Behavior:
// 1. Keep same component instance
// 2. Update props: showDetails false → true
// 3. Call component update lifecycle:
//    - For class: componentWillReceiveProps, shouldComponentUpdate, render
//    - For functional: Re-run function with new props
// 4. Reconcile the component's output recursively
// 5. Component state is PRESERVED

// Example
const UserProfile = ({ userId, showDetails }) => {
  const [isExpanded, setIsExpanded] = useState(false); // State preserved
  
  return (
    <div>
      <h2>User {userId}</h2>
      {showDetails && <Details />}
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? 'Collapse' : 'Expand'}
      </button>
    </div>
  );
};

// When showDetails prop changes:
// - Component instance kept
// - isExpanded state preserved
// - Only renders new output with updated props
```

**Rule 4: Recursion on Children**

```jsx
// React processes children recursively
// Old Virtual DOM
<ul>
  <li>First</li>
  <li>Second</li>
</ul>

// New Virtual DOM
<ul>
  <li>First</li>
  <li>Second</li>
  <li>Third</li>  // Added at end
</ul>

// React's Behavior:
// 1. Compare first child: "First" === "First" ✓
// 2. Compare second child: "Second" === "Second" ✓
// 3. Third child is new → Insert new <li>
// Efficient: Only 1 insertion

// ⚠️ Problem: Insertion at beginning
// Old Virtual DOM
<ul>
  <li>First</li>
  <li>Second</li>
</ul>

// New Virtual DOM
<ul>
  <li>Zero</li>   // Inserted at start
  <li>First</li>
  <li>Second</li>
</ul>

// Without keys, React compares by position:
// Position 0: "First" → "Zero" (UPDATE)
// Position 1: "Second" → "First" (UPDATE)
// Position 2: undefined → "Second" (INSERT)
// Inefficient: 2 updates + 1 insertion instead of 1 insertion!
```

**Rule 5: Keys (Most Important for Lists)**

```jsx
// ✅ WITH KEYS: React can track items
// Old Virtual DOM
<ul>
  <li key="a">First</li>
  <li key="b">Second</li>
</ul>

// New Virtual DOM
<ul>
  <li key="c">Zero</li>   // New item
  <li key="a">First</li>  // Moved
  <li key="b">Second</li> // Moved
</ul>

// React's Behavior with keys:
// 1. Recognizes key="a" and key="b" still exist (just moved)
// 2. Recognizes key="c" is new
// 3. Operation: Insert new <li key="c"> at position 0
// 4. Move existing elements (cheap in DOM)
// Efficient: 1 insertion, 0 updates!

// Real Example
const TodoList = ({ todos }) => {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>  {/* ✅ Stable, unique key */}
          <TodoItem todo={todo} />
        </li>
      ))}
    </ul>
  );
};

// When todos reordered/filtered:
// - React uses keys to identify which items moved
// - Preserves component state for each item
// - Minimal DOM operations
```

**Keys: Best Practices**

```jsx
// ✅ GOOD: Stable, unique IDs from data
<ul>
  {users.map(user => (
    <li key={user.id}>  // Database ID
      {user.name}
    </li>
  ))}
</ul>

// ✅ GOOD: Composite keys for nested data
<div>
  {posts.map(post => (
    <div key={post.id}>
      <h3>{post.title}</h3>
      <ul>
        {post.comments.map(comment => (
          <li key={`${post.id}-${comment.id}`}>  // Composite key
            {comment.text}
          </li>
        ))}
      </ul>
    </div>
  ))}
</div>

// ⚠️ ACCEPTABLE: Index as key (ONLY if list never reorders/filters)
<ul>
  {staticItems.map((item, index) => (
    <li key={index}>  // OK if list is static
      {item}
    </li>
  ))}
</ul>

// ❌ BAD: Index as key for dynamic lists
const TodoList = () => {
  const [todos, setTodos] = useState([...]);
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>  {/* ❌ BAD - causes bugs! */}
          <input type="checkbox" checked={todo.done} />
          {todo.text}
        </li>
      ))}
    </ul>
  );
};

// Problem with index keys:
// Initial: [0: "Buy milk", 1: "Do laundry"]
// User checks "Buy milk" checkbox
// User deletes "Buy milk"
// New: [0: "Do laundry"]
// React sees key=0 still exists, reuses DOM node
// Checkbox state incorrectly applied to "Do laundry"!

// ❌ VERY BAD: Random or unstable keys
{items.map(item => (
  <li key={Math.random()}>  {/* ❌ NEVER DO THIS */}
    {item}
  </li>
))}
// Every render creates new keys → React recreates all items!
```

**Reconciliation Performance**

```jsx
// Scenario: Large list update

// Without optimization
const LargeList = ({ items }) => {
  return (
    <div>
      {items.map(item => (
        <ExpensiveComponent key={item.id} item={item} />
      ))}
    </div>
  );
};

// Problem: Every parent re-render = all children re-render
// Solution: Memoization

// ✅ With React.memo (skip reconciliation for unchanged props)
const ExpensiveComponent = memo(({ item }) => {
  console.log(`Rendering item ${item.id}`);
  return <div>{/* Complex rendering */}</div>;
});

const LargeList = ({ items }) => {
  const [filter, setFilter] = useState('');
  
  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {items.map(item => (
        // Only items with changed props re-render
        <ExpensiveComponent key={item.id} item={item} />
      ))}
    </div>
  );
};

// When filter changes:
// - LargeList re-renders
// - React reconciles each child
// - memo() prevents re-render if item prop unchanged
// - Significant performance improvement!
```

**React Fiber: Modern Reconciliation**

```jsx
// Pre-React 16: Synchronous reconciliation
// - Reconciliation is blocking
// - Large updates freeze UI
// - No way to prioritize updates

// React 16+: Fiber reconciliation
// - Asynchronous, interruptible reconciliation
// - Can pause work and resume later
// - Priority-based scheduling
// - Incremental rendering

// Fiber enables:

// 1. Time slicing
const HeavyComponent = () => {
  // React can pause rendering if needed
  const result = complexCalculation(); // May be split into chunks
  return <div>{result}</div>;
};

// 2. Priority-based rendering (React 18+)
import { useTransition } from 'react';

const SearchResults = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    // High priority: Update input immediately
    setQuery(e.target.value);
    
    // Low priority: Results can wait
    startTransition(() => {
      const filtered = items.filter(item => 
        item.name.includes(e.target.value)
      );
      setResults(filtered);
    });
  };
  
  // User input stays responsive even with expensive filtering
};

// 3. Error boundaries
class ErrorBoundary extends Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    logError(error, info);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Fiber allows errors to be caught and handled gracefully
```

**Reconciliation Optimization Techniques**

```jsx
// 1. ✅ Use keys for lists
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>

// 2. ✅ Memoize expensive components
const ExpensiveList = memo(({ items }) => {
  return items.map(item => <Item key={item.id} {...item} />);
});

// 3. ✅ Use React.PureComponent or memo
const PureButton = memo(({ onClick, label }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>{label}</button>;
});

// 4. ✅ Avoid inline object/array creation
// ❌ Bad: Creates new object every render
<Component style={{ margin: 10 }} />

// ✅ Good: Reference stable
const style = { margin: 10 };
<Component style={style} />

// ✅ Or use useMemo
const style = useMemo(() => ({ margin: 10 }), []);
<Component style={style} />

// 5. ✅ Split large components
// ❌ Bad: Entire component re-renders
const Dashboard = () => {
  const [filter, setFilter] = useState('');
  const [data, setData] = useState([]);
  
  return (
    <div>
      <Header data={data} />  {/* Re-renders when filter changes */}
      <Filters filter={filter} onChange={setFilter} />
      <DataTable data={data} />
    </div>
  );
};

// ✅ Good: Isolate changing parts
const Dashboard = () => {
  return (
    <div>
      <Header />  {/* Doesn't re-render */}
      <FilterSection />  {/* Has own state */}
      <DataSection />
    </div>
  );
};
```

**Common Reconciliation Issues**

```jsx
// Issue 1: Missing keys in lists
// ❌ Problem
{items.map(item => <div>{item.name}</div>)}
// React uses index as key → bugs with reordering

// ✅ Solution
{items.map(item => <div key={item.id}>{item.name}</div>)}

// Issue 2: Changing component type
// ❌ Problem
{isLoggedIn ? <UserDashboard /> : <GuestDashboard />}
// Different component types → full remount, state lost

// ✅ Solution (if you want to preserve state)
<Dashboard userType={isLoggedIn ? 'user' : 'guest'} />

// Issue 3: Inline function props
// ❌ Problem: New function every render
<Child onClick={() => doSomething()} />
// React.memo won't help - props always "different"

// ✅ Solution: useCallback
const handleClick = useCallback(() => doSomething(), []);
<Child onClick={handleClick} />

// Issue 4: Conditional rendering position
// ❌ Problem
<div>
  {showA && <ComponentA />}
  {showB && <ComponentB />}
</div>
// Position in tree changes → reconciliation issues

// ✅ Solution: Stable positions
<div>
  {showA ? <ComponentA /> : null}
  {showB ? <ComponentB /> : null}
</div>
```

**Production Example: Optimized List Reconciliation**

```jsx
import { memo, useCallback, useMemo } from 'react';

// Memoized list item - only re-renders if props change
const TodoItem = memo(({ todo, onToggle, onDelete }) => {
  console.log(`Rendering todo ${todo.id}`);
  
  return (
    <li className={todo.completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});

TodoItem.displayName = 'TodoItem';

const TodoList = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Master reconciliation', completed: false },
  ]);
  const [filter, setFilter] = useState('all');
  
  // Memoized callbacks - stable references
  const handleToggle = useCallback((id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  const handleDelete = useCallback((id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
  }, []);
  
  // Memoized filtered list
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  }, [todos, filter]);
  
  return (
    <div>
      <div className="filters">
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('active')}>Active</button>
        <button onClick={() => setFilter('completed')}>Completed</button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <TodoItem
            key={todo.id}  // ✅ Stable unique key
            todo={todo}
            onToggle={handleToggle}  // ✅ Stable callback
            onDelete={handleDelete}  // ✅ Stable callback
          />
        ))}
      </ul>
      
      {/* Only re-renders when filteredTodos changes */}
      <p>Total: {filteredTodos.length} items</p>
    </div>
  );
};

// Performance characteristics:
// - Changing filter: Only filtered list updates, items with stable props don't re-render
// - Toggling one todo: Only that ONE TodoItem re-renders (thanks to memo + stable callbacks)
// - Deleting todo: Only list order reconciles, other items don't re-render
// - Adding todo: Only new item renders, existing items untouched
```

**Interview Tips:**
- Reconciliation is React's **diffing algorithm** to minimize DOM updates
- Uses **heuristic O(n) algorithm** (linear time) instead of O(n³)
- Two main assumptions: different element types = full rebuild, same type = update props
- **Keys are critical** for list reconciliation - enable React to track items across renders
- **React Fiber** (React 16+) enables asynchronous, interruptible reconciliation
- Mention optimization: React.memo, useCallback, useMemo to reduce reconciliation work
- Explain that reconciliation compares Virtual DOM trees, not Real DOM
- Keys should be stable and unique - never use Math.random() or index for dynamic lists
- Understanding reconciliation helps write performant React applications

</details>

---

### 10. What is the difference between state and props?

<details>
<summary>View Answer</summary>

**State vs Props: Core Differences**

**State** and **Props** are both JavaScript objects that hold data which influences the component's render output, but they have fundamentally different purposes and behaviors.

**Quick Comparison**

| Aspect | State | Props |
|--------|-------|-------|
| **Definition** | Internal data storage | External data passed from parent |
| **Ownership** | Owned by component | Owned by parent component |
| **Mutability** | Mutable (can be changed) | Immutable (read-only) |
| **Control** | Component controls its own state | Parent controls props |
| **Updates** | Use setState/useState | Parent re-renders with new values |
| **Initial Value** | Set in component | Received from parent |
| **Purpose** | Manage dynamic data within component | Pass data & callbacks between components |
| **Re-render Trigger** | Yes (when state updates) | Yes (when parent passes new props) |

**Fundamental Concepts**

```jsx
// PROPS: Data flows DOWN (parent → child)
const Parent = () => {
  return (
    <Child 
      name="John"           // Prop
      age={30}              // Prop
      onUpdate={handleUpdate}  // Prop (callback)
    />
  );
};

const Child = ({ name, age, onUpdate }) => {
  // ❌ Cannot modify props
  // name = "Jane"; // Error!
  
  return (
    <div>
      <p>{name} is {age} years old</p>
      <button onClick={onUpdate}>Update</button>
    </div>
  );
};

// STATE: Data managed WITHIN component
const Counter = () => {
  // ✅ Can modify own state
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1); // Allowed!
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};
```

**Props: Detailed Explanation**

```jsx
// Props are READ-ONLY and come from PARENT

// Parent component
const UserDashboard = () => {
  const [user, setUser] = useState({
    name: 'John Doe',
    email: 'john@example.com',
    role: 'admin'
  });
  
  return (
    <div>
      {/* Passing data down as props */}
      <UserProfile 
        user={user}                    // Data prop
        isAdmin={user.role === 'admin'} // Computed prop
        onUpdateEmail={(newEmail) => {  // Callback prop
          setUser({ ...user, email: newEmail });
        }}
      />
      
      <UserSettings 
        user={user}
        theme="dark"
      />
    </div>
  );
};

// Child component
const UserProfile = ({ user, isAdmin, onUpdateEmail }) => {
  // Props are read-only in child
  // user.name = "Jane"; // ❌ WRONG - Don't mutate props!
  
  // To "change" data, call parent's callback
  const handleEmailChange = (e) => {
    onUpdateEmail(e.target.value); // ✅ Correct
  };
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      {isAdmin && <span>Admin Badge</span>}
      <input onChange={handleEmailChange} placeholder="New email" />
    </div>
  );
};

// Key points about props:
// 1. Flow ONE direction: Parent → Child
// 2. Cannot be modified by child
// 3. To "update" props, child must notify parent via callback
// 4. Parent's state → Child's props
```

**State: Detailed Explanation**

```jsx
// State is MUTABLE and INTERNAL to component

const TodoApp = () => {
  // Component's own state - can modify anytime
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  const [filter, setFilter] = useState('all');
  
  // State can be updated based on user interactions
  const addTodo = () => {
    setTodos([...todos, {
      id: Date.now(),
      text: input,
      completed: false
    }]);
    setInput(''); // Clear input state
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  // Derived value from state (not stored in state)
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <div>
      <input 
        value={input}  // State
        onChange={(e) => setInput(e.target.value)}  // Update state
      />
      <button onClick={addTodo}>Add</button>
      
      <div>
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('active')}>Active</button>
        <button onClick={() => setFilter('completed')}>Completed</button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id} onClick={() => toggleTodo(todo.id)}>
            {todo.completed ? '✓' : '○'} {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
};

// Key points about state:
// 1. Private to component
// 2. Can be updated using setState/useState
// 3. Updates trigger re-render
// 4. Should contain minimal data (derive when possible)
```

**Data Flow: Unidirectional**

```jsx
// React's one-way data flow: State → Props → State → Props

// Grandparent: Has state
const GrandParent = () => {
  const [userData, setUserData] = useState({
    name: 'John',
    age: 30
  });
  
  return (
    <Parent 
      user={userData}                    // State → Props
      onUpdateUser={setUserData}         // Callback to update state
    />
  );
};

// Parent: Receives props, passes down
const Parent = ({ user, onUpdateUser }) => {
  return (
    <Child 
      userName={user.name}               // Props → Props
      userAge={user.age}                 // Props → Props
      onChangeName={(name) => {          // Transform callback
        onUpdateUser({ ...user, name });
      }}
    />
  );
};

// Child: Receives props, can trigger updates via callbacks
const Child = ({ userName, userAge, onChangeName }) => {
  const [localEdit, setLocalEdit] = useState(''); // Local state
  
  const handleSubmit = () => {
    onChangeName(localEdit);  // Notify parent via callback
    setLocalEdit('');          // Reset local state
  };
  
  return (
    <div>
      <p>{userName} ({userAge})</p>
      <input 
        value={localEdit}
        onChange={(e) => setLocalEdit(e.target.value)}
      />
      <button onClick={handleSubmit}>Update Name</button>
    </div>
  );
};

// Data flow:
// GrandParent state → Parent props → Child props
// Child callback → Parent callback → GrandParent state
// GrandParent state update → Re-render cascade
```

**When to Use State vs Props**

```jsx
// Use STATE when:
// 1. Data changes over time within component
const Timer = () => {
  const [seconds, setSeconds] = useState(0); // ✅ State - changes over time
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);
  
  return <div>Elapsed: {seconds}s</div>;
};

// 2. User input/interactions
const Form = () => {
  const [email, setEmail] = useState(''); // ✅ State - user input
  const [password, setPassword] = useState(''); // ✅ State - user input
  
  return (
    <form>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input value={password} onChange={(e) => setPassword(e.target.value)} />
    </form>
  );
};

// 3. Component-specific UI state
const Accordion = () => {
  const [isOpen, setIsOpen] = useState(false); // ✅ State - UI toggle
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Close' : 'Open'}
      </button>
      {isOpen && <div>Content</div>}
    </div>
  );
};

// Use PROPS when:
// 1. Data comes from parent
const UserCard = ({ name, email, avatar }) => { // ✅ Props - from parent
  return (
    <div>
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
};

// 2. Configuration/customization
const Button = ({ 
  label,           // ✅ Props - configuration
  variant,         // ✅ Props - configuration
  size,            // ✅ Props - configuration
  onClick          // ✅ Props - callback
}) => {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
};

// 3. Passing callbacks for communication
const Parent = () => {
  const handleChildClick = (data) => {
    console.log('Child sent:', data);
  };
  
  return <Child onAction={handleChildClick} />; // ✅ Props - callback
};

const Child = ({ onAction }) => {
  return <button onClick={() => onAction('Hello')}>Click</button>;
};
```

**State Lifting Pattern**

```jsx
// When multiple components need same data: Lift state UP

// ❌ Problem: Separate states, can't sync
const TemperatureInput1 = () => {
  const [temp, setTemp] = useState('');
  return <input value={temp} onChange={e => setTemp(e.target.value)} />;
};

const TemperatureInput2 = () => {
  const [temp, setTemp] = useState(''); // Different state!
  return <input value={temp} onChange={e => setTemp(e.target.value)} />;
};

// ✅ Solution: Lift state to common parent
const TemperatureConverter = () => {
  const [celsius, setCelsius] = useState(''); // Lifted state
  
  const fahrenheit = celsius ? (celsius * 9/5 + 32).toFixed(1) : '';
  
  return (
    <div>
      <TemperatureInput
        scale="Celsius"
        temperature={celsius}  // State → Props
        onTemperatureChange={setCelsius}  // Callback
      />
      <TemperatureInput
        scale="Fahrenheit"
        temperature={fahrenheit}  // State → Props (derived)
        onTemperatureChange={(f) => {
          setCelsius(((f - 32) * 5/9).toFixed(1));
        }}
      />
      <p>Water boils: {celsius >= 100 ? 'Yes' : 'No'}</p>
    </div>
  );
};

const TemperatureInput = ({ scale, temperature, onTemperatureChange }) => {
  return (
    <div>
      <label>{scale}:</label>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </div>
  );
};

// Pattern: State in parent, passed as props to children
// Children communicate changes via callbacks (also props)
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: Storing props in state (usually wrong)
const Child = ({ initialCount }) => {
  const [count, setCount] = useState(initialCount);
  // Problem: If parent changes initialCount, this component won't update
  
  return <div>{count}</div>;
};

// ✅ Fix: Use prop directly
const Child = ({ count }) => {
  return <div>{count}</div>;
};

// ✅ Or: Use useEffect to sync (if you really need local state)
const Child = ({ initialCount }) => {
  const [count, setCount] = useState(initialCount);
  
  useEffect(() => {
    setCount(initialCount); // Sync with prop changes
  }, [initialCount]);
  
  return <div>{count}</div>;
};

// ❌ Mistake 2: Mutating props
const Child = ({ user }) => {
  user.name = "Jane"; // ❌ NEVER mutate props!
  return <div>{user.name}</div>;
};

// ✅ Fix: Use callback to request parent to update
const Child = ({ user, onUpdateUser }) => {
  const handleClick = () => {
    onUpdateUser({ ...user, name: "Jane" }); // ✅ Notify parent
  };
  
  return <button onClick={handleClick}>Update</button>;
};

// ❌ Mistake 3: Using props when state is needed
const Input = ({ value }) => {
  // value is prop, can't change it for user typing
  return <input value={value} />; // ❌ Doesn't work for user input!
};

// ✅ Fix: Use state for user input
const Input = ({ initialValue, onSubmit }) => {
  const [value, setValue] = useState(initialValue);
  
  return (
    <div>
      <input 
        value={value} 
        onChange={(e) => setValue(e.target.value)}
      />
      <button onClick={() => onSubmit(value)}>Submit</button>
    </div>
  );
};
```

**Production Example: Complete Pattern**

```jsx
import { useState, useCallback } from 'react';

// Parent: Manages state
const ShoppingApp = () => {
  // STATE: Managed by this component
  const [products] = useState([
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Mouse', price: 29 },
    { id: 3, name: 'Keyboard', price: 79 },
  ]);
  
  const [cart, setCart] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  
  // Callbacks to update state
  const addToCart = useCallback((product) => {
    setCart(prevCart => {
      const existing = prevCart.find(item => item.id === product.id);
      if (existing) {
        return prevCart.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prevCart, { ...product, quantity: 1 }];
    });
  }, []);
  
  const removeFromCart = useCallback((productId) => {
    setCart(prevCart => prevCart.filter(item => item.id !== productId));
  }, []);
  
  // Derived data (not stored in state)
  const filteredProducts = products.filter(p =>
    p.name.toLowerCase().includes(searchQuery.toLowerCase())
  );
  
  const cartTotal = cart.reduce((sum, item) => 
    sum + (item.price * item.quantity), 0
  );
  
  return (
    <div className="shopping-app">
      {/* Pass state as props to children */}
      <SearchBar 
        query={searchQuery}
        onQueryChange={setSearchQuery}  // Callback prop
      />
      
      <ProductList 
        products={filteredProducts}     // Data prop (derived from state)
        onAddToCart={addToCart}         // Callback prop
      />
      
      <Cart 
        items={cart}                    // Data prop (state)
        total={cartTotal}               // Data prop (derived)
        onRemoveItem={removeFromCart}   // Callback prop
      />
    </div>
  );
};

// Child 1: Receives props, updates parent via callback
const SearchBar = ({ query, onQueryChange }) => {
  return (
    <input
      type="text"
      placeholder="Search products..."
      value={query}                              // Prop
      onChange={(e) => onQueryChange(e.target.value)}  // Callback
    />
  );
};

// Child 2: Receives props, has own local state
const ProductList = ({ products, onAddToCart }) => {
  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}              // Prop
          onAddToCart={onAddToCart}      // Prop (callback)
        />
      ))}
    </div>
  );
};

// Grandchild: Receives props, may have local UI state
const ProductCard = ({ product, onAddToCart }) => {
  const [showDetails, setShowDetails] = useState(false); // Local state
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      
      <button onClick={() => setShowDetails(!showDetails)}>
        {showDetails ? 'Hide' : 'Show'} Details
      </button>
      
      {showDetails && (
        <p>Product details here...</p>
      )}
      
      <button onClick={() => onAddToCart(product)}>
        Add to Cart
      </button>
    </div>
  );
};

// Child 3: Receives props, notifies parent
const Cart = ({ items, total, onRemoveItem }) => {
  return (
    <div className="cart">
      <h2>Shopping Cart</h2>
      {items.length === 0 ? (
        <p>Cart is empty</p>
      ) : (
        <>
          {items.map(item => (
            <div key={item.id} className="cart-item">
              <span>{item.name} x{item.quantity}</span>
              <span>${item.price * item.quantity}</span>
              <button onClick={() => onRemoveItem(item.id)}>Remove</button>
            </div>
          ))}
          <div className="cart-total">
            <strong>Total: ${total.toFixed(2)}</strong>
          </div>
        </>
      )}
    </div>
  );
};

export default ShoppingApp;

// Architecture summary:
// - State: Centralized in ShoppingApp
// - Props: Data flows down to children
// - Callbacks: Children notify parent of events
// - Local state: UI-specific state (like showDetails) stays local
// - Derived data: Calculated from state, not stored separately
```

**Interview Tips:**
- **State** is internal and mutable, **Props** are external and immutable
- Props enable **parent-to-child** communication (data down)
- Callbacks (passed as props) enable **child-to-parent** communication (events up)
- State changes trigger re-renders; prop changes (from parent) also trigger re-renders
- **Lift state up** to nearest common ancestor when multiple components need same data
- Don't store props in state unless you have a specific reason (controlled → uncontrolled)
- State should be minimal - derive values when possible
- Understanding the difference is fundamental to React's unidirectional data flow model

</details>
